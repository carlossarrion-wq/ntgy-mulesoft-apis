---
api: "ntgy-fua-e-fua"
sidebar_position: 8
sidebar_label: "GET /balanceBattery"
description: "Consulta el saldo de batería del cliente en el programa de alianzas"
keywords:
  - balanceBattery
  - saldo batería
  - alianzas
  - consulta cliente
  - programa fidelización
tags:
  - Experience API
  - Consulta
  - Alianzas
channel: "Aplicación móvil FUA"
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/balanceBattery"
    purpose: "Orquesta la consulta de saldo de batería con AWS API Gateway"
---

# GET /balanceBattery

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite consultar el saldo de batería del cliente en el programa de alianzas de Naturgy. Proporciona información sobre el balance acumulado de puntos o créditos que el cliente tiene disponible en su cuenta de batería virtual.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para canal móvil FUA (Frontal Único de Atención)

### Patrón de Integración

**Tipo:** Pass-Through con Manejo de Errores Especializado

**Descripción:** El Experience API actúa como proxy hacia el Process API, propagando los headers de contexto del usuario y aplicando manejo de errores específico para el canal móvil. No transforma la respuesta exitosa, pero sí adapta los mensajes de error para mejorar la experiencia del usuario.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente<br/>App Móvil FUA]
    E[Experience API<br/>ntgy-fua-e-fua<br/>GET /balanceBattery]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>GET /balanceBattery]
    S[AWS API Gateway<br/>Servicio Alianzas]
    B[Backend<br/>Sistema Batería]
    
    C -->|Request + Headers| E
    E -->|Propagate Headers| P
    P -->|API Key + Client Code| S
    S -->|Query| B
    B -->|Balance Data| S
    S -->|Response| P
    P -->|Response| E
    E -->|Adapted Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style B fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Consulta de Saldo Disponible**
   - El cliente consulta su saldo actual de batería desde la app móvil
   - Permite al usuario conocer cuántos puntos/créditos tiene acumulados
   - Facilita la toma de decisiones sobre uso o canje de puntos

2. **Validación de Participación en Programa**
   - Verifica si el cliente está registrado en el programa de alianzas
   - Identifica clientes no registrados con mensaje específico
   - Permite dirigir al usuario al proceso de registro si no existe

3. **Seguimiento de Beneficios**
   - Proporciona visibilidad del programa de fidelización
   - Permite al cliente hacer seguimiento de sus beneficios acumulados
   - Fomenta el engagement con el programa de alianzas

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Aplicación Móvil FUA

**Características del Canal:**
- **Plataforma**: iOS y Android
- **Dispositivos**: Smartphones y tablets
- **Conectividad**: Requiere conexión a internet (WiFi o datos móviles)
- **Experiencia de Usuario**: Interfaz optimizada para consultas rápidas

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Clientes de Naturgy que participan en el programa de alianzas de batería
- **Clientes Registrados**: Usuarios con cuenta activa en la app móvil FUA

**Contexto de Uso:**
- **Consulta Rápida**: Usuario revisa su saldo desde la app móvil
- **Pre-Canje**: Usuario verifica saldo antes de realizar un canje de puntos
- **Seguimiento Periódico**: Usuario monitorea la evolución de su saldo

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Propagación de Headers**: Se transmiten headers de contexto del usuario (x-client-code, x-user-id, etc.)
- **Manejo de Errores Mejorado**: Mensajes de error adaptados para el usuario móvil
- **Timeout Optimizado**: 60 segundos para garantizar respuesta en condiciones de red variable
- **Caso Especial "Cliente No Existe"**: Respuesta exitosa (200) con mensaje informativo en lugar de error

### 🔗 Endpoints Dependientes

Este Experience API consume los siguientes Process APIs:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/balanceBattery` | GET | Orquesta la consulta de saldo de batería con AWS API Gateway |

**Flujo de Dependencias:**

```mermaid
graph LR
    E[Experience API<br/>GET /balanceBattery]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>GET /balanceBattery]
    A[AWS API Gateway<br/>Servicio Alianzas]
    
    E -->|Query Params| P
    P -->|x-api-key<br/>x-client-code| A
    A -->|Balance Data| P
    P -->|Response| E
    
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:2px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style A fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

### Contexto Empresarial

