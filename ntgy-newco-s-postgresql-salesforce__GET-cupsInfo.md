---
api: "ntgy-newco-s-postgresql-salesforce"
sidebar_position: 6
sidebar_label: "GET /cupsInfo"
sql_metadata:
  data_sources:
    - database: "newcosalesforce"
      schema: "newcosalesforce"
      type: "PostgreSQL"
      tables:
        - name: "vlocity_cmt__premises__c"
          operations: ["READ"]
---

# Endpoint: GET /cupsInfo

## 📋 Descripción General

Recupera información del contrato activo asociado a un CUPS (Código Universal del Punto de Suministro) desde la base de datos PostgreSQL.

**Tipo de API:** System API  
**Capa:** System Layer (API-Led Connectivity)  
**Versión:** 1.42.0  
**Última actualización:** 2026-05-19

---

## 🎯 Propósito del Negocio

Este endpoint permite verificar si un CUPS tiene un contrato activo asociado y obtener el ID del cliente titular y el estado del contrato.

### Casos de Uso

1. **Validación de Contratos:** Verificar si un CUPS tiene contrato activo
2. **Identificación de Cliente:** Obtener el clientId asociado a un CUPS
3. **Estado de Contrato:** Consultar el estado del contrato del CUPS
4. **Portabilidad:** Verificar titular actual antes de cambio de comercializadora
5. **Alta de Servicio:** Comprobar si el CUPS ya tiene contrato activo

---

## 🔗 URL del Endpoint

```
GET /api/cupsInfo
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-salesforce-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-salesforce-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `cups` | string | Sí | Código CUPS a consultar | `ES0021000000000001JN` |

### Detalle del Parámetro

#### `cups` (Requerido)
- **Formato:** 20-22 caracteres alfanuméricos
- **Patrón:** ES + 4 dígitos (distribuidora) + 12 dígitos + 2 letras
- **Propósito:** Código Universal del Punto de Suministro español
- **Validación:** No puede ser null o vacío
- **Ejemplo:** `ES0021000000000001JN`

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

#### CUPS con Contrato Activo

```json
[{
  "clientId": "001aA000001234567",
  "contractStatus": "01"
}]
```

#### CUPS Sin Contrato o No Encontrado

```json
[{
  "clientId": "",
  "contractStatus": ""
}]
```

**⚠️ Importante:** 
- Siempre retorna HTTP 200
- Retorna un array con un único objeto
- Si `clientId` está vacío, significa que el CUPS no existe o no tiene contrato activo

### ❌ Errores Técnicos

#### 400 Bad Request

```json
{
  "errorCode": 400,
  "message": "Bad request",
  "description": "Invalid query parameter format"
}
```

#### 404 Not Found

```json
{
  "errorCode": 404,
  "message": "Resource not found",
  "description": "The requested endpoint does not exist"
}
```

#### 500 Internal Server Error

```json
{
  "errorCode": 500,
  "message": "DB:CONNECTIVITY",
  "description": "Could not establish connection to PostgreSQL database"
}
```

---

## 🔍 Estructura de Datos Detallada

### Array de Respuesta

La respuesta es un **array con un único objeto** que contiene:

| Campo | Tipo | Descripción | Ejemplo |
|-------|------|-------------|---------|
| `clientId` | string | ID de cuenta del cliente titular (Salesforce Account SFID). Vacío si no hay contrato | `"001aA000001234567"` o `""` |
| `contractStatus` | string | Estado del contrato. Vacío si no hay contrato | `"01"` (Activo), `"02"` (En proceso), o `""` |

### Códigos de Estado de Contrato

| Código | Descripción |
|--------|-------------|
| `"01"` | Contrato Activo |
| `"02"` | Contrato En Proceso |
| `""` | Sin contrato o CUPS no encontrado |

---

## 🔧 Implementación Técnica

### Query SQL Ejecutada

El endpoint ejecuta dos queries secuenciales:

#### 1. Query de Service Point (CUPS)

```sql
SELECT service.sfid
FROM newcosalesforce.vlocity_cmt__servicepoint__c as service
WHERE service.name IN (:cups_base, :cups_base + '0F', :cups_base + '1P')
```

**Lógica:**
- Extrae los primeros 20 caracteres del CUPS (`cups_base`)
- Busca variantes del CUPS (base, +0F, +1P) para cubrir diferentes formatos

#### 2. Query de Contrato y Datos Técnicos

```sql
SELECT 
    contract.accountid, 
    contract.status
FROM newcosalesforce.asset tableasset 
INNER JOIN newcosalesforce.vlocity_cmt__servicepoint__c as servicePoint 
    ON tableasset.vlocity_cmt__ServicePointId__c = servicePoint.sfid
INNER JOIN newcosalesforce.contract as contract 
    ON contract.sfid = tableasset.acn_fld_contract__c
WHERE
    tableasset.parentid IS NULL
    AND (tableasset.status IN ('02','03','04','06','07','09','10','11','12','16','18','20','22','23','24','25','26','27') 
         OR (tableasset.status = '08' AND tableasset.acn_fld_EndDateVersion__c IS NULL))
    AND contract.status IN ('01','02')
    AND contract.acn_fld_cups__c IN (:servicePointIds)
```

**Filtros aplicados:**
- Solo assets principales (sin parent)
- Estados de asset válidos (activos, en proceso, etc.)
- Contratos activos o en proceso (status 01, 02)
- CUPS asociados encontrados en la primera query

### Transformación DataWeave

#### Cuando NO encuentra CUPS (líneas 32-38):

```dataweave
%dw 2.0
output application/json
---
[{
  "clientId": "",
  "contractStatus": ""
}]
```

#### Cuando encuentra CUPS con contrato (líneas 72-78):

```dataweave
%dw 2.0
output application/json
---
[{
  "clientId": vars.mainResult.accountid,
  "contractStatus": vars.mainResult.status
}]
```

### Flujo de Ejecución

1. **Validar CUPS:** Extraer primeros 20 caracteres del CUPS
2. **Buscar Service Point:** Query con variantes del CUPS (base, +0F, +1P)
3. **Si no existe Service Point:** Retornar array con campos vacíos
4. **Si existe Service Point:** Buscar contrato activo asociado
5. **Transformar respuesta:** Mapear `accountid` y `status` del contrato
6. **Retornar:** Array con `clientId` y `contractStatus`

---

## 📊 Performance

### Métricas Esperadas

| Métrica | Valor Objetivo |
|---------|----------------|
| Tiempo de respuesta (p50) | < 300ms |
| Tiempo de respuesta (p95) | < 500ms |
| Throughput | 1000+ req/hora |
| Error rate | < 1% |

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: CUPS con contrato activo

```bash
curl -X GET "http://localhost:8081/api/cupsInfo?cups=ES0021000000000001JN" \
  -H "Content-Type: application/json"
```

**Respuesta:**
```json
[{
  "clientId": "001aA000001234567",
  "contractStatus": "01"
}]
```

### Ejemplo 2: CUPS sin contrato o no encontrado

```bash
curl -X GET "http://localhost:8081/api/cupsInfo?cups=ES9999999999999999XX" \
  -H "Content-Type: application/json"
```

**Respuesta:**
```json
[{
  "clientId": "",
  "contractStatus": ""
}]
```

---

## 🐛 Troubleshooting

### Problema: Respuesta con campos vacíos

**Respuesta:**
```json
[{
  "clientId": "",
  "contractStatus": ""
}]
```

**Causas posibles:**
- El CUPS no está registrado en el sistema
- El CUPS existe pero no tiene contratos activos (status 01 o 02)
- El CUPS tiene contratos pero los assets no están en estados válidos
- Error de tipeo en el código CUPS

**Solución:**
1. Verificar el formato del CUPS (20-22 caracteres)
2. Usar `/cupsExistence` para verificar si el CUPS existe en la base de datos
3. Consultar directamente la tabla `vlocity_cmt__ServicePoint__c`
4. Verificar que existan contratos con `status IN ('01','02')`
5. Verificar que los assets asociados estén en estados válidos

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-salesforce.raml`
- **Flow Implementation:** `src/main/mule/cups/getCUPS.xml`
- **Main Flow:** `src/main/mule/s-postgresql-salesforce.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`

---

**Última actualización:** 2026-05-19  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy