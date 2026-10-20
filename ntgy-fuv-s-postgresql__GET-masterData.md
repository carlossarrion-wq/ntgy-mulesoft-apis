---
api: "ntgy-fuv-s-postgresql"
sidebar_position: 20
sidebar_label: "GET /masterData"
description: "Consulta datos maestros (clave-valor) desde PostgreSQL filtrando por categoría master"
keywords: [master data, configuración, PostgreSQL, clave-valor, catálogos, FUV]
tags: [Master Data, Configuration, System API, PostgreSQL, Read]
sql_metadata:
  data_sources:
    - database: "PostgreSQL"
      schema: "public"
      type: "PostgreSQL"
      tables:
        - name: "t_fuv_masterdata"
          operations: ["READ"]
          description: "Tabla de datos maestros clave-valor del sistema FUV"
---

# GET /masterData

## 📌 Descripción General

Consulta datos maestros de configuración desde la tabla `t_fuv_masterdata` en PostgreSQL. Retorna pares clave-valor para una categoría específica de master data.

**Tipo de API:** System API  
**Método HTTP:** GET  
**Ruta:** `/masterData`  
**Backend:** PostgreSQL Database (t_fuv_masterdata)

---

## 🔧 Especificación Técnica

### Método HTTP

```
GET /api/masterData
```

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `master` | string | ✅ Sí | Categoría de master data a consultar | `"TIPO_DOCUMENTO"` |

**Valores Comunes:**
- `TIPO_DOCUMENTO` - Tipos de documento
- `ESTADO_PEDIDO` - Estados de pedidos
- `TIPO_PRODUCTO` - Tipos de productos
- Otros según configuración del sistema

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
      "clave": "DNI",
      "valor": "Documento Nacional de Identidad"
    },
    {
      "clave": "NIE",
      "valor": "Número de Identidad de Extranjero"
    },
    {
      "clave": "CIF",
      "valor": "Código de Identificación Fiscal"
    }
  ]
}
```

**Estructura de Campos:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | string | Código de resultado: "0000" = éxito |
| `result.msgResult` | string | Mensaje del resultado |
| `records` | array | Array de pares clave-valor |
| `records[].clave` | string | Clave del registro (código o identificador) |
| `records[].valor` | string | Valor asociado a la clave (descripción) |

**Nota sobre Validación y Manejo de Errores:**

- **Parámetro `master` falta**: APIKit valida RAML → 400 Bad Request
- **Parámetro vacío**: Sin validación explícita → 500 Internal Server Error
- **Errores de BD**: Sin error handler → 500 Internal Server Error
- **Sin resultados**: 200 OK con `records: []`

---

## 🏗️ Implementación Técnica

### Query SQL Ejecutada

La query se obtiene desde `${queries.getMasterData}`:

```sql
SELECT clave, valor 
FROM t_fuv_masterdata 
WHERE master = :master 
ORDER BY clave, valor
```

**Características:**
- **Operación:** `db:select` - Retorna múltiples registros
- **Parámetro:** `:master` mapeado desde `vars.master`
- **Tabla:** `t_fuv_masterdata`
- **Campos:** Solo 2 campos (clave, valor)
- **Ordenamiento:** Por clave y valor (alfabético)

### Transformación DataWeave

```dataweave
%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0000",
        "msgResult": "No hay error"
    },
    records: vars.recordsSelected map ( record , indexOfRecord ) -> {
        clave: record.clave,
        valor: record.valor
    }
}
```

**Características:**
- Estructura con `result` y `records`
- Mapeo directo de campos de PostgreSQL
- Solo 2 campos por registro: clave y valor
- Código de resultado estándar (0000 = éxito)

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta de Tipos de Documento

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/masterData?master=TIPO_DOCUMENTO" \
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
      "clave": "CIF",
      "valor": "Código de Identificación Fiscal"
    },
    {
      "clave": "DNI",
      "valor": "Documento Nacional de Identidad"
    },
    {
      "clave": "NIE",
      "valor": "Número de Identidad de Extranjero"
    }
  ]
}
```

### Ejemplo 2: Sin Resultados

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/masterData?master=CATEGORIA_INEXISTENTE" \
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
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/masterData" \
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

**Nota:** APIKit valida el RAML antes de ejecutar el flow.

---

## 📊 Estructura de la Tabla

La tabla `t_fuv_masterdata` tiene una estructura simple de clave-valor:

```sql
CREATE TABLE t_fuv_masterdata (
    master VARCHAR,      -- Categoría de master data
    clave VARCHAR,       -- Clave del registro
    valor VARCHAR        -- Valor asociado
);
```

**Características:**
- Estructura simple: 3 campos (master, clave, valor)
- Sin metadatos de auditoría
- Sin campo active/inactive
- Ordenamiento por clave y valor

---

## 💡 Casos de Uso

1. **Catálogos de Selección**
   - Obtener listas de valores para formularios
   - Validar valores de entrada contra catálogos
   - Presentar opciones en interfaces de usuario

2. **Configuración del Sistema**
   - Consultar parámetros de configuración
   - Obtener valores por defecto
   - Sincronizar catálogos entre sistemas

3. **Validaciones de Negocio**
   - Verificar que valores sean válidos
   - Obtener descripciones de códigos
   - Traducir claves a valores legibles