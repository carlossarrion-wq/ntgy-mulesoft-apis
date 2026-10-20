---
api: "ntgy-fua-s-ac-fua"
sidebar_position: 32
sidebar_label: "GET /login"
description: "Autenticación OAuth 2.0 para acceso a servicios del Área Cliente de Naturgy mediante credenciales de usuario"
keywords: [login, oauth, authentication, token, area-cliente, naturgy, oauth2, password-grant]
tags: [Authentication, System API, OAuth 2.0, AC, Security]
---

# GET /login

## 📌 Descripción General

### Propósito del Endpoint

El endpoint `/login` proporciona autenticación OAuth 2.0 utilizando el flujo **Resource Owner Password Credentials Grant** para acceder a los servicios del Área Cliente (AC) de Naturgy. Este endpoint actúa como proxy hacia el servicio de autenticación OAuth de AC, validando credenciales de usuario y retornando tokens de acceso.

El endpoint soporta:
- **Autenticación de Usuarios**: Validación de credenciales (username/password) contra el sistema AC
- **Generación de Tokens**: Emisión de access_token y refresh_token para sesiones autenticadas
- **Integración OAuth 2.0**: Implementación estándar del flujo Password Grant
- **Respuesta Estandarizada**: Formato de respuesta consistente con códigos de resultado

### Tipo de API

**Capa:** System API (API-Led Connectivity)

**Función:** Integración directa con el servicio OAuth 2.0 del Área Cliente (AC) de Naturgy para autenticación de usuarios y generación de tokens de acceso.

### Patrón de Integración

**Tipo:** Request-Reply Síncrono con HTTP

**Descripción:** El endpoint recibe credenciales OAuth 2.0 como query parameters, las reenvía al servicio de autenticación AC mediante HTTP GET, procesa la respuesta del servicio OAuth y retorna un resultado estandarizado con el token de acceso o información de error.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    Client[Cliente<br/>Web/Móvil/Sistema]
    E[Experience API<br/>ntgy-fua-e-*]
    P[Process API<br/>ntgy-fua-p-*]
    S[System API<br/>ntgy-fua-s-ac-fua<br/>GET /login]
    AC[Servicio AC OAuth<br/>rov.intranet.gasnatural.com<br/>/ovh-web/oauth/token]
    
    Client -->|Credenciales| E
    E -->|Request| P
    P -->|OAuth Request| S
    S -->|HTTP GET| AC
    AC -->|Token Response| S
    S -->|Standardized Response| P
    P -->|Token| E
    E -->|Access Token| Client
    
    style S fill:#e1f5ff,stroke:#01579b,stroke-width:3px
    style AC fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style Client fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Autenticación de Clientes Finales**
   - Los clientes de Naturgy acceden al portal web o aplicación móvil
   - Ingresan sus credenciales (email/username y password)
   - El sistema valida las credenciales y genera un token de sesión
   - **Beneficio:** Acceso seguro y personalizado a información de contratos, facturas y consumos

2. **Autenticación de Agentes de Ventas**
   - Los agentes del Frontal de Ventas (FUV) inician sesión en el sistema
   - Se validan sus credenciales corporativas
   - Se genera un token con permisos específicos de agente
   - **Beneficio:** Control de acceso basado en roles y trazabilidad de operaciones

3. **Integración de Sistemas Externos**
   - Sistemas de terceros autorizados requieren acceso a datos de AC
   - Utilizan credenciales de servicio (service accounts)
   - Obtienen tokens para consumir APIs de Naturgy
   - **Beneficio:** Integración segura con partners y sistemas externos

### Contexto Empresarial

- **Dominio:** Seguridad y Gestión de Acceso
- **Sector:** Energía - Utilities
- **División:** Naturgy España - Área Cliente (AC)
- **Proceso de Negocio:** Autenticación y Autorización de Usuarios
- **Sistema Backend:** Área Cliente (AC) - Servicio OAuth 2.0

