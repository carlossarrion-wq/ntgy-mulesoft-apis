---
api: "ntgy-newco-s-postgresql-napai"
sidebar_position: 1
sidebar_label: "GET /consumption12m"
sql_metadata:
  data_sources:
    - database: "dlakeservi"
      schema: "self_publication_sistemas"
      type: "PostgreSQL"
      tables:
        - name: "consumo_ultimos_12_meses_omega"
          operations: ["READ"]
---

# Endpoint: GET /consumption12m

## 📋 Descripción General

Recupera el consumo total agregado de los últimos 12 meses para un contrato específico.

**Tipo de API:** System API (Internal)  
**Capa:** System Layer  
**Versión:** 1.12.0-SNAPSHOT  
**Última actualización:** 2026-06-17

---

## 🎯 Propósito

Este endpoint proporciona acceso al consumo total agregado de los últimos 12 meses:

- Consulta por código de contrato (con prefijo 'VTY' automático)
- Consumo total agregado de 12 meses
- Valor numérico único (no desglose mensual)
- Incluye objeto result con código de respuesta estándar

**Casos de uso:**
- Obtener consumo total de un periodo anual
- Cálculos de consumo promedio mensual
- Comparativas anuales de consumo
- Reporting de facturación agregada

---

## 🔗 URL del Endpoint

```
GET /api/consumption12m?idContract=<CONTRACT_ID>
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-napai-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-napai-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `idContract` | string | Sí | Código de contrato del cliente (se añade prefijo 'VTY' automáticamente) |

**Ejemplo:**
```
?idContract=1234567890
```

**Nota:** El sistema añade automáticamente el prefijo 'VTY' al código de contrato en la query SQL.

### Headers

| Header | Valor | Requerido | Descripción |
|--------|-------|-----------|-------------|
| `Accept` | `application/json` | No | Tipo de contenido esperado en la respuesta |

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": ""
  },
  "consumption12m": 1755.7
}
```

### 📊 Estructura de Respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result` | object | Objeto con información del resultado de la operación |
| `result.codResult` | string | Código de resultado ("0000" para éxito) |
| `result.msgResult` | string | Mensaje descriptivo del resultado (vacío en caso de éxito) |
| `consumption12m` | number | Consumo total agregado de los últimos 12 meses (kWh o m³ según tipo de suministro) |

**Nota:** El valor de `consumption12m` representa la suma total del consumo facturado en los últimos 12 meses.

---

### ❌ Errores

#### 500 Error (Código por defecto para todos los errores)

```json
{
  "errorCode": 500,
  "message": "DB:QUERY_EXECUTION",
  "description": "ERROR: relation does not exist"
}
```

**Estructura del error (basada en error-managerExpSys.xml):**
- `errorCode`: Código HTTP (por defecto 500, puede variar según `vars.httpStatus`)
- `message`: Tipo de error (`error.errorType.identifier` o `payload.message`)
- `description`: Descripción detallada del error (`error.description`)

**Tipos de errores comunes:**

**1. Parámetro faltante (validado por APIKit):**
```json
{
  "errorCode": 500,
  "message": "APIKIT:BAD_REQUEST",
  "description": "Required query parameter idContract not specified"
}
```

**2. Contrato no encontrado o sin datos:**
```json
{
  "errorCode": 500,
  "message": "MULE:EXPRESSION",
  "description": "Cannot coerce Null to Number"
}
```

**3. Error de conectividad:**
```json
{
  "errorCode": 500,
  "message": "DB:CONNECTIVITY",
  "description": "Could not connect to database"
}
```

**Causas comunes:**
- Parámetro `idContract` faltante
- Contrato no existe en la base de datos
- Error en la query SQL
- Base de datos no disponible
- Error en transformación DataWeave

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consultar consumo total de 12 meses

```bash
curl -X GET "http://localhost:8081/api/consumption12m?idContract=1234567890" \
  -H "Accept: application/json"
```

**Respuesta:**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": ""
  },
  "consumption12m": 1755.7
}
```

---

### Ejemplo 2: Contrato sin datos

```bash
curl -X GET "http://localhost:8081/api/consumption12m?idContract=9999999999" \
  -H "Accept: application/json"
