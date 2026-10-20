---
api: "ntgy-cli-s-ac"
sidebar_position: 30
sidebar_label: "GET /validateToken"
description: "Validación de tokens de autenticación OAuth2 mediante servicio externo de Área Cliente"
keywords: [validateToken, authentication, oauth2, token, validation, security, area-cliente]
tags: [System API, Authentication, Security, REST, OAuth2]
rest_metadata:
  external_service: "Área Cliente OAuth Service"
  endpoint: "https://rovint.intranet.gasnatural.com/ovh-web/v1/oauth/token/access.gas"
  method: "GET"
  authentication: "Token Query Parameter"
  pattern: "Proxy Pass-Through"
---

# GET /validateToken

## 📌 Descripción General

### Propósito del Endpoint

El endpoint `/validateToken` valida tokens de autenticación OAuth2 mediante el servicio externo de Área Cliente de Naturgy. Actúa como un proxy que verifica la validez y vigencia de tokens de acceso, retornando información del usuario asociado si el token es válido.

El endpoint soporta:
- **Validación de Tokens OAuth2**: Verifica autenticidad y vigencia del token de acceso
- **Obtención de Datos de Usuario**: Retorna información del usuario si el token es válido
- **Verificación de Sesión**: Permite comprobar si una sesión de usuario sigue activa
- **Proxy Transparente**: Reenvía la respuesta del servicio de autenticación sin transformaciones

### Tipo de API

**Capa:** System API (API-Led Connectivity)

**Función:** Proxy de validación de tokens hacia el servicio de autenticación de Área Cliente de Naturgy.

### Patrón de Integración

**Tipo:** Request-Reply Síncrono con Proxy Pass-Through

**Descripción:** El endpoint recibe un token como query parameter, lo reenvía al servicio de autenticación externo de Área Cliente, y retorna la respuesta sin transformaciones. El servicio externo valida el token y retorna información del usuario asociado.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    Client[Cliente<br/>Web/Mobile/API]
    S[System API<br/>ntgy-cli-s-ac-impl<br/>GET /validateToken]
    AC[Servicio Externo<br/>Área Cliente OAuth<br/>rovint.intranet.gasnatural.com]
    
    Client -->|Query Param: token| S
    S -->|GET /ovh-web/v1/oauth/token/access.gas| AC
    AC -->|User Data / Error| S
    S -->|Response JSON| Client
    
    style S fill:#e1f5ff,stroke:#01579b,stroke-width:3px
    style AC fill:#fff3e0,stroke:#e65100,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Validación de Sesión de Usuario**
   - Un usuario intenta acceder a recursos protegidos
   - El sistema valida el token antes de permitir el acceso
   - Se verifica que el token no haya expirado
   - **Beneficio:** Seguridad en el acceso a recursos

2. **Verificación de Autenticación en Cada Request**
   - Aplicaciones frontend validan tokens en cada operación crítica
   - Se asegura que el usuario sigue autenticado
   - Se previene el uso de tokens robados o expirados
   - **Beneficio:** Protección contra accesos no autorizados

3. **Obtención de Información de Usuario Actualizada**
   - El sistema necesita datos actuales del usuario autenticado
   - Se valida el token y se obtienen roles/permisos
   - Se actualiza el contexto de seguridad de la aplicación
   - **Beneficio:** Autorización basada en información actualizada

4. **Renovación de Sesión**
   - Antes de renovar un token, se valida el token actual
   - Se verifica que el usuario tiene una sesión válida
   - Se decide si permitir la renovación
   - **Beneficio:** Gestión segura del ciclo de vida de sesiones

### Contexto Empresarial

- **Dominio:** Seguridad y Autenticación
- **Sector:** Energía - Utilities
- **División:** Naturgy España - Área Cliente
- **Proceso de Negocio:** Gestión de Identidad y Acceso (IAM)
- **Sistema Externo:** Servicio OAuth2 de Área Cliente

### Stakeholders

- **Desarrolladores Frontend**: Necesitan validar tokens antes de operaciones sensibles
- **Equipos de Seguridad**: Requieren trazabilidad de validaciones de tokens
- **Usuarios Finales**: Esperan acceso seguro y sin interrupciones
- **Administradores de Sistema**: Monitorean intentos de acceso no autorizados

### Valor de Negocio

