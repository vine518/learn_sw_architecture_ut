### Tema 3: Principios de Diseño de Software

#### **Introducción**
Los principios de diseño de software son directrices fundamentales que ayudan a los desarrolladores a crear sistemas eficientes, mantenibles y escalables. Estos principios establecen las bases para diseñar software de alta calidad que pueda adaptarse a los cambios y facilitar el trabajo en equipo.

---

### **1. Principios Fundamentales**

#### **1.1 Abstracción**
- **Definición:** El proceso de identificar las características esenciales de un objeto, ignorando los detalles innecesarios.
- **Objetivo:** Reducir la complejidad y centrar la atención en aspectos importantes.

**Ejemplo:** En una aplicación bancaria, abstraer una clase "Cuenta" con métodos como "depositar" y "retirar", sin detallar cómo se implementa cada operación.

#### **1.2 Encapsulación**
- **Definición:** Agrupar datos y métodos que operan sobre esos datos dentro de una clase, ocultando detalles internos.
- **Objetivo:** Proteger la integridad de los datos y controlar el acceso a ellos.

**Ejemplo:**
```java
public class Cuenta {
    private double saldo;

    public void depositar(double cantidad) {
        if (cantidad > 0) {
            saldo += cantidad;
        }
    }

    public double getSaldo() {
        return saldo;
    }
}
```

#### **1.3 Principio de Responsabilidad Única (SRP)**
- **Definición:** Una clase debe tener una única razón para cambiar.
- **Objetivo:** Reducir la complejidad al asignar responsabilidades específicas.

**Ejemplo:**
Antes:
```java
public class Reporte {
    public void generarReporte() {}
    public void guardarReporte() {}
}
```
Después:
```java
public class GeneradorReporte {
    public void generarReporte() {}
}

public class AlmacenadorReporte {
    public void guardarReporte() {}
}
```

#### **1.4 Principio Abierto/Cerrado (OCP)**
- **Definición:** Las clases deben estar abiertas para extensión, pero cerradas para modificación.

**Ejemplo:**
Antes:
```java
public class Calculadora {
    public int calcular(String operacion, int a, int b) {
        if (operacion.equals("suma")) {
            return a + b;
        } else if (operacion.equals("resta")) {
            return a - b;
        }
        return 0;
    }
}
```
Después:
```java
interface Operacion {
    int calcular(int a, int b);
}

public class Suma implements Operacion {
    public int calcular(int a, int b) {
        return a + b;
    }
}

public class Resta implements Operacion {
    public int calcular(int a, int b) {
        return a - b;
    }
}
```

#### **1.5 Principio de Sustitución de Liskov (LSP)**
- **Definición:** Las subclases deben ser reemplazables por sus clases base sin alterar el comportamiento del programa.

**Ejemplo:**
Antes:
```java
public class Rectangulo {
    public void setAncho(int ancho) {}
    public void setAlto(int alto) {}
}

public class Cuadrado extends Rectangulo {
    @Override
    public void setAncho(int lado) {}
}
```
Después:
```java
interface Figura {
    double calcularArea();
}

public class Rectangulo implements Figura {
    public double calcularArea() {}
}

public class Cuadrado implements Figura {
    public double calcularArea() {}
}
```

#### **1.6 Principio de Segregación de Interfaces (ISP)**
- **Definición:** Las interfaces deben ser específicas y evitar obligar a implementar métodos innecesarios.

**Ejemplo:**
Antes:
```java
interface Trabajador {
    void cocinar();
    void conducir();
    void limpiar();
}
```
Después:
```java
interface Cocinero {
    void cocinar();
}

interface Conductor {
    void conducir();
}
```

#### **1.7 Principio de Inversión de Dependencias (DIP)**
- **Definición:** Los módulos de alto nivel no deben depender de los módulos de bajo nivel; ambos deben depender de abstracciones.

**Ejemplo:**
Antes:
```java
public class Motor {
    public void encender() {}
}

public class Coche {
    private Motor motor = new Motor();
    public void encender() {
        motor.encender();
    }
}
```
Después:
```java
interface Motor {
    void encender();
}

public class MotorDiesel implements Motor {
    public void encender() {}
}

public class Coche {
    private Motor motor;

    public Coche(Motor motor) {
        this.motor = motor;
    }

    public void encender() {
        motor.encender();
    }
}
```

---

### **2. Principios Complementarios**

#### **2.1 DRY (Don't Repeat Yourself)**
- **Definición:** Evitar la duplicación de lógica o código.

**Ejemplo:**
Antes:
```java
public class Cliente {
    public String obtenerNombreCompleto(String nombre, String apellido) {
        return nombre + " " + apellido;
    }
}
```
Después:
```java
public class Utilidades {
    public static String obtenerNombreCompleto(String nombre, String apellido) {
        return nombre + " " + apellido;
    }
}
```

#### **2.2 KISS (Keep It Simple, Stupid)**
- **Definición:** Diseñar sistemas lo más simples posible.

**Ejemplo:**
Antes (violación de KISS):
```java
public class Calculadora {
    public int calcular(String operacion, int a, int b) {
        switch (operacion) {
            case "suma": return a + b;
            case "resta": return a - b;
            case "multiplicacion": return a * b;
            case "division": return a / b;
            default: throw new UnsupportedOperationException("Operación no soportada");
        }
    }
}
```
Después (aplicando KISS):
```java
public class Calculadora {
    public int sumar(int a, int b) {
        return a + b;
    }

    public int restar(int a, int b) {
        return a - b;
    }
}
```

#### **2.3 YAGNI (You Aren't Gonna Need It)**
- **Definición:** No implementar funcionalidad que no se necesite actualmente.

**Ejemplo:**
Antes (violación de YAGNI):
```java
public class Usuario {
    private String nombre;
    private String apellido;
    private String direccion;
    private String telefono;
    private String nacionalidad;
    private String hobbies;

    public Usuario(String nombre, String apellido) {
        this.nombre = nombre;
        this.apellido = apellido;
    }
}
```
Después (aplicando YAGNI):
```java
public class Usuario {
    private String nombre;
    private String apellido;

    public Usuario(String nombre, String apellido) {
        this.nombre = nombre;
        this.apellido = apellido;
    }
}
```

---

### **3. Referencia a Domain-Driven Design (DDD)**
Para obtener una descripción detallada de los conceptos básicos de Domain-Driven Design (DDD), como Bounded Contexts y Event Storming, consulta el documento "Introducción a Domain-Driven Design", que incluye ejemplos avanzados y una implementación práctica.

---

### **4. Ejercicios Prácticos**
1. Refactorizar un código que viole SRP.
2. Diseñar una aplicación usando principios complementarios como DRY y KISS.
3. Implementar una jerarquía de clases que cumpla con LSP.

---

### **Referencias y Recursos**
1. "Clean Code" - Robert C. Martin.
2. [Refactoring Guru: Principios de Diseño](https://refactoring.guru/design-patterns/solid).
3. [Domain-Driven Design Reference](https://domainlanguage.com/).

---

Este documento presenta los principios clave para diseñar software de alta calidad, con ejemplos prácticos y orientaciones claras para aplicarlos en proyectos reales.

