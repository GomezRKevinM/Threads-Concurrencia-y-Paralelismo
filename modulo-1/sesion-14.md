# Sesión 14 — ¿Qué aprendimos?

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 35 minutos |
| **Tipo** | 🎓 Cierre del módulo |
| **Dificultad** | — |

---

## 🗺️ El mapa completo de lo que dominaste

---

### 🧵 Bloque 1 — Fundamentos de hilos

```
✅ Proceso vs Hilo
   Proceso  → programa en ejecución, memoria aislada entre procesos
   Hilo     → unidad mínima de ejecución, comparte memoria del proceso

✅ Estados de un hilo
   NEW → RUNNABLE → RUNNING → WAITING/TIMED_WAITING → TERMINATED
   
   scheduler interrumpe  → RUNNABLE (listo, esperando CPU)
   sleep/wait/join        → WAITING  (dormido, necesita señal)

✅ El Scheduler
   Decide quién corre y cuándo — nunca asumas orden entre hilos
   Puede interrumpir entre cualquier par de instrucciones

✅ 3 formas de crear hilos
   extends Thread     → limitado por herencia única — uso raro
   implements Runnable → flexible, separación tarea/hilo — preferido
   Lambda             → limpio para tareas simples — más común hoy
```

---

### ⚠️ Bloque 2 — El problema central

```
✅ Race Condition
   Ocurre cuando 2+ hilos acceden a datos compartidos
   sin sincronización — el resultado es impredecible

✅ Por qué ocurre
   totalDescargados++ son 3 pasos: LEE → SUMA → ESCRIBE
   el scheduler puede interrumpir entre cualquier paso

✅ Por qué es peligroso
   No lanza excepciones — falla silenciosamente
   Puede no aparecer en desarrollo y explotar en producción
   Un bug que ocurre 1 de cada 1000 veces es igual de peligroso
```

---

### 🔒 Bloque 3 — Herramientas de sincronización

```
✅ synchronized
   Cerrojo del OBJETO COMPLETO — 1 hilo a la vez
   En método → protege todo el método
   En bloque  → protege solo la sección crítica

✅ ReentrantLock
   synchronized con superpoderes
   tryLock(), timeout, lockInterruptibly()
   SIEMPRE unlock() en finally — sin esto = deadlock eterno

✅ AtomicInteger / AtomicLong
   Para contadores — usa CAS a nivel de CPU
   Sin cerrojo → más eficiente que synchronized para operaciones simples
   No mezclar con synchronized — son estrategias distintas

✅ Semaphore
   Controla N hilos simultáneos (N permisos)
   acquire() → toma permiso (espera si no hay)
   tryAcquire() → toma permiso sin bloquear (retorna false si no hay)
   release() → devuelve permiso — SIEMPRE en finally

✅ CountDownLatch
   Un hilo espera a que N terminen
   countDown() DENTRO del hilo al terminar
   await() en el hilo que espera — NO wait()

✅ CyclicBarrier
   N hilos se esperan entre sí en un punto de encuentro
   Reutilizable — ideal para múltiples fases
   todos llegan a await() → todos continúan juntos

✅ BlockingQueue
   Cola thread-safe con bloqueo automático
   take()  → espera si está vacía (cuando siempre habrá datos)
   poll()  → retorna null si está vacía (cuando la cola puede vaciarse)
   put()   → agrega (espera si está llena)
```

---

### 📦 Bloque 4 — Colecciones thread-safe

```
✅ ConcurrentHashMap
   Cerrojos por segmento/nodo — múltiples hilos en paralelo
   Protege operaciones individuales
   Para secuencias completas → combinar con ReentrantLock

✅ CopyOnWriteArrayList
   Cada escritura crea copia interna
   Ideal para listas con muchas lecturas, pocas escrituras

✅ ConcurrentSkipListMap
   Como TreeMap pero thread-safe
   Mantiene orden natural de las claves — perfecto para reportes ordenados

❌ HashMap / ArrayList
   NO thread-safe — race condition garantizada
   Bajo alta concurrencia puede corromper su estructura interna
```

