---
api: "ntgy-fua-e-fua"
sidebar_position: 92
sidebar_label: "PUT /clients"
description: "Endpoint de Experience API para actualizar datos de clientes mediante orquestación con Process API Newco"
keywords: [clientes, actualización, modificaciones, experience api, newco, fua]
tags: [Experience API, Clientes, Actualización, Web, Mobile]
channel: web
endpoint_dependencies:
  - method: POST
    path: /modificaciones
    api_name: ntgy-newco-p-fua-evo
---

# PUT /clients

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite a los clientes actualizar sus datos personales y de contacto a través del portal FUA (Frontal de Atención al Usuario). Actúa como una capa de adaptación que enriquece la petición con headers de contexto y la IP del cliente antes de delegarla al Process API de Newco.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para canales web y móvil con enriquecimiento de contexto

### Patrón de Integración

**Tipo:** Pass-Through con Header Enrichment

**Descripción:** El endpoint recibe la petición del cliente, añade información de contexto (headers FUA + IP del cliente) y la reenvía al Process API de Newco que orquesta las modificaciones en Salesforce.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente<br/>Portal FUA / App Móvil]
    E[Experience API<br/>MUL-COM-FUA-E-FUA<br/>PUT /clients]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /modificaciones]
    S[System API<br/>ntgy-newco-s-salesforce]
    
    C -->|Request con datos<br/>de cliente| E
    E -->|Enriquece con<br/>headers + IP| P
    P -->|Router dinámico<br/>según operationType| S
    S -->|Actualiza<br/>Salesforce| S
    S -->|Response| P
    P -->|Response| E
    E -->|Response<br/>adaptada| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Actualización de Datos del Titular (operationType: "account")**
   - Permite a los clientes modificar su nombre, apellidos, nacionalidad
   - Actualiza información del titular de la cuenta (Account) en Salesforce
   - **Beneficio:** Autoservicio para mantener datos personales actualizados sin llamar al call center
   - **Uso típico:** Cambio de nombre tras matrimonio, corrección de datos personales

2. **Modificación de Datos de Contacto (operationType: "contacto")**
   - Actualiza teléfono, email, canal preferente de comunicación
   - Gestiona preferencias de contacto y horarios
   - **Beneficio:** Control total sobre cómo y cuándo ser contactado
   - **Uso típico:** Cambio de número de teléfono, actualización de email

3. **Cambio de Preferencias GDPR (operationType: "gdpr")**
   - Modifica consentimientos de privacidad y tratamiento de datos
   - Gestiona preferencias de comunicaciones comerciales
   - **Beneficio:** Cumplimiento normativo y control de privacidad
   - **Uso típico:** Gestión de consentimientos, opt-out de comunicaciones

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web | Mobile

**Características del Canal:**
- **Plataforma**: Portal web FUA responsive y aplicaciones móviles nativas
- **Dispositivos**: Desktop, tablets, smartphones (iOS/Android)
- **Conectividad**: Requiere conexión a internet estable
- **Experiencia de Usuario**: Formularios optimizados para autoservicio

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Clientes particulares de energía que gestionan sus datos personales
- **Clientes Business**: Pequeñas empresas que actualizan información de contacto