- **Dominio:** Programas de Fidelización y Alianzas
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Aplicación Móvil FUA
- **Programa:** Alianzas Batería Virtual

### Stakeholders

- **Clientes Finales**: Usuarios de la app móvil que consultan su saldo de batería
- **Equipo de Marketing**: Responsables del programa de alianzas y fidelización
- **Equipo de Producto Digital**: Gestores de la experiencia móvil FUA
- **Soporte al Cliente**: Agentes que asisten a clientes con consultas sobre el programa

### Valor de Negocio

- **Transparencia**: Proporciona visibilidad completa del saldo de batería al cliente
- **Engagement**: Fomenta la participación activa en el programa de alianzas
- **Autoservicio**: Reduce consultas al call center sobre saldos de batería
- **Experiencia Digital**: Mejora la experiencia del usuario en la app móvil
- **Retención**: Contribuye a la fidelización de clientes a través del programa de beneficios

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/balanceBattery` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/balanceBattery` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceBattery` |

### Método HTTP

```
GET /fua/api/balanceBattery
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Optimizado para llamadas a servicios externos (AWS API Gateway) |
| **Rate Limiting** | Según políticas de API Manager | Protección contra uso excesivo |
| **Autenticación** | JWT Token obligatorio | Seguridad y trazabilidad de usuario |

#### Headers Requeridos

**Headers de Contexto FUA (Propagados al Process API):**

| Header | Tipo | Obligatorio | Descripción |
|--------|------|-------------|-------------|
| `x-client-code` | String | ✅ Sí | Código del cliente en el sistema |
| `x-user-id` | String | ⚠️ Condicional | ID del usuario autenticado |
| `x-interaction-id` | String | ⚠️ Condicional | ID de correlación de la interacción |
| `x-interaction-origin` | String | ⚠️ Condicional | Origen de la interacción (ej: "mobile-app") |
| `x-client-doc-number` | String | ⚠️ Condicional | Número de documento del cliente |
| `x-origin` | String | ⚠️ Condicional | Origen de la petición |
| `x-user-role` | String | ⚠️ Condicional | Rol del usuario |
| `traceparent` | String | ⚠️ Condicional | Header de trazabilidad distribuida |
| `x-agent-mail` | String | ⚠️ Condicional | Email del agente (si aplica) |
| `x-language` | String | ⚠️ Condicional | Idioma preferido del usuario |
| `x-contact-id` | String | ⚠️ Condicional | ID de contacto |

**Nota:** Los headers condicionales se propagan solo si están presentes y no son nulos.

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Obligatorio | Descripción | Ejemplo |
|-----------|------|-------------|-------------|---------|
| `x-client-code` | String | ✅ Sí | Código del cliente para consultar el saldo | `"12345678"` |

**Nota:** El parámetro `x-client-code` se envía como query parameter al Process API.

### Respuestas

#### Respuesta Exitosa (200 OK)

**Descripción:** Saldo de batería consultado exitosamente.

**⚠️ IMPORTANTE - Pass-Through Sin Transformación:**

Este endpoint implementa un patrón de **pass-through completo**:
- El **Experience API** no transforma la respuesta
- El **Process API** no transforma la respuesta exitosa
- La estructura proviene directamente de la **Lambda AWS**

Por lo tanto, la estructura exacta de respuesta depende de la implementación de la Lambda en AWS y puede variar sin previo aviso si se actualiza la Lambda.

**Estructura Observada (Ejemplo Típico):**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Consulta exitosa"
  },
  "balance": {
    "amount": 150.50,
    "currency": "EUR",
    "lastUpdate": "2026-10-19T10:30:00Z"
  }
}
```

**Campos Observados (pueden variar):**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | String | Código de resultado ("0000" = éxito) |
| `result.msgResult` | String | Mensaje descriptivo del resultado |
| `balance.amount` | Number | Monto del saldo de batería |
| `balance.currency` | String | Moneda del saldo |
| `balance.lastUpdate` | String (ISO 8601) | Fecha de última actualización del saldo |

**📋 Nota para Desarrolladores:**

Para obtener la estructura definitiva y actualizada:
1. Consultar la documentación de la Lambda AWS en el servicio de Alianzas
2. Realizar pruebas en ambiente DEV/UAT para capturar la respuesta real
3. Contactar al equipo de AWS que mantiene la Lambda si hay dudas

La estructura mostrada aquí es un **ejemplo basado en observaciones** y debe ser validada para casos de uso críticos.

#### Respuesta Cliente No Existe (200 OK)

**Descripción:** El cliente no está registrado en el programa de alianzas.

**Caso Especial:** El Process API detecta el error "Client not found" de la Lambda AWS y lo convierte en una respuesta exitosa con mensaje informativo. Esta transformación SÍ se realiza en el Process API.

**Estructura Garantizada (transformada por Process API):**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No existe cliente"
  }
}
```

