---
api: "ntgy-newco-s-postgresql-napai"
sidebar_position: 6
sidebar_label: "POST /ejecutarSecureQuery"
sql_metadata:
  data_sources:
    - database: "dlakeservi"
      schema: null
      type: "PostgreSQL"
      tables:
        - name: "ALL"
          operations: ["READ"]
---

# Endpoint: POST /ejecutarSecureQuery

## ⚠️ ADVERTENCIA DE USO

**ENDPOINT DE CONSULTAS PARAMETRIZADAS - SOLO PARA APIS INTERNAS AUTORIZADAS**

---

## 📋 Descripción General

Ejecuta consultas SQL parametrizadas contra la base de datos PostgreSQL utilizando query parameters para la query SQL y body JSON para los valores de los parámetros.

**Tipo de API:** System API (Internal)  
**Capa:** System Layer  
**Versión:** 1.12.0-SNAPSHOT  
**Última actualización:** 2026-05-21

---

## 🎯 Propósito

Este endpoint proporciona una alternativa más segura a `/ejecutarQuery` mediante:
- Separación de la query SQL y sus valores de parámetros
- Parametrización de queries utilizando el Database Connector
- Prevención de SQL injection mediante placeholders
- Consultas ad-hoc con mayor seguridad

**Casos de uso:**
- Consultas parametrizadas desde APIs de proceso y experiencia
- Queries con múltiples parámetros dinámicos
- Operaciones que requieren mayor seguridad en la parametrización

---

## 🔄 Diferencias con /ejecutarQuery

| Característica | /ejecutarQuery | /ejecutarSecureQuery |
|----------------|----------------|----------------------|
| **Query SQL** | En el body JSON | En query parameter |
| **Parámetros** | Concatenados en la query | Separados en el body JSON |
| **Parametrización** | Manual (concatenación) | Automática (Database Connector) |
| **Seguridad** | Riesgo de SQL injection | Parametrización segura |
| **Uso recomendado** | Queries simples sin parámetros | Queries con múltiples parámetros |

---

## 🔗 URL del Endpoint

```
POST /api/ejecutarSecureQuery?query=<SQL_QUERY>
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-napai-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-napai-pro.cloudhub.io` (Acceso restringido vía API Gateway)

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `query` | string | Sí | Consulta SQL con placeholders (`:paramName`) para los parámetros |

**Ejemplo:**
```
?query=SELECT * FROM dlakeservi.self_publication_sistemas.facturas_ni_omega WHERE cod_cups = :cups AND cod_documento_identidad = :nif
```

### Request Body (JSON)

```json
{
  "cups": "ES0021000000000001JN",
  "nif": "12345678A"
}
```

**Descripción:** Objeto JSON donde cada clave corresponde a un placeholder en la query SQL.

### Headers

| Header | Valor | Requerido | Descripción |
|--------|-------|-----------|-------------|
| `Content-Type` | `application/json` | Sí | Tipo de contenido del request |

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

```json
[
  {
    "cod_num_factura": "123456789",
    "cod_cups": "ES0021000000000001JN",
    "cod_documento_identidad": "12345678A",
    "imp_total_factura": 125.50,
    "fec_emision_factura": "2026-01-15",
    "fec_vencimiento_factura": "2026-02-15",
    "deuda": "N"
  }
]
```

**Descripción:** Array de objetos JSON con los resultados de la query. La estructura depende de las columnas seleccionadas en el SELECT.

---

### ❌ Errores

#### 500 Error (Código por defecto para todos los errores)

```json
{
  "errorCode": 500,
  "message": "DB:QUERY_EXECUTION",
  "description": "ERROR: syntax error at or near \"SELEC\""
}
```

**Estructura del error (basada en error-managerExpSys.xml):**
- `errorCode`: Código HTTP (por defecto 500, puede variar según `vars.httpStatus`)
- `message`: Tipo de error (`error.errorType.identifier` o `payload.message`)
- `description`: Descripción detallada del error (`error.description`)

**Tipos de errores comunes:**

**1. Error de sintaxis SQL:**
```json
{
  "errorCode": 500,
  "message": "DB:QUERY_EXECUTION",
  "description": "ERROR: syntax error at or near \"SELEC\""
}
```

**2. Error de conectividad:**
```json
{
  "errorCode": 500,
  "message": "DB:CONNECTIVITY",
  "description": "Could not connect to database"
}
```

**3. Query parameter faltante (validado por APIKit):**
```json
{
  "errorCode": 500,
  "message": "APIKIT:BAD_REQUEST",
  "description": "Required query parameter query not specified"
}
```

**Causas comunes:**
- Error de sintaxis SQL
- Query inválido o tabla inexistente
- Parámetros no coinciden con placeholders
- Base de datos no disponible
- Pool de conexiones agotado
- Query parameter `query` faltante
- Body JSON mal formado

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consultar facturas por CUPS y NIF

