---
api: "ntgy-fua-e-fua"
sidebar_position: 84
sidebar_label: "POST /sendContractCopy"
description: "Envía copia del contrato por correo electrónico al cliente con enlace de descarga temporal"
keywords: [sendContractCopy, contrato, email, duplicado, marketing cloud, token temporal, fua]
tags: [Experience API, FUA, Contratos, Email]
channel: "FUA Web"
endpoint_dependencies:
  - method: "POST"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/sendContractCopy"
    purpose: "Orquestación del proceso completo de envío de copia de contrato por email"
---

# POST /sendContractCopy

## 📌 Descripción General

### Propósito del Endpoint

El endpoint **POST /sendContractCopy** permite a los agentes del FUA (Frontal Único de Atención) enviar copias de contratos por correo electrónico a los clientes. Este Experience API enriquece la solicitud con información contextual del canal FUA (headers de interacción, usuario, cliente) y la dirección IP del solicitante, delegando la orquestación del proceso completo al Process API correspondiente.

El endpoint facilita:
- **Envío inmediato de contratos**: El cliente recibe el documento en su email en minutos
- **Trazabilidad completa**: Registro automático de la gestión en Salesforce
- **Seguridad**: Enlaces de descarga con tokens temporales de 30 días
- **Contexto FUA**: Enriquecimiento automático con datos de la sesión del agente

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para el canal FUA Web, enriqueciendo requests con contexto de sesión y usuario antes de delegar al Process API.

### Patrón de Integración

**Tipo:** Pass-Through con Enriquecimiento de Headers

**Descripción:** El Experience API recibe la solicitud del portal FUA, enriquece el payload con headers de contexto (x-contact-id, x-client-id, x-interaction-id, x-user-id, x-agent-mail) y la dirección IP del cliente, y delega toda la lógica de orquestación al Process API. No aplica transformaciones complejas, solo añade metadata del canal.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Portal FUA Web<br/>Agentes Call Center]
    E[Experience API<br/>ntgy-fua-e-fua<br/>POST /sendContractCopy]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /sendContractCopy]
    S1[System API<br/>PostgreSQL/Salesforce]
    S2[System API<br/>DynamoDB]
    S3[System API<br/>Marketing Cloud]
    S4[System API<br/>Salesforce]
    
    C -->|Request + FUA Headers| E
    E -->|Enrich: Headers + IP| P
    P -->|Orchestrate| S1
    P -->|Orchestrate| S2
    P -->|Orchestrate| S3
    P -->|Orchestrate| S4
    S1 -->|Response| P
    S2 -->|Response| P
    S3 -->|Response| P
    S4 -->|Response| P
    P -->|Response| E
    E -->|Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S3 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S4 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Envío de Contrato por Agente de Call Center**
   - Un cliente llama solicitando una copia de su contrato
   - El agente accede al FUA y solicita el envío por email
   - El sistema genera un enlace seguro y envía el email automáticamente
   - **Beneficio**: Resolución inmediata sin procesos manuales (reducción de 24-48h a minutos)

2. **Autoservicio desde Portal FUA**
   - Cliente autenticado en el portal solicita copia de su contrato
   - El sistema procesa la solicitud automáticamente
   - Cliente recibe email con enlace de descarga
   - **Beneficio**: Reducción de carga en call center, mejora de experiencia del cliente

3. **Cumplimiento y Auditoría**
   - Cada envío genera un caso de gestión en Salesforce
   - Registro de IP, usuario, fecha y hora
   - Trazabilidad completa del proceso
   - **Beneficio**: Cumplimiento normativo y auditoría de accesos

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** FUA Web (Frontal Único de Atención)

**Características del Canal:**
- **Plataforma**: Portal web interno para agentes de Naturgy
- **Dispositivos**: Estaciones de trabajo de call center
- **Conectividad**: Red corporativa interna
- **Experiencia de Usuario**: Interfaz optimizada para gestión rápida de solicitudes

#### Audiencia Objetivo