**Nota:** A diferencia de la respuesta exitosa normal, esta estructura SÍ está garantizada porque es generada por el Process API, no por la Lambda AWS.

#### Respuesta de Error (400/401/403/404/500)

**Descripción:** Error en la consulta del saldo.

**Estructura:**

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Descripción del error"
  }
}
```

**Códigos HTTP de Error:**

| Código | Descripción | Causa |
|--------|-------------|-------|
| `400` | Bad Request | Parámetros inválidos o faltantes |
| `401` | Unauthorized | Token de autenticación inválido o expirado |
| `403` | Forbidden | Usuario sin permisos para consultar el saldo |
| `404` | Not Found | Recurso no encontrado |
| `500` | Internal Server Error | Error interno del servidor o del backend AWS |

### Autenticación

- **Tipo:** JWT Token (validado por flow `jwt-validation`)
- **Header:** `Authorization: Bearer <token>`
- **Validación:** Realizada antes de procesar la petición

### Timeouts

| Tipo | Valor | Configuración |
|------|-------|---------------|
| **Experience API → Process API** | 60 segundos | `${newco-p-fua-evo.timeout}` |
| **Process API → AWS API Gateway** | Configurado en Process API | `${s-alianzas.timeout}` |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request]) --> JWT[JWT Validation]
    JWT --> SetVars[Set fuaHeaders Variable]
    SetVars --> Router[APIKit Router]
    Router --> FlowRef[Flow: get-alianza-balanceBattery]
    FlowRef --> HTTPReq[HTTP Request to Process API]
    HTTPReq --> Success{Success?}
    Success -->|Yes| Response([HTTP 200 Response])
    Success -->|No| ErrorHandler[Error Handler]
    ErrorHandler --> Transform[Transform Error Message]
    Transform --> SetStatus[Set HTTP Status]
    SetStatus --> ErrorResponse([HTTP Error Response])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style Response fill:#c8e6c9,stroke:#2e7d32
    style ErrorResponse fill:#ffcdd2,stroke:#c62828
    style HTTPReq fill:#fff9c4,stroke:#f57f17
```

### Flujo Principal (Experience API)

**Archivo:** `e-fua.xml` (líneas 639-641)

```xml
<flow name="get:\balanceBattery:e-fua-config" doc:id="1ed7e7a1-a708-416e-93d5-e9a260b03f6b">
    <flow-ref doc:name="get-alianza-balanceBattery" 
              doc:id="6f134193-fa92-4eb6-a71e-83411d161964" 
              name="get-alianza-balanceBattery"/>
</flow>
```

### Subflow de Implementación (Experience API)

**Archivo:** `e-fua.xml` (proporcionado por el usuario)

```xml
<flow name="get-alianza-balanceBattery" doc:id="a9125c67-f9a7-4f13-9745-58d7c1b2b48d">
    <http:request method="GET" 
                  doc:name="ntgy-newco-p-fua-evo" 
                  doc:id="1f67dfc3-d5d5-49da-bc45-65c3424ac4d6" 
                  config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
                  path="/balanceBattery" 
                  responseTimeout="${newco-p-fua-evo.timeout}">
        <http:query-params><![CDATA[#[output application/java
---
vars.fuaHeaders]]]></http:query-params>
    </http:request>
    
    <error-handler>
        <on-error-propagate enableNotifications="true" 
                           logException="true" 
                           doc:name="On Error Propagate" 
                           doc:id="ebf41477-647f-42d8-a3ad-5ae367effd08">
            <!-- Transformación de error -->
        </on-error-propagate>
    </error-handler>
</flow>
```

**Características Clave:**
- **Propagación de Headers:** Los headers de contexto FUA se envían como query parameters al Process API
- **Timeout:** 60 segundos configurado para la llamada al Process API
- **Error Handling:** Manejo especializado de errores con transformación de mensajes

