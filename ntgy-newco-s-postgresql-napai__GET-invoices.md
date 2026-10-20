---
api: "ntgy-newco-s-postgresql-napai"
sidebar_position: 2
sidebar_label: "GET /invoices"
sql_metadata:
  data_sources:
    - database: "dlakeservi"
      schema: "self_publication_sistemas"
      type: "PostgreSQL"
      tables:
        - name: "facturas"
          operations: ["READ"]
---

# Endpoint: GET /invoices

## 📋 Descripción General

Recupera las facturas de clientes con cálculo automático de familias de productos basado en el análisis de los subtipos de factura.

**Tipo de API:** System API (Internal)  
**Capa:** System Layer  
**Versión:** 1.12.0-SNAPSHOT  
**Última actualización:** 2026-05-21

---

## 🎯 Propósito

Este endpoint proporciona acceso a las facturas de clientes con las siguientes características:

- Consulta facturas de múltiples puntos de suministro (CUPS)
- Filtrado por NIF del cliente
- Ventana temporal de 5 años (cumplimiento GDPR)
- **Cálculo automático de familias de productos** por factura
- Deduplicación de facturas
- Información de disponibilidad de PDF

**Casos de uso:**
- Consulta de historial de facturas en portales de cliente
- Análisis de consumo y facturación
- Descarga de facturas en PDF
- Reporting de facturación por familia de producto

---

## 🔗 URL del Endpoint

