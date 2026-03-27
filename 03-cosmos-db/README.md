# 🌌 Módulo 03 — Azure Cosmos DB

> 📚 **Serie:** Azure para Java Developers  
> 🗂️ **Módulo:** 03 de 08  
> 🎯 **Objetivo:** Entender qué es Cosmos DB, por qué existe, cuándo usarlo en lugar de Azure SQL, sus APIs, modelos de
> consistencia, y cómo integrarlo con Spring Boot.

---

## 📋 Tabla de Contenidos

1. [¿Qué es Cosmos DB?](#-1-qué-es-cosmos-db)
2. [¿Por qué NoSQL? El problema que resuelve](#-2-por-qué-nosql-el-problema-que-resuelve)
3. [Cómo está organizado Cosmos DB](#-3-cómo-está-organizado-cosmos-db)
4. [Las APIs de Cosmos DB](#-4-las-apis-de-cosmos-db)
5. [Modelos de consistencia](#-5-modelos-de-consistencia--el-concepto-más-importante)
6. [Partition Key — el concepto más crítico](#-6-partition-key--el-concepto-más-crítico)
7. [Throughput — RUs](#-7-throughput--rus-request-units)
8. [Cosmos DB vs Azure SQL](#-8-cosmos-db-vs-azure-sql--cuándo-usar-cada-uno)
9. [Integración con Spring Boot](#-9-integración-con-spring-boot)
10. [Ejemplo de código](#-10-ejemplo-de-código)
11. [Recursos y videos](#-11-recursos-y-videos-recomendados)
12. [Resumen ejecutivo](#-resumen-ejecutivo)

---

## 🌌 1. ¿Qué es Cosmos DB?

**Azure Cosmos DB** es la base de datos **NoSQL distribuida globalmente** de Microsoft Azure. Está diseñada para
aplicaciones que necesitan **baja latencia garantizada** (menos de 10 milisegundos), **escala masiva** y
**disponibilidad global** sin importar en qué parte del mundo estén los usuarios.

### La versión larga (con contexto real)

Para entender por qué existe Cosmos DB, necesitas entender primero el problema que resuelve. Imagina que construyes una
app de pagos digitales como Yape o Plin. En sus primeros días, Azure SQL funciona perfectamente. Pero a medida que
crece:

- 📱 Tienes **10 millones de usuarios** haciendo transacciones simultáneas.
- 🌎 Tus usuarios están en **Perú, Colombia, México y España**.
- ⚡ Cada transacción debe responder en **menos de 100ms** o el usuario abandona.
- 📈 En quincena o fin de mes, el tráfico se **multiplica por 10** de golpe.
- 🔄 Los datos de cada usuario son **diferentes entre sí** — algunos tienen más campos, otros menos.

Una base de datos relacional empieza a mostrar sus límites: escalar verticalmente tiene un techo, escalar
horizontalmente es complejo, replicar datos en múltiples regiones con consistencia es difícil, y un esquema rígido no
encaja con datos que varían entre registros.

**Cosmos DB nació para resolver exactamente esos problemas:**

- 🌍 **Distribución global nativa:** Con un click replicas tus datos en 60+ regiones. Los usuarios en México leen desde
  México, los de España desde España.
- ⚡ **Latencia garantizada por SLA:** Microsoft garantiza menos de 10ms en el percentil 99. No es un promedio — es una
  garantía contractual.
- 📈 **Escala elástica ilimitada:** Pasas de 1,000 a 1,000,000 operaciones por segundo ajustando un slider. Sin migrar,
  sin downtime.
- 🔓 **Esquema flexible:** Cada documento puede tener campos diferentes. No hay tablas ni columnas fijas.
- 🔌 **Multi-API:** Accedes a tus datos usando SQL, MongoDB, Cassandra y más sin cambiar el motor.

> 💡 **En resumen:** Cosmos DB es la base de datos de Azure para cuando Azure SQL ya no es suficiente en escala, latencia
> global o flexibilidad de esquema. Es la base de datos de las aplicaciones que operan a escala planetaria.

---

## 🤔 2. ¿Por qué NoSQL? El problema que resuelve

### El mundo relacional (SQL) — Estructura rígida

En una base de datos relacional, los datos viven en **tablas con columnas fijas**. Todos los registros tienen
exactamente los mismos campos.

```
Tabla: clientes
┌────┬───────────────┬──────────────────────┬──────────┐
│ id │ nombre        │ email                │ telefono │
├────┼───────────────┼──────────────────────┼──────────┤
│  1 │ Juan Pérez    │ juan@email.com       │ 987654   │
│  2 │ María López   │ maria@email.com      │ 923456   │
│  3 │ Carlos Ruiz   │ carlos@email.com     │ NULL     │
└────┴───────────────┴──────────────────────┴──────────┘
```

Si quieres agregar un campo nuevo (`redes_sociales`), el `ALTER TABLE` afecta **todos los registros** existentes. Con
100 millones de registros eso puede tardar horas y bloquear la tabla.

### El mundo NoSQL — Documentos flexibles

En Cosmos DB los datos se guardan como **documentos JSON**. Cada documento puede tener campos completamente diferentes.
No hay esquema fijo.

```json
// Documento 1 — Cliente básico
{
  "id": "cliente-001",
  "nombre": "Juan Pérez",
  "email": "juan@email.com"
}

// Documento 2 — Cliente con más información (misma colección, estructura diferente)
{
  "id": "cliente-002",
  "nombre": "María López",
  "email": "maria@email.com",
  "redesSociales": {
    "instagram": "@marialopez"
  },
  "historialCompras": [
    "pedido-123",
    "pedido-456"
  ]
}

// Documento 3 — Cliente empresa (estructura completamente diferente)
{
  "id": "cliente-003",
  "razonSocial": "Tech SAC",
  "ruc": "20123456789",
  "contactos": [
    {
      "nombre": "Ana García",
      "cargo": "Gerente"
    }
  ]
}
```

Los tres documentos coexisten en el mismo **contenedor** sin problema. Agregar un campo nuevo a futuros documentos no
requiere migración ni bloqueo.

### ¿Cuándo SQL y cuándo NoSQL?

| Criterio                    | 🗃️ Usa Azure SQL             | 🌌 Usa Cosmos DB                   |
|-----------------------------|-------------------------------|------------------------------------|
| **Estructura de datos**     | Bien definida y estable       | Variable o evoluciona rápido       |
| **Relaciones entre datos**  | Muchas (JOINs frecuentes)     | Pocas o datos auto-contenidos      |
| **Escala**                  | Miles a millones de registros | Millones a billones de registros   |
| **Distribución geográfica** | Una región                    | Múltiples regiones del mundo       |
| **Latencia requerida**      | Menos de 100ms es aceptable   | Menos de 10ms garantizado          |
| **Consistencia**            | ACID fuerte siempre           | Configurable según necesidad       |
| **Casos de uso típicos**    | ERP, contabilidad, RRHH       | Catálogos, IoT, gaming, e-commerce |

---

## 🏗️ 3. Cómo está organizado Cosmos DB

```
┌──────────────────────────────────────────────────────────────┐
│                    CUENTA DE COSMOS DB                       │
│            (mi-cuenta.documents.azure.com)                   │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                    BASE DE DATOS                       │  │
│  │                   "ecommerce-db"                       │  │
│  │                                                        │  │
│  │  ┌──────────────────┐    ┌──────────────────┐          │  │
│  │  │    CONTENEDOR    │    │    CONTENEDOR    │          │  │
│  │  │   "productos"    │    │    "pedidos"     │          │  │
│  │  │                  │    │                  │          │  │
│  │  │  { Documento 1 } │    │  { Documento 1 } │          │  │
│  │  │  { Documento 2 } │    │  { Documento 2 } │          │  │
│  │  │  { Documento 3 } │    │  { Documento 3 } │          │  │
│  │  │  PK=/categoria   │    │  PK=/clienteId   │          │  │
│  │  └──────────────────┘    └──────────────────┘          │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

| Nivel             | Equivalente en SQL | Descripción                                                       |
|-------------------|--------------------|-------------------------------------------------------------------|
| **Cuenta**        | Servidor lógico    | Punto de entrada único. Define la región y la URL de acceso.      |
| **Base de datos** | Base de datos      | Agrupa contenedores relacionados.                                 |
| **Contenedor**    | Tabla              | Colección de documentos. Define la Partition Key y el throughput. |
| **Documento**     | Fila / Registro    | Objeto JSON. Debe tener siempre un campo `id`.                    |

> 📌 Todo documento **debe tener un campo `id`**. Es el identificador único dentro de una partición. Si no lo incluyes,
> Cosmos DB genera un GUID automáticamente.

---

## 🔌 4. Las APIs de Cosmos DB

Cosmos DB soporta **múltiples APIs** — puedes usarlo como si fuera diferentes bases de datos NoSQL sin cambiar el motor.

```
                    ┌─────────────────────┐
                    │    COSMOS DB        │
                    │   (Motor único)     │
                    └──────────┬──────────┘
                               │
         ┌─────────────────────┼────────────────────┐
         │                     │                    │
   ┌─────▼──────┐      ┌───────▼─────┐      ┌───────▼──────┐
   │  API NoSQL │      │ API MongoDB │      │ API Cassandra│
   └────────────┘      └─────────────┘      └──────────────┘
         │                     │
   ┌─────▼──────┐      ┌───────▼─────┐
   │API Gremlin │      │  API Table  │
   │  (Grafos)  │      │             │
   └────────────┘      └─────────────┘
```

| API                       | ¿Qué es?                                      | ¿Cuándo usarla?                                   |
|---------------------------|-----------------------------------------------|---------------------------------------------------|
| ⭐ **API for NoSQL**       | API nativa. Queries SQL sobre documentos JSON | Apps nuevas — la más recomendada para Spring Boot |
| 🍃 **API for MongoDB**    | Compatible con protocolo MongoDB              | Si ya tienes código MongoDB y quieres migrar      |
| 🗄️ **API for Cassandra** | Compatible con Apache Cassandra               | Si vienes de Cassandra o necesitas columnas       |
| 🕸️ **API for Gremlin**   | Para bases de datos de grafos                 | Redes sociales, detección de fraude               |
| 📋 **API for Table**      | Compatible con Azure Table Storage            | Migración desde Table Storage                     |

> 📌 **Para Spring Boot, la API for NoSQL es la estándar.** Es la más optimizada y con mejor soporte del SDK de Java.

---

## ⚖️ 5. Modelos de Consistencia — El concepto más importante

Este es el concepto más diferenciador de Cosmos DB y el que más aparece en entrevistas. **Entenderlo bien te distingue
de otros candidatos.**

### El problema

Cuando tienes datos en múltiples regiones: si un usuario en Lima escribe un dato y otro en Madrid lo lee milisegundos
después — ¿Madrid ve el dato nuevo inmediatamente o puede ver el dato viejo? La respuesta depende del **modelo de
consistencia**.

```
MÁS CONSISTENTE                                          MÁS RÁPIDO
◄──────────────────────────────────────────────────────────────────►

  Strong → Bounded Staleness → Session → Consistent Prefix → Eventual
```

---

#### 1. 💪 Strong — "Todos ven lo que escribí inmediatamente"

**Garantía:** Toda lectura devuelve la versión más reciente del dato, sin excepción.

**Analogía:** Un pizarrón físico en una sala — cuando alguien escribe, todos lo ven al instante.

**Trade-off:** La más lenta — Azure sincroniza todas las réplicas antes de confirmar cada escritura.

**Uso:** Saldos bancarios, inventario crítico donde la inconsistencia nunca es aceptable.

```
Lima   → [ESCRIBE saldo = 1000]
               │  Azure sincroniza TODAS las réplicas
Madrid → [LEE saldo]  =  1000  ✅ Siempre correcto
```

---

#### 2. ⏱️ Bounded Staleness — "Lo que ves tiene máximo X segundos de retraso"

**Garantía:** Las lecturas pueden estar desfasadas, pero con un límite definido: máximo K versiones o T segundos.

**Analogía:** Un noticiero con 5 minutos de retraso. No es en vivo, pero sabes exactamente cuánto desfase tiene.

**Uso:** Dashboards de métricas, feeds de noticias, apps que toleran un pequeño retraso con límite conocido.

---

#### 3. 🔒 Session — "Yo siempre veo lo que yo mismo escribí" ⭐ El más usado

**Garantía:** Dentro de tu sesión, siempre lees tus propias escrituras. Otros usuarios pueden ver datos momentáneamente
desactualizados.

**Analogía:** Google Docs — tú ves tus cambios al instante, otros colaboradores pueden tardar un momento, pero eso es
completamente aceptable.

**Trade-off:** Excelente balance entre consistencia y rendimiento.

**Uso:** La mayoría de apps web y móviles. **Es el default de Cosmos DB y el más recomendado para Spring Boot.**

```
Usuario A → [ESCRIBE nombre = "Juan"]
Usuario A → [LEE nombre]  =  "Juan"   ✅ Ve su propio cambio
Usuario B → [LEE nombre]  =  "Pedro"  ⚠️ Puede ver dato anterior (eventual)
```

---

#### 4. 📋 Consistent Prefix — "El orden siempre es correcto, aunque vea versiones antiguas"

**Garantía:** Nunca verás actualizaciones fuera de orden. Si se escribió A → B → C, nunca verás C sin B primero.

**Analogía:** Leer un libro — puedes estar en el capítulo 5 mientras otros van en el 10, pero nunca leerás el 10 antes
que el 5.

**Uso:** Logs de auditoría, timelines de actividad donde el orden importa pero el retraso es tolerable.

---

#### 5. 🌊 Eventual — "En algún momento todos verán lo mismo, pero no sé cuándo"

**Garantía:** Eventualmente todos los nodos tendrán el mismo dato. Sin garantía de cuándo ni de orden en las lecturas
intermedias.

**Analogía:** El chisme en una oficina — la información se propaga, pero el orden y el tiempo son impredecibles.

**Uso:** Contadores de likes, vistas de videos, métricas no críticas.

---

### Resumen visual

| Nivel                 | Consistencia | Rendimiento | Costo RUs | Caso de uso típico           |
|-----------------------|--------------|-------------|-----------|------------------------------|
| **Strong**            | ⭐⭐⭐⭐⭐        | ⭐           | ⭐⭐⭐⭐⭐     | Saldos bancarios             |
| **Bounded Staleness** | ⭐⭐⭐⭐         | ⭐⭐          | ⭐⭐⭐⭐      | Dashboards en tiempo real    |
| **Session** ⭐ default | ⭐⭐⭐          | ⭐⭐⭐⭐        | ⭐⭐⭐       | E-commerce, apps web y móvil |
| **Consistent Prefix** | ⭐⭐           | ⭐⭐⭐⭐        | ⭐⭐        | Logs, timelines              |
| **Eventual**          | ⭐            | ⭐⭐⭐⭐⭐       | ⭐         | Likes, contadores            |

---

## 🗂️ 6. Partition Key — El concepto más crítico

La **Partition Key** es la decisión de diseño más importante en Cosmos DB. Una mala elección destruye el rendimiento.
Una buena elección permite escalar infinitamente.

### ¿Qué es una partición?

Cosmos DB divide físicamente tus datos en **particiones lógicas** basadas en el valor de la Partition Key. Cada
partición puede vivir en un servidor diferente.

```
Contenedor: "pedidos"  |  Partition Key: /clienteId

┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  Partición A     │  │  Partición B     │  │  Partición C     │
│  clienteId=001   │  │  clienteId=002   │  │  clienteId=003   │
│                  │  │                  │  │                  │
│  { pedido-1 }    │  │  { pedido-4 }    │  │  { pedido-7 }    │
│  { pedido-2 }    │  │  { pedido-5 }    │  │  { pedido-8 }    │
│  { pedido-3 }    │  │  { pedido-6 }    │  │  { pedido-9 }    │
└──────────────────┘  └──────────────────┘  └──────────────────┘
     Servidor 1             Servidor 2             Servidor 3
```

Cuando `clienteId=001` consulta sus pedidos → Cosmos DB va directo al Servidor 1. Sin buscar en los demás. Eso es lo que
hace las queries ultrarrápidas.

### Reglas de oro para elegir una buena Partition Key

**✅ Regla 1 — Alta cardinalidad:** Muchos valores posibles. `clienteId` (millones) es bueno. `pais` (pocos) es malo.

**✅ Regla 2 — Distribución uniforme:** Los datos deben repartirse equitativamente. Si el 80% tiene el mismo valor,
tienes una "hot partition" que satura un servidor.

**✅ Regla 3 — Aparece en tus queries frecuentes:** La query más común debería filtrar por la Partition Key. Si no lo
hace, Cosmos DB hace un "cross-partition query" — lento y caro en RUs.

### Ejemplos

```
Contenedor: pedidos
✅ BUENA:  /clienteId   → millones de valores, distribución uniforme
✅ BUENA:  /pedidoId    → máxima distribución posible
❌ MALA:   /estado      → solo 3 valores: pendiente/enviado/entregado
❌ MALA:   /pais        → pocos valores, hot partition en países grandes

Contenedor: productos
✅ BUENA:  /categoriaId → decenas de categorías, razonable
✅ BUENA:  /productoId  → máxima distribución
❌ MALA:   /activo      → solo true/false → hot partition masiva garantizada
```

> ⚠️ **La Partition Key NO se puede cambiar después de crear el contenedor.** Es una decisión permanente e irreversible.
> Por eso es crítico elegirla bien desde el inicio.

---

## 💰 7. Throughput — RUs (Request Units)

Las **Request Units (RU/s)** son la moneda de Cosmos DB. Todo lo que haces consume RUs: leer, escribir, actualizar,
eliminar y ejecutar queries.

**Referencia base:** leer un documento de 1KB consume exactamente **1 RU**.

| Operación                                 | RUs aproximadas |
|-------------------------------------------|-----------------|
| Leer 1KB por `id` + Partition Key         | 1 RU            |
| Escribir 1KB                              | 5 RUs           |
| Query simple con Partition Key            | 2–3 RUs         |
| Query sin Partition Key (cross-partition) | 10–100+ RUs     |
| Query compleja con agregaciones           | 50–500+ RUs     |

### Modos de throughput

| Modo            | Descripción                                               | Ideal para                              |
|-----------------|-----------------------------------------------------------|-----------------------------------------|
| **Provisioned** | Defines RU/s fijas. Pagas siempre esa capacidad.          | Tráfico predecible y constante          |
| **Autoscale**   | Defines un máximo. Escala entre 10% y 100% según demanda. | Tráfico variable — **el más común hoy** |
| **Serverless**  | Pagas por operación individual. Sin throughput definido.  | Desarrollo, pruebas, tráfico muy bajo   |

---

## ⚖️ 8. Cosmos DB vs Azure SQL — ¿Cuándo usar cada uno?

| Escenario                             | 🗃️ Azure SQL | 🌌 Cosmos DB |
|---------------------------------------|---------------|--------------|
| Sistema de contabilidad o ERP         | ✅             | ❌            |
| Catálogo de productos de e-commerce   | ⚠️            | ✅            |
| Historial de transacciones bancarias  | ✅             | ⚠️           |
| Perfil de usuario con datos variables | ❌             | ✅            |
| Reportes con JOINs complejos          | ✅             | ❌            |
| Feed de actividad en tiempo real      | ❌             | ✅            |
| Inventario crítico con ACID estricto  | ✅             | ⚠️           |
| IoT — millones de eventos por segundo | ❌             | ✅            |
| App con usuarios en múltiples países  | ⚠️            | ✅            |

> 💡 **La respuesta madura en entrevista:** En arquitecturas modernas no es excluyente. Lo ideal es usar **ambos**: Azure
> SQL para datos transaccionales críticos y Cosmos DB para alta escala y baja latencia. Este patrón se llama
> **persistencia políglota**.

---

## ☕ 9. Integración con Spring Boot

### Dependencias en `pom.xml`

```xml

<dependencies>
    <!-- SDK de Azure Cosmos DB — incluye Spring Data Cosmos -->
    <dependency>
        <groupId>com.azure.spring</groupId>
        <artifactId>spring-cloud-azure-starter-data-cosmos</artifactId>
        <version>5.8.0</version>
    </dependency>

    <!-- Azure Identity — autenticación con Managed Identity -->
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-identity</artifactId>
        <version>1.11.0</version>
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

### `application.properties`

```properties
# ============================================================
# Configuración Azure Cosmos DB
# Los valores reales vienen de Azure Application Settings
# ============================================================
# URI de la cuenta — Formato: https://[cuenta].documents.azure.com:443/
spring.cloud.azure.cosmos.endpoint=${COSMOS_ENDPOINT}
# Clave de acceso (Primary Key del portal)
# En producción preferir Managed Identity
spring.cloud.azure.cosmos.key=${COSMOS_KEY}
# Nombre de la base de datos
spring.cloud.azure.cosmos.database=${COSMOS_DATABASE}
# Nivel de consistencia
spring.cloud.azure.cosmos.consistency-level=Session
```

---

## 💻 10. Ejemplo de Código

### Escenario

Catálogo de productos para e-commerce. Los productos tienen estructuras variadas (algunos con variantes de talla/color,
otros no) — Cosmos DB es la elección correcta. La Partition Key es `/categoria` porque la query más frecuente es "dame
productos de una categoría".

### `Producto.java` — Modelo de documento

```java
package com.ejemplo.cosmos.model;

import com.azure.spring.data.cosmos.core.mapping.Container;
import com.azure.spring.data.cosmos.core.mapping.PartitionKey;
import org.springframework.data.annotation.Id;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

/**
 * DIFERENCIAS CLAVE con JPA:
 * - @Container en lugar de @Table
 * - @PartitionKey define la distribución de datos
 * - El ID es String (UUID), no Long autoincremental
 * - No hay @Column — todos los campos se serializan como JSON automáticamente
 * - Listas y objetos anidados sin tablas adicionales ni JOINs
 */
@Container(containerName = "productos")
public class Producto {

    @Id
    private String id;              // UUID — no Long autoincremental

    @PartitionKey
    private String categoria;       // Partition Key

    private String nombre;
    private String descripcion;
    private BigDecimal precio;
    private Integer stock;
    private Boolean activo;
    private String marca;

    // En SQL: tabla separada "etiquetas" con JOIN
    // En Cosmos DB: lista dentro del mismo documento — sin JOIN
    private List<String> etiquetas;

    // En SQL: tabla separada "variantes" con FK a productos + JOIN
    // En Cosmos DB: objeto anidado dentro del JSON — sin JOIN
    private List<VarianteProducto> variantes;

    private LocalDateTime fechaCreacion;
    private LocalDateTime fechaActualizacion;

    public Producto() {
    }

    public Producto(String id, String categoria, String nombre, BigDecimal precio) {
        this.id = id;
        this.categoria = categoria;
        this.nombre = nombre;
        this.precio = precio;
        this.activo = true;
        this.fechaCreacion = LocalDateTime.now();
        this.fechaActualizacion = LocalDateTime.now();
    }

    // Getters y Setters
    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getCategoria() {
        return categoria;
    }

    public void setCategoria(String c) {
        this.categoria = c;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String n) {
        this.nombre = n;
    }

    public String getDescripcion() {
        return descripcion;
    }

    public void setDescripcion(String d) {
        this.descripcion = d;
    }

    public BigDecimal getPrecio() {
        return precio;
    }

    public void setPrecio(BigDecimal p) {
        this.precio = p;
    }

    public Integer getStock() {
        return stock;
    }

    public void setStock(Integer s) {
        this.stock = s;
    }

    public Boolean getActivo() {
        return activo;
    }

    public void setActivo(Boolean a) {
        this.activo = a;
    }

    public String getMarca() {
        return marca;
    }

    public void setMarca(String m) {
        this.marca = m;
    }

    public List<String> getEtiquetas() {
        return etiquetas;
    }

    public void setEtiquetas(List<String> e) {
        this.etiquetas = e;
    }

    public List<VarianteProducto> getVariantes() {
        return variantes;
    }

    public void setVariantes(List<VarianteProducto> v) {
        this.variantes = v;
    }

    public LocalDateTime getFechaCreacion() {
        return fechaCreacion;
    }

    public LocalDateTime getFechaActualizacion() {
        return fechaActualizacion;
    }

    public void setFechaActualizacion(LocalDateTime f) {
        this.fechaActualizacion = f;
    }
}
```

### `VarianteProducto.java` — Objeto anidado

```java
package com.ejemplo.cosmos.model;

import java.math.BigDecimal;

/**
 * Objeto anidado dentro de Producto.
 * No requiere anotaciones especiales — Cosmos DB lo serializa como JSON automáticamente.
 * Elimina la necesidad de una tabla "variantes" con JOIN en SQL.
 */
public class VarianteProducto {

    private String sku;
    private String talla;
    private String color;
    private Integer stock;
    private BigDecimal precio;

    public VarianteProducto() {
    }

    public String getSku() {
        return sku;
    }

    public void setSku(String sku) {
        this.sku = sku;
    }

    public String getTalla() {
        return talla;
    }

    public void setTalla(String talla) {
        this.talla = talla;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public Integer getStock() {
        return stock;
    }

    public void setStock(Integer stock) {
        this.stock = stock;
    }

    public BigDecimal getPrecio() {
        return precio;
    }

    public void setPrecio(BigDecimal precio) {
        this.precio = precio;
    }
}
```

### `ProductoRepository.java` — Repositorio Spring Data Cosmos

```java
package com.ejemplo.cosmos.repository;

import com.azure.spring.data.cosmos.repository.CosmosRepository;
import com.ejemplo.cosmos.model.Producto;
import org.springframework.stereotype.Repository;

import java.math.BigDecimal;
import java.util.List;

/**
 * CosmosRepository funciona igual que JpaRepository.
 * Spring Data genera las queries por el nombre del método.
 *
 * EFICIENCIA:
 * ✅ Métodos con Partition Key (/categoria) → busca solo en esa partición → rápido y barato
 * ⚠️ Métodos sin Partition Key → cross-partition query → más lento y costoso en RUs
 */
@Repository
public interface ProductoRepository extends CosmosRepository<Producto, String> {

    // ✅ EFICIENTE — Partition Key incluida
    List<Producto> findByCategoria(String categoria);

    // ✅ EFICIENTE — Partition Key + filtro adicional
    List<Producto> findByCategoriaAndActivoTrue(String categoria);

    // ✅ EFICIENTE — Partition Key + rango de precios
    List<Producto> findByCategoriaAndPrecioBetween(
            String categoria, BigDecimal min, BigDecimal max);

    // ⚠️ CROSS-PARTITION — necesario cuando no se conoce la categoría
    List<Producto> findByNombreContainingIgnoreCase(String nombre);
}
```

### `ProductoService.java`

```java
package com.ejemplo.cosmos.service;

import com.ejemplo.cosmos.model.Producto;
import com.ejemplo.cosmos.repository.ProductoRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;
import java.util.UUID;

@Service
public class ProductoService {

    @Autowired
    private ProductoRepository productoRepository;

    public Producto crear(Producto producto) {
        // En Cosmos DB el ID es String UUID, no Long autoincremental
        if (producto.getId() == null || producto.getId().isBlank()) {
            producto.setId(UUID.randomUUID().toString());
        }
        producto.setFechaActualizacion(LocalDateTime.now());
        return productoRepository.save(producto);
    }

    public Optional<Producto> buscarPorId(String id) {
        return productoRepository.findById(id);
    }

    public List<Producto> listarPorCategoria(String categoria) {
        // ✅ Query eficiente — usa la Partition Key
        return productoRepository.findByCategoriaAndActivoTrue(categoria);
    }

    public List<Producto> buscarPorNombre(String nombre) {
        // ⚠️ Cross-partition — necesario cuando no se conoce la categoría
        return productoRepository.findByNombreContainingIgnoreCase(nombre);
    }

    public Producto actualizar(Producto producto) {
        producto.setFechaActualizacion(LocalDateTime.now());
        // save() en Cosmos DB hace upsert: inserta si no existe, actualiza si existe
        return productoRepository.save(producto);
    }

    public void eliminar(String id) {
        productoRepository.deleteById(id);
    }
}
```

### `ProductoController.java`

```java
package com.ejemplo.cosmos.controller;

import com.ejemplo.cosmos.model.Producto;
import com.ejemplo.cosmos.service.ProductoService;
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

    @PostMapping
    public ResponseEntity<Producto> crear(@RequestBody Producto producto) {
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(productoService.crear(producto));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Producto> buscarPorId(@PathVariable String id) {
        return productoService.buscarPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // ✅ Eficiente — filtra por Partition Key
    @GetMapping("/categoria/{categoria}")
    public ResponseEntity<List<Producto>> listarPorCategoria(
            @PathVariable String categoria) {
        return ResponseEntity.ok(productoService.listarPorCategoria(categoria));
    }

    // ⚠️ Cross-partition — solo cuando no se conoce la categoría
    @GetMapping("/buscar")
    public ResponseEntity<List<Producto>> buscar(@RequestParam String nombre) {
        return ResponseEntity.ok(productoService.buscarPorNombre(nombre));
    }

    @PutMapping
    public ResponseEntity<Producto> actualizar(@RequestBody Producto producto) {
        return ResponseEntity.ok(productoService.actualizar(producto));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable String id) {
        productoService.eliminar(id);
        return ResponseEntity.noContent().build();
    }
}
```

### ¿Cómo luce un documento guardado en Cosmos DB?

```json
{
  "id": "a3f8c2d1-9b4e-4f7a-8c6d-2e1f0a9b3c5d",
  "categoria": "electronica",
  "nombre": "Laptop HP Pavilion 15",
  "descripcion": "Laptop para uso profesional y gaming ligero",
  "precio": 2899.99,
  "stock": 45,
  "activo": true,
  "marca": "HP",
  "etiquetas": [
    "laptop",
    "hp",
    "gaming",
    "windows11"
  ],
  "variantes": [
    {
      "sku": "HP-PAV15-8GB-PLATA",
      "talla": null,
      "color": "Plateado",
      "stock": 30,
      "precio": 2899.99
    },
    {
      "sku": "HP-PAV15-16GB-PLATA",
      "talla": null,
      "color": "Plateado",
      "stock": 15,
      "precio": 3299.99
    }
  ],
  "fechaCreacion": "2025-03-15T10:30:00",
  "fechaActualizacion": "2025-03-15T10:30:00",
  "_rid": "abc123==",
  "_ts": 1710498600,
  "_etag": "\"00001234\"",
  "_self": "dbs/abc/colls/def/docs/ghi"
}
```

> 📌 Los campos `_rid`, `_ts`, `_etag` y `_self` son **metadatos internos** que Cosmos DB agrega automáticamente. No los
> defines tú. El `_etag` es especialmente útil para **control de concurrencia optimista**: si dos usuarios actualizan el
> mismo documento simultáneamente, Cosmos DB detecta el conflicto comparando los ETags.

---

## 🎥 11. Recursos y Videos Recomendados

| Recurso                                | Descripción                                 | Dónde encontrarlo                                                                         |
|----------------------------------------|---------------------------------------------|-------------------------------------------------------------------------------------------|
| **Microsoft Learn — Cosmos DB**        | Ruta oficial con laboratorios gratuitos     | [learn.microsoft.com/azure/cosmos-db](https://learn.microsoft.com/es-es/azure/cosmos-db/) |
| **"Azure Cosmos DB for Beginners"**    | Serie oficial del equipo de Cosmos DB       | Buscar en YouTube: `"Azure Cosmos DB beginners Microsoft"`                                |
| **"Consistency Levels Explained"**     | Video corto y muy claro sobre los 5 niveles | Buscar en YouTube: `"Cosmos DB consistency levels explained"`                             |
| **"Spring Boot + Cosmos DB Tutorial"** | Integración completa paso a paso            | Buscar en YouTube: `"Spring Boot Azure Cosmos DB tutorial 2024"`                          |

### 🎁 Recurso especial — Cosmos DB Emulator (sin cuenta Azure)

Microsoft ofrece un **Cosmos DB Emulator** completamente **gratuito** para Windows. Simula el comportamiento real de
Cosmos DB en tu PC sin cuenta de Azure ni tarjeta de crédito. Incluye interfaz web para explorar documentos, ejecutar
queries y ver consumo de RUs. **Es perfecto para tu situación actual.**

Búscalo como **"Azure Cosmos DB Emulator"** en la documentación oficial de Microsoft (`learn.microsoft.com`). La
instalación es un `.msi` directo.

---

## 🧠 Resumen Ejecutivo

> Lo que deberías poder decir en una entrevista: *"¿Qué es Cosmos DB y cuándo lo usarías?"*

**Azure Cosmos DB** es la base de datos NoSQL distribuida globalmente de Azure, diseñada para baja latencia garantizada
por SLA (menos de 10ms), escala elástica masiva y disponibilidad en múltiples regiones simultáneamente. Los datos se
almacenan como documentos JSON con esquema flexible — cada documento puede tener campos distintos sin migraciones. Se
organiza en cuentas, bases de datos, contenedores y documentos. El concepto más crítico es la **Partition Key**:
determina cómo se distribuyen los datos entre servidores, debe tener alta cardinalidad y distribución uniforme, y no
puede cambiarse una vez creado el contenedor. El throughput se mide en **RU/s** — leer 1KB cuesta 1 RU — y se gestiona
con autoscale para adaptarse al tráfico variable. Ofrece cinco modelos de consistencia: **Session** es el default y el
más usado en apps web. Desde Spring Boot se integra con `spring-cloud-azure-starter-data-cosmos` usando
`CosmosRepository`, que funciona igual que `JpaRepository`. Lo elegiría sobre Azure SQL cuando los datos tienen
estructura variable, la app necesita escala global, la latencia debe ser menor a 10ms, o el volumen supera lo que una BD
relacional maneja eficientemente. En arquitecturas modernas lo ideal es usar **ambos** — patrón conocido como
**persistencia políglota**.

---

## ⏭️ Siguiente módulo

> 📦 **Módulo 04 — Azure Storage:** El servicio de almacenamiento de objetos de Azure. Veremos Blob Storage, File
> Storage, Queue Storage y Table Storage, cuándo usar cada uno, y cómo subir y descargar archivos desde Spring Boot.

---

*📝 Documentación elaborada como parte del plan de aprendizaje Azure para desarrolladores Java Backend.*  
*🗓️ Última actualización: 2025*
