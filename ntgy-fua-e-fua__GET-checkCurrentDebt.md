---
api: "ntgy-fua-e-fua"
sidebar_position: 10
sidebar_label: "GET /checkCurrentDebt"
description: "Consulta si existe deuda pendiente para una cuenta contrato específica en el portal FUA"
keywords: [deuda, pendiente, cuenta contrato, consulta financiera, saldo, FUA, Experience API]
tags: [Financial Services, Debt Management, Experience API, Web]
channel: web
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/checkCurrentDebt"
    purpose: "Consulta de deuda pendiente en Process API"
---

# GET /checkCurrentDebt

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite consultar si existe deuda pendiente para una cuenta contrato específica, proporcionando el monto total de deuda actual. Es utilizado por el portal web FUA para mostrar información financiera del cliente en tiempo real.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para canal web del portal FUA

### Patrón de Integración

**Tipo:** Pass-Through con Headers FUA

**Descripción:** Este endpoint actúa como pass-through hacia el Process API, añadiendo los headers específicos del contexto FUA (x-interaction-id, x-user-id, x-client-id, etc.) que son necesarios para la trazabilidad y seguridad en toda la cadena de APIs.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Portal Web FUA<br/>Cliente]
    E[Experience API<br/>ntgy-fua-e-fua<br/>GET /checkCurrentDebt]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>GET /checkCurrentDebt]
    S[System API<br/>ntgy-newco-s-sap-isu<br/>GET /currentDebt]
    SAP[SAP ISU<br/>RFC: ZFI_FM_DEUDA_CC]
    
    C -->|Request + Query Params| E
    E -->|Request + FUA Headers| P
    P -->|Request + idCuentaContrato| S
    S -->|RFC Call| SAP
    SAP -->|Deuda Pendiente| S
    S -->|Response JSON| P
    P -->|Response JSON| E
    E -->|Response JSON| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style SAP fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Consulta de Deuda en Portal Web**
   - El cliente accede al portal FUA y visualiza su estado de deuda actual
   - Permite tomar decisiones sobre pagos pendientes

2. **Validación Previa a Operaciones**
   - Antes de realizar cambios de contrato o servicios adicionales
   - Verificación de estado financiero del cliente

3. **Información Financiera en Tiempo Real**
   - Consulta actualizada del saldo pendiente
   - Transparencia en la relación comercial con el cliente

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web

**Características del Canal:**
- **Plataforma**: Portal web FUA (Frontal Único de Atención)
- **Dispositivos**: Navegadores web (desktop y responsive)
- **Conectividad**: Conexión estable a internet
- **Experiencia de Usuario**: Interfaz web con información financiera en tiempo real

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Consultan su deuda pendiente para gestionar pagos
- **Clientes Empresariales**: Verifican estado de cuenta para operaciones comerciales
- **Agentes de Atención al Cliente**: Consultan deuda del cliente durante atención telefónica

**Contexto de Uso:**
- **Consulta de Estado Financiero**: Cliente revisa su situación de pagos pendientes
- **Preparación de Pago**: Cliente verifica monto antes de realizar un pago
- **Gestión de Cuenta**: Cliente evalúa su situación financiera con la compañía

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Headers FUA**: Se añaden headers de contexto (x-interaction-id, x-user-id, x-client-id, etc.)
- **Formato de Respuesta**: Pass-through del Process API sin transformaciones adicionales
- **Trazabilidad**: Headers de trazabilidad (traceparent) para seguimiento de requests
- **Seguridad**: Headers de autenticación y autorización (x-user-role, x-client-doc-number)

### 🔗 Endpoints Dependientes

Este Experience API consume los siguientes Process APIs:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/checkCurrentDebt` | GET | Consulta de deuda pendiente en capa de proceso |

**Cadena Completa de Dependencias:**

```
Experience API (ntgy-fua-e-fua)
    ↓ GET /checkCurrentDebt
Process API (ntgy-newco-p-fua-evo)
    ↓ GET /checkCurrentDebt
System API (ntgy-newco-s-sap-isu)
    ↓ GET /currentDebt
SAP ISU
    ↓ RFC: ZFI_FM_DEUDA_CC