**Contexto de Uso:**
- **Autoservicio 24/7**: Clientes actualizando datos fuera de horario de atención
- **Post-mudanza**: Actualización de dirección y datos de contacto tras cambio de domicilio
- **Gestión de Privacidad**: Modificación de consentimientos GDPR

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Enriquecimiento de Contexto**: Añade headers FUA con información de sesión
- **Captura de IP**: Registra IP del cliente para auditoría y seguridad
- **Formato Simplificado**: Mantiene estructura simple para facilitar integración frontend
- **Validación Temprana**: Headers OAuth 2.0 validados en Experience API

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/modificaciones` | POST | Router dinámico que orquesta modificaciones en Salesforce según tipo de operación |

**Nota sobre SSOT (Single Source of Truth):**

⚠️ **Este Experience API es pass-through con enriquecimiento mínimo.**

Para el schema completo de request/response y tipos de operaciones disponibles, consultar:
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `POST /modificaciones`**

El Process API documenta todos los tipos de `operationType` soportados y sus schemas específicos.

### Contexto Empresarial

- **Dominio:** Autoservicio de Clientes
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Portal FUA (Frontal de Atención al Usuario)

### Stakeholders

- **Clientes Finales**: Usuarios del portal FUA que actualizan sus datos
- **Equipo de Desarrollo Frontend**: Desarrolladores de portal web y apps móviles
- **Equipo de Atención al Cliente**: Reducción de llamadas por cambios de datos
- **Compliance Officer**: Gestión de consentimientos GDPR

### Valor de Negocio

- **Reducción de Costes Operativos**: Menos llamadas al call center por cambios de datos
- **Mejora de Experiencia de Cliente**: Autoservicio inmediato 24/7
- **Cumplimiento Normativo**: Gestión transparente de consentimientos GDPR
- **Calidad de Datos**: Datos actualizados directamente por el cliente

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/clients` |
| **Development** | `https://e-fua-dev.de-c1.cloudhub.io` | `https://e-fua-dev.de-c1.cloudhub.io/fua/api/clients` |
| **UAT** | `https://e-fua-uat.de-c1.cloudhub.io` | `https://e-fua-uat.de-c1.cloudhub.io/fua/api/clients` |
| **Production** | `https://e-fua-pro.de-c1.cloudhub.io` | `https://e-fua-pro.de-c1.cloudhub.io/fua/api/clients` |

### Método HTTP

```
PUT /fua/api/clients
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 90 segundos | Permite completar orquestación completa en Process API |
| **Tamaño de Respuesta** | 50 KB máximo | Respuestas simples de confirmación |
| **Rate Limiting** | 100 requests/minuto por cliente | Prevenir abuso y garantizar disponibilidad |
| **Autenticación** | OAuth 2.0 obligatorio | Seguridad y trazabilidad de modificaciones |

#### Campos Específicos del Canal

**Campos Añadidos por el Experience API:**

| Campo | Tipo | Descripción | Razón |
|-------|------|-------------|-------|
| `DirIp` | string | Dirección IP del cliente | Auditoría y seguridad - registro de origen de modificaciones |
| `x-*` headers | object | Headers FUA de contexto | Propagación de información de sesión al Process API |

**Campos del Payload Original:**
- El Experience API mantiene el payload original del cliente sin modificaciones
- Solo añade campos de contexto adicionales al objeto raíz

### Parámetros de Entrada

#### Headers (Requeridos)

| Header | Tipo | Requerido | Descripción |
|--------|------|-----------|-------------|
| `Content-Type` | string | Sí | Debe ser `application/json` |
| `Authorization` | string | Sí | Token OAuth 2.0: `Bearer {token}` |
| `x-client-id` | string | Sí | ID del cliente en Salesforce |
| `x-client-doc-number` | string | Sí | Documento de identidad del cliente |
| `x-interaction-id` | string | No | ID de correlación para trazabilidad |
| `x-access-token` | string | Sí | Token de acceso de la sesión |
| `x-language` | string | Sí | Idioma preferido (es, ca, eu, gl) |
| `x-trading-company` | string | No | Comercializadora (imp/newco) |
| `x-user-id` | string | No | ID del usuario autenticado |
| `x-client-code` | string | No | Código de cliente alternativo |

#### Body (JSON)

**Estructura General:**

```json
{
  "operationType": "string",
  "client": {
    // Objeto client según tipo de operación
  }
}
```

**Campos del Body:**

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `operationType` | string | Sí | Tipo de modificación a realizar (ver tabla de tipos) |
| `client` | object | Sí | Objeto con datos del cliente según operationType |

**Tipos de operationType Soportados:**

Para el detalle completo de cada `operationType` y su schema específico, consultar:
📖 **Process API `POST /modificaciones` - Sección "Subflujos Dinámicos Disponibles"**

**Tipos principales para actualización de clientes:**

| operationType | Propósito | Datos que Modifica |
|---------------|-----------|-------------------|
| `account` | **Actualizar datos del titular** | Nombre, apellidos, nacionalidad del Account |
| `contacto` | **Modificar datos de contacto** | Teléfono, email, canal preferente, horario |
| `gdpr` | **Preferencias de privacidad** | Consentimientos GDPR, comunicaciones |
| `language` | **Cambiar idioma** | Idioma preferido del cliente |
| `areacliente` | **Configuración área cliente** | Preferencias del portal |

**⚠️ Diferencia Importante:**
- **`account`**: Modifica datos del **titular** (persona/empresa) → Account en Salesforce
- **`contacto`**: Modifica datos de **contacto** (cómo comunicarse) → Contact en Salesforce

### Respuestas

#### 200 OK - Modificación Exitosa

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Ejecución correcta"
  }
}
```

