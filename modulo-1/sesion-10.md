# Sesión 10 — Sincronizando hilos en Screenmatch

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 55 minutos |
| **Tipo** | 💻 Práctica |
| **Dificultad** | Intermedia |

---

## 🎯 Objetivo

Sesión integradora — aplicar todo lo aprendido en un flujo coherente de Screenmatch.

---

## 🗺️ El escenario completo

```
1. Consultar datos de 5 series en paralelo (threads)
2. Limitar a 2 consultas simultáneas (Semaphore)
3. Guardar resultados thread-safe (ConcurrentHashMap)
4. Esperar que todas terminen (join)
5. Generar reporte final con tiempo real
```

---

## 🏗️ Estructura del proyecto

```
screenmatch/
├── model/
│   └── Serie.java
├── service/
│   ├── ConsumoAPI.java
│   └── GestorDescargas.java  ← nueva
└── Main.java
```

---

## 📝 `Serie.java` — actualizada

```java
package com.aluracursos.screenmatch.model;

public class Serie {
    private String nombre;
    private String datos;
    private long tiempoDescarga;

    public Serie(String nombre) { this.nombre = nombre; }

    public String getNombre() { return nombre; }
    public String getDatos() { return datos; }
    public long getTiempoDescarga() { return tiempoDescarga; }
    public void setDatos(String datos) { this.datos = datos; }
    public void setTiempoDescarga(long tiempo) { this.tiempoDescarga = tiempo; }

    @Override
    public String toString() {
        return String.format("%-20s | %dms | %s",
                           nombre, tiempoDescarga, datos);
    }
}
```

---

## 📝 `GestorDescargas.java`

```java
package com.aluracursos.screenmatch.service;

import com.aluracursos.screenmatch.model.Serie;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.Semaphore;

public class GestorDescargas {

    private final Semaphore semaforo = new Semaphore(2); // máximo 2 simultáneas
    private final ConcurrentHashMap<String, Serie> resultados = new ConcurrentHashMap<>();
    private final ConsumoAPI api = new ConsumoAPI();

    public void descargar(Serie serie) {
        try {
            semaforo.acquire();
            System.out.println("[" + Thread.currentThread().getName()
                             + "] ▶ Iniciando: " + serie.getNombre());

            long inicio = System.currentTimeMillis();
            String datos = api.obtenerDatosSerie(serie.getNombre());
            long fin = System.currentTimeMillis();

            serie.setDatos(datos);
            serie.setTiempoDescarga(fin - inicio);
            resultados.put(serie.getNombre(), serie);

            System.out.println("[" + Thread.currentThread().getName()
                             + "] ✅ Completada: " + serie.getNombre());

        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            semaforo.release(); // SIEMPRE en finally
        }
    }

    public void mostrarReporte() {
        System.out.println("\n" + "=".repeat(60));
        System.out.println("REPORTE FINAL — SCREENMATCH");
        System.out.println("=".repeat(60));
        System.out.printf("%-20s | %-8s | %s%n", "Serie", "Tiempo", "Estado");
        System.out.println("-".repeat(60));
        resultados.forEach((nombre, serie) -> System.out.println(serie));
        System.out.println("=".repeat(60));
        System.out.println("Series descargadas: " + resultados.size());
    }
}
```

---

## 📝 `Main.java`

```java
package com.aluracursos.screenmatch;

import com.aluracursos.screenmatch.model.Serie;
import com.aluracursos.screenmatch.service.GestorDescargas;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        GestorDescargas gestor = new GestorDescargas();

        String[] nombres = {
            "Breaking Bad", "Dark", "Squid Game",
            "Stranger Things", "The Crown"
        };

        System.out.println("🎬 SCREENMATCH — Iniciando descargas paralelas");
        System.out.println("Máximo 2 descargas simultáneas\n");

        long inicio = System.currentTimeMillis();

        Thread[] hilos = new Thread[nombres.length];
        for (int i = 0; i < nombres.length; i++) {
            Serie serie = new Serie(nombres[i]);
            hilos[i] = new Thread(
                () -> gestor.descargar(serie),
                "hilo-" + nombres[i]
            );
            hilos[i].start();
        }

        for (Thread hilo : hilos) {
            hilo.join(10000);
            if (hilo.isAlive()) {
                System.out.println("⚠️ Timeout: " + hilo.getName());
                hilo.interrupt();
            }
        }

        long fin = System.currentTimeMillis();
        gestor.mostrarReporte();
        System.out.println("⏱️  Tiempo total real:       " + (fin - inicio) + "ms");
        System.out.println("⏱️  Tiempo secuencial est.:  ~" + (nombres.length * 2000) + "ms");
    }
}
```

---

## 🧪 Experimento — cambia el Semaphore

```java
new Semaphore(1) // completamente secuencial → ~10000ms
new Semaphore(2) // balance controlado       → ~4000-6000ms
new Semaphore(5) // completamente paralelo   → ~2000-3000ms
```

> 💡 En producción usarías `Semaphore` para controlar la carga sobre una API o base de datos externa.

---

## ✅ Checklist de la sesión

- [ ] Proyecto ejecutado y viste el semáforo limitando las descargas
- [ ] Observaste la diferencia de tiempos con distintos valores de Semaphore
- [ ] Entiendes por qué `semaforo.release()` va en `finally`
- [ ] El reporte final muestra todas las series correctamente

---

## 🧠 Mini-test

**Pregunta 1**
¿Por qué `semaforo.release()` debe ir en `finally` y no al final del `try`?

- A) Porque `finally` ejecuta más rápido que el final del `try`
- B) Porque si ocurre una excepción dentro del `try`, el permiso nunca se liberaría — bloqueando el semáforo para siempre
- C) Porque `Semaphore` lanza excepción si `release()` está en el `try`
- D) Es solo una convención, no tiene consecuencias reales

**Pregunta 2**
Con `Semaphore(2)` y 5 hilos, ¿cuántos hilos están en estado WAITING mientras 2 ejecutan?

- A) 0 — todos corren en paralelo de todas formas
- B) 2 — el semáforo bloquea exactamente la mitad
- C) 3 — los que no tienen permiso esperan en la cola del semáforo
- D) 5 — todos esperan turno aunque haya permisos disponibles

**Pregunta 3**
¿Qué ventaja tiene `ConcurrentHashMap` sobre `HashMap` con `synchronized`?

- A) `ConcurrentHashMap` es más fácil de escribir
- B) Permite que múltiples hilos escriban en segmentos distintos simultáneamente sin bloquearse entre sí
- C) `ConcurrentHashMap` no necesita cerrojos en absoluto
- D) Son equivalentes en performance para este caso

**Pregunta 4**
Si cambias `Semaphore(2)` a `Semaphore(1)`, ¿en qué se convierte el sistema?

- A) El sistema falla porque `Semaphore(1)` no está permitido
- B) Solo el primer hilo ejecuta, los demás se cancelan
- C) Equivale a un sistema completamente secuencial — un hilo a la vez
- D) Los 5 hilos corren en paralelo pero con más lentitud

---

➡️ [Siguiente sesión: Sistema de biblioteca](./sesion-11.md)