### Flujo del Process API

**Archivo:** `services/alianzaBalanceBattery.xml` (proporcionado por el usuario)

```xml
<sub-flow name="alianzaBalanceBattery" doc:id="f97565df-5f2f-487a-9fff-83ebf4c11d3c">
    <logger level="INFO" 
            doc:name="Input alianza balanceBattery" 
            message="Input alianza balanceBattery: #[attributes.queryParams]"/>
    
    <try doc:name="Try" doc:id="f6323c13-07db-4de4-86cf-7648916a0411">
        <http:request method="${aws.alianzas.methodConsultaSaldo}" 
                      doc:name="Request AWS bateria" 
                      config-ref="HTTP_Request__AWS" 
                      path="${aws.alianzas.pathConsultaSaldo}" 
                      responseTimeout="${s-alianzas.timeout}">
            <http:headers><![CDATA[#[output application/java
---
{
    "x-api-key": p('aws.apiKey'),
    "x-client-code": attributes.queryParams.'x-client-code'
}]]]></http:headers>
        </http:request>
        
        <error-handler>
            <on-error-propagate>
                <choice doc:name="Choice">
                    <when expression='#[error.errorMessage.payload.error == "Client not found"]'>
                        <!-- Caso especial: Cliente no existe -->
                        <ee:transform doc:name='Error continue "no existe cliente"'>
                            <ee:message>
                                <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0000",
        "msgResult": "No existe cliente"
    }
}]]></ee:set-payload>
                            </ee:message>
                        </ee:transform>
                        <raise-error doc:name="Raise error - NTGY:ALIANZAS_CONTINUE" 
                                    type="NTGY:ALIANZAS_CONTINUE" 
                                    description='#[payload.result.msgResult default "No existe cliente"]'/>
                    </when>
                    <otherwise>
                        <!-- Otros errores -->
                        <ee:transform doc:name="control error">
                            <!-- Transformación de error genérico -->
                        </ee:transform>
                        <raise-error doc:name="Raise error - NTGY:ALIANZAS" 
                                    type="NTGY:ALIANZAS" 
                                    description="Error alianza BalanceBattery"/>
                    </otherwise>
                </choice>
            </on-error-propagate>
        </error-handler>
    </try>
    
    <logger level="INFO" 
            doc:name="Output alianza balanceBattery" 
            message="Output alianza balanceBattery: #[payload]"/>
</sub-flow>
```

**Características del Process API:**
- **Método HTTP:** Configurado dinámicamente desde properties (`${aws.alianzas.methodConsultaSaldo}`)
- **Path:** Configurado desde properties (`${aws.alianzas.pathConsultaSaldo}`)
- **Headers AWS:**
  - `x-api-key`: API Key de AWS (desde secure properties)
  - `x-client-code`: Código del cliente (desde query params)
- **Manejo Especial:** Error "Client not found" se convierte en respuesta exitosa

### Transformaciones de Canal

⚠️ **Este Experience API NO transforma la respuesta del Process API.**

**Patrón:** Pass-Through Completo (Experience → Process → Lambda AWS)

**Flujo de Datos:**
1. **Lambda AWS** → Genera la respuesta con estructura propia
2. **Process API** → Pass-through de respuesta exitosa (sin transformación)
3. **Experience API** → Pass-through completo (sin transformación)

**Implicaciones:**
- ✅ **Ventaja:** Menor latencia, sin overhead de transformación
- ⚠️ **Consideración:** La estructura de respuesta depende completamente de la Lambda AWS
- ⚠️ **Riesgo:** Cambios en la Lambda pueden afectar a los consumidores sin previo aviso

**Excepciones al Pass-Through:**
- ❌ **Respuesta Exitosa:** NO se transforma (pass-through completo)
- ✅ **Error "Client not found":** SÍ se transforma en el Process API (200 OK con mensaje)
- ✅ **Otros Errores:** SÍ se transforman en el Experience API (mapeo de códigos HTTP)

**📋 Recomendación para Consumidores:**

Dado que la estructura de respuesta exitosa no está garantizada por contrato:
1. Implementar manejo defensivo de la respuesta
2. Validar la presencia de campos antes de acceder a ellos
3. Tener fallbacks para campos opcionales
4. Monitorear cambios en la estructura mediante logs