- **Seguridad Mejorada**: Validación centralizada de tokens reduce riesgos de seguridad
- **Experiencia de Usuario**: Validación rápida permite acceso fluido a recursos
- **Cumplimiento Normativo**: Trazabilidad de accesos para auditorías de seguridad
- **Arquitectura Desacoplada**: Centraliza la lógica de validación en un único punto

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Aplicaciones web de autoservicio de clientes
- Aplicaciones móviles de Naturgy
- APIs de Process Layer que requieren validación de tokens
- Sistemas de terceros integrados con Naturgy (con autorización)
- Microservicios internos que necesitan verificar autenticación

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8050` | `http://localhost:8050/api/validateToken?token={token}` |
| **Development** | `https://ntgy-cli-s-ac-impl-dev.us-e2.cloudhub.io` | `https://ntgy-cli-s-ac-impl-dev.us-e2.cloudhub.io/api/validateToken?token={token}` |
| **UAT** | `https://ntgy-cli-s-ac-impl-uat.us-e2.cloudhub.io` | `https://ntgy-cli-s-ac-impl-uat.us-e2.cloudhub.io/api/validateToken?token={token}` |
| **Production** | `https://ntgy-cli-s-ac-impl-pro.us-e2.cloudhub.io` | `https://ntgy-cli-s-ac-impl-pro.us-e2.cloudhub.io/api/validateToken?token={token}` |

### Método HTTP

```
GET /api/validateToken
```

### Parámetros de Entrada

#### Headers HTTP

| Header | Tipo | Requerido | Descripción | Ejemplo |
|--------|------|-----------|-------------|---------|
| `Accept` | string | ❌ No | Tipo de contenido esperado en response | `application/json` |

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `token` | string | ✅ Sí | Token de acceso OAuth2 a validar | `"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."` |

**Validaciones:**
- El parámetro `token` es obligatorio
- El token debe ser una cadena no vacía
- El formato del token es validado por el servicio externo

### Respuestas

#### Respuesta Exitosa (200 OK)

**Descripción:** Token válido, retorna información del usuario

> ⚠️ **Importante:** Este endpoint actúa como proxy pass-through hacia el servicio de autenticación de Área Cliente. La respuesta se retorna sin transformaciones. La estructura mostrada está basada en el RAML del proyecto (`validateToken.raml`).

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Token válido"
  },
  "userid": "12345",
  "username": "juan.perez",
  "email": "juan.perez@naturgy.com"
}
```

**Campos de Respuesta (según RAML del proyecto):**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result` | object | Objeto de resultado con código y mensaje |
| `result.codResult` | string | Código de resultado de la validación |
| `result.msgResult` | string | Mensaje descriptivo del resultado |
| `userid` | string | Identificador único del usuario |
| `username` | string | Nombre de usuario |
| `email` | string | Email del usuario |

#### Respuesta de Error - Token Inválido (400 Bad Request)

**Descripción:** Token inválido, expirado o malformado

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

#### Respuesta de Error - Servicio No Disponible (400 Bad Request)

**Descripción:** El servicio de autenticación externo no está disponible

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

### Códigos de Estado HTTP

| Código | Descripción | Escenario |
|--------|-------------|-----------|
| `200` | OK | Token válido, información de usuario retornada |
| `400` | Bad Request | Token inválido, expirado o error técnico |
| `500` | Internal Server Error | Error interno del servidor |

### Autenticación y Autorización

- **Tipo:** Query Parameter (token)
- **Autorización:** El token debe ser válido en el servicio de Área Cliente
- **Permisos Requeridos:** 
  - Token debe haber sido generado por el servicio de autenticación de Área Cliente
  - Token no debe estar expirado
  - Token no debe estar revocado

### Timeouts

| Tipo | Valor | Descripción |
|------|-------|-------------|
| **Request Timeout** | 60 segundos | Tiempo máximo de espera para la respuesta del servicio externo |
| **Connection Timeout** | 10 segundos | Tiempo máximo para establecer conexión con el servicio externo |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request]) --> LogStart[Log Inicio Flow<br/>INFO: token recibido]
    LogStart --> ExtractToken[Extraer Token<br/>Query Parameter]
    ExtractToken --> HTTPRequest[HTTP GET Request<br/>Servicio Área Cliente<br/>Timeout: 60s]
    HTTPRequest --> Transform[Transform Response<br/>Set httpStatus variable]
    Transform --> LogEnd[Log Salida<br/>INFO: payload response]
    LogEnd --> End([HTTP Response])
    
    HTTPRequest -.->|Error| ErrorHandler[Error Handler]
    ErrorHandler --> LogError[Log Error Genérico<br/>INFO: error details]
    LogError --> ErrorTransform[Transform Error<br/>codResult: -0001<br/>httpStatus: 400]
    ErrorTransform --> End
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style HTTPRequest fill:#fff3e0,stroke:#e65100,stroke-width:3px
    style ErrorHandler fill:#ffebee,stroke:#c62828
    style End fill:#e8f5e9,stroke:#2e7d32
