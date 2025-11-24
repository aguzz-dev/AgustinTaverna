# Estrategias de Testing Avanzado

Más allá de los tests unitarios básicos, existen conceptos clave para asegurar la calidad en sistemas complejos y microservicios.

---

## 1. Dobles de Prueba (Test Doubles)

A menudo confundimos todo con "Mocks", pero hay diferencias importantes.

### A. Mocks (Simulacros)
*   **Objetivo**: Verificar **comportamiento**.
*   **Qué hacen**: Esperan que se les llame de cierta forma. Si no se les llama, el test falla.
*   **Herramienta**: Mockito.
*   *Ejemplo*: "Verificar que al guardar un usuario, se llamó al método `sendEmail` del `EmailService` exactamente 1 vez".

```java
// Mockito
verify(emailService, times(1)).sendEmail(anyString());
```

### B. Fakes (Falsificaciones)
*   **Objetivo**: Implementación ligera pero funcional.
*   **Qué hacen**: Tienen lógica real simplificada. No usan bases de datos ni redes.
*   **Uso**: Reemplazar una base de datos lenta por un `HashMap` en memoria para tests ultra-rápidos.
*   *Ejemplo*:
```java
public class InMemoryUserRepository implements UserRepository {
    private Map<Long, User> db = new HashMap<>();
    
    public void save(User user) { db.put(user.getId(), user); }
    public User findById(Long id) { return db.get(id); }
}
```

### C. Stubs (Talones)
*   **Objetivo**: Proveer respuestas predefinidas (hardcoded).
*   **Qué hacen**: "Si te llaman con X, devuelve Y". No verifican nada, solo ayudan a que el test pase.

---

## 2. Testear Escenarios de Error (Unhappy Paths)

Los tests que solo prueban el "camino feliz" (todo funciona bien) no sirven de mucho en producción. Lo valioso es testear **qué pasa cuando las cosas fallan**.

### ¿Qué testear?
1.  **Timeouts**: ¿Qué hace mi servicio si la API externa tarda 10 segundos en responder? ¿Lanza una excepción controlada o se cuelga?
2.  **Errores 500**: ¿Si el servicio de Pagos devuelve 500, mi servicio de Pedidos hace rollback de la transacción?
3.  **Datos Corruptos**: ¿Qué pasa si recibo un JSON mal formado?

### Cómo simularlo
Usando Mockito para forzar excepciones:

```java
// Simular que la DB lanza una excepción de conexión
when(repository.save(any())).thenThrow(new DataAccessException("DB Down"));

// Verificar que mi servicio maneja el error y no explota
assertThrows(ServiceException.class, () -> myService.crearPedido(pedido));
```

---

## 3. Contract Testing (Pruebas de Contrato)

En microservicios, el mayor miedo es: *"Si cambio la API del Servicio A, ¿romperé al Servicio B?"*.

### El Problema de los Mocks
Si en los tests del Servicio B "mockeas" al Servicio A, estás asumiendo que A responde de cierta forma. Pero si A cambia su respuesta en la realidad, **tus tests seguirán pasando (porque usas un mock)**, pero en producción **todo explotará**.

### La Solución: Contract Testing (e.g., Pact)
Es una técnica para garantizar que dos servicios se entienden sin necesidad de levantar ambos a la vez (que sería un test E2E lento).

1.  **Consumidor (Consumer)**: Define un **Contrato** (Pact). *"Espero que si hago GET /users/1, me devuelvas un JSON con `id` y `name`"*.
2.  **Productor (Provider)**: En su pipeline de CI/CD, se descarga ese contrato y ejecuta tests automáticos para verificar que **realmente** cumple lo que el consumidor espera.
3.  **Resultado**: Si el Productor cambia el nombre del campo `name` a `fullName`, su build fallará avisándole que rompería al Consumidor.

> [!TIP]
> Contract Testing reemplaza la necesidad de muchos tests E2E (End-to-End) que son lentos, costosos y frágiles.
