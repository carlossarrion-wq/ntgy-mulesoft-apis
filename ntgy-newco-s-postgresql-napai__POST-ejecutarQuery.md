---
api: "ntgy-newco-s-postgresql-napai"
sidebar_position: 5
sidebar_label: "POST /ejecutarQuery"
sql_metadata:
  data_sources:
    - database: "dlakeservi"
      schema: null
      type: "PostgreSQL"
      tables:
        - name: "ALL"
          operations: ["READ", "WRITE"]
---

# Endpoint: POST /ejecutarQuery

## ⚠️ ADVERTENCIA DE USO

**ENDPOINT DE CONSULTAS DINÁMICAS - SOLO PARA APIS INTERNAS AUTORIZADAS**

---

## 📋 Descripción General

Ejecuta consultas SQL SELECT dinámicas contra la base de datos PostgreSQL para proporcionar flexibilidad en el acceso a datos.

**Tipo de API:** System API (Internal)  
**Capa:** System Layer  
**Versión:** 1.12.0-SNAPSHOT  
**Última actualización:** 2026-05-21

---

## 🎯 Propósito

Este endpoint permite ejecutar queries SQL dinámicas para propósitos de:
- Consultas ad-hoc desde APIs de proceso y experiencia autorizadas
- Análisis de datos y reporting
- Operaciones de soporte y troubleshooting
- Flexibilidad para casos de uso no cubiertos por endpoints específicos

---

## 🔗 URL del Endpoint

```
POST /api/ejecutarQuery
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-napai-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-napai-pro.cloudhub.io` (Acceso restringido vía API Gateway)

---

## 📥 Parámetros de Entrada

### Request Body (JSON)

```json
{
  "query": "SELECT * FROM dlakeservi.self_publication_sistemas.facturas_ni_omega LIMIT 10"
}
```

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `query` | string | Sí | Consulta SQL SELECT a ejecutar contra la base de datos PostgreSQL |

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
  },
  {
    "cod_num_factura": "123456790",
    "cod_cups": "ES0021000000000002KP",
    "cod_documento_identidad": "87654321B",
    "imp_total_factura": 89.30,
    "fec_emision_factura": "2026-01-20",
    "fec_vencimiento_factura": "2026-02-20",
    "deuda": "S"
  }
]
```

**Descripción:** Array de objetos JSON con los resultados de la query. La estructura depende completamente de las columnas seleccionadas en el SELECT.

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

**3. Parámetro faltante (validado por APIKit):**
```json
{
  "errorCode": 500,
  "message": "APIKIT:BAD_REQUEST",
  "description": "Bad request"
}
```

**Causas comunes:**
- Error de sintaxis SQL
- Query inválido o tabla inexistente
- Base de datos no disponible
- Pool de conexiones agotado
- Campo `query` faltante en el body
- Body JSON mal formado

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consultar facturas

```bash
curl -X POST "http://localhost:8081/api/ejecutarQuery" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "SELECT cod_num_factura, imp_total_factura, fec_emision_factura FROM dlakeservi.self_publication_sistemas.facturas_ni_omega LIMIT 5"
  }'
```

**Respuesta:**
```json
[
  {
    "cod_num_factura": "123456789",
    "imp_total_factura": 125.50,
    "fec_emision_factura": "2026-01-15"
  }
]
```

---

### Ejemplo 2: Consultar con filtros

```bash
curl -X POST "http://localhost:8081/api/ejecutarQuery" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "SELECT * FROM dlakeservi.self_publication_sistemas.facturas_ni_omega WHERE cod_documento_identidad = '\''12345678A'\'' AND fec_emision_factura > CURRENT_DATE - INTERVAL '\''1 year'\''"
  }'
```

---

### Ejemplo 3: Consultar datos técnicos

```bash
curl -X POST "http://localhost:8081/api/ejecutarQuery" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "SELECT cod_cups, cod_tarifa FROM dlakeservi.self_publication_sistemas.tnpi_dim_suministros_ele WHERE cod_cups LIKE '\''ES0021%'\'' LIMIT 10"
  }'