Para información sobre la estructura actual de la Lambda AWS, consultar:
📖 **Documentación AWS Lambda del servicio de Alianzas**
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `/balanceBattery`** (para manejo de errores)

### Configuración HTTP

**Experience API → Process API:**

```yaml
# e-fua-pro-config.yaml
newco-p-fua-evo:
  host: "mule-worker-internal-ntgy-newco-p-fua-evo.de-c1.cloudhub.io"
  port: "8091"
  timeout: "60000"  # 60 segundos
```

**Configuración del Conector:**

```xml
<!-- global.xml -->
<http:request-config name="HTTP_FUA_PROCESS_NEWCO_EVO" 
                     basePath="/api" 
                     responseTimeout="${newco-p-fua-evo.timeout}">
    <http:request-connection host="${newco-p-fua-evo.host}" 
                            port="${newco-p-fua-evo.port}" 
                            connectionIdleTimeout="${newco-p-fua-evo.timeout}"/>
</http:request-config>
```

### Patrones de Diseño

**Patrón:** API Gateway con Propagación de Contexto

**Justificación:** 
- Centraliza la autenticación y validación JWT
- Propaga el contexto del usuario (headers) a través de las capas
- Permite manejo de errores específico del canal sin modificar el Process API

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente (App Móvil)
    participant E as Experience API
    participant P as Process API
    participant A as AWS API Gateway
    participant B as Backend Batería
    
    C->>E: GET /balanceBattery<br/>+ JWT Token<br/>+ x-client-code
    Note over E: JWT Validation
    Note over E: Set fuaHeaders
    E->>P: GET /balanceBattery<br/>+ fuaHeaders as query params
    Note over P: Log Input
    P->>A: GET ${pathConsultaSaldo}<br/>+ x-api-key<br/>+ x-client-code
    A->>B: Query Balance
    B-->>A: Balance Data
    A-->>P: 200 OK + Balance
    Note over P: Log Output
    P-->>E: 200 OK + Balance
    E-->>C: 200 OK + Balance
```

#### 2. Flujo con Cliente No Existe

```mermaid
sequenceDiagram
    participant C as Cliente (App Móvil)
    participant E as Experience API
    participant P as Process API
    participant A as AWS API Gateway
    participant B as Backend Batería
    
    C->>E: GET /balanceBattery<br/>+ JWT Token<br/>+ x-client-code
    Note over E: JWT Validation
    E->>P: GET /balanceBattery<br/>+ fuaHeaders
    P->>A: GET ${pathConsultaSaldo}<br/>+ x-api-key<br/>+ x-client-code
    A->>B: Query Balance
    B-->>A: Error: Client not found
    A-->>P: Error Response
    Note over P: Detect "Client not found"
    Note over P: Transform to Success
    P-->>E: 200 OK<br/>{"result": {"codResult": "0000",<br/>"msgResult": "No existe cliente"}}
    E-->>C: 200 OK<br/>Cliente no registrado
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant C as Cliente (App Móvil)
    participant E as Experience API
    participant P as Process API
    participant A as AWS API Gateway
    
    C->>E: GET /balanceBattery<br/>+ JWT Token<br/>+ x-client-code
    Note over E: JWT Validation
    E->>P: GET /balanceBattery<br/>+ fuaHeaders
    P->>A: GET ${pathConsultaSaldo}<br/>+ x-api-key<br/>+ x-client-code
    A-->>P: 500 Internal Server Error
    Note over P: Error Handler
    Note over P: Transform Error
    P-->>E: Error Response
    Note over E: Error Handler
    Note over E: Transform Error Message
    Note over E: Set HTTP Status
    E-->>C: 400/500 Error<br/>{"result": {"codResult": "0001",<br/>"msgResult": "Error description"}}
```

### Manejo de Errores

#### Experience API - Error Handler

**Transformación de Errores:**

```dataweave
%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0001",
        "msgResult": if(error.errorMessage.payload.result.msgResult != null) 
                        error.errorMessage.payload.result.msgResult 
                     else 
                        error.description
    }
}
```

**Mapeo de Códigos HTTP:**

```dataweave
%dw 2.0
output application/java
---
if((error.errorMessage.payload.result.msgResult contains ("400")) or (error.description contains ("400")))
    400
else if((error.errorMessage.payload.result.msgResult contains ("401")) or (error.description contains ("401")))
    401
