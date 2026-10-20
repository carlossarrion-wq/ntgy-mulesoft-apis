---
api: "ntgy-fuv-s-postgresql"
sidebar_position: 14
sidebar_label: "GET /documentationControlByDocNumber"
description: "Consulta documentos de control NewCo por número de documento, estado y destino con JOIN a tabla de integración"
keywords: [documentationControl, docNumber, PostgreSQL, NewCo, System API, JOIN, JSONB]
tags: [Documentation Control, System API, PostgreSQL, NewCo, Query]
sql_metadata:
  data_sources:
    - database: "PostgreSQL"
      schema: "public"
      type: "PostgreSQL"
      tables:
        - name: "t_fuv_documentation_control"
          operations: ["READ"]
          description: "Tabla de control de documentación"
        - name: "t_fuv_integration_nw"
          operations: ["READ"]
          description: "Tabla de integración NewCo (JOIN para obtener nif_representante)"
---

# GET /documentationControlByDocNumber

## 📌 Descripción General

### Propósito del Endpoint

Consulta registros de control de documentación **NewCo** desde PostgreSQL mediante un INNER JOIN entre `t_fuv_documentation_control` y `t_fuv_integration_nw`. Este endpoint filtra por número de documento, estado y destino, retornando solo documentos de los últimos 3 años asociados a contratos NewCo.

**⚠️ IMPORTANTE**: Este endpoint **SOLO funciona para contratos NewCo** debido al INNER JOIN con `t_fuv_integration_nw`. No retorna documentos de contratos GNS.

El endpoint soporta:
- **Búsqueda por Documento + Estado**: Requiere número de documento Y estado del documento
- **Filtro por Destino**: Permite filtrar por sistema destino (default: SIEBEL)
- **Extracción de NIF Representante**: Obtiene el NIF del representante desde el mensaje JSONB de NewCo
- **Filtro Temporal**: Solo retorna documentos de los últimos 3 años

### Tipo de API

**Capa:** System API (API-Led Connectivity)

**Función:** Integración con PostgreSQL para consulta de documentación NewCo con información enriquecida del contrato.

---

## 🔧 Especificación Técnica

### Método HTTP

```
GET /api/documentationControlByDocNumber
```

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Default | Descripción |
|-----------|------|-----------|---------|-------------|
| `docNumber` | string | ✅ Sí | - | Número de documento del cliente (NIF/NIE/CIF) |
| `status` | string | ✅ Sí | - | Estado del documento (PENDIENTE, PROCESADO, ERROR, etc.) |
| `destino` | string | ❌ No | "SIEBEL" | Sistema destino del documento |

**⚠️ Nota**: Los 3 parámetros son necesarios para la query (destino tiene default).

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
      "NIF_REPRESENTANTE": "12345678A",
      "NUM_DOCUMENTO": "12345678A",
      "T_FUV_DOCUMENTATION_CONTROL_ID": 12345,
      "CONTRACT_ID": "ORD-2024-001234",
      "DOCUMENT_NAME": "contrato_principal.pdf",
      "DOCUMENT_STATUS": "PROCESADO",
      "ATTEMPTS_NUMBER": 1,
      "CREATE_DATE": "2024-06-15T10:30:00+02:00",
      "UPDATE_DATE": "2024-06-15T11:00:00+02:00",
      "UPDATE_USER": "mulesoft",
      "UPDATE_PROGRAM": "mulesoft",
      "CABECERA_CONTRATO": "Contrato de suministro",
      "TIPO_DOCUMENTAL": "CONTRATO",
      "ENVIO_DISTRIBUIDORA": true,
      "LINEAS_CONTRATO": "Líneas del contrato",
      "DESTINO": "SIEBEL",
      "DOCUMENT_ID": "doc-12345"
    }
  ]
}
```

**Nota sobre Validación y Manejo de Errores:**

- **Parámetros faltan**: APIKit valida RAML → 400 Bad Request
- **Parámetros vacíos**: Sin validación explícita → 500 Internal Server Error
- **Errores de BD**: Sin error handler → 500 Internal Server Error
- **Sin resultados**: 200 OK con `records: []`

---

## 🏗️ Implementación Técnica

### Query SQL Ejecutada

La query se obtiene desde `${queries.getDocumentationByDocNumber}`:

```sql
SELECT 
    COALESCE(inw.message->'contacto'->>'documento', inw.message->'contacto'->>'nif') as nif_representante,
    dc.* 
