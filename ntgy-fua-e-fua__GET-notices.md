---
api: "ntgy-fua-e-fua"
sidebar_position: 28
sidebar_label: "GET /notices"
description: "Obtiene la lista de avisos personalizados vigentes para el usuario autenticado en el portal FUA"
keywords:
  - notices
  - avisos
  - notificaciones
  - dynamodb
  - fua
  - experience-api
  - cliente
tags:
  - Experience API
  - Avisos
  - Notificaciones
channel: "web"
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/notices"
    purpose: "Obtiene avisos del cliente desde DynamoDB con filtrado por vigencia"
---

# GET /notices

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite obtener la lista de avisos personalizados vigentes para el usuario autenticado en el portal FUA (Frontal Único de Atención). Los avisos son notificaciones importantes que se muestran al cliente sobre eventos relevantes como nuevas facturas online, FacturaE disponibles, o información importante relacionada con sus contratos de energía.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para canal web del portal FUA

### Patrón de Integración

**Tipo:** Pass-Through con Propagación de Headers

**Descripción:** Este Experience API actúa como pass-through, delegando la lógica completa al Process API. Su función principal es propagar los headers FUA necesarios para la autenticación y trazabilidad de la petición.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente Web<br/>Portal FUA]
    E[Experience API<br/>ntgy-fua-e-fua<br/>GET /notices]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>GET /notices]
    S[System API<br/>s-dynamodb<br/>POST /query]
    D[(DynamoDB<br/>ntgy-dydb-fua-notice)]
    
    C -->|Request + Headers FUA| E
    E -->|Pass-Through| P
    P -->|Query con CLIENT_CODE| S
    S -->|Consulta| D
    D -->|Items| S
    S -->|Response| P
    P -->|Filtrado por fechas| P
    P -->|Response| E
    E -->|Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style D fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Notificación de Nueva Factura Online (FOL)**
   - El cliente recibe un aviso cuando su nueva factura online está disponible
   - Incluye fecha estimada de cobro y enlace directo a la factura
   - Multiidioma: ES, CA, EN, Z2 (Euskera), Z3 (Gallego)

2. **Notificación de Nueva FacturaE**
   - Aviso cuando una nueva FacturaE está disponible en la plataforma
   - Información sobre acceso a la factura electrónica
   - Soporte multiidioma

3. **Gestión de Avisos Personalizados**
   - Visualización de avisos específicos por cliente (CLIENT_CODE)
   - Filtrado automático de avisos vigentes (no caducados)
   - Soporte para avisos por dirección de suministro

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web

**Características del Canal:**
- **Plataforma**: Portal FUA (Frontal Único de Atención)
- **Dispositivos**: Navegadores web (desktop y responsive)
- **Conectividad**: Conexión estable requerida
- **Experiencia de Usuario**: Notificaciones en tiempo real al acceder al portal

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Clientes de energía que acceden al portal para gestionar sus contratos
- **Clientes Empresariales**: Empresas que gestionan múltiples contratos de energía

**Contexto de Uso:**
- **Acceso al Portal**: Al iniciar sesión en el portal FUA
- **Consulta de Avisos**: Revisión periódica de notificaciones importantes
- **Gestión de Facturas**: Notificaciones relacionadas con facturación

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Propagación de Headers FUA**: Headers de autenticación y trazabilidad
- **Pass-Through Eficiente**: Sin transformaciones adicionales para minimizar latencia
- **Filtrado en Process API**: Lógica de filtrado por fechas delegada al Process API

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/notices` | GET | Obtiene avisos del cliente desde DynamoDB con filtrado por vigencia |

El Process API a su vez consume:

| System API | Endpoint | Método | Propósito |
|------------|----------|--------|-----------|
| `s-dynamodb` | `/query` | POST | Consulta avisos en tabla ntgy-dydb-fua-notice usando índice CLIENT_CODE-index |

### Contexto Empresarial

- **Dominio:** Autoservicio de Clientes - Notificaciones
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Web (Portal FUA)

### Stakeholders

- **Clientes de Energía**: Reciben notificaciones importantes sobre sus contratos y facturas
- **Equipo de Atención al Cliente**: Gestiona la comunicación con clientes a través de avisos
- **Equipo de Facturación**: Notifica disponibilidad de nuevas facturas

### Valor de Negocio

- **Comunicación Proactiva**: Notificación automática de eventos importantes al cliente
- **Reducción de Llamadas**: Los clientes están informados sin necesidad de contactar al call center
- **Mejora de Experiencia**: Información relevante y oportuna en el portal
- **Multiidioma**: Soporte para todas las lenguas oficiales de España

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/notices` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/notices` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices` |

