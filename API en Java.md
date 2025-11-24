# API en Java - Arquitectura en capas

## Índice
- [Controller](#controller)
- [Annotations del Controller Spring](#annotations-del-controller-spring)
- [Service (@Service)](#service-service)
- [Annotations del Service en Spring](#annotations-del-service-en-spring)
- [Repository (@Repository)](#repository-repository)
- [Annotations del Repository](#annotations-del-repository)
- [Resumen de Sintaxis JPA (JPQL)](#resumen-de-sintaxis-jpa-jpql)
- [DTO (Data Transfer Object)](#dto-data-transfer-object)
- [Tipos de Anotaciones en Java y Spring](#tipos-de-anotaciones-en-java-y-spring)
    - [Nativas de Java](#1-nativas-de-java-estándar-jakarta-ee)
    - [Propias de Spring Framework](#2-propias-de-spring-framework)
    - [Manejo Global de Errores](#manejo-global-de-errores-restcontrolleradvice)
- [Spring Security](#spring-security)
- [Mappers (MapStruct)](#mappers-mapstruct)
- [Documentación Automática (Swagger / OpenAPI)](#documentación-automática-swagger--openapi)
- [Testing Profesional en Spring Boot](#testing-profesional-en-spring-boot)

## Controller 
### *@RestController:* Solo recibe la petición HTTP, valida datos básicos y llama al servicio. Retorna la respuesta (DTO).

```java
package com.empresa.proyecto.controller;

import com.empresa.proyecto.dto.UsuarioRequest;
import com.empresa.proyecto.dto.UsuarioResponse;
import com.empresa.proyecto.service.UsuarioService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;


@RestController
@RequestMapping("/api/v1/usuarios") // 1. Versionado de API (Muy importante en empresas)
public class UsuarioController {

    private final UsuarioService usuarioService;

    // 2. Inyección de Dependencias por Constructor (Sin @Autowired, Sin Lombok)
    // Garantiza inmutabilidad y facilita los tests unitarios.
    public UsuarioController(UsuarioService usuarioService) {
        this.usuarioService = usuarioService;
    }

    /**
     * Crear un nuevo usuario.
     * @param request DTO con datos de entrada (incluye password, validaciones).
     * @return 201 Created y el DTO de respuesta (sin password).
     */
    @PostMapping
    public ResponseEntity<UsuarioResponse> crear(@Valid @RequestBody UsuarioRequest request) {
        // 3. @Valid: Si los datos están mal, Spring corta aquí y lanza error 400.
        // El Controller NO tiene lógica, solo delega al servicio.
        UsuarioResponse nuevoUsuario = usuarioService.crear(request);

        // 4. Retorno 201 Created (Estándar REST para creaciones)
        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(nuevoUsuario);
    }

    /**
     * Buscar un usuario por su ID.
     * @param id Identificador único.
     * @return 200 OK si existe.
     */
    @GetMapping("/{id}")
    public ResponseEntity<UsuarioResponse> buscarPorId(@PathVariable Long id) {
        // Si el usuario no existe, el Service debería lanzar una excepción (ej. UsuarioNotFoundException)
        // que será capturada por un @RestControllerAdvice global.
        UsuarioResponse usuario = usuarioService.buscarPorId(id);
        
        return ResponseEntity.ok(usuario); // 200 OK
    }
}
```

## Annotations del Controller Spring

### `@RestController`
**¿Qué hace?**  
Indica que esta clase maneja peticiones web y que **todas las respuestas serán JSON**, no vistas HTML.  
> Es una combinación de `@Controller` + `@ResponseBody`.

---

### `@RequestMapping("/api/v1/usuarios")`
**¿Qué hace?**  
Define la **ruta base** para todos los métodos del controlador.

**Uso:**  
Colocándolo en la clase evitas repetir `/api/v1/usuarios` en cada endpoint.

---

### `@PostMapping` / `@GetMapping` / `@PutMapping` / `@DeleteMapping`
**¿Qué hacen?**  
Mapean **métodos HTTP** a métodos Java.

- `@PostMapping` → Crear  
- `@GetMapping` → Leer  
- `@PutMapping` → Actualizar  
- `@DeleteMapping` → Eliminar  

---

### `@RequestBody`
**¿Qué hace?**  
Indica que Spring debe tomar el **JSON del cuerpo de la petición** y convertirlo en un objeto Java (DTO).  

> Sin esta anotación, tu objeto llegaría con valores `null`.

---

### `@PathVariable`
**¿Qué hace?**  
Extrae valores directamente desde la **URL**.

**Ejemplo:**  
En `/usuarios/5`, extrae el `5` y lo asigna a tu variable `Long id`.

---

### `@Valid` (Jakarta Validation)
**¿Qué hace?**  
Activa las validaciones del DTO (como `@NotBlank`, `@Email`, etc.).  
Si alguna falla, Spring **detiene la ejecución antes de entrar al método**.  
  
<br><br>

# Service (@Service):  
Aquí vive la lógica de negocio ("La magia"). Realiza cálculos, validaciones complejas y coordina llamadas a repositorios.

```java
package com.empresa.proyecto.service;

import com.empresa.proyecto.dto.UsuarioRequest;
import com.empresa.proyecto.dto.UsuarioResponse;
import com.empresa.proyecto.exception.EmailYaRegistradoException;
import com.empresa.proyecto.exception.UsuarioNoEncontradoException;
import com.empresa.proyecto.model.Usuario;
import com.empresa.proyecto.repository.UsuarioRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UsuarioService {

    private final UsuarioRepository usuarioRepository;
    // Aquí inyectarías también un PasswordEncoder real (ej. BCrypt)
    // private final PasswordEncoder passwordEncoder; 

    public UsuarioService(UsuarioRepository usuarioRepository) {
        this.usuarioRepository = usuarioRepository;
    }

    /**
     * Crea un usuario con validaciones de negocio.
     * @Transactional asegura que si algo falla, no se guarde nada a medias.
     */
    @Transactional
    public UsuarioResponse crear(UsuarioRequest request) {
        // 1. Validación de Negocio: ¿El email ya existe?
        // Esto NO lo puede saber el Controller ni el DTO.
        if (usuarioRepository.existsByEmail(request.email())) {
            throw new EmailYaRegistradoException("El email " + request.email() + " ya está en uso");
        }

        // 2. Mapeo: Convertir DTO (Request) a Entidad (DB)
        Usuario usuario = new Usuario();
        usuario.setNombre(request.nombre());
        usuario.setEmail(request.email());
        // usuario.setPassword(passwordEncoder.encode(request.password())); // Encriptar siempre!
        usuario.setPassword(request.password()); // (Simulado por ahora)

        // 3. Guardado en Base de Datos
        Usuario usuarioGuardado = usuarioRepository.save(usuario);

        // 4. Mapeo Inverso: Convertir Entidad guardada a DTO (Response)
        return new UsuarioResponse(
                usuarioGuardado.getId(),
                usuarioGuardado.getNombre(),
                usuarioGuardado.getEmail()
        );
    }

    /**
     * Busca un usuario o lanza excepción si no existe.
     * @Transactional(readOnly = true) optimiza el rendimiento para solo lectura.
     */
    @Transactional(readOnly = true)
    public UsuarioResponse buscarPorId(Long id) {
        return usuarioRepository.findById(id)
                .map(usuario -> new UsuarioResponse(
                        usuario.getId(),
                        usuario.getNombre(),
                        usuario.getEmail()
                ))
                .orElseThrow(() -> new UsuarioNoEncontradoException("Usuario no encontrado con id: " + id));
    }
}
```

## Annotations del Service en Spring

### `@Service`
**¿Qué hace?**  
Marca la clase como un **Componente de Servicio** dentro del contenedor de Spring.

- **Semántica:** Indica que aquí vive la **lógica de negocio**.  
- **Técnicamente:** Es equivalente a `@Component`, pero usar `@Service` mejora la legibilidad y permite que herramientas o aspectos identifiquen claramente la capa de negocio.

---

### `@Transactional`
**¿Qué hace?**  
Define el **alcance transaccional** de las operaciones que se ejecutan dentro del método.

**Uso crítico:**  
Si un método hace 3 operaciones en la base de datos y la tercera falla, `@Transactional` hace que las dos primeras **se deshagan automáticamente** (rollback).

**Modo lectura:**  
`@Transactional(readOnly = true)`  
Útil para consultas o listados. Mejora el rendimiento porque Hibernate no necesita vigilar cambios en las entidades.

---

### `@Autowired` (Opcional / No recomendada en campos)
**¿Qué hace?**  
Indica que Spring debe **inyectar una dependencia**.

**Nota:**  
En código moderno se prefiere **inyección por constructor**, donde `@Autowired` es implícito, por lo que ya no es necesario escribirlo.  
Sin embargo, sigue apareciendo mucho en código legacy.

---

### `@Async`
**¿Qué hace?**  
Ejecuta el método en un **hilo separado**, en segundo plano.

**Uso:**  
Perfecto para tareas lentas que no deben bloquear al usuario:

- enviar un email  
- generar un reporte pesado  
- procesar archivos  

**Requiere:**  
Habilitar `@EnableAsync` en la configuración principal.

---





### Repository (@Repository): 
- Interfaz que extiende de JpaRepository. Solo se encarga de hablar con la base de datos (PostgreSQL en tu caso).

```java
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
    boolean existsByEmail(String email);
}
```

```java
package com.empresa.proyecto.repository;

import com.empresa.proyecto.model.Usuario;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.time.LocalDate;
import java.util.List;
import java.util.Optional;

@Repository // 1. Marca la interfaz como componente de Spring
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {

    // --- A. MAGIC METHODS (Spring los implementa solo por el nombre) ---
    
    // SELECT * FROM usuarios WHERE email = ?
    boolean existsByEmail(String email);

    // SELECT * FROM usuarios WHERE email = ?
    Optional<Usuario> findByEmail(String email);

    // SELECT * FROM usuarios WHERE fecha_creacion > ?
    List<Usuario> findByFechaCreacionAfter(LocalDate fecha);


    // --- B. JPQL (Consultas con Objetos - Estándar y Recomendado) ---
    
    // Busca usuarios activos y administradores.
    // Fíjate que usamos la CLASE 'Usuario' y sus atributos, NO la tabla.
    @Query("SELECT u FROM Usuario u WHERE u.activo = true AND u.rol = 'ADMIN'")
    List<Usuario> buscarAdminsActivos();

    // Uso de JOIN implícito y parámetros nombrados (:dominio)
    @Query("SELECT u FROM Usuario u WHERE u.email LIKE %:dominio%")
    List<Usuario> buscarPorDominioEmail(@Param("dominio") String dominio);


    // --- C. NATIVE QUERY (SQL Puro - Solo para casos especiales) ---
    
    // A veces necesitas funciones específicas de la DB (ej. Postgres)
    @Query(value = "SELECT * FROM usuarios u WHERE u.deleted_at IS NOT NULL", nativeQuery = true)
    List<Usuario> buscarEliminados();


    // --- D. MODIFICACIONES (UPDATE / DELETE) ---
    
    // @Modifying es OBLIGATORIO para queries que cambian datos.
    @Modifying
    @Query("UPDATE Usuario u SET u.activo = false WHERE u.ultimoLogin < :fecha")
    void desactivarUsuariosInactivos(@Param("fecha") LocalDate fecha);
}
```

## 🗂️ Annotations del Repository

### `@Repository`
**¿Qué hace?**  
Indica que esta interfaz es un **DAO (Data Access Object)**.

**Importante:**  
Habilita la **traducción automática de excepciones SQL** a excepciones de Spring (`DataAccessException`).

---

### `@Query`
**¿Qué hace?**  
Permite definir consultas manuales en **JPQL o SQL**.

**¿Por qué usarla?**  
- Cuando los nombres de métodos derivados (`findByNombreAndApellidoAndEdad...`) se vuelven inmanejables.  
- Cuando necesitas **JOINS complejos** o lógica que no se puede expresar con métodos mágicos.

---

### `@Param`
**¿Qué hace?**  
Vincula un parámetro del método Java con la variable en la query (`:nombreVariable`).

**Importante:**  
Mejora la **claridad** del código y evita errores al mapear parámetros.

---

### `@Modifying`
**¿Qué hace?**  
Indica que la `@Query` **no es un SELECT**, sino un `UPDATE` o `DELETE`.

**Ojo:**  
Si olvidas agregarlo en un UPDATE, tendrás un **error en tiempo de ejecución**.

---

## 📌 Resumen de Sintaxis JPA (JPQL)

Recuerda: **JPQL trabaja sobre Entidades, no sobre Tablas**.

**SQL**
```sql
SELECT * FROM users_table WHERE user_email = 'x'
```

**JPQL**
```java
SELECT u FROM Usuario u WHERE u.email = 'x'
```

### DTO (Data Transfer Object): 
- Usa DTOs (pueden ser record en Java 21 o clases con @Data de lombok) para recibir datos (Request) y enviar datos (Response).
Esto desacopla tu base de datos de lo que ve el cliente de la API y evita problemas de seguridad o recursión infinita en JSON.

#### ¿Por qué usar DTOs? (Beneficios Reales)
Aunque parecen clases "tontas" sin lógica, son vitales en entornos profesionales por tres razones:

1.  **Seguridad (Evitar fugas):**
    Si devuelves tu Entidad `Usuario` directa, podrías enviar por error el campo `password` o `sueldo`. Con un `UsuarioResponse`, defines exactamente qué campos públicos quieres mostrar.

2.  **Desacoplamiento (Estabilidad de API):**
    Si cambias tu base de datos (ej. cambias `nombreCompleto` por `nombre` y `apellido`), no rompes el Frontend. Solo ajustas el mapeo en el DTO y la API sigue devolviendo el mismo JSON de siempre.

3.  **Validación Contextual:**
    - Al **Crear**, el password es obligatorio (`@NotNull`).
    - Al **Editar**, el password es opcional.
    Usando `CreateUserRequest` y `UpdateUserRequest` puedes tener reglas distintas para cada caso.

### Utilizar records en Java como DTO

```java
public record UsuarioDTO(
    @NotBlank String nombre,
    @Email String email
) {}
```

### Tipos de DTOs en una Arquitectura Profesional

1.  **Request DTO (`UsuarioRequest`):**
    *   **Origen:** Viene de afuera (Cliente/Frontend).
    *   **Destino:** Controller -> Service.
    *   **Características:** Tiene validaciones (`@NotNull`), password en texto plano (si es registro), datos crudos.
    
    ```java
    public record RegistroUsuarioRequest(
    @NotBlank(message = "El nombre es obligatorio")
    String nombre,

    @Email(message = "Email inválido")
    String email,

    @Size(min = 8, message = "Mínimo 8 caracteres")
    String password // ⚠️ Llega en texto plano para ser procesada
) {}
```

2.  **Response DTO (`UsuarioResponse`):**
    *   **Origen:** Sale de tu Service.
    *   **Destino:** Controller -> Cliente.
    *   **Características:** Datos limpios, sin password, fechas formateadas, URLs completas.
    
    ```java
    public record UsuarioResponse(
    Long id,
    String nombre,
    String email,
    String avatarUrl,
    LocalDate fechaCreacion,
    Boolean activo
) {}
```

3.  **Internal DTO (o simplemente "DTO" o "Model"):**
    *   **Origen:** Un Servicio A.
    *   **Destino:** Un Servicio B (o una cola de mensajes, o un reporte PDF).
    *   **Uso:** Para comunicar módulos internos sin exponer datos privados ni depender de la estructura web.
    
    ```java
public record DatosFacturacionInterna(
    Long usuarioId,
    String cuit,          // ⚠️ Dato sensible, no va al frontend
    String direccionFiscal,
    BigDecimal saldoPendiente // BigDecimal para dinero, siempre
) {}
```
## Tipos de Anotaciones en Java y Spring

## 1. Nativas de Java (Estándar Jakarta EE)

Estas forman parte de la **especificación oficial** de Java Enterprise.  
Si cambias Spring por Quarkus, Micronaut o Jakarta EE puro, **siguen funcionando igual**.

### ✔️ Validación (`jakarta.validation.*`)
- `@Valid`
- `@NotNull`, `@NotBlank`, `@Email`
- `@Size`, `@Min`, `@Max`

### ✔️ Persistencia / Base de Datos (`jakarta.persistence.*`)
- `@Entity`
- `@Table`
- `@Id`, `@GeneratedValue`
- `@Column`
- `@OneToMany`, `@ManyToOne`, `@JoinColumn`
- `@Transient` (Ignorar campo en la DB)

---

## 2. Propias de Spring Framework

Estas existen **solo en Spring**.  
Si no usas Spring, **no funcionan**.

### ✔️ Inyección y Componentes (`org.springframework.stereotype.*`)
- `@Component`
- `@Service`
- `@Repository`
- `@Controller`, `@RestController`

### ✔️ Web / API (`org.springframework.web.bind.annotation.*`)
- `@RequestMapping`
- `@GetMapping`, `@PostMapping`, etc.
- `@RequestBody`, `@PathVariable`, `@RequestParam`
- `@RestControllerAdvice`, `@ExceptionHandler`

### ✔️ Datos (`org.springframework.data.jpa.repository.*`)
- `@Query`
- `@Modifying`
- `@Param`

### ✔️ Lógica (`org.springframework.transaction.annotation.*`)
- `@Transactional`

---

## ⚠️ El caso especial: `@Transactional`

Existen **dos versiones**:

1. `jakarta.transaction.Transactional` → Estándar Java  
2. `org.springframework.transaction.annotation.Transactional` → Spring

### ¿Cuál usar en un proyecto Spring?
➡️ **Siempre la de Spring (`org.springframework...`).**

### ¿Por qué?
La de Spring es **mucho más completa**:

- `readOnly = true`
- `timeout`
- niveles de aislamiento (`isolation`)
- reglas de rollback (`rollbackFor`)
- integración profunda con el contexto Spring

La versión estándar de Java es **muy limitada** en comparación.

---

###Manejar excepciones de manera global:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UsuarioNotFoundException.class)
    public ResponseEntity<String> handleUsuarioNotFoundException(UsuarioNotFoundException ex) {
        return ResponseEntity.notFound().build();
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneralException(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
    }
}
```

## 🛡️ Manejo Global de Errores (`@RestControllerAdvice`)
### ¿Qué es `@RestControllerAdvice`?
Es un "Interceptor Global". Imagina que es una red de seguridad que pones debajo de todos tus Controladores.
Si **cualquier** controlador lanza una excepción (error), esta clase la "atrapa" antes de que llegue al usuario, permitiéndote transformar ese error feo en un JSON bonito.

---

### Paso 1: Definir tu estructura de error (El DTO de Error)
Primero, necesitamos una cajita (DTO) para enviar el error al cliente de forma ordenada.

```java
// ErrorResponse.java
public record ErrorResponse(
    String mensaje,
    String codigo, // Ej: "P-404" (P de Producto)
    LocalDateTime fecha
) {}
```

---

### Paso 2: Crear el Manejador Global
Aquí ocurre la magia. Usamos `@ExceptionHandler` para decirle a Spring qué hacer con cada tipo de error.

```java
package com.empresa.proyecto.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice // 1. Marca esta clase como el "Manejador Global"
public class GlobalExceptionHandler {

    // CASO 1: Errores de Negocio (Ej: Usuario no encontrado)
    // Asumimos que creaste una excepción propia llamada 'RecursoNoEncontradoException'
    @ExceptionHandler(RecursoNoEncontradoException.class)
    public ResponseEntity<ErrorResponse> manejarNoEncontrado(RecursoNoEncontradoException ex) {
        
        ErrorResponse error = new ErrorResponse(
            ex.getMessage(),
            "404-NOT-FOUND",
            LocalDateTime.now()
        );

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    // CASO 2: Errores de Validación (@Valid falló)
    // Este es CLAVE. Spring lanza 'MethodArgumentNotValidException' cuando @Valid falla.
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> manejarValidaciones(MethodArgumentNotValidException ex) {
        
        // Extraemos los errores campo por campo (ej: "email": "formato incorrecto")
        Map<String, String> errores = new HashMap<>();
        
        ex.getBindingResult().getFieldErrors().forEach(error -> {
            errores.put(error.getField(), error.getDefaultMessage());
        });

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(errores);
    }

    // CASO 3: Error Genérico (Para todo lo que no esperábamos)
    // Evita que el usuario vea stack traces feos.
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> manejarErrorGeneral(Exception ex) {
        
        ErrorResponse error = new ErrorResponse(
            "Ocurrió un error interno inesperado. Contacte a soporte.",
            "500-INTERNAL",
            LocalDateTime.now()
        );

        // Opcional: Loguear el error real en consola para que tú lo veas
        // log.error("Error no controlado: ", ex);

        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

### ¿Cómo funciona el flujo ahora?
1.  El usuario manda un email inválido.
2.  El Controller intenta procesarlo con `@Valid`.
3.  Falla y lanza `MethodArgumentNotValidException`.
4.  **ANTES** de responder al usuario, Spring busca en tu `GlobalExceptionHandler`.
5.  Encuentra el método `manejarValidaciones`.
6.  Ejecuta tu lógica y devuelve un JSON limpio:

```json
{
    "email": "Debe tener formato de correo",
    "password": "Mínimo 8 caracteres"
}
```

## En resumen: ¿Cómo funciona la "magia"?

Es como darle a Spring un **diccionario de errores**.
Cuando ocurre una excepción en cualquier lugar de tu código, Spring corre a tu `@RestControllerAdvice` y busca la instrucción exacta para ese error.

### La lógica de búsqueda (El "Match")

Spring busca el `@ExceptionHandler` más específico posible:

1.  **Búsqueda Exacta:** ¿Tienes un handler para `UsuarioNoEncontradoException`? -> Úsalo.
2.  **Búsqueda por Herencia:** Si no, ¿tienes uno para `RuntimeException`? -> Úsalo.
3.  **Fallback (Red de Seguridad):** Si no, usa el genérico `Exception.class`.

```java
@RestControllerAdvice
public class ManejadorErrores {

    // 1. ESPECÍFICO (Prioridad Alta)
    // Si explota "UsuarioNoEncontrado", entra aquí.
    @ExceptionHandler(UsuarioNoEncontradoException.class)
    public ResponseEntity<?> manejarUsuario(...) { ... }

    // 2. INTERMEDIO (Prioridad Media)
    // Si explota un NullPointer, entra aquí.
    @ExceptionHandler(NullPointerException.class)
    public ResponseEntity<?> manejarNulos(...) { ... }

    // 3. GENÉRICO (Prioridad Baja - "Cajón de Sastre")
    // Si explota "DivisionByZero" (que no definimos arriba), cae aquí.
    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> manejarTodoLoDemas(...) { ... }
}
```

### Roles en la Arquitectura

| Quién | Qué hace | ¿Usa ResponseEntity? |
| :--- | :--- | :--- |
| **Controller** | Atiende al cliente cuando **todo sale bien**. | **SÍ** (Para devolver 200 OK y los datos). |
| **Service** | Hace la lógica. Si falla, **lanza Excepción**. | **NO** (Nunca toca HTTP). |
| **Advice** | Atiende al cliente cuando **algo sale mal**. | **SÍ** (Para devolver 404/500 y el error). |


*Basicamente el ResponseEntity es como el ResponseJson de Laravel* 

# 🔐 Spring Security

## ¿Qué es Spring Security?
Es un **Framework de Seguridad** completo y altamente personalizable que se integra nativamente con Spring.
No es solo "login". Es un sistema que intercepta **cada** petición que entra a tu servidor y decide si pasa o no, basándose en reglas complejas.

> 🛡️ **Analogía:** Piénsalo como un sistema de **Aduana y Control Fronterizo** para tu API.

---

## ¿Para qué se usa PROFESIONALMENTE? (Los 4 Pilares)

En una API real (bancos, e-commerce, redes sociales), Spring Security resuelve estos problemas críticos:

### 1. Autenticación Robusta (Authentication)
**¿Quién eres?**
No basta con usuario/password. Spring Security maneja:
*   **JWT (JSON Web Tokens):** El estándar moderno para APIs.
*   **OAuth2 / OpenID Connect:** "Iniciar sesión con Google/Facebook".
*   **LDAP / Active Directory:** Para empresas que usan credenciales corporativas.
*   **MFA (Multi-Factor Authentication):** Códigos por SMS/Email.

### 2. Autorización Granular (Authorization)
**¿Qué puedes hacer?**
No es solo "Admin o Usuario". Permite reglas complejas:
*   **Basada en Roles:** `hasRole('ADMIN')`
*   **Basada en Permisos:** `hasAuthority('LEER_REPORTES')`
*   **Basada en Datos (ACL):** "El usuario Juan puede editar la factura #50 **SOLO SI** él la creó o si es gerente de la sucursal Norte". (Esto se hace con `@PreAuthorize`).

### 3. Protección contra Ataques (Security Hardening)
Spring Security viene configurado por defecto para bloquear ataques invisibles:
*   **CSRF:** Evita peticiones maliciosas desde otras webs (se desactiva en APIs REST).
*   **CORS:** Controla qué dominios (ej. `mi-frontend.com`) pueden llamar a tu API.
*   **Session Fixation:** Evita el robo de sesiones.

### 4. Auditoría y Trazabilidad
Saber **quién** hizo **qué** y **cuándo**.
Spring Security inyecta el usuario actual (`Principal`) en cualquier parte del código, permitiendo logs como: *"El usuario admin_juan eliminó el producto X a las 14:00"*.

---

## Guía de Implementación (De Esencial a Avanzado)

### Fase 1: Lo Esencial (Configuración Base)
Protege tu API y define reglas de acceso.

**Dependencia (`pom.xml`):**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**Configuración (`SecurityConfig.java`):**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable()) // Desactivar CSRF para APIs REST
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**", "/auth/**").permitAll() // Público
                .requestMatchers("/admin/**").hasRole("ADMIN") // Solo Admin
                .anyRequest().authenticated() // El resto requiere login
            )
            .sessionManagement(sess -> sess.sessionCreationPolicy(SessionCreationPolicy.STATELESS)) // Sin sesiones (JWT)
            .build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // Estándar de encriptación
    }
}
```

### Fase 2: Lo Robusto (Conectar a Base de Datos)
Conectar Spring Security con tu tabla `Usuario`.

1.  **Tu Entidad debe implementar `UserDetails`:**
    ```java
    @Entity
    public class Usuario implements UserDetails {
        // ...
        @Override
        public Collection<? extends GrantedAuthority> getAuthorities() {
            return List.of(new SimpleGrantedAuthority("ROLE_" + this.rol));
        }
        // Implementar métodos booleanos retornando 'true'
    }
    ```

2.  **Crear `CustomUserDetailsService`:**
    Servicio que busca el usuario en TU repositorio.

### Fase 3: Lo Avanzado (JWT)
Autenticación moderna sin estado.

1.  **`JwtService`:** Genera y valida tokens.
2.  **`JwtAuthenticationFilter`:** Intercepta peticiones, lee el header `Authorization: Bearer ...` y autentica al usuario.

---

### ¿Dónde ubico estas clases?
Organización profesional recomendada:
```text
com.empresa.proyecto
├── config
│   └── SecurityConfig.java
├── security
│   ├── JwtService.java
│   ├── JwtAuthenticationFilter.java
│   └── CustomUserDetailsService.java
└── model
    └── Usuario.java (implements UserDetails)
```

# Mappers (MapStruct)

## ¿Qué es MapStruct?
Es un **Generador de Código** en tiempo de compilación.
A diferencia de otras librerías que usan reflexión (lento), MapStruct **escribe el código Java por ti** antes de que corra la app.

> *Ventaja:* Es rápido, seguro y si te equivocas en un nombre de campo, **falla al compilar** (te avisa antes de correr).

---

## Implementación Profesional

### 1. Dependencias (`pom.xml`)
Necesitas la librería y el procesador de anotaciones.

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>
<!-- En el plugin maven-compiler-plugin -->
<path>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.5.5.Final</version>
</path>
```

### 2. La Interfaz del Mapper (`UsuarioMapper.java`)
Tú defines el **QUÉ**, MapStruct hace el **CÓMO**.

```java
@Mapper(componentModel = "spring") // Permite inyectarlo con @Autowired
public interface UsuarioMapper {

    // Caso 1: Mapeo directo (mismos nombres)
    UsuarioResponse toResponse(Usuario usuario);

    // Caso 2: Mapeo inverso
    Usuario toEntity(UsuarioRequest request);

    // Caso 3: Mapeo con nombres distintos
    // source = origen (Entidad), target = destino (DTO)
    @Mapping(source = "fechaCreacion", target = "fechaAlta")
    @Mapping(source = "categoria.nombre", target = "nombreCategoria") // Navegación profunda
    UsuarioDetalleDTO toDetalle(Usuario usuario);
    
    // Caso 4: Listas (¡Lo hace automático!)
    List<UsuarioResponse> toResponseList(List<Usuario> usuarios);
}
```

### 3. Uso en el Servicio
Gracias a `componentModel = "spring"`, lo inyectas como cualquier otro componente.

```java
@Service
public class UsuarioService {
    
    private final UsuarioRepository repositorio;
    private final UsuarioMapper mapper; // ¡Inyección!

    public UsuarioService(UsuarioRepository repositorio, UsuarioMapper mapper) {
        this.repositorio = repositorio;
        this.mapper = mapper;
    }

    public UsuarioResponse crear(UsuarioRequest request) {
        // 1. DTO -> Entidad (Automático)
        Usuario usuario = mapper.toEntity(request);
        
        // 2. Guardar
        usuario = repositorio.save(usuario);
        
        // 3. Entidad -> DTO (Automático)
        return mapper.toResponse(usuario);
    }
}
```

### ¿Qué hace MapStruct "por detrás"? (La Magia Revelada)

Cuando compilas tu proyecto (`mvn clean install`), MapStruct lee tu interfaz `UsuarioMapper` y **genera un archivo real** llamado `UsuarioMapperImpl.java` en la carpeta `target/generated-sources`.

**Este es el código que MapStruct escribe por ti (y que te ahorras de mantener):**

```java
// CÓDIGO GENERADO AUTOMÁTICAMENTE (Tú no lo escribes)
@Component
public class UsuarioMapperImpl implements UsuarioMapper {

    @Override
    public UsuarioResponse toResponse(Usuario usuario) {
        if ( usuario == null ) {
            return null;
        }

        // Mira cómo hace los setters y getters uno por uno
        Long id = usuario.getId();
        String nombre = usuario.getNombre();
        String email = usuario.getEmail();
        
        // ... 20 campos más ...

        UsuarioResponse usuarioResponse = new UsuarioResponse( id, nombre, email );
        return usuarioResponse;
    }

    @Override
    public UsuarioDetalleDTO toDetalle(Usuario usuario) {
        if ( usuario == null ) {
            return null;
        }

        UsuarioDetalleDTO usuarioDetalleDTO = new UsuarioDetalleDTO();

        // Aquí resuelve la navegación compleja (categoria.nombre)
        usuarioDetalleDTO.setNombreCategoria( usuarioCategoriaNombre( usuario ) );
        
        // Aquí resuelve el cambio de nombre (fechaCreacion -> fechaAlta)
        usuarioDetalleDTO.setFechaAlta( usuario.getFechaCreacion() );

        return usuarioDetalleDTO;
    }
    
    // Métodos auxiliares privados para evitar NullPointerExceptions
    private String usuarioCategoriaNombre(Usuario usuario) {
        if ( usuario == null ) {
            return null;
        }
        Categoria categoria = usuario.getCategoria();
        if ( categoria == null ) {
            return null;
        }
        return categoria.getNombre();
    }
}
```

> **Conclusión:** Te ahorras escribir (y testear) toda esa lógica repetitiva de `if (null)`, `get`, `set`, y navegación de objetos anidados. MapStruct lo hace perfecto y súper rápido.

# 🧪 Testing Profesional en Spring Boot

El testing es lo que separa a los programadores que duermen tranquilos de los que viven con miedo a desplegar.
En Spring Boot, dividimos las pruebas en **3 niveles principales**.

---

## 1. Unit Testing (Pruebas Unitarias)
**Objetivo:** Probar **UNA sola clase** (tu Servicio) aislada del resto del mundo.
**Herramientas:** `JUnit 5` (Framework) + `Mockito` (Simulación).

> **¿Por qué Mockito?**
> Tu `UsuarioService` depende de `UsuarioRepository`. Si pruebas el servicio usando la base de datos real, es lento y frágil.
> Con Mockito, creas un "Repositorio de Mentira" que hace exactamente lo que tú le digas.

```java
@ExtendWith(MockitoExtension.class) // Habilita Mockito
class UsuarioServiceTest {

    @Mock // Crea un simulacro del repositorio (no es real)
    private UsuarioRepository repositorioMock;

    @InjectMocks // Inyecta el mock dentro del servicio real
    private UsuarioService servicio;

    @Test
    void deberiaCrearUsuarioExitosamente() {
        // 1. GIVEN (Dado que...)
        UsuarioRequest request = new UsuarioRequest("Juan", "juan@gmail.com", "123456");
        Usuario usuarioGuardado = new Usuario(1L, "Juan", "juan@gmail.com", "123456");

        // Entrenamos al Mock: "Cuando alguien llame a save(), devuelve esto"
        when(repositorioMock.save(any(Usuario.class))).thenReturn(usuarioGuardado);

        // 2. WHEN (Cuando...)
        UsuarioResponse respuesta = servicio.crear(request);

        // 3. THEN (Entonces...)
        assertNotNull(respuesta);
        assertEquals(1L, respuesta.id());
        assertEquals("Juan", respuesta.nombre());
        
        // Verificamos que el repositorio fue llamado exactamente 1 vez
        verify(repositorioMock, times(1)).save(any(Usuario.class));
    }

    @Test
    void deberiaLanzarErrorSiEmailExiste() {
        // 1. GIVEN: Simulamos que el email ya existe
        when(repositorioMock.existsByEmail("juan@gmail.com")).thenReturn(true);

        // 2. WHEN & THEN: Verificamos que lance la excepción
        assertThrows(EmailYaRegistradoException.class, () -> {
            servicio.crear(new UsuarioRequest("Juan", "juan@gmail.com", "123"));
        });

        // Verificamos que NUNCA se intentó guardar
        verify(repositorioMock, never()).save(any());
    }
}
```

---

## 2. Integration Testing (Pruebas de Integración)
**Objetivo:** Probar que tus componentes (Controller + Service + Repository + DB) funcionan **juntos**.
**Herramientas:** `@SpringBootTest`, `@AutoConfigureMockMvc`.

Aquí levantamos un "Spring chiquito" y usamos una base de datos en memoria (H2) o Testcontainers.

```java
@SpringBootTest // Levanta el contexto completo de Spring
@AutoConfigureMockMvc // Configura el cliente HTTP falso
class UsuarioControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc; // Cliente para hacer peticiones HTTP falsas

    @Autowired
    private ObjectMapper objectMapper; // Para convertir objetos a JSON

    @Test
    void deberiaCrearUsuarioEndToEnd() throws Exception {
        UsuarioRequest request = new UsuarioRequest("Ana", "ana@test.com", "123456");

        mockMvc.perform(post("/api/usuarios")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                
                // Validaciones de la respuesta HTTP
                .andExpect(status().isCreated()) // Esperamos 201 Created
                .andExpect(jsonPath("$.nombre").value("Ana")) // Validamos el JSON de respuesta
                .andExpect(jsonPath("$.id").exists());
    }
}
```

---

## 3. Repository Testing (`@DataJpaTest`)
**Objetivo:** Probar solo tus queries personalizadas (`@Query`).
**Herramientas:** `@DataJpaTest`.

Esto configura **solo** la capa de base de datos, ignora controllers y services. Es muy rápido.

```java
@DataJpaTest
class UsuarioRepositoryTest {

    @Autowired
    private UsuarioRepository repositorio;

    @Test
    void deberiaBuscarAdminsActivos() {
        // Guardamos datos de prueba en la DB en memoria (H2)
        repositorio.save(new Usuario("Admin", "admin@test.com", "ROLE_ADMIN", true));
        repositorio.save(new Usuario("User", "user@test.com", "ROLE_USER", true));

        // Ejecutamos la query personalizada
        List<Usuario> admins = repositorio.buscarAdminsActivos();

        // Verificamos
        assertEquals(1, admins.size());
        assertEquals("Admin", admins.get(0).getNombre());
    }
}
```

---

## 📊 Estrategia Profesional (Pirámide de Testing)

1.  **Unit Tests (80%):** Son ultra rápidos (milisegundos). Prueban toda la lógica del Service. Si falla la lógica de negocio, debe fallar aquí.
2.  **Integration Tests (10%):** Son más lentos (segundos). Prueban que el Controller hable bien con el Service y la DB, y que los JSONs sean correctos.
3.  **Repository Tests (10%):** Solo necesarios si escribes queries complejas en JPQL/SQL. Si usas solo métodos estándar (`save`, `findById`), no hacen falta.

---

## 🧠 Deep Dive: ¿Cómo funciona esto "por detrás"?

### 1. La Magia de Mockito (Proxies Dinámicos)
Cuando pones `@Mock UsuarioRepository repo;`, Mockito no instancia tu clase real.
Usa una librería llamada **ByteBuddy** para crear una clase nueva en tiempo de ejecución (un **Proxy**) que "hereda" de tu repositorio.

*   **El Proxy:** Es un cascarón vacío. Todos sus métodos devuelven `null` o `0` por defecto.
*   **`when(repo.save(...)).thenReturn(...)`**: Estás programando ese proxy. Le dices: *"Oye, si alguien llama al método `save` con estos argumentos, no ejecutes nada real, solo devuelve este objeto que te doy"*
*   **`verify(...)`**: El proxy tiene una libreta interna donde anota: *"A las 10:00:01 se llamó al método `save`"*. `verify` solo lee esa libreta.

```java
// Lo que tú escribes:
@Mock UsuarioRepository repo;

// Lo que Mockito genera en memoria (simplificado):
public class UsuarioRepository$MockitoProxy extends UsuarioRepository {
    @Override
    public Usuario save(Usuario u) {
        // 1. Registrar la llamada en la "libreta"
        this.historial.add("save llamado con " + u);
        
        // 2. Verificar si hay instrucciones (stubs)
        if (instrucciones.containsKey("save")) {
            return instrucciones.get("save"); // Devuelve lo que configuraste en el 'when'
        }
        
        // 3. Por defecto devuelve null (no va a la DB real)
        return null; 
    }
}
```

### 2. La Magia de `@SpringBootTest` (Contexto de Spring)
A diferencia de los tests unitarios (que son Java puro), `@SpringBootTest` **arranca la aplicación real**.
1.  Lee tu `application.properties`.
2.  Escanea todos tus `@Service`, `@Controller`, `@Repository`.
3.  Crea el **ApplicationContext** (el contenedor de beans).
4.  **`MockMvc`**: No levanta un servidor Tomcat real (no abre puertos). En su lugar, crea una instancia del **DispatcherServlet** (el cerebro de Spring Web) y le pasa peticiones HTTP simuladas directamente en memoria. Es rapidísimo porque no hay red involucrada.

```java
// MockMvc simula el ciclo de vida HTTP sin salir de la JVM
MockHttpServletRequest request = new MockHttpServletRequest("POST", "/api/usuarios");
request.setContent("{\"nombre\": \"Juan\"}");

// Pasa directo al DispatcherServlet (sin TCP/IP, sin puertos)
MockHttpServletResponse response = dispatcherServlet.service(request);

// Verificamos el resultado
assert response.getStatus() == 201;
```

### 3. La Magia de H2 (`@DataJpaTest`)
H2 es una base de datos SQL real, pero escrita en Java y que vive **en la memoria RAM**.
*   Al iniciar el test: Spring lee tus entidades (`@Entity`) y ejecuta `CREATE TABLE` en la memoria RAM.
*   Durante el test: Guardas y consultas datos reales (SQL).
*   Al terminar el test: Se borra la memoria RAM. Todo desaparece. ¡Por eso los tests son aislados!

```yaml
# application-test.properties (Spring lo usa automáticamente en tests)
spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
spring.datasource.driverClassName=org.h2.Driver
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
# Esto hace que Hibernate cree las tablas al inicio
spring.jpa.hibernate.ddl-auto=create-drop 
```

---

## 📝 Ejemplo Explicado: Unit Test de un Servicio

Aquí probamos `UsuarioService.crear()` sin tocar la base de datos.

```java
@ExtendWith(MockitoExtension.class) // 1. Habilita Mockito para esta clase
class UsuarioServiceTest {

    // 2. @Mock: Crea un "doble" del repositorio.
    // No es una instancia real, es un objeto controlado por nosotros.
    @Mock
    private UsuarioRepository repositorioMock;

    // 3. @InjectMocks: Crea una instancia REAL del servicio e inyecta el Mock adentro.
    // Es como hacer: new UsuarioService(repositorioMock);
    @InjectMocks
    private UsuarioService servicio;

    @Test
    @DisplayName("Debería crear un usuario correctamente cuando los datos son válidos")
    void crearUsuarioExitoso() {
        // --- GIVEN (Dado que...) ---
        // Preparamos los datos de entrada
        UsuarioRequest request = new UsuarioRequest("Juan", "juan@mail.com", "123456");
        
        // Preparamos lo que devolverá el repositorio (simulado)
        Usuario usuarioGuardado = new Usuario(1L, "Juan", "juan@mail.com", "123456");

        // Programamos el Mock: "Si te llaman a save() con CUALQUIER usuario, devuelve 'usuarioGuardado'"
        when(repositorioMock.save(any(Usuario.class))).thenReturn(usuarioGuardado);

        // --- WHEN (Cuando...) ---
        // Ejecutamos el método real que queremos probar
        UsuarioResponse respuesta = servicio.crear(request);

        // --- THEN (Entonces...) ---
        // Verificamos que el resultado sea el esperado
        assertNotNull(respuesta); // No debe ser nulo
        assertEquals(1L, respuesta.id()); // El ID debe ser 1
        assertEquals("Juan", respuesta.nombre()); // El nombre debe coincidir

        // Verificación extra: Aseguramos que el servicio llamó al repositorio 1 vez
        verify(repositorioMock, times(1)).save(any(Usuario.class));
    }
}
```

## 📝 Ejemplo Explicado: Integration Test (Controller + Service + DB)

Aquí probamos el flujo completo: Petición HTTP -> Controller -> Service -> Base de Datos (H2).

```java
@SpringBootTest // 1. Levanta TODO Spring (como si arrancaras la app real)
@AutoConfigureMockMvc // 2. Configura el cliente HTTP falso (MockMvc)
@Transactional // 3. Importante: Hace Rollback al final de cada test para limpiar la DB
class UsuarioControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc; // El cliente para hacer peticiones

    @Autowired
    private ObjectMapper objectMapper; // Para convertir Objetos <-> JSON

    @Test
    @DisplayName("POST /api/usuarios - Debería devolver 201 Created y el usuario creado")
    void crearUsuarioEndToEnd() throws Exception {
        // --- GIVEN ---
        UsuarioRequest request = new UsuarioRequest("Ana", "ana@test.com", "123456");

        // --- WHEN & THEN ---
        mockMvc.perform(post("/api/usuarios") // Hacemos un POST a la URL real
                .contentType(MediaType.APPLICATION_JSON) // Decimos que enviamos JSON
                .content(objectMapper.writeValueAsString(request))) // Convertimos el objeto a String JSON
                
                // --- VALIDACIONES (Expectations) ---
                .andExpect(status().isCreated()) // Esperamos HTTP 201
                .andExpect(header().exists("Location")) // Esperamos header Location
                .andExpect(jsonPath("$.id").isNumber()) // El JSON de respuesta debe tener ID numérico
                .andExpect(jsonPath("$.nombre").value("Ana")) // El nombre debe ser Ana
                .andExpect(jsonPath("$.email").value("ana@test.com")); // El email debe coincidir
    }
}

# Documentación Automática (Swagger / OpenAPI)

## ¿Qué es Swagger / OpenAPI?
Es un estándar para describir tu API REST. Permite que otros desarrolladores (o el equipo de Frontend) entiendan qué endpoints existen, qué datos necesitan y qué responden, sin tener que leer tu código Java.

**SpringDoc** es la librería oficial para integrar OpenAPI 3 en Spring Boot.

---

## 1. Instalación (Dependencia)

Agrega esto a tu `pom.xml`. ¡Es todo lo que necesitas para empezar!

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

## 2. Acceso
Una vez agregada la dependencia, corre tu aplicación y entra a:

👉 **http://localhost:8080/swagger-ui/index.html**

Verás una interfaz gráfica interactiva donde puedes probar tus endpoints directamente.

---

## 3. Configuración Básica (Opcional)
Si quieres cambiar el título y la descripción que aparecen en la página de Swagger:

```java
@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("API de Gestión de Usuarios")
                        .version("1.0")
                        .description("Documentación de la API para el curso de Spring Boot")
                        .termsOfService("http://swagger.io/terms/")
                        .license(new License().name("Apache 2.0").url("http://springdoc.org")));
    }
}
```

---

## 4. Mejorando la Documentación con Anotaciones
Por defecto, Swagger escanea tus controladores y genera documentación básica. Puedes mejorarla con anotaciones:

### En el Controller (`@Tag`, `@Operation`, `@ApiResponse`)

```java
@RestController
@RequestMapping("/api/usuarios")
@Tag(name = "Usuarios", description = "Operaciones para gestionar usuarios") // Agrupa endpoints
public class UsuarioController {

    @Operation(summary = "Obtener usuario por ID", description = "Devuelve un usuario si existe, o 404 si no.")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Usuario encontrado"),
        @ApiResponse(responseCode = "404", description = "Usuario no encontrado")
    })
    @GetMapping("/{id}")
    public ResponseEntity<UsuarioResponse> buscarPorId(@PathVariable Long id) {
        // ...
    }
}
```

### En el DTO (`@Schema`)
Para explicar qué es cada campo del JSON.

```java
public record UsuarioRequest(
    @Schema(description = "Nombre completo del usuario", example = "Juan Perez")
    @NotBlank
    String nombre,

    @Schema(description = "Correo electrónico único", example = "juan@gmail.com")
    @Email
    String email
) {}
```

# 📝 Checklist: Pasos para crear una API Professional desde Cero

*Esta es tu "hoja de ruta" con ejemplos prácticos para no perderte.*

## 0. Estructura de Carpetas Recomendada
```
Así debería verse tu proyecto para mantener el orden profesional:

```text
com.empresa.api
├── config
│   └── SwaggerConfig.java
├── controller
│   └── UsuarioController.java
├── dto
│   ├── UsuarioRequest.java
│   └── UsuarioResponse.java
├── exception
│   ├── GlobalExceptionHandler.java
│   └── ResourceNotFoundException.java
├── model
│   └── Usuario.java
├── repository
│   └── UsuarioRepository.java
└── service
    └── UsuarioService.java
```

## 1. Configuración Inicial
**Objetivo:** Conectar la aplicación a la base de datos.

- [ ] **Configurar DB:** Editar `src/main/resources/application.properties`.

```properties
# Conexión a Base de Datos (PostgreSQL)
spring.datasource.url=jdbc:postgresql://localhost:5432/mi_base_de_datos
spring.datasource.username=postgres
spring.datasource.password=tu_password

# Hibernate (JPA)
# update: crea tablas si no existen, pero no borra datos.
spring.jpa.hibernate.ddl-auto=update 
spring.jpa.show-sql=true
```

---

## 2. Capa de Datos (Persistence)
**Objetivo:** Definir las tablas y cómo acceder a ellas.

- [ ] **Crear Entidad (`@Entity`):** `src/main/java/com/empresa/api/model/Usuario.java`

```java
@Entity
@Table(name = "usuarios")
@Getter @Setter @NoArgsConstructor // Lombok ahorra código
public class Usuario {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;
    private String email;
}
```

- [ ] **Crear Repositorio (`@Repository`):** `src/main/java/com/empresa/api/repository/UsuarioRepository.java`

```java
@Repository
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
    // Spring implementa esto solo:
    boolean existsByEmail(String email);
}
```

---

## 3. Capa de Diseño (DTOs)
**Objetivo:** Definir qué datos entran y salen de la API (Seguridad y Limpieza).

- [ ] **Crear Request DTO:** `src/main/java/com/empresa/api/dto/UsuarioRequest.java`

```java
public record UsuarioRequest(
    @NotBlank(message = "El nombre es obligatorio")
    String nombre,

    @Email(message = "Email inválido")
    String email
) {}
```

- [ ] **Crear Response DTO:** `src/main/java/com/empresa/api/dto/UsuarioResponse.java`

```java
public record UsuarioResponse(
    Long id,
    String nombre,
    String email
) {}
```

---

## 4. Capa de Lógica (Service)
**Objetivo:** Aplicar reglas de negocio y coordinar todo.

- [ ] **Crear Servicio:** `src/main/java/com/empresa/api/service/UsuarioService.java`

```java
@Service
public class UsuarioService {

    private final UsuarioRepository repositorio;

    // Inyección por constructor (Mejor práctica)
    public UsuarioService(UsuarioRepository repositorio) {
        this.repositorio = repositorio;
    }

    public UsuarioResponse crear(UsuarioRequest request) {
        // 1. Validar reglas de negocio
        if (repositorio.existsByEmail(request.email())) {
            throw new RuntimeException("El email ya existe");
        }

        // 2. Convertir DTO a Entidad (Manual o con MapStruct)
        Usuario usuario = new Usuario();
        usuario.setNombre(request.nombre());
        usuario.setEmail(request.email());

        // 3. Guardar en DB
        usuario = repositorio.save(usuario);

        // 4. Retornar DTO
        return new UsuarioResponse(usuario.getId(), usuario.getNombre(), usuario.getEmail());
    }
}
```

---

## 5. Capa Web (Controller)
**Objetivo:** Exponer los endpoints HTTP al mundo.

- [ ] **Crear Controlador:** `src/main/java/com/empresa/api/controller/UsuarioController.java`

```java
@RestController
@RequestMapping("/api/v1/usuarios")
public class UsuarioController {

    private final UsuarioService service;

    public UsuarioController(UsuarioService service) {
        this.service = service;
    }

    @PostMapping
    public ResponseEntity<UsuarioResponse> crear(@Valid @RequestBody UsuarioRequest request) {
        UsuarioResponse nuevoUsuario = service.crear(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(nuevoUsuario);
    }
}
```

---

## 6. Manejo de Errores (Safety Net)
**Objetivo:** Que los errores se vean bonitos en JSON, no como trazas de código.

- [ ] **Crear GlobalExceptionHandler:** `src/main/java/com/empresa/api/exception/GlobalExceptionHandler.java`

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, String> errores = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errores.put(error.getField(), error.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errores);
    }
}
```

---

## 7. Documentación (Swagger)
**Objetivo:** Que el frontend sepa cómo usar tu API.

- [ ] **Agregar Dependencia:** `pom.xml`

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

- [ ] **Probar:** Corre la app y entra a `http://localhost:8080/swagger-ui/index.html`.

<br><br>

# Implementar Validaciones Personalizadas en Spring Boot

Esta guía explica paso a paso cómo crear tus propias anotaciones de validación (como `@ValidServiceKey`) para mantener tu código limpio y reutilizable.

## Concepto General
Para crear una validación personalizada necesitas **dos partes**:
1.  **La Anotación (`@interface`)**: Es la "etiqueta" que pones en tu código (ej. `@MiValidacion`). Define la configuración.
2.  **El Validador (`ConstraintValidator`)**: Es la clase Java que contiene la **lógica** (el `if` que decide si es válido o no).

---

## Paso 1: Crear la Anotación (`@interface`)

Crea un nuevo archivo, por ejemplo `StrongPassword.java`. Esta será tu etiqueta.

```java
package com.mateine.dopamine.validation;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;
import java.lang.annotation.*;

@Documented
// 1. CONEXIÓN: Aquí dices "La lógica de esta etiqueta está en la clase StrongPasswordValidator"
@Constraint(validatedBy = StrongPasswordValidator.class) 

// 2. DÓNDE: ¿Dónde puedo pegar esta etiqueta? (FIELD = variables, METHOD = métodos, etc.)
@Target({ElementType.FIELD, ElementType.PARAMETER}) 

// 3. CUÁNDO: ¿Hasta cuándo vive? (RUNTIME = Spring la puede ver mientras corre la app)
@Retention(RetentionPolicy.RUNTIME) 
public @interface StrongPassword {

    // 4. ATRIBUTOS OBLIGATORIOS (Por estándar de Java)
    
    // El mensaje de error por defecto
    String message() default "La contraseña no es lo suficientemente segura";

    // Para agrupar validaciones (ej. solo validar al crear, no al editar)
    Class<?>[] groups() default {};

    // Para mandar info extra (severidad, códigos de error)
    Class<? extends Payload>[] payload() default {};
}
```

---

## Paso 2: Crear la Lógica (`ConstraintValidator`)

Crea la clase que hace el trabajo sucio, ej. `StrongPasswordValidator.java`.

```java
package com.mateine.dopamine.validation;

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

// Implementa ConstraintValidator<TuAnotacion, TipoDeDatoAValidar>
public class StrongPasswordValidator implements ConstraintValidator<StrongPassword, String> {

    @Override
    public void initialize(StrongPassword constraintAnnotation) {
        // Opcional: Si tu anotación tuviera parámetros (ej. minLength), 
        // aquí los leerías para usarlos luego.
    }

    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        // 1. Manejar nulos: Generalmente, si es null se considera válido 
        // (para eso usas @NotNull aparte) o inválido según tu regla.
        if (password == null) {
            return false; 
        }

        // 2. Tu lógica de validación
        // Ej: Debe tener más de 8 caracteres y contener "123"
        boolean esLarga = password.length() > 8;
        boolean tieneNumeros = password.matches(".*\\d.*");

        return esLarga && tieneNumeros;
    }
}
```

---

## Paso 3: Usarlo en tu DTO

Ahora simplemente usas tu nueva anotación donde la necesites.

```java
public record UserRegistrationRequest(
    @NotBlank
    String username,

    @StrongPassword(message = "Tu contraseña es muy débil, ponle ganas") // Uso personalizado
    String password
) {}
```

## Resumen de Componentes Clave

| Componente | Código | Para qué sirve |
| :--- | :--- | :--- |
| **@Constraint** | `@Constraint(validatedBy = ...)` | Conecta la etiqueta con su cerebro (la clase validadora). |
| **@Target** | `@Target(ElementType.FIELD)` | Evita que pongas la validación en lugares donde no tiene sentido (ej. una clase entera). |
| **@Retention** | `@Retention(RetentionPolicy.RUNTIME)` | **CRÍTICO**. Sin esto, Java borra la etiqueta al compilar y Spring nunca valida nada. |
| **isValid()** | `boolean isValid(...)` | El único método que importa. Devuelve `true` si pasa, `false` si falla. |














