# Sesión 09 — Esperando que los threads finalicen

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 50 minutos |
| **Tipo** | 💻 Práctica |
| **Dificultad** | Intermedia |

---

## 🎯 Objetivo

Resolver el misterio de la Sesión 3 — el tiempo total que marcaba 3ms — usando `join()` correctamente.

---

## 🔒 `join()` — el hilo principal se sienta a esperar

```java
ConsumoAPI api = new ConsumoAPI();
String[] series = {"Breaking Bad", "Dark", "Squid Game"};

long inicio = System.currentTimeMillis();

Thread[] hilos = new Thread[series.length];

for (int i = 0; i < series.length; i++) {
    String serie = series[i];
    hilos[i] = new Thread(() -> {
        String resultado = api.obtenerDatosSerie(serie);
        System.out.println(resultado);
    }, "hilo-" + serie);
    hilos[i].start();
}

// Esperar a TODOS antes de medir el tiempo
for (Thread hilo : hilos) {
    hilo.join(); // hilo principal se bloquea hasta que cada hilo termine
}

long fin = System.currentTimeMillis();
System.out.println("\nTiempo total real: " + (fin - inicio) + "ms");
// Ahora sí: ~2100ms (el más lento), no ~6000ms (la suma)
```

---

## 🔬 ¿Qué hace `join()` internamente?

```
Hilo principal ejecutando:
    hilos[0].join() ──► entra en estado WAITING
                        el hilo principal se duerme

Hilo-Breaking termina ──► notifica al hilo principal
                          hilo principal pasa a RUNNABLE

    hilos[1].join() ──► entra en estado WAITING nuevamente
                        espera a hilo-Dark...

    hilos[2].join() ──► espera a hilo-Squid...

Todos terminaron ──► hilo principal continúa ✅
```

---

## ⏱️ Variantes de `join()`

```java
// Espera indefinidamente hasta que el hilo termine
hilo.join();

// Espera máximo 3 segundos
hilo.join(3000);

// Con timeout — patrón de producción
hilo.join(5000);
if (hilo.isAlive()) {
    System.out.println("Timeout: el hilo no terminó a tiempo");
    hilo.interrupt();
}
```

---

## 🔴 `interrupt()` — pedirle a un hilo que pare

```java
Thread hilo = new Thread(() -> {
    try {
        System.out.println("Trabajando...");
        Thread.sleep(10000);
        System.out.println("Terminé"); // nunca llega aquí
    } catch (InterruptedException e) {
        System.out.println("Me interrumpieron, limpiando y saliendo...");
        Thread.currentThread().interrupt(); // restaura la bandera
    }
});

hilo.start();
Thread.sleep(2000);
hilo.interrupt(); // interrumpe el sleep del hilo
```

> ⚠️ `interrupt()` no mata el hilo a la fuerza. **Lanza una señal** que el hilo puede manejar. Por eso siempre captura `InterruptedException` y llama `Thread.currentThread().interrupt()` para restaurar la bandera.

---

## 🔄 `interrupt()` en dos escenarios

```
ESCENARIO 1 — hilo está en sleep/wait/join:
hilo.interrupt()
  → lanza InterruptedException DIRECTAMENTE en el hilo
  → la bandera interna NO se activa (se consume en la excepción)
  → el hilo entra al catch → decide qué hacer
  → Thread.currentThread().interrupt() → reactiva la bandera manualmente

ESCENARIO 2 — hilo está ejecutando código normal:
hilo.interrupt()
  → activa la bandera interna isInterrupted() = true
  → el hilo SIGUE ejecutando normalmente
  → hasta que tú consultes: if (Thread.currentThread().isInterrupted())
```

---

## 🔄 `sleep()` vs `join()` vs `wait()`

```
Thread.sleep(ms)  → pausa este hilo por X tiempo
                    no libera cerrojos que tenga tomados

hilo.join()       → espera a que ese hilo específico termine
                    el hilo actual se duerme hasta recibir notificación

obj.wait()        → espera hasta que alguien notifique en este objeto
                    libera el cerrojo del objeto mientras espera
                    requiere estar dentro de synchronized
```

---

## 🛠️ Practica — mide el tiempo correctamente

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        ConsumoAPI api = new ConsumoAPI();
        String[] series = {"Breaking Bad", "Dark", "Squid Game",
                          "Stranger Things", "The Crown"};

        System.out.println("=== INICIO DE DESCARGAS ===\n");
        long inicio = System.currentTimeMillis();

        Thread[] hilos = new Thread[series.length];

        for (int i = 0; i < series.length; i++) {
            String serie = series[i];
            hilos[i] = new Thread(() -> {
                String resultado = api.obtenerDatosSerie(serie);
                System.out.println(resultado);
            }, "hilo-" + serie);
            hilos[i].start();
        }

        // Esperar con timeout de 5 segundos por hilo
        for (Thread hilo : hilos) {
            hilo.join(5000);
            if (hilo.isAlive()) {
                System.out.println("⚠️ Timeout: " + hilo.getName());
                hilo.interrupt();
            }
        }

        long fin = System.currentTimeMillis();
        System.out.println("\n=== RESUMEN ===");
        System.out.println("Tiempo paralelo:            " + (fin - inicio) + "ms");
        System.out.println("Tiempo estimado secuencial: ~" + (series.length * 2000) + "ms");
    }
}
```

---

## ✅ Checklist de la sesión

- [ ] Ejecutaste el código con `join()` y viste el tiempo correcto
- [ ] Entiendes por qué `join()` pone al hilo principal en WAITING
- [ ] Probaste `join(timeout)` con `isAlive()`
- [ ] Entiendes la diferencia entre `sleep()`, `join()` y `wait()`

---

## 🧠 Mini-test

**Pregunta 1**
¿En qué estado queda el hilo principal cuando llama `hilo.join()`?

- A) TERMINATED — el hilo principal muere mientras espera
- B) RUNNABLE — sigue listo pero cede su turno voluntariamente
- C) WAITING — se duerme hasta que el hilo objetivo termine
- D) NEW — vuelve al estado inicial para reiniciarse

**Pregunta 2**
¿Qué diferencia hay entre `Thread.sleep(1000)` y `hilo.join(1000)`?

- A) Son idénticos — ambos pausan el hilo actual 1 segundo
- B) `sleep(1000)` pausa exactamente 1 segundo; `join(1000)` espera al hilo máximo 1 segundo, pero puede continuar antes si el hilo termina primero
- C) `join(1000)` es más preciso que `sleep(1000)`
- D) `sleep()` es para hilos, `join()` es solo para el hilo principal

**Pregunta 3**
Llamas `hilo.interrupt()` mientras el hilo está en `Thread.sleep()`. ¿Qué ocurre?

- A) El hilo muere inmediatamente sin ejecutar más código
- B) El hilo ignora la señal y sigue durmiendo
- C) Se lanza `InterruptedException` en el hilo, que puede manejarlo en el catch y terminar limpiamente
- D) El hilo termina el sleep inmediatamente y continúa normalmente

**Pregunta 4**
¿Por qué es importante llamar `Thread.currentThread().interrupt()` dentro del `catch(InterruptedException)`?

- A) Para relanzar la excepción al hilo padre
- B) Para restaurar la bandera de interrupción — al capturar la excepción se limpia automáticamente y código superior puede necesitar saber que hubo interrupción
- C) Porque sin esa línea el hilo no termina
- D) Es solo una convención, no tiene efecto real

---

➡️ [Siguiente sesión: Sincronizando hilos en Screenmatch](./sesion-10.md)