### Stakeholders

- **Clientes Finales**: Necesitan acceso seguro a su información personal, contratos y facturas
- **Agentes de Ventas**: Requieren autenticación para operar el Frontal de Ventas
- **Desarrolladores de Aplicaciones**: Necesitan un mecanismo estándar de autenticación para integrar aplicaciones
- **Equipo de Seguridad**: Responsables de garantizar la seguridad del acceso a datos sensibles
- **Equipo de Soporte**: Necesitan visibilidad de intentos de autenticación para troubleshooting

### Valor de Negocio

- **Seguridad Centralizada**: Punto único de autenticación para todos los canales (web, móvil, APIs)
- **Experiencia de Usuario Consistente**: Mismo proceso de login en todos los puntos de contacto
- **Trazabilidad**: Registro completo de intentos de autenticación para auditoría y seguridad
- **Escalabilidad**: Capacidad de soportar múltiples canales y sistemas consumidores
- **Cumplimiento Normativo**: Implementación de estándares OAuth 2.0 para seguridad

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- **Process APIs** de autenticación y gestión de sesiones
- **Experience APIs** de portales web y aplicaciones móviles
- **Frontal de Ventas (FUV)** para autenticación de agentes
- **Sistemas de Call Center** para validación de identidad
- **Integraciones de terceros** autorizados (con credenciales de servicio)

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/login` |
| **Development** | `https://ntgy-fua-s-ac-fua-dev.eu-central-1.cloudhub.io` | `https://ntgy-fua-s-ac-fua-dev.eu-central-1.cloudhub.io/api/login` |
| **Test** | `https://ntgy-fua-s-ac-fua-test.eu-central-1.cloudhub.io` | `https://ntgy-fua-s-ac-fua-test.eu-central-1.cloudhub.io/api/login` |
| **Production** | `https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io` | `https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io/api/login` |

### Método HTTP

```
GET /api/login
```

### Parámetros de Entrada

#### Headers HTTP

| Header | Tipo | Requerido | Descripción | Ejemplo |
|--------|------|-----------|-------------|---------|
| `Accept` | string | ❌ No | Tipo de contenido esperado en response | `application/json` |

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `username` | string | ✅ Sí | Identificador de usuario para OAuth2 (email o username) | `"usuario@naturgy.com"` |
| `password` | string | ✅ Sí | Contraseña del usuario | `"********"` |
| `client_id` | string | ✅ Sí | Identificador de cliente OAuth 2.0 | `"fua_client"` |
| `client_secret` | string | ✅ Sí | Secreto de cliente OAuth 2.0 | `"********"` |
| `grant_type` | string | ✅ Sí | Tipo de verificación OAuth 2.0 (debe ser "password") | `"password"` |

**Nota sobre grant_type:** El único valor válido es `"password"`, correspondiente al flujo **Resource Owner Password Credentials Grant** de OAuth 2.0.

### Respuestas

#### Respuesta Exitosa (200 OK)

