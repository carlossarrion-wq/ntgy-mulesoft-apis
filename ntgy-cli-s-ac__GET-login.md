---
api: "ntgy-cli-s-ac"
sidebar_position: 19
sidebar_label: "GET /login"
description: "Autenticación de usuarios mediante OAuth2 Password Grant contra servicio corporativo de Área Cliente"
keywords: [login, authentication, oauth2, token, area-cliente, password-grant, naturgy]
tags: [Authentication, System API, REST Integration, OAuth2]
rest_integration:
  external_service: "Autenticador Área Cliente"
  protocol: "HTTPS"
  host: "rovint.intranet.gasnatural.com"
  port: 443
  endpoint: "/ovh-web/oauth/token"
  method: "GET"
  timeout: 600000
  authentication_type: "OAuth2 Password Grant"
---

# GET /login

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint proporciona autenticación de usuarios mediante el protocolo OAuth2 Password Grant, actuando como proxy hacia el servicio corporativo de autenticación del Área Cliente de Naturgy. Valida las credenciales del usuario y retorna un token de acceso que permite el acceso a los servicios protegidos.

El endpoint soporta:
- **Validación de credenciales**: Verifica username y password contra el sistema de autenticación corporativo
- **Generación de token OAuth2**: Obtiene access_token y refresh_token para sesiones autenticadas
- **Wrapping de respuesta**: Envuelve la respuesta del servicio externo con códigos de resultado estándar
- **Manejo de errores**: Gestiona errores de autenticación (400, 401) de forma controlada

### Tipo de API

**Capa:** System API (API-Led Connectivity)

**Función:** Integración directa con el servicio de autenticación corporativo (Autenticador de Área Cliente) para validación de credenciales y generación de tokens OAuth2.

### Patrón de Integración

**Tipo:** Request-Reply Síncrono con REST API Externa

**Descripción:** El endpoint recibe las credenciales del usuario como query parameters, las reenvía al servicio de autenticación corporativo mediante HTTPS, y retorna el token OAuth2 envuelto en una estructura de respuesta estándar con códigos de resultado.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    Client[Cliente<br/>Web/Mobile App]
    S[System API<br/>ntgy-cli-s-ac-impl<br/>GET /login]
    Auth[Servicio Externo<br/>Autenticador AC<br/>rovint.intranet.gasnatural.com<br/>/ovh-web/oauth/token]
    
    Client -->|Query Params<br/>username, password<br/>client_id, client_secret| S
    S -->|HTTPS GET<br/>OAuth2 Password Grant| Auth
    Auth -->|OAuth2 Token<br/>access_token, refresh_token| S
    S -->|Response JSON<br/>+ result wrapper| Client
    
    style S fill:#e1f5ff,stroke:#01579b,stroke-width:3px
    style Auth fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style Client fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Inicio de Sesión en Área Cliente**
   - Usuario accede al portal web o aplicación móvil de Naturgy
   - Introduce sus credenciales (username y password)
   - El sistema valida las credenciales y genera un token de sesión
   - Usuario obtiene acceso a su información de contratos, facturas y consumos

2. **Autenticación para APIs Protegidas**
   - Aplicaciones cliente necesitan acceder a endpoints protegidos
   - Obtienen token OAuth2 mediante este endpoint
   - Utilizan el token en header Authorization para llamadas subsecuentes
   - Mantienen sesión activa mediante refresh_token

3. **Integración con Sistemas Externos**
   - Sistemas de terceros autorizados necesitan acceso a datos del cliente
   - Utilizan credenciales de servicio (service account)
   - Obtienen token para operaciones automatizadas
   - Acceden a APIs de Naturgy de forma segura

### Contexto Empresarial

- **Dominio:** Gestión de Identidad y Acceso (IAM)
- **Sector:** Energía - Utilities
- **División:** Naturgy España - Área Cliente
- **Proceso de Negocio:** Autenticación y autorización de usuarios para acceso a servicios digitales
- **Sistema Corporativo:** Autenticador de Área Cliente (AC)

### Stakeholders