else if((error.errorMessage.payload.result.msgResult contains ("403")) or (error.description contains ("403")))
    403
else if((error.errorMessage.payload.result.msgResult contains ("404")) or (error.description contains ("404")))
    404
else
    500
```

**Estrategia de Manejo:**
1. Captura el error del Process API
2. Extrae el mensaje de error del payload o de la descripción
3. Determina el código HTTP basándose en el contenido del mensaje
4. Transforma a formato estándar de respuesta

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta Exitosa de Saldo

#### Request con cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceBattery?x-client-code=12345678" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-interaction-id: 550e8400-e29b-41d4-a716-446655440000" \
  -H "x-interaction-origin: mobile-app" \
  -H "x-user-id: user123" \
  -H "x-client-code: 12345678"
```

#### Response (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Consulta exitosa"
  },
  "balance": {
    "amount": 150.50,
    "currency": "EUR",
    "lastUpdate": "2026-10-19T10:30:00Z"
  }
}
```

### Ejemplo 2: Cliente No Registrado en Programa

#### Request con cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceBattery?x-client-code=99999999" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-client-code: 99999999"
```

#### Response (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No existe cliente"
  }
}
```

**Nota:** Este es un caso especial donde el cliente no está registrado en el programa de alianzas. El Process API convierte el error "Client not found" del backend AWS en una respuesta exitosa con mensaje informativo.

### Ejemplo 3: Error de Autenticación

#### Request con cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceBattery?x-client-code=12345678" \
  -H "Authorization: Bearer token_invalido"
```

#### Response (401 Unauthorized)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Token de autenticación inválido o expirado"
  }
}
```

### Ejemplo 4: Parámetro Faltante

#### Request con cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceBattery" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Response (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Parámetro x-client-code es obligatorio"
  }
}
```

### Ejemplo 5: Error del Servicio Backend

#### Request con cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceBattery?x-client-code=12345678" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-client-code: 12345678"
```

#### Response (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error interno del servidor. Por favor, intente más tarde."
  }
}
```

### Ejemplo 6: JavaScript (Aplicación Móvil) - Con Manejo Defensivo

```javascript
// Función para consultar saldo de batería
// ⚠️ IMPORTANTE: Implementa manejo defensivo porque la estructura 
// de respuesta proviene directamente de Lambda AWS sin transformación
async function getBalanceBattery(clientCode, authToken) {
  try {
    const response = await fetch(
      `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceBattery?x-client-code=${clientCode}`,
      {
        method: 'GET',
        headers: {
          'Authorization': `Bearer ${authToken}`,
          'x-interaction-id': generateUUID(),
          'x-interaction-origin': 'mobile-app',
          'x-client-code': clientCode,
          'Content-Type': 'application/json'
        }
      }
    );

    const data = await response.json();

    if (response.ok) {
      // Validar estructura básica de respuesta
      if (!data?.result) {
        console.error('Estructura de respuesta inesperada:', data);
        throw new Error('Estructura de respuesta inválida');
      }

      // Caso especial: Cliente no existe (garantizado por Process API)
      if (data.result.msgResult === "No existe cliente") {
        console.log('Cliente no registrado en programa de alianzas');
        return { registered: false, balance: null };
      }

      // Validar presencia de datos de balance (estructura de Lambda AWS)
      if (data.balance && typeof data.balance.amount === 'number') {
        console.log('Saldo de batería:', data.balance.amount);
        return { 
          registered: true, 
          balance: {
            amount: data.balance.amount,
            currency: data.balance.currency ?? 'EUR',
            lastUpdate: data.balance.lastUpdate ?? new Date().toISOString()
          }
        };
      } else {
        // Estructura inesperada - log para monitoreo
        console.warn('Respuesta exitosa pero estructura de balance inesperada:', data);
        return { 
          registered: true, 
          balance: null,
          rawResponse: data // Guardar respuesta raw para análisis
        };
      }
    } else {
      // Error en la consulta
      const errorMsg = data?.result?.msgResult ?? 'Error desconocido';
      console.error('Error:', errorMsg);
      throw new Error(errorMsg);
    }
  } catch (error) {
    console.error('Error al consultar saldo:', error);
    throw error;
  }
}

// Uso
const clientCode = '12345678';
const authToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';

getBalanceBattery(clientCode, authToken)
  .then(result => {
    if (result.registered) {
      console.log(`Saldo disponible: ${result.balance.amount} ${result.balance.currency}`);
    } else {
      console.log('Invitar al cliente a registrarse en el programa');
    }
  })
  .catch(error => {
    console.error('Error:', error.message);
  });
```

