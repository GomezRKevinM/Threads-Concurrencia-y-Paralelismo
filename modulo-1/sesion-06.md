# Sesión 06 — Creando hilos de formas diferentes

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 45 minutos |
| **Tipo** | 💻 Práctica |
| **Dificultad** | Básica |

---

## 🧵 Forma 1 — Extendiendo la clase `Thread`

```java
public class HiloDescarga extends Thread {

    private String nombreSerie;

    public HiloDescarga(String nombreSerie) {
        super("hilo-" + nombreSerie);
        this.nombreSerie = nombreSerie;
    }

    @Override
    public void run() {
        System.out.println("[" + getName() + "] Descargando: " + nombreSerie);
        try {
            Thread.sleep(1500);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println("[" + getName() + "] Completado: " + nombreSerie);
    }
}

// Uso:
HiloDescarga hilo = new HiloDescarga("Breaking Bad");
hilo.start();
```

> ⚠️ **Problema:** Java no permite herencia múltiple. Si tu clase ya extiende otra, **no puedes usar esta forma.**

---

## 🧵 Forma 2 — Implementando `Runnable`

```java
public class TareaDescarga implements Runnable {

    private String nombreSerie;

    public TareaDescarga(String nombreSerie) {
        this.nombreSerie = nombreSerie;
    }

    @Override
    public void run() {
        System.out.println("[" + Thread.currentThread().getName()
                         + "] Descargando: " + nombreSerie);
        try {
            Thread.sleep(1500);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println("[" + Thread.currentThread().getName()
                         + "] Completado: " + nombreSerie);
    }
}

// Uso:
Runnable tarea = new TareaDescarga("Dark");
Thread hilo = new Thread(tarea, "hilo-Dark");
hilo.start();
```

> ✅ **Ventaja:** separas la **tarea** (qué hacer) del **hilo** (quién lo ejecuta).

---

## 🧵 Forma 3 — Lambda

Desde Java 8, `Runnable` es una **interfaz funcional** — tiene un solo método abstracto (`run()`). Puedes reemplazarla con una lambda:

```java
// Clase anónima (verbosa)
Thread hilo1 = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Descargando Squid Game...");
    }
});

// Lambda (equivalente, limpia)
Thread hilo2 = new Thread(() -> {
    System.out.println("Descargando Squid Game...");
});

// Lambda en una línea
Thread hilo3 = new Thread(() -> System.out.println("Descargando Squid Game..."));
```

---

## 🧵 Forma 4 — `Callable` + `Future`

Cuando necesitas que el hilo **retorne un resultado**:

```java
import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;

Callable<String> tarea = () -> {
    Thread.sleep(1500);
    return "Datos de Breaking Bad listos"; // ← puede retornar valor
};

FutureTask<String> futuro = new FutureTask<>(tarea);
Thread hilo = new Thread(futuro);
hilo.start();

// El hilo principal puede seguir haciendo otras cosas...
System.out.println("Haciendo otras tareas mientras descarga...");

// Cuando necesito el resultado:
String resultado = futuro.get(); // espera si no está listo
System.out.println(resultado);
```

---

## 📊 Comparativa

| | Hereda Thread | Retorna resultado | Lanza excepciones | Uso actual |
|---|---|---|---|---|
| `extends Thread` | ✅ | ❌ | ❌ | Raro |
| `Runnable` | ❌ | ❌ | ❌ | Común |
| `Lambda` | ❌ | ❌ | ❌ | Común |
| `Callable+Future` | ❌ | ✅ | ✅ | Módulo 4 |

---

## 🔑 Interfaz funcional — concepto clave

Una **interfaz funcional** es aquella que tiene **exactamente 1 método abstracto**. Por eso acepta lambdas — Java puede inferir cuál método implementa la lambda.

```java
// ✅ Interfaz funcional — 1 solo método abstracto
interface Saludo {
    String saludar(String nombre);
}

// Acepta lambda porque Java sabe que implementa saludar()
Saludo s = nombre -> "Hola, " + nombre;

// ❌ NO es interfaz funcional — 2 métodos abstractos
interface Procesador {
    void procesar(String dato);
    void validar(String dato);
    // Lambda lanzaría error de compilación
}
```

Puedes anotarla con `@FunctionalInterface` para que el compilador te avise si agregas un segundo método abstracto.

---

## 🛠️ Practica: las 3 formas juntas

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {

        // Forma 1 — extendiendo Thread
        HiloDescarga hilo1 = new HiloDescarga("Breaking Bad");

        // Forma 2 — Runnable
        Thread hilo2 = new Thread(new TareaDescarga("Dark"), "hilo-Dark");

        // Forma 3 — Lambda
        Thread hilo3 = new Thread(() -> {
            System.out.println("[hilo-Squid] Descargando: Squid Game");
            try { Thread.sleep(1500); }
            catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            System.out.println("[hilo-Squid] Completado: Squid Game");
        }, "hilo-Squid");

        hilo1.start();
        hilo2.start();
        hilo3.start();

        hilo1.join();
        hilo2.join();
        hilo3.join();

        System.out.println("\nTodas las descargas completadas.");
    }
}
```

---

## ✅ Checklist de la sesión

- [ ] Creaste `HiloDescarga.java` extendiendo `Thread`
- [ ] Creaste `TareaDescarga.java` implementando `Runnable`
- [ ] Ejecutaste las 3 formas simultáneamente
- [ ] Entiendes qué es una interfaz funcional y por qué acepta lambdas
- [ ] Entiendes por qué `extends Thread` se usa poco hoy en día

---

## 🧠 Mini-test

**Pregunta 1**
¿Cuál es la principal desventaja de crear hilos extendiendo la clase `Thread`?

- A) Es más lenta que usar `Runnable`
- B) No permite herencia múltiple — si tu clase ya extiende otra no puedes usarla
- C) No puedes darle nombre al hilo
- D) Solo funciona en Java 8 o superior

**Pregunta 2**
¿Qué diferencia fundamental tiene `Callable` respecto a `Runnable`?

- A) `Callable` crea el hilo automáticamente sin necesitar `new Thread()`
- B) `Callable` puede retornar un resultado y lanzar excepciones chequeadas
- C) `Callable` es más rápido porque usa menos memoria
- D) `Callable` garantiza el orden de ejecución entre hilos

**Pregunta 3**
¿Por qué una lambda puede usarse donde se espera un `Runnable`?

- A) Porque Java convierte automáticamente cualquier lambda en un `Thread`
- B) Porque `Runnable` es una interfaz funcional — tiene un solo método abstracto (`run()`)
- C) Porque las lambdas y `Runnable` son la misma cosa internamente
- D) Porque Java 8 eliminó `Runnable` y lo reemplazó con lambdas

**Pregunta 4**
Tienes una clase `Episodio` que ya extiende `Contenido`. Necesitas que también pueda ejecutarse en un hilo. ¿Qué forma usas?

- A) `extends Thread` — es la forma más simple
- B) No es posible, Java no permite clases que corran en hilos si ya extienden otra clase
- C) `implements Runnable` — no afecta la herencia existente
- D) Usar una lambda dentro de la clase

---

➡️ [Siguiente sesión: Evitando el saldo negativo](./sesion-07.md)