- **Clientes finales**: Necesitan acceso seguro a su información de contratos y consumos
- **Desarrolladores de aplicaciones**: Requieren mecanismo de autenticación para integrar con APIs de Naturgy
- **Equipo de seguridad**: Responsables de garantizar acceso seguro y auditable
- **Soporte técnico**: Necesitan visibilidad de problemas de autenticación para asistir a usuarios

### Valor de Negocio

- **Seguridad**: Autenticación centralizada mediante OAuth2, estándar de la industria
- **Experiencia de usuario**: Single Sign-On (SSO) para acceso a múltiples servicios
- **Trazabilidad**: Registro de intentos de autenticación para auditoría y seguridad
- **Escalabilidad**: Arquitectura desacoplada permite escalar autenticación independientemente

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Aplicaciones web del Área Cliente (portales de autoservicio)
- Aplicaciones móviles iOS y Android
- Process APIs que requieren autenticación de usuario
- Sistemas de terceros autorizados (partners, agregadores)
- Herramientas de testing y desarrollo

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8050` | `http://localhost:8050/api/login` |
| **Development** | `https://ntgy-cli-s-ac-impl-dev.us-e2.cloudhub.io` | `https://ntgy-cli-s-ac-impl-dev.us-e2.cloudhub.io/api/login` |
| **QA** | `https://ntgy-cli-s-ac-impl-cc.us-e2.cloudhub.io` | `https://ntgy-cli-s-ac-impl-cc.us-e2.cloudhub.io/api/login` |
| **Pre-Production** | `https://ntgy-cli-s-ac-impl-cl.us-e2.cloudhub.io` | `https://ntgy-cli-s-ac-impl-cl.us-e2.cloudhub.io/api/login` |
| **Production** | `https://ntgy-cli-s-ac-impl-pro.us-e2.cloudhub.io` | `https://ntgy-cli-s-ac-impl-pro.us-e2.cloudhub.io/api/login` |

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
| `username` | string | ✅ Sí | Identificador de usuario para OAuth2 | `"user@naturgy.com"` |
| `password` | string | ✅ Sí | Contraseña del usuario | `"SecurePass123!"` |
| `client_id` | string | ✅ Sí | Identificador de cliente OAuth2 | `"area-cliente-web"` |
| `client_secret` | string | ✅ Sí | Secret de cliente OAuth2 | `"secret123"` |
| `grant_type` | string | ✅ Sí | Tipo de grant OAuth2 (debe ser "password") | `"password"` |

**Nota de Seguridad:** Aunque este endpoint usa GET por compatibilidad con el servicio externo, las credenciales se transmiten en query parameters. **SIEMPRE debe usarse HTTPS** para proteger las credenciales en tránsito.

### Respuestas

