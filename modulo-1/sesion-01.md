# Sesión 01 — Presentación del curso

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 35 minutos |
| **Tipo** | 📖 Teoría |
| **Dificultad** | Introductoria |

---

## 🧠 ¿De qué trata este curso?

Este curso te enseña a escribir programas Java que hacen **varias cosas al mismo tiempo**. Suena simple, pero es uno de los temas más complejos y más demandados en el mundo real.

Vas a aprender a:
- Crear y manejar **hilos (threads)** en Java
- Evitar errores clásicos de concurrencia como **condiciones de carrera**
- Sincronizar acceso a recursos compartidos
- Usar herramientas modernas como `@Async`, `CompletableFuture` y **Virtual Threads** (Java 21)

---

## 🗺️ El mapa del curso en 1 minuto

| Módulo | Tema central | Tecnología clave |
|--------|-------------|-----------------|
| 1 | Hilos en Java puro | `Thread`, `Runnable`, `synchronized` |
| 2 | Sincronización en BD | JPA Locks, Optimistic/Pessimistic |
| 3 | Async en Spring Boot | `@Async`, `JavaMailSender` |
| 4 | Tareas programadas | `@Scheduled`, `CompletableFuture` |
| 5 | Performance máxima | Virtual Threads (Java 21) |

---

## 💡 Conceptos clave

### ¿Qué es un hilo (thread)?

Es la unidad mínima de ejecución. Tu programa normalmente corre en **un solo hilo** (el hilo principal). Con multithreading, puedes lanzar varios hilos que ejecutan código **en paralelo**.

```java
// Esto lo ejecuta UN solo hilo, de arriba a abajo
System.out.println("Tarea 1");
System.out.println("Tarea 2"); // espera a que termine Tarea 1
System.out.println("Tarea 3"); // espera a que terminen las anteriores
```

```java
// Con threads, las 3 tareas pueden correr AL MISMO TIEMPO
new Thread(() -> System.out.println("Tarea 1")).start();
new Thread(() -> System.out.println("Tarea 2")).start();
new Thread(() -> System.out.println("Tarea 3")).start();
// El orden de impresión ya NO está garantizado
```

### ¿Por qué importa?

- Un servidor web atiende miles de peticiones simultáneas → necesita threads
- Una app que descarga archivos no debe "congelarse" → thread en background
- Generar reportes pesados sin bloquear al usuario → async

---

## ⚠️ El gran desafío de la concurrencia

Cuando varios hilos comparten datos, pueden pisarse entre sí. Este es el problema que resolverás durante todo el curso:

```java
// ❌ Problema clásico: dos hilos modificando el mismo saldo
int saldo = 1000;

// Hilo A lee saldo = 1000, suma 500 → quiere escribir 1500
// Hilo B lee saldo = 1000, resta 800 → quiere escribir 200
// ¿Cuál gana? El resultado es impredecible 💥
```

Esto se llama **condición de carrera** (*race condition*) y lo vas a reproducir y resolver en la Sesión 7.

---

## 🔭 Vista previa: lo que construirás

A lo largo del módulo 1 trabajarás con proyectos reales:
- **Screenmatch** — una app de streaming donde sincronizarás la descarga paralela de episodios
- **Sistema de biblioteca** — donde manejarás préstamos concurrentes de libros

---

## ✅ Checklist de la sesión

- [ ] ¿Qué es un hilo y en qué se diferencia de un proceso?
- [ ] ¿Por qué el orden de ejecución de threads no está garantizado?
- [ ] ¿Qué problema genera que dos threads modifiquen el mismo dato?

---

## 🧠 Mini-test

**Pregunta 1**
¿Qué es un hilo y en qué se diferencia de un proceso?

**Pregunta 2**
¿Por qué el orden de ejecución de threads no está garantizado?

**Pregunta 3**
¿Qué problema genera que dos threads modifiquen el mismo dato?

---

## 🎯 Reflexión

> Piensa en una app que usas a diario (WhatsApp, Spotify, tu banco). ¿Qué cosas podrían estar corriendo en paralelo internamente? Los mensajes que llegan mientras escuchas música, las notificaciones push, la sincronización en background... todo eso son threads.

---

➡️ [Siguiente sesión: Preparando el ambiente](./sesion-02.md)