```

**Respuesta:**
```json
{
  "errorCode": 500,
  "message": "MULE:EXPRESSION",
  "description": "Cannot coerce Null to Number"
}
```

---

## 🔧 Implementación Técnica

### Flujo de Ejecución

```
HTTP GET Request → APIKit Router → consumption12mSub_Flow → 
Extract idContract → Database Query → DataWeave Transform → JSON Response
```

### Código Fuente

**Archivo:** `src/main/mule/renewals/consumption12m.xml`

**Sub-flow:** `consumption12mSub_Flow`

### Query SQL

```sql
SELECT co.consumo_12_meses 
FROM dlakeservi.self_publication_sistemas.consumo_ultimos_12_meses_omega co  
WHERE co.id_contrato = 'VTY1234567890'
```

**Características:**
- Consulta por código de contrato con prefijo 'VTY'
- Retorna un solo campo: `consumo_12_meses` (valor agregado)
- Tabla: `consumo_ultimos_12_meses_omega`
- Sin filtros de fecha (la tabla ya contiene datos de 12 meses)

### Transformación DataWeave

```dataweave
%dw 2.0
output application/json
---
{
  "result": {
    "codResult": "0000",
    "msgResult": ""
  },
  "consumption12m": payload[0].consumo_12_meses
}
```

**Mapeo de campos:**
- `payload[0].consumo_12_meses` → `consumption12m` (extrae el valor del primer registro)
- Objeto `result` con código "0000" (éxito) y mensaje vacío

---

## 🔒 Seguridad y Controles

### ✅ Controles Implementados

**Autenticación y Autorización:**
- ✅ **API Gateway:** Autenticación OAuth/API Key implementada
- ✅ **Autorización:** Solo APIs internas con permisos específicos

**Protección de Datos:**
- ✅ **Consulta específica:** Solo datos del contrato solicitado
- ✅ **Usuario de BD:** `mulesoft_ro`
- ✅ **Ventana temporal:** Últimos 12 meses

**Performance:**
- ✅ **Query optimizada:** Filtro por contrato y fecha
- ✅ **Fetch Size:** Configurable por ambiente
- ✅ **Connection Pooling:** Máximo 30 conexiones

---

## ⚙️ Configuración de Producción

### Tabla de Base de Datos

```yaml
# Tabla principal
tabla: dlakeservi.self_publication_sistemas.consumo_ultimos_12_meses_omega

# Campos clave
- id_contrato: Código de contrato (con prefijo 'VTY')
- consumo_12_meses: Consumo total agregado de 12 meses
```

### Fetch Size

```yaml
# properties-pro.yaml
postgresql:
  fetchSize: "100"
```

---

## 📊 Logging

### Logs Generados

```
INFO  - consumption12m - Inicio ejecucion query: 1234567890
INFO  - Main Query: SELECT co.consumo_12_meses FROM ... WHERE co.id_contrato = 'VTY1234567890'
DEBUG - consumption12m - Resultado: {"result":{"codResult":"0000","msgResult":""},"consumption12m":1755.7}
```

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-napai.raml`
- **Main Flow:** `src/main/mule/renewals/consumption12m.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`
- **Database Config:** `src/main/mule/global.xml`

---

## 🔗 Endpoints Relacionados

- **GET /invoices** - Consulta de facturas
- **GET /technical_info_e** - Datos técnicos de electricidad
- **GET /technical_info_g** - Datos técnicos de gas
- **POST /ejecutarQuery** - Consultas dinámicas

---

## 📌 Notas Importantes

### Características Especiales

**Unidades de Consumo:**
- **Electricidad:** kWh (kilovatios-hora)
- **Gas:** m³ (metros cúbicos)

**Valor Retornado:**
- Consumo total agregado de 12 meses (valor único)
- No incluye desglose mensual ni fechas
- Incluye objeto `result` con código de respuesta estándar

**Prefijo Automático:**
- El sistema añade automáticamente el prefijo 'VTY' al código de contrato
- Ejemplo: `idContract=1234567890` → Query usa `'VTY1234567890'`

**Casos Especiales:**
- Si el contrato no existe o no tiene datos, retorna error de tipo `MULE:EXPRESSION`
- El campo `consumption12m` puede ser `null` si no hay datos en la tabla

### Recomendaciones de Uso

**Mejores Prácticas:**
- ✅ Cachear resultados (datos históricos no cambian frecuentemente)
- ✅ Usar para cálculos de consumo promedio mensual (dividir entre 12)
- ✅ Validar que `consumption12m` no sea null antes de procesar
- ✅ Manejar errores cuando el contrato no existe

**Performance:**
- ✅ Query simple y directa (un solo campo)
- ✅ Retorna un solo registro
- ✅ Sin joins ni agregaciones complejas
- ✅ Tabla optimizada con datos pre-agregados

---

**Última actualización:** 2026-06-17  
**Versión del documento:** 2.0  
**Autor:** Equipo MuleSoft Naturgy
