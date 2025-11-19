Un **record** en Java es una clase especial para almacenar datos de forma simple e inmutable.

En lugar de escribir mucho código repetitivo **(constructores, getters, equals, hashCode, toString)**, solo declaras los campos y Java genera todo automáticamente.

**Ejemplo:**

```java
public record Persona(String nombre, int edad) {}
```

Esto te da automáticamente:

- Constructor
- Métodos `nombre()` y `edad()` para acceder a los datos
- Métodos `equals()`, `hashCode()` y `toString()`

Es perfecto para representar datos inmutables como DTOs, coordenadas, o cualquier objeto que solo necesite guardar información sin lógica compleja.

---
## Ampliamos

Un **record** es un tipo especial de clase introducido en **Java 14 (preview)** y estabilizado en **Java 16**, diseñado para representar **datos inmutables** de forma concisa.


```java
 public record Persona(String nombre, int edad) {}
```

Con esa sola línea, Java te genera automáticamente:

- Campos `private final nombre` y `edad`
    
- Un **constructor** que los inicializa
    
- Métodos **`getters`** implícitos, aunque sin el prefijo `get`, se acceden como `persona.nombre()`
    
- Implementaciones de **`equals()`**, **`hashCode()`** y **`toString()`**
    

👉 En otras palabras, un _record_ es una **clase de datos inmutable** con sintaxis reducida y comportamiento estándar.

## 💡 Qué problema vienen a resolver

Antes de los _records_, si querías una clase simple para transportar datos (un _DTO_, _Value Object_ o _POJO_), tenías que escribir mucho código repetitivo:

```java
public class Persona {
    private final String nombre;
    private final int edad;

    public Persona(String nombre, int edad) {
        this.nombre = nombre;
        this.edad = edad;
    }

    public String nombre() { return nombre; }
    public int edad() { return edad; }

    @Override
    public boolean equals(Object o) { ... }
    @Override
    public int hashCode() { ... }
    @Override
    public String toString() { ... }
}

```

Con _records_, todo eso se reduce a una sola línea, sin perder seguridad ni semántica.

Entonces, **los records vienen a reemplazar (o simplificar) los POJOs inmutables y los DTOs**.

---

## ⚙️ Cómo funcionan “por detrás”

Internamente, un _record_ **sigue siendo una clase**, pero con restricciones y generación automática de miembros.

El compilador de Java genera código similar a esto:

```java
public final class Persona extends java.lang.Record {
    private final String nombre;
    private final int edad;

    public Persona(String nombre, int edad) {
        this.nombre = nombre;
        this.edad = edad;
    }

    public String nombre() { return nombre; }
    public int edad() { return edad; }

    @Override
    public boolean equals(Object o) { /**/ }

    @Override
    public int hashCode() { /**/ }

    @Override
    public String toString() { 
        return "Persona[nombre=" + nombre + ", edad=" + edad + "]";
    }
}

```

Puntos clave:

- Heredan de una clase base especial: `java.lang.Record`
    
- Todos los campos son **`final`**
    
- La clase es **`final`** (no puede heredarse)
    
- La igualdad (`equals`/`hashCode`) se basa en **el valor de los campos**, no en la identidad del objeto

---

## 🧠 Conceptualmente hablando

En términos de diseño, un _record_ representa **datos**, no **comportamiento**.

Esto sigue la filosofía del **Value Object** en DDD:

> Dos objetos con los mismos valores se consideran iguales.

Por eso:

```java
var p1 = new Persona("Agus", 30);
var p2 = new Persona("Agus", 30);

System.out.println(p1.equals(p2)); // true ✅

```

Y como los campos son finales, no podés modificarlos:

```java
p1.nombre = "Otro"; // ❌ Error de compilación
```

---

## 🧩 Comparación con PHP

En PHP, podrías lograr algo parecido con:

```php
readonly class Persona {
    public function __construct(
        public string $nombre,
        public int $edad
    ) {}
}
```

Eso sería equivalente a un record:

- Clase inmutable (`readonly`)
    
- Constructor implícito con promoción de propiedades
    
- Sintaxis compacta
    

Pero Java te da además:

- `equals`, `hashCode`, `toString` generados automáticamente
    
- Tipado fuerte e integración con el sistema de _pattern matching_ (`instanceof`, _switch_, etc.)
    

---

## ⚙️ Personalización

Podés añadir lógica si lo necesitás, pero sin romper las reglas básicas:

```java
public record Persona(String nombre, int edad) {
    public Persona {
        if (edad < 0) throw new IllegalArgumentException("Edad negativa");
    }

    public String saludo() {
        return "Hola, soy " + nombre;
    }
}
```

Ese bloque `{ ... }` es un **compact constructor**, que se ejecuta automáticamente tras la asignación de campos.

---

## 🧾 En resumen

| Concepto        | Descripción                                       |
| --------------- | ------------------------------------------------- |
| **Qué es**      | Clase inmutable para contener datos               |
| **Desde**       | Java 16                                           |
| **Hereda de**   | `java.lang.Record`                                |
| **Campos**      | `final` por defecto                               |
| **Igualdad**    | Por valor (contenido), no por referencia          |
| **Uso típico**  | DTOs, Value Objects, estructuras de datos simples |
| **Reemplaza a** | POJOs / JavaBeans usados solo para datos          |

### Ejemplo completo

```java
import java.time.LocalDate;
import java.util.Objects;

public record Pedido(
    String id,
    Cliente cliente,
    double total,
    LocalDate fechaCreacion
) {
    // Constructor compacto con validaciones
    public Pedido {
        Objects.requireNonNull(id, "El ID no puede ser nulo");
        Objects.requireNonNull(cliente, "El cliente no puede ser nulo");
        Objects.requireNonNull(fechaCreacion, "La fecha no puede ser nula");

        if (total < 0) {
            throw new IllegalArgumentException("El total no puede ser negativo");
        }
    }

    // Método adicional (comportamiento derivado)
    public boolean esReciente() {
        return fechaCreacion.isAfter(LocalDate.now().minusDays(7));
    }

    // Método de negocio adicional
    public String resumen() {
        return "Pedido " + id + " de " + cliente.nombre()
                + " por $" + total + " (" + fechaCreacion + ")";
    }

    // Método estático de fábrica (patrón Factory Method)
    public static Pedido crearNuevo(Cliente cliente, double total) {
        return new Pedido(
            java.util.UUID.randomUUID().toString(),
            cliente,
            total,
            LocalDate.now()
        );
    }
}
```

#### Y el record `Cliente` podría ser así:

```java
public record Cliente(String nombre, String email) {
    public Cliente {
        if (nombre == null || nombre.isBlank())
            throw new IllegalArgumentException("El nombre es obligatorio");
        if (email == null || !email.contains("@"))
            throw new IllegalArgumentException("Email inválido");
    }

    public String dominioEmail() {
        return email.substring(email.indexOf("@") + 1);
    }
}
```

### 💬 Cómo se usaría

```java
public class Demo {
    public static void main(String[] args) {
        Cliente cliente = new Cliente("Agus", "agus@ejemplo.com");
        Pedido pedido = Pedido.crearNuevo(cliente, 199.99);

        System.out.println(pedido.resumen());
        System.out.println("¿Es reciente? " + pedido.esReciente());
        System.out.println("Dominio del email: " + cliente.dominioEmail());
    }
}
```

### Salida esperada:

```
Pedido 7b21b98a-4f3d-48e3-a16e-3f0a2b72bfc3 de Agus por $199.99 (2025-11-10)

¿Es reciente? true

Dominio del email: ejemplo.com
```

