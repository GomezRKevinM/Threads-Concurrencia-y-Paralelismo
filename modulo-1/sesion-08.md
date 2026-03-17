# Sesión 08 — Herramientas de sincronización en Java

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 40 minutos |
| **Tipo** | 📖 Teoría |
| **Dificultad** | Intermedia |

---

## 🔒 1. `ReentrantLock` — el `synchronized` con superpoderes

```java
import java.util.concurrent.locks.ReentrantLock;

public class CuentaBancaria {
    private double saldo;
    private ReentrantLock cerrojo = new ReentrantLock();

    public void depositar(double monto) {
        cerrojo.lock();
        try {
            saldo += monto;
        } finally {
            cerrojo.unlock(); // SIEMPRE en finally — nunca omitir
        }
    }
}
```

> ⚠️ **Regla de oro:** siempre libera el cerrojo en `finally`. Sin esto, una excepción deja el cerrojo tomado para siempre → **deadlock eterno**.

**Ventajas sobre `synchronized`:**

```java
// 1. tryLock() — intenta sin quedarse esperando
if (cerrojo.tryLock()) {
    try { saldo += monto; }
    finally { cerrojo.unlock(); }
} else {
    System.out.println("Cerrojo ocupado, lo intento después");
}

// 2. tryLock() con timeout
if (cerrojo.tryLock(2, TimeUnit.SECONDS)) {
    // entró antes de 2 segundos
}

// 3. lockInterruptibly() — permite cancelar la espera
cerrojo.lockInterruptibly();
```

---

## 🚦 2. `Semaphore` — controla cuántos hilos entran simultáneamente

```java
import java.util.concurrent.Semaphore;

// Solo 3 hilos pueden descargar al mismo tiempo
Semaphore semaforo = new Semaphore(3);

public void descargar(String serie) {
    try {
        semaforo.acquire(); // toma un permiso (espera si no hay)
        System.out.println(Thread.currentThread().getName() + " descargando " + serie);
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        semaforo.release(); // devuelve el permiso — SIEMPRE en finally
    }
}
```

**Analogía:**
```
Semaphore(3) = estacionamiento con 3 lugares:
- Si hay lugar → entras 🚗
- Si está lleno → esperas hasta que salga alguien 🚗💨
```

---

## 🏁 3. `CountDownLatch` — espera a que N hilos terminen

```java
import java.util.concurrent.CountDownLatch;

CountDownLatch latch = new CountDownLatch(3);

for (String serie : series) {
    new Thread(() -> {
        try {
            descargar(serie);
        } finally {
            latch.countDown(); // resta 1 al contador — dentro del hilo
        }
    }).start();
}

latch.await(); // el hilo principal espera hasta que el contador llegue a 0
System.out.println("Todas las descargas completadas ✅");
```

```
Contador inicial: 3
hilo-Breaking termina → countDown() → contador: 2
hilo-Dark termina     → countDown() → contador: 1
hilo-Squid termina    → countDown() → contador: 0 → await() se desbloquea ✅
```

> ⚠️ Usa **`await()`** no `wait()`. Son métodos completamente diferentes.

---

## 🔄 4. `CyclicBarrier` — sincroniza hilos en un punto de encuentro

```java
import java.util.concurrent.CyclicBarrier;

CyclicBarrier barrera = new CyclicBarrier(3, () ->
    System.out.println("¡Todos listos! Iniciando siguiente fase...")
);

for (String serie : series) {
    new Thread(() -> {
        System.out.println(serie + ": preparando...");
        try {
            barrera.await(); // espera a que los otros 2 lleguen aquí
            System.out.println(serie + ": procesando en paralelo!");
        } catch (Exception e) {
            Thread.currentThread().interrupt();
        }
    }).start();
}
```

**Diferencia con `CountDownLatch`:**

```
CountDownLatch  → un hilo espera a que otros terminen (1 → N)
                → no reutilizable

CyclicBarrier   → todos los hilos se esperan entre sí (N → N)
                → reutilizable — puede usarse en múltiples fases
```

---

## 📦 5. Colecciones thread-safe

