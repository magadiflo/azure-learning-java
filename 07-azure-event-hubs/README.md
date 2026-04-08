# 🌊 Módulo 07 — Azure Event Hubs

> 📚 **Serie:** Azure para Java Developers  
> 🗂️ **Módulo:** 07 de 08  
> 🎯 **Objetivo:** Entender qué es Azure Event Hubs, cómo se diferencia de Service Bus, el modelo de particiones y
> Consumer Groups, y cómo procesar flujos masivos de eventos desde Spring Boot con enfoque reactivo.

---

## 📋 Tabla de Contenidos

1. [¿Qué es el streaming de eventos?](#-1-qué-es-el-streaming-de-eventos-el-contexto-previo)
2. [¿Qué es Azure Event Hubs?](#-2-qué-es-azure-event-hubs)
3. [Cómo está organizado](#-3-cómo-está-organizado-azure-event-hubs)
4. [Particiones — El concepto más importante](#-4-particiones--el-concepto-más-importante)
5. [Consumer Groups — Lectura independiente por equipo](#-5-consumer-groups--lectura-independiente-por-equipo)
6. [Event Hubs vs Service Bus — La gran diferencia](#-6-event-hubs-vs-service-bus--la-gran-diferencia)
7. [Tiers y capacidad](#-7-tiers-y-capacidad-throughput-units)
8. [Event Hubs Capture — Archivado automático](#-8-event-hubs-capture--archivado-automático)
9. [Integración con Spring Boot](#-9-integración-con-spring-boot)
10. [Ejemplo de código completo](#-10-ejemplo-de-código-completo)
11. [Casos de uso reales](#-11-casos-de-uso-reales-en-el-mundo-laboral)
12. [Recursos y videos](#-12-recursos-y-videos-recomendados)
13. [Resumen ejecutivo](#-resumen-ejecutivo)

---

## 🌊 1. ¿Qué es el Streaming de Eventos? (El contexto previo)

Para entender Event Hubs, primero necesitas entender la diferencia entre **mensajería** y **streaming**. Son dos
paradigmas distintos aunque a veces se confunden.

### Mensajería vs Streaming — La diferencia fundamental

Imagina una empresa de logística que recibe paquetes:

```
MENSAJERÍA (Service Bus) — como una oficina de correos
┌─────────────────────────────────────────────────────────┐
│  Cada paquete (mensaje) tiene un destinatario.          │
│  Cuando el destinatario lo recoge, el paquete           │
│  desaparece. Si el destinatario no está, el paquete     │
│  espera. Máximo 14 días en el almacén.                  │
│  Volumen: miles de paquetes por hora.                   │
└─────────────────────────────────────────────────────────┘

STREAMING (Event Hubs) — como un río
┌─────────────────────────────────────────────────────────┐
│  El agua (eventos) fluye constantemente.                │
│  Múltiples personas pueden sacar agua del mismo río     │
│  al mismo tiempo sin que "se consuma".                  │
│  El agua fluye hacia adelante — nadie la "recoge".      │
│  Se puede retener hasta 90 días antes de evaporarse.    │
│  Volumen: millones de litros por segundo.               │
└─────────────────────────────────────────────────────────┘
```

### El problema que resuelve el streaming

Hay escenarios donde necesitas procesar **millones de eventos por segundo** provenientes de múltiples fuentes
simultáneamente:

- 🌡️ **IoT:** 100,000 sensores de temperatura enviando lecturas cada segundo.
- 📱 **Apps móviles:** 5 millones de usuarios generando clicks, vistas y acciones.
- 🏦 **Fintech:** Todas las transacciones de una red de cajeros en tiempo real.
- 🚗 **Telemática:** 50,000 vehículos enviando su posición GPS cada 5 segundos.
- 🖥️ **Logs de aplicaciones:** 200 microservicios generando logs de diagnóstico.

Para estos casos, Service Bus no está diseñado — tiene un límite de throughput y su modelo de mensajería uno-a-uno o
pub/sub no escala a millones de eventos por segundo. **Event Hubs sí está diseñado exactamente para eso.**

> 💡 **Analogía definitiva:**
> - **Service Bus** = Sistema postal. Cada carta tiene un destinatario, se entrega y desaparece.
> - **Event Hubs** = Estación de radio. La señal se emite continuamente, múltiples radios la captan al mismo tiempo, y
    cada uno escucha en el punto donde sintonizó.

---

## 🌊 2. ¿Qué es Azure Event Hubs?

**Azure Event Hubs** es el servicio de ingesta y streaming de eventos masivos de Microsoft Azure. Está diseñado para
recibir y procesar millones de eventos por segundo desde múltiples fuentes simultáneas, reteniéndolos durante un período
configurable para que múltiples consumidores los lean de forma independiente.

### La versión larga (con contexto real)

Imagina que eres el equipo técnico de un banco digital peruano con 3 millones de clientes. Tu app móvil genera eventos
constantemente:

- Cada vez que un usuario abre la app → evento.
- Cada vez que consulta su saldo → evento.
- Cada vez que hace una transferencia → evento.
- Cada tap, cada scroll, cada tiempo en pantalla → eventos.

En hora pico, eso puede ser **500,000 eventos por minuto**. Necesitas:

- Detectar fraude en tiempo real (analizar patrones en los últimos 60 segundos).
- Calcular métricas de uso para el equipo de producto.
- Alimentar un modelo de ML que personaliza ofertas.
- Guardar todo en un data lake para análisis histórico.

**¿Cómo procesas 500,000 eventos/minuto con múltiples consumidores independientes, sin perder ni uno, y sin que un
consumidor lento afecte a los demás?**

Eso es exactamente lo que hace Event Hubs:

```
                    AZURE EVENT HUBS
                  "telemetria-app-movil"
                 ┌────────────────────────────┐
App Móvil   ──►  │                            │──► [Detector de Fraude]
(500K eventos    │   500,000 eventos/min      │──► [Métricas de Producto]
 por minuto)     │   retenidos hasta 7 días   │──► [Modelo de ML]
                 │                            │──► [Data Lake / Azure Storage]
Cajeros     ──►  │                            │
ATMs        ──►  └────────────────────────────┘
                    (cada consumidor lee
                     de forma independiente,
                     a su propio ritmo)
```

**Características que definen a Event Hubs:**

- 📈 **Escala masiva:** Hasta millones de eventos por segundo con throughput elástico.
- 🔄 **Multi-consumidor:** Múltiples Consumer Groups leen el mismo stream de forma independiente.
- ⏰ **Retención configurable:** Los eventos persisten entre 1 y 90 días (no se eliminan al ser leídos).
- 📍 **Offset tracking:** Cada consumidor recuerda hasta dónde leyó — puede retroceder y releer.
- 🌍 **Protocolo AMQP y Kafka:** Compatible con aplicaciones que ya usan Apache Kafka sin cambiar el código.
- 💰 **Costo optimizado para volumen:** Diseñado para altísimo throughput a bajo costo por evento.

> 💡 **En resumen:** Azure Event Hubs es el "autopista de datos" de Azure. Diseñado para cuando necesitas mover millones
> de eventos por segundo de múltiples fuentes a múltiples destinos, con retención temporal para que cada consumidor lea
> a su propio ritmo.

---

## 🏗️ 3. Cómo está organizado Azure Event Hubs

```
┌──────────────────────────────────────────────────────────────────────┐
│                      Namespace de Event Hubs                         │
│                miempresa.servicebus.windows.net                      │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐   │
│  │                    Event Hub: "telemetria"                    │   │
│  │                                                               │   │
│  │   Partición 0  │  Partición 1  │  Partición 2  │ Partición 3  │   │
│  │  ┌───────────┐ │ ┌───────────┐ │ ┌───────────┐ │┌──────────┐  │   │
│  │  │ evento 1  │ │ │ evento 2  │ │ │ evento 3  │ ││ evento 4 │  │   │
│  │  │ evento 5  │ │ │ evento 6  │ │ │ evento 7  │ ││ evento 8 │  │   │
│  │  │ evento 9  │ │ │ evento 10 │ │ │ evento 11 │ ││ evento 12│  │   │
│  │  │    ...    │ │ │    ...    │ │ │    ...    │ ││   ...    │  │   │
│  │  └───────────┘ │ └───────────┘ │ └───────────┘ │└──────────┘  │   │
│  │                                                               │   │
│  │  Consumer Group: "fraude"   → Lee las 4 particiones           │   │
│  │  Consumer Group: "metricas" → Lee las 4 particiones           │   │
│  │  Consumer Group: "datalake" → Lee las 4 particiones           │   │
│  └───────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
```

| Concepto             | Descripción                                                                       |
|----------------------|-----------------------------------------------------------------------------------|
| **Namespace**        | Contenedor de alto nivel. Agrupa Event Hubs relacionados.                         |
| **Event Hub**        | El canal de streaming. Como un "topic" pero optimizado para volumen.              |
| **Partición**        | Unidad de paralelismo. Los eventos se distribuyen entre particiones.              |
| **Consumer Group**   | Vista independiente del stream para un equipo de consumidores.                    |
| **Evento**           | La unidad de datos. Cuerpo + propiedades + timestamp. Hasta 1 MB.                 |
| **Offset**           | La posición de un consumidor dentro de una partición. Como un marcador de página. |
| **Retention Period** | Cuántos días se conservan los eventos antes de ser eliminados automáticamente.    |

---

## 🔀 4. Particiones — El concepto más importante

Las **particiones** son el mecanismo que hace posible el escalado masivo de Event Hubs. Es el concepto técnico más
importante y el que más se pregunta en entrevistas.

### ¿Qué es una partición?

Una partición es una **secuencia ordenada e inmutable de eventos**, como un log append-only. Cada evento en una
partición tiene un número de secuencia (offset) que crece de forma incremental.

```
PARTICIÓN 0 (log ordenado e inmutable):
┌──────────────────────────────────────────────────────────────┐
│ Offset 0 │ Offset 1 │ Offset 2 │ Offset 3 │ Offset 4 │ ...   │
│ evento A │ evento B │ evento C │ evento D │ evento E │ ...   │
└──────────────────────────────────────────────────────────────┘
                                                         ▲
                                              Nuevos eventos
                                              siempre al final
```

### ¿Por qué importan las particiones?

Cada partición es procesada por **exactamente un consumidor** dentro de un Consumer Group. Esto es lo que permite el
paralelismo:

```
Event Hub con 4 particiones + Consumer Group con 4 instancias:

Partición 0 ──────────► Instancia del consumidor 1
Partición 1 ──────────► Instancia del consumidor 2
Partición 2 ──────────► Instancia del consumidor 3
Partición 3 ──────────► Instancia del consumidor 4

Cada instancia procesa su partición de forma independiente y en paralelo.
```

### Cómo se asignan los eventos a las particiones

Cuando publicas un evento, puedes controlar a qué partición va:

**1. Round-robin (sin partition key):** Los eventos se distribuyen automáticamente entre todas las particiones de forma
balanceada. El orden global no está garantizado.

```bash
// Sin partition key - distribución automática
producer.send(new EventData("evento sin clave"));
```

**2. Por Partition Key:** Todos los eventos con la misma clave van siempre a la misma partición. Garantiza orden para
ese grupo de eventos.

```bash
// Con partition key - todos los eventos del cliente van a la misma partición
EventDataBatch batch = producer.createBatch(new CreateBatchOptions().setPartitionKey("cliente-001"));
batch.tryAdd(new EventData("transaccion del cliente 001"));
producer.send(batch);
```

**3. Por Partition ID:** Especificas exactamente a qué partición enviar.

```java
// Directamente a la partición 2
EventDataBatch batch = producer.createBatch(new CreateBatchOptions().setPartitionId("2"));
```

### ¿Cuántas particiones necesito?

Las particiones se definen al crear el Event Hub y **no se pueden reducir** después (solo aumentar en el tier
Premium/Dedicated).

| Escenario                             | Particiones recomendadas |
|---------------------------------------|--------------------------|
| Desarrollo y pruebas                  | 2–4                      |
| Apps medianas (hasta 10 MB/s)         | 4–8                      |
| Apps de alto tráfico (hasta 100 MB/s) | 16–32                    |
| IoT masivo o telemetría enterprise    | 32+                      |

> ⚠️ **Regla importante:** El número máximo de consumidores en paralelo dentro de un Consumer Group está limitado por el
> número de particiones. Si tienes 4 particiones, como máximo 4 instancias del consumidor procesan en paralelo. Las
> instancias extra estarán ociosas esperando.

---

## 👥 5. Consumer Groups — Lectura Independiente por Equipo

Un **Consumer Group** es una vista independiente y aislada del stream completo. Es lo que permite que múltiples
aplicaciones lean el mismo flujo de eventos sin interferirse entre sí.

### La analogía del libro en la biblioteca

```
STREAM DE EVENTOS = Un libro en la biblioteca

Consumer Group "fraude"   → Tiene su propio marcador de página (offset)
Consumer Group "metricas" → Tiene su propio marcador de página (offset)
Consumer Group "datalake" → Tiene su propio marcador de página (offset)

Cada uno lee el mismo libro a su propio ritmo.
Si el equipo de "metricas" va lento, no afecta al de "fraude".
Si "datalake" quiere releer desde el principio, los otros no se ven afectados.
```

### Diagrama de Consumer Groups

```
EVENT HUB: "telemetria-bancaria"
──────────────────────────────────────────────────────────

                     PARTICIÓN 0
eventos ──► [ev1][ev2][ev3][ev4][ev5][ev6][ev7][ev8]...

Consumer Group "fraude":    posición actual → ev7 ────►
Consumer Group "metricas":  posición actual → ev4 ──►
Consumer Group "datalake":  posición actual → ev1 ─►

(Cada grupo lee a su propio ritmo, sin afectar a los demás)
```

### Consumer Group $Default

Event Hubs crea automáticamente un Consumer Group llamado `$Default`. En desarrollo puedes usarlo directamente, pero en
producción siempre crea Consumer Groups nombrados — uno por cada aplicación consumidora.

```java
// Usar Consumer Group específico
EventProcessorClient processor = new EventProcessorClientBuilder()
                .connectionString(connectionString)
                .eventHubName("telemetria-bancaria")
                .consumerGroup("fraude")          // Consumer Group específico
                .checkpointStore(checkpointStore)
                .processEvent(context -> { /* procesar evento */ })
                .buildEventProcessorClient();
```

### Límites de Consumer Groups

| Tier      | Máximo Consumer Groups por Event Hub |
|-----------|--------------------------------------|
| Basic     | 1 (solo $Default)                    |
| Standard  | 20                                   |
| Premium   | 100                                  |
| Dedicated | Ilimitado                            |

---

## ⚖️ 6. Event Hubs vs Service Bus — La Gran Diferencia

Esta es la pregunta que más aparece en entrevistas técnicas cuando se mencionan ambos servicios. La respuesta debe ser
clara y precisa.

```
¿Cuándo usar cuál?

PREGUNTA 1: ¿Cuántos eventos por segundo?
  Menos de 10,000/seg ──────────────────────────► Service Bus puede manejarlo
  Más de 10,000/seg ────────────────────────────► Event Hubs

PREGUNTA 2: ¿Los eventos se "consumen" o se "observan"?
  Se consumen (y desaparecen) ──────────────────► Service Bus (mensajería)
  Se observan (y persisten) ────────────────────► Event Hubs (streaming)

PREGUNTA 3: ¿Múltiples consumidores necesitan el MISMO evento?
  Solo uno lo procesa ──────────────────────────► Service Bus (Queue)
  Todos lo reciben ────────────────────────────► Service Bus (Topic) o Event Hubs
  Cada uno a su propio ritmo, con retención ───► Event Hubs

PREGUNTA 4: ¿Necesitas garantías empresariales?
  Dead-letter, transacciones, sesiones ────────► Service Bus
  Alto throughput con retención temporal ──────► Event Hubs
```

### Tabla comparativa completa

| Característica                            | 📨 Azure Service Bus               | 🌊 Azure Event Hubs                  |
|-------------------------------------------|------------------------------------|--------------------------------------|
| **Propósito**                             | Mensajería empresarial             | Streaming e ingesta masiva           |
| **Throughput máximo**                     | ~1,000 msg/seg (Standard)          | Millones de eventos/seg              |
| **Tamaño máximo por mensaje**             | 256 KB (Std) / 100 MB (Prem)       | 1 MB por evento                      |
| **Retención**                             | Hasta 14 días                      | Hasta 90 días                        |
| **El mensaje desaparece al leerlo**       | ✅ Sí (se elimina)                  | ❌ No (persiste hasta expiración)     |
| **Múltiples consumidores independientes** | ✅ Topics/Suscripciones             | ✅ Consumer Groups                    |
| **Orden garantizado**                     | ✅ Con Sessions                     | ✅ Por partición                      |
| **Dead-Letter Queue**                     | ✅ Nativa                           | ❌ No existe                          |
| **Reintentos automáticos**                | ✅ Configurable                     | ❌ Responsabilidad del consumidor     |
| **Exactly-once delivery**                 | ✅ Con Peek-Lock                    | ❌ At-least-once                      |
| **Releer eventos pasados**                | ❌ No                               | ✅ Sí (hasta el período de retención) |
| **Compatible con Kafka**                  | ❌ No                               | ✅ Sí                                 |
| **Protocolo**                             | AMQP, HTTP                         | AMQP, HTTP, Kafka                    |
| **Ideal para**                            | Comandos, transacciones, workflows | Telemetría, logs, IoT, analytics     |

> 💡 **La distinción conceptual más importante para entrevistas:**
> - Service Bus: **"Haz esto"** (comando, acción, tarea que debe completarse)
> - Event Hubs: **"Esto ocurrió"** (hecho, observación, dato que debe analizarse)

---

## 💰 7. Tiers y Capacidad (Throughput Units)

La capacidad de Event Hubs se mide en **Throughput Units (TU)** — unidades de rendimiento.

### ¿Qué es un Throughput Unit?

| Operación             | Por Throughput Unit          |
|-----------------------|------------------------------|
| **Ingesta (entrada)** | 1 MB/seg o 1,000 eventos/seg |
| **Egreso (salida)**   | 2 MB/seg o 4,096 eventos/seg |

Si necesitas ingestar 10 MB/seg, necesitas 10 TUs.

### Tiers disponibles

| Tier          | TUs      | Particiones | Consumer Groups | Retención | Precio aprox. |
|---------------|----------|-------------|-----------------|-----------|---------------|
| **Basic**     | 1–20     | Hasta 32    | 1 ($Default)    | 1 día     | ~$11/TU/mes   |
| **Standard**  | 1–20     | Hasta 32    | 20              | 1–7 días  | ~$22/TU/mes   |
| **Premium**   | 1–16 PU* | Hasta 100   | 100             | 1–90 días | ~$653/PU/mes  |
| **Dedicated** | 1+ CU**  | Ilimitadas  | Ilimitados      | 1–90 días | ~$7,000+/mes  |

> *PU = Processing Unit (equivale a aprox. 6 TUs)  
> **CU = Capacity Unit

**Auto-inflate (Standard/Premium):** Puedes configurar que Azure aumente automáticamente los TUs cuando el tráfico los
requiera, hasta un máximo que defines. Muy útil para aplicaciones con picos impredecibles.

---

## 💾 8. Event Hubs Capture — Archivado Automático

**Event Hubs Capture** es una función que archiva automáticamente todos los eventos recibidos en **Azure Blob Storage**
o **Azure Data Lake** en formato **Apache Avro**, sin que escribas una sola línea de código.

```
Productores ──► [Event Hubs] ──► Consumidores (tiempo real)
                    │
                    │ Capture (automático)
                    ▼
              [Azure Blob Storage]
              /captura/2025/03/15/10/00.avro
              /captura/2025/03/15/10/05.avro
              /captura/2025/03/15/10/10.avro
              ...
```

### ¿Por qué es importante?

- **Persistencia permanente:** Los eventos en Event Hubs tienen un TTL (expiración). Capture los archiva permanentemente
  antes de que expiren.
- **Análisis histórico:** Los archivos Avro pueden procesarse con Spark, Azure Synapse, Databricks o cualquier
  herramienta de Big Data.
- **Sin código:** Se configura en 2 clicks desde el portal de Azure.
- **Formato Avro:** Un formato binario comprimido y con schema, estándar en el mundo de Big Data.

### Configuración de Capture en `application.properties`

```properties
# Capture se configura en Azure Portal, no en el código.
# Una vez habilitado, Azure lo gestiona automáticamente.
# Solo necesitas saber el path donde se guardan los archivos:
# {Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}
```

---

## ☕ 9. Integración con Spring Boot

### Dependencias en `pom.xml`

```xml

<dependencies>
    <!-- SDK oficial de Azure para Event Hubs -->
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-messaging-eventhubs</artifactId>
        <version>5.15.0</version>
    </dependency>

    <!-- Event Processor Client — para consumir eventos con checkpointing -->
    <!-- Checkpointing = guardar la posición actual del consumidor en Azure Storage -->
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-messaging-eventhubs-checkpointstore-blob</artifactId>
        <version>1.19.0</version>
    </dependency>

    <!-- Azure Identity — Managed Identity sin contraseñas -->
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-identity</artifactId>
        <version>1.11.0</version>
    </dependency>

    <!-- Spring Boot Starter para Azure Event Hubs (alternativa con Spring Cloud Azure) -->
    <dependency>
        <groupId>com.azure.spring</groupId>
        <artifactId>spring-cloud-azure-starter-eventhubs</artifactId>
        <version>5.8.0</version>
    </dependency>

    <!-- WebFlux — para procesamiento reactivo -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
<dependencies>
    <dependency>
        <groupId>com.azure.spring</groupId>
        <artifactId>spring-cloud-azure-dependencies</artifactId>
        <version>5.8.0</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>
</dependencies>
</dependencyManagement>
```

### Configuración en `application.properties`

```properties
# ============================================================
# Configuración de Azure Event Hubs
# Los valores reales vienen de Azure Application Settings
# ============================================================
# Connection string del namespace de Event Hubs
spring.cloud.azure.eventhubs.connection-string=${EVENT_HUBS_CONNECTION_STRING}
# Nombre del Event Hub (el canal de streaming)
spring.cloud.azure.eventhubs.event-hub-name=${EVENT_HUB_NAME}
# Consumer Group que usará esta instancia
spring.cloud.azure.eventhubs.consumer.consumer-group=${EVENT_HUB_CONSUMER_GROUP}
# Connection string de Azure Storage para el checkpointing
# (guardar la posición del consumidor para no reprocesar eventos)
spring.cloud.azure.eventhubs.processor.checkpoint-store.account-name=${STORAGE_ACCOUNT_NAME}
spring.cloud.azure.eventhubs.processor.checkpoint-store.container-name=eventhubs-checkpoints
spring.cloud.azure.eventhubs.processor.checkpoint-store.connection-string=${STORAGE_CONNECTION_STRING}
# Número máximo de eventos en un lote
spring.cloud.azure.eventhubs.processor.max-wait-time=PT5S
spring.cloud.azure.eventhubs.processor.prefetch-count=500
```

### `EventHubsConfig.java` — Configuración de clientes

```java
package com.ejemplo.eventhubs.config;

import com.azure.messaging.eventhubs.EventHubProducerClient;
import com.azure.messaging.eventhubs.EventHubClientBuilder;
import com.azure.messaging.eventhubs.EventProcessorClient;
import com.azure.messaging.eventhubs.EventProcessorClientBuilder;
import com.azure.messaging.eventhubs.checkpointstore.blob.BlobCheckpointStore;
import com.azure.storage.blob.BlobContainerAsyncClient;
import com.azure.storage.blob.BlobContainerClientBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class EventHubsConfig {

    @Value("${EVENT_HUBS_CONNECTION_STRING}")
    private String eventHubsConnectionString;

    @Value("${EVENT_HUB_NAME}")
    private String eventHubName;

    @Value("${STORAGE_CONNECTION_STRING}")
    private String storageConnectionString;

    /**
     * Cliente para PUBLICAR eventos en Event Hubs.
     * Thread-safe — singleton.
     */
    @Bean
    public EventHubProducerClient eventHubProducerClient() {
        return new EventHubClientBuilder()
                .connectionString(eventHubsConnectionString, eventHubName)
                .buildProducerClient();
    }

    /**
     * BlobContainerAsyncClient para el checkpointing.
     *
     * El checkpointing guarda en Azure Blob Storage la posición
     * actual de cada consumidor en cada partición.
     * Si el consumidor se reinicia, retoma desde donde dejó
     * en lugar de reprocesar todos los eventos desde el inicio.
     */
    @Bean
    public BlobContainerAsyncClient checkpointBlobContainerClient() {
        BlobContainerAsyncClient containerClient = new BlobContainerClientBuilder()
                .connectionString(storageConnectionString)
                .containerName("eventhubs-checkpoints")
                .buildAsyncClient();

        // Crear el contenedor si no existe
        containerClient.createIfNotExists().block();
        return containerClient;
    }
}
```

---

## 💻 10. Ejemplo de Código Completo

### Escenario

Sistema de telemetría bancaria en tiempo real. La app móvil y los cajeros ATM generan eventos de actividad de usuario.
Tres Consumer Groups procesan el mismo stream de forma independiente: detección de fraude (tiempo real), métricas de
uso (análisis de producto) y archivado en data lake.

### Estructura del proyecto

```
mi-app-eventhubs/
└── src/main/java/com/ejemplo/eventhubs/
    ├── config/
    │   └── EventHubsConfig.java
    ├── model/
    │   └── EventoTelemetria.java
    ├── producer/
    │   └── TelemetriaProducer.java
    ├── consumer/
    │   ├── FraudeConsumer.java
    │   ├── MetricasConsumer.java
    │   └── DataLakeConsumer.java
    └── controller/
        └── TelemetriaController.java
```

### `EventoTelemetria.java` — Modelo del evento

```java
package com.ejemplo.eventhubs.model;

import com.fasterxml.jackson.annotation.JsonProperty;

import java.time.LocalDateTime;

/**
 * Representa un evento de telemetría generado por la app móvil o un cajero ATM.
 * Este mismo objeto es procesado por todos los Consumer Groups.
 */
public class EventoTelemetria {

    @JsonProperty("eventoId")
    private String eventoId;

    @JsonProperty("clienteId")
    private String clienteId;

    @JsonProperty("tipoEvento")
    private String tipoEvento;  // LOGIN, CONSULTA_SALDO, TRANSFERENCIA, RETIRO_ATM, etc.

    @JsonProperty("canal")
    private String canal;       // APP_MOVIL, ATM, WEB

    @JsonProperty("monto")
    private Double monto;       // null si no aplica (ej: LOGIN)

    @JsonProperty("ubicacion")
    private String ubicacion;   // Ciudad o coordenadas GPS

    @JsonProperty("dispositivoId")
    private String dispositivoId;

    @JsonProperty("timestamp")
    private LocalDateTime timestamp;

    @JsonProperty("exitoso")
    private Boolean exitoso;

    // Constructor vacío para Jackson
    public EventoTelemetria() {
    }

    public EventoTelemetria(String eventoId, String clienteId, String tipoEvento,
                            String canal, Double monto, String ubicacion,
                            String dispositivoId) {
        this.eventoId = eventoId;
        this.clienteId = clienteId;
        this.tipoEvento = tipoEvento;
        this.canal = canal;
        this.monto = monto;
        this.ubicacion = ubicacion;
        this.dispositivoId = dispositivoId;
        this.timestamp = LocalDateTime.now();
        this.exitoso = true;
    }

    /* Getters y Setters */

    @Override
    public String toString() {
        return String.format("EventoTelemetria{id=%s, cliente=%s, tipo=%s, canal=%s}",
                eventoId, clienteId, tipoEvento, canal);
    }
}
```

### `TelemetriaProducer.java` — Publicador de eventos en lote

```java
package com.ejemplo.eventhubs.producer;

import com.azure.messaging.eventhubs.EventData;
import com.azure.messaging.eventhubs.EventDataBatch;
import com.azure.messaging.eventhubs.EventHubProducerClient;
import com.azure.messaging.eventhubs.models.CreateBatchOptions;
import com.ejemplo.eventhubs.model.EventoTelemetria;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;

import java.util.List;

@Service
public class TelemetriaProducer {

    @Autowired
    private EventHubProducerClient producerClient;

    @Autowired
    private ObjectMapper objectMapper;

    /**
     * Publica UN evento individual en Event Hubs.
     *
     * Usa el clienteId como Partition Key para garantizar que todos
     * los eventos del mismo cliente vayan a la misma partición —
     * esto permite detectar patrones de comportamiento en orden cronológico.
     */
    public void publicarEvento(EventoTelemetria evento) {
        try {
            String eventoJson = objectMapper.writeValueAsString(evento);

            // Usar clienteId como Partition Key — orden garantizado por cliente
            CreateBatchOptions options = new CreateBatchOptions()
                    .setPartitionKey(evento.getClienteId());

            EventDataBatch batch = producerClient.createBatch(options);

            EventData eventData = new EventData(eventoJson);

            // Agregar metadatos como propiedades del evento (útiles para filtrado)
            eventData.getProperties().put("tipoEvento", evento.getTipoEvento());
            eventData.getProperties().put("canal", evento.getCanal());
            eventData.getProperties().put("clienteId", evento.getClienteId());

            batch.tryAdd(eventData);
            producerClient.send(batch);

            System.out.println("✅ Evento publicado en Event Hubs — " + evento);

        } catch (JsonProcessingException e) {
            throw new RuntimeException("Error serializando evento de telemetría", e);
        }
    }

    /**
     * Publica un LOTE de eventos en Event Hubs de forma eficiente.
     *
     * Event Hubs está optimizado para lotes — es mucho más eficiente
     * enviar 1,000 eventos en un lote que 1,000 llamadas individuales.
     * Esto reduce el costo y aumenta el throughput.
     *
     * @param eventos  Lista de eventos a publicar.
     */
    public void publicarLote(List<EventoTelemetria> eventos) {
        if (eventos == null || eventos.isEmpty()) return;

        try {
            // Agrupar eventos por clienteId para mantener orden por cliente
            // En producción usarías una lógica más sofisticada de agrupación
            EventDataBatch batch = producerClient.createBatch();
            int eventosEnviados = 0;

            for (EventoTelemetria evento : eventos) {
                String eventoJson = objectMapper.writeValueAsString(evento);
                EventData eventData = new EventData(eventoJson);
                eventData.getProperties().put("tipoEvento", evento.getTipoEvento());
                eventData.getProperties().put("clienteId", evento.getClienteId());

                // Si el lote está lleno, lo enviamos y creamos uno nuevo
                if (!batch.tryAdd(eventData)) {
                    producerClient.send(batch);
                    eventosEnviados += batch.getCount();
                    System.out.println("📦 Lote enviado: " + batch.getCount() + " eventos");

                    // Crear nuevo lote y agregar el evento que no cupo
                    batch = producerClient.createBatch();
                    batch.tryAdd(eventData);
                }
            }

            // Enviar el último lote (puede estar parcialmente lleno)
            if (batch.getCount() > 0) {
                producerClient.send(batch);
                eventosEnviados += batch.getCount();
            }

            System.out.println("✅ Total eventos publicados: " + eventosEnviados);

        } catch (JsonProcessingException e) {
            throw new RuntimeException("Error en publicación de lote", e);
        }
    }

    /**
     * Versión reactiva — para integración con Spring WebFlux.
     */
    public Mono<Void> publicarEventoReactivo(EventoTelemetria evento) {
        return Mono.fromRunnable(() -> publicarEvento(evento))
                .then();
    }
}
```

### `FraudeConsumer.java` — Consumidor reactivo con checkpointing

```java
package com.ejemplo.eventhubs.consumer;

import com.azure.messaging.eventhubs.EventProcessorClient;
import com.azure.messaging.eventhubs.EventProcessorClientBuilder;
import com.azure.messaging.eventhubs.checkpointstore.blob.BlobCheckpointStore;
import com.azure.messaging.eventhubs.models.ErrorContext;
import com.azure.messaging.eventhubs.models.EventContext;
import com.azure.storage.blob.BlobContainerAsyncClient;
import com.ejemplo.eventhubs.model.EventoTelemetria;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.ApplicationRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

/**
 * Consumer Group: "fraude"
 *
 * Procesa TODOS los eventos del stream en tiempo real buscando
 * patrones sospechosos. Completamente independiente de los demás
 * Consumer Groups.
 *
 * Usa EventProcessorClient — el cliente recomendado para producción
 * porque gestiona automáticamente:
 *   - Distribución de particiones entre instancias del consumidor
 *   - Checkpointing (guardar posición en Azure Blob Storage)
 *   - Rebalanceo cuando se agregan/quitan instancias
 */
@Component
public class FraudeConsumer {

    @Autowired
    private BlobContainerAsyncClient checkpointContainerClient;

    @Autowired
    private ObjectMapper objectMapper;

    @Value("${EVENT_HUBS_CONNECTION_STRING}")
    private String connectionString;

    @Value("${EVENT_HUB_NAME}")
    private String eventHubName;

    /**
     * ApplicationRunner inicia el procesador automáticamente al arrancar la app.
     *
     * En producción tendrías esta lógica en un @Service con @PostConstruct,
     * pero ApplicationRunner es más claro para el ejemplo.
     */
    @Bean
    public ApplicationRunner iniciarProcesadorFraude() {
        return args -> {
            EventProcessorClient procesador = new EventProcessorClientBuilder()
                    .connectionString(connectionString, eventHubName)
                    .consumerGroup("fraude")
                    .checkpointStore(
                            new BlobCheckpointStore(checkpointContainerClient)
                    )
                    .processEvent(this::procesarEvento)
                    .processError(this::manejarError)
                    .buildEventProcessorClient();

            procesador.start();
            System.out.println("🚀 [FraudeConsumer] Procesador iniciado — Consumer Group: fraude");
        };
    }

    /**
     * Procesa cada evento del stream.
     *
     * El EventContext contiene:
     * - El evento en sí (EventData)
     * - La información de la partición
     * - El método para hacer checkpoint
     */
    private void procesarEvento(EventContext context) {
        try {
            String eventoJson = context.getEventData().getBodyAsString();
            EventoTelemetria evento = objectMapper.readValue(eventoJson, EventoTelemetria.class);

            System.out.println("🔍 [FraudeConsumer] Analizando evento:");
            System.out.println("   Cliente:   " + evento.getClienteId());
            System.out.println("   Tipo:      " + evento.getTipoEvento());
            System.out.println("   Canal:     " + evento.getCanal());
            System.out.println("   Partición: " + context.getPartitionContext().getPartitionId());
            System.out.println("   Offset:    " + context.getEventData().getOffset());

            // Lógica de detección de fraude
            boolean esSospechoso = analizarPatronFraude(evento);

            if (esSospechoso) {
                System.out.println("⚠️  [FraudeConsumer] ALERTA DE FRAUDE detectada para: " +
                                   evento.getClienteId());
                // En producción: publicar alerta en Service Bus, bloquear cuenta, etc.
            }

            // CHECKPOINTING: guardar la posición actual en Azure Blob Storage.
            // Si la app se reinicia, retomará desde aquí en lugar de reprocesar todo.
            // No hacer checkpoint en cada evento — es costoso.
            // Hacer checkpoint cada N eventos o cada cierto tiempo.
            if (context.getEventData().getOffset() % 100 == 0) {
                context.updateCheckpoint();
                System.out.println("💾 [FraudeConsumer] Checkpoint guardado en offset: " +
                                   context.getEventData().getOffset());
            }

        } catch (Exception e) {
            System.err.println("❌ [FraudeConsumer] Error procesando evento: " + e.getMessage());
            // En Event Hubs no hay DLQ automática — debes manejar el error manualmente
            // Opciones: log + continuar, guardar en Storage para reprocesar, alertar
        }
    }

    private void manejarError(ErrorContext context) {
        System.err.println("❌ [FraudeConsumer] Error en partición " +
                           context.getPartitionContext().getPartitionId() +
                           ": " + context.getThrowable().getMessage());
    }

    /**
     * Lógica simplificada de detección de fraude.
     * En producción: modelo de ML, reglas de negocio complejas, análisis de ventanas de tiempo.
     */
    private boolean analizarPatronFraude(EventoTelemetria evento) {
        // Regla simple: transacciones altas en cajeros considerados sospechosas
        if ("RETIRO_ATM".equals(evento.getTipoEvento()) &&
            evento.getMonto() != null &&
            evento.getMonto() > 5000) {
            return true;
        }
        // Regla: múltiples logins desde dispositivos distintos (simplificado)
        if ("LOGIN".equals(evento.getTipoEvento()) &&
            evento.getDispositivoId() != null &&
            evento.getDispositivoId().startsWith("DESCONOCIDO")) {
            return true;
        }
        return false;
    }
}
```

### `MetricasConsumer.java` — Consumidor de métricas con procesamiento por lotes

```java
package com.ejemplo.eventhubs.consumer;

import com.azure.messaging.eventhubs.EventProcessorClient;
import com.azure.messaging.eventhubs.EventProcessorClientBuilder;
import com.azure.messaging.eventhubs.checkpointstore.blob.BlobCheckpointStore;
import com.azure.messaging.eventhubs.models.ErrorContext;
import com.azure.messaging.eventhubs.models.EventContext;
import com.azure.storage.blob.BlobContainerAsyncClient;
import com.ejemplo.eventhubs.model.EventoTelemetria;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.ApplicationRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

/**
 * Consumer Group: "metricas"
 *
 * Lee el MISMO stream que FraudeConsumer pero de forma completamente
 * independiente. Acumula métricas de uso para el equipo de producto.
 *
 * Nota clave: este consumidor puede ir más lento que FraudeConsumer
 * sin afectarlo en absoluto. Cada uno mantiene su propio offset.
 */
@Component
public class MetricasConsumer {

    @Autowired
    private BlobContainerAsyncClient checkpointContainerClient;

    @Autowired
    private ObjectMapper objectMapper;

    @Value("${EVENT_HUBS_CONNECTION_STRING}")
    private String connectionString;

    @Value("${EVENT_HUB_NAME}")
    private String eventHubName;

    // Contadores en memoria para métricas en tiempo real
    // En producción: usar Redis o Azure Table Storage para persistencia
    private final ConcurrentHashMap<String, AtomicLong> contadorPorTipo =
            new ConcurrentHashMap<>();
    private final ConcurrentHashMap<String, AtomicLong> contadorPorCanal =
            new ConcurrentHashMap<>();

    @Bean
    public ApplicationRunner iniciarProcesadorMetricas() {
        return args -> {
            EventProcessorClient procesador = new EventProcessorClientBuilder()
                    .connectionString(connectionString, eventHubName)
                    .consumerGroup("metricas")   // Consumer Group distinto al de fraude
                    .checkpointStore(
                            new BlobCheckpointStore(checkpointContainerClient)
                    )
                    .processEvent(this::procesarEvento)
                    .processError(this::manejarError)
                    .buildEventProcessorClient();

            procesador.start();
            System.out.println("🚀 [MetricasConsumer] Procesador iniciado — Consumer Group: metricas");
        };
    }

    private void procesarEvento(EventContext context) {
        try {
            String eventoJson = context.getEventData().getBodyAsString();
            EventoTelemetria evento = objectMapper.readValue(eventoJson, EventoTelemetria.class);

            // Acumular métricas por tipo de evento
            contadorPorTipo
                    .computeIfAbsent(evento.getTipoEvento(), k -> new AtomicLong(0))
                    .incrementAndGet();

            // Acumular métricas por canal
            contadorPorCanal
                    .computeIfAbsent(evento.getCanal(), k -> new AtomicLong(0))
                    .incrementAndGet();

            // Imprimir resumen cada 50 eventos
            long totalEventos = contadorPorTipo.values().stream()
                    .mapToLong(AtomicLong::get).sum();

            if (totalEventos % 50 == 0) {
                System.out.println("📊 [MetricasConsumer] Resumen acumulado:");
                contadorPorTipo.forEach((tipo, count) ->
                        System.out.println("   " + tipo + ": " + count.get() + " eventos")
                );
                contadorPorCanal.forEach((canal, count) ->
                        System.out.println("   Canal " + canal + ": " + count.get() + " eventos")
                );

                // Checkpoint cada 50 eventos
                context.updateCheckpoint();
            }

        } catch (Exception e) {
            System.err.println("❌ [MetricasConsumer] Error: " + e.getMessage());
        }
    }

    private void manejarError(ErrorContext context) {
        System.err.println("❌ [MetricasConsumer] Error: " + context.getThrowable().getMessage());
    }
}
```

### `TelemetriaController.java` — API REST para publicar eventos

```java
package com.ejemplo.eventhubs.controller;

import com.ejemplo.eventhubs.model.EventoTelemetria;
import com.ejemplo.eventhubs.producer.TelemetriaProducer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Mono;

import java.util.List;
import java.util.Map;
import java.util.UUID;

@RestController
@RequestMapping("/api/telemetria")
public class TelemetriaController {

    @Autowired
    private TelemetriaProducer producer;

    /**
     * POST /api/telemetria/evento
     * Publica un evento individual desde la app móvil o ATM.
     */
    @PostMapping("/evento")
    public Mono<ResponseEntity<Map<String, String>>> publicarEvento(
            @RequestBody Map<String, Object> request) {

        EventoTelemetria evento = new EventoTelemetria(
                UUID.randomUUID().toString(),
                (String) request.get("clienteId"),
                (String) request.get("tipoEvento"),
                (String) request.getOrDefault("canal", "APP_MOVIL"),
                request.get("monto") != null
                        ? Double.parseDouble(request.get("monto").toString()) : null,
                (String) request.getOrDefault("ubicacion", "Lima"),
                (String) request.getOrDefault("dispositivoId", "MOVIL-001")
        );

        // Publicar reactivamente — no bloqueante
        return producer.publicarEventoReactivo(evento)
                .thenReturn(ResponseEntity.ok(Map.of(
                        "eventoId", evento.getEventoId(),
                        "estado", "PUBLICADO",
                        "mensaje", "Evento registrado en Event Hubs"
                )));
    }

    /**
     * POST /api/telemetria/lote
     * Publica múltiples eventos en un solo lote.
     * Más eficiente que múltiples llamadas individuales.
     */
    @PostMapping("/lote")
    public ResponseEntity<Map<String, Object>> publicarLote(
            @RequestBody List<Map<String, Object>> requests) {

        List<EventoTelemetria> eventos = requests.stream()
                .map(req -> new EventoTelemetria(
                        UUID.randomUUID().toString(),
                        (String) req.get("clienteId"),
                        (String) req.get("tipoEvento"),
                        (String) req.getOrDefault("canal", "APP_MOVIL"),
                        req.get("monto") != null
                                ? Double.parseDouble(req.get("monto").toString()) : null,
                        (String) req.getOrDefault("ubicacion", "Lima"),
                        (String) req.getOrDefault("dispositivoId", "MOVIL-001")
                ))
                .toList();

        producer.publicarLote(eventos);

        return ResponseEntity.ok(Map.of(
                "eventosPublicados", eventos.size(),
                "estado", "PUBLICADO",
                "mensaje", "Lote publicado en Event Hubs"
        ));
    }
}
```

### Flujo completo del sistema

```
App Móvil / ATM
       │
       │  POST /api/telemetria/evento
       ▼
[TelemetriaController]
       │
       ▼
[TelemetriaProducer]
       │  Partition Key = clienteId
       │  (orden garantizado por cliente)
       ▼
[Event Hubs — "telemetria-bancaria"]
  Partición 0: eventos cliente-001, cliente-005, ...
  Partición 1: eventos cliente-002, cliente-006, ...
  Partición 2: eventos cliente-003, cliente-007, ...
  Partición 3: eventos cliente-004, cliente-008, ...
       │
   ────┼────────────────────────────────────────
   │              │                            │
   ▼              ▼                            ▼
Consumer        Consumer                   Consumer
Group:          Group:                     Group:
"fraude"        "metricas"                 "datalake"
   │              │                            │
   ▼              ▼                            ▼
Detecta        Acumula                    Archiva en
alertas en     contadores                 Azure Blob
tiempo real    por tipo/canal             Storage
               de evento                  (Avro)

(Los 3 leen el mismo stream, a su propio ritmo, con su propio offset)
```

---

## 🏢 11. Casos de Uso Reales en el Mundo Laboral

### 🏦 Sector Bancario / Fintech

| Escenario                              | Detalle                                                                       |
|----------------------------------------|-------------------------------------------------------------------------------|
| **Detección de fraude en tiempo real** | Analizar cada transacción en < 100ms para aprobar o rechazar                  |
| **Monitoreo de cajeros ATM**           | 5,000 cajeros enviando estado (temperatura, papel, billetes) cada 30 segundos |
| **Telemetría de la app móvil**         | Cada acción del usuario en la app para personalización y UX analytics         |
| **Reporte regulatorio en tiempo real** | Stream continuo hacia el BCRP / SBS según normativa                           |

### 🏭 IoT / Industria

| Escenario                 | Detalle                                                                   |
|---------------------------|---------------------------------------------------------------------------|
| **Sensores industriales** | 10,000 sensores de una planta enviando temperatura y presión cada segundo |
| **Flota de vehículos**    | 50,000 camiones enviando GPS, velocidad y combustible cada 5 segundos     |
| **Smart meters**          | Medidores de luz inteligentes reportando consumo cada minuto              |

### 🛒 E-commerce / Retail

| Escenario                     | Detalle                                                              |
|-------------------------------|----------------------------------------------------------------------|
| **Clickstream analytics**     | Cada click del usuario en la web para recomendaciones en tiempo real |
| **Inventario en tiempo real** | Miles de tiendas actualizando stock simultáneamente                  |
| **Pipeline de datos**         | Alimentar un data warehouse con todos los eventos del día            |

### 🖥️ Observabilidad / DevOps

| Escenario                  | Detalle                                                       |
|----------------------------|---------------------------------------------------------------|
| **Centralización de logs** | 200 microservicios enviando logs → Event Hubs → Azure Monitor |
| **Métricas de aplicación** | APM centralizado para detectar degradaciones de performance   |

---

## 🎥 12. Recursos y Videos Recomendados

| Recurso                                | Descripción                                           | Dónde encontrarlo                                                                           |
|----------------------------------------|-------------------------------------------------------|---------------------------------------------------------------------------------------------|
| **Microsoft Learn — Event Hubs**       | Ruta oficial con módulos interactivos y gratuitos     | [learn.microsoft.com/azure/event-hubs](https://learn.microsoft.com/es-es/azure/event-hubs/) |
| **"Azure Event Hubs Tutorial"**        | Explicación de particiones y Consumer Groups con demo | Buscar en YouTube: `"Azure Event Hubs tutorial partitions consumer groups"`                 |
| **"Event Hubs vs Service Bus"**        | Comparación en profundidad con casos de uso           | Buscar en YouTube: `"Azure Event Hubs vs Service Bus when to use"`                          |
| **"Spring Boot + Azure Event Hubs"**   | Integración completa con Spring Cloud Azure           | Buscar en YouTube: `"Spring Boot Azure Event Hubs Microsoft"`                               |
| **"Apache Kafka vs Azure Event Hubs"** | Para entender la compatibilidad con Kafka             | Buscar en YouTube: `"Kafka Azure Event Hubs comparison"`                                    |

> 🎯 **Para practicar sin cuenta Azure:**  
> Event Hubs tiene un **emulador local oficial** lanzado por Microsoft en 2024 que puedes correr con Docker:
> ```bash
> docker pull mcr.microsoft.com/azure-messaging/eventhubs-emulator:latest
> docker run -p 5672:5672 mcr.microsoft.com/azure-messaging/eventhubs-emulator:latest
> ```
> Con esto puedes publicar y consumir eventos localmente con el mismo SDK que usarías en Azure.

### 📖 Documentación oficial clave

| Documento                           | URL                                                                                                                                                          |
|-------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Introducción a Azure Event Hubs     | [learn.microsoft.com/azure/event-hubs/event-hubs-about](https://learn.microsoft.com/es-es/azure/event-hubs/event-hubs-about)                                 |
| Particiones en Event Hubs           | [learn.microsoft.com/azure/event-hubs/event-hubs-scalability](https://learn.microsoft.com/es-es/azure/event-hubs/event-hubs-scalability)                     |
| SDK Java — Enviar y recibir eventos | [learn.microsoft.com/azure/event-hubs/event-hubs-java-get-started-send](https://learn.microsoft.com/es-es/azure/event-hubs/event-hubs-java-get-started-send) |
| Event Hubs Capture                  | [learn.microsoft.com/azure/event-hubs/event-hubs-capture-overview](https://learn.microsoft.com/es-es/azure/event-hubs/event-hubs-capture-overview)           |
| Emulador local de Event Hubs        | [learn.microsoft.com/azure/event-hubs/overview-emulator](https://learn.microsoft.com/es-es/azure/event-hubs/overview-emulator)                               |

---

## 🧠 Resumen Ejecutivo

> Lo que deberías poder decir en una entrevista si te preguntan *"¿Qué es Azure Event Hubs y en qué se diferencia de
Service Bus?"*

**Azure Event Hubs** es el servicio de ingesta y streaming de eventos masivos de Azure, diseñado para recibir y procesar
millones de eventos por segundo desde múltiples fuentes simultáneas. La diferencia fundamental con Service Bus es
conceptual: Service Bus es mensajería (los mensajes se consumen y desaparecen, hay garantías empresariales como
dead-letter queue), mientras que Event Hubs es streaming (los eventos persisten hasta su expiración, múltiples
consumidores los leen de forma independiente a su propio ritmo). El modelo de escalado se basa en **particiones** —
secuencias ordenadas e inmutables de eventos — donde cada partición es procesada por una sola instancia del consumidor
en paralelo; la elección de la **Partition Key** determina a qué partición va cada evento, y usar el identificador del
cliente garantiza que todos sus eventos lleguen en orden. Los **Consumer Groups** permiten que múltiples aplicaciones
lean el mismo stream de forma completamente independiente, cada una con su propio offset (posición). El **checkpointing
** en Azure Blob Storage persiste esa posición para que el consumidor retome desde donde dejó si se reinicia. Desde
Spring Boot se integra con el SDK `azure-messaging-eventhubs` usando `EventHubProducerClient` para publicar y
`EventProcessorClient` para consumir con balanceo automático entre particiones. Event Hubs también es compatible con el
protocolo de Apache Kafka, permitiendo migrar aplicaciones existentes sin cambiar el código cliente. Los casos de uso
más comunes son: telemetría de aplicaciones, detección de fraude en tiempo real, IoT masivo, centralización de logs y
pipelines de datos hacia un data lake.

---

## ⏭️ Siguiente módulo

> 🚪 **Módulo 08 — Azure API Management:** El servicio de gestión de APIs de Azure. Veremos qué es un API Gateway, cómo
> APIM protege, transforma, versiona y monitorea tus APIs, políticas de seguridad y cómo se integra con tu App Service y
> tus Azure Functions.

---

*📝 Documentación elaborada como parte del plan de aprendizaje Azure para desarrolladores Java Backend.*  
*🗓️ Última actualización: 2025*
