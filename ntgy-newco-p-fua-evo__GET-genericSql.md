---
api: "ntgy-newco-p-fua-evo"
sidebar_position: 26
sidebar_label: "GET /genericSql"
description: "Ejecuta consultas SQL genéricas y seguras en la base de datos NAPAI mediante configuración centralizada en DynamoDB"
keywords: [genericSql, NAPAI, PostgreSQL, DynamoDB, consultas seguras, SQL parametrizado, Process API]
tags: [Consultas, SQL, NAPAI, DynamoDB, Process API, Orquestación]
endpoint_dependencies:
  - method: GET
    path: /item
    api_name: ntgy-fua-s-dynamodb
  - method: POST
    path: /ejecutarSecureQuery
    api_name: ntgy-newco-s-postgresql-napai
---

# GET /genericSql

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite ejecutar consultas SQL genéricas y seguras en la base de datos NAPAI (PostgreSQL) mediante un sistema de configuración centralizada. Las consultas están pre-configuradas en DynamoDB, lo que garantiza la seguridad y previene ataques de SQL injection.

### Tipo de API

**Capa:** Process API (API-Led Connectivity)

**Función:** Orquestación de consultas SQL seguras con configuración centralizada

### Patrón de Integración

**Tipo:** Orquestación con Transformación

**Descripción:** 
1. Busca la configuración de la query en DynamoDB usando el `queryId`
2. Ejecuta la query SQL parametrizada en PostgreSQL NAPAI
3. Retorna los resultados transformados

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    E[Experience APIs<br/>Portal FUA, Apps Móviles]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>GET /genericSql]
    S1[System API<br/>ntgy-fua-s-dynamo<br/>GET /item]
    S2[System API<br/>ntgy-newco-s-postgresql-napai<br/>POST /ejecutarSecureQuery]
    DB1[(DynamoDB<br/>Configuración Queries)]
    DB2[(PostgreSQL NAPAI<br/>Base de Datos)]
    
    E -->|Request con queryId| P
    P -->|1. Buscar configuración| S1
    S1 -->|Consulta| DB1
    DB1 -->|sqlCode| S1
    S1 -->|Configuración query| P
    P -->|2. Ejecutar query| S2
    S2 -->|Ejecuta SQL| DB2
    DB2 -->|Resultados| S2
    S2 -->|Datos| P
    P -->|Response| E
    
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:3px
    style S1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style DB1 fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style DB2 fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Consultas Dinámicas Seguras**
   - Permite ejecutar consultas SQL sin exponer la estructura de la base de datos
   - Previene SQL injection mediante queries pre-configuradas
   - Facilita el mantenimiento centralizado de consultas

2. **Acceso Flexible a Datos NAPAI**
   - Proporciona acceso a datos de NAPAI sin necesidad de crear endpoints específicos
   - Permite agregar nuevas consultas sin modificar código
   - Soporta consultas parametrizadas dinámicas

3. **Gestión Centralizada de Queries**
   - Todas las queries están almacenadas en DynamoDB
   - Facilita auditoría y control de acceso a datos
   - Permite actualizar queries sin redesplegar la aplicación

### 🔗 Endpoints Dependientes

Este Process API consume los siguientes System APIs:

| System API | Endpoint | Método | Propósito |
|------------|----------|--------|-----------|
| `ntgy-fua-s-dynamo` | `/item` | GET | Obtiene la configuración de la query SQL desde DynamoDB |
| `ntgy-newco-s-postgresql-napai` | `/ejecutarSecureQuery` | POST | Ejecuta la query SQL parametrizada en PostgreSQL NAPAI |

### Lógica de Orquestación

```mermaid
graph LR
    P[Process API<br/>genericSql] --> S1[System API<br/>s-dynamo<br/>GET /item]
    P --> S2[System API<br/>s-postgresql-napai<br/>POST /ejecutarSecureQuery]
    S1 --> D1[(DynamoDB<br/>Tabla Queries)]
    S2 --> D2[(PostgreSQL<br/>NAPAI)]
    
    style P fill:#ffe1f5
    style S1 fill:#e8f5e9
    style S2 fill:#e8f5e9
```