#### 400 Bad Request - Payload Inválido

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Payload inválido: operationType es requerido"
  }
}
```

#### 401 Unauthorized - Token Inválido

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Token de autenticación inválido o expirado"
  }
}
```

#### 500 Internal Server Error - Error en Process API

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al procesar modificación en Salesforce"
  }
}
```

### Códigos de Estado HTTP

| Código | Descripción | Cuándo Ocurre |
|--------|-------------|---------------|
| **200** | OK | Modificación procesada exitosamente |
| **400** | Bad Request | Payload inválido o campos requeridos faltantes |
| **401** | Unauthorized | Token OAuth 2.0 inválido o expirado |
| **500** | Internal Server Error | Error en Process API o System API |
| **503** | Service Unavailable | Process API no disponible |

### Validaciones

#### Validaciones de Entrada

1. **Validación de Headers OAuth 2.0:**
   - Token `Authorization` debe estar presente y ser válido
   - Headers `x-client-id` y `x-access-token` son obligatorios

2. **Validación de Payload:**
   - Campo `operationType` es obligatorio
   - Campo `client` debe ser un objeto válido

3. **Validación de IP:**
   - Se extrae automáticamente del `attributes.remoteAddress`
   - Formato: Se elimina el prefijo "/" y se toma solo la IP

### Autenticación

- **Tipo:** OAuth 2.0 con role-based access control
- **Ubicación:** Header `Authorization: Bearer {token}`
- **Validación:** Gestionada por API Manager de Anypoint Platform
- **Roles Soportados:** Customer (acceso estándar)

### Timeouts

| Tipo | Valor | Descripción |
|------|-------|-------------|
| **Request Timeout** | 90 segundos | Timeout total del endpoint (heredado del Process API) |
| **Connection Timeout** | 30 segundos | Timeout de conexión al Process API |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP PUT Request]) --> LogStart[Log Request<br/>Kinesis Logger]
    LogStart --> Transform[Transform Payload<br/>Add FUA Headers + IP]
    Transform --> CallProcess[HTTP Request<br/>POST /modificaciones<br/>Process API Newco]
    CallProcess --> Response[Return Response<br/>from Process API]
    Response --> End([HTTP Response])
    
    CallProcess -->|Error| ErrorHandler[Error Handler]
    ErrorHandler --> TransformError[Transform Error<br/>codResult: 0001]
    TransformError --> LogError[Log Error]
    LogError --> End
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style Transform fill:#fff3e0,stroke:#f57c00
    style CallProcess fill:#ffe1f5,stroke:#c2185b
    style ErrorHandler fill:#ffebee,stroke:#c62828
```

### Transformaciones de Canal

**Transformación 1: Enriquecimiento de Payload**

| Campo Original (Request) | Campo Añadido (Process API) | Transformación |
|--------------------------|----------------------------|----------------|
| `payload` | `payload` | Pass-through completo |
| `vars.fuaHeaders` | Headers FUA | Merge con payload original |
| `attributes.remoteAddress` | `DirIp` | Extracción de IP (elimina "/" y ":port") |

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

**Explicación:**
1. **`payload ++`**: Mantiene todo el payload original del cliente
2. **`vars.fuaHeaders ++`**: Añade headers de contexto FUA (x-client-id, x-user-id, etc.)
3. **`DirIp`**: Extrae IP limpia del formato "/192.168.1.1:12345" → "192.168.1.1"

### Patrones de Diseño

**Patrón:** API Gateway + Header Enrichment

