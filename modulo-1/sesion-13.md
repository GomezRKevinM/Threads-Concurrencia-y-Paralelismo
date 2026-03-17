# Sesión 13 — Proyecto Final del Módulo 1

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 60 minutos |
| **Tipo** | 🏆 Proyecto integrador |
| **Dificultad** | Avanzada |

---

## 🎯 El reto

Este proyecto integra **todo lo del módulo** en un solo sistema coherente. Sin andamiaje — tú diseñas, tú construyes.

---

## 📋 El escenario — Sistema de Streaming Paralelo

> **StreamFast** es una plataforma de streaming que necesita procesar su catálogo completo. Al arrancar el sistema debe:
>
> 1. Descargar información de 8 series en paralelo desde una "API" externa
> 2. Máximo 3 descargas simultáneas — la API no soporta más
> 3. Cada descarga retorna: título, género, rating (1.0 - 5.0) y episodios (10-50)
> 4. Mientras se descargan, un hilo clasificador las organiza por género en tiempo real
> 5. Al terminar todas las descargas, generar un reporte con:
>    - Series organizadas por género
>    - La mejor serie por rating en cada género
>    - Promedio de episodios por género
>    - Tiempo total del proceso

---

## 📊 Catálogo de series

```java
String[] catalogo = {
    "Breaking Bad",      // Drama
    "Stranger Things",   // Ciencia Ficción
    "Dark",              // Ciencia Ficción
    "The Crown",         // Drama
    "Squid Game",        // Thriller
    "Black Mirror",      // Ciencia Ficción
    "Narcos",            // Drama
    "Money Heist"        // Thriller
};
```

---

## 🎯 Salida esperada

```
🚀 StreamFast — Iniciando descarga del catálogo
Máximo 3 descargas simultáneas

[Descarga] ▶ Breaking Bad
[Descarga] ▶ Stranger Things
[Descarga] ▶ Dark
[Clasificador] 📂 Breaking Bad → Drama
[Descarga] ✅ Breaking Bad (1823ms)
[Descarga] ▶ The Crown
...

════════════════════════════════════════════
        REPORTE FINAL — STREAMFAST
════════════════════════════════════════════

📂 DRAMA
   Breaking Bad     ⭐ 4.8 | 45 episodios
   The Crown        ⭐ 4.2 | 30 episodios
   Narcos           ⭐ 4.5 | 25 episodios
   🏆 Mejor: Breaking Bad (4.8)
   📊 Promedio episodios: 33.3

📂 CIENCIA FICCIÓN
   ...

════════════════════════════════════════════
⏱️  Tiempo total: 4832ms
════════════════════════════════════════════
```

---

## 🗺️ Requisitos técnicos

```
Clases mínimas:
├── Serie.java           → título, género, rating, episodios
├── ApiSimulada.java     → simula descarga con sleep 1-3s
│                          retorna Serie con datos aleatorios
├── GestorDescargas.java → controla el semáforo y los hilos
├── Clasificador.java    → hilo que consume series descargadas
│                          y las organiza por género
└── Main.java            → orquesta todo

Herramientas obligatorias:
├── Semaphore(3)         → límite de descargas simultáneas
├── BlockingQueue        → canal entre descargas y clasificador
├── ConcurrentHashMap    → catálogo organizado por género
├── CountDownLatch       → esperar que todas las descargas terminen
└── join()               → esperar al clasificador antes del reporte
```

---

## 💡 El flujo completo

```
Main
 │
 ├── lanza 8 hilos de descarga (máx 3 simultáneos con Semaphore)
 │         │
 │         └── cada descarga exitosa → mete Serie en BlockingQueue
 │
 ├── lanza 1 hilo Clasificador
 │         │
 │         └── consume BlockingQueue → organiza en ConcurrentHashMap
 │
 ├── CountDownLatch espera las 8 descargas
 │         │
 │         └── cuando terminen → mete señal de veneno en BlockingQueue
 │
 ├── join() espera al Clasificador
 │
 └── genera reporte final
```

---

## ⚠️ Restricciones

```
1. El clasificador debe ser un hilo separado — no clasifiques
   dentro del hilo de descarga
2. El reporte solo se genera cuando el clasificador termina
3. El Semaphore debe estar en GestorDescargas, no en Main
4. Sin race conditions en ninguna estructura compartida
```

---

## 🔑 Tips para el reporte

```java
// Mejor serie por rating:
Serie mejor = lista.stream()
    .max(Comparator.comparingDouble(Serie::getRating))
    .orElse(null);

// Promedio de episodios:
double promedio = lista.stream()
    .mapToInt(Serie::getEpisodios)
    .average()
    .orElse(0.0);
```

---

## 🧠 Mini-test

**Pregunta 1**
¿Por qué el `Semaphore` debe estar en `GestorDescargas` y no en `Main`?

**Pregunta 2**
¿Por qué se usa `join()` para esperar al clasificador en lugar de `CountDownLatch`?

**Pregunta 3**
¿Qué pasa si no metes la señal de veneno en la `BlockingQueue` después de que terminen las descargas?

**Pregunta 4**
¿Por qué `ConcurrentHashMap<String, CopyOnWriteArrayList<Serie>>` y no `HashMap<String, ArrayList<Serie>>`?

---

➡️ [Siguiente sesión: ¿Qué aprendimos?](./sesion-14.md)