### Ejemplo 7: Python - Con Manejo Defensivo

```python
import requests
import uuid
from typing import Optional, Dict, Any

def get_balance_battery(client_code: str, auth_token: str) -> Optional[Dict[str, Any]]:
    """
    Consulta el saldo de batería del cliente
    
    ⚠️ IMPORTANTE: Implementa manejo defensivo porque la estructura 
    de respuesta proviene directamente de Lambda AWS sin transformación
    
    Args:
        client_code (str): Código del cliente
        auth_token (str): Token JWT de autenticación
        
    Returns:
        dict: Información del saldo o None si hay error
              - registered (bool): Si el cliente está registrado
              - balance (dict|None): Datos del balance si existen
              - raw_response (dict): Respuesta raw si estructura inesperada
    """
    url = "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceBattery"
    
    headers = {
        'Authorization': f'Bearer {auth_token}',
        'x-interaction-id': str(uuid.uuid4()),
        'x-interaction-origin': 'mobile-app',
        'x-client-code': client_code,
        'Content-Type': 'application/json'
    }
    
    params = {'x-client-code': client_code}
    
    try:
        response = requests.get(url, headers=headers, params=params, timeout=60)
        data = response.json()
        
        if response.status_code == 200:
            # Validar estructura básica
            if 'result' not in data:
                print(f'⚠️ Estructura de respuesta inesperada: {data}')
                return {'registered': False, 'balance': None, 'raw_response': data}
            
            # Caso especial: Cliente no existe (garantizado por Process API)
            if data.get('result', {}).get('msgResult') == "No existe cliente":
                print('Cliente no registrado en programa de alianzas')
                return {'registered': False, 'balance': None}
            
            # Validar presencia de datos de balance (estructura de Lambda AWS)
            balance_data = data.get('balance')
            if balance_data and isinstance(balance_data.get('amount'), (int, float)):
                print(f"Saldo de batería: {balance_data['amount']}")
                return {
                    'registered': True,
                    'balance': {
                        'amount': balance_data['amount'],
                        'currency': balance_data.get('currency', 'EUR'),
                        'lastUpdate': balance_data.get('lastUpdate', '')
                    }
                }
            else:
                # Estructura inesperada - log para monitoreo
                print(f'⚠️ Respuesta exitosa pero estructura de balance inesperada: {data}')
                return {
                    'registered': True,
                    'balance': None,
                    'raw_response': data  # Guardar respuesta raw para análisis
                }
        else:
            error_msg = data.get('result', {}).get('msgResult', 'Error desconocido')
            print(f"Error: {error_msg}")
            return None
            
    except requests.exceptions.Timeout:
        print('⏱️ Timeout: La consulta tardó más de 60 segundos')
        return None
    except requests.exceptions.RequestException as e:
        print(f'❌ Error en la petición: {str(e)}')
        return None
    except (KeyError, TypeError) as e:
        print(f'⚠️ Error procesando respuesta: {str(e)}')
        return None

# Uso
client_code = '12345678'
auth_token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'

result = get_balance_battery(client_code, auth_token)

if result and result['registered']:
    balance = result['balance']
    print(f"Saldo disponible: {balance['amount']} {balance['currency']}")
    print(f"Última actualización: {balance['lastUpdate']}")
else:
    print('Cliente no registrado o error en la consulta')
```

---

## 📊 Notas Adicionales

### Consideraciones de Implementación

1. **Timeout Extendido**: El timeout de 60 segundos está configurado para soportar la latencia de AWS API Gateway y el backend de alianzas.

2. **Caso Especial "Cliente No Existe"**: El Process API convierte el error "Client not found" en una respuesta exitosa (200 OK) con mensaje informativo. Esto permite a la aplicación móvil manejar este caso de forma amigable sin mostrar un error técnico al usuario.

3. **Propagación de Headers**: Todos los headers de contexto FUA se propagan como query parameters al Process API, permitiendo trazabilidad completa de la petición.