**Justificación:** 
- Separa responsabilidades: Experience API maneja contexto de canal, Process API maneja lógica de negocio
- Facilita reutilización del Process API por múltiples canales
- Centraliza enriquecimiento de contexto en una sola capa
- Simplifica frontend: no necesita conocer headers internos

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente<br/>Portal FUA
    participant E as Experience API<br/>PUT /clients
    participant P as Process API<br/>POST /modificaciones
    participant S as System API<br/>Salesforce
    
    C->>E: PUT /clients<br/>{operationType, client}
    activate E
    
    Note over E: Log Request<br/>(Kinesis)
    
    Note over E: Transform:<br/>payload + fuaHeaders + DirIp
    
    E->>P: POST /modificaciones<br/>{enriched payload}
    activate P
    
    Note over P: Router dinámico<br/>según operationType
    
    P->>S: POST /modificaciones/{tipo}<br/>{transformed data}
    activate S
    
    Note over S: Update Salesforce<br/>Account/Contact/GDPR
    
    S-->>P: {success: true}
    deactivate S
    
    P-->>E: {codResult: "0000"}
    deactivate P
    
    E-->>C: 200 OK<br/>{result}
    deactivate E
```

#### 2. Flujo con Error de Negocio

```mermaid
sequenceDiagram
    participant C as Cliente<br/>Portal FUA
    participant E as Experience API<br/>PUT /clients
    participant P as Process API<br/>POST /modificaciones
    
    C->>E: PUT /clients<br/>{operationType: "invalid"}
    activate E
    
    Note over E: Log Request
    
    Note over E: Transform Payload
    
    E->>P: POST /modificaciones<br/>{enriched payload}
    activate P
    
    Note over P: Router:<br/>operationType no encontrado
    
    P-->>E: 400 Bad Request<br/>{MODIFICATION_NOT_FOUND}
    deactivate P
    
    Note over E: Error Handler:<br/>Transform Error
    
    E-->>C: 400 Bad Request<br/>{codResult: "0001"}
    deactivate E
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant C as Cliente<br/>Portal FUA
    participant E as Experience API<br/>PUT /clients
    participant P as Process API<br/>POST /modificaciones
    
    C->>E: PUT /clients<br/>{operationType, client}
    activate E
    
    Note over E: Log Request
    
    Note over E: Transform Payload
    
    E->>P: POST /modificaciones
    activate P
    
    Note over P: ❌ Connection Timeout<br/>or Service Unavailable
    
    P--xE: HTTP Error
    deactivate P
    
    Note over E: Error Handler:<br/>Catch Exception
    
    Note over E: Transform Error:<br/>codResult: "0001"
    
    Note over E: Log Error
    
    E-->>C: 500 Internal Server Error<br/>{codResult: "0001"}
    deactivate E
```

### Optimizaciones del Canal

**1. Logging Estructurado:**
- Logs enviados a AWS Kinesis para análisis centralizado
- Incluye: método, URI, correlationId, payload
- Categoría: `kinesisLogger` para filtrado

**2. Manejo de Errores Robusto:**
- Error handler global captura todas las excepciones
- Transforma errores a formato estándar con `codResult: "0001"`
- Logs de error para troubleshooting

**3. Pass-Through Eficiente:**
- Mínima transformación de datos
- Delegación de lógica compleja al Process API
- Respuesta directa sin post-procesamiento

### Manejo de Errores

#### Estrategia de Manejo de Errores

1. **Error Handler Global:**
   - Captura todos los errores del flow
   - Transforma a formato estándar de respuesta
   - Registra error en logs

2. **Formato de Error Estándar:**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "{error.description}"
  }
}
```

3. **Propagación de Errores:**
   - Errores del Process API se propagan al cliente
   - Se mantiene el código de error original cuando es posible
   - Se añade contexto adicional en logs

#### Códigos de Error

| Código | Origen | Descripción | Acción Recomendada |
|--------|--------|-------------|-------------------|
| `0000` | Success | Operación exitosa | N/A |
| `0001` | Experience API | Error genérico | Revisar logs y mensaje de error |
| `MODIFICATION_NOT_FOUND` | Process API | Tipo de operación no soportado | Verificar operationType |
| `APEX_ERROR_*` | System API | Error en Salesforce | Contactar soporte técnico |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Actualización de Datos del Titular (operationType: "account")

**Caso de Uso:** Cliente cambia su nombre tras matrimonio

