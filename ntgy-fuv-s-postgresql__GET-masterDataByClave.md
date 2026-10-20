---
api: "ntgy-fuv-s-postgresql"
sidebar_position: 21
sidebar_label: "GET /masterDataByClave"
description: "Consulta un registro específico de datos maestros por master y clave con soporte de caché"
keywords: [master data, clave, valor, caché, PostgreSQL, configuración]
tags: [Master Data, PostgreSQL, System API, Cache, Query]
sql_metadata:
  data_sources:
    - database: "PostgreSQL"
      schema: "public"
      type: "PostgreSQL"
      tables:
        - name: "t_fuv_masterdata"
          operations: ["READ"]
          description: "Tabla de datos maestros clave-valor con soporte de caché"
---

# GET /masterDataByClave

## 📌 Descripción General

Consulta un registro específico de datos maestros desde la tabla `t_fuv_masterdata` en PostgreSQL, filtrando por categoría (master) y clave específica. Incluye soporte de caché opcional para mejorar el rendimiento.

**Tipo de API:** System API  
**Método HTTP:** GET  
**Ruta:** `/masterDataByClave`  
**Backend:** PostgreSQL Database (t_fuv_masterdata)

---

## 🔧 Especificación Técnica

### Método HTTP

```
GET /api/masterDataByClave
```

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Default | Descripción |
|-----------|------|-----------|---------|-------------|
| `master` | string | ✅ Sí | - | Categoría de master data | 
| `clave` | string | ✅ Sí | - | Clave específica a consultar |
| `cache` | boolean | ❌ No | `true` | Si se debe usar caché (true) o consulta directa (false) |

### Respuestas

#### Respuesta Exitosa - Registro Encontrado (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "record": {
    "clave": "DNI",
    "valor": "Documento Nacional de Identidad"
  }
}
```

#### Respuesta Exitosa - Registro NO Encontrado (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "record": {
    "clave": "CLAVE_INEXISTENTE",
    "valor": "VALOR PARA CLAVE NO LOCALIZADO"
  }
}
```

**⚠️ Nota Importante:** Cuando no se encuentra el registro, el endpoint retorna **200 OK** (no 404) con el mensaje "VALOR PARA CLAVE NO LOCALIZADO" en el campo `valor`.

**Estructura de Campos:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | string | Código de resultado: "0000" = éxito |
| `result.msgResult` | string | Mensaje del resultado |
| `record.clave` | string | Clave del registro consultado |
| `record.valor` | string | Valor asociado o "VALOR PARA CLAVE NO LOCALIZADO" si no existe |

**Nota sobre Validación y Manejo de Errores:**

- **Parámetros faltan**: APIKit valida RAML → 400 Bad Request
- **Parámetros vacíos**: Sin validación explícita → 500 Internal Server Error
- **Registro no encontrado**: 200 OK con mensaje especial (NO retorna 404)
- **Errores de BD**: Sin error handler → 500 Internal Server Error

---

## 🏗️ Implementación Técnica

### Lógica de Caché

El endpoint implementa una lógica condicional basada en el parámetro `cache`:

```xml
<choice>
    <when expression="#[vars.cache]">
        <ee:cache cachingStrategy-ref="Caching_Strategy_MasterData">
            <db:select>...</db:select>
        </ee:cache>
    </when>
    <otherwise>
        <db:select>...</db:select>
    </otherwise>
</choice>
```

**Flujo con Caché (cache=true, default):**
1. Busca en caché (Object Store)
2. Si existe (cache hit): Retorna desde caché
3. Si no existe (cache miss): Consulta BD y guarda en caché

**Flujo sin Caché (cache=false):**
1. Consulta directa a base de datos
2. No guarda en caché

### Query SQL Ejecutada

La query se obtiene desde `${queries.getMasterDataByClave}`:

```sql
SELECT clave, valor 
FROM t_fuv_masterdata 
WHERE master = :master 
  AND clave = :clave
```

**Características:**
- **Operación:** `db:select` - Retorna array (puede estar vacío)
- **Parámetros:** `:master` y `:clave`
- **Tabla:** `t_fuv_masterdata`
- **Campos:** Solo 2 campos (clave, valor)
- **Sin LIMIT:** Retorna todos los registros que coincidan

### Transformación DataWeave

```dataweave
%dw 2.0
output application/json
---
if (vars.recordSelected == null)
{
    "result": {
        "codResult": "0000",
        "msgResult": "No hay error"
    },
    record: {
        clave: vars.clave,
        valor: "VALOR PARA CLAVE NO LOCALIZADO"
    }	
}
else
{
    "result": {
        "codResult": "0000",
        "msgResult": "No hay error"
    },
    record: {
        clave: vars.recordSelected.clave,
        valor: vars.recordSelected.valor
    }
}
```

**Características:**
- Extrae el primer registro: `payload[0]`
- Si es null: Retorna mensaje especial
- Si existe: Retorna clave y valor reales
- Siempre retorna 200 OK

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta con Caché (Default)

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/masterDataByClave?master=TIPO_DOCUMENTO&clave=DNI" \
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
  "record": {
    "clave": "DNI",
    "valor": "Documento Nacional de Identidad"
  }
}
```

### Ejemplo 2: Consulta sin Caché

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/masterDataByClave?master=TIPO_DOCUMENTO&clave=NIE&cache=false" \
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
  "record": {
    "clave": "NIE",
    "valor": "Número de Identidad de Extranjero"
  }
}
```

### Ejemplo 3: Registro No Encontrado

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/masterDataByClave?master=TIPO_DOCUMENTO&clave=PASAPORTE_DIPLOMATICO" \
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
  "record": {
    "clave": "PASAPORTE_DIPLOMATICO",
    "valor": "VALOR PARA CLAVE NO LOCALIZADO"
  }
}
```

**Nota:** Retorna 200 OK, NO 404. El campo `valor` contiene el mensaje especial.

### Ejemplo 4: Error - Parámetros Faltantes

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/masterDataByClave?master=TIPO_DOCUMENTO" \
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

## 🔄 Diferencias con GET /masterData

| Aspecto | GET /masterData | GET /masterDataByClave |
|---------|-----------------|------------------------|
| **Filtro** | Solo master | master + clave |
| **Retorna** | Array de registros | Un solo registro |
| **Response** | `{result, records: [...]}` | `{result, record: {...}}` |
| **Caché** | No | Sí (opcional) |
| **No encontrado** | `records: []` | `valor: "VALOR PARA CLAVE NO LOCALIZADO"` |

---

## 💡 Casos de Uso

1. **Validación de Valores**
   - Verificar si una clave específica existe
   - Obtener descripción de un código
   - Validar valores antes de procesamiento

2. **Consulta con Caché**
   - Mejorar rendimiento para consultas frecuentes
   - Reducir carga en base de datos
   - Respuestas más rápidas

3. **Configuración Dinámica**
   - Obtener parámetros de configuración específicos
   - Consultar valores de catálogos
   - Acceso directo a datos maestros

---

## ⚠️ Consideraciones Importantes

### 1. **Comportamiento de "No Encontrado"**

A diferencia de endpoints estándar, este endpoint **NO retorna 404** cuando no encuentra el registro. En su lugar:
- Retorna 200 OK
- Campo `valor` contiene: "VALOR PARA CLAVE NO LOCALIZADO"
- Campo `clave` contiene la clave buscada

### 2. **Caché por Defecto**

El parámetro `cache` es `true` por defecto. Para forzar consulta directa a BD, usar `cache=false`.

### 3. **Estructura Simple**

Solo retorna 2 campos: `clave` y `valor`. No incluye metadatos adicionales.