#### Respuesta Exitosa (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 3600,
  "scope": "read write"
}
```

**Descripción de campos:**
- `result.codResult`: Código de resultado ("0000" = éxito) - **Verificado en código** (s-ac-login.xml, línea 33)
- `result.msgResult`: Mensaje descriptivo del resultado - **Verificado en código** (s-ac-login.xml, línea 34)
- `access_token`: Token para autenticación en llamadas subsecuentes - **Del servicio externo**
- `token_type`: Tipo de token - **Del servicio externo**
- `refresh_token`: Token para renovar el access_token cuando expire - **Del servicio externo**
- `expires_in`: Tiempo de vida del access_token en segundos - **Del servicio externo**
- `scope`: Permisos asociados al token - **Del servicio externo**

**⚠️ Nota sobre la estructura de respuesta:**
- Los campos `result.codResult` y `result.msgResult` están **verificados en el código fuente** (s-ac-login.xml, líneas 32-35) y son añadidos por el wrapper de MuleSoft
- Los campos OAuth2 (`access_token`, `token_type`, `refresh_token`, `expires_in`, `scope`) provienen del **servicio externo de autenticación corporativo** y son incluidos mediante spread operator (línea 36: `(payload)`)
- Los valores mostrados en los ejemplos son **ilustrativos**. Los valores reales dependen del servicio de autenticación y pueden variar en formato y contenido
- El formato del `access_token` puede ser JWT, UUID u otro formato según la configuración del servicio externo
- El código MuleSoft **no modifica** los campos OAuth2, solo los envuelve con la estructura `result`

#### Respuesta de Error de Autenticación (200 OK con error)

Cuando las credenciales son inválidas o el servicio retorna 400/401, el endpoint retorna 200 OK pero con código de error:

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
- `invalid_grant`: Credenciales inválidas
- `invalid_client`: client_id o client_secret incorrectos
- `unsupported_grant_type`: grant_type no es "password"

#### Respuesta de Error Técnico (400/500)

En caso de error técnico no controlado:

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

### Autenticación y Autorización

- **Tipo:** Este endpoint NO requiere autenticación previa (es el endpoint de autenticación)
- **Credenciales requeridas:** 
  - Username y password del usuario final
  - Client ID y Client Secret de la aplicación cliente
- **Autorización del servicio externo:** El servicio de autenticación corporativo valida las credenciales contra su base de datos de usuarios

### Timeouts

| Tipo | Valor | Descripción |
|------|-------|-------------|
| **Request Timeout** | 600 segundos (10 minutos) | Tiempo máximo de espera para la respuesta del servicio externo |
| **Connection Timeout** | 10 segundos | Tiempo máximo para establecer conexión HTTPS |
| **Connection Idle Timeout** | 600 segundos | Tiempo máximo de inactividad antes de cerrar conexión |

**Nota:** El timeout de 10 minutos es excepcionalmente alto para permitir operaciones de autenticación complejas, pero típicamente las respuestas se reciben en < 2 segundos.

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request]) --> LogEntry[Log Entrada<br/>Query Params]
    LogEntry --> PrepareParams[Prepare Query Params<br/>DataWeave Transform]
    PrepareParams --> HTTPRequest[HTTP Request<br/>GET to External Service<br/>rovint.intranet.gasnatural.com]
    HTTPRequest --> LogResponse[Log Response<br/>Payload + Attributes]
    LogResponse --> Transform[Transform Response<br/>Add result wrapper]
    Transform --> SetStatus[Set httpStatus Variable<br/>Based on statusCode]
    SetStatus --> LogExit[Log Salida<br/>Final Payload]
    LogExit --> End([HTTP Response])
    
    HTTPRequest -.->|Error| ErrorHandler[Error Handler]
    ErrorHandler --> LogError[Log Error Genérico]
    LogError --> ErrorTransform[Transform Error<br/>codResult: -0001]
    ErrorTransform --> SetErrorStatus[Set httpStatus = 400]
    SetErrorStatus --> End
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style HTTPRequest fill:#fff3e0,stroke:#e65100,stroke-width:3px
    style End fill:#e8f5e9,stroke:#2e7d32
    style ErrorHandler fill:#ffebee,stroke:#c62828
```

### Diagrama de Secuencia

```mermaid
sequenceDiagram
    participant Client as Cliente
    participant API as System API<br/>ntgy-cli-s-ac-impl
    participant Auth as Autenticador AC<br/>rovint.intranet.gasnatural.com
    
    Client->>API: GET /api/login?username=user&password=pass&...
    activate API
    
    API->>API: Log entrada (query params)
    API->>API: Prepare query params (DataWeave)
    
    API->>Auth: HTTPS GET /ovh-web/oauth/token
    activate Auth
    
    Auth->>Auth: Validate credentials
    Auth->>Auth: Generate OAuth2 tokens
    
    Auth-->>API: 200 OK + OAuth2 tokens
    deactivate Auth
    
    API->>API: Log response (payload + attributes)
    API->>API: Transform response (add result wrapper)
    API->>API: Set httpStatus variable
    API->>API: Log salida (final payload)
    
    API-->>Client: 200 OK + Wrapped response
    deactivate API
    
    Note over Client,Auth: En caso de error de autenticación
    
    Client->>API: GET /api/login (credenciales inválidas)
    activate API
    
    API->>Auth: HTTPS GET /ovh-web/oauth/token
    activate Auth
    Auth-->>API: 401 Unauthorized + error details
    deactivate Auth
    
    API->>API: Transform response (codResult: 2001)
    API-->>Client: 200 OK + Error wrapped
    deactivate API
```

### Componentes Técnicos

#### 1. HTTP Listener
**Configuración:** `HTTP_Listener_config`
- **Host:** `0.0.0.0`
- **Port:** `${s-ac.http.port}` (8050 en local)
- **Path:** `/api/*`
- **Protocol:** HTTP (HTTPS en CloudHub con load balancer)