```

---

## 🔧 Implementación Técnica

### Flujo de Ejecución

```
HTTP POST Request → APIKit Router → ejecutarQuerySub_Flow → 
Database Query → DataWeave Transform → JSON Response
```

### Código Fuente

**Archivo:** `src/main/mule/querys/ejecutarQuery.xml`

**Sub-flow:** `ejecutarQuerySub_Flow`

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

---

## 🔒 Seguridad y Controles

### ✅ Controles Implementados

**Autenticación y Autorización:**
- ✅ **API Gateway:** Autenticación OAuth/API Key implementada
- ✅ **Autorización:** Solo APIs internas con permisos específicos
- ✅ **Red Privada:** No expuesta públicamente, acceso solo interno

**Protección de Base de Datos:**
- ✅ **Usuario de BD:** `mulesoft_ro`
- ✅ **Connection Pooling:** Máximo 30 conexiones para evitar saturación
- ✅ **Timeout Configurado:** Límite de tiempo de ejecución por ambiente
- ✅ **Solo operaciones SELECT:** El código implementado solo ejecuta queries SELECT

**Auditoría y Monitoreo:**
- ✅ **Logging Completo:** Todas las queries son registradas
- ✅ **Trazabilidad:** Logs de inicio y fin de ejecución
- ✅ **Monitoreo:** CloudHub monitoring activo

### ⚠️ Consideraciones de Uso

**Riesgos Residuales:**
- ⚠️ **Performance:** Queries complejas pueden afectar el rendimiento de la BD
- ⚠️ **Validación:** No hay validación de sintaxis SQL antes de ejecución
- ⚠️ **Exposición de Datos:** El endpoint permite ejecutar cualquier query SQL que el usuario de BD tenga permisos para ejecutar

**Recomendaciones:**
- Preferir endpoints específicos cuando estén disponibles
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
- ❌ No hay operaciones de escritura implementadas (INSERT, UPDATE, DELETE)
- ⚠️ El endpoint `ejecutarQuery` podría ejecutar cualquier SQL que el usuario de BD tenga permisos para ejecutar

### Connection Pooling

```xml
<db:pooling-profile 
  maxPoolSize="30" 
  maxWait="${postgresql.maxWaitTime}" 
  maxIdleTime="${postgresql.maxIdleTime}" />
```

**Configuración:**
- **Max Pool Size:** 30 conexiones
- **Max Wait Time:** 15ms (producción)
- **Max Idle Time:** 15ms (producción)

---

## 📊 Logging

### Logs Generados

```
INFO  - Inicio ejecucion NAPAI ejecutarQuery: {"query":"SELECT ..."}
INFO  - Fin ejecucion NAPAI ejecutarQuery: [{"col1":"val1"}]
ERROR - DB:QUERY_EXECUTION: SQL syntax error...
```

### Categorías de Log

- **INFO:** Inicio y fin de ejecución
- **DEBUG:** Query SQL ejecutado (si está habilitado)
- **ERROR:** Errores de base de datos o ejecución

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-napai.raml`
- **Main Flow:** `src/main/mule/querys/ejecutarQuery.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`
- **Database Config:** `src/main/mule/global.xml`

---

## 🔗 Endpoints Relacionados

- **POST /ejecutarSecureQuery** - Versión con parametrización segura
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
- ✅ Acceso restringido a APIs internas autorizadas
- ⚠️ Usar con precaución: queries complejas pueden afectar performance

### Recomendaciones de Uso

**Mejores Prácticas:**
- ✅ Preferir endpoints específicos cuando estén disponibles (invoices, technical_info, etc.)
- ✅ Validar queries antes de ejecutar en producción
- ✅ Monitorear performance de queries ejecutadas
- ✅ Documentar queries frecuentes para crear endpoints dedicados
- ✅ Limitar el uso a casos donde no exista endpoint específico

**Casos de Uso Apropiados:**
- Consultas ad-hoc para análisis de datos
- Troubleshooting y soporte técnico
- Reporting y extracción de datos
- Prototipado de nuevos endpoints

**Casos de Uso NO Recomendados:**
- Queries complejas con múltiples JOINs en horario pico
- Consultas recurrentes (crear endpoint específico en su lugar)
- Queries que retornen grandes volúmenes de datos sin paginación