**Usuarios Finales:**
- **Agentes de Call Center**: Personal de atención al cliente que gestiona solicitudes telefónicas
- **Agentes de Oficina**: Personal de oficinas comerciales que atiende clientes presencialmente
- **Supervisores**: Personal con permisos elevados para gestiones especiales

**Contexto de Uso:**
- **Atención Telefónica**: Cliente llama solicitando documentación
- **Atención Presencial**: Cliente en oficina solicita copia de contrato
- **Gestión Proactiva**: Agente envía documentación como parte de un proceso de gestión

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Enriquecimiento Automático**: Headers FUA añadidos automáticamente (x-contact-id, x-client-id, x-interaction-id)
- **Captura de IP**: Dirección IP del solicitante extraída del request
- **Contexto de Sesión**: Información del agente logueado (x-user-id, x-agent-mail)
- **Validación de Sesión**: JWT validation aplicada en el flow principal

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/sendContractCopy` | POST | Orquestación del proceso completo: consulta metadata, genera token, envía email, crea caso |

**Nota:** El Process API se encarga de toda la orquestación con los System APIs (PostgreSQL, DynamoDB, Marketing Cloud, Salesforce).

### Contexto Empresarial

- **Dominio:** Gestión de Documentos y Comunicaciones con Clientes
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** FUA Web (Portal de Agentes)

### Stakeholders

- **Agentes de Call Center**: Necesitan enviar contratos de forma rápida y sencilla
- **Clientes**: Reciben documentación de forma inmediata por email
- **Área de Cumplimiento**: Requieren trazabilidad de envíos de documentación
- **Área de Sistemas**: Gestionan seguridad y auditoría de accesos

### Valor de Negocio

- **Eficiencia Operativa**: Reducción de tiempo de gestión de 24-48h a minutos
- **Mejora de Experiencia**: Cliente recibe documento inmediatamente
- **Reducción de Costes**: Elimina envíos postales y gestión manual
- **Seguridad**: Enlaces con tokens temporales y validación
- **Trazabilidad**: Registro automático en Salesforce para auditoría

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/sendContractCopy` |
| **Development** | `https://ntgy-fua-e-fua-dev.us-e2.cloudhub.io` | `https://ntgy-fua-e-fua-dev.us-e2.cloudhub.io/fua/api/sendContractCopy` |
| **UAT** | `https://ntgy-fua-e-fua-uat.us-e2.cloudhub.io` | `https://ntgy-fua-e-fua-uat.us-e2.cloudhub.io/fua/api/sendContractCopy` |
| **Production** | `https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io` | `https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/sendContractCopy` |

### Método HTTP

