---
api: "ntgy-fuv-s-postgresql"
sidebar_position: 11
sidebar_label: "GET /contractControlByState"
description: "Consulta registros de control de contratos GNS por estado con filtros automáticos de fecha y reintentos para reprocesamiento"
keywords: [contractControl, GNS, PostgreSQL, estado, filtros, reprocesamiento, System API]
tags: [Contract Control, Query, System API, PostgreSQL, GNS]
sql_metadata:
  data_sources:
    - database: "PostgreSQL"
      schema: "public"
      type: "PostgreSQL"
      tables:
        - name: "t_fuv_integration"
          operations: ["READ"]
          description: "Tabla de control de contratos GNS con filtros automáticos de fecha y reintentos"
---

# GET /contractControlByState

## 📌 Descripción General

Consulta registros de control de contratos GNS desde la tabla `t_fuv_integration` en PostgreSQL, filtrando por estado. Este endpoint permite recuperar contratos que requieren reprocesamiento, aplicando filtros automáticos de fecha y número de intentos para optimizar el procesamiento.

**Tipo de API:** System API  
**Método HTTP:** GET  
**Ruta:** `/contractControlByState`  
**Backend:** PostgreSQL Database (FUV - Facturación Única Virtual)

---

## 💼 Propósito del Negocio

### Objetivo
Facilitar la consulta de contratos GNS que se encuentran en un estado específico para su procesamiento o reprocesamiento. Este endpoint es fundamental para los procesos de recuperación de errores y reprocesamiento de contratos.

### Casos de Uso
1. **Reprocesamiento de Contratos en Error**
   - Consultar contratos en estado "ERROR" para reintento de procesamiento
   - Limitar a contratos con menos de 4 intentos para evitar loops infinitos
   - Priorizar contratos de los últimos 10 días

2. **Monitoreo de Contratos Pendientes**
   - Identificar contratos en estado "PENDIENTE" esperando procesamiento
   - Obtener contratos recientes para procesamiento prioritario
   - Generar reportes de contratos pendientes

3. **Auditoría y Seguimiento**
   - Consultar histórico de contratos por estado
   - Verificar contratos en proceso
   - Analizar patrones de error para mejoras

### Valor de Negocio
- **Recuperación Automática:** Permite identificar y reprocesar contratos fallidos
- **Optimización de Recursos:** Filtra contratos recientes y con intentos limitados
- **Visibilidad Operacional:** Facilita monitoreo del estado de contratos GNS

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/contractControlByState?estado={estado}` |
| **Development** | `https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io` | `https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/contractControlByState?estado={estado}` |
| **UAT** | `https://ntgy-fuv-s-postgresql-uat.us-e2.cloudhub.io` | `https://ntgy-fuv-s-postgresql-uat.us-e2.cloudhub.io/api/contractControlByState?estado={estado}` |
| **Production** | `https://ntgy-fuv-s-postgresql-pro.us-e2.cloudhub.io` | `https://ntgy-fuv-s-postgresql-pro.us-e2.cloudhub.io/api/contractControlByState?estado={estado}` |

### Método HTTP

```
GET /api/contractControlByState
```

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `estado` | string | ✅ Sí | Estado del contrato a filtrar. Ejemplos: "received", "ERROR", "PENDIENTE" | `"received"` |

### Respuestas

#### Respuesta Exitosa (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "records": [
    {
      "T_FUV_INTEGRATION_ID": 12345,
      "CONTRACT_ID": "ORD-2024-001234",
      "SYSTEM_MESSAGE": "Frontal",
      "CODE": "received",
      "MESSAGE": "{\"orderId\":\"ORD-2024-001234\",\"cliente\":{\"nombre\":\"Juan Pérez\"}}",
      "STATE": "received",
      "RESPONSE": null,
      "ERROR": null,
      "CONTRACT_ATR": 0,
      "CREATE_DATE": "2024-06-10T10:30:00+02:00",
      "UPDATE_DATE": "2024-06-15T14:20:00+02:00",
      "UPDATE_USER": "mulesoft",
      "UPDATE_PROGRAM": "mulesoft",
      "OPTIMIST_LOCK": 1,
      "VIEWABLE": 1,
      "ATTEMPTS_NUMBER": 2,
      "ID_CLIENTE": "CLI-789",
      "ID_PS": "ES0021000000000001AA"
    }
  ]
}
```

#### Respuesta Sin Resultados (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "records": []
}
```

#### Respuesta de Error Técnico (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error interno del servidor"
  }
}
```

**Nota sobre Validación y Manejo de Errores:**

- **Parámetro `estado` falta completamente**: APIKit valida el RAML y retorna **400 Bad Request** automáticamente.
- **Parámetro presente pero vacío/inválido**: No hay validación explícita. La query SQL fallará y retornará **500 Internal Server Error**.
- **Errores de base de datos**: El flow no tiene error handler propio, por lo que errores de PostgreSQL se propagan como **500 Internal Server Error**.
- **Sin resultados**: Retorna **200 OK** con array vacío (comportamiento normal, no es error).

---

## 🏗️ Implementación Técnica

### Query SQL Ejecutada

La query se obtiene desde `${queries.getContractControlByState}`:

```sql
SELECT * 
FROM t_fuv_integration 
WHERE state = :status 
  AND update_date > CURRENT_DATE - 10 
  AND attempts_number < 4 