**Pasos del Proceso:**

1. **Paso 1 - Validación de Entrada**: Extrae el `queryId` de los query parameters
2. **Paso 2 - Búsqueda en DynamoDB**: Consulta DynamoDB para obtener el `sqlCode` asociado al `queryId`
3. **Paso 3 - Preparación de Parámetros**: Transforma los query parameters en parámetros SQL (reemplaza `#` por `,`)
4. **Paso 4 - Ejecución en NAPAI**: Ejecuta la query SQL parametrizada en PostgreSQL NAPAI
5. **Paso 5 - Transformación de Respuesta**: Formatea la respuesta con estructura estándar

### Transformaciones Aplicadas

| Campo Original | Campo Transformado | Transformación |
|----------------|-------------------|----------------|
| `attributes.queryParams.queryId` | `vars.token.queryId.S` | Formato DynamoDB (String type) |
| `attributes.queryParams.*` (excepto queryId) | `vars.query` | Reemplaza `#` por `,` para separadores |
| `vars.dynamoItem.item.sqlCode` | Query parameter `query` | Extrae SQL code de DynamoDB |
| `vars.postgresResult` | `payload.body` | Envuelve resultado en estructura estándar |

### Contexto Empresarial

- **Dominio:** Gestión de Datos y Consultas Genéricas
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Proceso de Negocio:** Acceso seguro y flexible a datos operacionales en NAPAI

### Stakeholders

- **Desarrolladores de Portal FUA**: Necesitan acceso flexible a datos sin crear endpoints específicos
- **Administradores de Base de Datos**: Controlan y auditan las queries ejecutadas
- **Equipos de Seguridad**: Garantizan que no haya SQL injection mediante configuración centralizada

### Valor de Negocio

- **Seguridad**: Previene SQL injection mediante queries pre-configuradas
- **Flexibilidad**: Permite agregar nuevas consultas sin modificar código
- **Mantenibilidad**: Centraliza la gestión de queries en DynamoDB
- **Auditoría**: Facilita el seguimiento de queries ejecutadas

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Portal web FUA
- Aplicaciones móviles de clientes
- Sistemas de agentes de call center
- Herramientas de reporting y análisis

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/genericSql?queryId=QUERY_ID` |
| **Development** | `https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io` | `https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/genericSql?queryId=QUERY_ID` |
| **UAT** | `https://ntgy-newco-p-fua-evo-uat.us-e2.cloudhub.io` | `https://ntgy-newco-p-fua-evo-uat.us-e2.cloudhub.io/api/genericSql?queryId=QUERY_ID` |
| **Production** | `https://ntgy-newco-p-fua-evo-pro.us-e2.cloudhub.io` | `https://ntgy-newco-p-fua-evo-pro.us-e2.cloudhub.io/api/genericSql?queryId=QUERY_ID` |

### Método HTTP

```
GET /api/genericSql
```

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `queryId` | string | ✅ Sí | Identificador único de la query configurada en DynamoDB | `"QUERY_CUSTOMERS_BY_REGION"` |
| `[param1]` | string | ❌ No | Parámetros dinámicos para la query SQL (nombres definidos en la configuración) | `"Madrid"` |
| `[param2]` | string | ❌ No | Parámetros adicionales según la query | `"2024"` |

**Nota sobre parámetros dinámicos:**
- Los parámetros adicionales dependen de la query configurada en DynamoDB
- Se pueden pasar múltiples parámetros separados por `#` que serán convertidos a `,`
- Ejemplo: `param=value1#value2#value3` → se convierte en `param=value1,value2,value3`

### Estructura de Request