---

### 🏗️ Bloque 5 — Patrones que aplicaste

```
✅ Productor-Consumidor
   BlockingQueue como canal entre hilos
   Señal de veneno (-1 o objeto especial) para terminar la cadena

✅ Pipeline
   Datos fluyen entre fases a través de BlockingQueues
   Cada fase en su propio hilo
   FIFO garantiza orden de datos aunque los prints sean caóticos

✅ Inyección de dependencias
   Recursos compartidos creados en Main, pasados por constructor
   Cada clase una sola responsabilidad
   Main orquesta — no implementa lógica de negocio
```

---

## 💡 Las reglas de oro del módulo

```
1. "Nunca asumas orden entre hilos"
   El scheduler interrumpe cuando quiere — sin importar qué tan
   obvio parezca el timing

2. "Si puede pasar, eventualmente pasará en producción"
   Un bug de 0.01% de probabilidad es igual de peligroso —
   solo más difícil de detectar

3. "ConcurrentHashMap protege operaciones individuales
    ReentrantLock protege secuencias completas"
   Cuando necesitas ambas cosas, usa ambas herramientas

4. "unlock() y release() SIEMPRE en finally"
   Una excepción sin finally = cerrojo tomado = deadlock eterno

5. "take() cuando siempre habrá datos
    poll() cuando la cola puede vaciarse"

6. "synchronized y AtomicInteger no se mezclan"
   Son dos estrategias distintas para el mismo problema

7. "El output caótico en concurrencia es buena señal"
   Significa que el paralelismo es real — no estás serializando
   accidentalmente
```

---

## 🔭 Preview del Módulo 2

Lo que viene es concurrencia en el mundo real — Spring Boot y bases de datos:

```
Módulo 2 — Sincronizando Requisiciones
├── El mismo problema de race condition
│   pero ahora en una BD con múltiples usuarios
├── Optimistic Locking  → @Version en JPA
├── Pessimistic Locking → bloqueos a nivel de BD
└── Postman para simular usuarios simultáneos
```

Todo lo que aprendiste aquí — race conditions, sincronización, herramientas — lo vas a ver aparecer de nuevo pero a nivel de base de datos. La teoría es la misma, el contexto cambia.

---

## 🧠 Test final del módulo

**Pregunta 1**
¿Cuál es la diferencia entre RUNNABLE y WAITING en el ciclo de vida de un hilo?

**Pregunta 2**
¿Por qué `synchronized` en dos métodos distintos del mismo objeto impide que corran simultáneamente aunque toquen variables diferentes?

**Pregunta 3**
Tienes este código. ¿Qué problema tiene y cómo lo corriges?
```java
public class Inventario {
    private Map<String, Integer> stock = new HashMap<>();

    public boolean vender(String producto, int cantidad) {
        int stockActual = stock.getOrDefault(producto, 0);
        if (stockActual >= cantidad) {
            stock.put(producto, stockActual - cantidad);
            return true;
        }
        return false;
    }
}
```

**Pregunta 4**
¿Cuándo usarías `CyclicBarrier` en lugar de `CountDownLatch`?

**Pregunta 5**
Explica con tus palabras cómo funciona CAS (Compare And Swap) en `AtomicInteger`.

---

## 🎓 ¡Módulo 1 completado!

```
Sesión 1  ✅  Sesión 2  ✅  Sesión 3  ✅  Sesión 4  ✅
Sesión 5  ✅  Sesión 6  ✅  Sesión 7  ✅  Sesión 8  ✅
Sesión 9  ✅  Sesión 10 ✅  Sesión 11 ✅  Sesión 12 ✅
Sesión 13 ✅  Sesión 14 ✅

Módulo 1  ██████████████  14/14  100% 🎊
```

➡️ [Ir al Módulo 2](../modulo-2/README.md)