4. **⚠️ Pass-Through Completo**: El Experience API y Process API actúan como pass-through para respuestas exitosas. La estructura de datos proviene directamente de la Lambda AWS sin transformación ni validación.

5. **⚠️ Estructura de Respuesta No Garantizada**: Dado que no hay transformación, la estructura exacta de la respuesta exitosa depende de la implementación de la Lambda AWS y puede cambiar sin previo aviso. Los consumidores deben implementar manejo defensivo.

6. **Validación de Respuesta**: Se recomienda que los consumidores validen la estructura de respuesta antes de procesarla, especialmente en casos críticos de negocio.

### Mejores Prácticas

1. **Manejo de "No existe cliente"**: La aplicación móvil debe detectar el mensaje "No existe cliente" y ofrecer al usuario la opción de registrarse en el programa de alianzas.

2. **Retry Logic**: Implementar retry logic en el cliente para manejar errores transitorios (500, timeouts).

3. **Caché Local**: Considerar cachear el saldo localmente por un período corto (ej: 5 minutos) para reducir llamadas al backend.

4. **Logging**: Incluir siempre `x-interaction-id` para facilitar la trazabilidad de problemas.

5. **⚠️ Manejo Defensivo de Respuesta**: Dado que la estructura de respuesta proviene directamente de Lambda AWS sin transformación:
   ```javascript
   // ✅ BIEN - Manejo defensivo
   const balance = response?.balance?.amount ?? 0;
   const currency = response?.balance?.currency ?? 'EUR';
   
   // ❌ MAL - Asume estructura fija
   const balance = response.balance.amount;
   ```

6. **⚠️ Validación de Campos**: Validar la presencia de campos críticos antes de usarlos:
   ```javascript
   if (response?.result?.codResult === "0000") {
     if (response?.balance && typeof response.balance.amount === 'number') {
       // Procesar saldo
     } else {
       // Estructura inesperada - log y manejo de error
       console.error('Estructura de respuesta inesperada:', response);
     }
   }
   ```

7. **⚠️ Monitoreo de Cambios**: Implementar logging de la estructura de respuesta para detectar cambios en la Lambda AWS:
   ```javascript
   // Log estructura para monitoreo
   console.log('Response structure:', JSON.stringify(response, null, 2));
   ```

### Troubleshooting

**Problema:** Timeout después de 60 segundos

**Solución:** 
- Verificar conectividad con AWS API Gateway
- Revisar logs del Process API para identificar el punto de fallo
- Contactar al equipo de infraestructura si el problema persiste

**Problema:** Respuesta "No existe cliente" para cliente que debería estar registrado

**Solución:**
- Verificar que el `x-client-code` sea correcto
- Confirmar que el cliente está registrado en el sistema de alianzas
- Revisar logs del backend AWS para más detalles

**Problema:** Error 401 Unauthorized

**Solución:**
- Verificar que el token JWT sea válido y no haya expirado
- Renovar el token si es necesario
- Verificar que el usuario tenga permisos para consultar el saldo

**Problema:** Estructura de respuesta inesperada o campos faltantes

**Solución:**
- ⚠️ **Causa Probable:** La Lambda AWS cambió su estructura de respuesta
- Verificar logs de la aplicación para ver la estructura real recibida
- Contactar al equipo de AWS que mantiene la Lambda de Alianzas
- Actualizar el código cliente para manejar la nueva estructura
- Considerar implementar versionado de API si los cambios son frecuentes

### ⚠️ Nota Final Importante

**Sobre la Estructura de Respuesta:**

Este endpoint implementa un **pass-through completo** sin transformación. La estructura de respuesta exitosa mostrada en esta documentación es un **ejemplo observado**, no un contrato garantizado.

**Recomendaciones:**
1. ✅ Implementar siempre manejo defensivo de la respuesta
2. ✅ Validar la presencia de campos antes de acceder a ellos
3. ✅ Monitorear cambios en la estructura mediante logs
4. ✅ Tener fallbacks para campos opcionales
5. ✅ Consultar la documentación de la Lambda AWS para la estructura definitiva

**Para Obtener la Estructura Definitiva:**
- 📖 Documentación de la Lambda AWS del servicio de Alianzas
- 🧪 Realizar pruebas en ambiente DEV/UAT
- 👥 Contactar al equipo de AWS que mantiene la Lambda

**Última Validación de Estructura:** Pendiente de validación con equipo AWS