**Ejemplo de URL con parámetros:**
```
GET /api/genericSql?queryId=QUERY_CUSTOMERS&region=Madrid&year=2024
```

### Respuestas

#### ✅ Respuesta Exitosa (200 OK)

```json
{
  "body": [
    {
      "column1": "value1",
      "column2": "value2",
      "column3": "value3"
    },
    {
      "column1": "value4",
      "column2": "value5",
      "column3": "value6"
    }
  ],
  "result": {
    "codResult": "0000",
    "msgResult": "Resultado correcto!"
  }
}
```

**Campos de la respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `body` | array | Array de objetos con los resultados de la query SQL |
| `result.codResult` | string | Código de resultado (`"0000"` = éxito) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |

⚠️ **Nota sobre el contenido de `body`:**

El array `body` contiene los resultados de la query SQL ejecutada en PostgreSQL NAPAI. La estructura y campos específicos dependen de la query configurada en DynamoDB para cada `queryId`. 

**Evidencia del código (genericSqlFlow.xml):**
```dataweave
{
    "body": vars.postgresResult  // Resultados directos de PostgreSQL
} ++
{
    "result": {
        "codResult": "0000",
        "msgResult": "Resultado correcto!"
    }
}
```

Los ejemplos mostrados (`customerId`, `customerName`, `invoiceId`, etc.) son representativos y varían según la query específica ejecutada.

#### ❌ Error - Query No Encontrada en DynamoDB (500)

```json
{
  "result": {
    "codResult": "5000",
    "msgResult": "Error al llamar la BBDD de dynamo"
  }
}
```

#### ❌ Error - Fallo en Ejecución de Query NAPAI (500)

```json
{
  "result": {
    "codResult": "5000",
    "msgResult": "Error al llamar la BBDD de napai"
  }
}
```

### Códigos de Estado HTTP

| Código | Descripción | Escenario |
|--------|-------------|-----------|
| `200` | OK | Query ejecutada exitosamente |
| `400` | Bad Request | Falta el parámetro `queryId` o formato inválido |
| `500` | Internal Server Error | Error en DynamoDB o PostgreSQL NAPAI |

### Autenticación

- **Tipo:** Client ID & Client Secret (Anypoint Platform)
- **Headers requeridos:**
  - `client_id`: Client ID de la aplicación
  - `client_secret`: Client Secret de la aplicación

### Timeouts