```
GET /api/invoices?cups=<CUPS>&nif=<NIF>
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-napai-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-napai-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `cups` | string | Sí | Códigos CUPS separados por comas. Permite consultar facturas de múltiples puntos de suministro. |
| `nif` | string | Sí | NIF del cliente (Número de Identificación Fiscal) |

**Ejemplos:**
- Un CUPS: `?cups=ES0021000000000001JN&nif=12345678A`
- Múltiples CUPS: `?cups=ES0021000000000001JN,ES0021000000000002KP&nif=12345678A`

### Headers

| Header | Valor | Requerido | Descripción |
|--------|-------|-----------|-------------|
| `Accept` | `application/json` | No | Tipo de contenido esperado en la respuesta |

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

```json
[
  {
    "cuentaContrato": "1234567890",
    "invoiceId": "123456789",
    "invoiceNum": "123456789",
    "pdfDownload": true,
    "issuedDate": "2026-01-15",
    "billingDate": "2026-02-15",
    "amount": 125.50,
    "status": "",
    "pendingAmount": 0,
    "pdfDocumentId": "123456789",
    "statusDebt": "N",
    "productInvoiceRelationList": [
      {"productFamilyId": "gas"},
      {"productFamilyId": "luz"}
    ]
  },
  {
    "cuentaContrato": "1234567890",
    "invoiceId": "123456790",
    "invoiceNum": "123456790",
    "pdfDownload": false,
    "issuedDate": "2025-12-20",
    "billingDate": "2026-01-20",
    "amount": 89.30,
    "status": "",
    "pendingAmount": 0,
    "pdfDocumentId": "123456790",
    "statusDebt": "S",
    "productInvoiceRelationList": [
      {"productFamilyId": "luz"}
    ]
  }
]
```

### 📊 Estructura de Respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `cuentaContrato` | string | Código de contrato SAP |
| `invoiceId` | string | Identificador único de la factura |
| `invoiceNum` | string | Número de factura |
| `pdfDownload` | boolean | Indica si el PDF está disponible para descarga |
| `issuedDate` | date | Fecha de emisión de la factura |
| `billingDate` | date | Fecha de vencimiento de la factura |
| `amount` | number | Importe total de la factura |
| `status` | string | Estado de la factura (campo vacío) |
| `pendingAmount` | number | Importe pendiente (siempre 0) |
| `pdfDocumentId` | string | Identificador del documento PDF |
| `statusDebt` | string | Estado de deuda ("S" = Sí, "N" = No) |
| `productInvoiceRelationList` | array | Lista de familias de productos asociadas a la factura |

---

### ❌ Errores

#### 500 Error (Código por defecto para todos los errores)

```json
{
  "errorCode": 500,
  "message": "APIKIT:BAD_REQUEST",
  "description": "Required query parameter cups not specified"
}
```

**Estructura del error (basada en error-managerExpSys.xml):**
- `errorCode`: Código HTTP (por defecto 500, puede variar según `vars.httpStatus`)
- `message`: Tipo de error (`error.errorType.identifier` o `payload.message`)
- `description`: Descripción detallada del error (`error.description`)

**Tipos de errores comunes:**

**1. Parámetros faltantes (validado por APIKit):**
```json
{
  "errorCode": 500,
  "message": "APIKIT:BAD_REQUEST",
  "description": "Required query parameter cups not specified"
}
```

**2. Error en la query SQL:**
```json
{
  "errorCode": 500,
  "message": "DB:QUERY_EXECUTION",
  "description": "ERROR: relation \"facturas_ni_omega\" does not exist"
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

**4. Error en transformación DataWeave:**
```json
{
  "errorCode": 500,
  "message": "MULE:EXPRESSION",
  "description": "Error evaluating DataWeave expression"
}
```

**Causas comunes:**
- Parámetro `cups` o `nif` faltante
- Error en la ejecución de la query SQL
- Error en la transformación DataWeave (cálculo de familias)
- Base de datos no disponible
- Pool de conexiones agotado
- Datos inconsistentes en la base de datos

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consultar facturas de un CUPS

```bash
curl -X GET "http://localhost:8081/api/invoices?cups=ES0021000000000001JN&nif=12345678A" \
  -H "Accept: application/json"
```

**Respuesta:**
```json
[
  {
    "cuentaContrato": "1234567890",
    "invoiceId": "123456789",
    "invoiceNum": "123456789",
    "pdfDownload": true,
    "issuedDate": "2026-01-15",
    "billingDate": "2026-02-15",
    "amount": 125.50,
    "status": "",
    "pendingAmount": 0,
    "pdfDocumentId": "123456789",
    "statusDebt": "N",
    "productInvoiceRelationList": [
      {"productFamilyId": "gas"}
    ]
  }
]
```

---

### Ejemplo 2: Consultar facturas de múltiples CUPS

```bash
curl -X GET "http://localhost:8081/api/invoices?cups=ES0021000000000001JN,ES0021000000000002KP,ES0021000000000003LQ&nif=12345678A" \
  -H "Accept: application/json"
```

---

### Ejemplo 3: Cliente sin facturas

```bash
curl -X GET "http://localhost:8081/api/invoices?cups=ES0021000000000999ZZ&nif=99999999Z" \
  -H "Accept: application/json"
```

**Respuesta:**
```json
[]
```

---

## 🔧 Implementación Técnica

### Flujo de Ejecución

```
HTTP GET Request → APIKit Router → invoicesSub_Flow → 
Extract Parameters → Build SQL Query → Database Query → 
DataWeave Transformation (Product Family Calculation) → JSON Response
```

### Código Fuente

**Archivo:** `src/main/mule/invoices/invoices.xml`

**Sub-flow:** `invoicesSub_Flow`

### Query SQL Construida

```sql
SELECT DISTINCT *
FROM dlakeservi.self_publication_sistemas.facturas_ni_omega AS fno
WHERE fno.cod_cups IN ('ES0021000000000001JN', 'ES0021000000000002KP')
  AND fno.cod_documento_identidad = '12345678A'
  AND fno.fec_emision_factura > CURRENT_DATE - INTERVAL '5 years'
```

**Características:**
- **DISTINCT:** Elimina facturas duplicadas
- **IN clause:** Permite múltiples CUPS
- **Filtro temporal:** Últimos 5 años (GDPR)
- **Filtro por NIF:** Seguridad a nivel de datos

### Transformación DataWeave - Lógica de Familias de Productos

```dataweave
%dw 2.0
output application/json

fun hasGas(inv) = 
    sizeOf(payload filter (value, index) -> 
        ((value.cod_num_factura == inv) and (value.des_subtipo_factura contains("GAS")))) > 0

fun hasElec(inv) = 
    sizeOf(payload filter (value, index) -> 
        ((value.cod_num_factura == inv) and (value.des_subtipo_factura contains("ELECTRICIDAD")))) > 0

fun hasAmbas(inv) = 
    sizeOf(payload filter (value, index) -> 
        ((value.cod_num_factura == inv) and (value.des_subtipo_factura contains("DUAL")))) > 0

fun hasService(inv) = 
    sizeOf(payload filter (value, index) -> 
        ((value.cod_num_factura == inv) and (value.des_subtipo_factura contains("SERVICIOS")))) > 0

fun calculateFamily(inv) = 
    if (hasGas(inv) and !hasElec(inv) and !hasService(inv)) 
        [{productFamilyId: "gas"}]
    else if (hasGas(inv) and !hasElec(inv) and hasService(inv)) 
        [{productFamilyId: "gas"}, {productFamilyId: "servicios"}]
    else if (!hasGas(inv) and hasElec(inv) and !hasService(inv)) 
        [{productFamilyId: "luz"}]
    else if (!hasGas(inv) and hasElec(inv) and hasService(inv)) 
        [{productFamilyId: "luz"},{productFamilyId: "servicios"}]
    else if (hasAmbas(inv) and !hasService(inv)) 
        [{productFamilyId: "luz"},{productFamilyId: "gas"}]
    else if (hasAmbas(inv) and hasService(inv)) 
        [{productFamilyId: "luz"},{productFamilyId: "gas"},{productFamilyId: "servicios"}]
    else if (hasGas(inv) and hasElec(inv) and hasService(inv)) 
        [{productFamilyId: "gas"},{productFamilyId: "luz"},{productFamilyId: "servicios"}]
    else if (hasGas(inv) and hasElec(inv) and !hasService(inv)) 
        [{productFamilyId: "gas"},{productFamilyId: "luz"}]
    else if (!hasGas(inv) and !hasElec(inv) and !hasService(inv)) 
        []
    else if (!hasGas(inv) and !hasElec(inv) and hasService(inv)) 
        [{productFamilyId: "servicios"}]
    else []
---
payload distinctBy $.cod_num_factura map ((item,index) -> {
    "cuentaContrato": item.cod_contrato_sap_ult,
    "invoiceId": item.cod_num_factura,
    "invoiceNum": item.cod_num_factura,
    "pdfDownload": if(item.z.fecha_carga != "00000000") true else false,
    "issuedDate": item.fec_emision_factura,
    "billingDate": item.fec_vencimiento_factura,
    "amount": item.imp_total_factura,
    "status": "",
    "pendingAmount": 0,
    "pdfDocumentId": item.cod_num_factura,
    "statusDebt": item.deuda,
    "productInvoiceRelationList": calculateFamily(item.cod_num_factura)
})
```

---

## 🧮 Lógica de Cálculo de Familias de Productos

### Tabla de Combinaciones

| GAS | ELECTRICIDAD | DUAL | SERVICIOS | Resultado |
|-----|--------------|------|-----------|-----------|
| ✅ | ❌ | ❌ | ❌ | `[{productFamilyId: "gas"}]` |
| ✅ | ❌ | ❌ | ✅ | `[{productFamilyId: "gas"}, {productFamilyId: "servicios"}]` |
| ❌ | ✅ | ❌ | ❌ | `[{productFamilyId: "luz"}]` |
| ❌ | ✅ | ❌ | ✅ | `[{productFamilyId: "luz"}, {productFamilyId: "servicios"}]` |
| ❌ | ❌ | ✅ | ❌ | `[{productFamilyId: "luz"}, {productFamilyId: "gas"}]` |
| ❌ | ❌ | ✅ | ✅ | `[{productFamilyId: "luz"}, {productFamilyId: "gas"}, {productFamilyId: "servicios"}]` |
| ✅ | ✅ | ❌ | ❌ | `[{productFamilyId: "gas"}, {productFamilyId: "luz"}]` |
| ✅ | ✅ | ❌ | ✅ | `[{productFamilyId: "gas"}, {productFamilyId: "luz"}, {productFamilyId: "servicios"}]` |
| ❌ | ❌ | ❌ | ✅ | `[{productFamilyId: "servicios"}]` |
| ❌ | ❌ | ❌ | ❌ | `[]` (sin familias) |

### Proceso de Cálculo

1. **Análisis por Factura:** Para cada número de factura único
2. **Búsqueda de Subtipos:** Busca en todas las líneas de esa factura
3. **Detección de Palabras Clave:** Busca "GAS", "ELECTRICIDAD", "DUAL", "SERVICIOS" en `des_subtipo_factura`
4. **Aplicación de Reglas:** Aplica la lógica de combinaciones
5. **Resultado:** Array de familias de productos

### Ejemplos de Subtipos

- `"FACTURA GAS NATURAL"` → `[{productFamilyId: "gas"}]`
- `"FACTURA ELECTRICIDAD"` → `[{productFamilyId: "luz"}]`
- `"FACTURA DUAL GAS Y ELECTRICIDAD"` → `[{productFamilyId: "luz"}, {productFamilyId: "gas"}]`
- `"SERVICIOS ADICIONALES"` → `[{productFamilyId: "servicios"}]`

---

## 🔒 Seguridad y Controles

### ✅ Controles Implementados

**Autenticación y Autorización:**
- ✅ **API Gateway:** Autenticación OAuth/API Key implementada
- ✅ **Autorización:** Solo APIs internas con permisos específicos

**Protección de Datos:**
- ✅ **Filtro por NIF:** Solo facturas del cliente autenticado
- ✅ **Ventana Temporal:** Últimos 5 años (GDPR compliance)
- ✅ **Usuario de BD:** `mulesoft_ro`
- ✅ **Deduplicación:** `distinctBy` elimina facturas duplicadas

**Performance:**
- ✅ **Fetch Size:** Configurable por ambiente
- ✅ **Connection Pooling:** Máximo 30 conexiones
- ✅ **Índices de BD:** Optimización en CUPS, NIF, fecha

---

## ⚙️ Configuración de Producción

### Tabla de Base de Datos

```yaml
# Tabla principal
tabla: dlakeservi.self_publication_sistemas.facturas_ni_omega

# Campos clave
- cod_cups: Código CUPS
- cod_documento_identidad: NIF del cliente
- fec_emision_factura: Fecha de emisión
- cod_num_factura: Número de factura
- des_subtipo_factura: Subtipo (para cálculo de familias)
```

### Fetch Size

```yaml
# properties-pro.yaml
postgresql:
  fetchSize: "100"  # Optimizado para producción
```

---

## 📊 Logging

### Logs Generados

```
INFO  - invoices - Inicio ejecucion query: ES0021000000000001JN y 12345678A
DEBUG - Main Query: SELECT DISTINCT * FROM ...
DEBUG - invoices - Resultado: [{"cod_num_factura":"123456789",...}]
INFO  - invoices - Fin ejecucion
```

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-napai.raml`
- **Main Flow:** `src/main/mule/invoices/invoices.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`
- **Database Config:** `src/main/mule/global.xml`

---

## 🔗 Endpoints Relacionados

- **POST /ejecutarQuery** - Consultas dinámicas
- **POST /ejecutarSecureQuery** - Consultas parametrizadas
- **GET /technical_info_e** - Datos técnicos de electricidad
- **GET /technical_info_g** - Datos técnicos de gas

---

## 📌 Notas Importantes

### Características Especiales

**Múltiples CUPS:**
- ✅ Soporta consulta de múltiples puntos de suministro en una sola llamada
- ✅ Separación por comas en el parámetro `cups`
- ✅ Útil para clientes con múltiples contratos

**Cálculo de Familias:**
- ✅ Lógica compleja implementada en DataWeave
- ✅ Análisis de todas las líneas de factura
- ✅ Combinaciones automáticas de productos

**Cumplimiento GDPR:**
- ✅ Ventana de 5 años para retención de datos
- ✅ Filtro automático por fecha de emisión

### Recomendaciones de Uso

**Mejores Prácticas:**
- ✅ Limitar el número de CUPS por consulta (máximo recomendado: 10)
- ✅ Cachear resultados en el cliente para reducir llamadas
- ✅ Usar paginación si se esperan muchas facturas

**Performance:**
- ⚠️ Queries con muchos CUPS pueden ser lentas
- ⚠️ La lógica de cálculo de familias se ejecuta en memoria
- ✅ Optimizado con `distinctBy` para evitar duplicados

---

**Última actualización:** 2026-05-21  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy