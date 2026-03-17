# Sesión 03 — Tareas síncronas y asíncronas

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 50 minutos |
| **Tipo** | 💻 Práctica |
| **Dificultad** | Básica |

---

## 🔄 Síncrono vs Asíncrono

**Síncrono** significa *"espera aquí hasta que termine"*. Tu hilo principal se queda bloqueado.

**Asíncrono** significa *"arranca esto y sigue con tu vida"*. El hilo principal lanza la tarea y continúa.

```
// SÍNCRONO — como llamar por teléfono y quedarte en espera
☎️  "Hola, quiero info de Breaking Bad"
⏳  ... esperando ... esperando ...
✅  "Aquí está la info" → recién ahora llamas para Dark

// ASÍNCRONO — como enviar 3 WhatsApps y esperar las respuestas
📱  Envías mensaje a Breaking Bad
📱  Envías mensaje a Dark       ← sin esperar respuesta del anterior
📱  Envías mensaje a Squid Game ← sin esperar nada
✅  Las respuestas llegan cuando llegan, en cualquier orden
```

---

## 👀 El problema en tu código actual

```java
// Flujo interno del for síncrono:
// t=0ms    → empieza a buscar "Breaking Bad"
// t=1823ms → termina "Breaking Bad", AHORA empieza "Dark"
// t=3970ms → termina "Dark", AHORA empieza "Squid Game"
// t=5175ms → termina todo

for (String serie : series) {
    String resultado = api.obtenerDatosSerie(serie); // ← hilo BLOQUEADO aquí
    System.out.println(resultado);
}
```

---

## 🧵 Tu primer Thread

Modifica tu `Main.java`:

```java
public class Main {
    public static void main(String[] args) {
        ConsumoAPI api = new ConsumoAPI();
        String[] series = {"Breaking Bad", "Dark", "Squid Game"};

        long inicio = System.currentTimeMillis();

        for (String serie : series) {
            Thread hilo = new Thread(() -> {
                String resultado = api.obtenerDatosSerie(serie);
                System.out.println(resultado);
            });
            hilo.start(); // ← lanza el hilo y el for continúa SIN esperar
        }

        long fin = System.currentTimeMillis();
        System.out.println("Tiempo total: " + (fin - inicio) + "ms");
    }
}
```

---

## ▶️ Ejecútalo y observa algo inesperado

```
Tiempo total: 3ms   ← ¿¿¿esto está mal???
Datos de [Squid Game] obtenidos en 1205ms
Datos de [Breaking Bad] obtenidos en 1823ms
Datos de [Dark] obtenidos en 2147ms
```

**1. El tiempo total dice 3ms** — el hilo principal terminó casi de inmediato porque lanzó los 3 hilos y siguió de largo sin esperarlos.

**2. El orden cambió** — Squid Game llegó primero aunque se lanzó de último.

---

## 🔬 ¿Qué pasó con el tiempo?

```
Hilo principal:
t=0ms  ──► lanza hilo-Breaking-Bad
t=1ms  ──► lanza hilo-Dark
t=2ms  ──► lanza hilo-Squid-Game
t=3ms  ──► imprime "Tiempo total: 3ms" y MUERE
           │
           ├── hilo-Squid-Game ──────────────► imprime a t=1205ms
           ├── hilo-Breaking-Bad ────────────► imprime a t=1823ms
           └── hilo-Dark ───────────────────► imprime a t=2147ms
```

El hilo principal midió su propio tiempo de vida, no el del trabajo real.

---

## 🧪 Experimento — agrega nombres a los hilos

```java
for (String serie : series) {
    Thread hilo = new Thread(() -> {
        System.out.println("[" + Thread.currentThread().getName() + "] iniciando...");
        String resultado = api.obtenerDatosSerie(serie);
        System.out.println("[" + Thread.currentThread().getName() + "] " + resultado);
    }, "hilo-" + serie.replace(" ", "-"));

    hilo.start();
}
```

---

## ⚠️ `.start()` vs `.run()`

> **Error clásico:** llamar `hilo.run()` en vez de `hilo.start()`. Con `.run()` el código se ejecuta en el hilo actual, en secuencia, sin ningún paralelismo. `.start()` es lo que realmente crea un nuevo hilo en el SO.

---

## ✅ Checklist de la sesión

- [ ] `Main.java` modificado con threads y ejecutado correctamente
- [ ] Observaste que el tiempo total se imprime antes que los resultados
- [ ] Entiendes por qué `.start()` y no `.run()`
- [ ] Ejecutaste la versión con nombres de hilos

---

## 🧠 Mini-test

**Pregunta 1**
¿Por qué el tiempo total imprime ~3ms aunque los datos tardan ~2 segundos en llegar?

**Pregunta 2**
¿Qué diferencia hay entre `hilo.start()` y `hilo.run()`?

**Pregunta 3**
Si el hilo principal muere a los 3ms, ¿qué pasa con los hilos hijos que siguen corriendo?

**Pregunta 4**
¿Cómo harías para que el tiempo total refleje cuándo terminaron TODOS los hilos? (Pista para sesiones futuras)

---

➡️ [Siguiente sesión: Simulando tareas paralelas](./sesion-04.md)
