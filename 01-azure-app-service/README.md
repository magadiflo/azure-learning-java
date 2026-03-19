# 🚀 Módulo 01 — Azure App Service

> 📚 **Serie:** Azure para Java Developers  
> 🗂️ **Módulo:** 01 de 08  
> 🎯 **Objetivo:** Entender qué es Azure App Service, cómo funciona, sus conceptos clave y cómo desplegar una aplicación
> Spring Boot en él.

---

## 📋 Tabla de Contenidos

1. [¿Qué es Azure App Service?](#-1-qué-es-azure-app-service)
2. [¿Cómo funciona por dentro?](#-2-cómo-funciona-por-dentro)
3. [Planes de servicio (Pricing Tiers)](#-3-planes-de-servicio-pricing-tiers)
4. [Conceptos clave](#-4-conceptos-clave)
5. [App Service vs otras alternativas](#-5-app-service-vs-otras-alternativas-en-azure)
6. [Integración con Spring Boot](#-6-integración-con-spring-boot)
7. [Ejemplo de código](#-7-ejemplo-de-código)
8. [Despliegue paso a paso](#-8-despliegue-paso-a-paso-conceptual)
9. [Recursos y videos](#-9-recursos-y-videos-recomendados)
10. [Resumen ejecutivo](#-resumen-ejecutivo)

---

## 🚀 1. ¿Qué es Azure App Service?

**Azure App Service** es una plataforma **PaaS (Platform as a Service)** de Microsoft Azure que permite desplegar,
ejecutar y escalar aplicaciones web, APIs REST y backends móviles **sin necesidad de gestionar servidores, sistemas
operativos ni infraestructura**.

### La versión larga (con contexto real)

Imagina que terminaste de desarrollar tu API REST con Spring Boot. Tienes tu archivo `.jar` listo. Ahora necesitas que
esa API esté disponible en internet, las 24 horas del día, los 7 días de la semana. ¿Qué opciones tienes?

**Opción A — Sin nube (On-Premise):**

1. Compras o alquilas un servidor físico.
2. Instalas Linux o Windows Server.
3. Instalas el JDK correcto.
4. Configuras el firewall y los puertos.
5. Configuras un servicio systemd para que tu app arranque sola si el servidor se reinicia.
6. Configuras un balanceador de carga si necesitas escalar.
7. Monitoreas el servidor, aplicas parches de seguridad, etc.
8. **Todo eso antes de que un solo usuario pueda usar tu API.**

**Opción B — Con Azure App Service:**

1. Subes tu `.jar` (o tu imagen Docker).
2. Azure hace todo lo demás.
3. **Tu API está en internet en minutos.**

Esa es la propuesta de valor de App Service: **tú te enfocas en el código, Azure se encarga de la infraestructura**.

App Service soporta múltiples lenguajes y runtimes de forma nativa:

| Lenguaje / Runtime | Versiones soportadas                 |
|--------------------|--------------------------------------|
| ☕ **Java**         | 8, 11, 17, 21 (con Tomcat o Java SE) |
| 🐍 Python          | 3.8, 3.9, 3.10, 3.11                 |
| 🟢 Node.js         | 16, 18, 20                           |
| 💜 .NET            | 6, 7, 8                              |
| 🐘 PHP             | 8.0, 8.1, 8.2                        |
| 🐳 Docker          | Cualquier imagen de contenedor       |

> 💡 **En resumen:** Azure App Service es el servicio donde "vive" y "corre" tu aplicación Spring Boot en Azure. Es el
> punto de partida de casi cualquier arquitectura backend en Azure. Sin este servicio (o su equivalente), los demás
> servicios no tendrían dónde conectarse.

---

## ⚙️ 2. ¿Cómo funciona por dentro?

Entender la arquitectura interna de App Service te ayudará a razonar mejor sobre su comportamiento en producción y a
responder preguntas técnicas en entrevistas.

### Arquitectura interna

```
┌─────────────────────────────────────────────────────────┐
│                    AZURE APP SERVICE                    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │              App Service Plan                   │    │
│  │         (define CPU, RAM y precio)              │    │
│  │                                                 │    │
│  │   ┌──────────────┐    ┌──────────────┐          │    │
│  │   │   Web App 1  │    │   Web App 2  │          │    │
│  │   │ (Spring Boot)│    │  (Node.js)   │          │    │
│  │   └──────────────┘    └──────────────┘          │    │
│  │                                                 │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌─────────────────────┐    │
│  │  Scale   │  │  Deploy  │  │      Monitoring     │    │
│  │  Out/In  │  │  Slots   │  │  (Logs / Metrics)   │    │
│  └──────────┘  └──────────┘  └─────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

### Componentes principales

#### 🏗️ App Service Plan

Es el **plan de hosting** que define los recursos de cómputo (CPU y RAM) y el precio. Piénsalo como el contrato de
alquiler de un apartamento: defines cuánto espacio quieres y cuánto pagas, y dentro de ese espacio puedes meter una o
varias aplicaciones.

**Punto clave:** Puedes tener múltiples Web Apps dentro de un mismo App Service Plan. Todas comparten los mismos
recursos de cómputo pero se ejecutan de forma independiente.

#### 🌐 Web App

Es la instancia de tu aplicación dentro del App Service Plan. Cuando creas una Web App para Spring Boot, Azure:

- Asigna un subdominio: `tu-app.azurewebsites.net`
- Configura el runtime de Java seleccionado.
- Habilita HTTPS automáticamente.
- Prepara los logs y métricas.

#### 🔄 Deployment Slots

Son **ambientes paralelos** dentro de la misma Web App. El más importante es el concepto de **staging slot**:

```
[Código nuevo] → [Slot: Staging] → (pruebas) → SWAP → [Slot: Production]
```

Esto permite hacer despliegues sin tiempo de inactividad (**zero downtime deployments**). Pruebas tu nueva versión en
staging, y cuando estás listo, haces un "swap" y staging se convierte en producción de forma instantánea. Si algo falla,
haces swap de vuelta en segundos.

> 📌 **Los Deployment Slots solo están disponibles en planes Standard, Premium y Isolated.** En el plan Free o Shared no
> existen.

---

## 💰 3. Planes de Servicio (Pricing Tiers)

El **App Service Plan** define cuánta potencia tiene tu aplicación y cuánto pagas. Existen varias categorías:

| Tier            | Plan      | CPU                      | RAM       | Uso recomendado         | Precio aproximado |
|-----------------|-----------|--------------------------|-----------|-------------------------|-------------------|
| 🆓 **Free**     | F1        | Compartida               | 1 GB      | Aprendizaje y pruebas   | $0/mes            |
| 🔵 **Shared**   | D1        | Compartida               | 1 GB      | Prototipos simples      | ~$10/mes          |
| 🟢 **Basic**    | B1/B2/B3  | Dedicada                 | 1.75–7 GB | Apps de desarrollo      | ~$13–$55/mes      |
| 🟡 **Standard** | S1/S2/S3  | Dedicada                 | 1.75–7 GB | **Apps de producción**  | ~$70–$300/mes     |
| 🟠 **Premium**  | P1v3/P2v3 | Dedicada + mejor HW      | 8–32 GB   | Apps de alto tráfico    | ~$120–$500/mes    |
| 🔴 **Isolated** | I1/I2/I3  | Dedicada en VNet privada | 8–32 GB   | Banca, salud, regulados | ~$300–$1000+/mes  |

### ¿Cuál se usa en producción real?

En empresas medianas y grandes en Perú (banca, retail, seguros), lo más común es:

- **Standard S2/S3** para APIs backend de tráfico medio.
- **Premium P1v3/P2v3** para aplicaciones críticas con alta concurrencia.
- **Isolated** para entornos bancarios o regulados que requieren red privada dedicada.

> 💡 **Para tu aprendizaje:** El plan **Free (F1)** es suficiente para estudiar y hacer pruebas conceptuales, aunque
> tiene limitaciones como 60 minutos de CPU por día y sin dominio personalizado.

---

## 🔑 4. Conceptos Clave

Estos son los términos que debes manejar con fluidez en una entrevista técnica:

### 📌 Escalado (Scaling)

App Service soporta dos tipos de escalado:

**Scale Up (Escalado vertical):** Cambiar a un plan más potente. Por ejemplo, pasar de B1 (1 core, 1.75 GB RAM) a B3 (4
cores, 7 GB RAM). Es como cambiar a un apartamento más grande.

```
B1 (1 core, 1.75 GB) ──→ B3 (4 cores, 7 GB)
         ↑
    Scale Up
```

**Scale Out (Escalado horizontal):** Agregar más instancias de tu aplicación detrás de un balanceador de carga
automático. Es como alquilar varios apartamentos iguales para distribuir inquilinos.

```
                    ┌── Instancia 1 ──┐
Usuario → [Load Balancer] ── Instancia 2 ──┤ → Tu App Spring Boot
                    └── Instancia 3 ──┘
         ↑
    Scale Out (Auto-scaling)
```

App Service permite configurar **auto-scaling** basado en métricas: si el CPU supera el 70%, automáticamente agrega una
instancia nueva.

> ⚠️ **Punto importante para Spring Boot + WebFlux:** Si tu app usa estado en memoria (como cachés locales o sesiones en
> memoria), el scale-out puede causar problemas porque cada instancia tiene su propio estado. La solución es
> externalizar
> el estado a Redis o a una base de datos.

### 📌 Application Settings (Variables de Entorno)

En lugar de hardcodear configuraciones en tu `application.properties`, en App Service defines las configuraciones como
**Application Settings** en el portal de Azure. Spring Boot las lee automáticamente como variables de entorno.

```
Azure Portal → App Service → Configuration → Application Settings

SPRING_DATASOURCE_URL=jdbc:sqlserver://miservidor.database.windows.net...
SPRING_DATASOURCE_USERNAME=miusuario
SPRING_DATASOURCE_PASSWORD=mipassword
```

Esto es una práctica de seguridad crítica: **nunca hardcodeas credenciales en tu código**.

### 📌 Custom Domains y SSL/TLS

App Service te da por defecto un dominio `*.azurewebsites.net` con HTTPS habilitado. En producción, puedes configurar tu
propio dominio (`api.miempresa.com`) y App Service gestiona el certificado SSL automáticamente con **Managed
Certificates** gratuitos.

### 📌 Managed Identity

En lugar de usar usuario/contraseña para que tu app se conecte a otros servicios de Azure (como Azure SQL o Service
Bus), App Service puede usar una **Managed Identity**: una identidad gestionada por Azure que no requiere credenciales
explícitas. Es la forma más segura y recomendada de autenticación entre servicios Azure.

```java
// Con Managed Identity, no necesitas usuario/contraseña
// Azure maneja la autenticación automáticamente
TokenCredential credential = new DefaultAzureCredentialBuilder().build();
```

### 📌 Health Checks

App Service puede monitorear la salud de tu aplicación golpeando un endpoint de tu elección (por ejemplo
`/actuator/health` de Spring Boot Actuator). Si el endpoint falla, Azure automáticamente reinicia la instancia o la saca
del balanceador de carga.

---

## ⚖️ 5. App Service vs otras alternativas en Azure

Una pregunta común en entrevistas: *"¿Cuándo usarías App Service y cuándo no?"*

| Característica           | 🚀 App Service           | ⚡ Azure Functions          | 🐳 Azure Container Apps | ☸️ Azure Kubernetes Service   |
|--------------------------|--------------------------|----------------------------|-------------------------|-------------------------------|
| **Tipo**                 | PaaS                     | Serverless                 | PaaS (contenedores)     | IaaS (orquestación)           |
| **Unidad de despliegue** | App (.jar, .war, Docker) | Función individual         | Contenedor Docker       | Pod (contenedor)              |
| **Escalado**             | Manual / Auto            | Automático total           | Automático              | Manual / Auto                 |
| **Gestión de infra**     | Mínima                   | Ninguna                    | Mínima                  | Alta                          |
| **Ideal para**           | APIs REST, apps web      | Tareas puntuales, triggers | Microservicios modernos | Arquitecturas complejas       |
| **Costo base**           | Siempre activo           | Paga por ejecución         | Por uso                 | Alto (clúster siempre activo) |
| **Spring Boot**          | ✅ Soporte nativo         | ⚠️ Limitado                | ✅ Vía Docker            | ✅ Vía Docker                  |

> 💡 **Regla práctica:** Para una API REST Spring Boot tradicional con tráfico constante, **App Service es la elección
más directa**. Para tareas esporádicas o event-driven, Azure Functions. Para microservicios con muchos servicios
> independientes y equipos grandes, AKS o Container Apps.

---

## ☕ 6. Integración con Spring Boot

Así es como tu aplicación Spring Boot interactúa con Azure App Service:

### Dependencias necesarias en `pom.xml`

```xml
<!-- Plugin de Maven para despliegue directo a Azure App Service -->
<plugin>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-webapp-maven-plugin</artifactId>
    <version>2.13.0</version>
    <configuration>
        <subscriptionId>tu-subscription-id</subscriptionId>
        <resourceGroup>mi-grupo-recursos</resourceGroup>
        <appName>mi-spring-boot-app</appName>
        <region>brazilsouth</region>
        <pricingTier>B1</pricingTier>
        <runtime>
            <os>Linux</os>
            <javaVersion>Java 17</javaVersion>
            <webContainer>Java SE</webContainer>
        </runtime>
        <deployment>
            <resources>
                <resource>
                    <directory>${project.basedir}/target</directory>
                    <includes>
                        <include>*.jar</include>
                    </includes>
                </resource>
            </resources>
        </deployment>
    </configuration>
</plugin>

        <!-- Spring Boot Actuator para Health Checks -->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### Configuración en `application.properties`

```properties
# Puerto — App Service espera que la app escuche en el puerto 8080 por defecto
server.port=8080
#
# Actuator — expone /actuator/health para el Health Check de App Service
management.endpoints.web.exposure.include=health,info
management.endpoint.health.show-details=always
#
# La sintaxis ${VARIABLE} le dice a Spring Boot que lea el valor
# desde una variable de entorno en runtime.
# En Azure, esas variables de entorno se configuran como "Application Settings"
# en el portal (App Service → Configuration → Application Settings).
# Azure las inyecta automáticamente al arrancar la app.
# Lo que NO debe ir aquí son los valores reales (usuario, contraseña, URLs).
spring.datasource.url=${SPRING_DATASOURCE_URL}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD}
```

---

## 💻 7. Ejemplo de Código

### Escenario

Una API REST mínima con Spring Boot que está lista para ser desplegada en Azure App Service. Incluye un endpoint de
salud personalizado y lee configuración desde variables de entorno (Application Settings de Azure).

### Estructura del proyecto

```
mi-app-azure/
├── pom.xml
└── src/
    └── main/
        ├── java/com/ejemplo/azure/
        │   ├── MiAppAzureApplication.java
        │   ├── controller/
        │   │   └── SaludoController.java
        │   └── config/
        │       └── AppConfig.java
        └── resources/
            └── application.properties
```

### `MiAppAzureApplication.java`

```java
package com.ejemplo.azure;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MiAppAzureApplication {

    public static void main(String[] args) {
        SpringApplication.run(MiAppAzureApplication.class, args);
    }
}
```

### `AppConfig.java` — Lectura de Application Settings de Azure

```java
package com.ejemplo.azure.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

/**
 * Lee las variables de entorno configuradas como "Application Settings"
 * en el portal de Azure App Service.
 *
 * En Azure Portal: App Service → Configuration → Application Settings
 * Las variables de entorno se inyectan automáticamente en Spring Boot.
 */
@Configuration
public class AppConfig {

    // Azure App Service inyecta esta variable como Application Setting
    @Value("${app.ambiente:local}")
    private String ambiente;

    @Value("${app.version:1.0.0}")
    private String version;

    public String getAmbiente() {
        return ambiente;
    }

    public String getVersion() {
        return version;
    }
}
```

### `SaludoController.java` — API REST lista para App Service

```java
package com.ejemplo.azure.controller;

import com.ejemplo.azure.config.AppConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;
import java.util.Map;

@RestController
@RequestMapping("/api")
public class SaludoController {

    @Autowired
    private AppConfig appConfig;

    /**
     * Endpoint de prueba — verifica que la app está corriendo en App Service.
     * Acceso: GET https://mi-spring-boot-app.azurewebsites.net/api/info
     */
    @GetMapping("/info")
    public ResponseEntity<Map<String, Object>> info() {
        return ResponseEntity.ok(Map.of(
                "mensaje", "API corriendo en Azure App Service ✅",
                "ambiente", appConfig.getAmbiente(),   // Viene de Application Settings
                "version", appConfig.getVersion(),    // Viene de Application Settings
                "timestamp", LocalDateTime.now()
        ));
    }

    /**
     * Endpoint de salud personalizado.
     * App Service puede configurarse para golpear este endpoint
     * y verificar que la aplicación está saludable.
     * Acceso: GET https://mi-spring-boot-app.azurewebsites.net/api/salud
     */
    @GetMapping("/salud")
    public ResponseEntity<Map<String, String>> salud() {
        return ResponseEntity.ok(Map.of(
                "estado", "UP",
                "mensaje", "Aplicación funcionando correctamente"
        ));
    }
}
```

### `application.properties`

```properties
# Nombre de la aplicación
spring.application.name=mi-app-azure
# Puerto que App Service espera por defecto
server.port=8080
# Variables de entorno que se configuran en Azure Portal como Application Settings
# App Service las inyecta automáticamente — no hardcodear valores reales aquí
app.ambiente=${APP_AMBIENTE:local}
app.version=${APP_VERSION:1.0.0}
# Actuator: Health Check para App Service
management.endpoints.web.exposure.include=health,info
management.endpoint.health.show-details=always
management.endpoints.web.base-path=/actuator
```

### Respuesta esperada al llamar `/api/info`

```json
{
  "mensaje": "API corriendo en Azure App Service ✅",
  "ambiente": "produccion",
  "version": "2.1.0",
  "timestamp": "2025-03-15T10:30:00"
}
```

> 📌 Los valores `"produccion"` y `"2.1.0"` vienen de las **Application Settings** configuradas en Azure Portal, no del
`application.properties`. Esa es la práctica correcta en entornos reales.

---

## 🛠️ 8. Despliegue Paso a Paso (Conceptual)

Aunque no tengas cuenta Azure, es importante entender el flujo de despliegue porque es lo que te preguntarán en
entrevistas.

### Opción A — Despliegue con Maven Plugin (el más directo para Spring Boot)

```bash
# 1. Compilar y empaquetar tu app
mvn clean package -DskipTests

# 2. Autenticarte en Azure (se abre el navegador)
az login

# 3. Desplegar directamente a App Service
mvn azure-webapp:deploy

# ✅ Azure crea el App Service Plan y la Web App si no existen
# ✅ Sube el .jar y lo ejecuta automáticamente
# ✅ Tu API estará disponible en: https://mi-spring-boot-app.azurewebsites.net
```

### Opción B — Despliegue con Docker (más flexible)

```dockerfile
# Dockerfile para tu app Spring Boot
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Copia el .jar generado por Maven
COPY target/mi-app-azure-*.jar app.jar

# Puerto que App Service espera
EXPOSE 8080

# Comando de arranque
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```bash
# 1. Construir la imagen Docker
docker build -t mi-spring-app .

# 2. Subir imagen a Azure Container Registry (ACR)
az acr build --registry miRegistro --image mi-spring-app:v1 .

# 3. Configurar App Service para usar esa imagen
az webapp config container set \
  --name mi-spring-boot-app \
  --resource-group mi-grupo \
  --docker-custom-image-name miRegistro.azurecr.io/mi-spring-app:v1
```

### Opción C — CI/CD con GitHub Actions (lo que se usa en producción real)

```yaml
# .github/workflows/deploy-azure.yml
name: Deploy Spring Boot to Azure App Service

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout código
        uses: actions/checkout@v3

      - name: Configurar Java 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build con Maven
        run: mvn clean package -DskipTests

      - name: Desplegar a Azure App Service
        uses: azure/webapps-deploy@v2
        with:
          app-name: 'mi-spring-boot-app'
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: target/*.jar
```

> 📌 En producción real, el flujo más común es: **GitHub Actions → Build → Test → Deploy a Staging Slot → Pruebas de
integración → Swap a Production**. Así nunca hay downtime.

---

## 🎥 9. Recursos y Videos Recomendados

### 📺 Videos para ver en acción

| Recurso                                          | Descripción                                               | Dónde buscarlo                                                                                |
|--------------------------------------------------|-----------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| **Microsoft Learn — App Service**                | Documentación oficial interactiva con ejercicios guiados  | [learn.microsoft.com/azure/app-service](https://learn.microsoft.com/es-es/azure/app-service/) |
| **"Deploy Spring Boot to Azure App Service"**    | Tutorial oficial de Microsoft en YouTube                  | Buscar en YouTube: `"Deploy Spring Boot Azure App Service Microsoft"`                         |
| **"Azure App Service Tutorial for Beginners"**   | Visión general visual del portal y despliegue             | Buscar en YouTube: `"Azure App Service tutorial 2024"`                                        |
| **"Spring Boot on Azure" — Microsoft Developer** | Serie completa de Spring Boot + todos los servicios Azure | Buscar en YouTube: `"Spring Boot Azure Microsoft Developer"`                                  |

### 📖 Documentación oficial clave

| Documento                    | URL                                                                                                                                                   |
|------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| Documentación de App Service | [learn.microsoft.com/azure/app-service](https://learn.microsoft.com/es-es/azure/app-service/)                                                         |
| Java en App Service          | [learn.microsoft.com/azure/app-service/configure-language-java](https://learn.microsoft.com/es-es/azure/app-service/configure-language-java-security) |
| Azure SDK para Java (Spring) | [learn.microsoft.com/azure/developer/java/spring-framework](https://learn.microsoft.com/es-es/azure/developer/java/spring-framework/)                 |
| Plugin Maven para Azure      | [github.com/microsoft/azure-maven-plugins](https://github.com/microsoft/azure-maven-plugins)                                                          |

---

## 🧠 Resumen Ejecutivo

> Lo que deberías poder decir en una entrevista si te preguntan *"¿Qué es Azure App Service y cómo lo usarías con Spring
Boot?"*

**Azure App Service** es el servicio PaaS de Azure para desplegar y ejecutar aplicaciones web y APIs sin gestionar
servidores ni infraestructura. Soporta Java de forma nativa (JDK 8, 11, 17 y 21) y puede ejecutar aplicaciones Spring
Boot desplegando directamente el `.jar` o mediante contenedores Docker. El recurso de cómputo se define en el **App
Service Plan**, que determina CPU, RAM y precio; dentro de un plan puedes tener múltiples aplicaciones. Características
clave incluyen: **escalado horizontal automático** basado en métricas, **Deployment Slots** para despliegues sin
downtime, **Application Settings** para inyectar variables de entorno de forma segura sin hardcodear credenciales, y 
**Managed Identity** para autenticarse con otros servicios Azure sin usuario ni contraseña. En un proyecto real, Spring
Boot se integra con App Service mediante el plugin Maven de Azure o a través de un pipeline CI/CD con GitHub Actions que
despliega automáticamente en cada push a la rama principal.

---

## ⏭️ Siguiente módulo

> 🗃️ **Módulo 02 — Azure SQL:** La base de datos relacional gestionada de Azure. Veremos qué es, cómo se diferencia de
> un SQL Server local, sus niveles de servicio, y cómo conectarte desde Spring Boot usando Spring Data JPA e Hibernate.

---

*📝 Documentación elaborada como parte del plan de aprendizaje Azure para desarrolladores Java Backend.*  
*🗓️ Última actualización: 2025*