### Método HTTP

```
GET /fua/api/notices
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Timeout configurado en Process API |
| **Rate Limiting** | Según política OAuth 2.0 | Control de acceso por usuario |
| **Autenticación** | Obligatoria | Requiere headers FUA válidos |

#### Headers Específicos del Canal (FUA)

**Headers Propagados por el Experience API:**

| Header | Tipo | Descripción | Obligatorio |
|--------|------|-------------|-------------|
| `x-interaction-id` | string | ID único de la interacción para trazabilidad | No |
| `x-interaction-origin` | string | Origen de la interacción (web, mobile, etc.) | No |
| `x-user-id` | string | ID del usuario autenticado | No |
| `x-client-id` | string | ID del cliente en el sistema | No |
| `x-client-doc-number` | string | Número de documento del cliente | No |
| `x-client-code` | string | Código del cliente (usado para consulta) | Sí |
| `x-origin` | string | Origen de la petición | No |
| `x-user-role` | string | Rol del usuario (cliente, agente, etc.) | No |
| `traceparent` | string | Header de trazabilidad distribuida | No |
| `x-agent-mail` | string | Email del agente (si aplica) | No |
| `x-language` | string | Idioma preferido del usuario | No |
| `x-contact-id` | string | ID del contacto en Salesforce | No |

### Parámetros de Entrada

#### Query Parameters

Este endpoint **NO requiere query parameters**. La identificación del cliente se realiza a través del header `x-client-code`.

#### Headers Requeridos

```http
x-client-code: string (OBLIGATORIO)
```

**Descripción:** Código del cliente para consultar sus avisos personalizados.

**Ejemplo:**
```http
GET /fua/api/notices
x-client-code: 12345678A
x-user-id: user123
x-language: ES
```

### Respuestas

#### Respuesta Exitosa (200 OK)

**Estructura:**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Item found"
  },
  "notices": [
    {
      "clientCode": "string",
      "noticeId": "string",
      "type": "string",
      "text": "string",
      "title": "string",
      "visibility": "string",
      "read": boolean,
      "delete": boolean,
      "validity": "string",
      "startDate": "string",
      "endDate": "string",
      "idioma": "string",
      "dirSuministro": "string",
      "url": "string"
    }
  ]
}
```

**Descripción de Campos:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | string | Código de resultado ("0000" = éxito) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |
| `notices` | array | Lista de avisos vigentes del cliente |
| `notices[].clientCode` | string | Código del cliente (CLIENT_CODE) |
| `notices[].noticeId` | string | ID único del aviso (NOTICE_ID) |
| `notices[].type` | string | Tipo de aviso (Información, Informació, etc.) |
| `notices[].text` | string | Texto completo del aviso |
| `notices[].title` | string | Título del aviso |
| `notices[].visibility` | string | Visibilidad del aviso ("all", etc.) |
| `notices[].read` | boolean | Indica si el aviso ha sido leído |
| `notices[].delete` | boolean | Indica si el aviso ha sido eliminado |
| `notices[].validity` | string | Días de validez del aviso |
| `notices[].startDate` | string | Fecha de inicio (formato: yyyy/MM/dd) |
| `notices[].endDate` | string | Fecha de fin (formato: yyyy/MM/dd) |
| `notices[].idioma` | string | Idioma del aviso (ES, CA, EN, Z2, Z3) |
| `notices[].dirSuministro` | string | Dirección de suministro asociada |
| `notices[].url` | string | URL relacionada con el aviso (ej: enlace a factura) |

