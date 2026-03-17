# Sesión 02 — Preparando el ambiente

## 📋 Información

| Campo | Detalle |
|-------|---------|
| **Módulo** | 1 — Threads en Java |
| **Duración estimada** | 45 minutos |
| **Tipo** | 💻 Práctica |
| **Dificultad** | Introductoria |

---

## 🗂️ Estructura del proyecto base

El curso trabaja con un proyecto llamado **Screenmatch**, una app de catálogo de series/películas. La estructura típica es la de un proyecto Maven estándar:

```
screenmatch/
├── src/
│   └── main/
│       └── java/
│           └── com/aluracursos/screenmatch/
│               ├── Main.java
│               ├── model/
│               │   ├── Serie.java
│               │   └── Episodio.java
│               └── service/
│                   └── ConsumoAPI.java
├── pom.xml
```

---

## 🛠️ Configura tu proyecto base

Crea un proyecto Maven nuevo con este `pom.xml` mínimo:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.aluracursos</groupId>
    <artifactId>screenmatch</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
</project>
```

---

## 🧱 Las 3 clases base

### `Serie.java`

```java
package com.aluracursos.screenmatch.model;

public class Serie {
    private String nombre;
    private int totalEpisodios;

    public Serie(String nombre, int totalEpisodios) {
        this.nombre = nombre;
        this.totalEpisodios = totalEpisodios;
    }

    public String getNombre() { return nombre; }
    public int getTotalEpisodios() { return totalEpisodios; }

    @Override
    public String toString() {
        return "Serie: " + nombre + " (" + totalEpisodios + " episodios)";
    }
}
```

### `ConsumoAPI.java`

```java
package com.aluracursos.screenmatch.service;

public class ConsumoAPI {

    // Simula el tiempo de respuesta de una API real (entre 1 y 3 segundos)
    public String obtenerDatosSerie(String nombreSerie) {
        try {
            long tiempoSimulado = (long)(Math.random() * 2000) + 1000;
            Thread.sleep(tiempoSimulado);
            return "Datos de [" + nombreSerie + "] obtenidos en " + tiempoSimulado + "ms";
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return "Error al obtener datos de " + nombreSerie;
        }
    }
}
```

### `Main.java`

```java
package com.aluracursos.screenmatch;

import com.aluracursos.screenmatch.service.ConsumoAPI;

public class Main {
    public static void main(String[] args) {
        ConsumoAPI api = new ConsumoAPI();

        String[] series = {"Breaking Bad", "Dark", "Squid Game"};

        long inicio = System.currentTimeMillis();

        for (String serie : series) {
            String resultado = api.obtenerDatosSerie(serie);
            System.out.println(resultado);
        }

        long fin = System.currentTimeMillis();
        System.out.println("\nTiempo total: " + (fin - inicio) + "ms");
    }
}
```

---

## ▶️ Ejecútalo y observa

Al correr `Main.java` verás algo así:

```
Datos de [Breaking Bad] obtenidos en 1823ms
Datos de [Dark] obtenidos en 2147ms
Datos de [Squid Game] obtenidos en 1205ms

Tiempo total: 5175ms  ← suma de los 3 tiempos
```

**Este es el problema que vas a resolver en las sesiones siguientes.** Las 3 series se consultan una por una. El tiempo total es la *suma* de todos. Con threads, el tiempo total será aproximadamente el de la más lenta — no la suma.

---

## 🔍 Observa el `Thread.sleep()`

```java
Thread.sleep(tiempoSimulado);
```

Esta línea **pausa el hilo actual** por X milisegundos. Simula perfectamente lo que ocurre en la vida real cuando tu código espera una respuesta de API, una consulta a base de datos, leer un archivo. El hilo queda bloqueado sin hacer nada útil.

---

## ✅ Checklist de la sesión

- [ ] Proyecto Maven creado y compilando sin errores
- [ ] Las 3 clases creadas en sus paquetes correctos
- [ ] `Main.java` ejecutado y viste el tiempo total (~4-6 segundos)
- [ ] Entiendes por qué el tiempo es la suma y no el máximo

---

## 🧠 Mini-test

**Pregunta 1**
Si agregaras 10 series más al arreglo, ¿cuánto tiempo tomaría aproximadamente el proceso de forma síncrona?

**Pregunta 2**
Si esas 13 series corrieran en paralelo, ¿cuánto tiempo tomaría aproximadamente?

**Pregunta 3**
¿Qué hace exactamente `Thread.sleep()` y por qué es útil para simular llamadas a APIs?

---

➡️ [Siguiente sesión: Tareas síncronas y asíncronas](./sesion-03.md)
