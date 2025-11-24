# Arquitectura de Microservicios: Conceptos Clave

En una arquitectura de microservicios, el sistema se divide en pequeños servicios autónomos que colaboran entre sí. Para que esta arquitectura sea exitosa y no se convierta en un "monolito distribuido" (lo cual es peor que un monolito normal), dos pilares fundamentales son el **Desacoplamiento** y la **Resiliencia**.

---

## 1. Desacoplamiento (Decoupling)

El desacoplamiento es el grado de independencia que tienen los servicios entre sí. El objetivo es reducir las dependencias al mínimo para que los cambios en un servicio no rompan ni obliguen a cambiar a otros.

### ¿Por qué es importante?
Si dos servicios están muy acoplados (Tight Coupling), cada vez que necesitas desplegar una nueva versión del Servicio A, tienes que coordinar con el equipo del Servicio B. Esto mata la agilidad, que es la promesa principal de los microservicios.

### Tipos de Desacoplamiento

#### A. Desacoplamiento Temporal (Async)
Es la capacidad de que un servicio envíe una solicitud sin tener que esperar a que el otro responda inmediatamente.
*   **Acoplado (Síncrono)**: Servicio A llama a Servicio B vía HTTP REST y espera bloqueado. Si B cae, A cae o se queda colgado.
*   **Desacoplado (Asíncrono)**: Servicio A envía un mensaje a un Broker (RabbitMQ, Kafka). Servicio B lo procesará cuando pueda. Si B está caído, A no se entera y sigue funcionando; B procesará los mensajes acumulados cuando vuelva a levantarse.

#### B. Desacoplamiento Tecnológico
Como los servicios se comunican por interfaces estándar (API REST JSON, gRPC, Mensajes), no importa cómo están hechos por dentro.
*   El Servicio de Pagos puede estar en **Java 21**.
*   El Servicio de Recomendaciones puede estar en **Python** (para usar IA).
*   El Servicio de Frontend puede ser **Node.js**.

#### C. Desacoplamiento de Despliegue
Es la prueba de fuego. Deberías poder desplegar una nueva versión del Servicio A en producción un viernes por la tarde sin tener que avisar a nadie ni detener el resto del sistema.

---

## 2. Resiliencia (Resilience)

En un monolito, si la aplicación falla, todo cae. En microservicios, **los fallos son inevitables y parciales**. La red fallará, la base de datos tendrá latencia, un servicio se reiniciará.

La **Resiliencia** no es evitar que las cosas fallen (eso es imposible), sino diseñar el sistema para que **soporte los fallos y se recupere**, degradando la funcionalidad en lugar de colapsar completamente.

### Patrones de Resiliencia

#### A. Circuit Breaker (Cortocircuito)
Protege al sistema de fallos repetitivos. Es como un fusible eléctrico.
1.  **Cerrado (Normal)**: Las peticiones fluyen.
2.  **Abierto (Fallo)**: Si el Servicio B falla 10 veces seguidas, el Circuit Breaker se "abre" y deja de enviarle peticiones inmediatamente (fail fast) para no saturarlo más y no bloquear al Servicio A.
3.  **Semi-Abierto**: Después de un tiempo, deja pasar una petición de prueba. Si funciona, se cierra y vuelve a la normalidad.

#### B. Retry (Reintento)
Para fallos transitorios (un parpadeo en la red). Si una petición falla, se reintenta automáticamente un par de veces con una espera exponencial (backoff).
*   *Cuidado*: No hacer retries infinitos, o podrías crear un ataque DDoS a tu propio sistema.

#### C. Fallback (Plan B)
¿Qué hacemos si el servicio falla definitivamente?
*   Si el "Servicio de Recomendaciones" no responde, en lugar de mostrar un error 500, mostramos una lista de "Productos más vendidos" (que está en caché o es estática). El usuario ve la página, aunque no sea personalizada. Eso es **degradación elegante**.

#### D. Bulkhead (Mamparo)
Inspirado en los barcos. Si un casco se rompe, se cierran las compuertas para que solo se inunde esa sección y el barco no se hunda.
*   En software: Se asignan recursos limitados (hilos, conexiones) a cada dependencia. Si el Servicio de Facturación se vuelve lento y consume todos sus hilos, no afecta al Servicio de Catálogo, que tiene su propio pool de hilos reservado.

---

## 3. Colas y Mensajería (Queues / Messages)

El uso de colas es el mecanismo principal para lograr el **desacoplamiento temporal**.

### Concepto
Un servicio (Productor) pone un mensaje en una cola y se olvida. Otro servicio (Consumidor) lee de la cola y procesa el mensaje. No se conocen entre sí.

### Beneficios Clave
1.  **Load Leveling (Nivelación de Carga)**: Si de repente llegan 1000 pedidos por segundo, pero el servicio de facturación solo puede procesar 100, la cola actúa como un buffer (amortiguador). Los pedidos se encolan y se procesan al ritmo que el consumidor soporte, sin tumbar el sistema.
2.  **Tolerancia a Fallos**: Si el consumidor muere, los mensajes se quedan guardados en la cola (persistencia) y se procesan cuando el consumidor reviva. No se pierden datos.