#### 2. APIKit Router
**Configuración:** `s-ac-config`
- **RAML:** `s-ac.raml`
- **Flow generado:** `get:\login:s-ac-config`
- **Validación:** Query parameters según RAML

#### 3. HTTP Request (Outbound)
**Configuración:** `HTTP_Request_configuration_AC`

**Conexión:**
- **Protocol:** HTTPS
- **Host:** `${ac.host}` → `rovint.intranet.gasnatural.com`
- **Port:** `${ac.port}` → `443`
- **Path:** `${ac.path.login}` → `/ovh-web/oauth/token`
- **Method:** GET
- **Send Body Mode:** NEVER

**TLS/SSL:**
- **Truststore:** `s-ac-keystore.jks`
- **Password:** `changeit`
- **Type:** JKS
- **Insecure:** true (acepta certificados autofirmados en desarrollo)

**Timeouts:**
- **Response Timeout:** `${ac.timeout}` → `600000` ms (10 minutos)
- **Connection Idle Timeout:** `${ac.timeout}` → `600000` ms

**Response Validator:**
- **Success Status Codes:** `200, 400, 401`
- **Rationale:** 400 y 401 se consideran respuestas válidas (errores de negocio)

#### 4. Transformaciones DataWeave

**Transformación de Query Parameters (líneas 12-20):**
```dataweave
%dw 2.0
output application/java
---
{
    "username": attributes.queryParams.username,
    "client_secret": attributes.queryParams.client_secret,
    "grant_type": attributes.queryParams.grant_type,
    "password": attributes.queryParams.password,
    "client_id": attributes.queryParams.client_id
}
```

**Transformación de Respuesta (líneas 28-38):**
```dataweave
%dw 2.0
output application/json
---
{
    result: {
        codResult: if(attributes.statusCode == 200) "0000" else "2001",
        msgResult: if(attributes.statusCode == 200) "No hay error" else "Error técnico"
    },
    (payload)  // Spread operator: incluye todos los campos del payload original
}
```

**Lógica de códigos:**
- `statusCode == 200` → `codResult: "0000"` (éxito)
- `statusCode != 200` (400, 401) → `codResult: "2001"` (error técnico)

#### 5. Variables de Flow

**httpStatus Variable (líneas 41-44):**
```dataweave
%dw 2.0
output application/java
---
attributes.statusCode
```

Esta variable se usa en el HTTP Response para establecer el status code de la respuesta al cliente.

#### 6. Error Handler

**Tipos de error capturados:**
- `HTTP:BAD_GATEWAY`
- `HTTP:BAD_REQUEST`
- `HTTP:CLIENT_SECURITY`
- `HTTP:CONNECTIVITY`
- `HTTP:FORBIDDEN`
- `HTTP:METHOD_NOT_ALLOWED`
- `HTTP:NOT_ACCEPTABLE`
- `HTTP:NOT_FOUND`
- `HTTP:PARSING`
- `HTTP:RETRY_EXHAUSTED`
- `HTTP:SECURITY`
- `HTTP:SERVICE_UNAVAILABLE`
- `HTTP:TIMEOUT`
- `HTTP:TOO_MANY_REQUESTS`
- `HTTP:UNAUTHORIZED`
- `HTTP:UNSUPPORTED_MEDIA_TYPE`
- `EXPRESSION`
- `STREAM_MAXIMUM_SIZE_EXCEEDED`

**Respuesta de error (líneas 53-61):**
```dataweave
%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "-0001",
        "msgResult": "Error técnico"
    }
}
```

**httpStatus en error:** `400`

### Configuración por Ambiente

#### Local (s-ac-local-config.yaml)
```yaml
s-ac:
  http:
    port: "8050"

ac:
  path:
    login: "/ovh-web/oauth/token"
  host: "rovint.intranet.gasnatural.com"
  port: "443"
  timeout: "600000"
```

#### Otros ambientes
Las configuraciones de `des`, `cc`, `cl`, `pro` siguen la misma estructura pero con valores específicos de cada ambiente.

### Logging