ORDER BY update_date ASC
```

**Características:**
- **Operación:** `db:select` - Retorna múltiples registros
- **Parámetro:** `:status` mapeado desde `vars.estado`
- **Tabla:** `t_fuv_integration` (GNS)
- **Filtros automáticos:**
  - `update_date > CURRENT_DATE - 10`: Solo últimos 10 días
  - `attempts_number < 4`: Menos de 4 intentos
- **Ordenamiento:** `ORDER BY update_date ASC` (más antiguos primero)

### Filtros Automáticos Aplicados

1. **Filtro de Fecha:** `update_date > CURRENT_DATE - 10`
   - Solo retorna registros actualizados en los últimos 10 días
   - Optimiza el rendimiento evitando consultar histórico completo
   - Prioriza contratos recientes para reprocesamiento

2. **Filtro de Intentos:** `attempts_number < 4`
   - Limita a contratos con menos de 4 intentos
   - Previene loops infinitos de reprocesamiento
   - Identifica contratos que aún pueden ser recuperados

3. **Ordenamiento:** `ORDER BY update_date ASC`
   - Procesa primero los contratos más antiguos
   - Garantiza procesamiento FIFO (First In, First Out)
   - Mejora la equidad en el procesamiento

### Transformación DataWeave

```dataweave
%dw 2.0
output application/java
---
payload map (record, findex) -> {
    "T_FUV_INTEGRATION_ID": record.'t_fuv_integration_id',
    "CONTRACT_ID": record.'contract_id',
    "SYSTEM_MESSAGE": record.'system_message',
    "CODE": record.'code',
    "MESSAGE": record.'message'.'value',  // Extrae valor de campo JSONB
    "STATE": record.'state',
    "RESPONSE": record.'response',
    "ERROR": record.'error',
    "CONTRACT_ATR": record.'contract_atr',
    "CREATE_DATE": record.'create_date',
    "UPDATE_DATE": record.'update_date',
    "UPDATE_USER": record.'update_user',
    "UPDATE_PROGRAM": record.'update_program',
    "OPTIMIST_LOCK": record.'optimist_lock',
    "VIEWABLE": record.'viewable',
    "ATTEMPTS_NUMBER": record.'attempts_number',
    "ID_CLIENTE": record.'id_cliente',
    "ID_PS": record.'id_ps'
}
```

**Nota Importante:** El campo `message` es de tipo JSONB en PostgreSQL, por lo que se accede con `.value` para extraer el contenido JSON como string.

### Response Structure

```dataweave
{
    "result": {
        "codResult": "0000",
        "msgResult": "No hay error"
    },
    "records": vars.recordsSelected
}
```

**Códigos de Resultado:**
- `"0000"`: Éxito - Consulta ejecutada correctamente (puede retornar array vacío)

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta de Contratos en Estado "received"

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/contractControlByState?estado=received" \
  -H "Accept: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET"
```

**Response (200 OK):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "records": [
    {
      "T_FUV_INTEGRATION_ID": 12345,
      "CONTRACT_ID": "ORD-2024-001234",
      "SYSTEM_MESSAGE": "Frontal",
      "CODE": "received",
      "MESSAGE": "{\"orderId\":\"ORD-2024-001234\"}",
      "STATE": "received",
      "ATTEMPTS_NUMBER": 0,
      "ID_CLIENTE": "CLI-789",
      "ID_PS": "ES0021000000000001AA"
    }
  ]
}
```

### Ejemplo 2: Consulta Sin Resultados

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/contractControlByState?estado=COMPLETADO" \
  -H "Accept: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET"
```

**Response (200 OK):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "records": []
}
```

### Ejemplo 3: Error - Parámetro Faltante

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/contractControlByState" \
  -H "Accept: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET"
```

**Response (400 Bad Request):**
```json
{
  "message": "Bad request"
}
```

---

## 📊 Diferencias con GET /contractControlByState_NW

Este endpoint es similar a `GET /contractControlByState_NW` pero con diferencias importantes:

| Aspecto | GET /contractControlByState (GNS) | GET /contractControlByState_NW (NewCo) |
|---------|-----------------------------------|----------------------------------------|
| **Tabla** | `t_fuv_integration` | `t_fuv_integration_nw` |
| **Origen** | Frontal de Ventas GNS | Frontal de Ventas NewCo / Portabilidad Masiva |
| **Parámetro idProceso** | ❌ NO existe | ✅ Existe (opcional, para filtrar por lote) |
| **Lógica Condicional** | ❌ NO tiene | ✅ Tiene (3 casos según idProceso) |
| **Campos Adicionales** | `ID_CLIENTE`, `ID_PS` | No tiene estos campos |
| **Filtros Automáticos** | ✅ Mismos (fecha + intentos) | ✅ Mismos (fecha + intentos) |

**Nota Importante:** La versión GNS NO soporta filtrado por lote (`idProceso`). Si necesitas esta funcionalidad, usa la versión NewCo.