```bash
curl -X POST "http://localhost:8081/api/ejecutarSecureQuery?query=SELECT%20*%20FROM%20dlakeservi.self_publication_sistemas.facturas_ni_omega%20WHERE%20cod_cups%20%3D%20%3Acups%20AND%20cod_documento_identidad%20%3D%20%3Anif%20LIMIT%205" \
  -H "Content-Type: application/json" \
  -d '{
    "cups": "ES0021000000000001JN",
    "nif": "12345678A"
  }'
```

**Query decodificada:**
```sql
SELECT * FROM dlakeservi.self_publication_sistemas.facturas_ni_omega 
WHERE cod_cups = :cups AND cod_documento_identidad = :nif LIMIT 5
```

---

### Ejemplo 2: Consultar con múltiples parámetros

```bash
curl -X POST "http://localhost:8081/api/ejecutarSecureQuery?query=SELECT%20cod_num_factura%2C%20imp_total_factura%20FROM%20dlakeservi.self_publication_sistemas.facturas_ni_omega%20WHERE%20cod_documento_identidad%20%3D%20%3Anif%20AND%20fec_emision_factura%20%3E%3D%20%3AfechaDesde%20AND%20fec_emision_factura%20%3C%3D%20%3AfechaHasta" \
  -H "Content-Type: application/json" \
  -d '{
    "nif": "12345678A",
    "fechaDesde": "2025-01-01",
    "fechaHasta": "2025-12-31"
  }'
```

**Query decodificada:**
```sql
SELECT cod_num_factura, imp_total_factura 
FROM dlakeservi.self_publication_sistemas.facturas_ni_omega 
WHERE cod_documento_identidad = :nif 
  AND fec_emision_factura >= :fechaDesde 
  AND fec_emision_factura <= :fechaHasta
```

---

### Ejemplo 3: Consultar datos técnicos con parámetro

```bash
curl -X POST "http://localhost:8081/api/ejecutarSecureQuery?query=SELECT%20cod_cups%2C%20cod_tarifa%20FROM%20dlakeservi.self_publication_sistemas.tnpi_dim_suministros_ele%20WHERE%20cod_tarifa%20%3D%20%3Atarifa" \
  -H "Content-Type: application/json" \
  -d '{
    "tarifa": "2.0TD"
  }'
```

---

## 🔧 Implementación Técnica

### Flujo de Ejecución

```
HTTP POST Request → APIKit Router → ejecutarSecureQuerySub_Flow → 
Extract Query Parameter → Database Query (Parametrized) → 
DataWeave Transform → JSON Response
```

### Código Fuente

**Archivo:** `src/main/mule/querys/ejecutarSecureQuery.xml`

**Sub-flow:** `ejecutarSecureQuerySub_Flow`

### Diferencias Clave en el Código

**ejecutarQuery.xml:**
```xml
<db:sql><![CDATA[#[payload.query]]]></db:sql>
```

**ejecutarSecureQuery.xml:**
```xml
<db:sql><![CDATA[#[attributes.queryParams.query]]]></db:sql>
<db:input-parameters><![CDATA[#[payload]]]></db:input-parameters>
```

**Diferencia:** `ejecutarSecureQuery` utiliza `<db:input-parameters>` para parametrización segura.

### Transformación DataWeave

```dataweave
%dw 2.0
output application/json
---
payload
```

**Descripción:** Convierte directamente el resultado de la query a JSON sin transformaciones adicionales.

### Configuración de Base de Datos

- **Connector:** Database Connector (PostgreSQL)
- **Fetch Size:** `${postgresql.fetchSize}` (configurable por ambiente)
- **Connection Pooling:** Max 30 conexiones
- **Schema:** `dlakeservi.self_publication_sistemas`
- **Parametrización:** Automática mediante `input-parameters`

---

## 🔒 Seguridad y Controles

### ✅ Controles Implementados

**Autenticación y Autorización:**
- ✅ **API Gateway:** Autenticación OAuth/API Key implementada
- ✅ **Autorización:** Solo APIs internas con permisos específicos
- ✅ **Red Privada:** No expuesta públicamente, acceso solo interno

**Protección de Base de Datos:**
- ✅ **Usuario de BD:** `mulesoft_ro`
- ✅ **Parametrización Segura:** Uso de `input-parameters` del Database Connector
- ✅ **Connection Pooling:** Máximo 30 conexiones para evitar saturación
- ✅ **Timeout Configurado:** Límite de tiempo de ejecución por ambiente
- ✅ **Solo operaciones SELECT:** El código implementado solo ejecuta queries SELECT

**Auditoría y Monitoreo:**
- ✅ **Logging Completo:** Query y parámetros registrados por separado
- ✅ **Trazabilidad:** Logs de inicio y fin de ejecución
- ✅ **Monitoreo:** CloudHub monitoring activo

### ⚠️ Consideraciones de Uso

