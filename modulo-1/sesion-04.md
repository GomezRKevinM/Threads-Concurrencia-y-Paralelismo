# Sesión 04 — Simulando tareas paralelas

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 50 minutos |
| **Tipo** | 💻 Práctica |
| **Dificultad** | Básica |

---

## 🎯 Objetivo

Hoy vas a ver con tus propios ojos el problema más peligroso de la concurrencia: la **race condition**. Los hilos van a compartir datos y los van a corromper silenciosamente.

---

## 🏗️ El escenario: descarga paralela con contador compartido

Crea `DescargaEpisodios.java` en el paquete `service`:

```java
package com.aluracursos.screenmatch.service;

public class DescargaEpisodios {

    private int totalDescargados = 0; // ← dato COMPARTIDO entre hilos

    public void descargar(String serie, int episodios) {
        System.out.println("[" + Thread.currentThread().getName()
                         + "] Iniciando descarga de " + serie);

        for (int i = 1; i <= episodios; i++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            totalDescargados++; // ← AQUÍ está el problema
            System.out.println("[" + Thread.currentThread().getName()
                             + "] Episodio " + i + " de " + serie
                             + " | Total acumulado: " + totalDescargados);
        }
    }

    public int getTotalDescargados() {
        return totalDescargados;
    }
}
```

Actualiza `Main.java`:

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {

        DescargaEpisodios descarga = new DescargaEpisodios(); // UNA sola instancia compartida

        Thread hiloBreaking = new Thread(
            () -> descarga.descargar("Breaking Bad", 5), "hilo-Breaking"
        );
        Thread hiloDark = new Thread(
            () -> descarga.descargar("Dark", 5), "hilo-Dark"
        );
        Thread hiloSquid = new Thread(
            () -> descarga.descargar("Squid Game", 5), "hilo-Squid"
        );

        hiloBreaking.start();
        hiloDark.start();
        hiloSquid.start();

        hiloBreaking.join();
        hiloDark.join();
        hiloSquid.join();

        System.out.println("\n=== RESULTADO FINAL ===");
        System.out.println("Total esperado:  15 episodios");
        System.out.println("Total obtenido:  " + descarga.getTotalDescargados());
    }
}
```

---

## 🧪 Experimento: aumenta la escala

Cambia a 10,000 episodios y sin `Thread.sleep`:

```java
public void descargar(String serie, int episodios) {
    for (int i = 1; i <= episodios; i++) {
        totalDescargados++;
    }
}
```

```java
// En Main.java
descarga.descargar("Breaking Bad", 10000)  // 30,000 total esperado
descarga.descargar("Dark", 10000)
descarga.descargar("Squid Game", 10000)
```

Ejecútalo 5 veces seguidas y anota los resultados. Verás algo como:

```
Total esperado:  30000
Total obtenido:  25027  ← ejecución 1
Total obtenido:  28441  ← ejecución 2
Total obtenido:  30000  ← ejecución 3 (¡tuvo suerte!)
Total obtenido:  24819  ← ejecución 4
```

---

## 🔬 ¿Por qué se pierden episodios?

La línea `totalDescargados++` parece simple, pero son **3 pasos internos**:

```
// Lo que tú escribes:
totalDescargados++;

// Lo que realmente ejecuta la CPU:
1. LEE   → carga el valor actual en un registro temporal
2. SUMA  → incrementa ese registro en 1
3. ESCRIBE → guarda el nuevo valor en memoria
```

Cuando dos hilos ejecutan esto simultáneamente:

```
t=100ms  hilo-Breaking LEE totalDescargados = 5
t=100ms  hilo-Dark     LEE totalDescargados = 5  ← leyó el mismo valor
t=101ms  hilo-Breaking ESCRIBE 6
t=101ms  hilo-Dark     ESCRIBE 6  ← sobreescribe, perdimos un incremento
```

Ambos leyeron 5, ambos escribieron 6. El resultado debería ser 7 pero quedó en 6. **Se perdió un episodio.**

---

## ⚠️ La lección más importante

> Un bug que solo falla a veces es el peor tipo de bug — en producción puede pasar desapercibido durante meses.

Si no supieras que el total debería ser 30,000, **jamás te darías cuenta** de que algo está mal. El programa corrió sin errores, sin excepciones, con total confianza.

---

## 💡 Preview de la solución

```java
// Opción 1: synchronized (la clásica)
synchronized (this) {
    totalDescargados++;
}

// Opción 2: AtomicInteger (más moderna y eficiente)
private AtomicInteger totalDescargados = new AtomicInteger(0);
totalDescargados.incrementAndGet();
```

Lo implementarás en la Sesión 7.

---

## ✅ Checklist de la sesión

- [ ] `DescargaEpisodios.java` creado y ejecutado
- [ ] Observaste resultados inconsistentes en múltiples ejecuciones
- [ ] Ejecutaste la versión con 10,000 episodios y anotaste los resultados
- [ ] Entiendes por qué `totalDescargados++` no es atómico

---

## 🧠 Mini-test

**Pregunta 1**
¿Por qué `totalDescargados++` no es thread-safe si parece una sola operación?

**Pregunta 2**
¿Por qué a veces el resultado sí da 30,000 exacto? ¿Qué tendría que pasar en el timing de los hilos para que sea correcto por accidente?

**Pregunta 3**
¿Por qué los bugs de race condition son especialmente peligrosos en producción?

**Pregunta 4**
Si tuvieras una CPU de 1 solo núcleo con 1 hilo lógico, ¿seguiría ocurriendo la race condition? ¿Por qué?

---

➡️ [Siguiente sesión: ¿Cómo funcionan los hilos del SO?](./sesion-05.md)