**Puntos de log:**

1. **Log Entrada** (línea 10):
```
s-ac-login.logEntrada #[attributes.queryParams]
```

2. **Log Intermedio** (línea 25):
```
s-ac-login.log ## payload: #[payload] attributes: #[attributes]
```

3. **Log Salida** (línea 47):
```
s-ac-login.logSalida ## payload: #[payload] attributes: #[attributes]
```

4. **Log Error** (línea 50):
```
s-ac-login.logErrorGenerico
```

**Nivel de log:** INFO para operaciones normales, ERROR para excepciones

### Diagrama de Transformación de Datos

```mermaid
graph LR
    subgraph Input
        QP[Query Parameters<br/>username, password<br/>client_id, client_secret<br/>grant_type]
    end
    
    subgraph "HTTP Request"
        REQ[HTTPS GET<br/>rovint.intranet.gasnatural.com<br/>/ovh-web/oauth/token]
    end
    
    subgraph "External Service Response"
        EXT[OAuth2 Response<br/>access_token<br/>refresh_token<br/>expires_in<br/>token_type<br/>scope]
    end
    
    subgraph "DataWeave Transform"
        DW[Add result wrapper<br/>codResult: 0000/2001<br/>msgResult: success/error]
    end
    
    subgraph Output
        OUT[Final Response<br/>result + OAuth2 data]
    end
    
    QP -->|Pass through| REQ
    REQ -->|200/400/401| EXT
    EXT -->|Spread operator| DW
    DW --> OUT
    
    style QP fill:#e3f2fd
    style REQ fill:#fff3e0
    style EXT fill:#f3e5f5
    style DW fill:#e8f5e9
    style OUT fill:#e1f5fe
```

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Autenticación Exitosa con curl

```bash
curl -X GET "http://localhost:8050/api/login?username=user@naturgy.com&password=SecurePass123!&client_id=area-cliente-web&client_secret=secret123&grant_type=password" \
  -H "Accept: application/json"
```

**Respuesta (200 OK):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyQG5hdHVyZ3kuY29tIiwiaWF0IjoxNzA5NTY4MDAwLCJleHAiOjE3MDk1NzE2MDB9.signature",
  "token_type": "Bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyQG5hdHVyZ3kuY29tIiwiaWF0IjoxNzA5NTY4MDAwfQ.signature",
  "expires_in": 3600,
  "scope": "read write"
}
```

### Ejemplo 2: Error de Autenticación (Credenciales Inválidas)

```bash
curl -X GET "http://localhost:8050/api/login?username=user@naturgy.com&password=WrongPassword&client_id=area-cliente-web&client_secret=secret123&grant_type=password" \
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

### Ejemplo 3: Uso del Token en Llamadas Subsecuentes

Una vez obtenido el token, se usa en el header Authorization:

```bash
curl -X GET "http://localhost:8050/api/customersByDoc?docNumber=12345678A" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Accept: application/json"
```

### Ejemplo 4: Integración desde JavaScript (Frontend)

```javascript
async function login(username, password) {
  const params = new URLSearchParams({
    username: username,
    password: password,
    client_id: 'area-cliente-web',
    client_secret: 'secret123',
    grant_type: 'password'
  });

  try {
    const response = await fetch(`https://api.naturgy.com/api/login?${params}`, {
      method: 'GET',
      headers: {
        'Accept': 'application/json'
      }
    });

    const data = await response.json();

    if (data.result.codResult === '0000') {
      // Autenticación exitosa
      localStorage.setItem('access_token', data.access_token);
      localStorage.setItem('refresh_token', data.refresh_token);
      console.log('Login exitoso');
      return {
        success: true,
        token: data.access_token
      };
    } else {
      // Error de autenticación
      console.error('Error de login:', data.result.msgResult);
      return {
        success: false,
        error: data.error_description || data.result.msgResult
      };
    }
  } catch (error) {
    console.error('Error técnico:', error);
    return {
      success: false,
      error: 'Error de conexión'
    };
  }
}

// Uso
login('user@naturgy.com', 'SecurePass123!')
  .then(result => {
    if (result.success) {
      console.log('Token obtenido:', result.token);
      // Redirigir a dashboard
    } else {
      console.error('Login fallido:', result.error);
      // Mostrar mensaje de error al usuario
    }
  });