#### cURL

```bash
curl -X PUT "https://e-fua-dev.de-c1.cloudhub.io/fua/api/clients" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-client-id: 0011234567890ABC" \
  -H "x-client-doc-number: 12345678A" \
  -H "x-access-token: access_token_here" \
  -H "x-language: es" \
  -d '{
    "operationType": "account",
    "client": {
      "firstName": "María",
      "lastName": "García",
      "secondLastName": "López",
      "nationality": "ES"
    }
  }'
```

**Nota:** Este operationType actualiza el **Account** (titular) en Salesforce, NO los datos de contacto.

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Ejecución correcta"
  }
}
```

### Ejemplo 2: Modificación de Datos de Contacto (operationType: "contacto")

**Caso de Uso:** Cliente actualiza su teléfono y email tras mudanza

#### JavaScript (Fetch API)

```javascript
const response = await fetch('https://e-fua-dev.de-c1.cloudhub.io/fua/api/clients', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
    'x-client-id': '0011234567890ABC',
    'x-client-doc-number': '12345678A',
    'x-access-token': 'access_token_here',
    'x-language': 'es'
  },
  body: JSON.stringify({
    operationType: 'contacto',
    client: {
      phone: '+34912345678',
      mobile: '+34612345678',
      email: 'nuevo.email@example.com',
      preferredContactMethod: 'Email',
      preferredContactTime: '01'
    }
  })
});

const data = await response.json();
console.log('Datos de contacto actualizados:', data);
```

**Nota:** Este operationType actualiza el **Contact** (datos de contacto) en Salesforce, NO los datos del titular.

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Datos de contacto actualizados correctamente"
  }
}
```

### Ejemplo 3: Cambio de Idioma Preferido

#### Python

```python
import requests

url = 'https://e-fua-dev.de-c1.cloudhub.io/fua/api/clients'
headers = {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
    'x-client-id': '0011234567890ABC',
    'x-client-doc-number': '12345678A',
    'x-access-token': 'access_token_here',
    'x-language': 'es'
}
payload = {
    'operationType': 'language',
    'client': {
        'preferredLanguage': 'ca'  # Cambiar a catalán
    }
}

response = requests.put(url, json=payload, headers=headers)
print(response.json())
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Idioma actualizado correctamente"
  }
}
```

### Ejemplo 4: Modificación de Preferencias GDPR

#### cURL

```bash
curl -X PUT "https://e-fua-dev.de-c1.cloudhub.io/fua/api/clients" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-client-id: 0011234567890ABC" \
  -H "x-client-doc-number: 12345678A" \
  -H "x-access-token: access_token_here" \
  -H "x-language: es" \
  -d '{
    "operationType": "gdpr",
    "client": {
      "marketingConsent": false,
      "dataProcessingConsent": true,
      "thirdPartyDataSharing": false
    }
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Preferencias de privacidad actualizadas"
  }
}
```

### Ejemplo 5: Error - Tipo de Operación No Soportado

#### JavaScript (Fetch API)

```javascript
const response = await fetch('https://e-fua-dev.de-c1.cloudhub.io/fua/api/clients', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
    'x-client-id': '0011234567890ABC',
    'x-client-doc-number': '12345678A',
    'x-access-token': 'access_token_here',
    'x-language': 'es'
  },
  body: JSON.stringify({
    operationType: 'invalidType',  // Tipo no soportado
    client: {}
  })
});

const data = await response.json();
console.error('Error:', data);
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "La modificacion no se ha encontrado"
  }
}
```

### Ejemplo 6: Error - Token Inválido

#### Python

```python
import requests

url = 'https://e-fua-dev.de-c1.cloudhub.io/fua/api/clients'
headers = {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer invalid_token',  # Token inválido
    'x-client-id': '0011234567890ABC',
    'x-client-doc-number': '12345678A',
    'x-access-token': 'access_token_here',
    'x-language': 'es'
}
payload = {
    'operationType': 'account',
    'client': {}
}

response = requests.put(url, json=payload, headers=headers)
print(f"Status: {response.status_code}")
print(response.json())
```

#### Respuesta (401 Unauthorized)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Token de autenticación inválido o expirado"
  }
}