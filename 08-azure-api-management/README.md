# 🚪 Módulo 08 — Azure API Management (APIM)

> 📚 **Serie:** Azure para Java Developers  
> 🗂️ **Módulo:** 08 de 08 — MÓDULO FINAL  
> 🎯 **Objetivo:** Entender qué es un API Gateway, qué es Azure API Management, cómo protege, transforma, versiona y
> monitorea tus APIs, y cómo se integra con App Service y Azure Functions.

---

## 📋 Tabla de Contenidos

1. [¿Qué es un API Gateway?](#-1-qué-es-un-api-gateway-el-contexto-previo)
2. [¿Qué es Azure API Management?](#-2-qué-es-azure-api-management)
3. [Cómo está organizado APIM](#-3-cómo-está-organizado-apim)
4. [Políticas — El corazón de APIM](#-4-políticas--el-corazón-de-apim)
5. [Seguridad y autenticación](#-5-seguridad-y-autenticación)
6. [Versionado y revisiones de APIs](#-6-versionado-y-revisiones-de-apis)
7. [Rate Limiting y Throttling](#-7-rate-limiting-y-throttling)
8. [El Developer Portal](#-8-el-developer-portal)
9. [APIM con App Service y Azure Functions](#-9-apim-con-app-service-y-azure-functions)
10. [Tiers disponibles](#-10-tiers-disponibles)
11. [Ejemplo de políticas reales](#-11-ejemplo-de-políticas-reales)
12. [Casos de uso reales](#-12-casos-de-uso-reales-en-el-mundo-laboral)
13. [Recursos y videos](#-13-recursos-y-videos-recomendados)
14. [Resumen ejecutivo](#-resumen-ejecutivo)
15. [Cierre de la serie](#-cierre-de-la-serie)

---

## 🚪 1. ¿Qué es un API Gateway? (El contexto previo)

Para entender APIM necesitas entender primero el problema que resuelve en arquitecturas de microservicios.

### El problema sin API Gateway

Imagina que tienes 8 microservicios desplegados en Azure App Service. Cada uno expone su propia API REST. Un cliente
externo (app móvil, web, partner) necesita consumirlos:

```
SIN API GATEWAY — Caos total:

App Móvil ──────────────────────────────► Servicio Usuarios    :8081
App Móvil ──────────────────────────────► Servicio Pedidos     :8082
App Móvil ──────────────────────────────► Servicio Pagos       :8083
App Móvil ──────────────────────────────► Servicio Inventario  :8084
App Partner ────────────────────────────► Servicio Usuarios    :8081
App Partner ────────────────────────────► Servicio Pedidos     :8082
Web ────────────────────────────────────► Servicio Reportes    :8085

Problemas:
❌ Cada servicio tiene su propia URL — los clientes deben conocer todas.
❌ Cada servicio implementa autenticación por separado — código duplicado.
❌ No hay un punto centralizado para monitorear el tráfico.
❌ Para el rate limiting hay que implementarlo en cada servicio.
❌ Si cambias la URL de un servicio, debes actualizar todos los clientes.
❌ Datos sensibles (tokens internos) quedan expuestos al exterior.
```

### La solución: API Gateway

Un **API Gateway** es un punto de entrada único para todas las APIs. Actúa como intermediario inteligente entre los
clientes externos y los servicios internos:

```
CON API GATEWAY — Orden y control:

App Móvil ──────┐
App Partner ────┤──► [API GATEWAY] ──────► Servicio Usuarios    (interno)
Web ────────────┘        │          ├────► Servicio Pedidos     (interno)
                         │          ├────► Servicio Pagos       (interno)
                         │          ├────► Servicio Inventario  (interno)
                         │          └────► Servicio Reportes    (interno)
                         │
                    Aquí ocurre:
                    ✅ Autenticación centralizada
                    ✅ Rate limiting
                    ✅ Logging y monitoreo
                    ✅ Transformación de requests/responses
                    ✅ Versionado de APIs
                    ✅ Caché de respuestas
                    ✅ SSL termination
```

> 💡 **En resumen:** Un API Gateway es el "portero inteligente" de tu arquitectura. Todo el tráfico externo pasa por él
> antes de llegar a tus servicios internos. Centraliza la seguridad, el monitoreo, el control de tráfico y la
> documentación.

---

## 🚪 2. ¿Qué es Azure API Management?

**Azure API Management (APIM)** es el servicio de API Gateway gestionado de Microsoft Azure. Permite publicar, proteger,
transformar, documentar y monitorear APIs de cualquier origen — ya sea un App Service con Spring Boot, Azure Functions,
servicios externos o incluso APIs legacy on-premise.

### La versión larga (con contexto real)

Imagina que eres el arquitecto de una empresa fintech peruana que tiene:

- 5 microservicios en Azure App Service (Java/Spring Boot).
- 3 Azure Functions para tareas específicas.
- Una API legacy en un servidor on-premise que aún no se migró.
- Clientes internos (apps propias) y externos (partners, integradores).

Sin APIM, cada equipo implementa autenticación, logging y rate limiting por separado. Los partners no saben qué APIs
existen ni cómo usarlas. Cambiar la URL de un servicio rompe todas las integraciones.

**Con APIM:**

```
Partners externos  ──► api.mifintech.com/v1/transferencias
Apps propias       ──► api.mifintech.com/v1/usuarios
Integradores       ──► api.mifintech.com/v1/reportes
                              │
                              ▼
                    [Azure API Management]
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
    [App Service        [Azure         [Servidor
     Spring Boot]       Functions]      On-Premise]
    (servicios nuevos)  (tareas)       (legacy)
```

Los clientes solo conocen `api.mifintech.com`. APIM enruta internamente a donde corresponde. Si mueves un servicio,
cambias la regla en APIM — los clientes no saben que algo cambió.

**¿Qué hace APIM específicamente?**

- 🔐 **Seguridad:** Valida tokens OAuth2, API Keys, certificados de cliente.
- 📊 **Monitoreo:** Registra cada request con latencia, código de respuesta, usuario.
- 🚦 **Rate Limiting:** Limita cuántas llamadas puede hacer cada cliente por minuto.
- 🔄 **Transformación:** Modifica headers, body o URLs antes de reenviar al backend.
- 📋 **Documentación:** Genera automáticamente un portal para desarrolladores.
- 🗄️ **Caché:** Almacena respuestas frecuentes para reducir carga en los backends.
- 🔀 **Versionado:** Gestiona múltiples versiones de una API simultáneamente.
- 🌍 **Multi-región:** Despliega el gateway en múltiples regiones del mundo.

> 💡 **En resumen:** APIM es el API Gateway de Azure. Es el punto de entrada único para todas tus APIs, donde se
> centraliza la seguridad, el monitoreo, el control de tráfico y la documentación. Fue diseñado para organizaciones que
> tienen múltiples APIs y necesitan gestionarlas de forma coherente y escalable.

---

## 🏗️ 3. Cómo está organizado APIM

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Instancia de APIM                                │
│                api.miempresa.azure-api.net                          │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                         APIS                                  │  │
│  │                                                               │  │
│  │  ┌──────────────────┐    ┌──────────────────┐                 │  │
│  │  │  API: Usuarios   │    │  API: Pedidos    │                 │  │
│  │  │  v1, v2          │    │  v1              │                 │  │
│  │  │                  │    │                  │                 │  │
│  │  │  GET /usuarios   │    │  GET /pedidos    │                 │  │
│  │  │  POST /usuarios  │    │  POST /pedidos   │                 │  │
│  │  │  GET /usuarios/{id}   │  DELETE /pedidos │                 │  │
│  │  └──────────────────┘    └──────────────────┘                 │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌──────────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │    PRODUCTOS     │  │   POLÍTICAS  │  │      USUARIOS        │   │
│  │  (agrupan APIs   │  │  (reglas de  │  │  (developers que     │   │
│  │   por cliente)   │  │ transformac.)│  │   consumen la API)   │   │
│  └──────────────────┘  └──────────────┘  └──────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### Conceptos clave de la estructura

| Concepto        | Descripción                                                                                                                 |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------|
| **API**         | Representa un backend (App Service, Function, etc.). Agrupa sus operaciones.                                                |
| **Operación**   | Un endpoint específico: `GET /usuarios/{id}`, `POST /pedidos`.                                                              |
| **Producto**    | Agrupa APIs para exponerlas a un grupo de clientes. Ej: "Plan Basic" solo accede a /usuarios, "Plan Premium" accede a todo. |
| **Suscripción** | Un cliente se suscribe a un Producto y recibe una API Key.                                                                  |
| **Política**    | Regla que APIM aplica al request/response (autenticación, transformación, rate limit, etc.).                                |
| **Backend**     | El servicio real al que APIM reenvía las llamadas (App Service, Function, URL externa).                                     |
| **Gateway**     | El componente que procesa el tráfico. Puede estar en múltiples regiones.                                                    |

---

## 📜 4. Políticas — El corazón de APIM

Las **políticas** son el mecanismo más poderoso de APIM. Son fragmentos de XML que definen qué hace APIM con cada
request o response. Se pueden aplicar a nivel global, por producto, por API o por operación individual.

### El pipeline de una request en APIM

```
Cliente ──► [Inbound] ──► [Backend] ──► [Outbound] ──► Cliente
               │              │              │
           Políticas       Tu servicio    Políticas
           de entrada      real           de salida
           (antes de       (App Service,  (después de
           llegar al       Functions,     recibir la
           backend)        etc.)          respuesta)

Adicionalmente existe [On-Error] para manejar errores en cualquier etapa.
```

### Las 4 secciones de una política

```xml

<policies>
    <!-- Se ejecuta ANTES de enviar la request al backend -->
    <inbound>
        <base/>  <!-- Aplica las políticas del nivel superior (herencia) -->
        <!-- Aquí: autenticación, rate limiting, transformación del request -->
    </inbound>

    <!-- Controla cómo se envía la request al backend -->
    <backend>
        <base/>
        <!-- Aquí: retry, circuit breaker, load balancing -->
    </backend>

    <!-- Se ejecuta DESPUÉS de recibir la respuesta del backend -->
    <outbound>
        <base/>
        <!-- Aquí: transformación del response, agregar headers, caché -->
    </outbound>

    <!-- Se ejecuta si ocurre un error en cualquier etapa -->
    <on-error>
        <base/>
        <!-- Aquí: formato de error personalizado, logging de errores -->
    </on-error>
</policies>
```

---

## 🔐 5. Seguridad y Autenticación

### 🔑 API Keys (Subscription Keys)

El mecanismo más simple. Cada suscriptor recibe una clave única que debe enviar en cada request.

```
Cliente ──► GET /api/usuarios
            Header: Ocp-Apim-Subscription-Key: abc123xyz789...
                                    │
                                    ▼
                            [APIM valida la clave]
                                    │
                          ┌─────────┴─────────┐
                          ▼                   ▼
                     Clave válida         Clave inválida
                          │                   │
                          ▼                   ▼
                   Reenvía al backend    401 Unauthorized
```

**Política de validación de API Key:**

```xml

<inbound>
    <!-- APIM valida automáticamente el Ocp-Apim-Subscription-Key -->
    <!-- No necesitas escribir código — es el comportamiento por defecto -->
    <base/>
</inbound>
```

### 🎫 JWT (OAuth2 / Azure AD)

Para APIs de producción empresarial, la autenticación se hace con tokens JWT emitidos por Azure Active Directory.

```xml

<inbound>
    <base/>
    <!-- Valida el token JWT antes de reenviar al backend -->
    <validate-jwt header-name="Authorization"
                  failed-validation-httpcode="401"
                  failed-validation-error-message="Token inválido o expirado">
        <!-- URL del emisor del token (Azure AD) -->
        <openid-config url="https://login.microsoftonline.com/{tenant-id}/v2.0/.well-known/openid-configuration"/>
        <!-- Audiences válidas (el ID de tu aplicación en Azure AD) -->
        <audiences>
            <audience>api://mi-app-id</audience>
        </audiences>
        <!-- Roles requeridos en el token -->
        <required-claims>
            <claim name="roles" match="any">
                <value>Api.Read</value>
                <value>Api.Write</value>
            </claim>
        </required-claims>
    </validate-jwt>
</inbound>
```

### 🔒 Certificados de cliente (mTLS)

Para comunicaciones máximamente seguras entre sistemas (muy común en banca):

```xml

<inbound>
    <base/>
    <!-- Valida el certificado del cliente antes de procesar la request -->
    <choose>
        <when condition="@(context.Request.Certificate == null
                          || !context.Request.Certificate.Verify()
                          || context.Request.Certificate.Thumbprint
                             != '{{CERTIFICADO_THUMBPRINT_ESPERADO}}')">
            <return-response>
                <set-status code="403" reason="Certificado inválido"/>
            </return-response>
        </when>
    </choose>
</inbound>
```

### 🛡️ IP Filtering — Lista blanca de IPs

```xml

<inbound>
    <base/>
    <!-- Solo permite tráfico desde IPs específicas -->
    <ip-filter action="allow">
        <address>190.235.0.0/16</address>   <!-- Rango IP del partner A -->
        <address>200.48.225.130</address>   <!-- IP fija del sistema legado -->
    </ip-filter>
</inbound>
```

---

## 🔢 6. Versionado y Revisiones de APIs

Uno de los problemas más comunes al evolucionar APIs es no romper a los clientes existentes. APIM ofrece dos mecanismos
complementarios.

### Versiones (Breaking changes)

Cuando necesitas hacer cambios que rompen la compatibilidad, creas una nueva versión:

```
/api/v1/usuarios  → Versión antigua (sigue funcionando para clientes legacy)
/api/v2/usuarios  → Nueva versión (nuevos campos, estructura diferente)
```

APIM gestiona ambas versiones simultáneamente y enruta a diferentes backends:

```html
<!-- Esquemas de versionado soportados por APIM: -->

<!-- 1. Por URL path (más común) -->
https://api.miempresa.com/v1/usuarios
https://api.miempresa.com/v2/usuarios

<!-- 2. Por Query String -->
https://api.miempresa.com/usuarios?api-version=1.0
https://api.miempresa.com/usuarios?api-version=2.0

<!-- 3. Por Header -->
GET https://api.miempresa.com/usuarios
Api-Version: 1.0
```

### Revisiones (Non-breaking changes)

Para cambios menores que no rompen la compatibilidad (agregar un campo opcional, mejorar documentación), usas
**revisiones**. Las revisiones permiten probar cambios antes de hacerlos "current" sin exponer una nueva URL.

```
Revisión 1 → Producción actual
Revisión 2 → En pruebas (accesible en URL especial: /usuarios;rev=2)
Revisión 3 → En desarrollo

Cuando revisión 2 está lista → se hace "current" → pasa a ser producción
```

> 💡 **Diferencia clave:**
> - **Versión** = URL diferente, puede romper compatibilidad, los clientes eligen cuándo migrar.
> - **Revisión** = Misma URL, sin romper compatibilidad, cambios internos progresivos.

---

## 🚦 7. Rate Limiting y Throttling

El **Rate Limiting** protege tus backends de ser saturados por demasiadas requests. Es una de las políticas más
importantes en producción.

### Rate Limit por suscripción (por cliente)

```xml

<inbound>
    <base/>
    <!-- Máximo 100 llamadas cada 60 segundos por suscripción (cliente) -->
    <rate-limit calls="100" renewal-period="60"/>
</inbound>
```

Cuando se supera el límite, APIM retorna automáticamente `429 Too Many Requests`.

### Rate Limit por IP

```xml

<inbound>
    <base/>
    <!-- Máximo 50 llamadas por minuto desde la misma IP -->
    <rate-limit-by-key calls="50"
                       renewal-period="60"
                       counter-key="@(context.Request.IpAddress)"/>
</inbound>
```

### Quota — Límite total en un período largo

A diferencia del rate limit (por segundo/minuto), la **quota** limita el total de llamadas en un período mayor
(día, mes):

```xml

<inbound>
    <base/>
    <!-- Plan Basic: máximo 10,000 llamadas por mes por suscripción -->
    <quota calls="10000" renewal-period="2592000"/>
    <!-- renewal-period en segundos: 2592000 = 30 días -->
</inbound>
```

### Políticas combinadas (rate limit + quota)

```xml

<inbound>
    <base/>
    <!-- Rate limit: no más de 10 llamadas por segundo (anti-burst) -->
    <rate-limit calls="10" renewal-period="1"/>
    <!-- Quota mensual: máximo 100,000 llamadas en el mes -->
    <quota calls="100000" renewal-period="2592000"/>
</inbound>
```

### Planes de acceso diferenciados con Productos

```
PLAN FREE:
  - Rate limit: 5 req/seg
  - Quota: 1,000 req/mes
  - Sin SLA

PLAN STANDARD:
  - Rate limit: 50 req/seg
  - Quota: 100,000 req/mes
  - SLA 99.9%

PLAN PREMIUM:
  - Rate limit: 500 req/seg
  - Quota: ilimitada
  - SLA 99.99%
  - Soporte dedicado
```

---

## 📖 8. El Developer Portal

El **Developer Portal** es una de las características más valiosas de APIM y una de las menos conocidas.
Es un **portal web autogenerado** que documenta todas tus APIs y permite a los desarrolladores explorarlas
y probarlas.

```
┌──────────────────────────────────────────────────────────────┐
│                    Developer Portal                          │
│                developer.miempresa.com                       │
│                                                              │
│  📚 Catálogo de APIs                                         │
│  ├── API de Usuarios    → Documentación + Try it             │
│  ├── API de Pedidos     → Documentación + Try it             │
│  └── API de Pagos       → Documentación + Try it             │
│                                                              │
│  🔑 Mis Suscripciones   → Gestionar API Keys                 │
│  📊 Mi Uso              → Ver consumo y límites              │
│  📋 Changelog           → Historial de cambios               │
└──────────────────────────────────────────────────────────────┘
```

**¿Qué genera automáticamente APIM?**

- Documentación de cada endpoint con parámetros, tipos y descripciones.
- Ejemplos de request y response.
- Un cliente interactivo para probar los endpoints directamente desde el navegador (como Swagger UI pero integrado).
- Página de registro para que los desarrolladores obtengan sus API Keys.

**¿Por qué importa esto para tu perfil?**

En entrevistas de nivel medio-senior, saber que APIM genera documentación automática y que existe el concepto de
Developer Portal demuestra madurez en el diseño de APIs para terceros — algo muy valorado en empresas que tienen
integraciones con partners.

---

## ⚙️ 9. APIM con App Service y Azure Functions

Aquí es donde todo se conecta con lo que aprendiste en los módulos anteriores.

### Arquitectura completa con todos los servicios vistos

```
                    INTERNET
                       │
                       ▼
          ┌────────────────────────┐
          │   Azure API Management │
          │  api.miempresa.com     │
          │                        │
          │  [Autenticación JWT]   │
          │  [Rate Limiting]       │
          │  [Logging]             │
          │  [Caché]               │
          └───────────┬────────────┘
                      │
        ┌─────────────┼──────────────┐
        ▼             ▼              ▼
  [App Service]  [Azure        [Legacy API
  Spring Boot    Functions]     On-Premise]
  Módulo 01      Módulo 05
        │
   ┌────┼────┐
   ▼    ▼    ▼
[Azure [Cosmos [Service
  SQL]   DB]    Bus]
Mód.02 Mód.03  Mód.06
                 │
                 ▼
           [Event Hubs]
            Mód.07
```

### Cómo importar un App Service Spring Boot en APIM

```xml
<!-- Cuando defines el backend en APIM para tu App Service: -->
<backend>
    <base/>
    <!-- APIM reenvía la request a tu App Service -->
    <!-- La URL real del backend es interna — los clientes no la ven -->
</backend>
```

En el portal de Azure, APIM puede importar automáticamente la especificación **OpenAPI (Swagger)** de tu app Spring
Boot:

```java
// En tu app Spring Boot — agregar springdoc-openapi para generar el swagger
// pom.xml:
// <dependency>
//     <groupId>org.springdoc</groupId>
//     <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
//     <version>2.3.0</version>
// </dependency>

// APIM importa desde:
// https://mi-app.azurewebsites.net/v3/api-docs
// Y genera automáticamente todas las operaciones en APIM
```

### Ocultar la URL real del backend

Un beneficio importante de APIM: los clientes nunca conocen la URL real de tus servicios. Si cambias de
`mi-app.azurewebsites.net` a otro backend, los clientes no saben que algo cambió.

```xml

<inbound>
    <base/>
    <!-- Reemplaza la URL del host en el request antes de reenviar -->
    <!-- El cliente llamó a api.miempresa.com, el backend recibe la request -->
    <!-- como si viniera directamente -->
    <set-backend-service base-url="https://mi-app-nueva.azurewebsites.net"/>
</inbound>
```

### Llamar a Azure Functions desde APIM

```xml
<!-- Backend configurado para apuntar a una Azure Function -->
<backend>
    <base/>
    <set-backend-service
            base-url="https://mi-funcion.azurewebsites.net/api"
            backend-id="azure-functions-backend"/>
</backend>

<inbound>
<base/>
<!-- Agregar la Function Key en el header — el cliente no necesita conocerla -->
<set-header name="x-functions-key" exists-action="override">
    <value>{{AZURE_FUNCTION_KEY}}</value>
</set-header>
</inbound>
```

---

## 💰 10. Tiers Disponibles

| Tier            | Throughput      | SLA     | Multi-región | Precio aprox.             |
|-----------------|-----------------|---------|--------------|---------------------------|
| **Consumption** | Por llamada     | 99.95%  | No           | ~$3.50/millón de llamadas |
| **Developer**   | 500 req/seg     | Sin SLA | No           | ~$50/mes                  |
| **Basic**       | 1,000 req/seg   | 99.95%  | No           | ~$140/mes                 |
| **Standard**    | 2,500 req/seg   | 99.95%  | No           | ~$700/mes                 |
| **Premium**     | 10,000+ req/seg | 99.99%  | ✅ Sí         | ~$2,800/mes por unidad    |

> 📌 **Para aprendizaje:** El tier **Developer** no tiene SLA pero es funcionalmente idéntico al Standard — perfecto para
> entender APIM sin el costo de producción.
>
> **En el mercado laboral peruano:** Las empresas medianas (retail, fintech) típicamente usan **Standard**. Los bancos y
> empresas grandes con presencia regional usan **Premium** por la capacidad multi-región.

---

## 📋 11. Ejemplo de Políticas Reales

### Política completa de producción — API de Transferencias Bancarias

```xml
<!--
  Política aplicada a la API "transferencias" en APIM.
  Esta política representa un caso real de producción bancaria:
  - Valida token JWT de Azure AD
  - Aplica rate limiting diferenciado
  - Transforma el request agregando headers de trazabilidad
  - Loguea cada operación
  - Transforma el response eliminando datos internos
  - Maneja errores de forma consistente
-->
<policies>
    <inbound>
        <base/>

        <!-- 1. VALIDACIÓN JWT — Solo tokens de Azure AD son aceptados -->
        <validate-jwt header-name="Authorization"
                      failed-validation-httpcode="401"
                      failed-validation-error-message="Token de acceso inválido">
            <openid-config url="https://login.microsoftonline.com/{{TENANT_ID}}/v2.0/.well-known/openid-configuration"/>
            <audiences>
                <audience>api://transferencias-api</audience>
            </audiences>
        </validate-jwt>

        <!-- 2. RATE LIMITING — Protección contra abuso -->
        <!-- Máximo 10 transferencias por segundo por suscriptor -->
        <rate-limit calls="10" renewal-period="1"/>
        <!-- Máximo 50,000 transferencias por mes (quota del plan) -->
        <quota calls="50000" renewal-period="2592000"/>

        <!-- 3. TRAZABILIDAD — Agregar ID de correlación para rastrear en logs -->
        <set-header name="X-Correlation-Id" exists-action="override">
            <!-- Generar UUID único por request usando expresión de política -->
            <value>@(Guid.NewGuid().ToString())</value>
        </set-header>

        <!-- 4. IDENTIFICACIÓN — Agregar quién hace la llamada -->
        <set-header name="X-Caller-Id" exists-action="override">
            <value>@(context.Subscription.Id)</value>
        </set-header>

        <!-- 5. ENMASCARAMIENTO — Ocultar la URL real del backend -->
        <set-backend-service
                base-url="https://transferencias-service.azurewebsites.net"/>
    </inbound>

    <backend>
        <base/>
        <!-- RETRY — Reintentar hasta 3 veces si el backend falla -->
        <retry condition="@(context.Response.StatusCode >= 500)"
               count="3"
               interval="2"
               delta="1"
               max-interval="10"
               first-fast-retry="false">
            <forward-request/>
        </retry>
    </backend>

    <outbound>
        <base/>

        <!-- 6. SEGURIDAD — Eliminar headers internos antes de enviar al cliente -->
        <!-- El cliente no debe ver información sobre la infraestructura interna -->
        <set-header name="X-Powered-By" exists-action="delete"/>
        <set-header name="X-AspNet-Version" exists-action="delete"/>
        <set-header name="Server" exists-action="delete"/>

        <!-- 7. CACHÉ — Cachear respuestas de consulta de estado de transferencia -->
        <!-- Solo para operaciones GET — nunca para POST/PUT/DELETE -->
        <cache-store duration="30"/>

        <!-- 8. CORS — Permitir llamadas desde el frontend web -->
        <cors allow-credentials="true">
            <allowed-origins>
                <origin>https://app.miempresa.com</origin>
                <origin>https://portal.miempresa.com</origin>
            </allowed-origins>
            <allowed-methods>
                <method>GET</method>
                <method>POST</method>
            </allowed-methods>
            <allowed-headers>
                <header>Authorization</header>
                <header>Content-Type</header>
            </allowed-headers>
        </cors>
    </outbound>

    <on-error>
        <base/>

        <!-- 9. FORMATO DE ERROR CONSISTENTE — Todos los errores con el mismo formato -->
        <set-header name="Content-Type" exists-action="override">
            <value>application/json</value>
        </set-header>
        <set-body>@{
            return new JObject(
            new JProperty("codigo", context.Response.StatusCode),
            new JProperty("mensaje", context.LastError.Message),
            new JProperty("trazaId", context.Request.Headers
            .GetValueOrDefault("X-Correlation-Id", "N/A")),
            new JProperty("timestamp", DateTime.UtcNow.ToString("o"))
            ).ToString();
            }
        </set-body>
    </on-error>
</policies>
```

### Política de transformación de request — Agregar datos antes de llegar al backend

```xml

<inbound>
    <base/>
    <!-- Transformar el body del request antes de enviarlo al backend -->
    <!-- Caso de uso: agregar metadata que el cliente no conoce -->
    <set-body>@{
        var body = context.Request.Body.As<JObject>(preserveContent: true);
        // Agregar el ID del suscriptor al body (el backend lo necesita para auditoría)
        body["suscriptorId"] = context.Subscription.Id;
        body["ipOrigen"] = context.Request.IpAddress;
        body["timestamp"] = DateTime.UtcNow.ToString("o");
        return body.ToString();
        }
    </set-body>
</inbound>
```

### Política de caché — Reducir carga en el backend

```xml

<inbound>
    <base/>
    <!-- Buscar en caché antes de llamar al backend -->
    <!-- La clave de caché incluye el ID del recurso para ser específica -->
    <cache-lookup vary-by-developer="false"
                  vary-by-developer-groups="false"
                  allow-private-response-caching="false"
                  must-revalidate="false"
                  downstream-caching-type="none">
        <vary-by-header>Accept</vary-by-header>
        <vary-by-query-parameter>productoId</vary-by-query-parameter>
    </cache-lookup>
</inbound>
<outbound>
<base/>
<!-- Guardar en caché la respuesta por 5 minutos -->
<cache-store duration="300"/>
</outbound>
```

### Política de mock — Respuesta simulada sin tocar el backend

```xml
<!--
  Caso de uso: el frontend necesita consumir una API que aún no está lista.
  APIM puede retornar respuestas simuladas mientras el backend se desarrolla.
-->
<inbound>
    <base/>
    <mock-response status-code="200" content-type="application/json"/>
</inbound>
```

---

## 🏢 12. Casos de Uso Reales en el Mundo Laboral

### 🏦 Banco Digital Peruano

```
Arquitectura real de API Management en banca:

Clientes móviles   ──► api.banco.com.pe/v2/cuentas
Clientes web       ──► api.banco.com.pe/v2/transferencias
Partners (Yape,    ──► api.banco.com.pe/v1/pagos     ← Versión antigua aún activa
  Tunki, etc.)
BCRP (regulador)   ──► api.banco.com.pe/regulatorio/reportes
                              │
                     [Azure API Management]
                              │
                    Políticas que aplica:
                    ✅ JWT de Azure AD para apps propias
                    ✅ mTLS para partners regulados
                    ✅ IP Filtering para BCRP
                    ✅ Rate limiting por tier de cliente
                    ✅ Logging de cada operación (auditoría SBS)
                    ✅ Transformación: agrega campos de trazabilidad
                    ✅ Enmascaramiento: elimina datos internos del response
```

### 🛒 Marketplace Retail

```
Escenario: retailer con tiendas propias y sellers externos

App propia          ──► (Plan Premium: sin rate limit)
Sellers externos    ──► (Plan Standard: 1,000 req/hora)
Integradores ERP    ──► (Plan Basic: 100 req/hora)
                              │
                     [Azure API Management]
                     Products con cuotas distintas:
                     - Plan Premium: acceso total, sin quota
                     - Plan Standard: solo APIs de catálogo e inventario
                     - Plan Basic: solo API de pedidos
```

### 🏥 Telemedicina

```
Escenario: plataforma con médicos, pacientes y laboratorios

App de pacientes    ──► Producto "Pacientes"   → solo sus propios datos
App de médicos      ──► Producto "Médicos"     → datos de sus pacientes
Laboratorios        ──► Producto "Laboratorios"→ solo resultados
MINSA               ──► Producto "Regulatorio" → datos anonimizados
                              │
                     [Azure API Management]
                     Política clave: transformación que filtra
                     datos según el rol del consumidor
```

---

## 🎥 13. Recursos y Videos Recomendados

| Recurso                             | Descripción                                         | Dónde encontrarlo                                                                                   |
|-------------------------------------|-----------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| **Microsoft Learn — APIM**          | Ruta oficial con módulos interactivos gratuitos     | [learn.microsoft.com/azure/api-management](https://learn.microsoft.com/es-es/azure/api-management/) |
| **"Azure API Management Tutorial"** | Visión completa con demo del portal                 | Buscar en YouTube: `"Azure API Management tutorial complete"`                                       |
| **"APIM Policies Deep Dive"**       | Políticas avanzadas con ejemplos reales             | Buscar en YouTube: `"Azure API Management policies tutorial"`                                       |
| **"API Gateway Pattern"**           | Patrón de diseño en arquitecturas de microservicios | Buscar: `"API Gateway pattern microservices Martin Fowler"`                                         |
| **"APIM con Spring Boot"**          | Importar una API Spring Boot en APIM                | Buscar en YouTube: `"Azure API Management Spring Boot import OpenAPI"`                              |

> 🎯 **Para explorar APIM sin costo:**
> El tier **Consumption** cobra por llamada (~$3.50/millón) y no tiene costo fijo mensual. Si solo haces pruebas con
> pocas llamadas, el costo es prácticamente cero. También puedes usar el tier **Developer** con una prueba gratuita de
> Azure (si consigues una tarjeta prepago o virtual).

### 📖 Documentación oficial clave

| Documento                           | URL                                                                                                                                                                            |
|-------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Introducción a Azure API Management | [learn.microsoft.com/azure/api-management/api-management-key-concepts](https://learn.microsoft.com/es-es/azure/api-management/api-management-key-concepts)                     |
| Referencia de políticas APIM        | [learn.microsoft.com/azure/api-management/api-management-policies](https://learn.microsoft.com/es-es/azure/api-management/api-management-policies)                             |
| Importar API desde App Service      | [learn.microsoft.com/azure/api-management/import-app-service-as-api](https://learn.microsoft.com/es-es/azure/api-management/import-app-service-as-api)                         |
| Versionado de APIs en APIM          | [learn.microsoft.com/azure/api-management/api-management-versions](https://learn.microsoft.com/es-es/azure/api-management/api-management-versions)                             |
| Developer Portal                    | [learn.microsoft.com/azure/api-management/api-management-howto-developer-portal](https://learn.microsoft.com/es-es/azure/api-management/api-management-howto-developer-portal) |

---

## 🧠 Resumen Ejecutivo

> Lo que deberías poder decir en una entrevista si te preguntan *"¿Qué es Azure API Management y para qué lo usarías?"*

**Azure API Management (APIM)** es el servicio de API Gateway gestionado de Azure. Actúa como punto de entrada único
para todas las APIs de una organización, independientemente de si los backends son App Services con Spring Boot, Azure
Functions, servicios legacy on-premise o APIs externas. Su componente central son las **políticas**: fragmentos XML que
definen reglas aplicadas en cuatro etapas del ciclo de vida de cada request — inbound (antes de llegar al backend),
backend (cómo se reenvía), outbound (después de recibir la respuesta) y on-error (manejo de errores). Las políticas más
importantes incluyen: **validate-jwt** para autenticación con tokens de Azure AD, **rate-limit** y **quota** para
controlar el volumen de llamadas por cliente, **set-backend-service** para enmascarar las URLs reales de los backends,
**cache-lookup/cache-store** para reducir carga en los servicios, **set-body** para transformar requests y responses, y
**cors** para habilitar llamadas desde el frontend web. Los **Productos** permiten agrupar APIs y definir planes de
acceso diferenciados (Basic, Standard, Premium) con cuotas distintas. El **versionado** permite tener múltiples
versiones de una API simultáneamente sin romper clientes existentes. El **Developer Portal** autogenera documentación
interactiva para que equipos externos puedan explorar y probar las APIs. En una arquitectura real con Spring Boot en
Azure, APIM se coloca delante de todos los App Services y Azure Functions, importando sus especificaciones OpenAPI y
centralizando autenticación, monitoreo y control de tráfico en un solo lugar.

---

## 🏆 Cierre de la Serie

### ¡Completaste los 8 módulos!

```
azure-learning-java/
├── README.md
├── 00-azure-overview/      ✅ ¿Qué es Azure?
├── 01-azure-app-service/   ✅ Desplegar tu Spring Boot
├── 02-azure-sql/           ✅ Base de datos relacional
├── 03-cosmos-db/           ✅ Base de datos NoSQL
├── 04-azure-storage/       ✅ Archivos y blobs
├── 05-azure-functions/     ✅ Serverless y eventos
├── 06-azure-service-bus/   ✅ Mensajería empresarial
├── 07-azure-event-hubs/    ✅ Streaming masivo
└── 08-azure-api-management/✅ API Gateway
```

### Lo que puedes decir ahora en una entrevista

Si alguien te pregunta por Azure en una entrevista, ya puedes responder con criterio sobre cada uno de estos servicios:

| Servicio            | Lo que sabes responder                                                               |
|---------------------|--------------------------------------------------------------------------------------|
| **App Service**     | Cómo desplegar Spring Boot, planes, deployment slots, escalado, variables de entorno |
| **Azure SQL**       | SQL Server en la nube, tiers DTU vs vCore, JPA/Hibernate, HikariCP, seguridad        |
| **Cosmos DB**       | NoSQL, partition key, modelos de consistencia, APIs, cuándo usarlo vs Azure SQL      |
| **Azure Storage**   | Blob, Files, Queue, Table, access tiers, SAS tokens, patrón URL en BD                |
| **Azure Functions** | Serverless, todos los triggers, bindings, cold start, cuándo vs App Service          |
| **Service Bus**     | Colas vs Topics, pub/sub, dead-letter queue, peek-lock, integración con WebFlux      |
| **Event Hubs**      | Streaming vs mensajería, particiones, consumer groups, checkpointing                 |
| **API Management**  | API Gateway, políticas, rate limiting, versionado, developer portal                  |

### Tu ventaja competitiva

Dijiste al inicio que ibas a postular a Encora con 2 años de experiencia junior para una posición de +4 años. Ahora
tienes algo que muy pocos desarrolladores Java junior tienen: **conocimiento teórico sólido y documentado de todos los
servicios Azure que piden en el 99% de las ofertas del mercado peruano**.

En una entrevista, cuando te pregunten por Azure, no dirás *"no tengo experiencia"*. Dirás:

> *"No he trabajado directamente con Azure en producción, pero he estudiado en profundidad todos los servicios que
mencionan en esta oferta — App Service, Azure SQL, Cosmos DB, Storage, Functions, Service Bus, Event Hubs y API
Management — y tengo documentación propia que puedo mostrarles. Entiendo cuándo usar cada uno, cómo se integran con
Spring Boot y cuáles son las buenas prácticas de cada servicio."*

Eso, combinado con tus bases sólidas en Spring Boot y WebFlux, te pone en una posición mucho más competitiva de lo que
crees.

---

### Próximos pasos recomendados

```
1. 📁 Sube el repositorio a GitHub
      Crea el README.md del índice general (el módulo 00 ya lo tienes)
      y súbelo todo. Es tu portafolio de aprendizaje.

2. 🛠️ Practica con emuladores locales
      - Azurite → Storage y Queue
      - Azure Functions Core Tools → Azure Functions
      - Service Bus Emulator (Docker) → Service Bus
      - Event Hubs Emulator (Docker) → Event Hubs
      - Cosmos DB Emulator → Cosmos DB

3. 📜 Considera certificarte
      AZ-900 (Azure Fundamentals) → Prueba de conocimiento teórico
      AZ-204 (Azure Developer) → El siguiente paso natural para developers

4. 🔗 Conecta los puntos
      Intenta construir una mini-app que use al menos 3 de los servicios juntos:
      App Service + Azure SQL + Service Bus es un buen punto de partida.

5. 🚀 Postula
      Con tu repo en GitHub, tus bases en Spring Boot/WebFlux y este
      conocimiento teórico sólido — ya estás listo para intentarlo.
```

---

*📝 Documentación elaborada como parte del plan de aprendizaje Azure para desarrolladores Java Backend.*  
*🗓️ Última actualización: 2025*  
*💪 ¡Éxito en tu proceso con Encora y en tu carrera como desarrollador Java!*