```

### Ejemplo 5: Integración desde Python (Backend/Scripts)

```python
import requests
from urllib.parse import urlencode

def authenticate(username, password, client_id, client_secret):
    """
    Autentica un usuario y retorna el token OAuth2
    """
    base_url = "https://api.naturgy.com/api/login"
    
    params = {
        'username': username,
        'password': password,
        'client_id': client_id,
        'client_secret': client_secret,
        'grant_type': 'password'
    }
    
    try:
        response = requests.get(
            base_url,
            params=params,
            headers={'Accept': 'application/json'},
            timeout=30
        )
        
        data = response.json()
        
        if data['result']['codResult'] == '0000':
            return {
                'success': True,
                'access_token': data['access_token'],
                'refresh_token': data['refresh_token'],
                'expires_in': data['expires_in']
            }
        else:
            return {
                'success': False,
                'error': data.get('error_description', data['result']['msgResult'])
            }
            
    except requests.exceptions.Timeout:
        return {
            'success': False,
            'error': 'Timeout al conectar con el servicio de autenticación'
        }
    except requests.exceptions.RequestException as e:
        return {
            'success': False,
            'error': f'Error de conexión: {str(e)}'
        }

# Uso
result = authenticate(
    username='user@naturgy.com',
    password='SecurePass123!',
    client_id='area-cliente-web',
    client_secret='secret123'
)

if result['success']:
    print(f"Token obtenido: {result['access_token']}")
    print(f"Expira en: {result['expires_in']} segundos")
    
    # Usar el token en llamadas subsecuentes
    headers = {
        'Authorization': f"Bearer {result['access_token']}",
        'Accept': 'application/json'
    }
    
    # Ejemplo: obtener datos del cliente
    customer_response = requests.get(
        'https://api.naturgy.com/api/customersByDoc',
        params={'docNumber': '12345678A'},
        headers=headers
    )
    print(customer_response.json())
else:
    print(f"Error de autenticación: {result['error']}")
```

### Ejemplo 6: Manejo de Refresh Token

```javascript
async function refreshAccessToken(refreshToken) {
  // Nota: Este endpoint no está implementado en el código actual
  // pero es un patrón común en OAuth2
  const params = new URLSearchParams({
    refresh_token: refreshToken,
    client_id: 'area-cliente-web',
    client_secret: 'secret123',
    grant_type: 'refresh_token'
  });

  const response = await fetch(`https://api.naturgy.com/api/login?${params}`, {
    method: 'GET',
    headers: {
      'Accept': 'application/json'
    }
  });

  const data = await response.json();
  
  if (data.result.codResult === '0000') {
    localStorage.setItem('access_token', data.access_token);
    return data.access_token;
  }
  
  throw new Error('No se pudo renovar el token');
}
```

### Ejemplo 7: Postman Collection

```json
{
  "info": {
    "name": "Naturgy AC - Login",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Login Exitoso",
      "request": {
        "method": "GET",
        "header": [
          {
            "key": "Accept",
            "value": "application/json"
          }
        ],
        "url": {
          "raw": "{{base_url}}/api/login?username={{username}}&password={{password}}&client_id={{client_id}}&client_secret={{client_secret}}&grant_type=password",
          "host": ["{{base_url}}"],
          "path": ["api", "login"],
          "query": [
            {
              "key": "username",
              "value": "{{username}}"
            },
            {
              "key": "password",
              "value": "{{password}}"
            },
            {
              "key": "client_id",
              "value": "{{client_id}}"
            },
            {
              "key": "client_secret",
              "value": "{{client_secret}}"
            },
            {
              "key": "grant_type",
              "value": "password"
            }
          ]
        }
      },
      "response": []
    }
  ],
  "variable": [
    {
      "key": "base_url",
      "value": "http://localhost:8050"
    },
    {
      "key": "username",
      "value": "user@naturgy.com"
    },
    {
      "key": "password",
      "value": "SecurePass123!"
    },
    {
      "key": "client_id",
      "value": "area-cliente-web"
    },
    {
      "key": "client_secret",
      "value": "secret123"
    }
  ]
}
```
