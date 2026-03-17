# Sesión 12 — Lista de ejercicios: Explorando hilos

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 50 minutos |
| **Tipo** | 🏋️ Ejercicios prácticos |
| **Dificultad** | Progresiva |

---

## 📋 Reglas

```
✅ Puedes pedir pistas sin penalización
✅ Puedes resolver en el orden que quieras
⚠️ Cada ejercicio debe compilar y ejecutarse
⚠️ Sin race conditions en datos compartidos
```

## 📊 Puntuación

```
Ejercicio 1  → 1 punto    (Fácil)
Ejercicio 2  → 1.5 puntos (Fácil/Medio)
Ejercicio 3  → 2 puntos   (Medio)
Ejercicio 4  → 2.5 puntos (Medio/Difícil)
Ejercicio 5  → 3 puntos   (Difícil)
──────────────────────────
Total        → 10 puntos
```

---

## 🏋️ Ejercicio 1 — Carrera de hilos

**Escenario:**
> 5 corredores compiten en una carrera. Cada corredor tarda un tiempo aleatorio entre 1 y 3 segundos. El sistema debe anunciar el podio en orden de llegada.

**Requisitos:**
```
1. Cada corredor corre en su propio hilo
2. El orden de llegada es el orden real en que terminan
3. Registra los 3 primeros en llegar (podio)
4. Muestra el podio cuando todos terminen
```

**Pista:**
> ¿Qué estructura thread-safe mantiene el orden de inserción y permite múltiples hilos escribiendo?

---

## 🏋️ Ejercicio 2 — Productor-Consumidor

**Escenario:**
> Un productor genera números del 1 al 10 con un delay de 500ms. Un consumidor los lee y calcula el acumulado. Ambos en hilos separados.

**Requisitos:**
```
1. Productor y consumidor en hilos distintos
2. El consumidor no puede leer un número que aún no fue producido
3. Cuando el productor termina, el consumidor termina también
4. Muestra cada número consumido y el acumulado en ese momento
```

**Pista:**
> Necesitas una estructura que bloquee al consumidor si está vacía. ¿Cuál viste en la Sesión 8?

---

## 🏋️ Ejercicio 3 — Cajeros automáticos

**Escenario:**
> Un banco tiene 1 cuenta con $10,000. 3 cajeros procesan retiros simultáneamente. Cada cajero intenta hacer 5 retiros de $500. El saldo nunca puede ser negativo.

**Requisitos:**
```
1. 3 hilos (cajeros) acceden a la misma cuenta
2. Cada cajero intenta 5 retiros de $500
3. Si no hay saldo suficiente → registra intento fallido
4. Saldo final nunca negativo
5. Reporte: retiros exitosos, fallidos, saldo final
```

**Pista:**
> Los retiros tienen 2 pasos: verificar y retirar. ¿Qué herramienta protege secuencias completas?

---

## 🏋️ Ejercicio 4 — Pipeline de datos

**Escenario:**
> Un sistema procesa registros en 3 fases encadenadas, cada una en su propio hilo.

```
[Hilo Lector] → genera 8 números aleatorios
      ↓
[Hilo Transformador] → multiplica cada número por 2
      ↓
[Hilo Escritor] → imprime el resultado final
```

**Requisitos:**
```
1. Cada fase en su propio hilo
2. Los datos fluyen entre fases sin race conditions
3. Cada fase espera datos de la anterior automáticamente
4. Cuando el lector termina, la cadena completa termina
```

**Pista:**
> Necesitas 2 colas entre fases. ¿Qué estructura bloquea automáticamente si está vacía? ¿Cómo le dices al Transformador que el Lector terminó?

---

## 🏋️ Ejercicio 5 — Sistema de turnos

**Escenario:**
> Una clínica atiende pacientes por turno. Hay 3 consultorios que atienden simultáneamente. Los pacientes llegan en orden pero son atendidos por el primer consultorio disponible.

**Requisitos:**
```
1. 10 pacientes llegan en orden (Paciente-1 primero)
2. 3 consultorios atienden simultáneamente
3. Cada atención tarda 1-3 segundos
4. Registra qué consultorio atendió a cada paciente
5. Reporte final ordenado por número de paciente
```

**Restricción extra:**
> Los pacientes deben ser atendidos en orden de llegada — Paciente-1 antes que Paciente-2.

**Pistas:**
> - Los pacientes esperan en una cola FIFO
> - Los consultorios consumen de esa cola
> - Usa `poll()` para que el consultorio termine cuando la cola se vacíe
> - Para el reporte ordenado: ¿qué estructura conoces que sea thread-safe Y mantenga orden?

---

## 🔑 Conceptos que necesitas por ejercicio

| Ejercicio | Herramientas clave |
|-----------|-------------------|
| 1 — Carrera | `Thread`, `ConcurrentHashMap`, `CountDownLatch`, streams |
| 2 — Productor-Consumidor | `BlockingQueue`, señal de veneno (-1) |
| 3 — Cajeros | `synchronized` o `ReentrantLock`, `AtomicInteger` |
| 4 — Pipeline | `BlockingQueue` x2, `Runnable`, señal de veneno |
| 5 — Turnos | `BlockingQueue`, `ConcurrentSkipListMap`, `CountDownLatch` |

---

## 💡 Tips generales

```
1. Antes de escribir código → escribe el flujo en pseudocódigo
2. Identifica qué datos son compartidos entre hilos
3. Protege SOLO lo que necesita protección
4. Usa poll() cuando la cola puede vaciarse y el hilo debe terminar
5. Usa take() cuando siempre habrá más datos
6. El output "caótico" en concurrencia es una buena señal —
   significa que el paralelismo es real
```

---

➡️ [Siguiente sesión: Proyecto final](./sesion-13.md)
