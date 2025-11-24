# Beans e Inyección de Dependencias en Java (Spring Boot)

Este documento detalla los conceptos fundamentales de los Beans, la Inversión de Control (IoC) y la Inyección de Dependencias (DI) en el ecosistema de Spring, utilizando ejemplos modernos con **Java 21**.

---

## 1. Conceptos Fundamentales

### ¿Qué es un Bean?

En el contexto de **Spring Framework**, un **Bean** es un objeto que es instanciado, ensamblado y gestionado por el contenedor de Spring (IoC Container).

*   **Java Bean (Clásico)**: Una clase simple que sigue ciertas convenciones (serializable, constructor sin argumentos, getters/setters).
*   **Spring Bean**: Cualquier objeto gestionado por el contenedor de Spring. No necesita seguir todas las reglas de un Java Bean clásico.

### Inversión de Control (IoC)

Es un principio de diseño donde el control del flujo del programa se invierte. En lugar de que tu código controle la creación y gestión de objetos, un framework (Spring) se encarga de ello.

*   **Sin IoC**: Tú creas las instancias con `new Service()`.
*   **Con IoC**: El contenedor crea la instancia y te la da cuando la necesitas.

### Inyección de Dependencias (DI)

Es el patrón de diseño que implementa IoC. Permite que los objetos definan sus dependencias (los otros objetos con los que trabajan) a través de argumentos de constructor o métodos, y el contenedor se encarga de "inyectar" esas dependencias cuando crea el bean.

---

## 2. Tipos de Inyección de Dependencias

Existen tres formas principales de inyectar dependencias. En Java moderno, la **inyección por constructor** es la práctica estándar.

### A. Inyección por Constructor (Recomendada)

Es la forma más segura y recomendada. Garantiza que el objeto no se pueda crear sin sus dependencias y permite que los campos sean `final` (inmutables).

```java
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private final UserRepository userRepository;
    private final EmailService emailService;

    // Spring inyecta automáticamente las dependencias aquí
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }

    public void registerUser(String username) {
        // Lógica de negocio
        var user = userRepository.save(username);
        emailService.sendWelcomeEmail(user);
    }
}
```

> [!TIP]
> Desde Spring 4.3, si la clase tiene un único constructor, no es necesario anotar el constructor con `@Autowired`.

### B. Inyección por Setter

Útil para dependencias opcionales o que pueden cambiar en tiempo de ejecución. No permite campos `final`.

```java
@Service
public class NotificationService {

    private LoggerService logger;

    @Autowired // Necesario en setters
    public void setLogger(LoggerService logger) {
        this.logger = logger;
    }
}
```

### C. Inyección de Campo (Field Injection) - NO RECOMENDADA

Aunque es muy común en tutoriales antiguos por su brevedad, se considera una mala práctica porque oculta dependencias, dificulta los tests unitarios y viola la inmutabilidad.

```java
@Service
public class BadService {
    @Autowired // Evitar esto
    private UserRepository userRepository;
}
```

---

## 3. Definición de Beans

### A. Escaneo de Componentes (Stereotypes)

Spring escanea el classpath buscando clases anotadas para crear Beans automáticamente.

*   `@Component`: Componente genérico.
*   `@Service`: Capa de lógica de negocio.
*   `@Repository`: Capa de acceso a datos (maneja excepciones de DB).
*   `@Controller` / `@RestController`: Capa web (MVC o API REST).

### B. Configuración Explícita (@Configuration y @Bean)

Para librerías de terceros o configuraciones complejas donde no puedes anotar la clase directamente.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.thirdparty.PaymentGateway;

@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        // Configuración manual del bean
        var gateway = new PaymentGateway();
        gateway.setApiKey("secret-key");
        return gateway;
    }
}
```

---

## 4. Ciclo de Vida y Scopes (Ámbitos)

### Scopes de un Bean

El scope define cuántas instancias de un bean se crean y cuánto tiempo viven. Se define con `@Scope`.

| Scope | Descripción |
| :--- | :--- |
| **Singleton** (Default) | Una sola instancia por contenedor IoC. Es el valor por defecto. |
| **Prototype** | Se crea una nueva instancia cada vez que se solicita el bean. |
| **Request** | Una instancia por cada petición HTTP (Web). |
| **Session** | Una instancia por cada sesión HTTP (Web). |
| **Application** | Una instancia por ciclo de vida del ServletContext (Web). |

### Hooks de Ciclo de Vida

Puedes ejecutar lógica justo después de crear el bean o antes de destruirlo.

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class DatabaseConnection {

    @PostConstruct
    public void init() {
        System.out.println("Bean creado y dependencias inyectadas. Abriendo conexión...");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Cerrando conexión antes de destruir el bean...");
    }
}
```

---

## 5. Ejemplo Completo (Java 21)

Un ejemplo integrando un Controlador, Servicio y Repositorio simulado.

**Repository:**
```java
@Repository
public class ProductRepository {
    public List<String> findAll() {
        return List.of("Laptop", "Mouse", "Keyboard"); // Java 9+ List.of
    }
}
```

**Service:**
```java
@Service
public class ProductService {
    
    private final ProductRepository repository;

    public ProductService(ProductRepository repository) {
        this.repository = repository;
    }

    public List<String> getProducts() {
        // Uso de var (Java 10+)
        var products = repository.findAll();
        return products.stream()
                .map(String::toUpperCase)
                .toList(); // Java 16+ Stream.toList()
    }
}
```

**Controller:**
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService service;

    public ProductController(ProductService service) {
        this.service = service;
    }

    @GetMapping
    public List<String> listProducts() {
        return service.getProducts();
    }
}
```
