El **dominio** es el corazón de tu aplicación. Representa:

- Los conceptos del negocio (Libro, Usuario, Pedido)
- Las reglas de negocio (un libro prestado no se puede prestar de nuevo)
- La lógica que NO depende de frameworks o bases de datos

**Ejemplo:** Si haces una app de e-commerce, el dominio incluye conceptos como: Producto, Carrito, Orden, Cliente, Pago, etc.

## *¿Qué es el Domain?

El **dominio** son todos los conceptos y reglas de tu negocio:

### ✅SÍ es parte del dominio:

- **Entidades principales** del negocio (Libro, Usuario, Pedido, Producto)
- **Reglas de negocio** (un usuario no puede tener más de 3 libros prestados)
- **Excepciones del negocio** (LibroNoDisponibleException)
- **Enums del negocio** (EstadoLibro, TipoPago, RolUsuario)
- **Value Objects** (Dirección, Dinero, Email)


### ❌NO es parte del dominio:

- **Controllers** (son infraestructura HTTP)
- **DTOs** de Request/Response (son para comunicación)
- **Repositorios** JPA (son implementación técnica)
- **Configuraciones** (son detalles técnicos)

### 📁Ejemplo en arquitectura en capas:


```
src/main/java/com/biblioteca/
│
├── controller/              # ← Capa de Presentación
│   ├── LibroController.java
│   ├── UsuarioController.java
│   └── PrestamoController.java
│
├── service/                 # ← Capa de Servicio (DOMINIO aquí)
│   ├── LibroService.java
│   ├── UsuarioService.java
│   └── PrestamoService.java
│
├── repository/              # ← Capa de Persistencia
│   ├── LibroRepository.java
│   ├── UsuarioRepository.java
│   └── PrestamoRepository.java
│
├── model/                   # ← DOMINIO (Entidades)
│   ├── Libro.java
│   ├── Usuario.java
│   └── Prestamo.java
│
├── dto/                     # ← DTOs (Request/Response)
│   ├── request/
│   │   ├── CrearLibroRequest.java
│   │   └── PrestarLibroRequest.java
│   └── response/
│       ├── LibroResponse.java
│       └── PrestamoResponse.java
│
├── exception/               # ← DOMINIO (Excepciones de negocio)
│   ├── LibroNoDisponibleException.java
│   ├── UsuarioNoEncontradoException.java
│   └── GlobalExceptionHandler.java
│
├── enums/                   # ← DOMINIO (Enums)
│   ├── EstadoLibro.java
│   ├── RolUsuario.java
│   └── TipoPrestamo.java
│
└── config/                  # ← Configuración
    ├── SecurityConfig.java
    └── DatabaseConfig.java
```


### 📦 ¿Qué va en cada carpeta?

##### 1. **`model/`** - Entidades del DOMINIO

```java
// model/Libro.java

@Entity
@Table(name = "libros")
public class Libro {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String titulo;
    
    private String autor;
    
    @Column(unique = true)
    private String isbn;
    
    @Enumerated(EnumType.STRING)
    private EstadoLibro estado;
    
    @ManyToOne
    @JoinColumn(name = "usuario_id")
    private Usuario prestadoA;
    
    // ⚠️ En arquitectura tradicional, la lógica de negocio 
    // puede ir aquí o en el Service (preferiblemente Service)
    public void prestar(Usuario usuario) {
        if (this.estado != EstadoLibro.DISPONIBLE) {
            throw new LibroNoDisponibleException("El libro no está disponible");
        }
        this.prestadoA = usuario;
        this.estado = EstadoLibro.PRESTADO;
    }
    
    public void devolver() {
        this.prestadoA = null;
        this.estado = EstadoLibro.DISPONIBLE;
    }
    
    // Constructor, getters, setters
}
```


##### 2. **`service/`** - Lógica de negocio (DOMINIO)