#### Respuesta de Error (500)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error description"
  }
}
```

### Autenticación

- **Tipo:** OAuth 2.0 + Headers FUA
- **Validación:** JWT validation en el flow principal
- **Headers:** Propagación de headers FUA para autenticación y autorización

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Configurado en Process API para consultas a DynamoDB |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP Request<br/>GET /notices]) --> Validate[JWT Validation]
    Validate --> SetHeaders[Set fuaHeaders Variable]
    SetHeaders --> Router[APIKit Router]
    Router --> FlowRef[Flow Reference: get-notices]
    FlowRef --> HTTPReq[HTTP Request to Process API]
    HTTPReq --> PropagateQP[Propagate QueryParams + fuaHeaders]
    PropagateQP --> ProcessAPI[ntgy-newco-p-fua-evo<br/>GET /notices]
    ProcessAPI --> Response[Response from Process API]
    Response --> End([HTTP Response])
    
    HTTPReq -->|Error| ErrorHandler[Error Handler]
    ErrorHandler --> ErrorTransform[Transform Error Message]
    ErrorTransform --> ErrorLog[Log Error]
    ErrorLog --> ErrorResponse([Error Response])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style ErrorResponse fill:#ffebee,stroke:#c62828
    style ProcessAPI fill:#ffe1f5,stroke:#c2185b
```

### Flujo Detallado

#### 1. Experience API (ntgy-fua-e-fua)

**Archivo:** `src/main/mule/e-fua.xml` (línea 561-563)

```xml
<flow name="get:\notices:e-fua-config">
    <flow-ref name="get-notices" />
</flow>
```

**Sub-flow get-notices:**

```xml
<flow name="get-notices">
    <http:request method="GET" 
                  config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
                  path="/notices">
        <http:query-params>
            #[output application/java
            ---
            attributes.queryParams ++ vars.fuaHeaders]
        </http:query-params>
    </http:request>
    <error-handler>
        <on-error-propagate>
            <ee:transform>
                <ee:set-payload>
                    {
                        "result": {
                            "codResult": "0001",
                            "msgResult": error.description
                        }
                    }
                </ee:set-payload>
            </ee:transform>
            <logger level="ERROR" message="Error in get-notices"/>
        </on-error-propagate>
    </error-handler>
</flow>
```

**Configuración HTTP:**
- **Config:** `HTTP_FUA_PROCESS_NEWCO_EVO`
- **Host:** `${newco-p-fua-evo.host}` (mule-worker-internal-ntgy-newco-p-fua-evo.de-c1.cloudhub.io)
- **Port:** `${newco-p-fua-evo.port}` (8091)
- **Timeout:** `${newco-p-fua-evo.timeout}` (60000 ms)

#### 2. Process API (ntgy-newco-p-fua-evo)

**Archivo:** `src/main/mule/services/clients/notices.xml` (línea 10-142)

**Flow Principal:**

```xml
<flow name="get:\notices:ntgy-newco-p-fua-config">
    <logger level="DEBUG" message="get:\notices:ntgy-newco-p-fua-config"/>
    <set-variable variableName="docNumber" 
                  value="#[attributes.queryParams.'x-client-code' default payload.data.nif]"/>
    <flow-ref name="notices"/>
</flow>
```

**Sub-flow notices:**

1. **Preparar Query para DynamoDB:**

```dataweave
%dw 2.0
output application/json
---
{
    "attributes": {
        ":CLIENT_CODE": {
            "S": vars.docNumber 
        }
    },
    "condition": "CLIENT_CODE=:CLIENT_CODE",
    "tableName": "ntgy-dydb-fua-notice",
    "index": "CLIENT_CODE-index"
}
```

2. **Consultar DynamoDB:**

```xml
<http:request method="POST" 
              config-ref="HTTP_Request_s_dynamo" 
              path="/query" 
              target="resultAvisos"/>
```

3. **Filtrar y Transformar Respuesta:**

```dataweave
%dw 2.0
output application/json
---
{
  "noticesClient": vars.resultAvisos.items filter ((item) -> 
        item.END_DATE as Date {format: "yyyy/MM/dd"} > now() - |P1D|
    ) map ((item, index) -> {
      "clientCode": item.CLIENT_CODE,
      "noticeId": item.NOTICE_ID,
      "type": item.TYPE,
      "text": item.TEXT,
      "title": item.TITLE,
      "visibility": item.VISIBILITY,
      "read": item.READ default false,
      "delete": item.DELETE default false,
      "validity": item.VALIDITY,
      "startDate": item.START_DATE,
      "endDate": item.END_DATE,
      "idioma": item.IDIOMA,
      "dirSuministro": item.DIRSUMINISTRO,
      "url": item.URL 
  }) default []
}
```

4. **Respuesta Final:**