```
POST /fua/api/sendContractCopy
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Proceso complejo con múltiples integraciones |
| **Autenticación** | JWT obligatorio | Seguridad del canal FUA |
| **Headers FUA** | Obligatorios | Trazabilidad y contexto de sesión |
| **Rate Limiting** | Por configuración API Manager | Protección del servicio |

#### Campos Específicos del Canal

**Campos Añadidos por el Experience API:**

| Campo | Tipo | Descripción | Razón |
|-------|------|-------------|-------|
| `x-contact-id` | string (header) | ID del contacto en Salesforce | Trazabilidad del cliente |
| `x-client-id` | string (header) | ID del cliente en Salesforce | Identificación del account |
| `x-interaction-id` | string (header) | ID del caso de interacción padre | Vinculación de gestiones |
| `x-user-id` | string (header) | ID del usuario/agente | Auditoría de quién realiza la acción |
| `x-agent-mail` | string (header) | Email del agente | Para registro en Salesforce |
| `DirIp` | string (body) | IP del solicitante | Seguridad y auditoría |

**Campos Omitidos del Process API:**

Ninguno. Este Experience API hace pass-through completo del payload, solo añade campos adicionales.

### Parámetros de Entrada

#### Headers HTTP

| Header | Tipo | Requerido | Descripción | Ejemplo |
|--------|------|-----------|-------------|---------|
| `Content-Type` | string | ✅ Sí | Tipo de contenido | `application/json` |
| `Authorization` | string | ✅ Sí | JWT token de sesión FUA | `Bearer eyJhbGc...` |
| `x-contact-id` | string | ✅ Sí | ID del contacto en Salesforce | `003XXXXXXXXXXXXXXX` |
| `x-client-id` | string | ✅ Sí | ID del cliente en Salesforce | `001XXXXXXXXXXXXXXX` |
| `x-interaction-id` | string | ✅ Sí | ID del caso de interacción | `500XXXXXXXXXXXXXXX` |
| `x-user-id` | string | ❌ No | ID del usuario/agente | `005XXXXXXXXXXXXXXX` |
| `x-agent-mail` | string | ❌ No | Email del agente | `agente@naturgy.com` |

#### Body Parameters (JSON)

| Campo | Tipo | Requerido | Descripción | Ejemplo |
|-------|------|-----------|-------------|---------|
| `docNumber` | string | ❌ No | NIF/NIE del titular | `"12345678A"` |
| `documentId` | string | ✅ Sí | ID del documento | `"DOC123456"` |
| `cups` | string | ❌ No | CUPS del suministro | `"ES0021000000000001JN"` |
| `contractNumber` | string | ❌ No | Número de contrato | `"1234567890"` |
| `name` | string | ❌ No | Nombre del titular | `"Juan"` |
| `surname` | string | ❌ No | Primer apellido | `"García"` |
| `surname2` | string | ❌ No | Segundo apellido | `"López"` |
| `email` | string | ✅ Sí | Email destino | `"cliente@example.com"` |
| `language` | string | ✅ Sí | Código de idioma | `"ES"` |
| `reason` | string | ✅ Sí | Motivo de la solicitud | `"Solicitud de copia de contrato"` |
| `supplyAddress` | string | ✅ Sí | Dirección de suministro | `"Calle Mayor 1, Madrid"` |
| `contactId` | string | ❌ No | ID del contacto (alternativo) | `"003XXX"` |
| `clientId` | string | ❌ No | ID del cliente (alternativo) | `"001XXX"` |

**Notas:** 
- Los campos `contactId` y `clientId` en el body son opcionales porque se pueden obtener de los headers.
- El campo `DirIp` NO debe ser enviado por el cliente, se añade automáticamente por el Experience API extrayendo la IP del request.

### Respuestas

#### Respuesta Exitosa (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "OK"
  }
}
```

**Descripción:** El contrato fue enviado exitosamente. El cliente recibirá un email con el enlace de descarga.

#### Respuesta de Error (500 Internal Server Error)

```json
{
  "message": "Se ha producido un error en el proceso. Inténtelo mas tarde"
}
```

**Descripción:** Error durante el procesamiento. Puede ser error en el Process API o en alguno de los System APIs downstream.

**Causas Comunes:**
- Error al consultar metadata del documento
- Error al generar token en DynamoDB
- Error al enviar email con Marketing Cloud
- Error al crear caso en Salesforce
- Timeout en alguna de las integraciones

### Autenticación y Autorización

#### Autenticación del Consumidor
- **Tipo:** JWT Token (validación en flow principal)
- **Header:** `Authorization: Bearer {token}`
- **Validación:** Sub-flow `jwt-validation` ejecutado antes del APIKit router

#### Políticas de API Manager
- **Client ID Enforcement**: Requerido
- **Rate Limiting**: Según configuración del ambiente
- **CORS**: Configurado para portal FUA

### Timeouts y Retry

| Configuración | Valor | Descripción |
|---------------|-------|-------------|
| **Request Timeout** | 60 segundos | Timeout del Process API |
| **Connection Timeout** | 30 segundos | Timeout de conexión HTTP |
| **Retry Policy** | No aplica | El Process API maneja sus propios retries |

