# 🗃️ Módulo 02 — Azure SQL

> 📚 **Serie:** Azure para Java Developers  
> 🗂️ **Módulo:** 02 de 08  
> 🎯 **Objetivo:** Entender qué es Azure SQL, cómo se diferencia de un SQL Server local, sus niveles de servicio, y cómo
> conectarte desde Spring Boot usando Spring Data JPA.

---

## 📋 Tabla de Contenidos

1. [¿Qué es Azure SQL?](#-1-qué-es-azure-sql)
2. [¿Cómo está organizado Azure SQL?](#-2-cómo-está-organizado-azure-sql)
3. [Modelos de compra y niveles de servicio](#-3-modelos-de-compra-y-niveles-de-servicio)
4. [Conceptos clave](#-4-conceptos-clave)
5. [Azure SQL vs SQL Server local](#-5-azure-sql-vs-sql-server-local)
6. [Integración con Spring Boot](#-6-integración-con-spring-boot)
7. [Ejemplo de código](#-7-ejemplo-de-código)
8. [Seguridad y buenas prácticas](#-8-seguridad-y-buenas-prácticas)
9. [Recursos y videos](#-9-recursos-y-videos-recomendados)
10. [Resumen ejecutivo](#-resumen-ejecutivo)

---

## 🗃️ 1. ¿Qué es Azure SQL?

**Azure SQL** es la base de datos relacional gestionada de Microsoft Azure. Está basada en el motor de **Microsoft SQL
Server**, pero en lugar de instalarlo y administrarlo tú en un servidor, Azure lo gestiona completamente: parches,
backups, alta disponibilidad, escalado y seguridad.

### La versión larga (con contexto real)

Imagina que tu aplicación Spring Boot necesita una base de datos relacional. En un entorno tradicional, harías esto:

1. 🖥️ Instalas SQL Server en un servidor (físico o virtual).
2. ⚙️ Configuras el motor: memoria, conexiones máximas, collation, etc.
3. 🔐 Configuras usuarios, permisos y políticas de contraseñas.
4. 💾 Configuras backups automáticos y defines la política de retención.
5. 🔄 Configuras replicación si necesitas alta disponibilidad.
6. 🛡️ Aplicas parches de seguridad cada vez que Microsoft los libera.
7. 📊 Monitoreas el rendimiento y ajustas índices manualmente.
8. 📈 Si necesitas más capacidad, migas a un servidor más grande.

**Todo eso es trabajo de un DBA (Database Administrator).** En muchas empresas, ese rol es una persona entera dedicada
solo a eso.

**Azure SQL elimina casi toda esa carga.** Tú creas la base de datos en minutos desde el portal, defines cuánta potencia
necesitas, y Azure se encarga de todo lo demás. Tú solo te preocupas de tu esquema, tus queries y tu código.

**¿Qué gestiona Azure automáticamente?**

| Tarea                       | Sin Azure SQL          | Con Azure SQL                       |
|-----------------------------|------------------------|-------------------------------------|
| Instalación y configuración | 👨‍💻 Tú               | ✅ Azure                             |
| Parches y actualizaciones   | 👨‍💻 Tú               | ✅ Azure                             |
| Backups automáticos         | 👨‍💻 Tú               | ✅ Azure (hasta 35 días)             |
| Alta disponibilidad         | 👨‍💻 Tú (replicación) | ✅ Azure (99.99% SLA)                |
| Escalado de recursos        | 👨‍💻 Tú (migración)   | ✅ Azure (en minutos)                |
| Cifrado en reposo           | 👨‍💻 Tú               | ✅ Azure (por defecto)               |
| Monitoreo de rendimiento    | 👨‍💻 Tú               | ✅ Azure (Query Performance Insight) |

> 💡 **En resumen:** Azure SQL es SQL Server en la nube, completamente gestionado por Azure. Compatible al 100% con el
> SQL y los drivers de SQL Server que ya conoces, pero sin la carga operativa de administrar el motor de base de datos.

---

## 🏗️ 2. ¿Cómo está organizado Azure SQL?

Azure SQL no es un solo producto, sino una **familia de servicios** relacionados. Es importante entender las
diferencias:

```
┌─────────────────────── Familia Azure SQL ─────────────────────────┐
│                                                                   │
│  ┌─────────────────┐  ┌──────────────────┐  ┌────────────────┐    │
│  │  Azure SQL DB   │  │  Azure SQL       │  │  Azure SQL     │    │
│  │  (Database)     │  │  Managed Instance│  │  on VM         │    │
│  │                 │  │                  │  │                │    │
│  │  El más         │  │  Migración de    │  │  SQL Server    │    │
│  │  común para     │  │  SQL Server      │  │  completo en   │    │
│  │  apps nuevas    │  │  on-premise      │  │  una VM Azure  │    │
│  └─────────────────┘  └──────────────────┘  └────────────────┘    │
│                                                                   │
│       PaaS (más gestionado) ◄────────────────► IaaS (más control) │
└───────────────────────────────────────────────────────────────────┘
```

| Opción                      | ¿Qué es?                                        | ¿Cuándo usarlo?                                                    |
|-----------------------------|-------------------------------------------------|--------------------------------------------------------------------|
| ⭐ **Azure SQL Database**    | Base de datos individual, totalmente gestionada | Apps nuevas, microservicios, la opción más común                   |
| 🏢 **SQL Managed Instance** | Instancia SQL Server completa pero gestionada   | Migración de apps legacy que usan features avanzados de SQL Server |
| 🖥️ **SQL on VM**           | SQL Server instalado en una VM Azure            | Control total, cuando necesitas configuraciones muy específicas    |

> 📌 **En el contexto de ofertas laborales y desarrollo backend con Spring Boot, cuando dicen "Azure SQL" casi siempre se
refieren a Azure SQL Database.** Es el que estudiaremos a fondo.

---

## 💰 3. Modelos de Compra y Niveles de Servicio

Azure SQL Database tiene dos modelos de compra principales. Entender esto es clave para dimensionar correctamente una
base de datos en producción.

### Modelo DTU (Database Transaction Units)

Es el modelo más simple. DTU es una medida combinada de CPU, memoria e I/O. No necesitas entender los detalles técnicos
de cada recurso, solo eliges un nivel y Azure te da un paquete balanceado.

| Tier            | DTUs     | Almacenamiento | Uso recomendado             |
|-----------------|----------|----------------|-----------------------------|
| **Basic**       | 5 DTUs   | 2 GB           | Aprendizaje, apps pequeñas  |
| **Standard S0** | 10 DTUs  | 250 GB         | Apps de desarrollo          |
| **Standard S2** | 50 DTUs  | 250 GB         | Apps de producción pequeñas |
| **Standard S4** | 200 DTUs | 250 GB         | Apps de producción medianas |
| **Premium P1**  | 125 DTUs | 500 GB         | Apps críticas, alto tráfico |
| **Premium P4**  | 500 DTUs | 500 GB         | Apps de muy alto tráfico    |

### Modelo vCore (Virtual Cores) — el más usado en producción

Te da control granular sobre CPU y memoria por separado. Es más transparente y flexible.

| Tier                  | vCores | RAM       | Uso recomendado                                                    |
|-----------------------|--------|-----------|--------------------------------------------------------------------|
| **General Purpose**   | 2–80   | 10–408 GB | La mayoría de workloads de producción                              |
| **Business Critical** | 2–80   | 10–408 GB | Apps que requieren alta disponibilidad local y réplicas de lectura |
| **Hyperscale**        | 2–80   | 10–408 GB | Bases de datos de más de 4 TB, escalado masivo                     |

> 💡 **¿Cuál se usa en Perú en el sector financiero?**  
> Lo más común es **General Purpose con 4–8 vCores** para APIs backend de transacciones medias. **Business Critical**
> para sistemas core bancarios donde el tiempo de failover debe ser mínimo.

---

## 🔑 4. Conceptos Clave

### 📌 Servidor Lógico (Logical Server)

En Azure SQL, antes de crear una base de datos necesitas crear un **servidor lógico**. Este servidor es un concepto
administrativo — no es un servidor físico real — que actúa como contenedor de una o varias bases de datos y define
configuraciones compartidas como:

- El nombre del host de conexión: `mi-servidor.database.windows.net`
- Las reglas de firewall
- El administrador del servidor
- La política de autenticación

```
┌─────────────────────────────────────────┐
│         Servidor Lógico                 │
│    mi-servidor.database.windows.net     │
│                                         │
│  ┌──────────┐  ┌──────────┐  ┌───────┐  │
│  │  DB:     │  │  DB:     │  │  DB:  │  │
│  │ usuarios │  │ pedidos  │  │  logs │  │
│  └──────────┘  └──────────┘  └───────┘  │
│                                         │
│  [Firewall Rules]  [Admin]  [Auth]      │
└─────────────────────────────────────────┘
```

### 📌 Firewall Rules (Reglas de Firewall)

Por defecto, Azure SQL **bloquea todas las conexiones externas**. Para que tu app Spring Boot (corriendo en App Service)
pueda conectarse, debes agregar una regla de firewall.

Existen dos tipos:

- **Regla de IP específica:** Permite conexiones desde una IP concreta. Útil para tu PC de desarrollo.
- **"Allow Azure Services":** Permite conexiones desde cualquier servicio dentro de Azure (como tu App Service). Esta
  opción se activa con un toggle en el portal.

> ⚠️ **Nunca actives "Allow all IPs" (0.0.0.0 - 255.255.255.255) en producción.** Es un riesgo de seguridad grave.

### 📌 Connection String

La cadena de conexión para SQL Server desde Java tiene este formato:

```
jdbc:sqlserver://[servidor].database.windows.net:1433;
    database=[nombre-db];
    encrypt=true;
    trustServerCertificate=false;
    loginTimeout=30;
```

El puerto **1433** es el estándar de SQL Server. El parámetro `encrypt=true` es obligatorio en Azure SQL — todas las
conexiones deben ir cifradas.

### 📌 Geo-Replication (Replicación Geográfica)

Azure SQL permite crear réplicas de lectura en otras regiones del mundo. Si tu base de datos principal está en
`Brazil South`, puedes tener una réplica en `East US`. Esto sirve para:

- **Disaster Recovery:** Si la región principal falla, el failover es automático.
- **Lectura local:** Aplicaciones en distintas regiones leen desde la réplica más cercana, reduciendo latencia.

### 📌 Elastic Pool

Cuando tienes múltiples bases de datos con picos de uso en diferentes momentos, el **Elastic Pool** te permite que
compartan un pool de recursos (DTUs o vCores). Es más eficiente y económico que asignar recursos fijos a cada base de
datos individualmente.

```
Sin Elastic Pool:           Con Elastic Pool:
DB1: 50 DTUs (usa 10)       ┌─────────────────────┐
DB2: 50 DTUs (usa 5)        │   Pool: 100 DTUs    │
DB3: 50 DTUs (usa 40)       │  ┌───┐ ┌───┐ ┌───┐  │
Total: 150 DTUs pagadas     │  │DB1│ │DB2│ │DB3│  │
                            │  └───┘ └───┘ └───┘  │
                            └─────────────────────┘
                            Total: 100 DTUs pagadas
                            (se comparten según demanda)
```

---

## ⚖️ 5. Azure SQL vs SQL Server Local

| Característica           | 🗃️ Azure SQL Database                 | 🖥️ SQL Server Local                      |
|--------------------------|----------------------------------------|-------------------------------------------|
| **Instalación**          | Cero — listo en minutos                | Horas/días de configuración               |
| **Backups**              | Automáticos (hasta 35 días retención)  | Manual o scripts propios                  |
| **Alta disponibilidad**  | 99.99% SLA incluido                    | Debes configurar Always On / Clustering   |
| **Escalado**             | En minutos desde el portal             | Requiere migración a hardware mayor       |
| **Parches**              | Automáticos por Azure                  | Responsabilidad del DBA                   |
| **Costo**                | Por uso (pagar lo que consumes)        | Licencia + hardware + operación           |
| **Compatibilidad T-SQL** | ~95% compatible con SQL Server         | 100%                                      |
| **Features avanzados**   | Algunos no disponibles (ej. SQL Agent) | Todos disponibles                         |
| **Acceso a nivel SO**    | ❌ No                                   | ✅ Sí                                      |
| **Ideal para**           | Apps nuevas en la nube                 | Migración, requerimientos muy específicos |

> 📌 **La compatibilidad del 95% en T-SQL es importante.** Features como SQL Server Agent Jobs, algunas CLR integrations
> o cross-database queries no están disponibles en Azure SQL Database. Para esos casos existe **SQL Managed Instance**
> que
> sí los soporta.

---

## ☕ 6. Integración con Spring Boot

### Dependencias en `pom.xml`

```xml

<dependencies>
    <!-- Spring Data JPA — ORM y abstracción de base de datos -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Driver JDBC de Microsoft para SQL Server / Azure SQL -->
    <!-- Este es el driver oficial — NO usar el de jTDS que es legacy -->
    <dependency>
        <groupId>com.microsoft.sqlserver</groupId>
        <artifactId>mssql-jdbc</artifactId>
        <scope>runtime</scope>
        <!-- Spring Boot gestiona la versión automáticamente -->
    </dependency>

    <!-- Azure Identity — para autenticación con Managed Identity (sin contraseña) -->
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-identity</artifactId>
        <version>1.11.0</version>
    </dependency>
</dependencies>
```

### Configuración en `application.properties`

```properties
# ============================================================
# Configuración de Azure SQL con Spring Data JPA
# Los valores reales vienen de Azure Application Settings
# ============================================================
# Cadena de conexión a Azure SQL
# Formato: jdbc:sqlserver://[servidor].database.windows.net:1433;database=[nombre-db];encrypt=true
spring.datasource.url=${AZURE_SQL_URL}
spring.datasource.username=${AZURE_SQL_USERNAME}
spring.datasource.password=${AZURE_SQL_PASSWORD}
# Driver JDBC de Microsoft SQL Server
spring.datasource.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver
# Dialecto de Hibernate para SQL Server
spring.jpa.database-platform=org.hibernate.dialect.SQLServerDialect
# Estrategia de creación del esquema:
# validate  → verifica que el esquema exista (recomendado en producción)
# update    → actualiza el esquema si hay cambios (solo desarrollo)
# create    → crea el esquema desde cero (solo pruebas — BORRA DATOS)
# none      → no hace nada (para cuando usas Flyway o Liquibase)
spring.jpa.hibernate.ddl-auto=validate
# Mostrar SQL generado por Hibernate (solo para desarrollo, false en producción)
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true
# Pool de conexiones — HikariCP (viene incluido con Spring Boot)
# Azure SQL tiene un límite de conexiones según el tier — configurar correctamente
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
```

> 📌 **¿Por qué configurar HikariCP?** Azure SQL tiene un límite de conexiones simultáneas según el tier contratado. Si
> tu app abre más conexiones de las permitidas, Azure SQL las rechaza. HikariCP es el pool de conexiones que Spring Boot
> usa por defecto y gestiona esto automáticamente si lo configuras bien.

---

## 💻 7. Ejemplo de Código

### Escenario

Una API REST que gestiona productos, conectada a Azure SQL mediante Spring Data JPA. El ejemplo muestra la estructura
completa: entidad, repositorio, servicio y controlador.

### Estructura del proyecto

```
mi-app-azure/
└── src/main/java/com/ejemplo/azure/
    ├── MiAppAzureApplication.java
    ├── entity/
    │   └── Producto.java
    ├── repository/
    │   └── ProductoRepository.java
    ├── service/
    │   └── ProductoService.java
    └── controller/
        └── ProductoController.java
```

### `Producto.java` — Entidad JPA

```java
package com.ejemplo.azure.entity;

import jakarta.persistence.*;

import java.math.BigDecimal;
import java.time.LocalDateTime;

/**
 * Entidad mapeada a la tabla "productos" en Azure SQL.
 * Hibernate generará o validará esta tabla según ddl-auto.
 */
@Entity
@Table(name = "productos")
public class Producto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "nombre", nullable = false, length = 100)
    private String nombre;

    @Column(name = "descripcion", length = 500)
    private String descripcion;

    @Column(name = "precio", nullable = false, precision = 10, scale = 2)
    private BigDecimal precio;

    @Column(name = "stock", nullable = false)
    private Integer stock;

    @Column(name = "activo", nullable = false)
    private Boolean activo = true;

    // Auditoría — cuándo fue creado y actualizado el registro
    @Column(name = "fecha_creacion", updatable = false)
    private LocalDateTime fechaCreacion;

    @Column(name = "fecha_actualizacion")
    private LocalDateTime fechaActualizacion;

    @PrePersist
    protected void onCreate() {
        fechaCreacion = LocalDateTime.now();
        fechaActualizacion = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        fechaActualizacion = LocalDateTime.now();
    }

    // Getters y Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public String getDescripcion() {
        return descripcion;
    }

    public void setDescripcion(String descripcion) {
        this.descripcion = descripcion;
    }

    public BigDecimal getPrecio() {
        return precio;
    }

    public void setPrecio(BigDecimal precio) {
        this.precio = precio;
    }

    public Integer getStock() {
        return stock;
    }

    public void setStock(Integer stock) {
        this.stock = stock;
    }

    public Boolean getActivo() {
        return activo;
    }

    public void setActivo(Boolean activo) {
        this.activo = activo;
    }

    public LocalDateTime getFechaCreacion() {
        return fechaCreacion;
    }

    public LocalDateTime getFechaActualizacion() {
        return fechaActualizacion;
    }
}
```

### `ProductoRepository.java` — Repositorio Spring Data JPA

```java
package com.ejemplo.azure.repository;

import com.ejemplo.azure.entity.Producto;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.math.BigDecimal;
import java.util.List;

/**
 * Spring Data JPA genera automáticamente las queries SQL
 * a partir del nombre de los métodos.
 * No necesitas escribir SQL para operaciones básicas.
 */
@Repository
public interface ProductoRepository extends JpaRepository<Producto, Long> {

    // Spring Data genera: SELECT * FROM productos WHERE activo = 1
    List<Producto> findByActivoTrue();

    // Spring Data genera: SELECT * FROM productos WHERE nombre LIKE '%?%'
    List<Producto> findByNombreContainingIgnoreCase(String nombre);

    // Spring Data genera: SELECT * FROM productos WHERE precio BETWEEN ? AND ?
    List<Producto> findByPrecioBetween(BigDecimal precioMin, BigDecimal precioMax);

    // Query JPQL personalizada (lenguaje de JPA, independiente del motor SQL)
    @Query("SELECT p FROM Producto p WHERE p.stock < :stockMinimo AND p.activo = true")
    List<Producto> findProductosBajoStock(@Param("stockMinimo") Integer stockMinimo);

    // Query nativa SQL (específica para SQL Server / Azure SQL)
    // Usar solo cuando JPQL no alcanza — ata el código al motor de BD
    @Query(value = "SELECT TOP 10 * FROM productos ORDER BY fecha_creacion DESC",
            nativeQuery = true)
    List<Producto> findUltimos10Productos();
}
```

### `ProductoService.java` — Capa de servicio

```java
package com.ejemplo.azure.service;

import com.ejemplo.azure.entity.Producto;
import com.ejemplo.azure.repository.ProductoRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
public class ProductoService {

    @Autowired
    private ProductoRepository productoRepository;

    public List<Producto> listarActivos() {
        return productoRepository.findByActivoTrue();
    }

    public Optional<Producto> buscarPorId(Long id) {
        return productoRepository.findById(id);
    }

    /**
     * @Transactional garantiza que si algo falla durante el guardado,
     * Azure SQL hace rollback automático de toda la operación.
     */
    @Transactional
    public Producto guardar(Producto producto) {
        return productoRepository.save(producto);
    }

    /**
     * Baja lógica — no borramos el registro, solo lo marcamos como inactivo.
     * Patrón muy común en sistemas empresariales y bancarios en Perú.
     */
    @Transactional
    public void desactivar(Long id) {
        productoRepository.findById(id).ifPresent(p -> {
            p.setActivo(false);
            productoRepository.save(p);
        });
    }

    public List<Producto> buscarPorNombre(String nombre) {
        return productoRepository.findByNombreContainingIgnoreCase(nombre);
    }

    public List<Producto> productosBajoStock(Integer stockMinimo) {
        return productoRepository.findProductosBajoStock(stockMinimo);
    }
}
```

### `ProductoController.java` — API REST

```java
package com.ejemplo.azure.controller;

import com.ejemplo.azure.entity.Producto;
import com.ejemplo.azure.service.ProductoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/productos")
public class ProductoController {

    @Autowired
    private ProductoService productoService;

    // GET /api/productos
    @GetMapping
    public ResponseEntity<List<Producto>> listar() {
        return ResponseEntity.ok(productoService.listarActivos());
    }

    // GET /api/productos/1
    @GetMapping("/{id}")
    public ResponseEntity<Producto> buscarPorId(@PathVariable Long id) {
        return productoService.buscarPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // GET /api/productos/buscar?nombre=laptop
    @GetMapping("/buscar")
    public ResponseEntity<List<Producto>> buscarPorNombre(@RequestParam String nombre) {
        return ResponseEntity.ok(productoService.buscarPorNombre(nombre));
    }

    // POST /api/productos
    @PostMapping
    public ResponseEntity<Producto> crear(@RequestBody Producto producto) {
        Producto guardado = productoService.guardar(producto);
        return ResponseEntity.status(HttpStatus.CREATED).body(guardado);
    }

    // PUT /api/productos/1
    @PutMapping("/{id}")
    public ResponseEntity<Producto> actualizar(@PathVariable Long id,
                                               @RequestBody Producto producto) {
        return productoService.buscarPorId(id)
                .map(existente -> {
                    existente.setNombre(producto.getNombre());
                    existente.setDescripcion(producto.getDescripcion());
                    existente.setPrecio(producto.getPrecio());
                    existente.setStock(producto.getStock());
                    return ResponseEntity.ok(productoService.guardar(existente));
                })
                .orElse(ResponseEntity.notFound().build());
    }

    // DELETE /api/productos/1 — baja lógica, no física
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> desactivar(@PathVariable Long id) {
        productoService.desactivar(id);
        return ResponseEntity.noContent().build();
    }
}
```

### SQL de creación de tabla (para referencia)

```sql
-- Crear la tabla en Azure SQL (ejecutar en el Query Editor del portal de Azure)
CREATE TABLE productos
(
    id                  BIGINT IDENTITY(1,1) PRIMARY KEY,
    nombre              NVARCHAR(100)  NOT NULL,
    descripcion         NVARCHAR(500)  NULL,
    precio              DECIMAL(10, 2) NOT NULL,
    stock               INT            NOT NULL DEFAULT 0,
    activo              BIT            NOT NULL DEFAULT 1,
    fecha_creacion      DATETIME2      NOT NULL DEFAULT GETDATE(),
    fecha_actualizacion DATETIME2      NOT NULL DEFAULT GETDATE()
);

-- Índice para búsquedas por nombre (mejora el rendimiento de findByNombre...)
CREATE INDEX IX_productos_nombre ON productos (nombre);

-- Índice para filtrar productos activos (query más frecuente de la app)
CREATE INDEX IX_productos_activo ON productos (activo);
```

---

## 🔐 8. Seguridad y Buenas Prácticas

La seguridad en Azure SQL es un tema que aparece siempre en entrevistas de nivel medio-senior. Estos son los conceptos
clave:

### 🛡️ Autenticación: SQL vs Azure Active Directory

Azure SQL soporta dos modos de autenticación:

**SQL Authentication (usuario/contraseña) — Simple pero menos seguro:**

```properties
# application.properties — los valores vienen de Application Settings en Azure
spring.datasource.username=${AZURE_SQL_USERNAME}
spring.datasource.password=${AZURE_SQL_PASSWORD}
```

**Azure Active Directory con Managed Identity — La forma recomendada en producción:**

```java
// Con Managed Identity no hay contraseña — Azure gestiona la autenticación.
// La app en App Service tiene una identidad que Azure SQL reconoce directamente.
// Solo necesitas la URL de conexión, sin usuario ni contraseña explícitos.
@Bean
public DataSource dataSource() {
    HikariConfig config = new HikariConfig();
    // La autenticación AAD se especifica en la connection string
    config.setJdbcUrl(
            System.getenv("AZURE_SQL_URL") +
            ";authentication=ActiveDirectoryMSI"
    );
    config.setMaximumPoolSize(10);
    return new HikariDataSource(config);
}
```

### 🔒 Transparent Data Encryption (TDE)

Azure SQL cifra automáticamente **todos los datos en reposo** (archivos de base de datos, backups y logs de
transacciones) usando AES-256. Está habilitado por defecto y no requiere ninguna configuración de tu parte.

### 🌐 Private Endpoint

En producción bancaria/enterprise, Azure SQL no debería ser accesible desde internet público. Se configura un **Private
Endpoint** para que solo sea accesible desde dentro de la red virtual (VNet) de Azure donde vive tu App Service.

```
Internet ──✗──► [Azure SQL]    ← No accesible desde internet
                    ▲
                    │ Solo acceso interno por VNet
                    │
[App Service en VNet] ──────► [Azure SQL con Private Endpoint]
```

### 📋 Auditoría

Azure SQL puede registrar **todas las queries** ejecutadas en la base de datos hacia un Azure Storage Account. Esto es
obligatorio en entornos bancarios y regulados en Perú (SBS).

---

## 🎥 9. Recursos y Videos Recomendados

| Recurso                                              | Descripción                                                    | Dónde encontrarlo                                                                                                                                                          |
|------------------------------------------------------|----------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Microsoft Learn — Azure SQL**                      | Ruta oficial con ejercicios interactivos                       | [learn.microsoft.com/azure/azure-sql](https://learn.microsoft.com/es-es/azure/azure-sql/)                                                                                  |
| **"Azure SQL for Beginners"**                        | Serie de 60 videos cortos del equipo de Azure SQL en Microsoft | Buscar en YouTube: `"Azure SQL for beginners Microsoft"`                                                                                                                   |
| **"Spring Boot + Azure SQL Tutorial"**               | Integración completa paso a paso                               | Buscar en YouTube: `"Spring Boot Azure SQL Database tutorial"`                                                                                                             |
| **Quickstart oficial — Spring Data JPA + Azure SQL** | Guía oficial de Microsoft para Java                            | [learn.microsoft.com → Spring Data JPA Azure SQL](https://learn.microsoft.com/es-es/azure/developer/java/spring-framework/configure-spring-data-jpa-with-azure-sql-server) |

---

## 🧠 Resumen Ejecutivo

> Lo que deberías poder decir en una entrevista si te preguntan *"¿Qué es Azure SQL y cómo lo integrarías con Spring
Boot?"*

**Azure SQL** es la base de datos relacional gestionada de Azure, basada en el motor de SQL Server. Elimina la carga
operativa de administrar el motor: Azure gestiona automáticamente los backups con hasta 35 días de retención, alta
disponibilidad con SLA del 99.99%, parches de seguridad y escalado de recursos en minutos. Se organiza en un **servidor
lógico** que actúa como contenedor administrativo de una o más bases de datos y centraliza las reglas de firewall y la
autenticación. Para dimensionar los recursos se puede usar el modelo **DTU** (paquetes predefinidos, más simple) o
**vCore** (control granular de CPU y RAM, más usado en producción). Desde Spring Boot, la integración se hace con
**Spring Data JPA** y el driver JDBC oficial de Microsoft (`mssql-jdbc`), configurando la cadena de conexión con
`encrypt=true` (obligatorio en Azure SQL) y gestionando el pool de conexiones con **HikariCP** para no exceder el límite
de conexiones del tier contratado. La forma más segura de autenticación es mediante **Managed Identity** con Azure
Active Directory, que elimina la necesidad de manejar contraseñas. En producción, Azure SQL se protege con **Private
Endpoints** para que no sea accesible desde internet público, y se habilita **auditoría de queries** para cumplimiento
regulatorio.

---

## ⏭️ Siguiente módulo

> 🌌 **Módulo 03 — Cosmos DB:** La base de datos NoSQL distribuida y multi-modelo de Azure. Veremos qué es, cuándo
> elegirlo sobre Azure SQL, sus APIs (SQL, MongoDB, Cassandra), y cómo conectarte desde Spring Boot.

---

*📝 Documentación elaborada como parte del plan de aprendizaje Azure para desarrolladores Java Backend.*  
*🗓️ Última actualización: 2025*
