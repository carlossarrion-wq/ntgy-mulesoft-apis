---
api: "ntgy-fuv-s-postgresql"
sidebar_position: 18
sidebar_label: "GET /execReprocesoPedidos"
description: "Ejecuta reprocesamiento de pedidos mediante stored procedures condicionales según sistema destino (SIEBEL/NewCo)"
keywords: [reprocesamiento, pedidos, stored procedure, PostgreSQL, SIEBEL, NewCo, batch]
tags: [Batch Operations, System API, PostgreSQL, Reprocessing]
sql_metadata:
  data_sources:
    - database: "PostgreSQL"
      schema: "public"
      type: "PostgreSQL"
      tables:
        - name: "procedures.pedidosSiebel"
          operations: ["EXECUTE"]
          description: "Stored procedure para reprocesamiento de pedidos SIEBEL"
        - name: "procedures.pedidosNewco"
          operations: ["EXECUTE"]
          description: "Stored procedure para reprocesamiento de pedidos NewCo"
---

# GET /execReprocesoPedidos

## 📌 Descripción General

Ejecuta el reprocesamiento de pedidos mediante stored procedures de PostgreSQL. El endpoint tiene **lógica condicional** que ejecuta diferentes stored procedures según el sistema destino especificado.

**Tipo de API:** System API  
**Método HTTP:** GET  
**Ruta:** `/execReprocesoPedidos`  
**Backend:** PostgreSQL Database (Stored Procedures)

---

## 🔧 Especificación Técnica

### Método HTTP

```
GET /api/execReprocesoPedidos
```

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Default | Descripción |
|-----------|------|-----------|---------|-------------|
| `destino` | string | ❌ No | "SIEBEL" | Sistema destino: "SIEBEL" o cualquier otro valor para NewCo |
| `idProceso` | number | ❌ No | 0 | ID del proceso/lote a reprocesar |

**⚠️ Nota Importante**: Aunque los parámetros son opcionales, el parámetro `destino` determina qué stored procedure se ejecuta.

### Respuestas

#### Respuesta Exitosa (200 OK)

```json
{
  "idnewlote": 12345
}
```

**Valores por Defecto:**
- Si `destino = "SIEBEL"` y no hay resultado: `{"idnewlote": 0}`
- Si `destino != "SIEBEL"` y no hay resultado: `{"idnewlote": -1}`

#### Respuesta de Error (200 OK)

```json
{
  "idnewlote": 0
}
```

**Nota:** En caso de error, el endpoint retorna 200 OK con:
- `{"idnewlote": 0}` si destino = "SIEBEL"
- `{"idnewlote": -1}` si destino != "SIEBEL"

---

## 🏗️ Implementación Técnica

### Lógica Condicional

El endpoint ejecuta **diferentes stored procedures** según el parámetro `destino`:

```dataweave
if (vars.destino == "SIEBEL")
  p('procedures.pedidosSiebel')
else
  p('procedures.pedidosNewco')
```

**Flujo SIEBEL** (destino = "SIEBEL"):
- Ejecuta: `procedures.pedidosSiebel`
- Parámetro: `idNewLote` (⚠️ ver nota sobre bug potencial)
- Default success: `{"idnewlote": 0}`
- Default error: `{"idnewlote": 0}`

**Flujo NewCo** (destino != "SIEBEL"):
- Ejecuta: `procedures.pedidosNewco`
- Parámetro: `idNewLote` (⚠️ ver nota sobre bug potencial)
- Default success: `{"idnewlote": -1}`
- Default error: `{"idnewlote": -1}`

### Stored Procedure Call

```xml
<db:stored-procedure config-ref="Database_Config">
    <db:sql>#[vars.Procedure]</db:sql>
    <db:input-parameters>#[{
        'idNewLote' : vars.idNewLote
    }]</db:input-parameters>
</db:stored-procedure>
```

**⚠️ Posible Bug en el Código:**

El código usa `vars.idNewLote` como parámetro de la stored procedure, pero esta variable **no se define en ninguna parte del flow**. 

Probablemente debería ser:
```dataweave
'idNewLote' : vars.idProceso
```

Esto podría causar que el parámetro siempre sea `null` o genere un error.

### Transformación de Response

```dataweave
%dw 2.0
output application/json
---
if (vars.destino == "SIEBEL")
   payload.resultSet1[0] default { "idnewlote": 0}
else
   payload.resultSet1[0] default { "idnewlote": -1}
```

**Características:**
- Extrae el primer registro del `resultSet1`
- Valores por defecto diferentes según destino
- Estructura minimalista: solo retorna `idnewlote`

### Error Handler

```dataweave
if (vars.destino == "SIEBEL")
  { "idnewlote": 0}
else
  { "idnewlote": -1}
```

**Nota:** El error handler retorna la misma estructura que el caso de éxito sin resultados, haciendo difícil distinguir entre éxito sin datos y error.

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Reprocesamiento SIEBEL

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/execReprocesoPedidos?destino=SIEBEL&idProceso=123" \
  -H "Accept: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET"
```

**Response (200 OK):**
```json
{
  "idnewlote": 12345
}
```

### Ejemplo 2: Reprocesamiento NewCo

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/execReprocesoPedidos?destino=NEWCO&idProceso=456" \
  -H "Accept: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET"
```

**Response (200 OK):**
```json
{
  "idnewlote": 67890
}
```

### Ejemplo 3: Sin Parámetros (Default SIEBEL)

```bash
curl -X GET "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/execReprocesoPedidos" \
  -H "Accept: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET"
```

**Response (200 OK):**
```json
{
  "idnewlote": 0
}
```

**Nota:** Sin parámetros, usa destino="SIEBEL" por defecto y retorna 0 si no hay resultados.

---

## ⚠️ Limitaciones y Consideraciones

### 1. **Bug Potencial en el Código**

El código usa `vars.idNewLote` que no está definida. Esto podría causar:
- Parámetro null en la stored procedure
- Error de ejecución
- Comportamiento inesperado

**Solución sugerida:** Cambiar `vars.idNewLote` por `vars.idProceso` en el código.

### 2. **Respuesta Minimalista**

El endpoint solo retorna `{idnewlote: number}`, sin:
- Mensaje descriptivo
- Número de pedidos procesados
- Detalles de la operación
- Distinción clara entre éxito y error

### 3. **Error Handling Ambiguo**

Los errores retornan 200 OK con valores por defecto, haciendo imposible distinguir:
- Éxito sin resultados vs Error
- Tipo de error ocurrido
- Detalles del problema

### 4. **Stored Procedures en Configuración**

Los nombres reales de las stored procedures están en archivos de configuración (properties), no en el código XML.

---

## 📊 Diferencias entre SIEBEL y NewCo

| Aspecto | SIEBEL (destino="SIEBEL") | NewCo (destino!="SIEBEL") |
|---------|---------------------------|---------------------------|
| **Stored Procedure** | `procedures.pedidosSiebel` | `procedures.pedidosNewco` |
| **Default Success** | `{"idnewlote": 0}` | `{"idnewlote": -1}` |
| **Default Error** | `{"idnewlote": 0}` | `{"idnewlote": -1}` |
| **Parámetro** | `idNewLote` (⚠️ bug) | `idNewLote` (⚠️ bug) |