Cuando las credenciales son válidas, el servicio AC retorna un token de acceso:

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "scope": "read write"
}
```

**Campos de la respuesta:**
- `result.codResult`: Código de resultado ("0000" = éxito)
- `result.msgResult`: Mensaje descriptivo del resultado
- `access_token`: Token JWT para autenticar requests subsecuentes
- `token_type`: Tipo de token (siempre "Bearer")
- `expires_in`: Tiempo de expiración del token en segundos (típicamente 3600 = 1 hora)
- `refresh_token`: Token para renovar el access_token cuando expire
- `scope`: Permisos asociados al token

#### Respuesta de Error de Autenticación (200 OK con error)

Cuando las credenciales son inválidas (HTTP 400 o 401 del servicio AC):

```json
{
  "result": {
    "codResult": "2001",
    "msgResult": "Error técnico"
  },
  "error": "invalid_grant",
  "error_description": "Invalid username or password"
}
```

**Códigos de error comunes:**
- `invalid_grant`: Credenciales incorrectas
- `invalid_client`: Client ID o Client Secret inválidos
- `unsupported_grant_type`: Grant type no soportado

#### Respuesta de Error Técnico (500 Internal Server Error)

Cuando ocurre un error en el sistema (timeout, conexión, etc.):

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

### Autenticación y Autorización

- **Tipo de Autenticación del Endpoint:** Client ID & Client Secret (Anypoint Platform)
- **Autenticación del Usuario:** OAuth 2.0 Password Grant
- **Autorización AC:** El usuario debe existir en el sistema AC con credenciales válidas
- **Permisos Requeridos:** 
  - Usuario activo en base de datos AC
  - Credenciales no bloqueadas
  - Cuenta no suspendida

### Timeouts

| Tipo | Valor | Descripción |
|------|-------|-------------|
| **Request Timeout** | 15 segundos | Tiempo máximo de espera para la respuesta del servicio AC |
| **Connection Timeout** | 15 segundos | Tiempo máximo para establecer conexión con AC |

### Códigos de Estado HTTP

| Código | Descripción | Escenario |
|--------|-------------|-----------|
| `200 OK` | Autenticación exitosa o error de negocio | Credenciales válidas o inválidas (ver `codResult`) |
| `400 Bad Request` | Request malformado | Parámetros faltantes o formato incorrecto |
| `500 Internal Server Error` | Error técnico del sistema | Timeout, error de conexión, error interno |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request]) --> LogStart[Log Entrada<br/>s-ac-login.logEntrada]
    LogStart --> PrepareParams[Preparar Query Parameters<br/>DataWeave Inline]
    PrepareParams --> HTTPCall[HTTP Request<br/>GET /oauth/token<br/>AC OAuth Service]
    HTTPCall --> ValidateStatus[Validar Status Code<br/>200, 400, 401]
    ValidateStatus --> LogMiddle[Log Intermedio<br/>payload + attributes]
    LogMiddle --> Transform[Transform Response<br/>DataWeave: Agregar result]
    Transform --> SetVar[Set Variable<br/>httpStatus]
    SetVar --> LogEnd[Log Salida<br/>s-ac-login.logSalida]
    LogEnd --> End([HTTP Response])
    
    HTTPCall -->|Error| ErrorHandler[Error Handler<br/>Global]
    ErrorHandler --> LogError[Log Error<br/>s-ac-login.logErrorGenerico]
    LogError --> ErrorTransform[Transform Error<br/>codResult: -0001]
    ErrorTransform --> ErrorEnd([HTTP 400 Response])
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style HTTPCall fill:#fff3e0,stroke:#e65100,stroke-width:3px
    style ErrorHandler fill:#ffebee,stroke:#c62828,stroke-width:2px
    style End fill:#e8f5e9,stroke:#2e7d32
```

### Detalles del Backend Call

#### Configuración HTTP

**Archivo:** `global.xml` (configuración HTTP_Request_configuration_AC)

**Configuración:**
- **Host:** `rov.intranet.gasnatural.com`
- **Port:** `443` (HTTPS)
- **Protocol:** HTTPS
- **Base Path:** `/ovh-web`
- **Timeout:** 15000ms (15 segundos)

**Endpoint Completo:**
```
https://rov.intranet.gasnatural.com:443/ovh-web/oauth/token
```

#### HTTP Request Operation

**Archivo:** `s-ac-login.xml` (línea 11)

```xml
<http:request method="GET" 
              config-ref="HTTP_Request_configuration_AC" 
              path="${ac.path.login}" 
              sendBodyMode="NEVER">
    <http:query-params><![CDATA[#[output application/java
---
{
    "username" : attributes.queryParams.username,
    "client_secret" : attributes.queryParams.client_secret,
    "grant_type" : attributes.queryParams.grant_type,
    "password" : attributes.queryParams.password,
    "client_id" : attributes.queryParams.client_id
}]]]></http:query-params>
    <http:response-validator>
        <http:success-status-code-validator values="200,400,401" />
    </http:response-validator>
</http:request>
```