**Nota:** El Experience API no implementa retry logic. Si el Process API falla, retorna error inmediatamente.

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request]) --> JWT[JWT Validation<br/>Sub-flow]
    JWT --> SetHeaders[Set fuaHeaders Variable<br/>Extract FUA Headers]
    SetHeaders --> Router[APIKit Router]
    Router --> MainFlow[Flow: post:\sendContractCopy]
    MainFlow --> FlowRef[Flow-Ref:<br/>post-send-contract-copy]
    FlowRef --> Transform[DataWeave Transform:<br/>Enrich Payload]
    Transform --> AddIP[Add DirIp Field<br/>Extract from remoteAddress]
    AddIP --> HTTPRequest[HTTP Request:<br/>POST to Process API]
    HTTPRequest --> Success[Success Response]
    Success --> End([HTTP 200 Response])
    
    HTTPRequest -.->|Error| ErrorHandler[On-Error-Propagate]
    ErrorHandler --> SetError[Set Error Message]
    SetError --> LogError[Log Error]
    LogError --> ErrorEnd([HTTP 500 Response])
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style End fill:#e8f5e9,stroke:#2e7d32
    style ErrorEnd fill:#ffebee,stroke:#c62828
    style HTTPRequest fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style Transform fill:#fff3e0,stroke:#f57c00
```

### Transformaciones de Canal

**Transformación de Enriquecimiento:**

| Campo Original | Campo Añadido | Transformación |
|----------------|---------------|----------------|
| `payload.*` | `payload.*` | Pass-through completo |
| `vars.fuaHeaders` | `payload.x-*` | Merge de headers FUA al payload |
| `attributes.remoteAddress` | `payload.DirIp` | Extracción de IP del request |

**Lógica de Transformación:**

```dataweave
%dw 2.2
import * from dw::core::Strings
output application/json
---
payload
++
vars.fuaHeaders
++
{
    "DirIp": substringBefore(substringAfter(attributes.remoteAddress,"/"),":")
}
```

**Descripción:**
1. **payload**: Mantiene todos los campos originales del request
2. **vars.fuaHeaders**: Añade headers FUA (x-contact-id, x-client-id, x-interaction-id, x-user-id, x-agent-mail, x-language)
3. **DirIp**: Extrae la IP del campo `attributes.remoteAddress` (formato: `/192.168.1.1:12345`)

**Ejemplo de Transformación:**

**Input (Request Body):**
```json
{
  "documentId": "DOC123",
  "email": "cliente@example.com",
  "language": "ES"
}
```

**Headers FUA (vars.fuaHeaders):**
```json
{
  "x-contact-id": "003XXX",
  "x-client-id": "001XXX",
  "x-interaction-id": "500XXX",
  "x-user-id": "005XXX",
  "x-agent-mail": "agente@naturgy.com"
}
```

**Output (Enviado al Process API):**
```json
{
  "documentId": "DOC123",
  "email": "cliente@example.com",
  "language": "ES",
  "x-contact-id": "003XXX",
  "x-client-id": "001XXX",
  "x-interaction-id": "500XXX",
  "x-user-id": "005XXX",
  "x-agent-mail": "agente@naturgy.com",
  "DirIp": "192.168.1.1"
}
```

### Patrones de Diseño

**Patrón:** Pass-Through con Enriquecimiento (Enrichment Pattern)

**Justificación:** 
- El Experience API no contiene lógica de negocio compleja
- Su responsabilidad es adaptar el request al contexto del canal FUA
- Añade metadata de sesión y contexto sin modificar la estructura original
- Delega toda la orquestación al Process API

**Patrón:** Header Propagation

**Justificación:**
- Los headers FUA son críticos para trazabilidad
- Se extraen una vez en el flow principal y se propagan a todos los sub-flows
- Evita duplicación de lógica de extracción de headers

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant A as Agente FUA
    participant E as Experience API
    participant P as Process API
    participant S as System APIs
    
    A->>E: POST /sendContractCopy<br/>Headers: JWT, x-contact-id, x-client-id<br/>Body: documentId, email, language
    
    Note over E: JWT Validation
    Note over E: Extract FUA Headers
    
    E->>E: DataWeave Transform<br/>Enrich: payload + fuaHeaders + DirIp
    
    E->>P: POST /sendContractCopy<br/>Enriched Payload
    
    Note over P: Orchestration
    P->>S: Query Metadata (PostgreSQL)
    S-->>P: Document Info
    P->>S: Generate Token (DynamoDB)
    S-->>P: Token Created
    P->>S: Send Email (Marketing Cloud)
    S-->>P: Email Sent
    P->>S: Create Case (Salesforce)
    S-->>P: Case Created
    
    P-->>E: 200 OK<br/>{codResult: "0000"}
    E-->>A: 200 OK<br/>{codResult: "0000"}
```