```

### Contexto Empresarial

- **Dominio:** Gestión Financiera de Clientes
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Portal Web FUA

### Stakeholders

- **Clientes de Energía**: Necesitan conocer su deuda pendiente para gestionar pagos
- **Departamento de Cobranzas**: Utiliza esta información para seguimiento de cuentas
- **Agentes de Atención al Cliente**: Consultan deuda durante atención al cliente
- **Área Financiera**: Monitorea estado de cuentas por cobrar

### Valor de Negocio

- **Transparencia Financiera**: Cliente tiene visibilidad completa de su deuda
- **Reducción de Llamadas**: Cliente puede consultar su deuda sin contactar call center
- **Mejora en Cobranza**: Cliente informado es más propenso a realizar pagos
- **Experiencia de Usuario**: Información financiera accesible 24/7

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/checkCurrentDebt?contractAccountId={id}` |
| **Development** | `https://ntgy-fua-e-fua-dev.us-e2.cloudhub.io` | `https://ntgy-fua-e-fua-dev.us-e2.cloudhub.io/fua/api/checkCurrentDebt?contractAccountId={id}` |
| **UAT** | `https://ntgy-fua-e-fua-uat.us-e2.cloudhub.io` | `https://ntgy-fua-e-fua-uat.us-e2.cloudhub.io/fua/api/checkCurrentDebt?contractAccountId={id}` |
| **Production** | `https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io` | `https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/checkCurrentDebt?contractAccountId={id}` |

### Método HTTP

```
GET /fua/api/checkCurrentDebt
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 30 segundos | Timeout estándar para consultas síncronas en portal web |
| **Rate Limiting** | Según política API Gateway | Protección contra abuso |
| **Autenticación** | JWT Token obligatorio | Seguridad y trazabilidad de usuario |

#### Headers Específicos del Canal FUA

**Headers Añadidos por el Experience API:**

| Header | Tipo | Descripción | Obligatorio |
|--------|------|-------------|-------------|
| `x-interaction-id` | String | ID único de la interacción para trazabilidad | Sí |
| `x-interaction-origin` | String | Origen de la interacción (ej: "FUA_WEB") | Sí |
| `x-user-id` | String | ID del usuario autenticado | Sí |
| `x-client-id` | String | ID del cliente en el sistema | Sí |
| `x-client-doc-number` | String | Número de documento del cliente | Sí |
| `x-client-code` | String | Código del cliente | Sí |
| `x-origin` | String | Sistema origen de la petición | Sí |
| `x-user-role` | String | Rol del usuario (customer, agent, admin) | Sí |
| `traceparent` | String | Header de trazabilidad distribuida | Opcional |
| `x-agent-mail` | String | Email del agente (si aplica) | Opcional |
| `x-language` | String | Idioma preferido del usuario | Opcional |
| `x-contact-id` | String | ID del contacto en Salesforce | Opcional |

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Obligatorio | Descripción | Ejemplo |
|-----------|------|-------------|-------------|---------|
| `contractAccountId` | String | Sí | ID de la cuenta contrato a consultar | "200012345678" |

### Estructura de Respuesta

#### Respuesta Exitosa (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Success"
  },
  "pendingAmount": 123.45
}
```

**Campos de la Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result` | Object | Objeto con resultado de la operación |
| `result.codResult` | String | Código de resultado ("0000" = éxito, otros = error) |
| `result.msgResult` | String | Mensaje descriptivo del resultado |
| `pendingAmount` | Number | Monto total de deuda pendiente en euros |

#### Respuesta de Error (400/500)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error description"
  }
}
```

### Autenticación

- **Tipo:** JWT Token (OAuth 2.0)
- **Header:** `Authorization: Bearer {token}`
- **Validación:** El token es validado por el flow `jwt-validation` antes de procesar la request

### Códigos de Respuesta HTTP