```dataweave
%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0000",
        "msgResult": "Item found"
    },
    "notices": flatten(vars.noticesClient.noticesClient) 
}
```

### Transformaciones de Canal

**Este Experience API NO transforma la respuesta del Process API.**

⚠️ **Pass-Through Completo**: La respuesta del Process API se devuelve tal cual al cliente.

Para el schema completo y lógica de filtrado, consultar:
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `/notices`**

### Patrones de Diseño

**Patrón:** Pass-Through con Propagación de Headers

**Justificación:** 
- Minimiza latencia al no realizar transformaciones adicionales
- Centraliza la lógica de negocio en el Process API
- Permite reutilización del Process API por otros canales
- Simplifica el mantenimiento al tener una única fuente de lógica

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API
    participant P as Process API
    participant S as System API (DynamoDB)
    participant D as DynamoDB
    
    C->>E: GET /notices<br/>Headers: x-client-code
    Note over E: JWT Validation
    Note over E: Set fuaHeaders
    E->>P: GET /notices<br/>QueryParams + fuaHeaders
    Note over P: Extract docNumber
    P->>P: Build DynamoDB Query
    P->>S: POST /query<br/>CLIENT_CODE filter
    S->>D: Query with Index
    D-->>S: Items Array
    S-->>P: DynamoDB Response
    Note over P: Filter by END_DATE > now() - 1 day
    Note over P: Transform to notices structure
    P-->>E: Response with notices array
    E-->>C: 200 OK<br/>notices array
```

#### 2. Flujo con Error de Negocio (Sin Avisos)

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API
    participant P as Process API
    participant S as System API (DynamoDB)
    participant D as DynamoDB
    
    C->>E: GET /notices<br/>Headers: x-client-code
    E->>P: GET /notices
    P->>S: POST /query
    S->>D: Query with Index
    D-->>S: Empty Items Array
    S-->>P: Empty Response
    Note over P: Filter returns empty array
    P-->>E: Response with empty notices
    E-->>C: 200 OK<br/>notices: []
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API
    participant P as Process API
    participant S as System API (DynamoDB)
    
    C->>E: GET /notices<br/>Headers: x-client-code
    E->>P: GET /notices
    P->>S: POST /query
    S-->>P: 500 Error
    Note over P: Error Handler
    P-->>E: Error Response
    Note over E: Transform Error
    Note over E: Log Error
    E-->>C: 500 Error<br/>codResult: "0001"
```

### Lógica de Filtrado por Fechas

**Condición de Filtrado:**

```dataweave
item.END_DATE as Date {format: "yyyy/MM/dd"} > now() - |P1D|
```

**Explicación:**
- Convierte `END_DATE` de string a Date con formato `yyyy/MM/dd`
- Compara con la fecha actual menos 1 día (`|P1D|` = Period 1 Day)
- Solo incluye avisos cuya fecha de fin sea posterior a ayer
- Esto permite mostrar avisos del día actual y futuros

**Ejemplo:**
- Fecha actual: 2026-10-15
- Fecha de comparación: 2026-10-14
- Avisos con END_DATE >= 2026-10-15 se incluyen
- Avisos con END_DATE < 2026-10-15 se excluyen

### Tabla DynamoDB

**Nombre:** `ntgy-dydb-fua-notice`

**Índice Utilizado:** `CLIENT_CODE-index`

**Estructura de Campos:**

| Campo DynamoDB | Tipo | Descripción |
|----------------|------|-------------|
| `CLIENT_CODE` | String | Código del cliente (partition key del índice) |
| `NOTICE_ID` | String | ID único del aviso |
| `TYPE` | String | Tipo de aviso |
| `TEXT` | String | Texto completo del aviso |
| `TITLE` | String | Título del aviso |
| `VISIBILITY` | String | Visibilidad ("all", etc.) |
| `READ` | Boolean | Estado de lectura |
| `DELETE` | Boolean | Estado de eliminación |
| `VALIDITY` | String | Días de validez |
| `START_DATE` | String | Fecha de inicio (yyyy/MM/dd) |
| `END_DATE` | String | Fecha de fin (yyyy/MM/dd) |
| `IDIOMA` | String | Código de idioma |
| `DIRSUMINISTRO` | String | Dirección de suministro |
| `URL` | String | URL relacionada |

### Manejo de Errores

