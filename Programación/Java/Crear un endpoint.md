#### Cómo pensar al desarrollar un endpoint en Java

### 1. *Entender el Dominio y el Contrato*

- **¿Qué hace este endpoint?**   (su responsabilidad única)
- **¿Qué recibe?**                        (parámetros, body, headers)
- **¿Qué devuelve?**                   (estructura de respuesta, códigos HTTP)
- **¿Quién lo consume?**            (frontend, otro servicio, móvil)

**Ejemplo mental:** `Necesito un endpoint que permita hacer tal cosa.` 
`Va a recibir N y va a devolver Z, o un error en caso que falle`


### 2. *Diseñar de afuera hacia adentro -> Outside-In*

 *Tomando de ejemplo una **arquitectura en capas**, sería:*
#### Capa de Presentación -> Controller

- Define primero la **firma del endpoint**: método HTTP, path, request/response DTOs

- Esta capa **solo coordina**, **NO** tiene lógica de negocio

- Pensá: "¿Cómo quiero que se vea la API desde afuera?"


#### Capa de Aplicación/Servicio -> Service

- Aquí va la **orquestación** de la lógica de negocio

- Coordina entre múltiples dominios si es necesario

- Pensá: "¿Qué pasos necesito ejecutar? ¿Qué validaciones hago?"


#### Capa de Dominio -> Business Logic

- Las **reglas de negocio** viven acá

- Entidades, value objects, reglas de validación del dominio

- Pensá: "¿Qué reglas del negocio tengo que cumplir?"


#### Capa de Infraestructura -> Repository/DAO

- Acceso a **datos externos** (BD, APIs, cache)

- Pensá: "¿Qué datos necesito persistir o recuperar?"


### 3. Definir los Contratos entre Capas

 *Cada capa se comunica mediante **interfaces**:
 
`Controller → usa → ServiceInterface`
`Service    → usa → RepositoryInterface`

**Ventaja:** Puedes cambiar implementaciones sin afectar otras capas.

**Pensá:** "¿Qué métodos necesita exponer cada **capa** para que la capa superior funcione?"


### 4. Mapeo de Datos entre Capas

*Cada capa tiene su propio **modelo de datos**:*

- **Controller:** DTOs (Data Transfer Objects) - para request/response
- **Service:** DTOs de aplicación o directamente entidades
- **Dominio:** Entidades del negocio
- **Repository:** Entidades de BD (pueden ser las mismas del dominio)

**Pensá:** "¿Necesito transformar datos entre capas? ¿Uso mappers?"

### 5. Manejo de Errores por Capa

*Define qué errores puede lanzar cada capa:*

- **Dominio:** Excepciones de negocio (`InvalidDataException`, `BusinessRuleViolation`)

- **Servicio:** Excepciones de aplicación (`EntityNotFoundException`, `DuplicateException`)

- **Controller:** Traduce excepciones a códigos HTTP apropiados

**Piensa:** "¿Qué puede salir mal en cada paso? ¿Cómo comunico esos errores?"


### 6. Orden de Implementación Recomendado

1. **DTOs (request/response)**      → defines el contrato
2. **Entidades de Dominio**          → defines tu modelo
3. **Repository Interface**             → defines cómo accedes a datos
4. **Repository Implementation** → implementas acceso real
5. **Service Interface**                   → defines la lógica de negocio
6. **Service Implementation**       → implementas la orquestación
7. **Controller**                              → expones el endpoint
8. **Exception Handlers**               → manejas errores globalmente
9. **Tests** (idealmente TDD, pero al menos después)







