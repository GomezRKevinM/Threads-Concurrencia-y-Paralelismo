# Sesión 11 — Sistema de Biblioteca (Manos a la obra)

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 60 minutos |
| **Tipo** | 🛠️ Manos a la obra |
| **Dificultad** | Intermedia |

---

## 🎯 Formato de esta sesión

Sesión **guiada pero autónoma**. Se te da el escenario, los requisitos y pistas — tú construyes. El objetivo es aplicar lo aprendido sin andamiaje.

---

## 📚 El escenario

Una biblioteca digital tiene libros que múltiples usuarios pueden intentar tomar prestados simultáneamente. El sistema debe:

```
1. Tener un catálogo de 5 libros con stock inicial de 3 copias cada uno
2. Múltiples usuarios (hilos) intentan tomar prestado el mismo libro
3. Nunca prestar más copias de las disponibles (stock nunca negativo)
4. Llevar un registro thread-safe de todos los préstamos exitosos
5. Cuando todos los usuarios terminen, mostrar el reporte final
6. Medir el tiempo total del proceso
```

---

## 👥 Comportamiento de cada usuario

```
1. Elegir un libro al azar del catálogo
2. Intentar tomarlo prestado
3. Si hay stock → préstamo exitoso → registrarlo
4. Si no hay stock → registrar intento fallido
5. Simular tiempo de procesamiento (sleep aleatorio 500-1500ms)
```

---

## 📋 Requisitos técnicos

```
Clases mínimas:
├── Libro.java              → nombre, autor, stock disponible
├── Biblioteca.java         → catálogo, lógica de préstamo
├── Usuario.java            → implementa Runnable
└── Main.java               → lanza 20 usuarios simultáneos

Herramientas que DEBES usar:
├── Al menos 1 mecanismo de sincronización (tú eliges cuál)
├── Una estructura thread-safe para el registro
├── join() para esperar a todos los usuarios
└── Medición de tiempo total

Restricciones:
├── El stock NUNCA puede ser negativo
├── El total de préstamos exitosos no puede superar
    stock_inicial × número_de_libros = 15
└── Sin race conditions en el registro de préstamos
```

---

## 🗺️ Pistas

**Pista 1 — sobre la sincronización:**
> El préstamo tiene 2 pasos: verificar stock y decrementar. ¿Recuerdas qué herramienta protege secuencias completas?

**Pista 2 — sobre el registro:**
> Múltiples hilos escribirán préstamos simultáneamente. ¿Qué estructura viste que es thread-safe para colecciones?

**Pista 3 — sobre Usuario:**
> Un usuario es una tarea que corre en un hilo. ¿Qué interfaz implementan las tareas en Java?

**Pista 4 — sobre el reporte:**
> El hilo principal no debe mostrar el reporte hasta que todos los usuarios terminen. ¿Qué método viste para esto?

---

## ✅ Criterios de evaluación

```
✅ Funcionalidad    → el sistema corre sin errores
✅ Thread-safety    → stock nunca negativo, sin race conditions
✅ Herramientas     → usaste las correctas para cada problema
✅ Estructura       → responsabilidades bien separadas por clase
✅ Código limpio    → nombres claros, finally donde corresponde
```

---

## 💡 Solución de referencia

### `Libro.java`

```java
public class Libro {
    private String nombre;
    private String autor;

    public Libro(String nombre, String autor) {
        this.nombre = nombre;
        this.autor = autor;
    }

    public String getNombre() { return nombre; }
    public String getAutor() { return autor; }

    @Override
    public String toString() { return nombre; }
}
```

### `Biblioteca.java`

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.concurrent.Semaphore;

public class Biblioteca {

    // Cada libro tiene su propio Semaphore con sus copias
    private final ConcurrentHashMap<Libro, Semaphore> catalogo = new ConcurrentHashMap<>();
    private final ConcurrentHashMap<Libro, CopyOnWriteArrayList<String>> registros = new ConcurrentHashMap<>();
    private final ConcurrentHashMap<String, Integer> fallos = new ConcurrentHashMap<>();

    public void agregarLibro(Libro libro, int copias) {
        catalogo.put(libro, new Semaphore(copias));
        registros.put(libro, new CopyOnWriteArrayList<>());
    }

    public void prestar(Libro libro, String usuario) {
        Semaphore copias = catalogo.get(libro);
        boolean disponible = copias.tryAcquire(); // no bloquea

        if (disponible) {
            try {
                registros.get(libro).add(usuario);
                System.out.printf("✅ [%s] tomó prestado: %s | Copias restantes: %d%n",
                        usuario, libro, copias.availablePermits());
                Thread.sleep((long)(Math.random() * 1000) + 500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                copias.release(); // SIEMPRE devuelve la copia en finally
                System.out.printf("📖 [%s] devolvió: %s%n", usuario, libro);
            }
        } else {
            fallos.merge(libro.getNombre(), 1, Integer::sum);
            System.out.printf("❌ [%s] no pudo tomar: %s (sin copias)%n", usuario, libro);
        }
    }
}
```

### `Main.java`

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {

        Biblioteca biblioteca = new Biblioteca();

        Libro[] libros = {
            new Libro("Clean Code", "Robert Martin"),
            new Libro("The Pragmatic Programmer", "Hunt & Thomas"),
            new Libro("Effective Java", "Joshua Bloch"),
            new Libro("Design Patterns", "Gang of Four"),
            new Libro("Refactoring", "Martin Fowler")
        };

        for (Libro libro : libros) biblioteca.agregarLibro(libro, 3);

        System.out.println("📚 BIBLIOTECA DIGITAL — 5 libros · 3 copias · 20 usuarios\n");
        long inicio = System.currentTimeMillis();

        Thread[] hilos = new Thread[20];
        for (int i = 0; i < 20; i++) {
            String usuario = "Usuario-" + (i + 1);
            Libro libroElegido = libros[(int)(Math.random() * libros.length)];
            hilos[i] = new Thread(() -> biblioteca.prestar(libroElegido, usuario));
            hilos[i].start();
        }

        for (Thread hilo : hilos) hilo.join();

        long fin = System.currentTimeMillis();
        System.out.println("⏱️  Tiempo total: " + (fin - inicio) + "ms");
    }
}
```

---

## 🔑 Decisiones clave de diseño

```
Un Semaphore por libro:
❌ Semaphore global → tomar "Clean Code" afecta disponibilidad de "Effective Java"
✅ Semaphore por libro → cada libro controla sus propias copias independientemente

tryAcquire() en lugar de acquire():
acquire()    → el hilo espera hasta que haya copia (no refleja la realidad)
tryAcquire() → si no hay copia → retorna false inmediatamente (más realista)

CopyOnWriteArrayList para registros:
→ thread-safe para múltiples escritores simultáneos
→ permite guardar múltiples usuarios por libro
```

---

## 🧠 Mini-test

**Pregunta 1**
¿Por qué se usa un `Semaphore` por libro en lugar de uno global?

**Pregunta 2**
¿Cuál es la diferencia entre `acquire()` y `tryAcquire()`? ¿Cuándo conviene cada uno?

**Pregunta 3**
Si usaras `ArrayList` en lugar de `CopyOnWriteArrayList` para el registro, ¿qué problema podría ocurrir?

**Pregunta 4**
¿Por qué `copias.release()` debe estar en `finally` y no al final del bloque `try`?

---

➡️ [Siguiente sesión: Lista de ejercicios](./sesion-12.md)