**Error Handler en Experience API:**

```xml
<error-handler>
    <on-error-propagate>
        <ee:transform>
            <ee:set-payload>
                {
                    "result": {
                        "codResult": "0001",
                        "msgResult": error.description as String replace /\"/ with("")
                    }
                }
            </ee:set-payload>
        </ee:transform>
        <logger level="ERROR" message="#[payload]"/>
    </on-error-propagate>
</error-handler>
```

**Códigos de Error:**

| Código | Descripción | Causa |
|--------|-------------|-------|
| `0000` | Éxito | Avisos encontrados (puede ser array vacío) |
| `0001` | Error | Error técnico en la consulta o procesamiento |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Llamada Exitosa con Avisos

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices" \
  -H "Content-Type: application/json" \
  -H "x-client-code: 12345678A" \
  -H "x-user-id: user123" \
  -H "x-language: ES" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Item found"
  },
  "notices": [
    {
      "clientCode": "12345678A",
      "noticeId": "550e8400-e29b-41d4-a716-446655440000",
      "type": "Información",
      "text": "Acabamos de enviarte un email con tu nueva factura online, que tiene fecha estimada de cobro el 25/10/2026. En el correo puedes ver un pequeño resumen con los datos más relevantes o descargarte la versión completa. También puedes acceder a ella haciendo click {{urlFOL}}. Si no recibes el correo, por favor revisa en tu Área Clientes que lo tengamos bien escrito.",
      "title": "Tu nueva factura online ya está disponible",
      "visibility": "all",
      "read": false,
      "delete": false,
      "validity": "90",
      "startDate": "2026/10/15",
      "endDate": "2027/01/13",
      "idioma": "ES",
      "dirSuministro": "Calle Mayor 123, Madrid",
      "url": "https://portal.naturgy.es/factura/123456"
    },
    {
      "clientCode": "12345678A",
      "noticeId": "660e8400-e29b-41d4-a716-446655440001",
      "type": "Información",
      "text": "Recibirás un email informándote de la nueva FacturaE, que también puedes consultar en tu plataforma.",
      "title": "Tu nueva FacturaE ya está disponible",
      "visibility": "all",
      "read": false,
      "delete": false,
      "validity": "90",
      "startDate": "2026/10/10",
      "endDate": "2027/01/08",
      "idioma": "ES",
      "dirSuministro": "",
      "url": ""
    }
  ]
}
```

### Ejemplo 2: Llamada Exitosa sin Avisos

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices" \
  -H "Content-Type: application/json" \
  -H "x-client-code: 87654321B" \
  -H "x-user-id: user456" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Item found"
  },
  "notices": []
}
```

### Ejemplo 3: Error de Autenticación

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices" \
  -H "Content-Type: application/json" \
  -H "x-client-code: 12345678A"
  # Sin header Authorization
```

#### Respuesta (401 Unauthorized)

```json
{
  "message": "Unauthorized"
}
```

### Ejemplo 4: Error Técnico

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices" \
  -H "Content-Type: application/json" \
  -H "x-client-code: 12345678A" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Respuesta (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error connecting to DynamoDB"
  }
}
```

### Ejemplo 5: JavaScript (Frontend Web)

```javascript
// Función para obtener avisos del cliente
async function getClientNotices(clientCode, authToken) {
  try {
    const response = await fetch('https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'x-client-code': clientCode,
        'x-user-id': 'user123',
        'x-language': 'ES',
        'Authorization': `Bearer ${authToken}`
      }
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    
    if (data.result.codResult === '0000') {
      console.log(`Se encontraron ${data.notices.length} avisos`);
      return data.notices;
    } else {
      console.error('Error en la respuesta:', data.result.msgResult);
      return [];
    }
  } catch (error) {
    console.error('Error al obtener avisos:', error);
    return [];
  }
}

// Uso
const notices = await getClientNotices('12345678A', 'your-jwt-token');
notices.forEach(notice => {
  console.log(`${notice.title}: ${notice.text}`);
});
```

### Ejemplo 6: Python

```python
import requests
from typing import List, Dict, Optional

def get_client_notices(
    client_code: str, 
    auth_token: str,
    base_url: str = "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io"
) -> Optional[List[Dict]]:
    """
    Obtiene los avisos de un cliente desde el portal FUA.
    
    Args:
        client_code