| Tipo | Valor |
|------|-------|
| **Request Timeout** | 60 segundos |
| **Connection Timeout** | 30 segundos |
| **DynamoDB Timeout** | 30 segundos |
| **PostgreSQL NAPAI Timeout** | 60 segundos |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request]) --> LogStart[Log: Inicio genericSqlFlow]
    LogStart --> SetToken[Set Variable: token<br/>queryId en formato DynamoDB]
    SetToken --> LogDynamo[Log: Input Dynamo]
    LogDynamo --> CallDynamo[HTTP Request: s-dynamo<br/>GET /item]
    CallDynamo --> LogDynamoOut[Log: Output Dynamo]
    LogDynamoOut --> Transform[Transform: Mapeo Query Params<br/>Reemplaza # por ,]
    Transform --> LogNapai[Log: Input NAPAI]
    LogNapai --> CallNapai[HTTP Request: s-postgresql-napai<br/>POST /ejecutarSecureQuery]
    CallNapai --> LogNapaiOut[Log: Output NAPAI]
    LogNapaiOut --> BuildResponse[Transform: Build Response<br/>Estructura estándar]
    BuildResponse --> LogEnd[Log: Fin genericSqlFlow]
    LogEnd --> End([HTTP Response 200])
    
    CallDynamo -->|Error| ErrorDynamo[Error Handler: APP:ERROR_DYNAMO]
    CallNapai -->|Error| ErrorNapai[Error Handler: APP:NAPAI]
    ErrorDynamo --> ErrorResponse1[Response: Error DynamoDB]
    ErrorNapai --> ErrorResponse2[Response: Error NAPAI]
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style End fill:#e8f5e9,stroke:#2e7d32
    style CallDynamo fill:#fff3e0,stroke:#e65100
    style CallNapai fill:#e3f2fd,stroke:#1565c0
    style ErrorDynamo fill:#ffebee,stroke:#c62828
    style ErrorNapai fill:#ffebee,stroke:#c62828
```

### Transformaciones DataWeave

#### 1. Preparación del Token para DynamoDB

```dataweave
%dw 2.0
output application/json
---
{
  "queryId": {"S": attributes.queryParams.queryId}
}
```

**Propósito:** Convierte el `queryId` al formato requerido por DynamoDB (tipo String con estructura `{"S": "valor"}`).

#### 2. Mapeo de Query Parameters

```dataweave
%dw 2.0
output application/json
---
attributes.queryParams mapObject ((value, key) -> {
    (key): (value replace "#" with ",")
} - "queryId")
```

**Propósito:** 
- Transforma todos los query parameters excepto `queryId`
- Reemplaza el separador `#` por `,` para permitir múltiples valores
- Ejemplo: `param=val1#val2#val3` → `param=val1,val2,val3`

#### 3. Construcción de Respuesta Final

```dataweave
%dw 2.0
output application/json
---
{
    "body": vars.postgresResult 	
} ++
{
    "result": {
        "codResult": "0000",
        "msgResult": "Resultado correcto!"
    }
}
```

**Propósito:** Envuelve los resultados de PostgreSQL en una estructura estándar con código de resultado.

### Configuración de HTTP Requests

#### Request a DynamoDB (s-dynamo)

```xml
<http:request 
    method="GET" 
    doc:name="Buscamos token en api de dynamo DB" 
    config-ref="HTTP_Request_s_dynamo" 
    path="/item" 
    sendBodyMode="ALWAYS" 
    target="dynamoItem">
    <error-mapping targetType="APP:ERROR_DYNAMO" />
    <http:body><![CDATA[#[vars.token]]]></http:body>
    <http:query-params>
        <![CDATA[#[output application/java
---
{
    "table" : p('fua-s-dynamo.dynamo.table')
}]]]>
    </http:query-params>
</http:request>
```

**Configuración:**
- **Método:** GET
- **Path:** `/item`
- **Body:** Token con queryId en formato DynamoDB
- **Query Param:** `table` (nombre de la tabla DynamoDB desde properties)
- **Target Variable:** `dynamoItem`

#### Request a PostgreSQL NAPAI (s-postgresql-napai)

```xml
<http:request 
    method="POST" 
    doc:name="Request s-postgresql-napai" 
    config-ref="HTTP_Request_s_postgresql-napai" 
    path="/ejecutarSecureQuery" 
    target="postgresResult">
    <error-mapping targetType="APP:NAPAI" />
    <http:body><![CDATA[#[vars.query]]]></http:body>
    <http:query-params>
        <![CDATA[#[output application/java
---
{
    "query" : vars.dynamoItem.item.sqlCode
}]]]>
    </http:query-params>
</http:request>
```

**Configuración:**
- **Método:** POST
- **Path:** `/ejecutarSecureQuery`
- **Body:** Parámetros de la query transformados
- **Query Param:** `query` (SQL code extraído de DynamoDB)
- **Target Variable:** `postgresResult`

### Patrones de Diseño

**Patrón:** Configuration-Driven Query Execution

**Justificación:** 
- Separa la configuración de queries del código de la aplicación
- Permite agregar/modificar queries sin redesplegar
- Mejora la seguridad al prevenir SQL injection
- Facilita auditoría y control de acceso

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API<br/>genericSql
    participant D as System API<br/>s-dynamo
    participant DB1 as DynamoDB
    participant N as System API<br/>s-postgresql-napai
    participant DB2 as PostgreSQL NAPAI
    
    C->>P: GET /genericSql?queryId=QUERY_ID&param1=value1
    activate P
    Note over P: Log inicio + Set token
    
    P->>D: GET /item (body: token, table: queries)
    activate D
    D->>DB1: Query item by queryId
    activate DB1
    DB1-->>D: {item: {sqlCode: "SELECT..."}}
    deactivate DB1
    D-->>P: Query configuration
    deactivate D
    Note over P: Log output Dynamo
    
    Note over P: Transform query params<br/>(replace # with ,)
    Note over P: Log input NAPAI
    
    P->>N: POST /ejecutarSecureQuery<br/>(query: sqlCode, body: params)
    activate N
    N->>DB2: Execute SQL query
    activate DB2
    DB2-->>N: Query results
    deactivate DB2
    N-->>P: Results array
    deactivate N
    Note over P: Log output NAPAI
    
    Note over P: Build standard response
    P-->>C: 200 OK<br/>{body: results, result: {codResult: "0000"}}
    deactivate P
```

#### 2. Flujo con Error en DynamoDB

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API<br/>genericSql
    participant D as System API<br/>s-dynamo
    participant DB1 as DynamoDB
    
    C->>P: GET /genericSql?queryId=INVALID_ID
    activate P
    Note over P: Log inicio + Set token
    
    P->>D: GET /item (body: token)
    activate D
    D->>DB1: Query item by queryId
    activate DB1
    DB1-->>D: Item not found / Error
    deactivate DB1
    D-->>P: Error response
    deactivate D
    
    Note over P: Error Handler: APP:ERROR_DYNAMO
    Note over P: Log error
    Note over P: Transform error response
    
    P-->>C: 500 Internal Server Error<br/>{result: {codResult: "5000",<br/>msgResult: "Error al llamar la BBDD de dynamo"}}
    deactivate P
```

#### 3. Flujo con Error en PostgreSQL NAPAI

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API<br/>genericSql
    participant D as System API<br/>s-dynamo
    participant N as System API<br/>s-postgresql-napai
    participant DB2 as PostgreSQL NAPAI
    
    C->>P: GET /genericSql?queryId=QUERY_ID
    activate P
    
    P->>D: GET /item
    activate D
    D-->>P: Query configuration
    deactivate D
    
    P->>N: POST /ejecutarSecureQuery
    activate N
    N->>DB2: Execute SQL query
    activate DB2
    DB2-->>N: SQL Error (syntax, timeout, etc.)
    deactivate DB2
    N-->>P: Error response
    deactivate N
    
    Note over P: Error Handler: APP:NAPAI
    Note over P: Log error
    Note over P: Transform error response
    
    P-->>C: 500 Internal Server Error<br/>{result: {codResult: "5000",<br/>msgResult: "Error al llamar la BBDD de napai"}}
    deactivate P
```

### Manejo de Errores

#### Error Handlers Configurados

**1. APP:ERROR_DYNAMO**
- **Trigger:** Error al llamar al System API s-dynamo
- **Acción:** 
  - Log del error con el queryId
  - Retorna código `"5000"` con mensaje "Error al llamar la BBDD de dynamo"
- **Tipo:** `on-error-continue` (no propaga el error)

**2. APP:NAPAI**
- **Trigger:** Error al llamar al System API s-postgresql-napai
- **Acción:**
  - Log del error con el queryId
  - Retorna código `"5000"` con mensaje "Error al llamar la BBDD de napai"
- **Tipo:** `on-error-continue` (no propaga el error)

#### Estrategia de Logging

El flow incluye logging extensivo en cada paso:
1. **Inicio:** Log de query parameters de entrada
2. **Pre-DynamoDB:** Log del token a enviar
3. **Post-DynamoDB:** Log de la configuración obtenida
4. **Pre-NAPAI:** Log de los parámetros transformados
5. **Post-NAPAI:** Log de los resultados obtenidos
6. **Fin:** Log de finalización del flow

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta Simple con Un Parámetro

#### cURL

```bash
curl -X GET "https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/genericSql?queryId=QUERY_CUSTOMERS_BY_REGION&region=Madrid" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET"
```

#### Respuesta (200 OK)

```json
{
  "body": [
    {
      "customerId": "12345",
      "customerName": "Juan Pérez",
      "region": "Madrid",
      "contractCount": 3
    },
    {
      "customerId": "67890",
      "customerName": "María García",
      "region": "Madrid",
      "contractCount": 2
    }
  ],
  "result": {
    "codResult": "0000",
    "msgResult": "Resultado correcto!"
  }
}
```

### Ejemplo 2: Consulta con Múltiples Parámetros

#### cURL

```bash
curl -X GET "https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/genericSql?queryId=QUERY_INVOICES_BY_DATE&startDate=2024-01-01&endDate=2024-12-31&status=PAID" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET"
```

#### Respuesta (200 OK)

```json
{
  "body": [
    {
      "invoiceId": "INV-001",
      "invoiceDate": "2024-03-15",
      "amount": 125.50,
      "status": "PAID"
    },
    {
      "invoiceId": "INV-002",
      "invoiceDate": "2024-06-20",
      "amount": 89.75,
      "status": "PAID"
    }
  ],
  "result": {
    "codResult": "0000",
    "msgResult": "Resultado correcto!"
  }
}
```

### Ejemplo 3: Consulta con Parámetros Múltiples (usando #)

#### cURL

```bash
curl -X GET "https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/genericSql?queryId=QUERY_CONTRACTS_BY_IDS&contractIds=C001#C002#C003" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET"
```

**Nota:** El parámetro `contractIds=C001#C002#C003` se transforma internamente a `contractIds=C001,C002,C003` para la query SQL.

#### Respuesta (200 OK)

```json
{
  "body": [
    {
      "contractId": "C001",
      "contractType": "Electricity",
      "status": "Active"
    },
    {
      "contractId": "C002",
      "contractType": "Gas",
      "status": "Active"
    },
    {
      "contractId": "C003",
      "contractType": "Electricity",
      "status": "Suspended"
    }
  ],
  "result": {
    "codResult": "0000",
    "msgResult": "Resultado correcto!"
  }
}
```

### Ejemplo 4: Error - Query No Encontrada

#### cURL

```bash
curl -X GET "https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/genericSql?queryId=INVALID_QUERY_ID" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET"
```

#### Respuesta (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "5000",
    "msgResult": "Error al llamar la BBDD de dynamo"
  }
}
```

### Ejemplo 5: JavaScript (Fetch API)

```javascript
const queryId = 'QUERY_CUSTOMERS_BY_REGION';
const region = 'Madrid';

const response = await fetch(
  `https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/genericSql?queryId=${queryId}&region=${region}`,
  {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      'client_id': 'YOUR_CLIENT_ID',
      'client_secret': 'YOUR_CLIENT_SECRET'
    }
  }
);

const data = await response.json();

if (data.result.codResult === '0000') {
  console.log('Query ejecutada exitosamente');
  console.log('Resultados:', data.body);
} else {
  console.error('Error en la query:', data.result.msgResult);
}
```

### Ejemplo 6: Python (Requests)

```python
import requests

query_id = 'QUERY_CUSTOMERS_BY_REGION'
region = 'Madrid'

url = f'https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/genericSql'
params = {
    'queryId': query_id,
    'region': region
}
headers = {
    'Content-Type': 'application/json',
    'client_id': 'YOUR_CLIENT_ID',
    'client_secret': 'YOUR_CLIENT_SECRET'
}

response = requests.get(url, params=params, headers=headers)
data = response.json()

if data['result']['codResult'] == '0000':
    print('Query ejecutada exitosamente')
    print('Resultados:', data['body'])
else:
    print('Error:', data['result']['msgResult'])
```
