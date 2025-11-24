# Programación Orientada a Objetos (POO)

La **Programación Orientada a Objetos (POO)** es un paradigma de programación que utiliza "objetos" para diseñar aplicaciones y programas informáticos. Se basa en varias técnicas, incluyendo herencia, cohesión, abstracción, polimorfismo, acoplamiento y encapsulamiento.

## ÍNDICE
1. [CLASE](#1-clase)
2. [OBJETO](#2-objeto)
3. [ATRIBUTOS](#3-atributos)
4. [MÉTODOS](#4-métodos)
5. [CONSTRUCTORES](#5-constructores)
6. [ENCAPSULAMIENTO](#6-encapsulamiento)
7. [ABSTRACCIÓN](#7-abstracción)
8. [HERENCIA](#8-herencia)
9. [POLIMORFISMO](#9-polimorfismo)
10. [CONCEPTOS AVANZADOS](#10-conceptos-avanzados)
11. [RELACIONES ENTRE CLASES](#11-relaciones-entre-clases)
12. [INTERFACES VS CLASES ABSTRACTAS](#12-interfaces-vs-clases-abstractas)
13. [PAQUETES (PACKAGES)](#13-paquetes-packages)
14. [MODIFICADORES DE ACCESO](#14-modificadores-de-acceso)
15. [MODIFICADOR STATIC](#15-modificador-static)
16. [MODIFICADOR FINAL](#16-modificador-final)
17. [COHESIÓN Y ACOPLAMIENTO](#17-cohesión-y-acoplamiento)
18. [CLASES INTERNAS (INNER CLASSES)](#18-clases-internas-inner-classes)
19. [ENUMERACIONES (ENUMS)](#19-enumeraciones-enums)
20. [MANEJO DE EXCEPCIONES (POO)](#20-manejo-de-excepciones-poo)

<br><br>

## 1. CLASE

Una **Clase** es una **plantilla** o un **molde** a partir del cual se crean los objetos. Define la estructura y el comportamiento (atributos y métodos) que tendrán los objetos creados a partir de ella. No ocupa memoria por sí misma (en términos de datos de instancia) hasta que se crea un objeto.

### Características Principales
*   **Abstracción:** La clase abstrae las características comunes de un conjunto de objetos.
*   **Definición de Tipos:** Al crear una clase, estás creando un nuevo tipo de dato en tu programa.
*   **No ocupa memoria de instancia:** La definición de la clase reside en el área de código/metadatos, pero no reserva memoria para datos hasta que se crea un objeto (instancia).

> **Analogía:** Piensa en una clase como el **plano arquitectónico de una casa**. El plano especifica dónde van las paredes, las ventanas y qué materiales se usarán. Sin embargo, no puedes vivir en el plano; es solo la definición de cómo será la casa.

<br><br>

## 2. OBJETO

Un **Objeto** es una entidad concreta que existe en tiempo de ejecución. Es una **instancia** de una clase. Mientras que la clase es el concepto abstracto, el objeto es la realización física de ese concepto en la memoria del ordenador.

### Componentes de un Objeto
Todo objeto en POO se compone de tres características esenciales:

#### A. Estado (Atributos / Propiedades)
Representa los datos almacenados en el objeto en un momento dado.
*   Son las **variables** declaradas dentro de la clase.
*   El estado puede cambiar a lo largo del tiempo (mutabilidad).
*   *Ejemplo:* El color rojo de un coche, la velocidad actual (0 km/h), el nivel de gasolina.

#### B. Comportamiento (Métodos)
Representa lo que el objeto puede hacer o cómo puede responder a mensajes.
*   Son las **funciones** o procedimientos definidos dentro de la clase.
*   El comportamiento a menudo modifica el estado del objeto.
*   *Ejemplo:* `acelerar()`, `frenar()`, `cambiarMarcha()`.

#### C. Identidad
Es la propiedad que permite distinguir a un objeto de otros, incluso si su estado es idéntico.
*   En programación, esto suele referirse a la dirección de memoria o una referencia única.
*   Dos coches pueden ser ambos "Toyota Corolla Rojo 2022" (mismo estado), pero son dos objetos físicos distintos (distinta identidad).

> **Analogía:** El objeto es la **casa construida** siguiendo el plano. Puedes construir 100 casas idénticas (objetos) con el mismo plano (clase). Cada casa tiene su propia dirección física (identidad), sus propios muebles (estado) y sus propias puertas que se abren y cierran (comportamiento).

### Ejemplo Práctico en Java

A continuación, veremos cómo se traducen estos conceptos a código Java.

#### Definición de la CLASE (`Coche.java`)

```java
// CLASE: La definición o molde
public class Coche {
    // ESTADO (Atributos)
    String marca;
    String modelo;
    String color;
    int anio;

    // Constructor: Inicializa el estado del objeto
    public Coche(String marca, String modelo, String color, int anio) {
        this.marca = marca;
        this.modelo = modelo;
        this.color = color;
        this.anio = anio;
    }

    // COMPORTAMIENTO (Métodos)
    public void acelerar() {
        System.out.println("El " + marca + " " + modelo + " está acelerando...");
    }

    public void frenar() {
        System.out.println("El " + marca + " " + modelo + " ha frenado.");
    }

    public void mostrarInfo() {
        System.out.println("Coche: " + marca + " " + modelo + " (" + color + ")");
    }
}
```

#### Creación y Uso de OBJETOS (`Main.java`)

```java
public class Main {
    public static void main(String[] args) {
        // Instanciación: Creando OBJETOS a partir de la CLASE
        
        // Objeto 1: Tiene su propia identidad y estado
        Coche miCoche = new Coche("Toyota", "Corolla", "Rojo", 2022);
        
        // Objeto 2: Otra identidad, diferente estado
        Coche tuCoche = new Coche("Ford", "Mustang", "Azul", 2023);

        // Interactuando con el comportamiento de los objetos
        miCoche.mostrarInfo(); // Salida: Coche: Toyota Corolla (Rojo)
        miCoche.acelerar();    // Modifica o usa el estado interno

        tuCoche.mostrarInfo(); // Salida: Coche: Ford Mustang (Azul)
        tuCoche.frenar();      
    }
}
```

<br>

### Resumen: Clase vs Objeto

| Característica | Clase | Objeto |
| :--- | :--- | :--- |
| **Rol** | Plantilla / Molde. | Instancia / Entidad. |
| **Naturaleza** | Abstracta (Lógica). | Concreta (Física en memoria). |
| **Componentes** | Define atributos y métodos. | Posee estado, comportamiento e identidad. |
| **Creación** | Se escribe una vez en código. | Se pueden crear múltiples instancias (`new`). |

<br><br>
## 3. ATRIBUTOS

Los **Atributos** (también llamados campos, propiedades o variables de instancia) son las variables que se definen dentro de una clase. Representan el **estado** o las características de los objetos creados a partir de esa clase.

### Características
*   **Almacenamiento de Datos:** Guardan la información específica de cada objeto.
*   **Tipos de Datos:** Pueden ser tipos primitivos (`int`, `double`, `boolean`) u otros objetos (`String`, `Date`, `Motor`).
*   **Visibilidad:** Su acceso puede controlarse mediante modificadores de acceso (`private`, `public`, `protected`). En POO, es buena práctica mantenerlos `private` (Encapsulamiento).

### Ejemplo
En una clase `CuentaBancaria`, los atributos podrían ser:
```java
public class CuentaBancaria {
    // Atributos
    private String numeroCuenta; // Identificador único
    private double saldo;        // Estado cambiante (dinero actual)
    private String titular;      // Nombre del dueño
}
```

<br><br>

## 4. MÉTODOS

Los **Métodos** son funciones definidas dentro de una clase que describen el **comportamiento** de los objetos. Son las acciones que un objeto puede realizar o que se pueden realizar sobre él.

### Tipos Comunes de Métodos
1.  **Constructores:** Métodos especiales que se ejecutan al crear una instancia (`new`). Su objetivo es inicializar los atributos. Tienen el mismo nombre que la clase y no devuelven valor.
2.  **Getters y Setters:** Métodos para leer (`get`) o modificar (`set`) el valor de los atributos privados.
3.  **Métodos de Negocio/Lógica:** Realizan operaciones específicas del dominio del problema (ej. `transferir`, `calcularImpuesto`).

### Ejemplo
Siguiendo con `CuentaBancaria`:

```java
public class CuentaBancaria {
    // ... atributos definidos arriba ...

    // 1. Constructor
    public CuentaBancaria(String titular, String numeroCuenta) {
        this.titular = titular;
        this.numeroCuenta = numeroCuenta;
        this.saldo = 0.0; // Inicializamos saldo en 0
    }

    // 2. Método de Lógica (Comportamiento)
    public void depositar(double cantidad) {
        if (cantidad > 0) {
            this.saldo += cantidad;
            System.out.println("Se depositaron: " + cantidad);
        }
    }

    // 3. Getter (Lectura de estado)
    public double getSaldo() {
        return this.saldo;
    }
}
```

<br><br>

## 5. CONSTRUCTORES

Un **Constructor** es un bloque de código especial dentro de una clase que se ejecuta automáticamente cuando se crea una nueva instancia (objeto) de esa clase.

### Características Clave
*   **Mismo nombre que la clase:** El constructor debe llamarse exactamente igual que la clase.
*   **Sin tipo de retorno:** A diferencia de los métodos normales, no devuelve ningún valor (ni siquiera `void`).
*   **Propósito:** Su función principal es **inicializar** el estado del objeto, es decir, asignar valores iniciales a sus atributos.

### Tipos de Constructores

#### 1. Constructor por Defecto (Vacío)
Si no defines ningún constructor, Java crea uno automáticamente sin parámetros que no hace nada (o inicializa atributos con valores por defecto).
```java
public class Persona {
    String nombre;
    
    // Constructor vacío explícito
    public Persona() {
        this.nombre = "Sin nombre";
    }
}
```

#### 2. Constructor con Parámetros
Permite pasar valores al momento de crear el objeto para inicializar sus atributos con datos específicos.
```java
public class Persona {
    String nombre;
    int edad;

    // Constructor con parámetros
    public Persona(String nombre, int edad) {
        this.nombre = nombre;
        this.edad = edad;
    }
}
```

### Sobrecarga de Constructores
Una clase puede tener múltiples constructores, siempre que tengan diferente número o tipo de parámetros. Esto se conoce como **sobrecarga**.

```java
// Uso
Persona p1 = new Persona();             // Usa el constructor vacío
Persona p2 = new Persona("Ana", 25);    // Usa el constructor con parámetros
```

<br><br>

## 6. ENCAPSULAMIENTO

El **Encapsulamiento** es el principio que consiste en ocultar los detalles internos de funcionamiento de un objeto y exponer solo lo necesario para interactuar con él. Se logra restringiendo el acceso directo a los atributos de la clase y obligando a usar métodos públicos (getters y setters) para manipularlos.

### Beneficios
*   **Control:** Permite validar los datos antes de asignarlos (ej. evitar una edad negativa).
*   **Flexibilidad:** Puedes cambiar la implementación interna sin afectar a quien usa la clase.
*   **Seguridad:** Protege el estado del objeto de modificaciones no autorizadas o inconsistentes.

### Modificadores de Acceso
*   `private`: Solo visible dentro de la misma clase. (Máximo encapsulamiento).
*   `public`: Visible desde cualquier lugar.
*   `protected`: Visible en el mismo paquete y subclases.
*   *(default)*: Visible solo en el mismo paquete.

### Ejemplo
```java
public class Usuario {
    // Atributos privados (Ocultos)
    private String password;
    private int edad;

    // Métodos públicos (Interfaz expuesta)
    public void setEdad(int edad) {
        if (edad >= 0) { // Validación
            this.edad = edad;
        } else {
            System.out.println("Edad no válida");
        }
    }

    public int getEdad() {
        return this.edad;
    }
}
```

<br><br>

## 7. ABSTRACCIÓN

La **Abstracción** es el proceso de identificar las características esenciales de un objeto, ignorando los detalles irrelevantes o complejos. Nos permite centrarnos en **qué** hace un objeto en lugar de **cómo** lo hace internamente.

### Conceptos Clave
*   **Clases Abstractas:** Clases que no se pueden instanciar directamente y pueden contener métodos sin implementación (abstractos). Sirven como base para otras clases.
*   **Interfaces:** Contratos que definen un conjunto de métodos que una clase debe implementar.

### Ejemplo
Imagina un sistema de pagos. No necesitamos saber cómo se conecta con el banco, solo que podemos `procesarPago`.

```java
// Definimos la abstracción (el "qué")
abstract class MetodoPago {
    abstract void procesarPago(double cantidad);
}

// Implementación concreta (el "cómo")
class TarjetaCredito extends MetodoPago {
    @Override
    void procesarPago(double cantidad) {
        System.out.println("Pagando $" + cantidad + " con Tarjeta.");
    }
}

class PayPal extends MetodoPago {
    @Override
    void procesarPago(double cantidad) {
        System.out.println("Pagando $" + cantidad + " vía PayPal.");
    }
}
```

<br><br>

## 8. HERENCIA

La **Herencia** es el mecanismo que permite crear nuevas clases basadas en clases existentes. La nueva clase (Hija/Subclase) adquiere los atributos y métodos de la clase original (Padre/Superclase).

### Beneficios
*   **Reutilización de Código:** No necesitas reescribir código común.
*   **Jerarquía:** Permite organizar las clases de lo más general a lo más específico.
*   **Extensibilidad:** Puedes añadir nuevas funcionalidades a las subclases sin modificar la superclase.

### Sintaxis
En Java se usa la palabra clave `extends`.

### Ejemplo
```java
// Superclase
class Animal {
    void comer() {
        System.out.println("Este animal come.");
    }
}

// Subclase (Hereda de Animal)
class Perro extends Animal {
    void ladrar() {
        System.out.println("Guau!");
    }
}

// Uso
Perro miPerro = new Perro();
miPerro.comer();  // Heredado de Animal
miPerro.ladrar(); // Propio de Perro
```

<br><br>

## 9. POLIMORFISMO

El **Polimorfismo** (del griego "muchas formas") es la capacidad de un objeto de tomar muchas formas. Permite que una referencia de una clase padre apunte a un objeto de cualquier clase hija, y que el método ejecutado sea el específico de la clase hija.

### Tipos Principales
1.  **Sobrecarga (Compile-time):** Mismo nombre de método, diferentes parámetros (visto en Constructores).
2.  **Sobreescritura (Runtime):** Una subclase redefine un método de la superclase para darle un comportamiento específico.

### Ejemplo de Polimorfismo (Sobreescritura)

```java
class Animal {
    void hacerSonido() {
        System.out.println("Sonido genérico");
    }
}

class Perro extends Animal {
    @Override
    void hacerSonido() {
        System.out.println("Guau!");
    }
}

class Gato extends Animal {
    @Override
    void hacerSonido() {
        System.out.println("Miau!");
    }
}

public class Main {
    public static void main(String[] args) {
        // Polimorfismo: Referencia de tipo Animal, objetos de tipos distintos
        Animal miMascota1 = new Perro();
        Animal miMascota2 = new Gato();

        miMascota1.hacerSonido(); // Salida: Guau! (Ejecuta el de Perro)
        miMascota2.hacerSonido(); // Salida: Miau! (Ejecuta el de Gato)
    }
}
```

<br><br>

## 10. CONCEPTOS AVANZADOS

### A. Sobrecarga de Métodos (Overloading)
Ocurre cuando una clase tiene **múltiples métodos con el mismo nombre pero con diferentes parámetros** (cantidad, tipo u orden).
*   **Tipo:** Polimorfismo en tiempo de compilación (estático).
*   **Regla:** La firma del método debe cambiar. El tipo de retorno no es suficiente para diferenciar.

```java
class Calculadora {
    // Método 1: Suma dos enteros
    int sumar(int a, int b) {
        return a + b;
    }

    // Método 2: Suma tres enteros (Sobrecarga por cantidad)
    int sumar(int a, int b, int c) {
        return a + b + c;
    }

    // Método 3: Suma dos doubles (Sobrecarga por tipo)
    double sumar(double a, double b) {
        return a + b;
    }
}
```

### B. Sobrescritura de Métodos (Overriding)
Ocurre cuando una **subclase proporciona una implementación específica** de un método que ya está definido en su superclase.
*   **Tipo:** Polimorfismo en tiempo de ejecución (dinámico).
*   **Regla:** Debe tener exactamente el mismo nombre, parámetros y tipo de retorno (o compatible). Se usa la anotación `@Override`.

```java
class Vehiculo {
    void mover() {
        System.out.println("El vehículo se mueve");
    }
}

class Coche extends Vehiculo {
    @Override
    void mover() {
        System.out.println("El coche conduce por la carretera");
    }
}
```

### Diferencias Clave

| Característica | Sobrecarga (Overloading) | Sobrescritura (Overriding) |
| :--- | :--- | :--- |
| **Ubicación** | Misma clase. | Entre Superclase y Subclase. |
| **Firma** | Debe ser diferente (parámetros). | Debe ser idéntica. |
| **Momento** | Tiempo de Compilación. | Tiempo de Ejecución. |
| **Propósito** | Aumentar legibilidad/flexibilidad. | Modificar comportamiento heredado. |

<br><br>

## 11. RELACIONES ENTRE CLASES

Además de la herencia, las clases pueden relacionarse mediante la **Asociación**. Dos formas especiales de asociación son la Agregación y la Composición.

### A. Agregación (Relación "Tiene un" débil)
Es una relación donde **el objeto hijo puede existir independientemente del padre**. Si el objeto padre se destruye, el hijo sobrevive.
*   **Tipo:** Relación débil.
*   **Ejemplo:** Un `Equipo` tiene `Jugadores`. Si el equipo se disuelve, los jugadores siguen existiendo.

```java
class Jugador {
    String nombre;
}

class Equipo {
    String nombre;
    List<Jugador> jugadores; // Agregación: El equipo tiene jugadores
}
```

### B. Composición (Relación "Tiene un" fuerte)
Es una relación donde **el objeto hijo NO puede existir sin el padre**. El ciclo de vida del hijo depende del padre. Si el padre muere, el hijo también.
*   **Tipo:** Relación fuerte (Muerte en cascada).
*   **Ejemplo:** Una `Casa` tiene `Habitaciones`. Si demueles la casa, las habitaciones dejan de existir.

```java
class Habitacion {
    String tipo;
}

class Casa {
    private List<Habitacion> habitaciones;

    public Casa() {
        // Composición: Las habitaciones se crean junto con la casa
        this.habitaciones = new ArrayList<>();
        this.habitaciones.add(new Habitacion());
    }
}
```

### Diferencias Clave

| Característica | Agregación | Composición |
| :--- | :--- | :--- |
| **Dependencia** | Independiente (Débil). | Dependiente (Fuerte). |
| **Ciclo de Vida** | El hijo sobrevive al padre. | El hijo muere con el padre. |
| **Ejemplo Real** | Coche y Ruedas (puedes cambiar las ruedas). | Libro y Páginas (sin libro no hay páginas). |

<br><br>

## 12. INTERFACES VS CLASES ABSTRACTAS

Ambas se utilizan para lograr la abstracción y definir contratos, pero tienen diferencias fundamentales en su uso y propósito.

### A. Clase Abstracta
Es una clase que no se puede instanciar y puede contener tanto métodos abstractos (sin cuerpo) como métodos concretos (con implementación).
*   **Uso:** Cuando quieres compartir código entre clases relacionadas (relación "es un").
*   **Estado:** Puede tener atributos no estáticos y finales.
*   **Constructor:** Puede tener constructores.

```java
abstract class Animal {
    String nombre;
    
    // Método concreto (compartido)
    void dormir() {
        System.out.println("Zzz...");
    }
    
    // Método abstracto (obligatorio implementar)
    abstract void hacerSonido();
}
```

### B. Interfaz
Es un contrato que define **qué** debe hacer una clase, pero no **cómo**. Antes de Java 8, solo podía tener métodos abstractos y constantes.
*   **Uso:** Para definir capacidades o comportamientos comunes en clases no relacionadas (relación "puede hacer").
*   **Herencia Múltiple:** Una clase puede implementar múltiples interfaces.
*   **Estado:** Solo puede tener constantes (`public static final`).

```java
interface Volador {
    void volar(); // Abstracto por defecto
}

interface Nadador {
    void nadar();
}

// Pato implementa múltiples capacidades
class Pato extends Animal implements Volador, Nadador {
    @Override
    void hacerSonido() { System.out.println("Cuack!"); }
    
    @Override
    public void volar() { System.out.println("Pato volando"); }
    
    @Override
    public void nadar() { System.out.println("Pato nadando"); }
}
```

### Tabla Comparativa

| Característica | Clase Abstracta | Interfaz |
| :--- | :--- | :--- |
| **Palabra clave** | `abstract class` | `interface` |
| **Métodos** | Abstractos y Concretos. | Abstractos (default/static desde Java 8). |
| **Herencia** | Simple (`extends`). | Múltiple (`implements`). |
| **Atributos** | Cualquier tipo. | Solo constantes (`static final`). |
| **Propósito** | Definir una base común (ser algo). | Definir capacidades (hacer algo). |
<br><br>

## 13. PAQUETES (PACKAGES)

Un **Paquete** es un contenedor que agrupa clases, interfaces y subpaquetes relacionados. Funciona como una carpeta en tu sistema de archivos.

### Beneficios
*   **Organización:** Mantiene el código ordenado por funcionalidad (ej. `modelo`, `vista`, `controlador`).
*   **Evita conflictos de nombres:** Puedes tener dos clases con el mismo nombre si están en paquetes diferentes.
*   **Control de Acceso:** Permite usar el modificador de acceso `protected` y *default*.

### Sintaxis
```java
package com.miempresa.proyecto.utilidades;

public class Calculadora {
    // ...
}
```

Para usar una clase de otro paquete, debes importarla:
```java
import com.miempresa.proyecto.utilidades.Calculadora;
```

<br><br>

## 14. MODIFICADORES DE ACCESO

Determinan la **visibilidad** de clases, métodos y atributos. Son fundamentales para el Encapsulamiento.

### Niveles de Acceso (de más restrictivo a menos)

1.  **`private`**:
    *   **Visibilidad:** Solo dentro de la **misma clase**.
    *   **Uso:** Para atributos y métodos internos auxiliares.

2.  **`default` (sin modificador)**:
    *   **Visibilidad:** Dentro del **mismo paquete**.
    *   **Uso:** Clases que solo colaboran entre sí dentro de un módulo.

3.  **`protected`**:
    *   **Visibilidad:** Dentro del **mismo paquete** Y en **subclases** (aunque estén en otro paquete).
    *   **Uso:** Para permitir que las clases hijas accedan a ciertos miembros del padre.

4.  **`public`**:
    *   **Visibilidad:** Desde **cualquier lugar** (cualquier clase, cualquier paquete).
    *   **Uso:** Para la API pública de tu clase (métodos que quieres que otros usen).

### Tabla Resumen

| Modificador | Clase | Paquete | Subclase (otro paquete) | Mundo |
| :--- | :---: | :---: | :---: | :---: |
| **private** | ✅ | ❌ | ❌ | ❌ |
| **default** | ✅ | ✅ | ❌ | ❌ |
| **protected**| ✅ | ✅ | ✅ | ❌ |
| **public** | ✅ | ✅ | ✅ | ✅ |
<br><br>

## 15. MODIFICADOR STATIC

La palabra clave `static` indica que un miembro (atributo o método) **pertenece a la clase en sí misma**, en lugar de a las instancias (objetos) de esa clase.

### Características
*   **Memoria:** Se crea una única copia en memoria, compartida por todos los objetos.
*   **Acceso:** Se puede acceder directamente usando el nombre de la clase, sin necesidad de crear un objeto.

### Usos Comunes
1.  **Atributos Estáticos:** Para contadores compartidos o constantes.
2.  **Métodos Estáticos:** Para utilidades que no dependen del estado de un objeto (ej. `Math.sqrt()`).

### Ejemplo
```java
class Calculadora {
    // Atributo estático (compartido)
    static int contadorOperaciones = 0;

    // Método estático
    static int sumar(int a, int b) {
        contadorOperaciones++;
        return a + b;
    }
}

// Uso
// No necesitamos: Calculadora calc = new Calculadora();
int resultado = Calculadora.sumar(5, 3);
System.out.println(Calculadora.contadorOperaciones); // Acceso directo
```

<br><br>

## 16. MODIFICADOR FINAL

La palabra clave `final` se usa para restringir la modificación. Su efecto depende de dónde se aplique:

### 1. En Variables (Constantes)
Una vez asignado un valor, **no puede cambiar**.
```java
final double PI = 3.14159;
// PI = 3.14; // Error de compilación
```

### 2. En Métodos
El método **no puede ser sobrescrito** (@Override) por las subclases.
```java
class Padre {
    final void metodoSeguro() {
        // Lógica crítica
    }
}
```
```java
class Hijo extends Padre {
    // void metodoSeguro() { ... } // Error: No se puede sobrescribir
}
```

### 3. En Clases
La clase **no puede tener subclases** (no se puede heredar de ella).
```java
final class ClaseSegura { }

// class Intruso extends ClaseSegura { } // Error: No se puede heredar
```

<br><br>

## 17. COHESIÓN Y ACOPLAMIENTO

Son dos métricas fundamentales para evaluar la calidad del diseño de software. El objetivo siempre es: **Alta Cohesión y Bajo Acoplamiento**.

### A. Cohesión (Alta es mejor)
Mide qué tan relacionadas están las responsabilidades dentro de una sola clase o módulo.
*   **Alta Cohesión:** La clase hace una sola cosa y la hace bien (Principio de Responsabilidad Única). Es fácil de entender y mantener.
*   **Baja Cohesión:** La clase hace muchas cosas no relacionadas (Clase "Dios"). Es difícil de entender.

> **Ejemplo:** Una clase `Factura` debe calcular totales e impuestos (Alta cohesión). No debe enviar emails ni guardar en base de datos (Baja cohesión).

### B. Acoplamiento (Bajo es mejor)
Mide el grado de dependencia entre diferentes clases o módulos.
*   **Bajo Acoplamiento:** Las clases son independientes. Cambiar una no afecta a las otras.
*   **Alto Acoplamiento:** Las clases están muy pegadas. Si cambias una, rompes las otras.

> **Ejemplo:** Si `ClaseA` usa directamente `new ClaseB()`, están muy acopladas. Si `ClaseA` usa una `InterfazB`, el acoplamiento es bajo (puedes cambiar la implementación de B sin tocar A).

<br><br>

## 18. CLASES INTERNAS (INNER CLASSES)

Son clases definidas **dentro de otra clase**. Permiten agrupar lógicamente clases que solo se usan en un lugar, aumentando la encapsulación.

### Tipos Principales

#### 1. Clase Interna Miembro (Member Inner Class)
Es un miembro más de la clase externa. Tiene acceso a los miembros privados de la clase externa.
```java
class Externa {
    private String mensaje = "Hola";

    class Interna {
        void imprimir() {
            System.out.println(mensaje); // Accede a privado
        }
    }
}
```

#### 2. Clase Interna Estática (Static Nested Class)
Es como una clase normal, pero anidada dentro de otra para agrupar. No tiene acceso directo a los miembros de instancia de la clase externa.
```java
class Externa {
    static class Estatica {
        // ...
    }
}
```

#### 3. Clase Interna Local (Local Inner Class)
Definida dentro de un método. Solo visible dentro de ese método.

#### 4. Clase Anónima (Anonymous Class)
Clase sin nombre declarada e instanciada en una sola expresión. Muy usada para implementar interfaces al vuelo (ej. Listeners).
```java
boton.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent e) {
        System.out.println("Click!");
    }
});
```

<br><br>

## 19. ENUMERACIONES (ENUMS)

Un **Enum** es un tipo de clase especial que representa un grupo fijo de constantes (variables inmutables).

### Características
*   **Tipado Fuerte:** Evita errores al usar constantes numéricas o cadenas mágicas.
*   **Son Clases:** En Java, los enums pueden tener atributos, métodos y constructores.
*   **Uso:** Días de la semana, estados de un pedido, colores, roles de usuario.

### Ejemplo
```java
enum Dia {
    LUNES, MARTES, MIERCOLES, JUEVES, VIERNES, SABADO, DOMINGO
}

// Enum con atributos y comportamiento
enum Nivel {
    BAJO(1), MEDIO(5), ALTO(10); // Constantes invocan al constructor

    private final int valor;

    Nivel(int valor) {
        this.valor = valor;
    }

    public int getValor() {
        return valor;
    }
}

// Uso
Nivel miNivel = Nivel.ALTO;
System.out.println(miNivel.getValor()); // 10
```

<br><br>

## 20. MANEJO DE EXCEPCIONES (POO)

En POO, los errores son objetos. Cuando ocurre un error, se "lanza" (throw) un objeto de tipo `Exception` que contiene información sobre el problema.

### Jerarquía de Excepciones
Todas heredan de `Throwable`.
1.  **Error:** Errores graves de la JVM (ej. `OutOfMemoryError`). No se suelen capturar.
2.  **Exception:** Errores que el programa puede manejar.
    *   **Checked Exceptions:** El compilador obliga a manejarlas (ej. `IOException`, `SQLException`).
    *   **Unchecked Exceptions (Runtime):** Errores de programación (ej. `NullPointerException`, `ArithmeticException`). No es obligatorio capturarlas.

### Bloque Try-Catch-Finally
Estructura para manejar el flujo en caso de error.

```java
try {
    // Código que puede fallar
    int division = 10 / 0;
} catch (ArithmeticException e) {
    // Qué hacer si ocurre la excepción específica
    System.out.println("No se puede dividir por cero: " + e.getMessage());
} catch (Exception e) {
    // Captura genérica (polimorfismo de excepciones)
    System.out.println("Ocurrió otro error");
} finally {
    // Código que SIEMPRE se ejecuta (éxito o error)
    System.out.println("Operación finalizada");
}
```

### Excepciones Personalizadas
Puedes crear tus propias excepciones heredando de `Exception` o `RuntimeException`.

```java
class SaldoInsuficienteException extends Exception {
    public SaldoInsuficienteException(String mensaje) {
        super(mensaje);
    }
}
```





