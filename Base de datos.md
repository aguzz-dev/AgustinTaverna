# Bases de Datos: Optimización y Performance

Este documento se centra en cómo investigar y resolver problemas de rendimiento en bases de datos, una pregunta clásica en entrevistas técnicas.

**Pregunta de Entrevista**: *"Si un endpoint empieza a funcionar lento por consultas a la base, ¿cómo lo investigarías y optimizarías?"*

---

## 1. Diagnóstico e Investigación

Antes de optimizar, hay que medir. No adivines qué está lento, encuéntralo.

### A. Identificar la Consulta Lenta
1.  **APM (Application Performance Monitoring)**: Herramientas como Datadog, New Relic o Dynatrace te muestran exactamente qué query SQL está tardando más tiempo dentro de una traza HTTP.
2.  **Logs de "Slow Query"**: Todas las bases de datos (PostgreSQL, MySQL) tienen una configuración para loguear consultas que tardan más de X milisegundos.
    *   *Ejemplo*: `log_min_duration_statement = 200ms` en Postgres.

### B. Analizar el Plan de Ejecución (`EXPLAIN`)
Una vez tienes la query SQL culpable, ejecútala con el comando `EXPLAIN ANALYZE` (en Postgres) o `EXPLAIN` (en MySQL).

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

Esto te dirá **cómo** la base de datos está buscando los datos:
*   🔴 **Full Table Scan (Seq Scan)**: La base de datos está leyendo **toda** la tabla, fila por fila, para encontrar el dato. Esto es O(N) y es terrible si la tabla tiene millones de registros.
*   🟢 **Index Scan**: La base de datos está usando un índice (como el índice de un libro) para saltar directamente al dato. Esto es O(log N) y es lo que buscamos.

---

## 2. Estrategias de Optimización

### A. Índices (Indexes)

Un índice es una estructura de datos (generalmente un B-Tree) que guarda una copia ordenada de ciertas columnas para acelerar las búsquedas.

*   **Cuándo crear un índice**:
    *   En columnas que usas frecuentemente en el `WHERE` (e.g., `email`, `dni`, `status`).
    *   En columnas que usas para unir tablas (`JOIN`).
    *   En columnas que usas para ordenar (`ORDER BY`).

*   **Trade-off (Costo)**: Los índices no son gratis.
    *   Aceleran las lecturas (`SELECT`).
    *   **Ralentizan las escrituras** (`INSERT`, `UPDATE`, `DELETE`), porque la base de datos tiene que actualizar la tabla Y el índice cada vez.
    *   Ocupan espacio en disco.

*   **Índices Compuestos**: Si buscas por `nombre` Y `apellido` a la vez, un índice solo en `nombre` ayuda, pero un índice compuesto `(nombre, apellido)` es mucho mejor.
    *   *Importante*: El orden importa. Un índice `(A, B)` sirve para buscar por `A` y por `A y B`, pero **NO** sirve para buscar solo por `B`.

### B. Refactorización de Consultas

A veces el problema no es la falta de índices, sino cómo pedimos los datos.

*   **Evitar `SELECT *`**: Trae solo las columnas que necesitas. Traer columnas de texto gigantes innecesariamente consume red y memoria.
*   **Problema N+1 (Hibernate/JPA)**: El asesino silencioso del rendimiento.
    *   *Escenario*: Tienes una lista de `Pedidos` y cada pedido tiene un `Usuario`.
    *   *El Error*: Haces 1 query para traer los pedidos, y luego, al recorrer la lista y llamar a `pedido.getUsuario().getNombre()`, Hibernate hace **una query extra por cada pedido** para traer al usuario. Si son 1000 pedidos, haces 1001 queries.
    *   *Solución*: Usar `JOIN FETCH` en JPQL para traer todo en una sola query SQL.
    ```java
    // MAL: N+1 queries
    List<Pedido> pedidos = repository.findAll(); 
    
    // BIEN: 1 sola query con JOIN
    @Query("SELECT p FROM Pedido p JOIN FETCH p.usuario")
    List<Pedido> findAllWithUsuarios();
    ```

