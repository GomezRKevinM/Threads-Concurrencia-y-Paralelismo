# Sesión 07 — Evitando el saldo negativo en la cuenta

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 55 minutos |
| **Tipo** | 💻 Práctica |
| **Dificultad** | Intermedia |

---

## 🎯 Objetivo

Resolver la race condition de 3 formas distintas y entender cuándo usar cada una.

---

## 🏦 El escenario: cuenta bancaria compartida

### `CuentaBancaria.java`

```java
package com.aluracursos.screenmatch.model;

public class CuentaBancaria {

    private String titular;
    private double saldo;

    public CuentaBancaria(String titular, double saldoInicial) {
        this.titular = titular;
        this.saldo = saldoInicial;
    }

    // ❌ Versión insegura — tiene race condition
    public void depositar(double monto) {
        double saldoActual = saldo;   // LEE
        saldoActual += monto;         // SUMA
        saldo = saldoActual;          // ESCRIBE
    }

    public void retirar(double monto) {
        if (saldo >= monto) {
            double saldoActual = saldo;
            saldoActual -= monto;
            saldo = saldoActual;
        }
    }

    public double getSaldo() { return saldo; }
}
```

### `Main.java` para reproducir el problema

```java
CuentaBancaria cuenta = new CuentaBancaria("Juan", 0);

// 1000 hilos depositan $1 cada uno → esperamos $1000 final
Thread[] hilos = new Thread[1000];
for (int i = 0; i < 1000; i++) {
    hilos[i] = new Thread(() -> cuenta.depositar(1.0));
    hilos[i].start();
}
for (Thread h : hilos) h.join();

System.out.println("Saldo esperado: $1000.0");
System.out.println("Saldo obtenido: $" + cuenta.getSaldo());
// Resultado: $978.0, $992.0, $961.0... dinero que desaparece 💀
```

---

## 🔒 Solución 1 — `synchronized` en método

```java
public synchronized void depositar(double monto) {
    double saldoActual = saldo;
    saldoActual += monto;
    saldo = saldoActual;
}

public synchronized void retirar(double monto) {
    if (saldo >= monto) {
        double saldoActual = saldo;
        saldoActual -= monto;
        saldo = saldoActual;
    }
}
```

**¿Cómo funciona internamente?**

```
Cada objeto Java tiene un "monitor" (cerrojo interno)

Hilo-1 llega a depositar() → toma el cerrojo 🔒
Hilo-2 llega a depositar() → encuentra el cerrojo tomado → ESPERA
Hilo-3 llega a depositar() → encuentra el cerrojo tomado → ESPERA

Hilo-1 termina depositar() → libera el cerrojo 🔓
Hilo-2 toma el cerrojo 🔒 → ejecuta → libera 🔓
```

> ⚠️ El cerrojo es **del objeto completo** — no de la variable. Aunque dos métodos toquen variables distintas, comparten el mismo cerrojo.

---

## 🔒 Solución 2 — `AtomicInteger`

```java
import java.util.concurrent.atomic.AtomicInteger;

// Cambia esto:
private int totalDescargados = 0;
totalDescargados++;

// Por esto:
private AtomicInteger totalDescargados = new AtomicInteger(0);
totalDescargados.incrementAndGet(); // atómica por diseño, sin cerrojo
```

**¿Por qué es más eficiente?**

```
synchronized → usa un cerrojo → los hilos hacen COLA → más lento
AtomicInteger → usa CAS (Compare And Swap) a nivel de CPU
               → no hay cerrojo → los hilos no esperan → más rápido
```

**¿Cómo funciona CAS?**
```
1. LEE el valor actual → guarda "lo que vi"
2. CALCULA el nuevo valor
3. Antes de escribir pregunta: ¿el valor sigue igual a "lo que vi"?
   SÍ → escribe ✅
   NO → alguien lo cambió → vuelve al paso 1 y reintenta 🔄

La pregunta + escritura son UNA instrucción indivisible de CPU.
```

---

## 🔒 Solución 3 — Bloque `synchronized`

```java
public void depositar(double monto) {

    // Código NO crítico — corre en paralelo
    System.out.println(Thread.currentThread().getName() + " depositando...");

    // Solo esta parte necesita protección
    synchronized (this) {
        double saldoActual = saldo;
        saldoActual += monto;
        saldo = saldoActual;
    }

    // Código NO crítico nuevamente
    System.out.println(Thread.currentThread().getName() + " listo.");
}
```

---

## 📊 Comparativa

| Herramienta | Simplicidad | Performance | Cuándo usar |
|---|---|---|---|
| `synchronized` método | ✅✅ | ⚠️ | Lógica compleja |
| `synchronized` bloque | ⚠️ | ✅ | Método mixto |
| `AtomicInteger` | ✅✅ | ✅✅ | Contadores simples |

---

## ✅ Checklist de la sesión

- [ ] Reprodujiste la race condition en `CuentaBancaria`
- [ ] Aplicaste `synchronized` y confirmaste `$1000.0` consistente
- [ ] Aplicaste `AtomicInteger` en `DescargaEpisodios` y viste 30,000 consistente
- [ ] Entiendes la diferencia entre `synchronized` en método vs bloque

---

## 🧠 Mini-test

**Pregunta 1**
¿Qué garantiza `synchronized` en un método?

- A) Que el método se ejecuta más rápido al tener un solo hilo
- B) Que solo un hilo puede ejecutar ese método a la vez en el mismo objeto
- C) Que el método nunca puede ser interrumpido por el scheduler
- D) Que todos los hilos ejecutan el método en el orden en que llegaron

**Pregunta 2**
¿Por qué `AtomicInteger` es más eficiente que `synchronized` para contadores?

- A) Porque `AtomicInteger` usa múltiples hilos internamente para ser más rápido
- B) Porque evita el cerrojo usando instrucciones atómicas de la CPU, sin hacer esperar a los hilos
- C) Porque `AtomicInteger` solo funciona con números, lo que lo hace más simple
- D) Porque `synchronized` tiene un bug conocido con números enteros

**Pregunta 3**
Tienes un método que hace 3 cosas: validar un dato, actualizar el saldo, y enviar un log. Solo actualizar el saldo es crítico. ¿Qué usarías?

- A) `synchronized` en el método completo — es más seguro
- B) `synchronized` en bloque — solo protege la sección crítica
- C) `AtomicInteger` — siempre es la mejor opción
- D) No necesita sincronización si las otras operaciones no tocan el saldo

**Pregunta 4**
¿Qué pasaría si dos métodos `synchronized` del mismo objeto intentan ejecutarse simultáneamente desde dos hilos distintos?

- A) Ambos corren en paralelo porque son métodos diferentes
- B) El segundo hilo espera — comparten el mismo cerrojo del objeto
- C) Lanza una excepción de concurrencia
- D) El SO decide cuál ejecutar y cancela el otro

---

➡️ [Siguiente sesión: Herramientas de sincronización](./sesion-08.md)