### Herramientas Comunes
*   **RabbitMQ**: Colas tradicionales, muy flexible, garantiza entrega.
*   **Apache Kafka**: Streaming de eventos, altísimo rendimiento, retención de mensajes a largo plazo.

---

## 4. Timeouts (Tiempos de Espera)

Un **Timeout** es el tiempo máximo que un servicio está dispuesto a esperar una respuesta antes de cancelar la operación y dar error.

### ¿Por qué son vitales?
Sin timeouts, si el Servicio B se cuelga y nunca responde, el hilo del Servicio A se queda esperando "para siempre" (o hasta el timeout por defecto de TCP que puede ser minutos). Si llegan muchas peticiones, todos los hilos de A se quedarán bloqueados esperando a B, y A dejará de responder (efecto cascada).

> [!IMPORTANT]
> **Regla de Oro**: Nunca hagas una llamada de red sin un timeout explícito y corto.

---

## 5. Escenario Práctico: Sistema de Pedidos y Pagos

**Pregunta**: *"Si tuvieras que diseñar un servicio que procese pagos o pedidos, ¿cómo lo separarías en componentes? ¿Qué consideraciones de escalabilidad y fallos tendrías?"*

### A. Separación de Componentes (Microservicios)

En lugar de una sola aplicación gigante, dividimos por dominio de negocio (Bounded Contexts):

1.  **Order Service (Pedidos)**:
    *   Recibe la petición de compra del usuario.
    *   Valida datos básicos.
    *   Crea la orden en estado `PENDING_PAYMENT`.
    *   Publica evento: `OrderCreatedEvent`.
2.  **Payment Service (Pagos)**:
    *   Escucha `OrderCreatedEvent`.
    *   Contacta con la pasarela externa (Stripe/PayPal).
    *   Si éxito: Publica `PaymentSuccessEvent`.
    *   Si fallo: Publica `PaymentFailedEvent`.
3.  **Inventory Service (Inventario)**:
    *   Escucha `PaymentSuccessEvent` para descontar stock.
    *   (Opcional: Podría reservar stock antes del pago, depende del negocio).
4.  **Notification Service**:
    *   Escucha eventos para enviar emails al usuario ("Pedido Recibido", "Pago Aceptado").

### B. Consideraciones de Escalabilidad

*   **Stateless (Sin Estado)**: Los servicios no deben guardar sesión de usuario en memoria. Así podemos levantar 10 instancias del `Order Service` y poner un Load Balancer delante. Cualquiera puede atender cualquier petición.
*   **Database per Service**: Cada microservicio tiene SU propia base de datos. `Order Service` no puede hacer un `JOIN` con la tabla de `Payments`. Se comunican por API o Eventos. Esto permite escalar la BD de pedidos independientemente de la de pagos.

### C. Consideraciones de Fallos (Resiliencia)

*   **Retry Policies (Políticas de Reintento)**: Cuando el `Payment Service` contacta a la pasarela externa (Stripe/PayPal):
    *   Si recibe un error **transitorio** (e.g., Timeout, 503 Service Unavailable), se aplica un **retry** con *exponential backoff* (esperar 1s, luego 2s, luego 4s).
    *   Si recibe un error **definitivo** (e.g., 400 Bad Request, Tarjeta Inválida), **NO** se reintenta.
*   **Circuit Breaker (Cortocircuito)**: Si la pasarela de pagos se cae completamente (e.g., 100% de errores en 30 segundos):
    *   El Circuit Breaker se **abre** para dejar de enviar peticiones inútiles y no saturar nuestros hilos.
    *   **Fallback**: En lugar de rechazar el pedido, lo guardamos en estado `PAYMENT_RETRY_LATER` y notificamos al usuario que el procesamiento tardará unos minutos, reintentando cuando el circuito se cierre.
*   **Idempotencia en Pagos**: ¿Qué pasa si el mensaje de "Procesar Pago" llega dos veces (por un retry de red)? El `Payment Service` debe ser **idempotente**: si recibe el mismo ID de pedido dos veces, la segunda vez no debe cobrar de nuevo, sino devolver el resultado de la primera vez.
*   **Dead Letter Queues (DLQ)**: Si un mensaje de pedido está mal formado y hace crashear al consumidor, no debemos reintentarlo infinitamente. Tras X intentos fallidos, se mueve a una "Cola de Muertos" (DLQ) para que un humano o un proceso automático lo revise, sin bloquear el resto de pedidos.
*   **Patrón Saga**: Como no tenemos una transacción única en base de datos (ACID) que cubra Pedido + Pago + Inventario, usamos Sagas. Si el pago falla después de haber creado el pedido, se ejecuta una transacción compensatoria (e.g., cancelar el pedido o marcarlo como "Fallido") para mantener la consistencia eventual.