### C. Revisión de Uso de ORM (Hibernate/JPA)

El ORM facilita la vida pero oculta la complejidad. Hay que usarlo con cuidado.

*   **Lazy vs Eager Loading**:
    *   **Peligro**: Nunca uses `FetchType.EAGER` en relaciones `@OneToMany` o `@ManyToMany`. Si traes un `Usuario`, te traerá todos sus `Pedidos`, y los `Detalles` de cada pedido... te puedes traer media base de datos sin querer.
    *   **Regla**: Usa siempre `FetchType.LAZY` por defecto. Usa `JOIN FETCH` (como vimos arriba) solo cuando realmente necesites los datos relacionados.
*   **Proyecciones (DTOs) vs Entidades**:
    *   Si solo necesitas mostrar una tabla con "Nombre" y "Email", **NO** traigas la Entidad `Usuario` completa (que pesa mucho y tiene relaciones).
    *   Usa **DTO Projections**:
    ```java
    // JPQL optimizado: Solo trae 2 columnas, no el objeto entero
    @Query("SELECT new com.app.dto.UserSummary(u.nombre, u.email) FROM Usuario u")
    List<UserSummary> findSummaries();
    ```

### D. Otras Técnicas

*   **Paginación**: Nunca devuelvas una lista ilimitada. Usa `LIMIT` y `OFFSET` (o paginación por cursor/keyset para mayor velocidad).

---

## 3. Estrategias de Escalabilidad Avanzada

Cuando la optimización de queries e índices no es suficiente, necesitamos estrategias arquitectónicas.

### A. Cachés (Caching)

Introducir una capa de almacenamiento rápido (RAM) entre la aplicación y la base de datos (e.g., **Redis**, Memcached).

*   **Patrón Cache-Aside (Lazy Loading)**:
    1.  La App pide el dato a la Caché.
    2.  ¿Está? ✅ Retorna el dato (rápido).
    3.  ¿No está? ❌ La App va a la DB, obtiene el dato, lo guarda en Caché y lo retorna.
*   **El Gran Desafío: Invalidación**: *"Solo hay dos cosas difíciles en computación: invalidación de caché y nombrar cosas"*.
    *   Si actualizas un dato en la DB, la caché queda obsoleta (stale).
    *   **Solución**: Usar **TTL (Time To Live)** para que los datos expiren automáticamente, o borrar explícitamente la entrada en caché al hacer un `UPDATE`.

### B. Particionado y Sharding

Cuando una tabla es demasiado grande para un solo servidor.

#### 1. Particionado (Partitioning)
Dividir una tabla lógica en varias tablas físicas **dentro de la misma instancia** de base de datos.
*   *Ejemplo*: Tabla `Logs` particionada por mes (`logs_2023_01`, `logs_2023_02`).
*   **Ventaja**: **Partition Pruning**. Si haces `SELECT * FROM logs WHERE date = '2023-01-15'`, la base de datos sabe que solo tiene que buscar en la partición de Enero y ignora el resto.

#### 2. Sharding (Fragmentación)
Distribuir los datos en **múltiples servidores físicos** diferentes.
*   **Sharding Horizontal**: Dividir las filas.
    *   Usuarios A-M ➡️ Servidor 1.
    *   Usuarios N-Z ➡️ Servidor 2.
*   **Sharding Vertical**: Dividir por funcionalidad.
    *   Tablas de `Usuarios` ➡️ Servidor 1.
    *   Tablas de `Pedidos` ➡️ Servidor 2.
*   **Complejidad**: Es muy complejo de mantener. Pierdes las transacciones ACID entre shards y los `JOIN` se vuelven casi imposibles (tienes que hacerlos en código). **Úsalo solo como último recurso**.