**Características:**
- **Método:** GET (según especificación del servicio AC)
- **Send Body Mode:** NEVER (parámetros en query string)
- **Response Validator:** Acepta 200, 400, 401 como respuestas válidas
- **Query Parameters:** Mapeados directamente desde el request entrante

### Transformaciones DataWeave

#### Transformación de Respuesta

**Archivo:** `s-ac-login.xml` (líneas 26-46)

```dataweave
%dw 2.0
output application/json
---
{
    result: {    
        codResult: if(attributes.statusCode==200) "0000" else "2001",
        msgResult: if(attributes.statusCode==200) "No hay error" else "Error técnico" 
    },
    (payload)
}
```

**Lógica de Transformación:**
1. **Evalúa el status code HTTP** del servicio AC
2. **Si es 200:** Agrega `result` con código "0000" (éxito) y mantiene el payload original del servicio AC (que incluye access_token, refresh_token, etc.)
3. **Si es 400/401:** Agrega `result` con código "2001" (error) y mantiene el payload de error del servicio AC
4. **Spread operator `(payload)`:** Incluye todos los campos del response original de AC

#### Variable de Estado HTTP

```dataweave
%dw 2.0
output application/java
---
attributes.statusCode
```

**Propósito:** Almacena el código de estado HTTP en la variable `httpStatus` para uso posterior en el flow.

### Manejo de Errores

#### Error Handler Global

**Archivo:** `s-ac-login.xml` (líneas 48-67)

```xml
<error-handler>
    <on-error-propagate 
        type="HTTP:BAD_GATEWAY, HTTP:BAD_REQUEST, HTTP:CLIENT_SECURITY, 
              HTTP:CONNECTIVITY, HTTP:FORBIDDEN, HTTP:METHOD_NOT_ALLOWED, 
              HTTP:NOT_ACCEPTABLE, HTTP:NOT_FOUND, HTTP:PARSING, 
              HTTP:RETRY_EXHAUSTED, HTTP:SECURITY, HTTP:SERVICE_UNAVAILABLE, 
              HTTP:TIMEOUT, HTTP:TOO_MANY_REQUESTS, HTTP:UNAUTHORIZED, 
              HTTP:UNSUPPORTED_MEDIA_TYPE, EXPRESSION, STREAM_MAXIMUM_SIZE_EXCEEDED">
        
        <logger level="ERROR" message="s-ac-login.logErrorGenerico" />
        
        <ee:transform>
            <ee:message>
                <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "-0001",
        "msgResult": "Error técnico"
    }
}]]></ee:set-payload>
            </ee:message>
            <ee:variables>
                <ee:set-variable variableName="httpStatus"><![CDATA[400]]></ee:set-variable>
            </ee:variables>
        </ee:transform>
    </on-error-propagate>
</error-handler>
```

**Tipos de Errores Manejados:**
- **HTTP:CONNECTIVITY**: Problemas de conexión con el servicio AC
- **HTTP:TIMEOUT**: Timeout en la llamada HTTP (>15 segundos)
- **HTTP:BAD_GATEWAY**: Servicio AC no disponible
- **EXPRESSION**: Errores en expresiones DataWeave
- **Otros errores HTTP**: 4xx y 5xx del servicio AC

**Respuesta de Error Estandarizada:**
- Código: `-0001`
- Mensaje: `"Error técnico"`
- HTTP Status: `400 Bad Request`

### Logging

#### Log de Entrada

**Ubicación:** Línea 10

```xml
<logger level="INFO" 
        message="s-ac-login.logEntrada #[attributes.queryParams]"/>
```

