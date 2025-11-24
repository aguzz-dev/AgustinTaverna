# Concurrencia y Manejo de Race Conditions

Este documento aborda cómo manejar múltiples peticiones simultáneas que intentan modificar el mismo recurso, un problema crítico en sistemas distribuidos y de alta carga.

**Pregunta de Entrevista**: *"¿Cómo manejarías múltiples requests actualizando el mismo recurso, evitando condiciones de carrera?"*

---

## 1. El Problema: Race Conditions (Condiciones de Carrera)

Ocurre cuando dos o más hilos acceden a datos compartidos y tratan de cambiarlos al mismo tiempo.

*   **Ejemplo Clásico (Lost Update)**:
    1.  Hilo A lee `stock = 10`.
    2.  Hilo B lee `stock = 10`.
    3.  Hilo A vende 1 y guarda `stock = 9`.
    4.  Hilo B vende 1 y guarda `stock = 9`.
    *   **Resultado**: Se vendieron 2 productos, pero el stock bajó solo 1. ¡Desastre!

---

## 2. Solución 1: Transacciones (ACID)

La base de defensa es usar transacciones de base de datos.

*   **Atomicidad**: O todo se guarda, o nada se guarda.
*   **Aislamiento**: Define cómo una transacción ve los cambios de otras.
*   **En Spring**: Usar `@Transactional` asegura que el método se ejecute dentro de una transacción. Si hay una excepción, se hace `rollback`.

```java
@Transactional
public void procesarPedido(Long pedidoId) {
    // Todo lo que pase aquí es atómico
}
```

Sin embargo, `@Transactional` por sí solo (con aislamiento por defecto `READ_COMMITTED`) **NO** previene el problema del "Lost Update" descrito arriba. Para eso necesitamos **Locks**.

---

## 3. Solución 2: Locking (Bloqueos)

Existen dos estrategias principales para evitar conflictos de concurrencia.

### A. Optimistic Locking (Bloqueo Optimista)

*   **Filosofía**: "Soy optimista. Creo que **casi nunca** vamos a chocar. Así que no bloqueo nada al leer. Solo verifico si hubo cambios justo antes de guardar".
*   **Cómo funciona**: Se añade una columna de `version` a la tabla.
    1.  Leo el registro (tiene `version = 1`).
    2.  Hago mis cálculos.
    3.  Al guardar, le digo a la DB: *"Actualiza SOLO si `version` sigue siendo 1, y sube la versión a 2"*.
    4.  Si alguien más lo cambió (y ahora `version` es 2), mi actualización falla (0 filas actualizadas) y lanzo una excepción (`OptimisticLockException`).
*   **Cuándo usarlo**: Sistemas con **muchas lecturas y pocas escrituras** concurrentes (e.g., editar perfil de usuario, posts de blog). Es muy eficiente porque no bloquea la base de datos.

**Ejemplo en JPA/Hibernate**:
```java
@Entity
public class Producto {
    @Id
    private Long id;
    
    @Version // ¡Magia de JPA!
    private Long version; 
    
    private int stock;
}
```

### B. Pessimistic Locking (Bloqueo Pesimista)

*   **Filosofía**: "Soy pesimista. Seguro que alguien más va a intentar tocar esto. Así que **bloqueo** el registro apenas lo leo para que nadie más pueda tocarlo hasta que yo termine".
*   **Cómo funciona**: Usa `SELECT ... FOR UPDATE` a nivel de base de datos.
    1.  La transacción A lee el registro y adquiere un "Lock de Escritura".
    2.  Si la transacción B intenta leer ese mismo registro, **se queda esperando** (bloqueada) hasta que A termine (commit o rollback).
*   **Cuándo usarlo**: Sistemas críticos con **alta concurrencia de escritura** donde el conflicto es muy probable y costoso (e.g., sistema de venta de entradas de conciertos, control de saldo bancario).

**Ejemplo en JPA/Spring Data**:
```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT p FROM Producto p WHERE p.id = :id")
Optional<Producto> findByIdWithLock(Long id);
```

---

## 4. Soluciones Avanzadas (Sin Bloqueos)

### A. Compare-and-Swap (CAS)

*   **Concepto**: Es una operación atómica soportada por el hardware (CPU) que dice: *"Actualiza esta variable al valor B SOLO si su valor actual es A"*.
*   **Ventaja**: Es **Lock-Free**. No usa bloqueos pesados del Sistema Operativo ni de la Base de Datos. Es extremadamente rápido (nanosegundos).
*   **En Java**: Se usa a través de clases como `AtomicInteger`, `AtomicReference` (paquete `java.util.concurrent.atomic`).
    ```java
    AtomicInteger contador = new AtomicInteger(0);
    // Incremento atómico y thread-safe sin usar 'synchronized'
    contador.incrementAndGet(); 
    ```

### B. Serialización por Colas (Queues)

*   **Filosofía**: *"Si solo un hilo toca el recurso a la vez, no necesitas locks"*.
*   **Cómo funciona**:
    1.  En lugar de que 50 hilos intenten actualizar el stock a la vez, todos envían un mensaje ("Vender 1 Producto X") a una **Cola** (Kafka, RabbitMQ o interna).
    2.  Un **único consumidor (Single Thread)** lee los mensajes de la cola uno por uno y actualiza el estado.
*   **Ventaja**: Elimina totalmente las Race Conditions y los Deadlocks por diseño.
*   **Uso**: Sistemas de ultra-baja latencia (e.g., LMAX Disruptor) o Motores de Juegos.

---

## 5. Idempotencia

Aunque no es un "lock", es un concepto vital para manejar la concurrencia y los fallos de red.

*   **Definición**: Una operación es idempotente si **ejecutarla múltiples veces tiene el mismo efecto que ejecutarla una sola vez**.
    *   *Ejemplo*: Pulsar el botón de llamada del ascensor 10 veces es igual a pulsarlo 1 vez.
    *   *En API REST*: `PUT` y `DELETE` deberían ser idempotentes. `POST` generalmente no lo es.
*   **El Problema**: Si envías una petición de "Cobrar $10" y se corta la red justo antes de recibir la respuesta, no sabes si se cobró o no. Si reintentas (Retry), podrías cobrar $20.
*   **La Solución: Idempotency Keys**:
    1.  El cliente genera un ID único (UUID) para la operación y lo envía en un header: `X-Idempotency-Key: 123-abc`.
    2.  El servidor recibe la petición.
    3.  Busca en una tabla/Redis: "¿Ya procesé la key `123-abc`?".
        *   **SÍ**: No hago nada. Devuelvo la misma respuesta que la primera vez (o un 200 OK).
        *   **NO**: Proceso el cobro y guardo la key `123-abc` como "procesada".

---

## Resumen: ¿Cuál elegir?

| Estrategia | Mecanismo | Costo de Performance | Caso de Uso |
| :--- | :--- | :--- | :--- |
| **Optimistic** | Columna `@Version` | Bajo (sin bloqueos de DB) | Perfiles, CMS, baja contención. |
| **Pessimistic** | `SELECT FOR UPDATE` | Alto (serializa el acceso) | Dinero, Stock crítico, alta contención. |
