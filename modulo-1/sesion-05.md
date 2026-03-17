# Sesión 05 — ¿Cómo funcionan los hilos del Sistema Operativo?

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 40 minutos |
| **Tipo** | 📖 Teoría |
| **Dificultad** | Básica |

---

## 🖥️ Proceso vs Hilo — la versión completa

```
┌─────────────────────────────────────────┐
│            PROCESO (tu app Java)        │
│                                         │
│  Memoria propia  │  Recursos del SO     │
│  ─────────────────────────────────      │
│                                         │
│  ┌──────────┐  ┌──────────┐  ┌───────┐ │
│  │ Hilo 1   │  │ Hilo 2   │  │Hilo 3 │ │
│  │(Breaking)│  │  (Dark)  │  │(Squid)│ │
│  └──────────┘  └──────────┘  └───────┘ │
│                                         │
│  Los 3 comparten la misma memoria       │
│  → por eso totalDescargados             │
│    es visible para todos                │
└─────────────────────────────────────────┘
```

**Clave:** los hilos comparten memoria dentro del mismo proceso. Los procesos están aislados entre sí. Eso es lo que hace a los hilos más livianos pero también más peligrosos.

---

## ⚙️ El Scheduler: el árbitro invisible

Cuando haces `hilo.start()`, Java le dice al SO *"necesito un hilo nuevo"*. El SO lo crea y lo mete en una cola. Pero tu CPU tiene núcleos limitados — una CPU de 8 núcleos solo puede ejecutar 8 hilos literalmente al mismo tiempo.

El componente que decide quién corre y cuándo se llama **scheduler**:

```
Cola de hilos listos para ejecutar:
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│ hilo-A   │ hilo-B   │ hilo-C   │ hilo-D   │ hilo-E   │
└──────────┴──────────┴──────────┴──────────┴──────────┘

CPU (2 núcleos disponibles):
├── Núcleo 1: ejecuta hilo-A por ~10ms → lo pausa → toma hilo-C
└── Núcleo 2: ejecuta hilo-B por ~10ms → lo pausa → toma hilo-D

Ese intervalo de ~10ms se llama "quantum" o "time slice"
```

El scheduler pausa y reanuda hilos miles de veces por segundo. Tú como programador **nunca sabes** en qué momento exacto ocurre ese cambio.

---

## 🔄 Los estados de un hilo

```
                    hilo.start()
NEW ──────────────────────────► RUNNABLE
(creado pero                   (listo, esperando
no iniciado)                    que el scheduler
                                le dé CPU)
                                    │
                    ┌───────────────┘
                    ▼
                 RUNNING ──── hilo.sleep() ──► TIMED_WAITING
                 (tiene         obj.wait()  ──► WAITING
                  la CPU)       hilo.join() ──► WAITING
                    │
                    └──── tarea terminada ──► TERMINATED
```

> ⚠️ **Importante:** cuando el scheduler interrumpe un hilo por quantum agotado, el hilo pasa a **RUNNABLE** — no a WAITING. WAITING es cuando el hilo llama explícitamente a `wait()`, `join()` o `sleep()`.

---

## 🧠 ¿Paralelismo real o simulado?

```
CPU de 1 núcleo → CONCURRENCIA SIMULADA
El scheduler alterna tan rápido que parece paralelo
pero en realidad solo corre 1 hilo a la vez

CPU multinúcleo → PARALELISMO REAL
Varios hilos corren LITERALMENTE al mismo tiempo
→ aquí las race conditions son más frecuentes
  porque la simultaneidad es real, no simulada
```

---

## 🌐 ¿Y la JVM qué papel juega?

```
Tu código Java
      ↓
    JVM  ←── traduce Thread de Java a Thread del SO
      ↓
Sistema Operativo
      ↓
Hardware (CPU, núcleos)
```

Cuando haces `new Thread()` en Java, la JVM crea un **hilo nativo del SO** — un hilo real. El scheduler del SO tiene control total sobre cuándo corre tu hilo.

---

## 💡 Por qué esto explica tu race condition

```
Núcleo 1 ejecutando hilo-Breaking:
  1. LEE totalDescargados = 5432
  2. scheduler interrumpe ← justo aquí

Núcleo 2 ejecutando hilo-Dark (simultáneo):
  1. LEE totalDescargados = 5432  ← mismo valor
  2. SUMA → 5433
  3. ESCRIBE totalDescargados = 5433

Núcleo 1 reanuda hilo-Breaking:
  3. SUMA → 5433  ← calculó sobre el valor viejo
  4. ESCRIBE totalDescargados = 5433  ← sobreescribe

Resultado: dos incrementos, pero totalDescargados solo subió 1
→ perdiste un episodio
```

---

## 🔑 Regla de oro

> **"Nunca asumas que dos líneas de código consecutivas se ejecutan sin interrupción."**
>
> El scheduler no lee tu código. No sabe que esas 3 operaciones deberían ser atómicas. Interrumpe cuando quiere.

---

## ✅ Checklist de la sesión

- [ ] Entiendes que los hilos comparten memoria dentro del mismo proceso
- [ ] Sabes qué es el scheduler y por qué hace el orden impredecible
- [ ] Conoces los estados de un hilo (NEW, RUNNABLE, RUNNING, WAITING, TERMINATED)
- [ ] Puedes explicar por qué una CPU multinúcleo hace la race condition más visible

---

## 🧠 Mini-test

**Pregunta 1**
¿Cuál es la diferencia fundamental entre un proceso y un hilo en términos de memoria?

**Pregunta 2**
Cuando el scheduler interrumpe un hilo por quantum agotado, ¿en qué estado queda ese hilo: RUNNABLE o WAITING? ¿Por qué?

**Pregunta 3**
¿Por qué en una CPU multinúcleo las race conditions son más frecuentes que en una CPU de 1 núcleo?

**Pregunta 4**
¿Qué papel juega la JVM entre tu código Java y el hardware?

---

➡️ [Siguiente sesión: Creando hilos de formas diferentes](./sesion-06.md)