```

### Componentes del Flow

#### 1. Flow Principal: `validateTokenFlow`

**Ubicación:** `src/main/mule/s-ac-validateToken.xml` (líneas 9-54)

**Responsabilidades:**
- Recibir request con token como query parameter
- Reenviar token al servicio de autenticación externo
- Retornar respuesta sin transformaciones
- Manejar errores de comunicación

#### 2. Logging de Inicio

**Componente:** Logger (línea 10)

```xml
<logger level="INFO" message='#["Inicio validateTokenFlow: " ++ attributes.queryParams.token]'/>
```

**Propósito:** Registrar el inicio del flow con el token recibido (para trazabilidad)

#### 3. HTTP Request al Servicio Externo

**Componente:** HTTP Request (líneas 11-17)

```xml
<http:request method="GET" 
              config-ref="HTTP_Request_configuration_AC" 
              path="${ac.path.validateToken}" 
              sendBodyMode="NEVER" 
              responseTimeout="60000">
    <http:query-params>
        <![CDATA[#[output application/java
---
{
    "token" : attributes.queryParams.token
}]]]>
    </http:query-params>
</http:request>
```

**Configuración:**
- **Config Reference:** `HTTP_Request_configuration_AC`
- **Path:** `${ac.path.validateToken}` → `/ovh-web/v1/oauth/token/access.gas`
- **Host:** `rovint.intranet.gasnatural.com`
- **Port:** 443 (HTTPS)
- **Method:** GET
- **Response Timeout:** 60 segundos
- **Send Body Mode:** NEVER (no envía body en GET)

**Query Parameters:**
- `token`: Extraído de `attributes.queryParams.token`

#### 4. Transformación de Respuesta

**Componente:** Transform Message (líneas 18-31)

```dataweave
%dw 2.0
output application/json
---
payload
```

**Propósito:** 
- Convertir respuesta a JSON (si no lo es)
- Capturar HTTP status code en variable `httpStatus`
- Pass-through del payload sin modificaciones

#### 5. Logging de Salida

**Componente:** Logger (línea 32)

```xml
<logger level="INFO" message="s-ac-validateToken.logSalida ## payload:#[payload]"/>
```

**Propósito:** Registrar la respuesta completa para auditoría

#### 6. Error Handler

**Componente:** On Error Propagate (líneas 33-53)

**Manejo de Errores:**
- Captura cualquier error (type="ANY")
- Registra error completo con payload, attributes y error details
- Transforma a respuesta estándar de error
- Establece HTTP status 400

**Respuesta de Error:**
```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

### Diagrama de Secuencia Detallado

```mermaid
sequenceDiagram
    participant Client as Cliente
    participant API as System API<br/>validateTokenFlow
    participant AC as Área Cliente<br/>OAuth Service
    
    Client->>API: GET /validateToken?token=xyz
    activate API
    
    API->>API: Log: Inicio validateTokenFlow
    API->>API: Extraer token de query params
    
    API->>AC: GET /ovh-web/v1/oauth/token/access.gas?token=xyz
    activate AC
    
    alt Token Válido
        AC-->>API: 200 OK + User Data
        API->>API: Transform: Set httpStatus
        API->>API: Log: Salida con payload
        API-->>Client: 200 OK + User Data
    else Token Inválido/Expirado
        AC-->>API: 401/403 Unauthorized
        API->>API: Error Handler
        API->>API: Log: Error genérico
        API->>API: Transform: Error response
        API-->>Client: 400 Bad Request + Error JSON
    else Timeout/Error de Red
        AC--xAPI: Timeout (60s)
        API->>API: Error Handler
        API->>API: Log: Error genérico
        API->>API: Transform: Error response
        API-->>Client: 400 Bad Request + Error JSON
    end
    
    deactivate AC
    deactivate API
```