| Código | Descripción | Escenario |
|--------|-------------|-----------|
| **200** | OK | Consulta exitosa, deuda obtenida correctamente |
| **400** | Bad Request | Parámetros inválidos o faltantes |
| **401** | Unauthorized | Token JWT inválido o expirado |
| **404** | Not Found | Cuenta contrato no encontrada |
| **500** | Internal Server Error | Error en Process API o System API |
| **503** | Service Unavailable | SAP ISU no disponible |

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 30 segundos | Optimizado para consultas web síncronas |
| **Connection Timeout** | 10 segundos | Tiempo máximo para establecer conexión con Process API |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request<br/>Portal Web FUA]) --> Listener[HTTP Listener<br/>Path: /fua/api/*]
    Listener --> JWT[JWT Validation<br/>flow-ref: jwt-validation]
    JWT --> SetHeaders[Set FUA Headers Variable<br/>Transform Message]
    SetHeaders --> Router[APIKit Router<br/>Route to Flow]
    Router --> Flow[get:\checkCurrentDebt:e-fua-config]
    Flow --> FlowRef[Flow Reference<br/>name: get-checkCurrentDebt]
    FlowRef --> HTTPReq[HTTP Request<br/>Process API: ntgy-newco-p-fua-evo<br/>GET /checkCurrentDebt]
    HTTPReq --> Response[Response<br/>Pass-through JSON]
    Response --> End([HTTP Response<br/>200 OK])
    
    HTTPReq -->|Error| ErrorHandler[Error Handler<br/>Transform Error Message]
    ErrorHandler --> ErrorResponse([HTTP Response<br/>Error])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style ErrorResponse fill:#ffebee,stroke:#c62828
    style Flow fill:#fff4e1,stroke:#f57c00,stroke-width:3px
```

### Flujo de Ejecución Detallado

#### 1. Recepción de Request (HTTP Listener)

**Archivo:** `e-fua.xml` (línea 5-13)

```xml
<http:listener config-ref="e-fua-httpListenerConfig" path="/api/*">
    <http:response statusCode="#[vars.httpStatus default 200]">
        <http:headers>#[vars.outboundHeaders default {}]</http:headers>
    </http:response>
</http:listener>
```

#### 2. Validación JWT

**Archivo:** `e-fua.xml` (línea 14)

```xml
<flow-ref doc:name="jwt-validation" name="jwt-validation" />
```

**Propósito:** Valida el token JWT del usuario antes de procesar la request.

#### 3. Configuración de Headers FUA

**Archivo:** `e-fua.xml` (líneas 15-37)

```dataweave
%dw 2.2
output application/java
---
{
    ("x-interaction-id": attributes.headers['x-interaction-id']) if (attributes.headers['x-interaction-id'] != "null" and !isEmpty(attributes.headers['x-interaction-id'])),
    ("x-interaction-origin": attributes.headers['x-interaction-origin']) if (attributes.headers['x-interaction-origin'] != "null" and !isEmpty(attributes.headers['x-interaction-origin'])),
    ("x-user-id": attributes.headers['x-user-id']) if (attributes.headers['x-user-id'] != "null" and !isEmpty(attributes.headers['x-user-id'])),
    ("x-client-id": attributes.headers['x-client-id']) if (attributes.headers['x-client-id'] != "null" and !isEmpty(attributes.headers['x-client-id'])),
    ("x-client-doc-number": attributes.headers['x-client-doc-number']) if (attributes.headers['x-client-doc-number'] != "null" and !isEmpty(attributes.headers['x-client-doc-number'])),
    ("x-client-code": attributes.headers['x-client-code']) if (attributes.headers['x-client-code'] != "null" and !isEmpty(attributes.headers['x-client-code'])),
    ("x-origin": attributes.headers['x-origin']) if (attributes.headers['x-origin'] != "null" and !isEmpty(attributes.headers['x-origin'])),
    ("x-user-role": attributes.headers['x-user-role']) if (attributes.headers['x-user-role'] != "null" and !isEmpty(attributes.headers['x-user-role'])),
    ("traceparent": attributes.headers['traceparent']) if (attributes.headers['traceparent'] != "null" and !isEmpty(attributes.headers['traceparent'])),
    ("x-agent-mail": attributes.headers['x-agent-mail']) if (attributes.headers['x-agent-mail'] != "null" and !isEmpty(attributes.headers['x-agent-mail'])),
    ("x-language": attributes.headers['x-language']) if (attributes.headers['x-language'] != "null" and !isEmpty(attributes.headers['x-language'])),
    ("x-contact-id": attributes.headers['x-contact-id']) if (attributes.headers['x-contact-id'] != "null" and !isEmpty(attributes.headers['x-contact-id']))
}
```

**Propósito:** Extrae y valida los headers FUA del request original, almacenándolos en la variable `fuaHeaders` para propagarlos al Process API.

#### 4. Routing APIKit

**Archivo:** `e-fua.xml` (línea 38)

```xml
<apikit:router config-ref="e-fua-config" />
```

**Propósito:** Enruta la request al flow específico basándose en el path y método HTTP.

#### 5. Flow del Endpoint

**Archivo:** `e-fua.xml` (líneas 485-487)

```xml
<flow name="get:\checkCurrentDebt:e-fua-config">
    <flow-ref doc:name="Flow Reference" name="get-checkCurrentDebt" />
</flow>
```

#### 6. Implementación del Subflow

**Archivo:** `get-checkCurrentDebt.xml` (proporcionado en la tarea)

```xml
<flow name="get-checkCurrentDebt">
    <http:request method="GET" 
                  doc:name="Newco-evo Get CheckCurrentDebt Request" 
                  config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
                  path="/checkCurrentDebt">
        <http:query-params><![CDATA[#[output application/java
---
attributes.queryParams ++ vars.fuaHeaders]]]></http:query-params>
    </http:request>
    <error-handler>
        <on-error-propagate enableNotifications="true" logException="true">
            <ee:transform doc:name="Set Error Message">
                <ee:message>
                    <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0001",
        "msgResult": error.description as String replace /\"/ with("")
    }
}]]></ee:set-payload>
                </ee:message>
            </ee:transform>
            <logger level="ERROR" doc:name="Error" message='...' />
        </on-error-propagate>
    </error-handler>
</flow>
```

**Componentes Clave:**

1. **HTTP Request al Process API:**
   - **Config:** `HTTP_FUA_PROCESS_NEWCO_EVO`
   - **Path:** `/checkCurrentDebt`
   - **Query Params:** Combina `attributes.queryParams` (contractAccountId) con `vars.fuaHeaders`

2. **Error Handler:**
   - Captura cualquier error del Process API
   - Transforma el error a formato estándar
   - Registra el error en logs

### Transformaciones de Canal

⚠️ **Este Experience API NO transforma la respuesta del Process API.**

**Patrón:** Pass-Through con Headers FUA

**Operación:**
- **Input:** Query params del cliente + Headers FUA del contexto
- **Output:** Response del Process API sin modificaciones

Para el schema completo de la respuesta, consultar:
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `/checkCurrentDebt`**
📖 **System API: `ntgy-newco-s-sap-isu` endpoint `/currentDebt`**

### Cadena de Llamadas Completa

#### Experience API → Process API

**Transformación de Query Params:**

```dataweave
output application/java
---
attributes.queryParams ++ vars.fuaHeaders
```

**Resultado:**
```json
{
  "contractAccountId": "200012345678",
  "x-interaction-id": "uuid-123",
  "x-user-id": "user-456",
  "x-client-id": "client-789",
  // ... otros headers FUA
}
```

#### Process API → System API

**Archivo:** `ntgy-newco-p-fua-evo/checkCurrentDebt.xml` (proporcionado en la tarea)

**Transformación:**

```dataweave
ee:set-variable variableName="idCuentaContrato">
    <![CDATA[attributes.queryParams.'contractAccountId' default ""]]>
</ee:set-variable>
```

**HTTP Request:**

```xml
<http:request method="GET" 
              path="${ntgy-newco-s-sap.functions.getCurrentDebt}"
              config-ref="HTTP_Request_ntgy-newco-s-sap">
    <http:query-params><![CDATA[#[output application/java
---
{
    idCuentaContrato : vars.idCuentaContrato
}]]]></http:query-params>
</http:request>
```

#### System API → SAP ISU

**Archivo:** `ntgy-newco-s-sap-isu/currentDebtFlow.xml` (proporcionado en la tarea)

**Transformación Input (DataWeave):**

**Archivo:** `current-debt-rfc-input.dwl`

```dataweave
%dw 2.0
output application/xml
---
{
    ZFI_FM_DEUDA_CC: {
        "import": {
            I_VKONT: attributes.queryParams.idCuentaContrato
        }
    }
}
```

**RFC Call:**

```xml
<sap:synchronous-remote-function-call 
    key="ZFI_FM_DEUDA_CC" 
    config-ref="SAP_Config" />
```

**Transformación Output:**

```dataweave
%dw 2.0
output application/json

var salida = payload.ZFI_FM_DEUDA_CC.export
---
{
    "result": {
        "codResult": "0" ++ salida.ES_ERROR.CODE,
        "msgResult": salida.ES_ERROR.DESCRIPTION
    },
    "pendingAmount": salida.E_SUM_OPENITEMS
}
```

### Patrones de Diseño

**Patrón:** API Gateway con Header Propagation

**Justificación:** 
- El Experience API actúa como gateway que añade contexto de seguridad y trazabilidad
- Los headers FUA son esenciales para auditoría y seguimiento de transacciones
- Permite identificar el origen y usuario de cada request en toda la cadena de APIs

**Beneficios:**
- **Trazabilidad Completa:** Cada request puede ser rastreada desde el cliente hasta SAP
- **Seguridad:** Headers de autenticación y autorización propagados
- **Auditoría:** Información de usuario y contexto disponible en todos los niveles
- **Debugging:** Facilita la identificación de problemas en producción

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente Web FUA
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant S as System API<br/>ntgy-newco-s-sap-isu
    participant SAP as SAP ISU<br/>RFC ZFI_FM_DEUDA_CC
    
    C->>E: GET /checkCurrentDebt?contractAccountId=200012345678
    Note over E: Validate JWT Token
    Note over E: Extract FUA Headers
    E->>P: GET /checkCurrentDebt<br/>+ contractAccountId<br/>+ FUA Headers
    Note over P: Map contractAccountId<br/>to idCuentaContrato
    P->>S: GET /currentDebt?idCuentaContrato=200012345678
    Note over S: Transform to RFC Input
    S->>SAP: Call RFC ZFI_FM_DEUDA_CC<br/>I_VKONT=200012345678
    SAP-->>S: E_SUM_OPENITEMS=123.45<br/>ES_ERROR.CODE=00
    Note over S: Transform RFC Output
    S-->>P: {"result":{"codResult":"0000"},"pendingAmount":123.45}
    P-->>E: {"result":{"codResult":"0000"},"pendingAmount":123.45}
    E-->>C: 200 OK<br/>{"result":{"codResult":"0000"},"pendingAmount":123.45}
```

#### 2. Flujo con Error de Negocio (Cuenta No Encontrada)

```mermaid
sequenceDiagram
    participant C as Cliente Web FUA
    participant E as Experience API
    participant P as Process API
    participant S as System API
    participant SAP as SAP ISU
    
    C->>E: GET /checkCurrentDebt?contractAccountId=999999999
    Note over E: Validate JWT Token
    E->>P: GET /checkCurrentDebt + Headers
    P->>S: GET /currentDebt?idCuentaContrato=999999999
    S->>SAP: Call RFC ZFI_FM_DEUDA_CC
    SAP-->>S: ES_ERROR.CODE=01<br/>ES_ERROR.DESCRIPTION=Cuenta no encontrada
    Note over S: Transform Error
    S-->>P: {"result":{"codResult":"0001","msgResult":"Cuenta no encontrada"}}
    P-->>E: {"result":{"codResult":"0001","msgResult":"Cuenta no encontrada"}}
    E-->>C: 200 OK<br/>{"result":{"codResult":"0001","msgResult":"Cuenta no encontrada"}}
```

#### 3. Flujo con Error Técnico (Timeout SAP)

```mermaid
sequenceDiagram
    participant C as Cliente Web FUA
    participant E as Experience API
    participant P as Process API
    participant S as System API
    participant SAP as SAP ISU
    
    C->>E: GET /checkCurrentDebt?contractAccountId=200012345678
    Note over E: Validate JWT Token
    E->>P: GET /checkCurrentDebt + Headers
    P->>S: GET /currentDebt
    S->>SAP: Call RFC ZFI_FM_DEUDA_CC
    Note over SAP: Timeout (30s)
    SAP--xS: Connection Timeout
    Note over S: Error Handler
    S-->>P: 500 Internal Server Error
    Note over P: Error Handler
    P-->>E: HTTP Error Response
    Note over E: Transform Error
    E-->>C: 500 Internal Server Error<br/>{"result":{"codResult":"0001","msgResult":"Timeout error"}}
```

### Manejo de Errores

#### Error Handler del Experience API

**Ubicación:** `get-checkCurrentDebt.xml`

```xml
<error-handler>
    <on-error-propagate enableNotifications="true" logException="true">
        <ee:transform doc:name="Set Error Message">
            <ee:message>
                <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0001",
        "msgResult": error.description as String replace /\"/ with("")
    }
}]]></ee:set-payload>
            </ee:message>
        </ee:transform>
        <logger level="ERROR" doc:name="Error" />
    </on-error-propagate>
</error-handler>
```

**Estrategia:**
1. Captura cualquier error del Process API
2. Transforma el error a formato estándar de respuesta
3. Registra el error en logs para auditoría
4. Propaga el error al cliente con código 0001

#### Tipos de Errores Manejados

| Tipo de Error | Origen | Código | Mensaje | HTTP Status |
|---------------|--------|--------|---------|-------------|
| **Cuenta No Encontrada** | SAP ISU | 0001 | "Cuenta no encontrada" | 200 |
| **Error de Conexión SAP** | System API | 0001 | "Connection timeout" | 500 |
| **Token JWT Inválido** | Experience API | - | "Unauthorized" | 401 |
| **Parámetro Faltante** | Experience API | - | "Bad request" | 400 |
| **Error Genérico** | Cualquiera | 0001 | error.description | 500 |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta Exitosa con Deuda Pendiente

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/checkCurrentDebt?contractAccountId=200012345678" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -H "x-interaction-id: 550e8400-e29b-41d4-a716-446655440000" \
  -H "x-interaction-origin: FUA_WEB" \
  -H "x-user-id: user-12345" \
  -H "x-client-id: client-67890" \
  -H "x-client-doc-number: 12345678A" \
  -H "x-client-code: CLI001" \
  -H "x-origin: WEB_PORTAL" \
  -H "x-user-role: customer"
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Success"
  },
  "pendingAmount": 156.78
}
```

**Interpretación:**
- `codResult: "0000"` indica operación exitosa
- `pendingAmount: 156.78` indica que el cliente tiene 156.78€ de deuda pendiente

### Ejemplo 2: Consulta Exitosa sin Deuda Pendiente

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/checkCurrentDebt?contractAccountId=200087654321" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -H "x-interaction-id: 550e8400-e29b-41d4-a716-446655440001" \
  -H "x-interaction-origin: FUA_WEB" \
  -H "x-user-id: user-12345" \
  -H "x-client-id: client-67890" \
  -H "x-client-doc-number: 12345678A" \
  -H "x-client-code: CLI001" \
  -H "x-origin: WEB_PORTAL" \
  -H "x-user-role: customer"
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Success"
  },
  "pendingAmount": 0.00
}
```

**Interpretación:**
- `codResult: "0000"` indica operación exitosa
- `pendingAmount: 0.00` indica que el cliente NO tiene deuda pendiente

### Ejemplo 3: Error - Cuenta No Encontrada

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/checkCurrentDebt?contractAccountId=999999999" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -H "x-interaction-id: 550e8400-e29b-41d4-a716-446655440002" \
  -H "x-interaction-origin: FUA_WEB" \
  -H "x-user-id: user-12345" \
  -H "x-client-id: client-67890" \
  -H "x-client-doc-number: 12345678A" \
  -H "x-client-code: CLI001" \
  -H "x-origin: WEB_PORTAL" \
  -H "x-user-role: customer"
```

#### Respuesta (200 OK - Error de Negocio)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Cuenta contrato no encontrada en SAP ISU"
  }
}
```

**Interpretación:**
- `codResult: "0001"` indica error de negocio
- La cuenta contrato no existe en el sistema SAP ISU

### Ejemplo 4: JavaScript (React/Vue) - Portal Web FUA

```javascript
// Función para consultar deuda pendiente
async function checkCurrentDebt(contractAccountId, authToken, fuaHeaders) {
  try {
    const response = await fetch(
      `https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/checkCurrentDebt?contractAccountId=${contractAccountId}`,
      {
        method: 'GET',
        headers: {
          'Authorization': `Bearer ${authToken}`,
          'Content-Type': 'application/json',
          'x-interaction-id': fuaHeaders.interactionId,
          'x-interaction-origin': 'FUA_WEB',
          'x-user-id': fuaHeaders.userId,
          'x-client-id': fuaHeaders.clientId,
          'x-client-doc-number': fuaHeaders.docNumber,
          'x-client-code': fuaHeaders.clientCode,
          'x-origin': 'WEB_PORTAL',
          'x-user-role': fuaHeaders.userRole
        }
      }
    );

    const data = await response.json();
    
    if (data.result.codResult === "0000") {
      console.log(`Deuda pendiente: ${data.pendingAmount}€`);
      return {
        success: true,
        pendingAmount: data.pendingAmount,
        hasDebt: data.pendingAmount > 0
      };
    } else {
      console.error(`Error: ${data.result.msgResult}`);
      return {
        success: false,
        error: data.result.msgResult
      };
    }
  } catch (error) {
    console.error('Error al consultar deuda:', error);
    return {
      success: false,
      error: 'Error de conexión'
    };
  }
}

// Ejemplo de uso
const result = await checkCurrentDebt(
  '200012345678',
  'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
  {
    interactionId: '550e8400-e29b-41d4-a716-446655440000',
    userId: 'user-12345',
    clientId: 'client-67890',
    docNumber: '12345678A',
    clientCode: 'CLI001',
    userRole: 'customer'
  }
);

if (result.success) {
  if (result.hasDebt) {
    alert(`Tiene una deuda pendiente de ${result.pendingAmount}€`);
  } else {
    alert('No tiene deuda pendiente');
  }
}
```

### Ejemplo 5: Python - Script de Consulta

```python
import requests
import json
from typing import Dict, Optional

def check_current_debt(
    contract_account_id: str,
    auth_token: str,
    fua_headers: Dict[str, str]
) -> Dict:
    """
    Consulta la deuda pendiente de una cuenta contrato.
    
    Args:
        contract_account_id: ID de la cuenta contrato
        auth_token: Token JWT de autenticación
        fua_headers: Headers FUA requeridos
        
    Returns:
        Dict con resultado de la consulta
    """
    url = "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/checkCurrentDebt"
    
    headers = {
        'Authorization': f'Bearer {auth_token}',
        'Content-Type': 'application/json',
        'x-interaction-id': fua_headers.get('interaction_id'),
        'x-interaction-origin': 'FUA_WEB',
        'x-user-id': fua_headers.get('user_id'),
        'x-client-id': fua_headers.get('client_id'),
        'x-client-doc-number': fua_headers.get('doc_number'),
        'x-client-code': fua_headers.get('client_code'),
        'x-origin': 'WEB_PORTAL',
        'x-user-role': fua_headers.get('user_role')
    }
    
    params = {
        'contractAccountId': contract_account_id
    }
    
    try:
        response = requests.get(url, headers=headers, params=params, timeout=30)
        response.raise_for_status()
        
        data = response.json()
        
        if data['result']['codResult'] == '0000':
            return {
                'success': True,
                'pending_amount': data['pendingAmount'],
                'has_debt': data['pendingAmount'] > 0
            }
        else:
            return {
                'success': False,
                'error': data['result']['msgResult']
            }
            
    except requests.exceptions.Timeout:
        return {'success': False, 'error': 'Timeout al consultar deuda'}
    except requests.exceptions.RequestException as e:
        return {'success': False, 'error': f'Error de conexión: {str(e)}'}
    except json.JSONDecodeError:
        return {'success': False, 'error': 'Error al parsear respuesta JSON'}

# Ejemplo de uso
if __name__ == '__main__':
    result = check_current_debt(
        contract_account_id='200012345678',
        auth_token='eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
        fua_headers={
            'interaction_id': '550e8400-e29b-41d4-a716-446655440000',
            'user_id': 'user-12345',
            'client_id': 'client-67890',
            'doc_number': '12345678A',
            'client_code': 'CLI001',
            'user_role': 'customer'
        }
    )
    
    if result['success']:
        if result['has_debt']:
            print(f"Deuda pendiente: {result['pending_amount']}€")
        else:
            print("No hay deuda pendiente")
    else:
        print(f"Error: {result['error']}")
```

### Ejemplo 6: Error de Autenticación (401)

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/checkCurrentDebt?contractAccountId=200012345678" \
  -H "Authorization: Bearer token_invalido_o_expirado" \
  -H "Content-Type: application/json"
```

#### Respuesta (401 Unauthorized)

```json
{
  "error": "Unauthorized",
  "message": "Invalid or expired JWT token"
}
```

**Interpretación:**
- El token JWT es inválido o ha expirado
- El usuario debe autenticarse nuevamente