**Registra:** Query parameters del request (con datos sensibles enmascarados por el custom logger)

#### Log Intermedio

**Ubicación:** Línea 25

```xml
<logger level="INFO" 
        message="s-ac-login.log ## payload: #[payload] attributes: #[attributes]"/>
```

**Registra:** Respuesta del servicio AC y atributos HTTP

#### Log de Salida

**Ubicación:** Línea 47

```xml
<logger level="INFO" 
        message="s-ac-login.logSalida ## payload: #[payload] attributes: #[attributes]"/>
```

**Registra:** Respuesta final transformada antes de enviar al cliente

#### Log de Error

**Ubicación:** Línea 50

```xml
<logger level="ERROR" 
        message="s-ac-login.logErrorGenerico" />
```

**Registra:** Errores técnicos capturados por el error handler

### Configuración

#### Propiedades de Configuración

**Archivo:** `ntgy-fua-s-ac-fua-{env}-config.yaml`

```yaml
ac:
  path:
    login: "/oauth/token"
  host: "rov.intranet.gasnatural.com"
  port: "443"
  protocol: "HTTPS"
```

**Variables de Entorno:**
- `${ac.path.login}`: Path del endpoint de login en AC
- Configuración HTTP heredada de `HTTP_Request_configuration_AC`

### Diagrama de Secuencia Detallado

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as System API<br/>ntgy-fua-s-ac-fua
    participant AC as AC OAuth Service<br/>rov.intranet.gasnatural.com
    
    C->>API: GET /api/login?username=user&password=***&...
    activate API
    
    Note over API: Log Entrada<br/>(query params)
    
    API->>API: Preparar Query Parameters<br/>(DataWeave)
    
    API->>AC: GET /ovh-web/oauth/token<br/>?username=user&password=***&...
    activate AC
    
    alt Credenciales Válidas
        AC-->>API: 200 OK<br/>{access_token, refresh_token, ...}
        Note over API: Log Intermedio<br/>(payload + attributes)
        API->>API: Transform Response<br/>Agregar result: {codResult: "0000"}
        API->>API: Set httpStatus = 200
        Note over API: Log Salida<br/>(response final)
        API-->>C: 200 OK<br/>{result: {codResult: "0000"}, access_token, ...}
    else Credenciales Inválidas
        AC-->>API: 400/401<br/>{error: "invalid_grant"}
        Note over API: Log Intermedio<br/>(error payload)
        API->>API: Transform Response<br/>Agregar result: {codResult: "2001"}
        API->>API: Set httpStatus = 400/401
        Note over API: Log Salida<br/>(error response)
        API-->>C: 200 OK<br/>{result: {codResult: "2001"}, error: ...}
    end
    
    deactivate AC
    deactivate API
    
    Note over C,AC: Error Técnico (Timeout, Conexión, etc.)
    
    C->>API: GET /api/login?...
    activate API
    API->>AC: GET /ovh-web/oauth/token?...
    AC--xAPI: Timeout / Error
    Note over API: Error Handler<br/>Log Error
    API->>API: Transform Error<br/>{codResult: "-0001"}
    API-->>C: 400 Bad Request<br/>{result: {codResult: "-0001"}}
    deactivate API
