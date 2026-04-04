# 📨 Módulo 06 — Azure Service Bus

> 📚 **Serie:** Azure para Java Developers  
> 🗂️ **Módulo:** 06 de 08  
> 🎯 **Objetivo:** Entender qué es Azure Service Bus, la diferencia entre colas y topics, patrones pub/sub, garantías de
> entrega, dead-letter queues, y cómo integrarlo con Spring Boot — incluyendo Spring WebFlux para procesamiento
> reactivo.

---

## 📋 Tabla de Contenidos

1. [¿Qué es la mensajería asíncrona?](#-1-qué-es-la-mensajería-asíncrona-el-contexto-previo)
2. [¿Qué es Azure Service Bus?](#-2-qué-es-azure-service-bus)
3. [Cómo está organizado](#-3-cómo-está-organizado-azure-service-bus)
4. [Colas vs Topics — La diferencia fundamental](#-4-colas-vs-topics--la-diferencia-fundamental)
5. [Conceptos clave y garantías](#-5-conceptos-clave-y-garantías)
6. [Dead-Letter Queue](#-6-dead-letter-queue-dlq--el-salvavidas-de-los-mensajes)
7. [Service Bus vs Queue Storage](#-7-azure-service-bus-vs-azure-queue-storage)
8. [Integración con Spring Boot](#-8-integración-con-spring-boot)
9. [Integración con Spring WebFlux](#-9-integración-con-spring-webflux-reactivo)
10. [Ejemplo de código completo](#-10-ejemplo-de-código-completo)
11. [Casos de uso reales](#-11-casos-de-uso-reales-en-el-mundo-laboral)
12. [Recursos y videos](#-12-recursos-y-videos-recomendados)
13. [Resumen ejecutivo](#-resumen-ejecutivo)

---

## 🔄 1. ¿Qué es la Mensajería Asíncrona? (El contexto previo)

Antes de hablar de Azure Service Bus, necesitas entender bien el problema que resuelve: **la comunicación asíncrona
entre servicios**.

### El problema de la comunicación síncrona

Cuando el Servicio A llama directamente al Servicio B (una llamada HTTP REST, por ejemplo), tienes **comunicación
síncrona**:

```
Servicio A ──── HTTP Request ────► Servicio B
               (espera...)
Servicio A ◄─── HTTP Response ──── Servicio B
```

Esto funciona bien en muchos casos, pero tiene problemas serios en arquitecturas de microservicios:

- 🔗 **Acoplamiento temporal:** Si el Servicio B está caído, el Servicio A también falla.
- ⏳ **Bloqueo:** El Servicio A espera bloqueado hasta que B responda.
- 📈 **Cascada de fallos:** Si B tarda mucho, A se queda sin threads disponibles.
- 🔁 **Reintentos manuales:** Si B falla, A tiene que implementar lógica de reintento.

### La solución: mensajería asíncrona

En lugar de llamarse directamente, los servicios se comunican a través de un **intermediario de mensajes (message
broker)**:

```
                     MESSAGE BROKER
                   ┌──────────────────┐
Servicio A ──────► │    📨 Mensaje    │ ──────► Servicio B
  (publica y       │    📨 Mensaje    │          (consume cuando
   sigue sin       │    📨 Mensaje    │           puede, a su
   esperar)        └──────────────────┘           propio ritmo)
```

**Ventajas inmediatas:**

- ✅ **Desacoplamiento:** A no sabe nada de B. Si B está caído, los mensajes esperan en la cola.
- ✅ **Sin bloqueo:** A publica y continúa — no espera respuesta.
- ✅ **Reintentos automáticos:** Si B falla al procesar, el broker reintenta automáticamente.
- ✅ **Balanceo de carga natural:** Múltiples instancias de B consumen de la misma cola.
- ✅ **Picos de tráfico absorbidos:** Si llegan 10,000 pedidos en 1 minuto, la cola los absorbe y B los procesa a su
  ritmo.

### La analogía del buzón de correo 📮

```
Comunicación síncrona (teléfono):
  "Llamo a Juan, espero que conteste, si no contesta fallo"

Comunicación asíncrona (correo/mensajería):
  "Dejo el mensaje en el buzón, Juan lo lee cuando pueda,
   si Juan no está en casa el mensaje igual llega"
```

> 💡 **En resumen:** La mensajería asíncrona desacopla productores y consumidores de mensajes en el tiempo y en el
> espacio. El mensaje broker actúa de intermediario garantizando que los mensajes lleguen aunque el consumidor esté
> temporalmente inactivo.

---

## 📨 2. ¿Qué es Azure Service Bus?

**Azure Service Bus** es el servicio de mensajería empresarial gestionado de Microsoft Azure. Actúa como intermediario
de mensajes confiable entre servicios y aplicaciones, garantizando que cada mensaje sea entregado, procesado exactamente
una vez y en el orden correcto, incluso si el consumidor falla.

### La versión larga (con contexto real)

Imagina el sistema de una empresa bancaria peruana que procesa pagos. Cuando un cliente hace una transferencia, el
sistema necesita:

1. Debitar la cuenta del emisor.
2. Acreditar la cuenta del receptor.
3. Notificar al emisor por email y SMS.
4. Registrar la transacción en el sistema de auditoría (SBS).
5. Actualizar el historial de movimientos en la app móvil.
6. Si es una transferencia interbancaria, notificar al BCRP.

Hacer todo eso en una sola transacción síncrona sería:

- Lento (el usuario espera a que todo se complete).
- Frágil (si falla la notificación al BCRP, ¿se revierte el débito?).
- Imposible de escalar (cada transferencia bloquea recursos hasta completar todos los pasos).

**Con Azure Service Bus:**

```
Cliente inicia transferencia
          │
          ▼
[Servicio de Transferencias] ──── Debita y Acredita (síncrono, crítico)
          │
          │  Publica mensaje en Service Bus:
          │  "TransferenciaCompletada: {id, monto, emisor, receptor}"
          │
          ▼
    [Service Bus]
          │
    ┌─────┼─────┬─────────┬──────────────┐
    ▼     ▼     ▼         ▼              ▼
[Email] [SMS] [Auditoría] [App Móvil] [BCRP]
   (cada servicio consume independientemente, a su propio ritmo)
```

El cliente recibe la confirmación de la transferencia en milisegundos. Todo lo demás ocurre de forma asíncrona y
confiable.

**¿Qué hace a Service Bus "empresarial"?**

- 🛡️ **Garantía de entrega:** Cada mensaje es entregado al menos una vez (at-least-once) o exactamente una vez
  (exactly-once).
- 📋 **Ordenamiento:** Los mensajes se pueden procesar en el orden exacto en que fueron enviados (FIFO).
- ⏱️ **Mensajes programados:** Puedes decirle a Service Bus "entrega este mensaje en 24 horas".
- 🔁 **Reintentos automáticos:** Si el consumidor falla, Service Bus reintenta automáticamente con backoff exponencial.
- ☠️ **Dead-Letter Queue:** Los mensajes que no se pueden procesar van a una cola especial para análisis.
- 📏 **Mensajes grandes:** Hasta 100 MB por mensaje (con Premium).
- 🔐 **Seguridad:** Cifrado en tránsito y reposo, autenticación con Azure Active Directory.

> 💡 **En resumen:** Azure Service Bus es el intermediario de mensajes confiable de Azure. Garantiza que los mensajes
> entre tus microservicios lleguen, se procesen correctamente y no se pierdan, incluso ante fallos parciales del
> sistema.

---

## 🏗️ 3. Cómo está organizado Azure Service Bus

```
┌──────────────────────────────────────────────────────────────────┐
│                    Namespace de Service Bus                      │
│           miempresa.servicebus.windows.net                       │
│                                                                  │
│   ┌──────────────────────────┐  ┌────────────────────────────┐   │
│   │          COLA            │  │          TOPIC             │   │
│   │    "pedidos-nuevos"      │  │    "transferencias"        │   │
│   │                          │  │                            │   │
│   │  Un solo consumidor      │  │  ┌──────────────────────┐  │   │
│   │  (o grupo balanceado)    │  │  │ Suscripción: "email" │  │   │
│   │                          │  │  └──────────────────────┘  │   │
│   │  📨 → [Consumidor]       │  │  ┌───────────────────────┐ │   │ 
│   │                          │  │  │ Suscripción: "sms"    │ │   │
│   └──────────────────────────┘  │  └───────────────────────┘ │   │
│                                 │  ┌───────────────────────┐ │   │
│                                 │  │ Suscripción:"auditoria│ │   │
│                                 │  └───────────────────────┘ │   │
│                                 └────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

| Concepto         | Descripción                                                                             |
|------------------|-----------------------------------------------------------------------------------------|
| **Namespace**    | Contenedor de alto nivel. Define la URL de acceso y el tier (Basic, Standard, Premium). |
| **Cola (Queue)** | Canal de mensajes punto a punto. Un productor, un consumidor (o grupo).                 |
| **Topic**        | Canal pub/sub. Un productor, múltiples suscriptores independientes.                     |
| **Suscripción**  | Copia del topic para cada consumidor. Cada suscripción recibe todos los mensajes.       |
| **Mensaje**      | La unidad de datos. Tiene un cuerpo (body) y metadatos (properties).                    |

---

## 🔀 4. Colas vs Topics — La diferencia fundamental

Esta es la distinción más importante de Service Bus y la que más se pregunta en entrevistas.

### 📥 Cola (Queue) — Punto a Punto

**Un mensaje llega a UN solo consumidor.** Si hay múltiples instancias del consumidor, el mensaje va a una de ellas (
balanceo de carga automático).

```
                         COLA: "pedidos-nuevos"
                        ┌──────────────────────┐
Servicio de Pedidos ──► │  📨  📨  📨  📨  📨 │ ──► Procesador de Pedidos
    (productor)         └──────────────────────┘      (consumidor — un solo servicio
                                                        puede haber varias instancias
                                                        pero cada mensaje va a una sola)
```

**¿Cuándo usar Queue?**

- Procesar tareas en background (generar PDFs, enviar emails).
- Distribuir trabajo entre múltiples workers (cada pedido lo procesa un solo worker).
- Comunicación entre dos microservicios específicos.

### 📢 Topic — Publicar/Suscribir (Pub/Sub)

**Un mensaje llega a TODOS los suscriptores.** Cada suscripción recibe una copia independiente del mensaje. Los
suscriptores no saben nada los unos de los otros.

```
                          TOPIC: "transferencias"
                         ┌────────────────────────┐
                         │                        │──► [Suscripción: email]     ──► Servicio Email
Servicio de         ──►  │       📨 Mensaje       │──► [Suscripción: sms]       ──► Servicio SMS
Transferencias           │                        │──► [Suscripción: auditoria] ──► Servicio Auditoría
    (productor)          └────────────────────────┘──► [Suscripción: app-movil] ──► Servicio App Móvil
```

**¿Cuándo usar Topic?**

- Un evento que múltiples sistemas necesitan conocer (una transferencia, un nuevo usuario, etc.).
- Arquitecturas event-driven donde no sabes de antemano quién consume el evento.
- Cuando quieres agregar nuevos consumidores sin modificar el productor.

### La diferencia visual

```
COLA (uno a uno):                    TOPIC (uno a muchos):

Productor ──► [Cola] ──► Consumidor  Productor ──► [Topic] ──► Suscripción A ──► Consumidor A
                                                          └──► Suscripción B ──► Consumidor B
                                                          └──► Suscripción C ──► Consumidor C
```

> 📌 **Regla práctica:**
> - ¿Un solo servicio necesita procesar el mensaje? → **Cola**
> - ¿Múltiples servicios necesitan reaccionar al mismo evento? → **Topic**

---

## 🔑 5. Conceptos Clave y Garantías

### 📋 Modos de recepción de mensajes

#### Peek-Lock (el modo recomendado)

El mensaje se "bloquea" temporalmente para el consumidor. Mientras lo procesa, ningún otro consumidor puede tomarlo. Al
terminar el procesamiento, el consumidor debe confirmar (complete) o rechazar (abandon).

```
Consumidor solicita mensaje
        │
        ▼
Service Bus bloquea el mensaje (lock duration: 60 segundos por defecto)
        │
        ▼
Consumidor procesa el mensaje
        │
   ┌────┴────┐
   ▼         ▼
Éxito     Falla
   │         │
   ▼         ▼
Complete   Abandon / DeadLetter
(eliminado  (vuelve a la cola
 de la cola) para reintento)
```

#### Receive and Delete (no recomendado en producción)

El mensaje se elimina de la cola en el momento en que el consumidor lo recibe. Si el consumidor falla antes de
procesarlo, el mensaje se pierde para siempre.

### 🔢 Delivery Count (contador de entregas)

Cada mensaje tiene un contador que registra cuántas veces ha sido entregado. Cuando supera el máximo configurado (por
defecto 10), el mensaje va automáticamente a la **Dead-Letter Queue**.

```
Entrega 1 → Consumidor falla → Abandon
Entrega 2 → Consumidor falla → Abandon
...
Entrega 10 → Consumidor falla → Abandon
Entrega 11 → Service Bus mueve a Dead-Letter Queue automáticamente ☠️
```

### ⏱️ Message TTL (Time To Live)

Cada mensaje tiene un tiempo de vida máximo. Si nadie lo consume en ese tiempo, se mueve a la Dead-Letter Queue.
Configurable por namespace, cola/topic, o mensaje individual.

```properties
# Tiempo de vida: 1 día (formato ISO 8601)
spring.jms.servicebus.idle-timeout=PT24H
```

### 📅 Mensajes Programados (Scheduled Messages)

Puedes instruir a Service Bus para que entregue un mensaje en una fecha y hora específica en el futuro:

```bash
// Programar un mensaje para entregarse en 24 horas
OffsetDateTime tiempoEntrega = OffsetDateTime.now().plusHours(24);
sender.scheduleMessage(mensaje, tiempoEntrega);
```

**Caso de uso real:** Cuando un usuario crea una cuenta, programas un mensaje para 3 días después que dispare el envío
de un email de seguimiento si no ha completado su perfil.

### 🔒 Sessions (Sesiones) — Procesamiento ordenado

Las sesiones garantizan que todos los mensajes con el mismo `SessionId` sean procesados por el mismo consumidor y en el
orden en que llegaron. Esencial para casos donde el orden importa.

```
Mensajes del cliente CLI-001:
  Msg1(sessionId=CLI-001) → Siempre al mismo consumidor, en orden
  Msg2(sessionId=CLI-001) → Garantizado: después del Msg1
  Msg3(sessionId=CLI-001) → Garantizado: después del Msg2
```

**Caso de uso:** Transacciones bancarias de un mismo cliente — el orden de débitos y créditos importa para el saldo.

---

## ☠️ 6. Dead-Letter Queue (DLQ) — El salvavidas de los mensajes

La **Dead-Letter Queue** (DLQ) es una cola especial, adjunta a cada cola o suscripción, donde van los mensajes que no
pudieron ser procesados correctamente.

### ¿Cuándo va un mensaje a la DLQ?

```
┌─────────────────────────────────────────────────────┐
│          Razones para ir a la DLQ                   │
│                                                     │
│  1. Superó el máximo de entregas (delivery count)   │
│  2. Expiró el TTL sin ser consumido                 │
│  3. El consumidor lo envió explícitamente a DLQ     │
│  4. El mensaje tiene un formato inválido            │
│  5. Filtros de suscripción no coinciden             │
└─────────────────────────────────────────────────────┘
```

### ¿Por qué es importante?

Sin DLQ, un mensaje que no puede procesarse se perdería para siempre. Con DLQ:

- Los mensajes problemáticos se preservan para análisis.
- Un equipo de operaciones puede revisarlos y decidir si reenviarlos o descartarlos.
- Puedes identificar patrones de error (ej: todos los mensajes de un cliente específico fallan).
- Nadie pierde datos críticos aunque haya un bug en el consumidor.

### Acceder a la DLQ desde código

```bash
// La DLQ tiene un nombre especial: nombreCola/$DeadLetterQueue
String dlqName = "pedidos-nuevos/$DeadLetterQueue";

ServiceBusReceiverClient dlqReceiver = new ServiceBusClientBuilder()
    .connectionString(connectionString)
    .receiver()
    .queueName(dlqName)
    .buildClient();

// Procesar mensajes de la DLQ
dlqReceiver.receiveMessages(10).forEach(mensaje -> {
    System.out.println("Mensaje en DLQ: " + mensaje.getBody());
    System.out.println("Razón: " + mensaje.getDeadLetterReason());
    System.out.println("Descripción: " + mensaje.getDeadLetterErrorDescription());

    // Decidir: reenviar a la cola original o descartar
    dlqReceiver.complete(mensaje);
});
```

> 📌 **Buena práctica:** En producción siempre monitorea la DLQ. Si empieza a acumular mensajes, es una señal de que
> algo está fallando en tu sistema. Configura alertas en Azure Monitor para notificarte cuando la DLQ supere N mensajes.

---

## ⚖️ 7. Azure Service Bus vs Azure Queue Storage

Una pregunta frecuente en entrevistas: *"¿Cuándo usarías Service Bus en lugar de Queue Storage del módulo anterior?"*

| Característica                 | 📨 Azure Service Bus                     | 📦 Azure Queue Storage                  |
|--------------------------------|------------------------------------------|-----------------------------------------|
| **Tamaño máximo de mensaje**   | 256 KB (Standard) / 100 MB (Premium)     | 64 KB                                   |
| **Tiempo máximo de retención** | 14 días                                  | 7 días                                  |
| **Garantía de orden (FIFO)**   | ✅ Sí (con Sessions)                      | ❌ No garantizado                        |
| **Topics / Pub-Sub**           | ✅ Sí                                     | ❌ No                                    |
| **Dead-Letter Queue**          | ✅ Nativa                                 | ❌ No existe                             |
| **Exactly-once delivery**      | ✅ Sí (Peek-Lock)                         | ❌ At-least-once solo                    |
| **Mensajes programados**       | ✅ Sí                                     | ❌ No                                    |
| **Sesiones (orden por grupo)** | ✅ Sí                                     | ❌ No                                    |
| **Filtros en suscripciones**   | ✅ Sí                                     | ❌ No                                    |
| **Transacciones**              | ✅ Sí                                     | ❌ No                                    |
| **Precio base**                | ~$10/mes (Standard)                      | Muy bajo (por operación)                |
| **Complejidad**                | Media-Alta                               | Baja                                    |
| **¿Cuándo usarlo?**            | Sistemas críticos, microservicios, banca | Scripts simples, desacoplamiento básico |

> 💡 **Regla simple:**
> - Necesitas topics, DLQ, orden garantizado o mensajes grandes → **Service Bus**
> - Solo necesitas una cola simple y barata entre dos componentes → **Queue Storage**

---

## ☕ 8. Integración con Spring Boot

### Tiers disponibles

| Tier         | Características                                                    | Precio                       |
|--------------|--------------------------------------------------------------------|------------------------------|
| **Basic**    | Solo colas, sin topics                                             | ~$0.05/millón de operaciones |
| **Standard** | Colas + Topics, mensajes hasta 256 KB                              | ~$10/mes base                |
| **Premium**  | Todo + mensajes hasta 100 MB, red privada, rendimiento garantizado | ~$677/mes por unidad         |

> 📌 Para desarrollo y aprendizaje, el tier **Standard** es suficiente.

### Dependencias en `pom.xml`

```xml

<dependencies>
    <!-- Spring Boot Starter para Azure Service Bus con JMS -->
    <!-- JMS (Java Message Service) es la API estándar de Java para mensajería -->
    <dependency>
        <groupId>com.azure.spring</groupId>
        <artifactId>spring-cloud-azure-starter-servicebus-jms</artifactId>
        <version>5.8.0</version>
    </dependency>

    <!-- Alternativa: SDK nativo de Azure Service Bus (más control, más verboso) -->
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-messaging-servicebus</artifactId>
        <version>7.15.0</version>
    </dependency>

    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Jackson — serialización JSON -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
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
# Configuración de Azure Service Bus
# Los valores reales vienen de Azure Application Settings
# ============================================================
# Connection string del namespace de Service Bus
spring.jms.servicebus.connection-string=${SERVICE_BUS_CONNECTION_STRING}
# Tier del namespace (basic no soporta topics — usar standard o premium)
spring.jms.servicebus.pricing-tier=standard
# Número máximo de mensajes procesados en paralelo por consumidor
spring.jms.listener.concurrency=5
spring.jms.listener.max-concurrency=10
# Modo de reconocimiento — CLIENT_ACKNOWLEDGE para control manual (recomendado)
spring.jms.listener.acknowledge-mode=client
# Tiempo máximo de bloqueo de un mensaje (lock duration)
# Debe ser mayor que el tiempo máximo de procesamiento
spring.jms.servicebus.idle-timeout=PT2M
```

### `ServiceBusConfig.java` — Configuración del cliente nativo

```java
package com.ejemplo.servicebus.config;

import com.azure.messaging.servicebus.ServiceBusClientBuilder;
import com.azure.messaging.servicebus.ServiceBusSenderClient;
import com.azure.messaging.servicebus.ServiceBusProcessorClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ServiceBusConfig {

    @Value("${SERVICE_BUS_CONNECTION_STRING}")
    private String connectionString;

    /**
     * Cliente para ENVIAR mensajes a la cola "pedidos-nuevos".
     * Es thread-safe — se registra como singleton.
     */
    @Bean(name = "pedidosSender")
    public ServiceBusSenderClient pedidosSenderClient() {
        return new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .sender()
                .queueName("pedidos-nuevos")
                .buildClient();
    }

    /**
     * Cliente para ENVIAR mensajes al topic "transferencias".
     */
    @Bean(name = "transferenciasSender")
    public ServiceBusSenderClient transferenciasSenderClient() {
        return new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .sender()
                .topicName("transferencias")
                .buildClient();
    }
}
```

---

## 🌊 9. Integración con Spring WebFlux (Reactivo)

Este punto es especialmente relevante para tu perfil. Azure Service Bus y Spring WebFlux se complementan perfectamente
para crear sistemas reactivos y no bloqueantes.

### ¿Por qué WebFlux + Service Bus?

```
SIN WEBFLUX (bloqueante):
Thread 1 ──► Recibe mensaje ──► Procesa (espera I/O) ──► Siguiente mensaje
Thread 2 ──► Recibe mensaje ──► Procesa (espera I/O) ──► Siguiente mensaje
... (necesitas muchos threads para alta concurrencia)

CON WEBFLUX (reactivo / no bloqueante):
Thread 1 ──► Recibe mensaje ──► Inicia proceso ──► Libera thread
              ◄── Continúa cuando I/O termina ──────────────────
Thread 1 ──► Recibe siguiente mensaje (¡mismo thread!) ...
... (pocos threads, alta concurrencia)
```

### Dependencia adicional para WebFlux

```xml
<!-- Spring WebFlux para procesamiento reactivo -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

        <!-- Reactor — implementación de Reactive Streams -->
<dependency>
<groupId>io.projectreactor</groupId>
<artifactId>reactor-core</artifactId>
</dependency>
```

### Configuración del procesador reactivo

```java
package com.ejemplo.servicebus.config;

import com.azure.messaging.servicebus.*;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import reactor.core.publisher.Mono;

import java.util.function.Consumer;

@Configuration
public class ServiceBusReactivoConfig {

    @Value("${SERVICE_BUS_CONNECTION_STRING}")
    private String connectionString;

    /**
     * ServiceBusProcessorClient con procesamiento reactivo.
     *
     * A diferencia del cliente bloqueante, este procesa mensajes
     * de forma asíncrona usando lambdas para éxito y error.
     */
    @Bean
    public ServiceBusProcessorClient procesadorReactivo(
            Consumer<ServiceBusReceivedMessageContext> procesadorMensaje,
            Consumer<ServiceBusErrorContext> manejadorError) {

        return new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .processor()
                .queueName("pedidos-nuevos")
                .maxConcurrentCalls(10)      // Procesar hasta 10 mensajes en paralelo
                .disableAutoComplete()        // Control manual del acknowledge
                .processMessage(procesadorMensaje)
                .processError(manejadorError)
                .buildProcessorClient();
    }
}
```

---

## 💻 10. Ejemplo de Código Completo

### Escenario

Sistema de transferencias bancarias entre microservicios. El **Servicio de Transferencias** publica un evento en un
Topic de Service Bus. Tres servicios independientes reaccionan: notificaciones (email/SMS), auditoría y actualización
del saldo en la app móvil. Todo usando Spring Boot con integración reactiva donde aplica.

### Estructura del proyecto

```
mi-app-servicebus/
└── src/main/java/com/ejemplo/servicebus/
    ├── config/
    │   ├── ServiceBusConfig.java
    │   └── JmsConfig.java
    ├── model/
    │   └── TransferenciaEvent.java
    ├── producer/
    │   └── TransferenciaProducer.java
    ├── consumer/
    │   ├── NotificacionConsumer.java
    │   ├── AuditoriaConsumer.java
    │   └── AppMovilConsumer.java
    └── controller/
        └── TransferenciaController.java
```

### `TransferenciaEvent.java` — Modelo del evento

```java
package com.ejemplo.servicebus.model;

import com.fasterxml.jackson.annotation.JsonProperty;

import java.time.LocalDateTime;

/**
 * Representa el evento que se publica en Service Bus
 * cuando una transferencia es completada.
 * Este mismo objeto lo leen todos los suscriptores del topic.
 */
public class TransferenciaEvent {

    @JsonProperty("transferenciaId")
    private String transferenciaId;

    @JsonProperty("cuentaOrigen")
    private String cuentaOrigen;

    @JsonProperty("cuentaDestino")
    private String cuentaDestino;

    @JsonProperty("monto")
    private double monto;

    @JsonProperty("moneda")
    private String moneda;

    @JsonProperty("emailCliente")
    private String emailCliente;

    @JsonProperty("telefonoCliente")
    private String telefonoCliente;

    @JsonProperty("timestamp")
    private LocalDateTime timestamp;

    @JsonProperty("estado")
    private String estado;

    // Constructor vacío para Jackson
    public TransferenciaEvent() {
    }

    public TransferenciaEvent(String transferenciaId, String cuentaOrigen,
                              String cuentaDestino, double monto, String moneda,
                              String emailCliente, String telefonoCliente) {
        this.transferenciaId = transferenciaId;
        this.cuentaOrigen = cuentaOrigen;
        this.cuentaDestino = cuentaDestino;
        this.monto = monto;
        this.moneda = moneda;
        this.emailCliente = emailCliente;
        this.telefonoCliente = telefonoCliente;
        this.timestamp = LocalDateTime.now();
        this.estado = "COMPLETADA";
    }

    /* Getters y Setters */
}
```

### `TransferenciaProducer.java` — Publicador de eventos

```java
package com.ejemplo.servicebus.producer;

import com.azure.messaging.servicebus.ServiceBusMessage;
import com.azure.messaging.servicebus.ServiceBusSenderClient;
import com.ejemplo.servicebus.model.TransferenciaEvent;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;

@Service
public class TransferenciaProducer {

    @Autowired
    @Qualifier("transferenciasSender")
    private ServiceBusSenderClient senderClient;

    @Autowired
    private ObjectMapper objectMapper;

    /**
     * Publica un evento de transferencia en el Topic de Service Bus.
     *
     * Todos los suscriptores del topic "transferencias" recibirán
     * una copia de este mensaje de forma independiente:
     *   - Suscripción "notificaciones"  → Envía email y SMS
     *   - Suscripción "auditoria"       → Registra para SBS
     *   - Suscripción "app-movil"       → Actualiza historial en tiempo real
     *
     * @param evento  El evento de transferencia completada.
     */
    public void publicarTransferenciaCompletada(TransferenciaEvent evento) {
        try {
            // 1. Serializar el evento a JSON
            String eventoJson = objectMapper.writeValueAsString(evento);

            // 2. Crear el mensaje de Service Bus
            ServiceBusMessage mensaje = new ServiceBusMessage(eventoJson);

            // 3. Agregar propiedades de metadatos al mensaje
            //    Estas propiedades pueden usarse en filtros de suscripción
            mensaje.getApplicationProperties().put("tipo", "TRANSFERENCIA_COMPLETADA");
            mensaje.getApplicationProperties().put("moneda", evento.getMoneda());
            mensaje.getApplicationProperties().put("monto", evento.getMonto());

            // 4. Establecer el ID del mensaje (útil para deduplicación)
            mensaje.setMessageId(evento.getTransferenciaId());

            // 5. Enviar al topic
            senderClient.sendMessage(mensaje);

            System.out.println("✅ Evento publicado en Service Bus — ID: " +
                               evento.getTransferenciaId());
            System.out.println("   Monto: " + evento.getMoneda() + " " + evento.getMonto());

        } catch (JsonProcessingException e) {
            throw new RuntimeException("Error serializando evento de transferencia", e);
        }
    }

    /**
     * Versión reactiva del publicador — retorna Mono<Void> para integración con WebFlux.
     * Úsala cuando el método que la llama también es reactivo.
     */
    public Mono<Void> publicarTransferenciaReactivo(TransferenciaEvent evento) {
        return Mono.fromRunnable(() -> publicarTransferenciaCompletada(evento))
                .then();
    }
}
```

### `NotificacionConsumer.java` — Consumidor de la suscripción "notificaciones"

```java
package com.ejemplo.servicebus.consumer;

import com.ejemplo.servicebus.model.TransferenciaEvent;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

/**
 * Escucha la suscripción "notificaciones" del topic "transferencias".
 *
 * @JmsListener conecta automáticamente con Azure Service Bus usando
 * la configuración de spring.jms.servicebus.connection-string.
 *
 * Este consumidor es INDEPENDIENTE de los otros consumidores.
 * Si este falla, los demás siguen funcionando.
 */
@Component
public class NotificacionConsumer {

    @Autowired
    private ObjectMapper objectMapper;

    @JmsListener(
            destination = "transferencias",       // Nombre del topic
            subscription = "notificaciones",       // Nombre de la suscripción
            containerFactory = "topicJmsListenerContainerFactory"
    )
    public void procesarNotificacion(String mensajeJson) {
        try {
            TransferenciaEvent evento = objectMapper.readValue(
                    mensajeJson, TransferenciaEvent.class
            );

            System.out.println("📧 [NotificacionConsumer] Procesando transferencia: " +
                               evento.getTransferenciaId());

            // 1. Enviar email de confirmación
            enviarEmail(evento);

            // 2. Enviar SMS
            enviarSms(evento);

            System.out.println("✅ [NotificacionConsumer] Notificaciones enviadas para: " +
                               evento.getTransferenciaId());

        } catch (Exception e) {
            // Si lanzamos excepción, Service Bus incrementa el delivery count
            // y reintentará el mensaje automáticamente.
            // Si supera el máximo de reintentos, va a la Dead-Letter Queue.
            System.err.println("❌ [NotificacionConsumer] Error: " + e.getMessage());
            throw new RuntimeException("Error procesando notificación", e);
        }
    }

    private void enviarEmail(TransferenciaEvent evento) {
        // En producción: integración con SendGrid, AWS SES, etc.
        System.out.println("   📧 Email enviado a: " + evento.getEmailCliente() +
                           " — Transferencia de " + evento.getMoneda() +
                           " " + evento.getMonto());
    }

    private void enviarSms(TransferenciaEvent evento) {
        // En producción: integración con Twilio, AWS SNS, etc.
        System.out.println("   📱 SMS enviado a: " + evento.getTelefonoCliente() +
                           " — Transferencia confirmada: " + evento.getMonto());
    }
}
```

### `AuditoriaConsumer.java` — Consumidor de la suscripción "auditoria"

```java
package com.ejemplo.servicebus.consumer;

import com.ejemplo.servicebus.model.TransferenciaEvent;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * Escucha la suscripción "auditoria" del topic "transferencias".
 *
 * Registra cada transferencia en el sistema de auditoría para
 * cumplimiento regulatorio (SBS en el caso peruano).
 *
 * Completamente independiente de NotificacionConsumer —
 * si el servicio de notificaciones falla, la auditoría sigue funcionando.
 */
@Component
public class AuditoriaConsumer {

    @Autowired
    private ObjectMapper objectMapper;

    @JmsListener(
            destination = "transferencias",
            subscription = "auditoria",
            containerFactory = "topicJmsListenerContainerFactory"
    )
    public void registrarAuditoria(String mensajeJson) {
        try {
            TransferenciaEvent evento = objectMapper.readValue(
                    mensajeJson, TransferenciaEvent.class
            );

            System.out.println("🔍 [AuditoriaConsumer] Registrando para auditoría: " +
                               evento.getTransferenciaId());

            // En producción: guardar en base de datos de auditoría inmutable
            // (Azure SQL con LEDGER tables o Cosmos DB append-only)
            registrarEnBD(evento);

            System.out.println("✅ [AuditoriaConsumer] Auditoría registrada: " +
                               evento.getTransferenciaId());

        } catch (Exception e) {
            System.err.println("❌ [AuditoriaConsumer] Error: " + e.getMessage());
            throw new RuntimeException("Error en auditoría", e);
        }
    }

    private void registrarEnBD(TransferenciaEvent evento) {
        // Registro de auditoría — en producción va a una tabla inmutable
        System.out.println("   📋 Registro de auditoría:");
        System.out.println("      ID:       " + evento.getTransferenciaId());
        System.out.println("      Origen:   " + evento.getCuentaOrigen());
        System.out.println("      Destino:  " + evento.getCuentaDestino());
        System.out.println("      Monto:    " + evento.getMoneda() + " " + evento.getMonto());
        System.out.println("      Timestamp:" + LocalDateTime.now());
    }
}
```

### `AppMovilConsumer.java` — Consumidor reactivo con WebFlux

```java
package com.ejemplo.servicebus.consumer;

import com.ejemplo.servicebus.model.TransferenciaEvent;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Mono;

/**
 * Escucha la suscripción "app-movil" del topic "transferencias".
 *
 * Usa Spring WebFlux (Mono/Flux) internamente para las operaciones
 * de I/O — simulando llamadas reactivas a servicios externos como
 * push notifications o actualización de caché reactivo (Redis).
 *
 * Este es el patrón que verás en empresas que usan WebFlux:
 * el listener es síncrono (requerimiento de JMS) pero internamente
 * usa reactive programming para las operaciones de I/O.
 */
@Component
public class AppMovilConsumer {

    @Autowired
    private ObjectMapper objectMapper;

    @JmsListener(
            destination = "transferencias",
            subscription = "app-movil",
            containerFactory = "topicJmsListenerContainerFactory"
    )
    public void actualizarAppMovil(String mensajeJson) {
        try {
            TransferenciaEvent evento = objectMapper.readValue(
                    mensajeJson, TransferenciaEvent.class
            );

            System.out.println("📱 [AppMovilConsumer] Actualizando app para: " +
                               evento.getTransferenciaId());

            // Uso de WebFlux internamente: encadenamos operaciones reactivas
            // y bloqueamos al final porque JMS requiere método síncrono
            procesarReactivo(evento)
                    .doOnSuccess(v -> System.out.println(
                            "✅ [AppMovilConsumer] App actualizada: " + evento.getTransferenciaId()
                    ))
                    .doOnError(e -> System.err.println(
                            "❌ [AppMovilConsumer] Error: " + e.getMessage()
                    ))
                    .block(); // Necesario porque JMS no soporta async nativo

        } catch (Exception e) {
            throw new RuntimeException("Error actualizando app móvil", e);
        }
    }

    /**
     * Cadena reactiva para actualizar la app móvil.
     * En producción: llamadas a Firebase FCM, actualización de Redis, etc.
     */
    private Mono<Void> procesarReactivo(TransferenciaEvent evento) {
        return enviarPushNotification(evento)
                .then(actualizarHistorialMovimientos(evento))
                .then(actualizarSaldoEnCache(evento));
    }

    private Mono<Void> enviarPushNotification(TransferenciaEvent evento) {
        return Mono.fromRunnable(() ->
                System.out.println("   🔔 Push notification enviada — " +
                                   "Transferencia: " + evento.getMonto() + " " + evento.getMoneda())
        );
    }

    private Mono<Void> actualizarHistorialMovimientos(TransferenciaEvent evento) {
        return Mono.fromRunnable(() ->
                System.out.println("   📜 Historial de movimientos actualizado — " +
                                   "Cuenta: " + evento.getCuentaOrigen())
        );
    }

    private Mono<Void> actualizarSaldoEnCache(TransferenciaEvent evento) {
        return Mono.fromRunnable(() ->
                System.out.println("   💰 Caché de saldo actualizado — " +
                                   "Cuenta: " + evento.getCuentaOrigen())
        );
    }
}
```

### `TransferenciaController.java` — API REST que dispara el flujo

```java
package com.ejemplo.servicebus.controller;

import com.ejemplo.servicebus.model.TransferenciaEvent;
import com.ejemplo.servicebus.producer.TransferenciaProducer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Mono;

import java.util.Map;
import java.util.UUID;

@RestController
@RequestMapping("/api/transferencias")
public class TransferenciaController {

    @Autowired
    private TransferenciaProducer producer;

    /**
     * POST /api/transferencias
     *
     * Versión síncrona — para Spring Boot MVC tradicional.
     * Procesa la transferencia, publica en Service Bus y retorna
     * respuesta al cliente sin esperar a que los consumidores terminen.
     */
    @PostMapping
    public ResponseEntity<Map<String, Object>> realizarTransferencia(
            @RequestBody Map<String, Object> request) {

        String transferenciaId = UUID.randomUUID().toString();

        TransferenciaEvent evento = new TransferenciaEvent(
                transferenciaId,
                (String) request.get("cuentaOrigen"),
                (String) request.get("cuentaDestino"),
                Double.parseDouble(request.get("monto").toString()),
                (String) request.getOrDefault("moneda", "PEN"),
                (String) request.get("emailCliente"),
                (String) request.get("telefonoCliente")
        );

        // Publicar evento en Service Bus (asíncrono — no esperamos consumidores)
        producer.publicarTransferenciaCompletada(evento);

        // Respuesta inmediata al cliente — sin esperar email, SMS, auditoría, etc.
        return ResponseEntity.ok(Map.of(
                "transferenciaId", transferenciaId,
                "estado", "PROCESADA",
                "mensaje", "Transferencia procesada. Recibirás confirmación en breve."
        ));
    }

    /**
     * POST /api/transferencias/reactivo
     *
     * Versión reactiva — para Spring WebFlux.
     * Retorna Mono<ResponseEntity> para procesamiento no bloqueante.
     */
    @PostMapping("/reactivo")
    public Mono<ResponseEntity<Map<String, Object>>> realizarTransferenciaReactivo(
            @RequestBody Map<String, Object> request) {

        String transferenciaId = UUID.randomUUID().toString();

        TransferenciaEvent evento = new TransferenciaEvent(
                transferenciaId,
                (String) request.get("cuentaOrigen"),
                (String) request.get("cuentaDestino"),
                Double.parseDouble(request.get("monto").toString()),
                (String) request.getOrDefault("moneda", "PEN"),
                (String) request.get("emailCliente"),
                (String) request.get("telefonoCliente")
        );

        // Publicar reactivamente y retornar respuesta al cliente
        return producer.publicarTransferenciaReactivo(evento)
                .thenReturn(ResponseEntity.ok(Map.<String, Object>of(
                        "transferenciaId", transferenciaId,
                        "estado", "PROCESADA",
                        "mensaje", "Transferencia procesada exitosamente"
                )));
    }
}
```

### Flujo completo del ejemplo

```
POST /api/transferencias
        │
        ▼
[TransferenciaController]
        │ Crea TransferenciaEvent
        │
        ▼
[TransferenciaProducer]
        │ Serializa a JSON
        │ Publica en Topic "transferencias"
        │
        ▼
[Service Bus — Topic: "transferencias"]
        │
   ─────┼──────────────────────────────────
   │              │                        │
   ▼              ▼                        ▼
[Suscripción:  [Suscripción:         [Suscripción:
 notificaciones] auditoria]           app-movil]
   │              │                        │
   ▼              ▼                        ▼
[Notification  [Auditoria           [AppMovil
 Consumer]      Consumer]            Consumer]
   │              │                        │
   ├─ Email       └─ BD Auditoría          ├─ Push Notification
   └─ SMS           (SBS)                  ├─ Historial Movimientos
                                           └─ Caché Saldo

(Cada consumidor es independiente — si uno falla, los demás siguen)
```

---

## 🏢 11. Casos de Uso Reales en el Mundo Laboral

### 🏦 Sector Bancario / Fintech

| Escenario                       | Patrón                   | Detalle                                                  |
|---------------------------------|--------------------------|----------------------------------------------------------|
| Notificación de movimientos     | Topic + 3 suscripciones  | Email, SMS y push por cada débito/crédito                |
| Procesamiento de lotes de pagos | Cola + múltiples workers | Distribución de carga entre instancias                   |
| Fraude detectado                | Topic                    | Bloqueo de cuenta + notificación + auditoría simultáneos |
| Transferencias interbancarias   | Cola con Sessions        | Orden garantizado por cuenta de cliente                  |

### 🛒 E-commerce / Retail

| Escenario                       | Patrón       | Detalle                                                             |
|---------------------------------|--------------|---------------------------------------------------------------------|
| Nuevo pedido                    | Topic        | Email confirmación + actualizar inventario + notificar almacén      |
| Actualización masiva de precios | Cola         | Miles de actualizaciones distribuidas entre workers                 |
| Devolución de producto          | Cola con DLQ | Si falla el reembolso, el mensaje queda en DLQ para revisión manual |

### 🏥 Sector Salud

| Escenario                      | Patrón       | Detalle                                                               |
|--------------------------------|--------------|-----------------------------------------------------------------------|
| Resultado de laboratorio listo | Topic        | Notificar médico + actualizar expediente + alertar al paciente        |
| Cita médica confirmada         | Cola + Timer | Confirmación inmediata + recordatorio programado para el día anterior |

---

## 🎥 12. Recursos y Videos Recomendados

| Recurso                               | Descripción                                                                  | Dónde encontrarlo                                                                                                 |
|---------------------------------------|------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| **Microsoft Learn — Service Bus**     | Ruta oficial con ejercicios interactivos                                     | [learn.microsoft.com/azure/service-bus-messaging](https://learn.microsoft.com/es-es/azure/service-bus-messaging/) |
| **"Azure Service Bus Tutorial"**      | Colas y topics explicados con demo                                           | Buscar en YouTube: `"Azure Service Bus tutorial queues topics"`                                                   |
| **"Spring Boot + Azure Service Bus"** | Integración completa con Spring                                              | Buscar en YouTube: `"Spring Boot Azure Service Bus Microsoft"`                                                    |
| **Service Bus Explorer**              | Herramienta visual para explorar colas y topics (como SSMS para Service Bus) | Buscar: `"Service Bus Explorer GitHub" (peterbarberliabr)`                                                        |

> 🎯 **Practica sin cuenta Azure:**  
> Instala **Service Bus Explorer** — una herramienta open source gratuita que te permite conectarte a un namespace real
> de Service Bus y ver/enviar/recibir mensajes visualmente. Para el emulador local, busca **"Azure Service Bus Emulator"
** — Microsoft lanzó un emulador oficial en 2024 que puedes correr con Docker.

### 📖 Documentación oficial clave

| Documento                            | URL                                                                                                                                                                                                                                                              |
|--------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Introducción a Azure Service Bus     | [learn.microsoft.com/azure/service-bus-messaging/service-bus-messaging-overview](https://learn.microsoft.com/es-es/azure/service-bus-messaging/service-bus-messaging-overview)                                                                                   |
| Spring Cloud Azure — Service Bus JMS | [learn.microsoft.com/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-azure-service-bus](https://learn.microsoft.com/es-es/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-azure-service-bus) |
| Dead-Letter Queues                   | [learn.microsoft.com/azure/service-bus-messaging/service-bus-dead-letter-queues](https://learn.microsoft.com/es-es/azure/service-bus-messaging/service-bus-dead-letter-queues)                                                                                   |
| Emulador local de Service Bus        | [learn.microsoft.com/azure/service-bus-messaging/overview-emulator](https://learn.microsoft.com/es-es/azure/service-bus-messaging/overview-emulator)                                                                                                             |

---

## 🧠 Resumen Ejecutivo

> Lo que deberías poder decir en una entrevista si te preguntan *"¿Qué es Azure Service Bus y cómo lo integrarías con
Spring Boot?"*

**Azure Service Bus** es el servicio de mensajería empresarial gestionado de Azure. Actúa como intermediario de mensajes
confiable entre microservicios, desacoplando productores y consumidores en el tiempo y en el espacio. La diferencia
fundamental es entre **Colas** (un mensaje llega a un solo consumidor — ideal para distribución de trabajo) y **Topics
con suscripciones** (un mensaje llega a todos los suscriptores — patrón pub/sub ideal para eventos que múltiples
servicios necesitan conocer). Sus garantías empresariales clave son: **Peek-Lock** para procesamiento exactly-once (el
mensaje se bloquea, el consumidor confirma al terminar), **Dead-Letter Queue** para preservar mensajes que no pudieron
procesarse (esencial en producción para no perder datos críticos), **Sessions** para garantizar orden dentro de un grupo
de mensajes, y **mensajes programados** para entrega diferida. A diferencia de Azure Queue Storage, Service Bus soporta
mensajes hasta 100 MB, retención de 14 días, orden garantizado y reintentos automáticos con backoff. Desde Spring Boot,
la integración se hace con `spring-cloud-azure-starter-servicebus-jms` usando la anotación `@JmsListener` para
consumidores y `ServiceBusSenderClient` para productores. Con **Spring WebFlux**, los consumidores pueden usar `Mono` y
`Flux` internamente para operaciones de I/O no bloqueantes, aunque el método listener en sí debe ser síncrono por las
limitaciones del protocolo JMS. En arquitecturas reales, Service Bus conecta el App Service principal con Azure
Functions para procesamiento asíncrono de tareas secundarias.

---

## ⏭️ Siguiente módulo

> 🌊 **Módulo 07 — Azure Event Hubs:** El servicio de streaming de eventos masivos de Azure. Veremos qué es, cómo se
> diferencia de Service Bus, el concepto de Consumer Groups, particiones, y cómo procesar millones de eventos por
> segundo
> desde Spring Boot.

---

*📝 Documentación elaborada como parte del plan de aprendizaje Azure para desarrolladores Java Backend.*  
*🗓️ Última actualización: 2025*