#### 2. Flujo con Error en Process API

```mermaid
sequenceDiagram
    participant A as Agente FUA
    participant E as Experience API
    participant P as Process API
    
    A->>E: POST /sendContractCopy<br/>Request Data
    
    Note over E: JWT Validation OK
    Note over E: Enrich Payload
    
    E->>P: POST /sendContractCopy
    
    Note over P: Error en DynamoDB
    P-->>E: 500 Internal Server Error
    
    Note over E: On-Error-Propagate
    E->>E: Set Error Message
    E->>E: Log Error
    
    E-->>A: 500 Internal Server Error<br/>"Se ha producido un error..."
```

#### 3. Flujo con Error de Autenticación

```mermaid
sequenceDiagram
    participant A as Agente FUA
    participant E as Experience API
    
    A->>E: POST /sendContractCopy<br/>Invalid/Missing JWT
    
    Note over E: JWT Validation FAILED
    
    E-->>A: 401 Unauthorized<br/>"Invalid token"
```

### Configuración HTTP

#### HTTP Request al Process API

```xml
<http:request method="POST" 
              config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
              path="/sendContractCopy">
</http:request>
```

**Configuración del Connector:**
```xml
<http:request-config name="HTTP_FUA_PROCESS_NEWCO_EVO" 
                     basePath="/api" 
                     responseTimeout="${newco-p-fua-evo.timeout}">
    <http:request-connection 
        host="${newco-p-fua-evo.host}" 
        port="${newco-p-fua-evo.port}" 
        connectionIdleTimeout="${newco-p-fua-evo.timeout}"/>
</http:request-config>
```

**Propiedades (Producción):**
- `host`: `mule-worker-internal-ntgy-newco-p-fua-evo.de-c1.cloudhub.io`
- `port`: `8091`
- `timeout`: `60000` ms (60 segundos)

### Manejo de Errores

#### Error Handler del Flow

```xml
<error-handler>
    <on-error-propagate enableNotifications="true" 
                        logException="true">
        <set-payload value="#[&quot;Se ha producido un error en el proceso. Inténtelo mas tarde&quot;]"/>
    </on-error-propagate>
</error-handler>
```

**Estrategia:**
- **Propagate**: El error se propaga al cliente con mensaje genérico
- **Logging**: Se registra el error completo en logs para troubleshooting
- **Mensaje**: Mensaje user-friendly sin detalles técnicos

#### Tipos de Errores Manejados

| Error Type | HTTP Status | Mensaje | Acción |
|------------|-------------|---------|--------|
| JWT Validation Failed | 401 | "Invalid token" | Rechazar request |
| Process API Error | 500 | "Se ha producido un error..." | Log + mensaje genérico |
| Timeout | 500 | "Se ha producido un error..." | Log + mensaje genérico |
| Connectivity Error | 500 | "Se ha producido un error..." | Log + mensaje genérico |

### Variables de Contexto

#### fuaHeaders Variable

**Definición:**
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

**Propósito:** Extrae y valida headers FUA del request, filtrando valores null o vacíos.

**Ubicación:** Flow principal `e-fua-main`, antes del APIKit router.

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Envío Exitoso desde Portal FUA