FROM t_fuv_documentation_control dc 
INNER JOIN t_fuv_integration_nw inw ON inw.contract_id = dc.contract_id 
WHERE dc.document_status = :status 
  AND dc.num_documento = :docNumber 
  AND dc.destino = :destino 
  AND dc.create_date >= (CURRENT_DATE - INTERVAL '3 years') 
ORDER BY dc.create_date DESC
```

**Características Críticas:**

1. **INNER JOIN con NewCo**: Solo retorna documentos de contratos que existen en `t_fuv_integration_nw`
2. **Extracción JSONB**: Obtiene `nif_representante` del campo `message` (JSONB) usando operadores JSON de PostgreSQL
3. **Filtro de fecha**: Solo últimos 3 años (`create_date >= CURRENT_DATE - INTERVAL '3 years'`)
4. **3 filtros obligatorios**: status, docNumber, destino

### Filtros Automáticos Aplicados

1. **Filtro de Fecha**: `create_date >= (CURRENT_DATE - INTERVAL '3 years')`
   - Solo retorna documentos de los últimos 3 años
   - Optimiza rendimiento y limita volumen de datos

2. **Filtro de Estado**: `document_status = :status`
   - Obligatorio, no tiene default
   - Permite filtrar por estado específico

3. **Filtro de Destino**: `destino = :destino`
   - Default: "SIEBEL"
   - Permite filtrar por sistema destino

4. **JOIN con NewCo**: `INNER JOIN t_fuv_integration_nw`
   - **CRÍTICO**: Solo retorna documentos de contratos NewCo
   - Excluye documentos de contratos GNS

### Transformación DataWeave

```dataweave
%dw 2.0
output application/json
---
payload map (record, findex) -> {
    "NIF_REPRESENTANTE": record.'nif_representante',  // Campo adicional del JOIN
    "NUM_DOCUMENTO": record.'num_documento',
    "T_FUV_DOCUMENTATION_CONTROL_ID": record.'t_fuv_documentation_control_id',
    "CONTRACT_ID": record.'contract_id',
    "DOCUMENT_NAME": record.'document_name',
    "DOCUMENT_STATUS": record.'document_status',
    "ATTEMPTS_NUMBER": record.'attempts_number',
    "CREATE_DATE": record.'create_date',
    "UPDATE_DATE": record.'update_date',
    "UPDATE_USER": record.'update_user',
    "UPDATE_PROGRAM": record.'update_program',
    "CABECERA_CONTRATO": record.'cabecera_contrato',
    "TIPO_DOCUMENTAL": record.'tipo_documental',
    "ENVIO_DISTRIBUIDORA": if(record.'envio_distribuidora' == 'Y') true else false,
    "LINEAS_CONTRATO": record.'lineas_contrato',
    "DOCUMENT_ID": record.document_id,
    "DESTINO": record.'destino'
}
```

**Campo Adicional**: `NIF_REPRESENTANTE` se extrae del mensaje JSONB del contrato NewCo.

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta con Todos los Parámetros

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/documentationControlByDocNumber?docNumber=12345678A&status=PROCESADO&destino=SIEBEL" \
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
      "NIF_REPRESENTANTE": "12345678A",
      "NUM_DOCUMENTO": "12345678A",
      "CONTRACT_ID": "ORD-2024-001234",
      "DOCUMENT_STATUS": "PROCESADO"
    }
  ]
}
```

### Ejemplo 2: Sin Resultados

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/documentationControlByDocNumber?docNumber=99999999Z&status=PROCESADO" \
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

---

## ⚠️ Limitaciones Importantes

1. **Solo Contratos NewCo**: Debido al INNER JOIN con `t_fuv_integration_nw`, solo retorna documentos de contratos NewCo
2. **Últimos 3 Años**: Solo retorna documentos creados en los últimos 3 años
3. **Requiere Estado**: El parámetro `status` es obligatorio (no tiene default)
4. **Dependencia de JSONB**: El campo `nif_representante` depende de la estructura del mensaje JSONB en NewCo

---

## 📊 Diferencias con Otros Endpoints

| Aspecto | GET /documentationControl | GET /documentationControlByDocNumber |
|---------|---------------------------|--------------------------------------|
| **Parámetros** | idPedido + nombreDocumento (opcional) | docNumber + status + destino (opcional) |
| **JOIN** | No tiene | INNER JOIN con t_fuv_integration_nw |
| **Alcance** | Todos los contratos | Solo contratos NewCo |
| **Campo adicional** | No | nif_representante (JSONB) |
| **Filtro temporal** | No | Últimos 3 años |