```java
// ❌ No thread-safe — race condition garantizada
List<String> lista = new ArrayList<>();
Map<String, Integer> mapa = new HashMap<>();

// ✅ Thread-safe — diseñadas para concurrencia
List<String> lista = new CopyOnWriteArrayList<>();
Map<String, Integer> mapa = new ConcurrentHashMap<>();
Queue<String> cola = new ConcurrentLinkedQueue<>();
```

**¿Cómo funciona `ConcurrentHashMap`?**

```
HashMap normal:
┌─────────────────────────────┐
│     Un solo array interno   │
│  Hilo-A escribe en [2] 🔒  │
│  Hilo-B escribe en [5] ⏳  │ ← espera aunque toca otra posición
└─────────────────────────────┘

ConcurrentHashMap:
┌──────────┬──────────┬──────────┐
│Segmento 1│Segmento 2│Segmento 3│
│ Hilo-A ✅│ Hilo-B ✅│ Hilo-C ✅│ ← los 3 en paralelo sin bloquearse
└──────────┴──────────┴──────────┘
```

---

## 🗺️ Mapa mental — ¿qué herramienta usar?

```
¿Qué necesitas?
│
├── Proteger 1 sección crítica
│   ├── Lógica simple (contador)     → AtomicInteger
│   ├── Lógica compleja              → synchronized / ReentrantLock
│   └── Necesitas tryLock/timeout    → ReentrantLock
│
├── Controlar cuántos hilos entran
│   └── Recurso con capacidad N      → Semaphore
│
├── Esperar que hilos terminen
│   ├── Pocos hilos con referencia   → join()
│   └── Muchos hilos sin referencia  → CountDownLatch
│
├── Sincronizar hilos en un punto
│   └── Todos esperan a todos        → CyclicBarrier
│
└── Compartir colecciones
    └── Lista / Map / Queue          → ConcurrentHashMap
                                       CopyOnWriteArrayList
```

---

## ✅ Checklist de la sesión

- [ ] Entiendes cuándo usar `ReentrantLock` sobre `synchronized`
- [ ] Sabes qué problema resuelve `Semaphore`
- [ ] Entiendes la diferencia entre `CountDownLatch` y `CyclicBarrier`
- [ ] Sabes por qué `ArrayList` y `HashMap` son peligrosos en multihilo

---

## 🧠 Mini-test

**Pregunta 1**
¿Por qué es obligatorio liberar un `ReentrantLock` en un bloque `finally`?

- A) Porque `finally` es más rápido que un `if`
- B) Porque si ocurre una excepción antes del `unlock()` sin `finally`, el cerrojo queda tomado bloqueando todos los hilos que esperen
- C) Porque `ReentrantLock` lanza una excepción si no se usa `finally`
- D) Es solo una buena práctica, no tiene consecuencias reales

**Pregunta 2**
Tienes una API externa que solo soporta 5 conexiones simultáneas. ¿Qué herramienta usarías?

- A) `synchronized` — solo deja pasar 1 hilo a la vez
- B) `CountDownLatch` — cuenta cuántas conexiones han terminado
- C) `Semaphore(5)` — permite exactamente 5 hilos simultáneos
- D) `CyclicBarrier(5)` — espera a que haya 5 hilos listos antes de conectar

**Pregunta 3**
¿Cuál es la diferencia clave entre `CountDownLatch` y `CyclicBarrier`?

- A) `CountDownLatch` es más rápido porque usa menos memoria
- B) `CountDownLatch`: un hilo espera a que otros terminen; `CyclicBarrier`: todos los hilos se esperan entre sí y es reutilizable
- C) `CyclicBarrier` solo funciona con exactamente 2 hilos
- D) Son idénticos, solo cambia el nombre

**Pregunta 4**
¿Por qué `HashMap` es peligroso en un entorno multihilo?

- A) Porque `HashMap` no acepta valores `null`
- B) Porque sus operaciones internas no son atómicas — dos hilos modificándolo simultáneamente pueden corromper su estructura interna
- C) Porque `HashMap` usa demasiada memoria con múltiples hilos
- D) Porque Java prohíbe usar `HashMap` con `Thread`

---

➡️ [Siguiente sesión: Esperando que los threads finalicen](./sesion-09.md)
