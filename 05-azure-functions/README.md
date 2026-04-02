# ⚡ Módulo 05 — Azure Functions

> 📚 **Serie:** Azure para Java Developers  
> 🗂️ **Módulo:** 05 de 08  
> 🎯 **Objetivo:** Entender qué es la arquitectura serverless, qué es Azure Functions, sus tipos de triggers, cómo se
> diferencia de App Service, y cómo crear funciones con Java que reaccionen a eventos del sistema.

---

## 📋 Tabla de Contenidos

1. [¿Qué es Serverless?](#-1-qué-es-serverless-el-contexto-previo)
2. [¿Qué es Azure Functions?](#-2-qué-es-azure-functions)
3. [Cómo funciona por dentro](#-3-cómo-funciona-por-dentro)
4. [Tipos de Triggers](#-4-tipos-de-triggers-el-concepto-central)
5. [Tipos de Bindings](#-5-tipos-de-bindings-entradas-y-salidas)
6. [Azure Functions vs App Service](#-6-azure-functions-vs-app-service)
7. [Planes de hospedaje](#-7-planes-de-hospedaje)
8. [Integración con Java](#-8-integración-con-java)
9. [Ejemplo de código](#-9-ejemplo-de-código)
10. [Casos de uso reales](#-10-casos-de-uso-reales-en-el-mundo-laboral)
11. [Recursos y videos](#-11-recursos-y-videos-recomendados)
12. [Resumen ejecutivo](#-resumen-ejecutivo)

---

## 🌩️ 1. ¿Qué es Serverless? (El contexto previo)

Antes de hablar de Azure Functions, es fundamental entender el concepto de **Serverless**, porque es un paradigma
completamente distinto a todo lo que vimos hasta ahora.

### La versión larga (con contexto real)

En los módulos anteriores vimos **Azure App Service**: tú despliegas tu app Spring Boot y Azure gestiona los servidores.
Pero el servidor siempre está activo — si nadie usa tu app a las 3 de la mañana, el servidor igual está corriendo y
consumiendo recursos (y generando costo).

**Serverless lleva esto un paso más allá con una pregunta radical:**

> *"¿Qué pasa si el servidor solo existe en el momento exacto en que hay algo que procesar, y desaparece cuando
termina?"*

Eso es exactamente **Serverless**: no significa que no hay servidores (siempre hay hardware en algún datacenter).
Significa que **tú no piensas en servidores**. Azure los crea y destruye automáticamente según la demanda, y tú pagas
solo por los milisegundos que tu código estuvo ejecutándose.

### La analogía del taxi vs el auto propio

```
App Service (servidor siempre activo)    Azure Functions (serverless)
┌─────────────────────────────────┐      ┌─────────────────────────────┐
│         🚗 Auto propio          │      │         🚕 Taxi / Uber      │
│                                 │      │                             │
│ Siempre disponible              │      │ Solo existe cuando lo pides │
│ Costo fijo mensual              │      │ Pagas solo el viaje         │
│ Tú lo mantienes                 │      │ No hay mantenimiento        │
│ Aunque no lo uses, paga         │      │ Si no usas, no pagas        │
│ Escala: compras otro auto       │      │ Escala: más taxis aparecen  │
└─────────────────────────────────┘      └─────────────────────────────┘
```

### ¿Qué cambia en la forma de pensar?

En App Service escribes una **aplicación completa** con ciclo de vida propio. En Serverless escribes **funciones
individuales** — pequeños fragmentos de código que hacen una sola cosa y terminan. No hay estado persistente entre
ejecuciones (a menos que lo externalices).

```
App Service (aplicación)           Azure Functions (funciones)
┌──────────────────────────┐       ┌──────────────────────────────────────┐
│   Spring Boot App        │       │  fn: procesarPago()                  │
│                          │       │  fn: enviarEmailBienvenida()         │
│  Controllers             │       │  fn: generarReporteMensual()         │
│  Services                │       │  fn: redimensionarImagen()           │
│  Repositories            │       │  fn: sincronizarInventario()         │
│  Configs                 │       │                                      │
│  (siempre activa)        │       │  (cada una vive solo cuando se llama)│
└──────────────────────────┘       └──────────────────────────────────────┘
```

> 💡 **En resumen:** Serverless es un modelo donde escribes funciones individuales que se ejecutan en respuesta a
> eventos, sin pensar en servidores. Pagas solo por el tiempo de ejecución real, con escalado automático e ilimitado.

---

## ⚡ 2. ¿Qué es Azure Functions?

**Azure Functions** es el servicio Serverless de Microsoft Azure. Permite ejecutar fragmentos de código (funciones) en
respuesta a eventos — una petición HTTP, un mensaje en una cola, un nuevo archivo en Storage, un timer, etc. — sin
provisionar ni gestionar servidores.

### La versión larga (con contexto real)

Imagina que tienes una aplicación de e-commerce con Spring Boot en App Service. Cada vez que un usuario hace un pedido,
necesitas:

1. Enviarle un email de confirmación.
2. Notificar al sistema de inventario para descontar el stock.
3. Generar una factura PDF y guardarla en Azure Storage.
4. Si el pedido supera S/. 500, notificar al equipo de ventas por Slack.

Podrías hacer todo eso dentro de tu app Spring Boot en el mismo hilo de la petición HTTP. Pero eso tiene problemas:

- La respuesta al usuario tarda más (espera a que se genere el PDF, se envíe el email, etc.).
- Si el servicio de email falla, falla toda la operación del pedido.
- Estos procesos secundarios consumen recursos de tu servidor principal.

**La solución con Azure Functions:**

```
Usuario hace pedido
        │
        ▼
[Spring Boot — App Service]
        │
        │  Guarda el pedido en BD
        │  Publica mensaje en Service Bus
        │  Retorna respuesta al usuario (rápido ✅)
        │
        └──► [Service Bus] ──────────────────────────────┐
                                                         │
                    ┌──────────────────────────────────  ▼ ───────────┐
                    │              Azure Functions                    │
                    │                                                 │
                    │  fn: enviarEmail()       → Email de confirmación│
                    │  fn: actualizarStock()   → Inventario           │
                    │  fn: generarFactura()    → PDF en Storage       │
                    │  fn: notificarVentas()   → Slack (si > S/.500)  │
                    └─────────────────────────────────────────────────┘
                          (se ejecutan en paralelo, de forma asíncrona)
```

Tu app principal responde al usuario en milisegundos. Las funciones procesan todo lo demás de forma asíncrona,
independiente y escalable.

**Características principales de Azure Functions:**

- ⚡ **Event-driven:** Solo se ejecutan cuando ocurre un evento. Sin eventos, no hay ejecución ni costo.
- 📈 **Escalado automático:** Si llegan 1,000 pedidos al mismo tiempo, Azure crea 1,000 instancias de la función en
  paralelo automáticamente.
- 💰 **Modelo de precio por consumo:** Pagas por el número de ejecuciones y el tiempo de CPU consumido, no por tiempo
  activo.
- 🌐 **Políglota:** Soporta Java, C#, Python, JavaScript, PowerShell y más.
- 🔗 **Integrado con el ecosistema Azure:** Se conecta nativamente con Service Bus, Event Hubs, Blob Storage, Cosmos DB,
  etc.

> 💡 **En resumen:** Azure Functions es el servicio serverless de Azure. Escribes funciones pequeñas que reaccionan a
> eventos (HTTP, mensajes, timers, archivos) sin preocuparte por servidores. Se escalan automáticamente y se pagan por
> ejecución.

---

## ⚙️ 3. Cómo Funciona por Dentro

### El ciclo de vida de una función

```
     EVENTO                  AZURE FUNCTIONS                  RESULTADO
       │                           │                              │
       ▼                           ▼                              ▼
  HTTP Request          1. Azure detecta el evento          Response HTTP
  Mensaje en cola   →   2. Levanta una instancia         →  Mensaje procesado
  Archivo subido        3. Ejecuta tu función               Archivo transformado
  Timer disparado       4. Instancia desaparece             Tarea completada
                           (si no hay más eventos)
```

### Host y Function App

```
┌─────────────────────────────────────────────────────┐
│                   Function App                      │
│            (contenedor de funciones)                │
│                                                     │
│  ┌─────────────────┐    ┌─────────────────────┐     │
│  │  Function 1     │    │  Function 2         │     │
│  │  procesarPago   │    │  enviarEmail        │     │
│  │  [HTTP Trigger] │    │  [Queue Trigger]    │     │
│  └─────────────────┘    └─────────────────────┘     │
│                                                     │
│  ┌─────────────────┐    ┌─────────────────────┐     │
│  │  Function 3     │    │  Function 4         │     │
│  │  generarReporte │    │  limpiarArchivos    │     │
│  │  [Timer Trigger]│    │  [Blob Trigger]     │     │
│  └─────────────────┘    └─────────────────────┘     │
│                                                     │
│  [host.json — configuración global]                 │
│  [local.settings.json — variables locales]          │
└─────────────────────────────────────────────────────┘
```

Una **Function App** es el contenedor lógico que agrupa múltiples funciones relacionadas. Comparten la misma
configuración, el mismo plan de hospedaje y las mismas variables de entorno (Application Settings).

### El archivo `host.json`

Controla el comportamiento global de todas las funciones en la Function App:

```json
{
  "version": "2.0",
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.*, 5.0.0)"
  },
  "functionTimeout": "00:10:00",
  "logging": {
    "logLevel": {
      "default": "Information",
      "Host.Results": "Error",
      "Function": "Information"
    }
  }
}
```

---

## 🎯 4. Tipos de Triggers — El concepto central

Un **Trigger** es el evento que dispara la ejecución de una función. Es el concepto más importante de Azure Functions.
Cada función tiene exactamente **un trigger** — no pueden tener más de uno.

### 🌐 HTTP Trigger — El más conocido

La función se ejecuta cuando recibe una petición HTTP. Puede actuar como un endpoint REST tradicional.

```
Cliente HTTP → GET /api/productos → [Azure Function] → Response JSON
```

**¿Cuándo usarlo?**

- APIs REST simples y puntuales que no justifican una app completa en App Service.
- Webhooks: recibir notificaciones de sistemas externos (pagos, envíos, etc.).
- Backends de formularios o landing pages.

```java
// Ejemplo de HTTP Trigger en Java
@FunctionName("obtenerProducto")
public HttpResponseMessage run(
        @HttpTrigger(
                name = "req",
                methods = {HttpMethod.GET},
                authLevel = AuthorizationLevel.FUNCTION,
                route = "productos/{id}"
        ) HttpRequestMessage<Optional<String>> request,
        @BindingName("id") String id,
        final ExecutionContext context) {

    context.getLogger().info("Buscando producto: " + id);
    // Tu lógica aquí
    return request.createResponseBuilder(HttpStatus.OK)
            .body("{\"id\": \"" + id + "\", \"nombre\": \"Laptop\"}")
            .build();
}
```

### ⏰ Timer Trigger — Tareas programadas

La función se ejecuta en un horario definido mediante una expresión **CRON**.

```
Cada día a las 02:00 AM → [Azure Function] → Genera reporte y lo envía por email
```

**¿Cuándo usarlo?**

- Reportes automáticos (diarios, semanales, mensuales).
- Limpieza de datos temporales o expirados.
- Sincronización periódica entre sistemas.
- Envío de recordatorios o notificaciones programadas.

```java
// Expresión CRON: "segundos minutos horas día mes díaSemana"
// "0 0 2 * * *" = todos los días a las 02:00:00

@FunctionName("generarReporteDiario")
public void run(
        @TimerTrigger(
                name = "timer",
                schedule = "0 0 2 * * *"  // Todos los días a las 2:00 AM
        ) String timerInfo,
        final ExecutionContext context) {

    context.getLogger().info("Generando reporte diario: " + LocalDateTime.now());
    // Lógica de generación de reporte
}
```

**Guía rápida de expresiones CRON:**

| Expresión          | Significado                                       |
|--------------------|---------------------------------------------------|
| `0 0 2 * * *`      | Todos los días a las 2:00 AM                      |
| `0 */30 * * * *`   | Cada 30 minutos                                   |
| `0 0 9 * * 1`      | Todos los lunes a las 9:00 AM                     |
| `0 0 0 1 * *`      | El primer día de cada mes a medianoche            |
| `0 0 8-18 * * 1-5` | Cada hora entre 8 AM y 6 PM, solo lunes a viernes |

### 📦 Blob Trigger — Reaccionar a archivos

La función se ejecuta automáticamente cuando se crea o modifica un archivo en Azure Blob Storage.

```
Usuario sube foto de perfil → [Azure Storage] → [Azure Function] → Genera miniatura y la guarda
```

**¿Cuándo usarlo?**

- Redimensionar imágenes automáticamente al subirlas.
- Procesar documentos PDF (extraer texto, validar estructura).
- Analizar archivos CSV o Excel con datos masivos.
- Convertir formatos de archivo (MP4 → diferentes resoluciones).
- Escanear archivos con antivirus antes de hacerlos disponibles.

```java

@FunctionName("procesarImagenSubida")
public void run(
        @BlobTrigger(
                name = "blob",
                path = "imagenes-originales/{nombre}",  // Escucha este contenedor
                connection = "AZURE_STORAGE_CONNECTION_STRING"
        ) byte[] contenidoImagen,
        @BindingName("nombre") String nombreArchivo,
        final ExecutionContext context) {

    context.getLogger().info("Nueva imagen detectada: " + nombreArchivo);
    // Aquí redimensionarías la imagen y la guardarías en otro contenedor
}
```

### 📨 Queue Trigger — Procesar mensajes de cola

La función se ejecuta cuando llega un mensaje a una cola de Azure Queue Storage o Service Bus.

```
App Spring Boot → Encola mensaje → [Azure Queue] → [Azure Function] → Procesa mensaje
```

**¿Cuándo usarlo?**

- Procesar tareas en segundo plano desacopladas de la app principal.
- Envío de emails o notificaciones push sin bloquear el hilo principal.
- Operaciones que pueden tardar (generación de PDFs, llamadas a APIs externas).
- Reintentar operaciones fallidas con backoff exponencial.

```java

@FunctionName("procesarEmailPendiente")
public void run(
        @QueueTrigger(
                name = "message",
                queueName = "emails-pendientes",
                connection = "AZURE_STORAGE_CONNECTION_STRING"
        ) String mensajeJson,
        final ExecutionContext context) {

    context.getLogger().info("Procesando email: " + mensajeJson);
    // Deserializar mensajeJson y enviar el email
}
```

### 📬 Service Bus Trigger — Mensajería empresarial

Similar al Queue Trigger pero con Service Bus, que ofrece garantías más robustas: exactly-once delivery, dead-letter
queues, topics con múltiples suscriptores, etc.

```
App Spring Boot → Publica en Topic → [Service Bus] → [Función A] [Función B] [Función C]
                                                       (cada una recibe una copia)
```

```java

@FunctionName("procesarPedidoNuevo")
public void run(
        @ServiceBusQueueTrigger(
                name = "message",
                queueName = "pedidos-nuevos",
                connection = "SERVICE_BUS_CONNECTION_STRING"
        ) String pedidoJson,
        final ExecutionContext context) {

    context.getLogger().info("Pedido recibido: " + pedidoJson);
    // Procesar el pedido
}
```

### 🌊 Event Hub Trigger — Streaming de eventos masivos

La función procesa eventos de Azure Event Hubs en lotes, ideal para telemetría e IoT.

```java

@FunctionName("procesarTelemetria")
public void run(
        @EventHubTrigger(
                name = "events",
                eventHubName = "telemetria-iot",
                connection = "EVENT_HUB_CONNECTION_STRING",
                cardinality = Cardinality.MANY   // Procesa múltiples eventos en un lote
        ) List<String> eventos,
        final ExecutionContext context) {

    context.getLogger().info("Procesando lote de " + eventos.size() + " eventos");
    eventos.forEach(evento -> {
        // Procesar cada evento del lote
    });
}
```

### 🌌 Cosmos DB Trigger — Reaccionar a cambios en la base de datos

La función se ejecuta cuando se insertan o modifican documentos en un contenedor de Cosmos DB. Usa el **Change Feed**
de Cosmos DB.

```
App guarda nuevo pedido en Cosmos DB → [Change Feed] → [Azure Function] → Notifica al cliente
```

```java

@FunctionName("onNuevoDocumentoCosmosDB")
public void run(
        @CosmosDBTrigger(
                name = "documents",
                databaseName = "tienda-db",
                containerName = "pedidos",
                leaseContainerName = "leases",   // Contenedor auxiliar para tracking
                connection = "COSMOS_CONNECTION_STRING",
                createLeaseContainerIfNotExists = true
        ) List<String> documentos,
        final ExecutionContext context) {

    context.getLogger().info("Cambios detectados: " + documentos.size() + " documentos");
    // Reaccionar a los cambios
}
```

### Resumen de todos los triggers

| Trigger            | Se activa cuando...                    | Caso de uso típico                   |
|--------------------|----------------------------------------|--------------------------------------|
| 🌐 **HTTP**        | Llega una petición HTTP                | APIs REST, webhooks                  |
| ⏰ **Timer**        | Se cumple una expresión CRON           | Reportes, limpieza, sincronización   |
| 📦 **Blob**        | Se crea/modifica un archivo en Storage | Procesar imágenes, PDFs, CSV         |
| 📨 **Queue**       | Llega un mensaje a Queue Storage       | Tareas en segundo plano simples      |
| 📬 **Service Bus** | Llega un mensaje a Service Bus         | Tareas críticas, pub/sub empresarial |
| 🌊 **Event Hub**   | Llegan eventos a Event Hubs            | Telemetría, IoT, streaming           |
| 🌌 **Cosmos DB**   | Cambia un documento en Cosmos DB       | Reaccionar a cambios de datos        |

---

## 🔗 5. Tipos de Bindings — Entradas y Salidas

Los **Bindings** son conexiones declarativas entre tu función y otros servicios de Azure. Son la forma en que Azure
Functions se integra con el ecosistema sin que escribas código de conexión.

Existen dos tipos:

- **Input Binding:** Lee datos de una fuente externa cuando la función se ejecuta.
- **Output Binding:** Escribe datos a un destino externo cuando la función termina.

```
                    INPUT BINDING         OUTPUT BINDING
                         │                     │
                         ▼                     ▼
[Trigger] → [Function] ←── [Cosmos DB]   ──→ [Azure Storage]
                       ←── [Table Storage] ──→ [Service Bus]
                                          ──→ [Email / SendGrid]
```

### Ejemplo con múltiples bindings

Una función que: se activa por HTTP (trigger), lee datos de Cosmos DB (input binding) y escribe un archivo en Blob
Storage (output binding):

```java

@FunctionName("generarReporteUsuario")
public HttpResponseMessage run(
        // TRIGGER — HTTP que activa la función
        @HttpTrigger(
                name = "req",
                methods = {HttpMethod.GET},
                route = "usuarios/{userId}/reporte"
        ) HttpRequestMessage<Optional<String>> request,

        // INPUT BINDING — Lee el usuario de Cosmos DB automáticamente
        @CosmosDBInput(
                name = "usuario",
                databaseName = "app-db",
                containerName = "usuarios",
                id = "{userId}",
                partitionKey = "{userId}",
                connection = "COSMOS_CONNECTION_STRING"
        ) String usuarioJson,

        // OUTPUT BINDING — Escribe el reporte en Blob Storage al finalizar
        @BlobOutput(
                name = "reporteBlob",
                path = "reportes/{userId}-reporte.json",
                connection = "AZURE_STORAGE_CONNECTION_STRING"
        ) OutputBinding<String> reporteBlob,

        final ExecutionContext context) {

    // Azure ya tiene el usuario cargado desde Cosmos DB — sin código de conexión
    context.getLogger().info("Generando reporte para: " + usuarioJson);

    String reporte = "{ \"usuario\": " + usuarioJson + ", \"generado\": \"" +
                     LocalDateTime.now() + "\" }";

    // Azure guarda esto en Blob Storage automáticamente — sin código de subida
    reporteBlob.setValue(reporte);

    return request.createResponseBuilder(HttpStatus.OK)
            .body("Reporte generado correctamente")
            .build();
}
```

> 💡 **La magia de los Bindings:** Sin bindings, tendrías que escribir código para conectarte a Cosmos DB, leer el
> usuario, luego conectarte a Azure Storage y subir el archivo. Con bindings declarativos, Azure hace todo eso
> automáticamente — tú solo escribes la lógica de negocio.

---

## ⚖️ 6. Azure Functions vs App Service

Esta es la pregunta más frecuente en entrevistas cuando se toca el tema serverless:

| Característica                 | 🚀 App Service                     | ⚡ Azure Functions                             |
|--------------------------------|------------------------------------|-----------------------------------------------|
| **Modelo**                     | Aplicación siempre activa          | Función que vive solo cuando se ejecuta       |
| **Unidad de código**           | Aplicación completa (Spring Boot)  | Función individual                            |
| **Arranque**                   | App siempre lista                  | Cold start en primer arranque (300ms–2s)      |
| **Escalado**                   | Manual o auto (hasta N instancias) | Automático e ilimitado                        |
| **Tiempo máximo de ejecución** | Sin límite                         | 5 min (Consumption) / Sin límite (Premium)    |
| **Estado**                     | Puede tener estado en memoria      | Sin estado entre ejecuciones                  |
| **Costo**                      | Fijo por tiempo activo             | Variable — por ejecución y duración           |
| **Spring Boot**                | ✅ Soporte nativo completo          | ⚠️ Parcial — solo funciones, no toda la app   |
| **Ideal para**                 | APIs REST con tráfico constante    | Tareas event-driven, esporádicas o asíncronas |
| **Observabilidad**             | Logs y métricas de la app completa | Por función individual                        |

### ¿Cuándo elegir cada uno?

```
¿El código necesita estar siempre disponible
y recibe tráfico constante?
            │
     SÍ ────┤────── NO
            │              │
            ▼              ▼
       App Service    ¿Se activa por un evento
                      (mensaje, archivo, timer)?
                                │
                         SÍ ────┤
                                │
                                ▼
                         Azure Functions ✅
```

> 📌 **Regla práctica:** Tu app Spring Boot principal vive en **App Service**. Los procesos secundarios, asincrónicos o
> basados en eventos van en **Azure Functions**. Los dos se complementan — rara vez se excluyen.

### El problema del Cold Start

El **Cold Start** es el tiempo que tarda Azure en levantar una instancia nueva de tu función cuando no había ninguna
activa. En Java puede tomar entre 1 y 3 segundos, lo que puede ser inaceptable para endpoints HTTP que esperan respuesta
rápida.

**Soluciones:**

| Solución                 | Descripción                                | Costo adicional               |
|--------------------------|--------------------------------------------|-------------------------------|
| **Premium Plan**         | Instancias siempre precalentadas           | Sí — instancia siempre activa |
| **GraalVM Native Image** | Compilación AOT — arranque en milisegundos | Tiempo de compilación         |
| **Keep-alive**           | Timer que llama a la función cada 5 min    | Mínimo                        |

---

## 💰 7. Planes de Hospedaje

| Plan                        | Escala                                 | Timeout      | Cold Start     | Precio                | ¿Para qué?                           |
|-----------------------------|----------------------------------------|--------------|----------------|-----------------------|--------------------------------------|
| **Consumption** ⭐           | Automática e ilimitada                 | 5 min        | Sí             | Por ejecución         | Funciones esporádicas, desarrollo    |
| **Flex Consumption**        | Automática con más control             | Configurable | Reducido       | Por ejecución + min   | Balance entre costo y latencia       |
| **Premium**                 | Automática + instancias pre-calentadas | Sin límite   | Sin cold start | Por instancia activa  | Funciones críticas con baja latencia |
| **Dedicated (App Service)** | Igual que App Service                  | Sin límite   | Sin cold start | Igual que App Service | Cuando ya tienes un App Service Plan |

> 💡 **Para aprendizaje y proyectos personales:** El plan **Consumption** es gratuito dentro del free tier (1 millón de
> ejecuciones/mes y 400,000 GB-segundos de cómputo gratis). Perfecto para practicar sin costo.

---

## ☕ 8. Integración con Java

### Estructura de un proyecto Azure Functions en Java

```
mi-funcion-azure/
├── pom.xml
├── host.json
├── local.settings.json           ← Variables de entorno locales (NO subir a Git)
└── src/
    └── main/
        └── java/com/ejemplo/functions/
            ├── HttpTriggerFunction.java
            ├── TimerTriggerFunction.java
            ├── BlobTriggerFunction.java
            └── ServiceBusTriggerFunction.java
```

### `pom.xml` — Dependencias y plugin de Maven

```xml

<project>
    <groupId>com.ejemplo</groupId>
    <artifactId>mi-funcion-azure</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>17</java.version>
        <azure.functions.maven.plugin.version>1.24.0</azure.functions.maven.plugin.version>
        <azure.functions.java.library.version>3.1.0</azure.functions.java.library.version>
        <functionAppName>mi-funcion-app</functionAppName>
        <functionRegion>brazilsouth</functionRegion>
    </properties>

    <dependencies>
        <!-- SDK de Azure Functions para Java — OBLIGATORIO -->
        <dependency>
            <groupId>com.microsoft.azure.functions</groupId>
            <artifactId>azure-functions-java-library</artifactId>
            <version>${azure.functions.java.library.version}</version>
        </dependency>

        <!-- Jackson — para serializar/deserializar JSON -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.15.2</version>
        </dependency>

        <!-- Azure Storage SDK — para interactuar con Blob Storage desde funciones -->
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-storage-blob</artifactId>
            <version>12.25.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Plugin de Maven para desplegar a Azure Functions -->
            <plugin>
                <groupId>com.microsoft.azure</groupId>
                <artifactId>azure-functions-maven-plugin</artifactId>
                <version>${azure.functions.maven.plugin.version}</version>
                <configuration>
                    <appName>${functionAppName}</appName>
                    <resourceGroup>mi-grupo-recursos</resourceGroup>
                    <region>${functionRegion}</region>
                    <runtime>
                        <os>linux</os>
                        <javaVersion>17</javaVersion>
                    </runtime>
                    <appSettings>
                        <property>
                            <name>FUNCTIONS_EXTENSION_VERSION</name>
                            <value>~4</value>
                        </property>
                    </appSettings>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### `local.settings.json` — Variables locales de desarrollo

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "java",
    "AZURE_STORAGE_CONNECTION_STRING": "DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;...",
    "SERVICE_BUS_CONNECTION_STRING": "tu-connection-string-local",
    "COSMOS_CONNECTION_STRING": "tu-connection-string-local"
  }
}
```

> ⚠️ **Importante:** `local.settings.json` contiene credenciales locales. Siempre agrégalo al `.gitignore` — nunca debe
> subirse al repositorio.

---

## 💻 9. Ejemplo de Código

### Escenario completo

Un sistema de procesamiento de pedidos para un e-commerce. Cuando un usuario hace un pedido desde la app Spring Boot
principal, esta publica un mensaje en Service Bus. Azure Functions procesa ese mensaje de forma asíncrona realizando
tres tareas: enviar email de confirmación, generar factura PDF y actualizar inventario.

### `PedidoDto.java` — Modelo compartido

```java
package com.ejemplo.functions.model;

import com.fasterxml.jackson.annotation.JsonProperty;

public class PedidoDto {

    @JsonProperty("pedidoId")
    private String pedidoId;

    @JsonProperty("clienteEmail")
    private String clienteEmail;

    @JsonProperty("clienteNombre")
    private String clienteNombre;

    @JsonProperty("total")
    private double total;

    @JsonProperty("productoId")
    private String productoId;

    @JsonProperty("cantidad")
    private int cantidad;

    /* Getters y Setters */
}
```

### `ProcesarPedidoFunction.java` — Service Bus Trigger

```java
package com.ejemplo.functions;

import com.ejemplo.functions.model.PedidoDto;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.microsoft.azure.functions.*;
import com.microsoft.azure.functions.annotation.*;

import java.time.LocalDateTime;

/**
 * Se activa cuando llega un nuevo pedido al Service Bus.
 * La app Spring Boot publica el pedido en "pedidos-nuevos" y esta función lo procesa.
 *
 * Flujo completo:
 * App Spring Boot → Service Bus (pedidos-nuevos) → Esta función → Email + Factura + Stock
 */
public class ProcesarPedidoFunction {

    private static final ObjectMapper objectMapper = new ObjectMapper();

    @FunctionName("procesarPedidoNuevo")
    public void run(
            @ServiceBusQueueTrigger(
                    name = "pedido",
                    queueName = "pedidos-nuevos",
                    connection = "SERVICE_BUS_CONNECTION_STRING"
            ) String pedidoJson,
            final ExecutionContext context) {

        context.getLogger().info("📦 Nuevo pedido recibido desde Service Bus");
        context.getLogger().info("Payload: " + pedidoJson);

        try {
            // 1. Deserializar el mensaje JSON
            PedidoDto pedido = objectMapper.readValue(pedidoJson, PedidoDto.class);

            context.getLogger().info("✅ Pedido ID: " + pedido.getPedidoId());
            context.getLogger().info("👤 Cliente: " + pedido.getClienteNombre());
            context.getLogger().info("💰 Total: S/. " + pedido.getTotal());

            // 2. Simular envío de email de confirmación
            enviarEmailConfirmacion(pedido, context);

            // 3. Simular generación de factura
            generarFactura(pedido, context);

            // 4. Simular actualización de inventario
            actualizarInventario(pedido, context);

            context.getLogger().info("✅ Pedido " + pedido.getPedidoId() + " procesado completamente");

        } catch (Exception e) {
            // Si ocurre un error, Azure Functions reintenta automáticamente
            // según la política de reintentos configurada en host.json.
            // Si supera el máximo de reintentos, el mensaje va al Dead Letter Queue.
            context.getLogger().severe("❌ Error procesando pedido: " + e.getMessage());
            throw new RuntimeException("Error en procesamiento", e);
        }
    }

    private void enviarEmailConfirmacion(PedidoDto pedido, ExecutionContext context) {
        // En producción: llamada a SendGrid, AWS SES o similar
        context.getLogger().info("📧 Email enviado a: " + pedido.getClienteEmail());
    }

    private void generarFactura(PedidoDto pedido, ExecutionContext context) {
        // En producción: generar PDF y subirlo a Azure Blob Storage
        context.getLogger().info("🧾 Factura generada para pedido: " + pedido.getPedidoId());
    }

    private void actualizarInventario(PedidoDto pedido, ExecutionContext context) {
        // En producción: llamada a la API de inventario o actualización en BD
        context.getLogger().info("📊 Inventario actualizado — producto: " +
                                 pedido.getProductoId() + " cantidad: -" + pedido.getCantidad());
    }
}
```

### `GenerarReporteDiarioFunction.java` — Timer Trigger

```java
package com.ejemplo.functions;

import com.microsoft.azure.functions.*;
import com.microsoft.azure.functions.annotation.*;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * Se ejecuta automáticamente todos los días a las 2:00 AM.
 * Genera el reporte de ventas del día anterior y lo envía por email.
 *
 * Expresión CRON: "0 0 2 * * *"
 * Formato: segundos minutos horas díaMes mes díaSemana
 */
public class GenerarReporteDiarioFunction {

    @FunctionName("generarReporteDiario")
    public void run(
            @TimerTrigger(
                    name = "timer",
                    schedule = "0 0 2 * * *"   // Todos los días a las 2:00 AM
            ) String timerInfo,
            final ExecutionContext context) {

        String fechaHoy = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));
        context.getLogger().info("⏰ Timer disparado: " + LocalDateTime.now());
        context.getLogger().info("📊 Iniciando generación de reporte para: " + fechaHoy);

        try {
            // 1. Consultar ventas del día anterior en la base de datos
            context.getLogger().info("📥 Consultando ventas del día...");
            // ventasService.obtenerVentasDelDia(LocalDate.now().minusDays(1));

            // 2. Generar el reporte en formato JSON o PDF
            String reporteJson = generarContenidoReporte(fechaHoy);
            context.getLogger().info("✅ Reporte generado: " + reporteJson);

            // 3. Guardar en Azure Blob Storage
            // blobStorageService.subirArchivo(reporte, "reportes/" + fechaHoy + ".json");

            // 4. Enviar email con el reporte adjunto
            // emailService.enviarReporte(reporte);

            context.getLogger().info("✅ Reporte diario completado para: " + fechaHoy);

        } catch (Exception e) {
            context.getLogger().severe("❌ Error generando reporte: " + e.getMessage());
            throw new RuntimeException("Error en generación de reporte", e);
        }
    }

    private String generarContenidoReporte(String fecha) {
        return String.format(
                "{\"fecha\": \"%s\", \"totalVentas\": 15750.00, \"numeroPedidos\": 42, " +
                "\"productoMasVendido\": \"Laptop Dell XPS\"}",
                fecha
        );
    }
}
```

### `ProcesarImagenSubidaFunction.java` — Blob Trigger

```java
package com.ejemplo.functions;

import com.microsoft.azure.functions.*;
import com.microsoft.azure.functions.annotation.*;

/**
 * Se activa automáticamente cuando se sube una imagen al contenedor
 * "imagenes-originales" en Azure Blob Storage.
 *
 * Flujo:
 * Usuario sube foto → Blob Storage (imagenes-originales) →
 * Esta función → Procesa la imagen → Guarda miniatura en (imagenes-miniaturas)
 */
public class ProcesarImagenSubidaFunction {

    @FunctionName("procesarImagenSubida")
    public void run(
            @BlobTrigger(
                    name = "imagenOriginal",
                    path = "imagenes-originales/{nombre}",   // Escucha este contenedor
                    connection = "AZURE_STORAGE_CONNECTION_STRING"
            ) byte[] contenidoImagen,

            // Output Binding — Escribe la miniatura en otro contenedor automáticamente
            @BlobOutput(
                    name = "miniatura",
                    path = "imagenes-miniaturas/{nombre}",   // Escribe en este contenedor
                    connection = "AZURE_STORAGE_CONNECTION_STRING"
            ) OutputBinding<byte[]> miniatura,

            @BindingName("nombre") String nombreArchivo,
            final ExecutionContext context) {

        context.getLogger().info("🖼️ Nueva imagen detectada: " + nombreArchivo);
        context.getLogger().info("📏 Tamaño original: " + contenidoImagen.length + " bytes");

        try {
            // En producción: usar una librería como Thumbnailator o ImageIO
            // para redimensionar la imagen a 200x200 píxeles
            byte[] imagenRedimensionada = redimensionarImagen(contenidoImagen, context);

            // El Output Binding sube automáticamente el resultado a Blob Storage
            miniatura.setValue(imagenRedimensionada);

            context.getLogger().info("✅ Miniatura generada para: " + nombreArchivo);
            context.getLogger().info("📏 Tamaño miniatura: " + imagenRedimensionada.length + " bytes");

        } catch (Exception e) {
            context.getLogger().severe("❌ Error procesando imagen: " + e.getMessage());
            throw new RuntimeException("Error procesando imagen " + nombreArchivo, e);
        }
    }

    private byte[] redimensionarImagen(byte[] imagenOriginal, ExecutionContext context) {
        // Simulación — en producción usar Thumbnailator:
        // Thumbnails.of(new ByteArrayInputStream(imagenOriginal))
        //           .size(200, 200)
        //           .outputFormat("jpg")
        //           .toOutputStream(outputStream);
        context.getLogger().info("🔄 Redimensionando imagen a 200x200px...");
        return imagenOriginal; // Retornamos original como simulación
    }
}
```

### `ApiHealthFunction.java` — HTTP Trigger simple

```java
package com.ejemplo.functions;

import com.microsoft.azure.functions.*;
import com.microsoft.azure.functions.annotation.*;

import java.time.LocalDateTime;
import java.util.Optional;

/**
 * Endpoint HTTP simple para verificar que la Function App está activa.
 * Útil para health checks y monitoreo.
 *
 * Acceso: GET https://mi-funcion-app.azurewebsites.net/api/health
 */
public class ApiHealthFunction {

    @FunctionName("health")
    public HttpResponseMessage run(
            @HttpTrigger(
                    name = "req",
                    methods = {HttpMethod.GET},
                    authLevel = AuthorizationLevel.ANONYMOUS,  // Sin autenticación
                    route = "health"
            ) HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {

        context.getLogger().info("🏥 Health check solicitado");

        String respuesta = String.format(
                "{\"estado\": \"UP\", \"funcion\": \"%s\", \"timestamp\": \"%s\"}",
                context.getFunctionName(),
                LocalDateTime.now()
        );

        return request
                .createResponseBuilder(HttpStatus.OK)
                .header("Content-Type", "application/json")
                .body(respuesta)
                .build();
    }
}
```

### Comandos para ejecutar localmente

```bash
# 1. Instalar Azure Functions Core Tools (necesario para ejecutar localmente)
npm install -g azure-functions-core-tools@4

# 2. Compilar el proyecto
mvn clean package

# 3. Ejecutar las funciones localmente (sin necesitar Azure)
mvn azure-functions:run

# 4. Desplegar a Azure (requiere cuenta Azure)
mvn azure-functions:deploy
```

---

## 🏢 10. Casos de Uso Reales en el Mundo Laboral

### 🏦 Sector Bancario / Fintech

| Función                           | Trigger                    | Qué hace                                                     |
|-----------------------------------|----------------------------|--------------------------------------------------------------|
| Alertas de fraude                 | Cosmos DB Trigger          | Se activa con cada transacción, analiza patrones sospechosos |
| Reporte regulatorio SBS           | Timer (primer día del mes) | Genera y envía reporte mensual obligatorio                   |
| Procesamiento de estado de cuenta | Queue Trigger              | Genera PDFs de estados de cuenta en batch                    |
| Notificaciones de movimientos     | Service Bus Trigger        | Envía push notification por cada débito/crédito              |

### 🛒 E-commerce / Retail

| Función                    | Trigger             | Qué hace                                                                 |
|----------------------------|---------------------|--------------------------------------------------------------------------|
| Redimensionar imágenes     | Blob Trigger        | Al subir foto de producto, genera 3 tamaños (original, medio, miniatura) |
| Confirmar pedidos          | Service Bus Trigger | Email + SMS de confirmación al hacer un pedido                           |
| Actualizar precios masivos | Timer (cada hora)   | Sincroniza precios desde el ERP                                          |
| Reporte de ventas diario   | Timer (2:00 AM)     | Dashboard de ventas del día anterior                                     |

### 🏥 Sector Salud

| Función                            | Trigger              | Qué hace                                                              |
|------------------------------------|----------------------|-----------------------------------------------------------------------|
| Procesar resultados de laboratorio | Blob Trigger         | Al subir un PDF de resultados, extrae texto y actualiza el expediente |
| Recordatorio de citas              | Timer (cada mañana)  | Envía SMS/email a pacientes con cita ese día                          |
| Alertas de stock de medicamentos   | Timer (cada 6 horas) | Notifica cuando el stock baja del mínimo                              |

---

## 🎥 11. Recursos y Videos Recomendados

| Recurso                                       | Descripción                                              | Dónde encontrarlo                                                                                     |
|-----------------------------------------------|----------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| **Microsoft Learn — Azure Functions**         | Ruta oficial con módulos interactivos gratuitos          | [learn.microsoft.com/azure/azure-functions](https://learn.microsoft.com/es-es/azure/azure-functions/) |
| **"Azure Functions Java Tutorial"**           | Crear funciones con Java paso a paso                     | Buscar en YouTube: `"Azure Functions Java tutorial Microsoft"`                                        |
| **"Serverless con Azure Functions"**          | Concepto serverless explicado en español                 | Buscar en YouTube: `"Azure Functions serverless español"`                                             |
| **"Timer Trigger y CRON en Azure Functions"** | Específico para funciones programadas                    | Buscar en YouTube: `"Azure Functions Timer Trigger CRON"`                                             |
| **Azure Functions Core Tools**                | Herramienta para ejecutar funciones localmente sin Azure | `npm install -g azure-functions-core-tools@4`                                                         |

> 🎯 **Puedes practicar sin cuenta Azure:**  
> Instala **Azure Functions Core Tools** con `npm install -g azure-functions-core-tools@4` y podrás crear y ejecutar
> funciones completamente en local. Para los triggers que necesitan Storage (Blob, Queue), combínalo con **Azurite** que
> instalaste en el módulo anterior.

### 📖 Documentación oficial clave

| Documento                              | URL                                                                                                                                                          |
|----------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Guía del desarrollador Java            | [learn.microsoft.com/azure/azure-functions/functions-reference-java](https://learn.microsoft.com/es-es/azure/azure-functions/functions-reference-java)       |
| Todos los tipos de Triggers y Bindings | [learn.microsoft.com/azure/azure-functions/functions-triggers-bindings](https://learn.microsoft.com/es-es/azure/azure-functions/functions-triggers-bindings) |
| Expresiones CRON en Azure Functions    | [learn.microsoft.com/azure/azure-functions/functions-bindings-timer](https://learn.microsoft.com/es-es/azure/azure-functions/functions-bindings-timer)       |
| Planes de hospedaje comparados         | [learn.microsoft.com/azure/azure-functions/functions-scale](https://learn.microsoft.com/es-es/azure/azure-functions/functions-scale)                         |

---

## 🧠 Resumen Ejecutivo

> Lo que deberías poder decir en una entrevista si te preguntan *"¿Qué es Azure Functions y cuándo lo usarías?"*

**Azure Functions** es el servicio serverless de Azure. Permite ejecutar pequeños fragmentos de código en respuesta a
eventos, sin provisionar ni gestionar servidores. El código solo vive mientras se está ejecutando — Azure crea y
destruye las instancias automáticamente, y el costo es por ejecución (número de llamadas y tiempo de CPU), no por tiempo
activo. El concepto central es el **Trigger**: el evento que dispara la función. Los más importantes son HTTP (petición
web), Timer (expresión CRON para tareas programadas), Blob (nuevo archivo en Azure Storage), Queue y Service Bus (
mensajes en colas) y Cosmos DB (cambios en documentos). Complementariamente, los **Bindings** permiten conectar la
función a otros servicios Azure de forma declarativa, sin escribir código de conexión. A diferencia de **App Service**,
donde vive una aplicación completa siempre activa, Azure Functions es ideal para tareas event-driven, esporádicas o
asíncronas: envío de emails, generación de reportes, procesamiento de imágenes, sincronización de datos. En un proyecto
real con Spring Boot, la app principal corre en App Service y las tareas secundarias asíncronas se delegan a Azure
Functions. El principal trade-off a conocer es el **Cold Start**: el tiempo de arranque inicial de una instancia nueva (
1-3 segundos en Java), que se mitiga con el plan Premium o compilación nativa con GraalVM.

---

## ⏭️ Siguiente módulo

> 📨 **Módulo 06 — Azure Service Bus:** El servicio de mensajería empresarial de Azure. Veremos qué es, la diferencia
> entre colas y topics, patrones pub/sub, dead-letter queues, y cómo integrarlo con Spring Boot para comunicación
> asíncrona entre microservicios — clave para arquitecturas reactivas con WebFlux.

---

*📝 Documentación elaborada como parte del plan de aprendizaje Azure para desarrolladores Java Backend.*  
*🗓️ Última actualización: 2025*