```

### Consideraciones de Seguridad

#### Datos Sensibles

**Enmascaramiento en Logs:**
- El custom logger extension (`ntgy-cli-logger-extension`) enmascara automáticamente:
  - `password`: Reemplazado por `"********"`
  - `client_secret`: Reemplazado por `"********"`
  - `access_token`: Parcialmente enmascarado
  - `refresh_token`: Parcialmente enmascarado

**Ejemplo de Log:**
```json
{
  "timestamp": "2026-07-20T11:30:00Z",
  "level": "INFO",
  "message": "s-ac-login.logEntrada",
  "queryParams": {
    "username": "usuario@naturgy.com",
    "password": "********",
    "client_id": "fua_client",
    "client_secret": "********",
    "grant_type": "password"
  }
}
```

#### Comunicación Segura

- **Protocolo:** HTTPS (TLS 1.2+)
- **Puerto:** 443
- **Certificados:** Validados por el connector HTTP
- **Encriptación:** End-to-end entre API y servicio AC

#### Validación de Respuestas

- **Status Codes Aceptados:** 200, 400, 401
- **Otros códigos:** Tratados como error técnico
- **Timeout:** 15 segundos para prevenir ataques de denegación de servicio

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Autenticación Exitosa con curl

```bash
curl -X GET "https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io/api/login?username=usuario@naturgy.com&password=MiPassword123&client_id=fua_client&client_secret=secret123&grant_type=password" \
  -H "Accept: application/json"
```

**Respuesta (200 OK):**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c3VhcmlvQG5hdHVyZ3kuY29tIiwiaWF0IjoxNjQwOTk1MjAwLCJleHAiOjE2NDA5OTg4MDAsInNjb3BlIjoicmVhZCB3cml0ZSJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c3VhcmlvQG5hdHVyZ3kuY29tIiwiaWF0IjoxNjQwOTk1MjAwLCJleHAiOjE2NDE1OTk5OTl9.dGVzdF9yZWZyZXNoX3Rva2Vu",
  "scope": "read write"
}
```

### Ejemplo 2: Error de Credenciales Inválidas

```bash
curl -X GET "https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io/api/login?username=usuario@naturgy.com&password=PasswordIncorrecto&client_id=fua_client&client_secret=secret123&grant_type=password" \
  -H "Accept: application/json"
```

**Respuesta (200 OK con error):**

```json
{
  "result": {
    "codResult": "2001",
    "msgResult": "Error técnico"
  },
  "error": "invalid_grant",
  "error_description": "Invalid username or password"
}
```

### Ejemplo 3: Error de Client ID Inválido

```bash
curl -X GET "https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io/api/login?username=usuario@naturgy.com&password=MiPassword123&client_id=cliente_invalido&client_secret=secret123&grant_type=password" \
  -H "Accept: application/json"
```

**Respuesta (200 OK con error):**

```json
{
  "result": {
    "codResult": "2001",
    "msgResult": "Error técnico"
  },
  "error": "invalid_client",
  "error_description": "Client authentication failed"
}
```

### Ejemplo 4: Error Técnico (Timeout)

```bash
curl -X GET "https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io/api/login?username=usuario@naturgy.com&password=MiPassword123&client_id=fua_client&client_secret=secret123&grant_type=password" \
  -H "Accept: application/json" \
  --max-time 20
```

**Respuesta (400 Bad Request):**

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

### Ejemplo 5: Integración con JavaScript (Node.js)

```javascript
const axios = require('axios');

async function login(username, password) {
  try {
    const response = await axios.get('https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io/api/login', {
      params: {
        username: username,
        password: password,
        client_id: 'fua_client',
        client_secret: 'secret123',
        grant_type: 'password'
      },
      headers: {
        'Accept': 'application/json'
      },
      timeout: 15000
    });

    if (response.data.result.codResult === '0000') {
      console.log('Login exitoso');
      console.log('Access Token:', response.data.access_token);
      console.log('Expira en:', response.data.expires_in, 'segundos');
      return {
        success: true,
        token: response.data.access_token,
        refreshToken: response.data.refresh_token
      };
    } else {
      console.error('Error de autenticación:', response.data.error_description);
      return {
        success: false,
        error: response.data.error
      };
    }
  } catch (error) {
    console.error('Error técnico:', error.message);
    return {
      success: false,
      error: 'technical_error'
    };
  }
}

// Uso
login('usuario@naturgy.com', 'MiPassword123')
  .then(result => {
    if (result.success) {
      console.log('Token obtenido:', result.token);
    } else {
      console.log('Error:', result.error);
    }
  });
```