**Ventajas sobre /ejecutarQuery:**
- ✅ **Mayor Seguridad:** Parametrización automática previene SQL injection
- ✅ **Separación de Concerns:** Query SQL separada de los valores
- ✅ **Mejor Auditoría:** Query y parámetros logueados por separado

**Riesgos Residuales:**
- ⚠️ **Performance:** Queries complejas pueden afectar el rendimiento de la BD
- ⚠️ **Validación:** No hay validación de sintaxis SQL antes de ejecución
- ⚠️ **Exposición de Datos:** El endpoint permite ejecutar cualquier query SQL que el usuario de BD tenga permisos para ejecutar

**Recomendaciones:**
- Preferir este endpoint sobre `/ejecutarQuery` cuando se usen parámetros dinámicos
- Validar queries antes de ejecutar en producción
- Monitorear performance de queries ejecutadas
- Documentar queries frecuentes para crear endpoints dedicados

---

## ⚙️ Configuración de Producción

### Usuario de Base de Datos

```yaml
# properties-pro.yaml
postgresql:
  host: "dlakeservi-aurora-postgresql-napai.cluster-ro-cbapdsgpqbhu.eu-west-1.rds.amazonaws.com"
  port: "5432"
  database: "dlakeservi"
  username: "mulesoft_ro"
  fetchSize: "100"
  maxWaitTime: "15"
  maxIdleTime: "15"
```

**Nota:** Los permisos específicos del usuario `mulesoft_ro` en la base de datos deben ser verificados con el equipo de DBA.

### Operaciones Implementadas

**En el código MuleSoft:**
- ✅ Solo se utilizan operaciones `<db:select>` (SELECT)
- ✅ Parametrización mediante `<db:input-parameters>`
- ❌ No hay operaciones de escritura implementadas (INSERT, UPDATE, DELETE)

### Connection Pooling

```xml
<db:pooling-profile 
  maxPoolSize="30" 
  maxWait="${postgresql.maxWaitTime}" 
  maxIdleTime="${postgresql.maxIdleTime}" />
```

---

## 📊 Logging

### Logs Generados

```
INFO  - Inicio ejecucion NAPAI ejecutarSecureQuery: {"param1":"value1"}
INFO  - Query sin transformar: SELECT * FROM table WHERE col = :param1
INFO  - Fin ejecucion NAPAI ejecutarSecureQuery: [{"col1":"val1"}]
ERROR - DB:QUERY_EXECUTION: SQL syntax error...
```

### Categorías de Log

- **INFO:** Inicio y fin de ejecución, query SQL sin transformar
- **DEBUG:** Parámetros de entrada (si está habilitado)
- **ERROR:** Errores de base de datos o ejecución

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-napai.raml`
- **Main Flow:** `src/main/mule/querys/ejecutarSecureQuery.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`
- **Database Config:** `src/main/mule/global.xml`

---

## 🔗 Endpoints Relacionados

- **POST /ejecutarQuery** - Versión sin parametrización (menos segura)
- **GET /invoices** - Consulta específica de facturas
- **GET /technical_info_e** - Datos técnicos de electricidad
- **GET /technical_info_g** - Datos técnicos de gas

---

## 📌 Notas Importantes

### Uso en Producción

**Estado Actual:**
- ✅ Endpoint activo y operacional en todos los ambientes
- ✅ Protegido por API Gateway con autenticación/autorización
- ✅ Usuario de BD configurado: `mulesoft_ro`
- ✅ Parametrización segura implementada
- ✅ Acceso restringido a APIs internas autorizadas

### Recomendaciones de Uso

**Cuándo usar este endpoint:**
- ✅ Queries con parámetros dinámicos (WHERE clauses con valores variables)
- ✅ Cuando se requiere mayor seguridad en la parametrización
- ✅ Queries que se ejecutarán con diferentes valores de parámetros

**Cuándo usar /ejecutarQuery en su lugar:**
- ✅ Queries estáticas sin parámetros
- ✅ Queries de análisis ad-hoc sin valores dinámicos
- ✅ Cuando la simplicidad es más importante que la parametrización

**Mejores Prácticas:**
- ✅ Usar placeholders (`:paramName`) en lugar de concatenación
- ✅ Validar que todos los placeholders tengan valores en el body
- ✅ Preferir endpoints específicos cuando estén disponibles
- ✅ Monitorear performance de queries ejecutadas
- ✅ Documentar queries frecuentes para crear endpoints dedicados

**Casos de Uso Apropiados:**
- Consultas parametrizadas para análisis de datos
- Troubleshooting con filtros dinámicos
- Reporting con parámetros variables
- Prototipado de nuevos endpoints con parámetros

**Casos de Uso NO Recomendados:**
- Queries complejas con múltiples JOINs en horario pico
- Consultas recurrentes (crear endpoint específico en su lugar)
- Queries que retornen grandes volúmenes de datos sin paginación

---

**Última actualización:** 2026-05-21  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy