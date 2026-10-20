---
api: "ntgy-newco-s-postgresql-salesforce"
sidebar_position: 5
sidebar_label: "GET /cupsExistence"
sql_metadata:
  data_sources:
    - database: "newcosalesforce"
      schema: "newcosalesforce"
      type: "PostgreSQL"
      tables:
        - name: "vlocity_cmt__premises__c"
          operations: ["READ"]
---

# Endpoint: GET /cupsExistence

## 📋 Descripción General

Verifica si un CUPS (Código Universal del Punto de Suministro) existe en la base de datos PostgreSQL, retornando un indicador booleano de existencia.

**Tipo de API:** System API  
**Capa:** System Layer (API-Led Connectivity)  
**Versión:** 1.42.0  
**Última actualización:** 2026-06-18

---

## 🎯 Propósito del Negocio

Este endpoint proporciona validación rápida de la existencia de un CUPS en el sistema, permitiendo verificar puntos de suministro antes de crear contratos, pedidos o realizar otras operaciones.

### Casos de Uso

1. **Validación de Contratos:** Verificar CUPS antes de crear un nuevo contrato
2. **Validación de Pedidos:** Comprobar existencia del punto de suministro
3. **Portabilidad:** Validar CUPS en procesos de cambio de comercializadora
4. **Alta de Servicio:** Verificar que el CUPS esté registrado
5. **Integración con Sistemas Externos:** Validación de CUPS desde otros sistemas

---

## 🔗 URL del Endpoint

```
GET /api/cupsExistence
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-salesforce-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-salesforce-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `cups` | string | Sí | Código CUPS a verificar | `ES0021000000000001JN` |

### Detalle del Parámetro

#### `cups` (Requerido)
- **Formato:** 20-22 caracteres alfanuméricos
- **Patrón:** ES + 4 dígitos (distribuidora) + 12 dígitos + 2 letras
- **Propósito:** Código Universal del Punto de Suministro español
- **Validación:** No puede ser null o vacío
- **Ejemplo:** `ES0021000000000001JN`

### Validaciones

- **cups:** Longitud entre 20-22 caracteres
- **cups vacío:** Retorna error 400

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

#### CUPS Existe (`codResult: "0000"`)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "existingCUPS": true
}
```

#### CUPS No Existe (`codResult: "0001"`)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "No existe el CUPS"
  },
  "existingCUPS": false
}
```

**⚠️ Importante:** 
- Ambas respuestas retornan HTTP 200
- El campo `existingCUPS` SIEMPRE está presente en la respuesta
- Cuando el CUPS existe: `existingCUPS: true` y `codResult: "0000"`
- Cuando NO existe: `existingCUPS: false` y `codResult: "0001"`

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

### Objeto: `result`

| Campo | Tipo | Descripción | Valores Posibles |
|-------|------|-------------|------------------|
| `codResult` | string | Código de resultado | `"0000"` (existe), `"0001"` (no existe) |
| `msgResult` | string | Mensaje descriptivo | `"No hay error"`, `"CUPS Inexistente"` |

### Campo: `existingCUPS`

| Campo | Tipo | Descripción | Valores Posibles |
|-------|------|-------------|------------------|
| `existingCUPS` | boolean | Si el CUPS existe en la base de datos | `true` (existe), `false` (no existe) |

**Importante:** Este campo SIEMPRE está presente en la respuesta

---

## 🔧 Implementación Técnica

### Query SQL Ejecutada

```sql
SELECT 
  servicepoint.sfid as servicepointID,
  servicepoint.Name as servicepointName
FROM newcosalesforce.vlocity_cmt__ServicePoint__c as servicepoint
WHERE servicepoint.Name = :cups
```

**Lógica:**
- Si la query retorna resultados → `codResult: "0000"` y `existingCUPS: true`
- Si la query NO retorna resultados → `codResult: "0001"` y `existingCUPS: false`

### Transformación DataWeave

```dataweave
%dw 2.0
output application/json

var isCups = !isEmpty(vars.mainResult)
---
{
    "result": {
        "codResult": if(isCups) "0000" else "0001",
        "msgResult": if(isCups) "No hay error" else "No existe el CUPS"
    },
    "existingCUPS": isCups
}
```

**Nota:** El campo `existingCUPS` siempre está presente, con valor `true` o `false` según el resultado de la consulta.

---

## 📊 Performance

### Métricas Esperadas

| Métrica | Valor Objetivo |
|---------|----------------|
| Tiempo de respuesta (p50) | < 200ms |
| Tiempo de respuesta (p95) | < 400ms |
| Throughput | 2000+ req/hora |
| Error rate | < 1% |

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: CUPS existe

```bash
curl -X GET "http://localhost:8081/api/cupsExistence?cups=ES0021000000000001JN" \
  -H "Content-Type: application/json"
```

**Respuesta:**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "existingCUPS": true
}
```

### Ejemplo 2: CUPS no existe

```bash
curl -X GET "http://localhost:8081/api/cupsExistence?cups=ES9999999999999999XX" \
  -H "Content-Type: application/json"
```

**Respuesta:**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "No existe el CUPS"
  },
  "existingCUPS": false
}
```

### Ejemplo 3: CUPS con formato inválido

```bash
curl -X GET "http://localhost:8081/api/cupsExistence?cups=INVALID" \
  -H "Content-Type: application/json"
```

**Respuesta (400):**
```json
{
  "errorCode": 400,
  "message": "Bad request",
  "description": "Invalid query parameter format"
}
```

---

## 🐛 Troubleshooting

### Problema: "CUPS Inexistente" (codResult: 0001)

**Causas posibles:**
- El CUPS no está registrado en el sistema
- Error de tipeo en el código CUPS
- CUPS de otra distribuidora no sincronizado

**Solución:**
1. Verificar el formato del CUPS (20-22 caracteres)
2. Consultar directamente la tabla `vlocity_cmt__ServicePoint__c`
3. Verificar que el CUPS esté dado de alta en Salesforce
4. Comprobar sincronización con sistemas de distribuidora

### Problema: Respuesta lenta

**Causas posibles:**
- Índice faltante en columna `Name` de ServicePoint
- Pool de conexiones saturado
- Alto volumen de consultas concurrentes

**Solución:**
1. Verificar índices en base de datos
2. Revisar utilización del connection pool
3. Considerar caché para CUPS frecuentes

---

## 📝 Notas Adicionales

### Diferencia con `/cupsInfo`

| Aspecto | `/cupsExistence` | `/cupsInfo` |
|---------|------------------|-------------|
| **Propósito** | Validar existencia | Obtener información detallada |
| **Respuesta** | Boolean simple | Objeto completo con datos técnicos |
| **Uso** | Validación rápida | Consulta de especificaciones |
| **Performance** | Más rápido | Más lento (más datos) |

### Formato CUPS Español

El CUPS español tiene la siguiente estructura:
- **Posiciones 1-2:** "ES" (código país)
- **Posiciones 3-6:** Código distribuidora (4 dígitos)
- **Posiciones 7-18:** Identificador único (12 dígitos)
- **Posiciones 19-20:** Dígitos de control (2 letras)

**Ejemplo:** `ES0021000000000001JN`
- ES: España
- 0021: UFD Distribución
- 000000000001: Identificador
- JN: Control

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-salesforce.raml`
- **Flow Implementation:** `src/main/mule/cups/getCUPS.xml`
- **Main Flow:** `src/main/mule/s-postgresql-salesforce.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`

---

**Última actualización:** 2026-06-18  
**Versión del documento:** 1.1  
**Autor:** Equipo MuleSoft Naturgy  
**Cambios v1.1:** Corrección del nombre del campo de respuesta de `cupsExistence` a `existingCUPS` para coincidir con la implementación real