### Configuración del Servicio Externo

#### HTTP Request Configuration

**Archivo:** `src/main/resources/s-ac-local-config.yaml` (líneas 24-31)

```yaml
ac:
  path:
    login: "/ovh-web/oauth/token"
    validateToken: "/ovh-web/v1/oauth/token/access.gas"
  host: "rovint.intranet.gasnatural.com"
  port: "443"
  timeout: "600000"
```

**Parámetros:**
- **host:** Servidor del servicio de Área Cliente
- **port:** 443 (HTTPS)
- **path.validateToken:** Ruta específica para validación de tokens
- **timeout:** 600 segundos (10 minutos) - timeout general de configuración

**URL Completa Construida:**
```
https://rovint.intranet.gasnatural.com:443/ovh-web/v1/oauth/token/access.gas?token={token}
```

### Transformaciones DataWeave

#### Transformación de Query Parameters (Inline)

**Ubicación:** Dentro del HTTP Request (líneas 12-16)

```dataweave
%dw 2.0
output application/java
---
{
    "token" : attributes.queryParams.token
}
```

**Propósito:** Extraer el token del query parameter del request original y pasarlo al servicio externo

#### Transformación de Respuesta (Inline)

**Ubicación:** Transform Message (líneas 20-23)

```dataweave
%dw 2.0
output application/json
---
payload
```

**Propósito:** Asegurar que la respuesta esté en formato JSON (pass-through)

#### Captura de HTTP Status Code

**Ubicación:** Transform Message - Variables (líneas 26-29)

```dataweave
%dw 2.0
output application/java
---
attributes.statusCode
```

**Propósito:** Capturar el código de estado HTTP del servicio externo para retornarlo al cliente

### Manejo de Errores

#### Estrategia de Error Handling

1. **Errores de Comunicación:**
   - Timeout de conexión (60 segundos)
   - Errores de red
   - Servicio no disponible
   - **Acción:** Retornar error técnico genérico (400)

2. **Errores de Validación:**
   - Token inválido
   - Token expirado
   - Token malformado
   - **Acción:** Propagar respuesta del servicio externo

3. **Errores Inesperados:**
   - Cualquier excepción no controlada
   - **Acción:** Retornar error técnico genérico (400)

#### Logging de Errores

**Nivel:** INFO (para todos los errores)

**Información Registrada:**
- Payload del request
- Attributes del request
- Detalles completos del error (error object)

**Ejemplo de Log:**
```
s-ac-validateToken.logErrorGenerico payload:#[payload] ## attributes: #[attributes] ## error: #[error]
```

### Consideraciones de Seguridad

1. **Token en Query Parameter:**
   - ⚠️ Los tokens en query parameters pueden quedar en logs
   - Se recomienda usar HTTPS para proteger el token en tránsito
   - Los logs deben ser protegidos adecuadamente

2. **Timeout Configurado:**
   - 60 segundos previene ataques de denegación de servicio
   - Balancea entre tiempo de respuesta y disponibilidad

3. **Error Handling Genérico:**
   - No expone detalles internos del sistema
   - Previene información sensible en respuestas de error

4. **Logging Completo:**
   - Permite auditoría de intentos de validación
   - Facilita detección de patrones de ataque

### Consideraciones de Performance

1. **Timeout de 60 segundos:**
   - Adecuado para validaciones de token
   - Previene bloqueos prolongados

2. **Sin Transformaciones Complejas:**
   - Pass-through de respuesta minimiza latencia
   - No hay procesamiento pesado de datos

3. **Logging Eficiente:**
   - Logs en nivel INFO (no DEBUG en producción)
   - Información suficiente sin sobrecarga

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Validación Exitosa de Token

#### Request

```bash
curl -X GET "https://ntgy-cli-s-ac-impl-dev.us-e2.cloudhub.io/api/validateToken?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c" \
  -H "Accept: application/json"
```

#### Response (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Token válido"
  },
  "userid": "12345",
  "username": "juan.perez",
  "email": "juan.perez@naturgy.com"
}
```

### Ejemplo 2: Token Inválido

#### Request

```bash
curl -X GET "https://ntgy-cli-s-ac-impl-dev.us-e2.cloudhub.io/api/validateToken?token=invalid_token_12345" \
  -H "Accept: application/json"
```

#### Response (400 Bad Request)

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

### Ejemplo 3: Token Expirado

#### Request

```bash
curl -X GET "https://ntgy-cli-s-ac-impl-dev.us-e2.cloudhub.io/api/validateToken?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.expired_token" \
  -H "Accept: application/json"
```

#### Response (400 Bad Request)

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

### Ejemplo 4: Parámetro Token Faltante

#### Request

```bash
curl -X GET "https://ntgy-cli-s-ac-impl-dev.us-e2.cloudhub.io/api/validateToken" \
  -H "Accept: application/json"
```

#### Response (400 Bad Request)

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

### Ejemplo 5: Uso en JavaScript (Frontend)

```javascript
async function validateToken(token) {
  try {
    const response = await fetch(
      `https://ntgy-cli-s-ac-impl-pro.us-e2.cloudhub.io/api/validateToken?token=${encodeURIComponent(token)}`,
      {
        method: 'GET',
        headers: {
          'Accept': 'application/json'
        }
      }
    );

    if (!response.ok) {
      throw new Error('Token inválido o expirado');
    }

    const userData = await response.json();
    console.log('Usuario validado:', userData);
    
    // Verificar resultado
    if (userData.result && userData.result.codResult === "0000") {
      return userData;
    } else {
      throw new Error(userData.result?.msgResult || 'Error desconocido');
    }

  } catch (error) {
    console.error('Error validando token:', error);
    throw error;
  }
}

// Uso
const token = localStorage.getItem('authToken');
validateToken(token)
  .then(user => {
    console.log('Token válido para usuario:', user.username);
    console.log('User ID:', user.userid);
  })
  .catch(error => {
    console.error('Redirigir a login');
  });
```

### Ejemplo 6: Uso en Python (Backend)

```python
import requests
from typing import Dict, Optional

def validate_token(token: str) -> Optional[Dict]:
    """
    Valida un token de autenticación OAuth2
    
    Args:
        token: Token de acceso a validar
        
    Returns:
        Dict con información del usuario si el token es válido
        None si el token es inválido
    """
    url = "https://ntgy-cli-s-ac-impl-pro.us-e2.cloudhub.io/api/validateToken"
    
    try:
        response = requests.get(
            url,
            params={"token": token},
            headers={"Accept": "application/json"},
            timeout=30
        )
        
        if response.status_code == 200:
            data = response.json()
            # Verificar resultado
            if data.get('result', {}).get('codResult') == '0000':
                return data
            else:
                print(f"Token inválido: {data.get('result', {}).get('msgResult')}")
                return None
        else:
            print(f"Error HTTP: {response.status_code}")
            return None
            
    except requests.exceptions.Timeout:
        print("Timeout validando token")
        return None
    except requests.exceptions.RequestException as e:
        print(f"Error validando token: {e}")
        return None

# Uso
token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
user_data = validate_token(token)

if user_data:
    print(f"Token válido para usuario: {user_data['username']}")
    print(f"User ID: {user_data['userid']}")
    print(f"Email: {user_data['email']}")
else:
    print("Token inválido - redirigir a login")
```

### Ejemplo 7: Integración con Middleware de Autenticación

```javascript
// Express.js Middleware
const axios = require('axios');

async function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'Token no proporcionado' });
  }

  try {
    const response = await axios.get(
      'https://ntgy-cli-s-ac-impl-pro.us-e2.cloudhub.io/api/validateToken',
      {
        params: { token },
        timeout: 30000
      }
    );

    // Agregar información del usuario al request
    req.user = response.data;
    next();

  } catch (error) {
    return res.status(401).json({ error: 'Token inválido o expirado' });
  }
}

// Uso en rutas protegidas
app.get('/api/protected-resource', authMiddleware, (req, res) => {
  res.json({
    message: 'Recurso protegido',
    user: {
      userid: req.user.userid,
      username: req.user.username,
      email: req.user.email
    }
  });
});
```

### Ejemplo 8: Postman Collection

```json
{
  "info": {
    "name": "Validate Token",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Validate Token - Success",
      "request": {
        "method": "GET",
        "header": [
          {
            "key": "Accept",
            "value": "application/json"
          }
        ],
        "url": {
          "raw": "{{baseUrl}}/api/validateToken?token={{validToken}}",
          "host": ["{{baseUrl}}"],
          "path": ["api", "validateToken"],
          "query": [
            {
              "key": "token",
              "value": "{{validToken}}"
            }
          ]
        }
      }
    }
  ]
}