```java
// service/LibroService.java
@Service
public class LibroService {
    
    @Autowired
    private LibroRepository libroRepository;
    
    @Autowired
    private UsuarioRepository usuarioRepository;
    
    // ← LÓGICA DE NEGOCIO aquí
    public Libro prestarLibro(Long libroId, Long usuarioId) {
        
        // 1. Validar reglas de negocio
        Libro libro = libroRepository.findById(libroId)
            .orElseThrow(() -> new LibroNoEncontradoException(libroId));
        
        Usuario usuario = usuarioRepository.findById(usuarioId)
            .orElseThrow(() -> new UsuarioNoEncontradoException(usuarioId));
        
        // 2. Validar que el usuario no tenga más de 3 libros prestados
        long librosActivos = libroRepository.countByPrestadoAAndEstado(
            usuario, EstadoLibro.PRESTADO
        );
        
        if (librosActivos >= 3) {
            throw new LimitePrestamosExcedidoException(
                "El usuario ya tiene 3 libros prestados"
            );
        }
        
        // 3. Aplicar la lógica (puede estar en la entidad o aquí)
        libro.prestar(usuario);
        
        // 4. Persistir
        return libroRepository.save(libro);
    }
    
    public List<Libro> obtenerDisponibles() {
        return libroRepository.findByEstado(EstadoLibro.DISPONIBLE);
    }
    
    public Libro crearLibro(Libro libro) {
        // Validar que el ISBN no exista
        if (libroRepository.existsByIsbn(libro.getIsbn())) {
            throw new IsbnDuplicadoException(
                "Ya existe un libro con ese ISBN"
            );
        }
        
        libro.setEstado(EstadoLibro.DISPONIBLE);
        return libroRepository.save(libro);
    }
}
```


##### 3. **`exception/`** - Excepciones del DOMINIO

```java
// exception/LibroNoDisponibleException.java
public class LibroNoDisponibleException extends RuntimeException {
    public LibroNoDisponibleException(String mensaje) {
        super(mensaje);
    }
}

// exception/UsuarioNoEncontradoException.java
public class UsuarioNoEncontradoException extends RuntimeException {
    public UsuarioNoEncontradoException(Long id) {
        super("No se encontró el usuario con ID: " + id);
    }
}

// exception/LimitePrestamosExcedidoException.java
public class LimitePrestamosExcedidoException extends RuntimeException {
    public LimitePrestamosExcedidoException(String mensaje) {
        super(mensaje);
    }
}

// exception/GlobalExceptionHandler.java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(LibroNoDisponibleException.class)
    public ResponseEntity<ErrorResponse> handleLibroNoDisponible(
            LibroNoDisponibleException ex) {
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }
    
    @ExceptionHandler(UsuarioNoEncontradoException.class)
    public ResponseEntity<ErrorResponse> handleUsuarioNoEncontrado(
            UsuarioNoEncontradoException ex) {
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
}
```


##### 4. **`enums/`** - Enumeraciones del DOMINIO

```java
// enums/EstadoLibro.java
public enum EstadoLibro {
    DISPONIBLE,
    PRESTADO,
    MANTENIMIENTO,
    PERDIDO
}

// enums/RolUsuario.java
public enum RolUsuario {
    USUARIO,
    BIBLIOTECARIO,
    ADMINISTRADOR
}

// enums/TipoPrestamo.java
public enum TipoPrestamo {
    CORTO_PLAZO(7),      // 7 días
    MEDIANO_PLAZO(14),   // 14 días
    LARGO_PLAZO(30);     // 30 días
    
    private final int dias;
    
    TipoPrestamo(int dias) {
        this.dias = dias;
    }
    
    public int getDias() {
        return dias;
    }
}
```

```
src/main/java/com/biblioteca/
│
├── model/                   # ← DOMINIO
│   ├── Libro.java
│   ├── Usuario.java
│   └── Prestamo.java
│
├── service/                 # ← DOMINIO (lógica de negocio)
│   ├── LibroService.java
│   ├── UsuarioService.java
│   └── PrestamoService.java
│
├── exception/               # ← DOMINIO (excepciones de negocio)
│   ├── LibroNoDisponibleException.java
│   ├── UsuarioNoEncontradoException.java
│   └── LimitePrestamosExcedidoException.java
│
└── enums/                   # ← DOMINIO (enums del negocio)
    ├── EstadoLibro.java
    ├── RolUsuario.java
    └── TipoPrestamo.java
```
Eso es TODO el dominio.