### Ejemplo 6: Integración con Python

```python
import requests
from typing import Dict, Optional

def login(username: str, password: str, 
          client_id: str = 'fua_client', 
          client_secret: str = 'secret123') -> Dict:
    """
    Autentica un usuario contra el servicio AC de Naturgy.
    
    Args:
        username: Email o username del usuario
        password: Contraseña del usuario
        client_id: ID del cliente OAuth 2.0
        client_secret: Secreto del cliente OAuth 2.0
    
    Returns:
        Dict con resultado de autenticación
    """
    url = 'https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io/api/login'
    
    params = {
        'username': username,
        'password': password,
        'client_id': client_id,
        'client_secret': client_secret,
        'grant_type': 'password'
    }
    
    headers = {
        'Accept': 'application/json'
    }
    
    try:
        response = requests.get(url, params=params, headers=headers, timeout=15)
        data = response.json()
        
        if data['result']['codResult'] == '0000':
            print('Login exitoso')
            return {
                'success': True,
                'token': data['access_token'],
                'refresh_token': data['refresh_token'],
                'expires_in': data['expires_in']
            }
        else:
            print(f"Error de autenticación: {data.get('error_description', 'Unknown error')}")
            return {
                'success': False,
                'error': data.get('error', 'authentication_failed')
            }
    
    except requests.exceptions.Timeout:
        print('Error: Timeout en la petición')
        return {
            'success': False,
            'error': 'timeout'
        }
    except requests.exceptions.RequestException as e:
        print(f'Error técnico: {str(e)}')
        return {
            'success': False,
            'error': 'technical_error'
        }

# Uso
if __name__ == '__main__':
    result = login('usuario@naturgy.com', 'MiPassword123')
    
    if result['success']:
        print(f"Token obtenido: {result['token'][:50]}...")
        print(f"Expira en: {result['expires_in']} segundos")
    else:
        print(f"Error: {result['error']}")
```

### Ejemplo 7: Uso del Token en Requests Subsecuentes

Una vez obtenido el token, se puede usar para autenticar requests a otros endpoints:

```bash
# 1. Obtener token
TOKEN=$(curl -s -X GET "https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io/api/login?username=usuario@naturgy.com&password=MiPassword123&client_id=fua_client&client_secret=secret123&grant_type=password" \
  -H "Accept: application/json" | jq -r '.access_token')

# 2. Usar token en otro endpoint
curl -X GET "https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io/api/customersByDoc?docNumber=12345678A" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Accept: application/json"
```

### Ejemplo 8: Manejo de Expiración de Token

```javascript
class AuthService {
  constructor(baseUrl, clientId, clientSecret) {
    this.baseUrl = baseUrl;
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.token = null;
    this.refreshToken = null;
    this.expiresAt = null;
  }

  async login(username, password) {
    const response = await axios.get(`${this.baseUrl}/api/login`, {
      params: {
        username,
        password,
        client_id: this.clientId,
        client_secret: this.clientSecret,
        grant_type: 'password'
      }
    });

    if (response.data.result.codResult === '0000') {
      this.token = response.data.access_token;
      this.refreshToken = response.data.refresh_token;
      this.expiresAt = Date.now() + (response.data.expires_in * 1000);
      return true;
    }
    return false;
  }

  isTokenValid() {
    return this.token && Date.now() < this.expiresAt;
  }

  async getValidToken() {
    if (!this.isTokenValid()) {
      // Aquí se implementaría la lógica de refresh token
      throw new Error('Token expirado, se requiere re-autenticación');
    }
    return this.token;
  }
}

// Uso
const authService = new AuthService(
  'https://ntgy-fua-s-ac-fua-pro.eu-central-1.cloudhub.io',
  'fua_client',
  'secret123'
);

await authService.login('usuario@naturgy.com', 'MiPassword123');
const token = await authService.getValidToken();
```
