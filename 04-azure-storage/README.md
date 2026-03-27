# 📦 Módulo 04 — Azure Storage

> 📚 **Serie:** Azure para Java Developers  
> 🗂️ **Módulo:** 04 de 08  
> 🎯 **Objetivo:** Entender qué es Azure Storage, sus cuatro tipos de almacenamiento, cuándo usar cada uno, y cómo subir,
> descargar y gestionar archivos desde Spring Boot.

---

## 📋 Tabla de Contenidos

1. [¿Qué es Azure Storage?](#-1-qué-es-azure-storage)
2. [Los cuatro tipos de almacenamiento](#-2-los-cuatro-tipos-de-almacenamiento)
3. [Blob Storage a fondo](#-3-blob-storage-a-fondo-el-más-importante)
4. [Niveles de acceso (Access Tiers)](#-4-niveles-de-acceso-access-tiers)
5. [Conceptos clave de seguridad](#-5-conceptos-clave-de-seguridad)
6. [Azure Storage vs base de datos](#-6-azure-storage-vs-base-de-datos-un-error-común)
7. [Integración con Spring Boot](#-7-integración-con-spring-boot)
8. [Ejemplo de código](#-8-ejemplo-de-código)
9. [Casos de uso reales](#-9-casos-de-uso-reales-en-el-mundo-laboral)
10. [Recursos y videos](#-10-recursos-y-videos-recomendados)
11. [Resumen ejecutivo](#-resumen-ejecutivo)

---

## 📦 1. ¿Qué es Azure Storage?

**Azure Storage** es el servicio de almacenamiento de objetos de Microsoft Azure. Permite guardar cualquier tipo de dato
no estructurado — archivos, imágenes, videos, documentos PDF, backups, logs, datos binarios — de forma masiva, duradera
y económica, con acceso desde cualquier parte del mundo a través de HTTP/HTTPS.

### La versión larga (con contexto real)

Piensa en cómo funciona una aplicación empresarial real. Tu base de datos (Azure SQL o Cosmos DB) guarda datos
estructurados: nombres, precios, fechas, montos. Pero las aplicaciones también manejan otro tipo de información:

- 🖼️ La foto de perfil del usuario.
- 📄 El contrato PDF que el cliente firmó digitalmente.
- 🎥 El video tutorial del producto.
- 📊 El reporte Excel que se genera cada mes.
- 📦 El backup de la base de datos de ayer.
- 📝 Los logs de auditoría de las últimas 24 horas.

**¿Podrías guardar todo eso en una base de datos?** Técnicamente sí, como campos `BLOB` o `VARBINARY`. Pero es una mala
práctica por varias razones:

- Las bases de datos son caras por GB comparado con el almacenamiento de objetos.
- Los archivos grandes degradan el rendimiento de las queries.
- Las bases de datos no están optimizadas para servir archivos directamente a clientes web.
- El escalado de almacenamiento en una base de datos es complejo y costoso.

**Azure Storage resuelve todo eso.** Es un servicio diseñado específicamente para almacenar archivos a escala 
masiva con estas garantías:

| Garantía              | Detalle                                                               |
|-----------------------|-----------------------------------------------------------------------|
| 🛡️ **Durabilidad**   | 99.999999999% (11 nueves) — prácticamente imposible perder un archivo |
| 📈 **Escala**         | Hasta 5 PB (petabytes) por cuenta de storage — sin límite práctico    |
| 🌍 **Disponibilidad** | 99.9% a 99.99% según configuración                                    |
| 🔐 **Seguridad**      | Cifrado AES-256 en reposo por defecto, HTTPS obligatorio              |
| 💰 **Costo**          | Desde $0.018 por GB/mes — mucho más barato que una base de datos      |

> 💡 **En resumen:** Azure Storage es el "disco duro gigante en la nube" de tu aplicación. Todo lo que no encaja en una
> base de datos relacional o NoSQL — archivos, imágenes, videos, documentos — va a Azure Storage.

---

## 🗂️ 2. Los Cuatro Tipos de Almacenamiento

Azure Storage no es un solo servicio, es una **cuenta de almacenamiento** que ofrece cuatro tipos de almacenamiento con
propósitos distintos:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Cuenta de Azure Storage                      │
│              (miempresa.blob.core.windows.net)                  │
│                                                                 │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───┐   │
│   │     Blob     │  │    Files     │  │    Queue     │  │Tbl│   │
│   │   Storage    │  │   Storage    │  │   Storage    │  │   │   │
│   │              │  │              │  │              │  │   │   │
│   │ Archivos,    │  │ Carpetas     │  │ Mensajes     │  │K/V│   │
│   │ imágenes,    │  │ compartidas  │  │ entre        │  │   │   │
│   │ videos, PDFs │  │ (como NFS)   │  │ servicios    │  │   │   │
│   └──────────────┘  └──────────────┘  └──────────────┘  └───┘   │
│       El más usado      Para VMs        Para colas       Legacy │
└─────────────────────────────────────────────────────────────────┘
```

### 📁 Blob Storage — El más importante

**Blob = Binary Large Object.** Es el almacenamiento de objetos de Azure. Ideal para cualquier archivo: imágenes, PDFs,
videos, backups, logs, datos de ML.

**¿Cuándo usarlo?**

- Subir y servir archivos desde una app web o móvil.
- Almacenar backups de bases de datos.
- Guardar logs y datos de auditoría.
- Hospedar imágenes estáticas de un e-commerce.
- Almacenar datos de entrada/salida de Azure Functions.

### 📂 Azure Files — Carpetas compartidas en red

Sistema de archivos compartido accesible vía protocolo SMB (el mismo que usan las carpetas de red en Windows) o NFS.
Las VMs y contenedores pueden montarlo como una unidad de red local.

**¿Cuándo usarlo?**

- Aplicaciones legacy que esperan archivos en una ruta de red (`\\servidor\carpeta`).
- Compartir archivos de configuración entre múltiples instancias de una app.
- Migración de aplicaciones on-premise que usan file shares.

### 📨 Queue Storage — Cola de mensajes simple

Permite enviar y recibir mensajes entre componentes de una aplicación. Cada mensaje puede tener hasta 64 KB y persiste
hasta 7 días.

**¿Cuándo usarlo?**

- Desacoplar componentes: el frontend encola una tarea y el backend la procesa cuando puede.
- Comunicación simple entre servicios sin necesidad de Service Bus.
- Procesar tareas en segundo plano (envío de emails, generación de reportes).

> ⚠️ **Importante:** Queue Storage es para casos simples. Si necesitas garantías de entrega, dead-letter queues, topics
> o suscripciones, usa **Azure Service Bus** (Módulo 06).

### 📋 Table Storage — Clave-Valor estructurado (Legacy)

Almacenamiento NoSQL de clave-valor semi-estructurado. Es el antecesor de Cosmos DB Table API y hoy en día se considera
legacy.

**¿Cuándo usarlo?**

- Casi nunca en proyectos nuevos. Para casos de uso NoSQL simples, Cosmos DB es superior.
- Mantener compatibilidad con aplicaciones existentes que ya lo usan.

---

## 🔍 3. Blob Storage a Fondo — El más importante

Dado que Blob Storage es el que más usarás como desarrollador Java backend, profundizamos en su estructura y conceptos.

### Jerarquía de Blob Storage

```
┌─────────────────────────────────────────────────────────────┐
│              Cuenta de Storage                              │
│         miempresa.blob.core.windows.net                     │
│                                                             │
│   ┌─────────────────────┐   ┌───────────────────────────┐   │
│   │  Contenedor:        │   │  Contenedor:              │   │
│   │  "imagenes-perfil"  │   │  "documentos-contratos"   │   │
│   │  (acceso público)   │   │  (acceso privado)         │   │
│   │                     │   │                           │   │
│   │  📷 user-001.jpg    │   │  📄 contrato-2024-01.pdf  │   │
│   │  📷 user-002.jpg    │   │  📄 contrato-2024-02.pdf  │   │
│   │  📷 user-003.png    │   │  📄 poliza-seguro.pdf     │   │
│   └─────────────────────┘   └───────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

| Concepto                   | Descripción                          | Equivalente                  |
|----------------------------|--------------------------------------|------------------------------|
| **Cuenta de Storage**      | Punto de entrada, define la URL base | Como un servidor de archivos |
| **Contenedor (Container)** | Agrupa blobs relacionados            | Como una carpeta raíz        |
| **Blob**                   | El archivo en sí                     | Como un archivo              |

### Los tres tipos de Blob

| Tipo             | Para qué                                    | Características                                                    |
|------------------|---------------------------------------------|--------------------------------------------------------------------|
| **Block Blob** ⭐ | Archivos en general: imágenes, PDFs, videos | El más común. Optimizado para subidas y descargas. Hasta 190 TB.   |
| **Append Blob**  | Logs que crecen continuamente               | Solo permite agregar datos al final. Ideal para logs de auditoría. |
| **Page Blob**    | Discos virtuales de VMs                     | Optimizado para acceso aleatorio. Lo usan los discos de Azure VMs. |

> 📌 Como desarrollador backend Java, trabajarás casi exclusivamente con **Block Blobs**.

### URL de un Blob

Cada blob tiene una URL única y predecible:

```
https://[cuenta].blob.core.windows.net/[contenedor]/[nombre-del-blob]

Ejemplo:
https://miempresa.blob.core.windows.net/imagenes-perfil/user-001.jpg
https://miempresa.blob.core.windows.net/documentos-contratos/contrato-2024-01.pdf
```

---

## 🌡️ 4. Niveles de Acceso (Access Tiers)

El **Access Tier** define el trade-off entre costo de almacenamiento y costo de acceso.

```
ACCESO FRECUENTE ◄───────────────────────────────────► ARCHIVADO

     Hot                  Cool                  Archive
  (Caliente)             (Frío)               (Archivado)
      │                    │                      │
      ▼                    ▼                      ▼
 Mayor costo de        Menor costo de         Mínimo costo de
 storage               storage                storage
 Menor costo de        Mayor costo de         Máximo costo de
 acceso                acceso                 acceso + rehidratación
      │                    │                  de 1 a 15 horas
      ▼                    ▼                      │
 Fotos de perfil,     Backups recientes,          ▼
 documentos activos   facturas de hace        Backups anuales,
                      3 meses                 datos legales SBS
```

| Tier        | Costo almacenamiento | Costo acceso | Recuperación |
|-------------|----------------------|--------------|--------------|
| **Hot**     | ~$0.018/GB/mes       | Mínimo       | Inmediato    |
| **Cool**    | ~$0.01/GB/mes        | Medio        | Inmediato    |
| **Cold**    | ~$0.004/GB/mes       | Alto         | Inmediato    |
| **Archive** | ~$0.001/GB/mes       | Muy alto     | 1–15 horas   |

> 💡 **Caso real en Perú:** Facturas del mes → Hot. Facturas de hace 3 meses → Cool. Facturas del año pasado → Cold.
> Facturas de hace 5 años (requisito SBS) → Archive.

---

## 🔐 5. Conceptos Clave de Seguridad

### 🔑 Storage Account Keys

Cada cuenta tiene dos Access Keys que dan acceso total. Nunca en el código fuente.

```properties
# ❌ MAL — nunca hardcodear
azure.storage.connection-string=DefaultEndpointsProtocol=https;AccountKey=clave_real...
```

```properties
# ✅ BIEN — desde Application Settings de Azure
azure.storage.connection-string=${AZURE_STORAGE_CONNECTION_STRING}
```

### 🎫 Shared Access Signature (SAS Token)

URL firmada que da acceso **temporal y limitado** a un blob, sin exponer las claves.

```
URL con SAS Token (acceso de 1 hora, solo lectura):
https://miempresa.blob.core.windows.net/contratos/contrato-001.pdf
  ?sv=2023-01-03
  &se=2025-03-15T11%3A00%3A00Z   ← Expira en 1 hora
  &sr=b                           ← Aplica a un blob
  &sp=r                           ← Solo lectura
  &sig=ABC123XYZ...               ← Firma criptográfica
```

**Parámetros clave:**

- `se` → Expiración (expiry time).
- `sp` → Permisos: `r` leer, `w` escribir, `d` eliminar.
- `sr` → Recurso: `b` blob, `c` contenedor.

> 📌 **Flujo real:** Usuario pide descargar contrato → backend genera SAS URL de 5 minutos → frontend redirige al
> usuario → descarga directa desde Azure Storage sin pasar por el backend.

### 🔒 Niveles de acceso del contenedor

| Nivel         | Acceso                       | Usar cuando                                    |
|---------------|------------------------------|------------------------------------------------|
| **Private**   | Solo con clave o SAS         | Contratos, facturas, documentos confidenciales |
| **Blob**      | Cualquiera con la URL exacta | Imágenes de productos, recursos públicos       |
| **Container** | Cualquiera puede listar todo | Evitar en producción                           |

### 🛡️ Managed Identity

```java
// Sin contraseñas — Azure gestiona el token automáticamente
BlobServiceClient client = new BlobServiceClientBuilder()
                .endpoint("https://miempresa.blob.core.windows.net")
                .credential(new DefaultAzureCredentialBuilder().build())
                .buildClient();
```

---

## ⚖️ 6. Azure Storage vs Base de Datos — Un error común

| Criterio                | 🗄️ Base de Datos                    | 📦 Azure Storage (Blob)            |
|-------------------------|--------------------------------------|------------------------------------|
| **Tipo de dato**        | Datos estructurados (texto, números) | Archivos binarios (imágenes, PDFs) |
| **Consulta**            | SQL, filtros, JOINs                  | Solo por URL o nombre del blob     |
| **Tamaño típico**       | Bytes a kilobytes                    | Kilobytes a gigabytes              |
| **Costo por GB**        | Alto (~$0.12–$0.50/GB/mes)           | Bajo (~$0.018/GB/mes)              |
| **Servir archivos web** | ❌ No optimizado                      | ✅ CDN integrado, URLs directas     |
| **Transacciones ACID**  | ✅ Sí                                 | ❌ No                               |

**El patrón correcto:**

```
Base de Datos                          Azure Storage
┌──────────────────────────┐           ┌──────────────────────────┐
│ tabla: usuarios          │           │ Contenedor: fotos-perfil │
│                          │           │                          │
│ id: 1                    │           │ 📷 user-1-foto.jpg       │
│ nombre: "Juan"           │  ──────►  │ 📷 user-2-foto.jpg       │
│ foto_url: "...user-1.jpg"│           │ 📷 user-3-foto.jpg       │
└──────────────────────────┘           └──────────────────────────┘
  Guarda la URL (texto barato)           Guarda el archivo real
```

> 💡 **Regla de oro:** En la base de datos guardas la **URL o nombre del blob**. En Azure Storage guardas el
> **archivo real**. Nunca al revés.

---

## ☕ 7. Integración con Spring Boot

### Dependencias en `pom.xml`

```xml

<dependencies>
    <!-- SDK oficial de Azure para Blob Storage -->
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-storage-blob</artifactId>
        <version>12.25.0</version>
    </dependency>

    <!-- Azure Identity — Managed Identity sin contraseña -->
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-identity</artifactId>
        <version>1.11.0</version>
    </dependency>

    <!-- Spring Web — para endpoints multipart/form-data -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### `application.properties`

```properties
# Cuenta de storage (nombre sin el .blob.core.windows.net)
azure.storage.account-name=${AZURE_STORAGE_ACCOUNT_NAME}
# Connection string (desarrollo) — en producción usar Managed Identity
azure.storage.connection-string=${AZURE_STORAGE_CONNECTION_STRING}
# Nombre del contenedor
azure.storage.container-name=${AZURE_STORAGE_CONTAINER_NAME}
# Expiración de SAS Tokens en minutos
azure.storage.sas-expiry-minutes=60
# Límite de tamaño de archivos en Spring Boot
spring.servlet.multipart.max-file-size=50MB
spring.servlet.multipart.max-request-size=50MB
```

### `BlobStorageConfig.java`

```java
package com.ejemplo.storage.config;

import com.azure.storage.blob.BlobServiceClient;
import com.azure.storage.blob.BlobServiceClientBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BlobStorageConfig {

    @Value("${azure.storage.connection-string}")
    private String connectionString;

    /**
     * BlobServiceClient es thread-safe — se registra como singleton @Bean.
     *
     * Para producción con Managed Identity:
     *   new BlobServiceClientBuilder()
     *       .endpoint("https://{cuenta}.blob.core.windows.net")
     *       .credential(new DefaultAzureCredentialBuilder().build())
     *       .buildClient();
     */
    @Bean
    public BlobServiceClient blobServiceClient() {
        return new BlobServiceClientBuilder()
                .connectionString(connectionString)
                .buildClient();
    }
}
```

---

## 💻 8. Ejemplo de Código

### Escenario

Sistema de gestión de documentos: subir archivos, obtener URL temporal de descarga (SAS Token), listar y eliminar. La
URL del archivo se guarda en la base de datos.

### Estructura del proyecto

```
mi-app-storage/
└── src/main/java/com/ejemplo/storage/
    ├── config/
    │   └── BlobStorageConfig.java
    ├── service/
    │   └── BlobStorageService.java
    ├── controller/
    │   └── ArchivoController.java
    └── dto/
        ├── ArchivoSubidoResponse.java
        └── ArchivoUrlResponse.java
```

### `BlobStorageService.java`

```java
package com.ejemplo.storage.service;

import com.azure.storage.blob.BlobClient;
import com.azure.storage.blob.BlobContainerClient;
import com.azure.storage.blob.BlobServiceClient;
import com.azure.storage.blob.models.BlobItem;
import com.azure.storage.blob.sas.BlobSasPermission;
import com.azure.storage.blob.sas.BlobServiceSasSignatureValues;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.io.InputStream;
import java.time.OffsetDateTime;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;

@Service
public class BlobStorageService {

    @Autowired
    private BlobServiceClient blobServiceClient;

    @Value("${azure.storage.container-name}")
    private String containerName;

    @Value("${azure.storage.sas-expiry-minutes}")
    private int sasExpiryMinutes;

    /**
     * Sube un archivo a Azure Blob Storage.
     *
     * Flujo:
     * 1. Genera nombre único con UUID + extensión original.
     * 2. Obtiene el cliente del contenedor.
     * 3. Sube el stream del archivo.
     * 4. Retorna la URL pública del blob.
     */
    public String subirArchivo(MultipartFile archivo, String carpeta) throws IOException {
        String extension = obtenerExtension(archivo.getOriginalFilename());
        String nombreBlob = carpeta + "/" + UUID.randomUUID() + "." + extension;

        BlobContainerClient containerClient = blobServiceClient
                .getBlobContainerClient(containerName);
        BlobClient blobClient = containerClient.getBlobClient(nombreBlob);

        try (InputStream inputStream = archivo.getInputStream()) {
            blobClient.upload(inputStream, archivo.getSize(), true);
            // true = sobreescribir si existe el blob con ese nombre
        }

        return blobClient.getBlobUrl();
    }

    /**
     * Genera URL temporal con SAS Token para acceso privado y seguro.
     *
     * El archivo va directo de Azure Storage al usuario — sin pasar por el backend.
     * Esto ahorra ancho de banda y es la práctica recomendada en producción.
     */
    public String generarUrlTemporalDescarga(String nombreBlob) {
        BlobClient blobClient = blobServiceClient
                .getBlobContainerClient(containerName)
                .getBlobClient(nombreBlob);

        BlobSasPermission permisos = new BlobSasPermission()
                .setReadPermission(true);  // Solo lectura

        BlobServiceSasSignatureValues sasValues = new BlobServiceSasSignatureValues(
                OffsetDateTime.now().plusMinutes(sasExpiryMinutes),
                permisos
        );

        String sasToken = blobClient.generateSas(sasValues);
        return blobClient.getBlobUrl() + "?" + sasToken;
    }

    /**
     * Elimina un blob — verifica existencia antes para evitar excepciones.
     */
    public boolean eliminarArchivo(String nombreBlob) {
        BlobClient blobClient = blobServiceClient
                .getBlobContainerClient(containerName)
                .getBlobClient(nombreBlob);

        if (blobClient.exists()) {
            blobClient.delete();
            return true;
        }
        return false;
    }

    /**
     * Lista blobs dentro de una carpeta (prefijo) del contenedor.
     */
    public List<String> listarArchivos(String carpeta) {
        return blobServiceClient
                .getBlobContainerClient(containerName)
                .listBlobsByHierarchy(carpeta)
                .stream()
                .map(BlobItem::getName)
                .collect(Collectors.toList());
    }

    public boolean existeArchivo(String nombreBlob) {
        return blobServiceClient
                .getBlobContainerClient(containerName)
                .getBlobClient(nombreBlob)
                .exists();
    }

    private String obtenerExtension(String nombreArchivo) {
        if (nombreArchivo == null || !nombreArchivo.contains(".")) return "bin";
        return nombreArchivo.substring(nombreArchivo.lastIndexOf(".") + 1).toLowerCase();
    }
}
```

### DTOs de respuesta

```java
package com.ejemplo.storage.dto;

public class ArchivoSubidoResponse {
    private String nombreBlob;  // Guardar este valor en la base de datos
    private String urlPublica;
    private String mensaje;

    public ArchivoSubidoResponse(String nombreBlob, String urlPublica, String mensaje) {
        this.nombreBlob = nombreBlob;
        this.urlPublica = urlPublica;
        this.mensaje = mensaje;
    }

    public String getNombreBlob() {
        return nombreBlob;
    }

    public String getUrlPublica() {
        return urlPublica;
    }

    public String getMensaje() {
        return mensaje;
    }
}
```

```java
package com.ejemplo.storage.dto;

public class ArchivoUrlResponse {
    private String urlDescarga;    // URL con SAS Token — temporal
    private int expiraEnMinutos;
    private String mensaje;

    public ArchivoUrlResponse(String urlDescarga, int expiraEnMinutos, String mensaje) {
        this.urlDescarga = urlDescarga;
        this.expiraEnMinutos = expiraEnMinutos;
        this.mensaje = mensaje;
    }

    public String getUrlDescarga() {
        return urlDescarga;
    }

    public int getExpiraEnMinutos() {
        return expiraEnMinutos;
    }

    public String getMensaje() {
        return mensaje;
    }
}
```

### `ArchivoController.java`

```java
package com.ejemplo.storage.controller;

import com.ejemplo.storage.dto.ArchivoSubidoResponse;
import com.ejemplo.storage.dto.ArchivoUrlResponse;
import com.ejemplo.storage.service.BlobStorageService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.util.List;
import java.util.Map;

@RestController
@RequestMapping("/api/archivos")
public class ArchivoController {

    @Autowired
    private BlobStorageService blobStorageService;

    @Value("${azure.storage.sas-expiry-minutes}")
    private int sasExpiryMinutes;

    /**
     * POST /api/archivos/subir?carpeta=contratos
     * Recibe archivo como multipart/form-data, lo sube a Azure Storage
     * y retorna la URL. Esa URL se guarda en la base de datos.
     */
    @PostMapping("/subir")
    public ResponseEntity<ArchivoSubidoResponse> subirArchivo(
            @RequestParam("archivo") MultipartFile archivo,
            @RequestParam(value = "carpeta", defaultValue = "general") String carpeta) {

        if (archivo.isEmpty()) return ResponseEntity.badRequest().build();

        try {
            String urlBlob = blobStorageService.subirArchivo(archivo, carpeta);
            String nombreBlob = carpeta + "/" + urlBlob.substring(urlBlob.lastIndexOf("/") + 1);

            return ResponseEntity.status(HttpStatus.CREATED).body(
                    new ArchivoSubidoResponse(nombreBlob, urlBlob, "Archivo subido exitosamente")
            );
        } catch (IOException e) {
            return ResponseEntity.internalServerError().build();
        }
    }

    /**
     * GET /api/archivos/url-descarga?blob=contratos/uuid.pdf
     * Genera URL temporal con SAS Token para descarga directa desde Azure Storage.
     */
    @GetMapping("/url-descarga")
    public ResponseEntity<ArchivoUrlResponse> obtenerUrlDescarga(
            @RequestParam("blob") String nombreBlob) {

        if (!blobStorageService.existeArchivo(nombreBlob)) {
            return ResponseEntity.notFound().build();
        }

        String urlTemporal = blobStorageService.generarUrlTemporalDescarga(nombreBlob);
        return ResponseEntity.ok(new ArchivoUrlResponse(
                urlTemporal, sasExpiryMinutes, "URL válida por " + sasExpiryMinutes + " minutos"
        ));
    }

    /** GET /api/archivos/listar?carpeta=contratos */
    @GetMapping("/listar")
    public ResponseEntity<List<String>> listarArchivos(
            @RequestParam(value = "carpeta", defaultValue = "") String carpeta) {
        return ResponseEntity.ok(blobStorageService.listarArchivos(carpeta));
    }

    /** DELETE /api/archivos/eliminar?blob=contratos/uuid.pdf */
    @DeleteMapping("/eliminar")
    public ResponseEntity<Map<String, String>> eliminarArchivo(
            @RequestParam("blob") String nombreBlob) {

        boolean eliminado = blobStorageService.eliminarArchivo(nombreBlob);
        if (eliminado) {
            return ResponseEntity.ok(Map.of("mensaje", "Archivo eliminado correctamente"));
        }
        return ResponseEntity.notFound().build();
    }
}
```

### Respuestas JSON de ejemplo

```bash
// POST /api/archivos/subir?carpeta=contratos
{
  "nombreBlob": "contratos/a3f9b2c1-4d5e-6f7a.pdf",
  "urlPublica": "https://miempresa.blob.core.windows.net/documentos/contratos/a3f9b2c1-4d5e-6f7a.pdf",
  "mensaje": "Archivo subido exitosamente"
}

// GET /api/archivos/url-descarga?blob=contratos/a3f9b2c1-4d5e-6f7a.pdf
{
  "urlDescarga": "https://miempresa.blob.core.windows.net/documentos/contratos/a3f9b2c1.pdf?sv=2023-01-03&se=2025-03-15T11%3A00%3A00Z&sr=b&sp=r&sig=ABC123...",
  "expiraEnMinutos": 60,
  "mensaje": "URL válida por 60 minutos"
}
```

---

## 🏢 9. Casos de Uso Reales en el Mundo Laboral

### 🏦 Sector Bancario

```
Usuario sube voucher de depósito (JPG)
           │
           ▼
[Spring Boot — App Service]
           │
           ├──► Azure Blob Storage   → blob: "vouchers/2025/03/uuid.jpg"
           │    contenedor: "vouchers"
           │
           └──► Azure SQL            → tabla: transacciones
                                       campo: voucher_blob = "vouchers/2025/03/uuid.jpg"
```

### 🛒 E-commerce

```
Admin sube foto de producto (PNG)
           │
           ▼
[Spring Boot]
           │
           ├──► Blob Storage (público) → URL accesible directamente desde el navegador
           │
           └──► Cosmos DB             → documento con campo imagen_url: "https://..."
```

### 📋 RRHH / Legal

```
Candidato sube CV (PDF)
           │
           ▼
[Spring Boot]
           │
           ├──► Blob Storage (privado) → Solo accesible con SAS Token temporal
           │
           └──► Azure SQL             → tabla: candidatos
                                        campo: cv_blob = "cvs/uuid.pdf"
                                        (no la URL — más seguro y flexible)
```

---

## 🎥 10. Recursos y Videos Recomendados

| Recurso                                | Descripción                                        | Dónde encontrarlo                                                                     |
|----------------------------------------|----------------------------------------------------|---------------------------------------------------------------------------------------|
| **Microsoft Learn — Azure Storage**    | Ruta oficial con módulos interactivos gratuitos    | [learn.microsoft.com/azure/storage](https://learn.microsoft.com/es-es/azure/storage/) |
| **"Azure Blob Storage Java Tutorial"** | Integración completa con Java y Spring Boot        | Buscar en YouTube: `"Azure Blob Storage Java Spring Boot tutorial"`                   |
| **"Azure Storage for Beginners"**      | Visión general de los 4 tipos de storage           | Buscar en YouTube: `"Azure Storage explained beginners Microsoft"`                    |
| **Azure Storage Explorer**             | Herramienta visual gratuita para explorar blobs    | Buscar: `"Azure Storage Explorer download"`                                           |
| **Azurite**                            | Emulador local de Azure Storage (sin cuenta Azure) | `npm install -g azurite`                                                              |

> 🎯 **Dos herramientas gratuitas que puedes usar ahora mismo:**
> - **Azurite:** Emulador local de Azure Storage. Instala con `npm install -g azurite` y tendrás Blob, Queue y Table
    Storage en tu máquina sin costo.
> - **Azure Storage Explorer:** Interfaz visual para conectarte a Azurite y explorar blobs localmente. Es como el SSMS
    pero para archivos en la nube.

### 📖 Documentación oficial clave

| Documento                          | URL                                                                                                                                                          |
|------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Introducción a Azure Blob Storage  | [learn.microsoft.com/azure/storage/blobs/storage-blobs-introduction](https://learn.microsoft.com/es-es/azure/storage/blobs/storage-blobs-introduction)       |
| Quickstart SDK Java — Blob Storage | [learn.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-java](https://learn.microsoft.com/es-es/azure/storage/blobs/storage-quickstart-blobs-java) |
| SAS Tokens explicados              | [learn.microsoft.com/azure/storage/common/storage-sas-overview](https://learn.microsoft.com/es-es/azure/storage/common/storage-sas-overview)                 |
| Azurite — Emulador local           | [learn.microsoft.com/azure/storage/common/storage-use-azurite](https://learn.microsoft.com/es-es/azure/storage/common/storage-use-azurite)                   |

---

## 🧠 Resumen Ejecutivo

> Lo que deberías poder decir en una entrevista si te preguntan *"¿Qué es Azure Storage y cómo lo usarías con Spring
Boot?"*

**Azure Storage** es el servicio de almacenamiento de objetos de Azure, diseñado para guardar archivos, imágenes,
videos, backups y cualquier dato no estructurado a escala masiva, con 99.999999999% de durabilidad garantizada. Ofrece
cuatro tipos: **Blob Storage** (el más importante — para archivos en general), **Azure Files** (carpetas compartidas
tipo red empresarial), **Queue Storage** (mensajes simples entre servicios) y **Table Storage** (clave-valor legacy).
Los blobs se organizan en **contenedores** dentro de una cuenta de storage, y cada blob tiene una URL única predecible.
El costo se optimiza con **Access Tiers**: Hot para acceso frecuente, Cool para acceso mensual, Cold para acceso raro y
Archive para retención regulatoria a largo plazo (obligatorio en sectores como banca en Perú). La seguridad se gestiona
con **Access Keys** (siempre en Application Settings, nunca en el código), **SAS Tokens** para generar URLs temporales y
limitadas de acceso a archivos privados, y **Managed Identity** como forma más segura en producción. Desde Spring Boot,
la integración usa el SDK oficial `azure-storage-blob` con un `BlobServiceClient` singleton. El patrón correcto es
guardar en la base de datos solo el **nombre o URL del blob**, nunca el archivo binario. En producción los archivos
confidenciales van en contenedores privados y se exponen mediante SAS Tokens que expiran — el archivo va directo de
Azure Storage al cliente sin pasar por el servidor backend.

---

## ⏭️ Siguiente módulo

> ⚡ **Módulo 05 — Azure Functions:** El servicio serverless de Azure. Veremos qué es la arquitectura serverless, los
> tipos de triggers (HTTP, Timer, Blob, Service Bus), cómo se diferencia de App Service, y cómo crear funciones con Java
> que reaccionen a eventos.

---

*📝 Documentación elaborada como parte del plan de aprendizaje Azure para desarrolladores Java Backend.*  
*🗓️ Última actualización: 2025*
