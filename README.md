# ☁️ ¿Qué es Microsoft Azure?

> 📚 **Serie:** Azure para Java Developers  
> 🗂️ **Módulo:** 00 de 08  
> 🎯 **Objetivo:** Entender qué es Azure, cómo funciona y por qué es relevante para un desarrollador backend Java.

---

## 📋 Tabla de Contenidos

1. [¿Qué es la nube?](#-1-qué-es-la-nube-el-contexto-previo)
2. [¿Qué es Microsoft Azure?](#-2-qué-es-microsoft-azure)
3. [¿Cómo está organizado Azure?](#-3-cómo-está-organizado-azure-regiones-y-zonas)
4. [Modelos de servicio en la nube](#-4-modelos-de-servicio-en-la-nube-iaas-paas-saas)
5. [Servicios que estudiaremos](#-5-servicios-azure-que-estudiaremos)
6. [Azure en el mundo real](#-6-azure-en-el-mundo-real)
7. [Azure vs otros proveedores](#-7-azure-vs-aws-vs-gcp)
8. [¿Cómo se accede a Azure?](#-8-cómo-se-accede-a-azure)
9. [Recursos y videos](#-9-recursos-y-videos-recomendados)
10. [Resumen ejecutivo](#-resumen-ejecutivo)

---

## ☁️ 1. ¿Qué es la nube? (El contexto previo)

Antes de hablar de Azure, es importante entender qué significa **"la nube"**, porque es un término que se usa mucho pero
pocas veces se explica bien.

### La versión larga (con contexto real)

Imagina que tienes una startup y necesitas lanzar una aplicación web. En el año 2005, harías esto:

1. 🖥️ Compras servidores físicos (miles de dólares).
2. 🏢 Alquilas un espacio en un datacenter para colocarlos.
3. 🔌 Contratas electricidad, refrigeración y conectividad.
4. 👨‍💻 Contratas un equipo de infraestructura para mantenerlos.
5. ⏳ Todo ese proceso tarda meses antes de que puedas escribir una sola línea de código.
6. 📈 Si tu app tiene éxito y necesitas más capacidad, repites el proceso desde cero.

Ese modelo tiene un problema enorme: **pagas por la capacidad máxima que podrías necesitar, aunque el 90% del tiempo no
la uses**.

**La nube resuelve exactamente eso.** En lugar de comprar servidores, los *alquilas* por el tiempo que los necesitas.
¿Tu app tiene pico de tráfico los viernes por la noche? Alquilas más capacidad el viernes y la liberas el sábado.
¿Necesitas una base de datos solo para hacer pruebas? La creas en 2 minutos y la destruyes cuando terminas.

> 💡 **En resumen:** La nube es la capacidad de alquilar infraestructura tecnológica (servidores, bases de datos, redes,
> almacenamiento, etc.) de forma remota, por demanda y pagando solo por lo que usas.

---

## 🔷 2. ¿Qué es Microsoft Azure?

**Microsoft Azure** es la plataforma de computación en la nube de Microsoft. Es uno de los tres grandes proveedores de
nube a nivel mundial (junto con AWS de Amazon y GCP de Google).

### La versión larga (con contexto real)

Azure no es solo un conjunto de servidores virtuales. Es una **plataforma completa** que ofrece más de **200 servicios**
organizados en categorías: cómputo, almacenamiento, bases de datos, inteligencia artificial, mensajería, redes,
seguridad, monitoreo, y mucho más.

Fue lanzado en 2010 bajo el nombre "Windows Azure" y renombrado a "Microsoft Azure" en 2014. Desde entonces ha crecido
hasta ser el segundo proveedor de nube más grande del mundo, con una cuota de mercado de aproximadamente el 23%.

**¿Por qué las empresas eligen Azure sobre otros proveedores?**

- 🏢 **Integración con el ecosistema Microsoft:** Si una empresa ya usa Windows Server, Active Directory, Office 365, SQL
  Server o .NET, Azure se integra de forma nativa con todas esas herramientas. En Latinoamérica, el 80% de las empresas
  medianas y grandes usan infraestructura Microsoft.
- 🤝 **Soporte empresarial:** Microsoft tiene décadas de experiencia vendiendo a empresas (no a startups), lo que se
  traduce en contratos, SLAs y soporte de nivel enterprise muy maduros.
- 🌍 **Presencia global:** Azure opera en más de 60 regiones geográficas alrededor del mundo.
- 🔐 **Cumplimiento y seguridad:** Azure tiene certificaciones de cumplimiento para industrias muy reguladas (banca,
  salud, gobierno).

> 💡 **En resumen:** Azure es la plataforma cloud de Microsoft. Ofrece más de 200 servicios en la nube que permiten a
> empresas y desarrolladores construir, desplegar y escalar aplicaciones sin necesidad de administrar infraestructura
> física.

---

## 🌍 3. Cómo está organizado Azure (Regiones y Zonas)

Entender la organización geográfica de Azure es fundamental, porque cuando creas un recurso (una base de datos, un
servidor, etc.) siempre debes elegir **en qué región del mundo** quieres que viva.

### Jerarquía geográfica de Azure

```
🌎 Geography (Geografía)
└── 📍 Region (Región)
    └── 🏢 Availability Zone (Zona de Disponibilidad)
        └── 🖥️ Datacenter físico
```

| Concepto              | Descripción                                                  | Ejemplo                                  |
|-----------------------|--------------------------------------------------------------|------------------------------------------|
| **Geography**         | Agrupación de regiones por país o área geopolítica           | América, Europa, Asia-Pacífico           |
| **Region**            | Conjunto de datacenters en una ubicación geográfica          | `East US`, `Brazil South`, `West Europe` |
| **Availability Zone** | Datacenters físicamente separados dentro de una misma región | Zona 1, Zona 2, Zona 3                   |

### ¿Por qué importa la región?

- **Latencia:** Cuanto más cerca esté el servidor de tus usuarios, más rápida será la respuesta.
- **Cumplimiento legal:** En algunos países, los datos no pueden salir del territorio nacional (ej. datos bancarios en
  Perú deben estar en servidores en Perú o en una región aprobada).
- **Precio:** El mismo servicio puede costar diferente según la región.
- **Disponibilidad de servicios:** No todos los servicios de Azure están disponibles en todas las regiones.

> 📌 Para Perú y Latinoamérica, las regiones más comunes son **Brazil South** (São Paulo) y **East US** (Virginia). La
> región de Brasil suele ser preferida por temas de latencia y regulación de datos.

---

## 🏗️ 4. Modelos de servicio en la nube: IaaS, PaaS, SaaS

Este es uno de los conceptos más importantes de la nube y uno de los que más aparece en entrevistas. Los tres modelos
definen **cuánto control tienes tú vs. cuánto gestiona el proveedor**.

### Analogía del restaurant 🍽️

| Modelo                    | Analogía                 | Tú gestionas            | Azure gestiona                |
|---------------------------|--------------------------|-------------------------|-------------------------------|
| **On-Premise** (sin nube) | Cocinas en casa          | Todo                    | Nada                          |
| **IaaS**                  | Cocina alquilada         | SO, runtime, app, datos | Hardware, red, virtualización |
| **PaaS**                  | Pides comida para llevar | App y datos             | Todo lo demás                 |
| **SaaS**                  | Comes en un restaurant   | Solo usas el servicio   | Todo                          |

### IaaS — Infrastructure as a Service

Tú alquilas la infraestructura (servidores virtuales, redes, almacenamiento) pero instalas y gestionas todo lo demás: el
sistema operativo, el runtime de Java, el servidor de aplicaciones, etc.

**Ejemplo Azure:** Azure Virtual Machines (una VM donde instalas tú el JDK y tu app).

**¿Cuándo se usa?** Cuando necesitas control total sobre el entorno, o cuando estás migrando una app legacy que no puede
modificarse.

### PaaS — Platform as a Service

Tú despliegas tu **código y datos**. Azure gestiona el sistema operativo, el runtime, la escalabilidad y la
disponibilidad.

**Ejemplo Azure:** Azure App Service (subes tu `.jar` de Spring Boot y Azure lo ejecuta, escala y mantiene disponible).

**¿Cuándo se usa?** Para la mayoría de aplicaciones modernas. Es el modelo más común para desarrolladores backend.

### SaaS — Software as a Service

Consumes un software completamente listo. No hay nada que instalar ni configurar.

**Ejemplo Azure:** Microsoft 365, Azure DevOps, GitHub (de Microsoft).

**¿Cuándo se usa?** Para herramientas de productividad y colaboración.

> 💡 **En resumen:** Como desarrollador Java backend, trabajarás principalmente con **PaaS** (despliegas tu app y Azure
> gestiona la infraestructura) y consumirás servicios **SaaS** como bases de datos gestionadas, colas de mensajes y
> almacenamiento.

---

## 🗂️ 5. Servicios Azure que estudiaremos

En esta serie nos enfocaremos en los servicios más demandados en el mercado laboral peruano para desarrolladores Java:

| #  | Ícono | Servicio              | Categoría                | ¿Para qué sirve?                                                      | 🧭 Razonamiento del orden                                                                                                         |
|----|-------|-----------------------|--------------------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| 01 | 🚀    | **Azure App Service** | Cómputo / PaaS           | Desplegar y ejecutar aplicaciones web y APIs sin gestionar servidores | Va primero porque es donde *vive* tu app Spring Boot. Entender esto da contexto a todos los demás servicios que tu app consumirá. |
| 02 | 🗃️   | **Azure SQL**         | Base de datos relacional | Base de datos SQL gestionada, compatible con SQL Server               | Tu app ya corre en App Service, ahora necesita persistir datos relacionales. Es lo más cercano a tu mundo actual.                 |
| 03 | 🌌    | **Cosmos DB**         | Base de datos NoSQL      | Base de datos distribuida y multi-modelo                              | Contraste natural con Azure SQL: relacional vs. NoSQL, estudiados uno seguido del otro.                                           |
| 04 | 📦    | **Azure Storage**     | Almacenamiento           | Blobs, archivos, colas y tablas en la nube                            | Tu app ya tiene datos estructurados; ahora necesita guardar archivos, imágenes y documentos. Concepto transversal a casi todo.    |
| 05 | ⚡     | **Azure Functions**   | Serverless / Cómputo     | Ejecutar código sin gestionar servidores                              | Complemento de App Service para lógica puntual y tareas sin estado. Tiene más sentido después de ver App Service.                 |
| 06 | 📨    | **Service Bus**       | Mensajería               | Cola de mensajes empresarial (pub/sub)                                | Comunicación asíncrona entre microservicios. Muy relevante para arquitecturas reactivas con WebFlux.                              |
| 07 | 🌊    | **Event Hubs**        | Streaming de eventos     | Ingesta masiva de eventos en tiempo real                              | Complemento natural del Service Bus: mensajería puntual vs. streaming masivo, juntos tienen más sentido.                          |
| 08 | 🚪    | **API Management**    | API Gateway              | Publicar, proteger y monitorear APIs                                  | Va último porque para apreciarlo necesitas entender todos los servicios que expone y gestiona.                                    |

---

## 🏢 6. Azure en el mundo real

¿Qué tipo de aplicaciones se construyen con Azure? Aquí algunos ejemplos reales:

### Arquitectura típica de una aplicación bancaria en Azure

```
[Mobile App / Web App]
        │
        ▼
[API Management]          ← Punto de entrada único para todas las APIs
        │
        ▼
[Azure Functions / App Service]   ← Lógica de negocio (tu código Java)
        │
   ┌────┴────┐
   ▼         ▼
[Azure SQL]  [Cosmos DB]   ← Datos relacionales y NoSQL
        │
        ▼
[Service Bus]              ← Comunicación asíncrona entre microservicios
        │
        ▼
[Event Hubs]               ← Registro de eventos y auditoría
        │
        ▼
[Azure Storage]            ← Archivos, imágenes, documentos
```

> 📌 Este tipo de arquitectura es exactamente lo que empresas como bancos, fintechs y retailers en Perú están
> construyendo. Por eso estos servicios aparecen en casi todas las ofertas laborales del sector.

---

## ⚖️ 7. Azure vs AWS vs GCP

Una pregunta común en entrevistas: *"¿Por qué Azure y no AWS o GCP?"*

| Característica       | ☁️ Azure                           | 🟠 AWS                           | 🔵 GCP                           |
|----------------------|------------------------------------|----------------------------------|----------------------------------|
| **Cuota de mercado** | ~23% (2do)                         | ~32% (1ro)                       | ~11% (3ro)                       |
| **Fortaleza**        | Empresas y ecosistema Microsoft    | Startups y variedad de servicios | Machine Learning y Big Data      |
| **Integración**      | Excelente con .NET, SQL Server, AD | Universal                        | Excelente con tecnologías Google |
| **Perú / LATAM**     | Muy fuerte en banca y retail       | Fuerte en tech y startups        | Creciendo en ML/AI               |
| **Certificaciones**  | AZ-900, AZ-204, etc.               | AWS Cloud Practitioner, etc.     | Associate Cloud Engineer, etc.   |

> 💡 En el mercado laboral peruano, **Azure domina en el sector financiero y retail tradicional**, que son los que más
> pagan y más contratan desarrolladores backend Java senior.

---

## 🛠️ 8. Cómo se accede a Azure

Aunque no tengas cuenta, es importante saber cómo se trabaja con Azure en el día a día:

### 1. 🌐 Azure Portal

Interfaz web en [portal.azure.com](https://portal.azure.com). Es el panel de control visual donde puedes crear,
configurar y monitorear todos tus recursos.

### 2. 💻 Azure CLI

Herramienta de línea de comandos para gestionar recursos desde la terminal.

```bash
# Ejemplo: listar todos los grupos de recursos
az group list --output table

# Ejemplo: crear una base de datos Azure SQL
az sql db create --resource-group miGrupo --server miServidor --name miBaseDeDatos
```

### 3. ☕ SDK para Java / Spring Boot

Para integrar servicios Azure desde tu código Java, usas las librerías oficiales del **Azure SDK for Java**.

```xml
<!-- Dependencia Maven para Azure SDK -->
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-identity</artifactId>
    <version>1.11.0</version>
</dependency>
```

### 4. 🔧 Terraform / Bicep

Herramientas de **Infraestructura como Código (IaC)** para definir y desplegar recursos Azure de forma declarativa y
reproducible.

```hcl
# Ejemplo Terraform: crear un grupo de recursos en Azure
resource "azurerm_resource_group" "ejemplo" {
  name     = "mi-grupo-recursos"
  location = "Brazil South"
}
```

### 5. 🔑 Autenticación con Azure Active Directory (AAD)

En el mundo real, tu app Spring Boot no usa usuario/contraseña para conectarse a Azure. Usa **Managed Identities** y
**Service Principals** con tokens OAuth2. Esto es un concepto clave para entrevistas.

---

## 🎥 9. Recursos y Videos Recomendados

### 📺 Para entender Azure desde cero (en español)

| Recurso                      | Descripción                                                            | Enlace                                                                                                  |
|------------------------------|------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| **Microsoft Learn**          | Curso oficial gratuito: "Aspectos básicos de Microsoft Azure (AZ-900)" | [learn.microsoft.com](https://learn.microsoft.com/es-es/training/paths/az-900-describe-cloud-concepts/) |
| **YouTube - Absolute Azure** | Canal en inglés con tutoriales visuales claros                         | Buscar: "What is Microsoft Azure"                                                                       |
| **YouTube - Pelado Nerd**    | Canal en español sobre cloud y DevOps, incluye Azure                   | Buscar: "Azure tutorial Pelado Nerd"                                                                    |
| **freeCodeCamp (YouTube)**   | Curso completo AZ-900 de 3 horas en inglés                             | Buscar: "AZ-900 Azure Fundamentals freeCodeCamp"                                                        |

### 📖 Documentación oficial

- [Azure Architecture Center](https://learn.microsoft.com/es-es/azure/architecture/) — Patrones y arquitecturas de
  referencia.
- [Azure para desarrolladores Java](https://learn.microsoft.com/es-es/azure/developer/java/) — Punto de entrada oficial
  para integraciones Java/Spring Boot.

---

## 🧠 Resumen Ejecutivo

> Este es el resumen corto que deberías poder decir en una entrevista si te preguntan *"¿Qué es Azure?"*

**Microsoft Azure** es la plataforma de computación en la nube de Microsoft. Permite a las empresas construir, desplegar
y escalar aplicaciones sin gestionar infraestructura física, pagando solo por lo que usan. Ofrece más de 200 servicios
en categorías como cómputo, almacenamiento, bases de datos, mensajería, inteligencia artificial y seguridad. Es el
segundo proveedor cloud más grande del mundo y el dominante en el sector financiero y empresarial en Latinoamérica,
especialmente por su integración nativa con el ecosistema Microsoft (SQL Server, Active Directory, .NET). Para un
desarrollador Java backend, Azure es relevante porque sus servicios de bases de datos, mensajería y APIs son parte
central de las arquitecturas de microservicios modernas.

---

## ⏭️ Siguiente módulo

> 🚀 **Módulo 01 — Azure App Service:** El servicio PaaS de Azure para desplegar aplicaciones web y APIs. Veremos qué es,
> cómo funciona, sus planes de precios, y cómo desplegar una app Spring Boot directamente desde Maven o un contenedor
> Docker.

---

*📝 Documentación elaborada como parte del plan de aprendizaje Azure para desarrolladores Java Backend.*  
*🗓️ Última actualización: 2025*