#### cURL

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/sendContractCopy" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-contact-id: 003XXXXXXXXXXXXXXX" \
  -H "x-client-id: 001XXXXXXXXXXXXXXX" \
  -H "x-interaction-id: 500XXXXXXXXXXXXXXX" \
  -H "x-user-id: 005XXXXXXXXXXXXXXX" \
  -H "x-agent-mail: agente.callcenter@naturgy.com" \
  -d '{
    "documentId": "DOC123456",
    "contractNumber": "1234567890",
    "docNumber": "12345678A",
    "cups": "ES0021000000000001JN",
    "name": "Juan",
    "surname": "García",
    "surname2": "López",
    "email": "juan.garcia@example.com",
    "language": "ES",
    "reason": "Solicitud de copia de contrato por parte del cliente",
    "supplyAddress": "Calle Mayor 1, 28001 Madrid"
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "OK"
  }
}
```

---

### Ejemplo 2: JavaScript (Fetch API) - Portal FUA

```javascript
const sendContractCopy = async (contractData) => {
  try {
    const response = await fetch(
      'https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/sendContractCopy',
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${sessionStorage.getItem('jwtToken')}`,
          'x-contact-id': sessionStorage.getItem('contactId'),
          'x-client-id': sessionStorage.getItem('clientId'),
          'x-interaction-id': sessionStorage.getItem('interactionId'),
          'x-user-id': sessionStorage.getItem('userId'),
          'x-agent-mail': sessionStorage.getItem('agentEmail')
        },
        body: JSON.stringify({
          documentId: contractData.documentId,
          contractNumber: contractData.contractNumber,
          docNumber: contractData.docNumber,
          cups: contractData.cups,
          name: contractData.name,
          surname: contractData.surname,
          surname2: contractData.surname2,
          email: contractData.email,
          language: contractData.language || 'ES',
          reason: contractData.reason,
          supplyAddress: contractData.supplyAddress
        })
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    
    if (data.result.codResult === '0000') {
      console.log('Contrato enviado exitosamente');
      return { success: true, data };
    } else {
      console.error('Error en el envío:', data.result.msgResult);
      return { success: false, error: data.result.msgResult };
    }
  } catch (error) {
    console.error('Error al enviar contrato:', error);
    return { success: false, error: error.message };
  }
};

// Uso en el portal FUA
const contractData = {
  documentId: 'DOC123456',
  contractNumber: '1234567890',
  docNumber: '12345678A',
  cups: 'ES0021000000000001JN',
  name: 'Juan',
  surname: 'García',
  surname2: 'López',
  email: 'juan.garcia@example.com',
  language: 'ES',
  reason: 'Solicitud de copia de contrato por parte del cliente',
  supplyAddress: 'Calle Mayor 1, 28001 Madrid'
};

sendContractCopy(contractData)
  .then(result => {
    if (result.success) {
      alert('Contrato enviado exitosamente al email del cliente');
    } else {
      alert('Error al enviar contrato: ' + result.error);
    }
  });
```

---

### Ejemplo 3: Python (Requests) - Integración Backend

```python
import requests
import os

def send_contract_copy(contract_data, session_headers):
    """
    Envía copia de contrato desde sistema backend
    
    Args:
        contract_data (dict): Datos del contrato
        session_headers (dict): Headers de sesión FUA
    
    Returns:
        dict: Response del API
    """
    url = "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/sendContractCopy"
    
    headers = {
        'Content-Type': 'application/json',
        'Authorization': f"Bearer {session_headers['jwt_token']}",
        'x-contact-id': session_headers['contact_id'],
        'x-client-id': session_headers['client_id'],
        'x-interaction-id': session_headers['interaction_id'],
        'x-user-id': session_headers.get('user_id', ''),
        'x-agent-mail': session_headers.get('agent_mail', '')
    }
    
    try:
        response = requests.post(url, json=contract_data, headers=headers, timeout=65)
        response.raise_for_status()
        
        result = response.json()
        if result['result']['codResult'] == '0000':
            print(f"✓ Contrato enviado exitosamente a {contract_data['email']}")
            return {'success': True, 'data': result}
        else:
            print(f"✗ Error: {result['result']['msgResult']}")
            return {'success': False, 'error': result['result']['msgResult']}
            
    except requests.exceptions.Timeout:
        print("✗ Timeout: El proceso tardó más de 60 segundos")
        return {'success': False, 'error': 'Timeout'}
    except requests.exceptions.HTTPError as e:
        print(f"✗ HTTP Error: {e.response.status_code}")
        return {'success': False, 'error': str(e)}
    except Exception as e:
        print(f"✗ Error inesperado: {str(e)}")
        return {'success': False, 'error': str(e)}

# Ejemplo de uso
if __name__ == "__main__":
    contract_data = {
        'documentId': 'DOC123456',
        'contractNumber': '1234567890',
        'docNumber': '12345678A',
        'cups': 'ES0021000000000001JN',
        'name': 'María',
        'surname': 'Rodríguez',
        'surname2': 'Martínez',
        'email': 'maria.rodriguez@example.com',
        'language': 'ES',
        'reason': 'Solicitud de copia de contrato por parte del cliente',
        'supplyAddress': 'Calle Mayor 1, 28001 Madrid'
    }
    
    session_headers = {
        'jwt_token': 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
        'contact_id': '003XXXXXXXXXXXXXXX',
        'client_id': '001XXXXXXXXXXXXXXX',
        'interaction_id': '500XXXXXXXXXXXXXXX',
        'user_id': '005XXXXXXXXXXXXXXX',
        'agent_mail': 'agente@naturgy.com'
    }
    
    result = send_contract_copy(contract_data, session_headers)
    print(f"Resultado: {result}")
```

---

### Ejemplo 4: Error - Falta Header Obligatorio

#### Request

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/sendContractCopy" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGc..." \
  -d '{
    "documentId": "DOC123456",
    "email": "cliente@example.com",
    "language": "ES"
  }'
```

#### Respuesta (400 Bad Request)

```json
{
  "message": "Bad request",
  "details": "Missing required header: x-contact-id"
}
```

---

### Ejemplo 5: Error - Token JWT Inválido

#### Request

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/sendContractCopy" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer INVALID_TOKEN" \
  -H "x-contact-id: 003XXX" \
  -H "x-client-id: 001XXX" \
  -H "x-interaction-id: 500XXX" \
  -d '{
    "documentId": "DOC123456",
    "email": "cliente@example.com",
    "language": "ES"
  }'
```

#### Respuesta (401 Unauthorized)

```json
{
  "error": "Unauthorized",
  "message": "Invalid or expired JWT token"
}
```

---

### Notas de Implementación

#### Mejores Prácticas para Consumidores

1. **Gestión de Sesión FUA**
   - Mantener headers FUA en sessionStorage/localStorage
   - Renovar JWT token antes de expiración
   - Limpiar sesión al logout

2. **Manejo de Errores**
   - Implementar retry solo para errores de red (no para errores de negocio)
   - Mostrar mensajes user-friendly al usuario
   - Registrar errores para análisis

3. **Timeouts**
   - Configurar timeout del cliente en 65 segundos (5s más que el servidor)
   - Mostrar indicador de progreso al usuario
   - Permitir cancelación de request

4. **Validación de Datos**
   - Validar email antes de enviar
   - Validar que documentId existe
   - Confirmar con usuario antes de enviar

#### Consideraciones de Performance

- **Latencia Esperada**: 5-10 segundos (incluye orquestación completa)
- **Throughput**: Limitado por rate limits de API Manager
- **Concurrencia**: El API puede manejar múltiples requests concurrentes
- **Caché**: No aplica, cada envío es único

#### Troubleshooting

**Problema:** Error "Invalid JWT token"
- **Causa**: Token expirado o inválido
- **Solución**: Renovar token de sesión FUA

**Problema:** Error "Missing required header"
- **Causa**: Headers FUA no enviados
- **Solución**: Verificar que todos los headers obligatorios están presentes

**Problema:** Timeout después de 60 segundos
- **Causa**: Process API tardando más de lo esperado
- **Solución**: Verificar estado de System APIs downstream, reintentar

**Problema:** Error "Se ha producido un error en el proceso"
- **Causa**: Error en Process API o System APIs
- **Solución**: Revisar logs del Process API, verificar conectividad con System APIs
