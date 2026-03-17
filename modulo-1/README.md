# ☕ Módulo 1 — Threads en Java

## 📋 Información general

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 de 5 |
| **Tema** | Threads en Java puro |
| **Sesiones** | 14 sesiones |
| **Duración total** | ~8 horas |
| **Tiempo por sesión** | 35 - 60 minutos |
| **Días sugeridos** | 7 días (2 sesiones/día) |
| **Estado** | ✅ Completo |
| **Tecnología** | Java puro — sin frameworks |

## 🗺️ Sesiones del módulo

| # | Sesión | Duración | Tipo | Estado |
|---|--------|----------|------|--------|
| 01 | [Presentación del curso](./sesion-01.md) | 35 min | 📖 Teoría | ✅ |
| 02 | [Preparando el ambiente](./sesion-02.md) | 45 min | 💻 Práctica | ✅ |
| 03 | [Tareas síncronas y asíncronas](./sesion-03.md) | 50 min | 💻 Práctica | ✅ |
| 04 | [Simulando tareas paralelas](./sesion-04.md) | 50 min | 💻 Práctica | ✅ |
| 05 | [¿Cómo funcionan los hilos del SO?](./sesion-05.md) | 40 min | 📖 Teoría | ✅ |
| 06 | [Creando hilos de formas diferentes](./sesion-06.md) | 45 min | 💻 Práctica | ✅ |
| 07 | [Evitando el saldo negativo](./sesion-07.md) | 55 min | 💻 Práctica | ✅ |
| 08 | [Herramientas de sincronización](./sesion-08.md) | 40 min | 📖 Teoría | ✅ |
| 09 | [Esperando que los threads finalicen](./sesion-09.md) | 50 min | 💻 Práctica | ✅ |
| 10 | [Sincronizando hilos en Screenmatch](./sesion-10.md) | 55 min | 💻 Práctica | ✅ |
| 11 | [Sistema de biblioteca](./sesion-11.md) | 60 min | 🛠️ Manos a la obra | ✅ |
| 12 | [Lista de ejercicios](./sesion-12.md) | 50 min | 🏋️ Ejercicios | ✅ |
| 13 | [Proyecto final](./sesion-13.md) | 60 min | 🏆 Proyecto | ✅ |
| 14 | [¿Qué aprendimos?](./sesion-14.md) | 35 min | 🎓 Cierre | ✅ |

## 🧠 Lo que aprenderás

### Bloque 1 — Fundamentos
- Diferencia entre proceso e hilo
- Estados de un hilo: NEW → RUNNABLE → WAITING → TERMINATED
- Cómo funciona el Scheduler del SO
- 3 formas de crear hilos en Java

### Bloque 2 — El problema central
- Race conditions y por qué son peligrosas
- Por qué `totalDescargados++` no es atómico
- Cómo detectar bugs de concurrencia

### Bloque 3 — Herramientas de sincronización
- `synchronized` — método y bloque
- `ReentrantLock` — con tryLock y timeout
- `AtomicInteger` / `AtomicLong` — CAS a nivel de CPU
- `Semaphore` — control de acceso concurrente
- `CountDownLatch` — esperar N hilos
- `CyclicBarrier` — punto de encuentro reutilizable
- `BlockingQueue` — productor-consumidor

### Bloque 4 — Colecciones thread-safe
- `ConcurrentHashMap`
- `CopyOnWriteArrayList`
- `ConcurrentSkipListMap`

### Bloque 5 — Patrones aplicados
- Productor-Consumidor
- Pipeline de datos
- Inyección de dependencias con hilos

## 📝 Notas del módulo

> Este módulo trabaja con **Java puro** — sin Spring Boot ni frameworks.
> El proyecto base es **Screenmatch**, una app de catálogo de series.
> A partir del Módulo 2 se incorpora Spring Boot y JPA.

## 🔗 Siguiente módulo

➡️ [Módulo 2 — Sincronizando Requisiciones](../modulo-2/README.md)
