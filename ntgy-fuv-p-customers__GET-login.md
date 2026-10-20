---
api: "ntgy-fuv-p-customers"
sidebar_position: 6
sidebar_label: "GET /login"
description: "Endpoint de autenticación multi-backend que genera tokens OAuth 2.0 mediante estrategia Scatter-Gather con priorización configurable entre AC y AWS Cognito"
keywords: [autenticación, login, OAuth2, token, scatter-gather, AC, Cognito, multi-backend, alta disponibilidad]
tags: [Autenticación, Process API, Orquestación, Alta Disponibilidad]
endpoint_dependencies:
  - method: "GET"
    api_name: "s-ac"
    path: "/login"
    purpose: "Autenticación de usuario en Sistema AC (Área Cliente)"
  - method: "POST"
    api_name: "CognitoAPI"
    path: "/signinuser"
    purpose: "Autenticación de usuario en AWS Cognito"
---

# GET /login

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint implementa un sistema de autenticación multi-backend que orquesta llamadas paralelas a dos sistemas de autenticación diferentes (AC y AWS Cognito) mediante un patrón Scatter-Gather. Genera tokens OAuth 2.0 con estrategia de priorización configurable y fallback automático, garantizando alta disponibilidad del servicio de autenticación.

### Tipo de API

**Capa:** Process API (API-Led Connectivity)

**Función:** Orquestación de autenticación multi-origen con lógica de priorización y resiliencia

### Patrón de Integración

**Tipo:** Orquestación Paralela (Scatter-Gather) con Consolidación Inteligente

**Descripción:** 
- Ejecuta autenticaciones en paralelo contra AC y Cognito
- Manejo de errores independiente por backend
- Consolidación de respuestas con lógica de priorización configurable
- Fallback automático si el backend prioritario falla

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    E[Experience APIs<br/>Portales Web, Apps Móviles, Agentes]
    P[Process API<br/>ntgy-fuv-p-customers<br/>GET /login]
    S1[System API AC<br/>s-ac<br/>GET /login]
    S2[AWS Cognito API<br/>CognitoAPI<br/>POST /signinuser]
    
    E -->|Request OAuth2| P
    P -->|Scatter-Gather| S1
    P -->|Scatter-Gather| S2
    S1 -->|Token AC| P
    S2 -->|Token Cognito| P
    P -->|Token Prioritario| E
    
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:3px
    style S1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Autenticación de Clientes en Portal Web**
   - Usuario ingresa credenciales en portal de autoservicio
   - Sistema valida contra AC o Cognito según disponibilidad
   - Beneficio: Acceso continuo incluso si un sistema falla

2. **Login en Aplicaciones Móviles**
   - App móvil solicita token para sesión de usuario
   - Autenticación transparente con fallback automático
   - Beneficio: Experiencia de usuario sin interrupciones

3. **Autenticación de Agentes de Call Center**
   - Agente inicia sesión en sistema de gestión
   - Validación rápida con alta disponibilidad
   - Beneficio: Productividad sin tiempos de espera por fallos

### 🔗 Endpoints Dependientes

Este Process API consume los siguientes backends:

| Backend | Endpoint | Método | Propósito |
|---------|----------|--------|-----------|
| `s-ac` | `/login` | GET | Autenticación en Sistema AC (Área Cliente) |
| `CognitoAPI` | `/signinuser` | POST | Autenticación en AWS Cognito |

### Lógica de Orquestación

```mermaid
graph LR
    P[Process API<br/>GET /login] --> SG{Scatter-Gather}
    SG --> R1[Ruta 1:<br/>AC Login]
    SG --> R2[Ruta 2:<br/>Cognito Login]
    R1 --> AC[s-ac<br/>GET /login]
    R2 --> CG[CognitoAPI<br/>POST /signinuser]
    AC --> C[Consolidación]
    CG --> C
    C --> PR{Priorización}
    PR --> RES[Token Final]
    
    style P fill:#ffe1f5
    style SG fill:#fff3e0
    style AC fill:#e8f5e9
    style CG fill:#e8f5e9
    style PR fill:#e3f2fd
```

**Pasos del Proceso:**

1. **Scatter-Gather Paralelo**: Ejecuta ambas autenticaciones simultáneamente
2. **Manejo de Errores Independiente**: Cada ruta captura errores sin afectar la otra
3. **Consolidación de Respuestas**: Evalúa ambos resultados
4. **Priorización Configurable**: Selecciona token según propiedad `login_token_prioridad`
5. **Fallback Automático**: Si el prioritario falla, usa el alternativo

### Transformaciones Aplicadas

#### Transformación AC (Línea 28-31)
```dataweave
vars.loginAC ++ { "token_area": "AC" }
```
**Propósito:** Añade identificador de origen del token

#### Transformación Cognito (Línea 96-110)
```dataweave
{
    "result": {
        "codResult": "0000",
        "msgResult": "No hay error"
    },	
   "access_token": vars.loginOmega.AccessToken,
   "token_type":"bearer",
   "refresh_token": vars.loginOmega.RefreshToken,
   "expires_in": 49999,
   "scope": "read write",
   "token_area": "CG"
}
```
**Propósito:** Normaliza respuesta de Cognito al formato estándar OAuth 2.0

#### Consolidación Final (Línea 139-153)
```dataweave
var tokenPrioritario = p('login_token_prioridad')
---
if (tokenPrioritario == 'AC')
	if (vars.loginAC.access_token != null) 
		vars.loginAC
	else
		vars.loginOmega
else
	if (vars.loginOmega.access_token != null) 
		vars.loginOmega
	else
		vars.loginAC
```
**Propósito:** Implementa lógica de priorización con fallback

### Contexto Empresarial

- **Dominio:** Gestión de Identidad y Acceso
- **Sector:** Energía - Utilities
- **División:** Naturgy España - Clientes
- **Proceso de Negocio:** Autenticación y Autorización de Usuarios

### Stakeholders

- **Usuarios Finales**: Clientes que acceden a portales y apps
- **Agentes de Servicio**: Personal de call center que requiere autenticación
- **Desarrolladores de Experience APIs**: Consumen este endpoint para autenticación
- **Equipo de Seguridad**: Define políticas de autenticación y tokens

### Valor de Negocio

- **Alta Disponibilidad**: 99.9%+ uptime mediante redundancia de backends
- **Resiliencia**: Fallback automático sin intervención manual
- **Flexibilidad**: Priorización configurable según estrategia de negocio
- **Experiencia de Usuario**: Autenticación transparente sin errores visibles

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Portales web de autoservicio (FUA Portal)
- Aplicaciones móviles (iOS, Android)
- Sistemas de agentes de call center
- Aplicaciones de terceros autorizados
- Servicios internos que requieren autenticación

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/login` |
| **Development** | `https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io` | `https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/login` |
| **UAT** | `https://ntgy-fuv-p-customers-uat.us-e2.cloudhub.io` | `https://ntgy-fuv-p-customers-uat.us-e2.cloudhub.io/api/login` |
| **Production** | `https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io` | `https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io/api/login` |

### Método HTTP

```
GET /api/login
```

### Parámetros de Entrada

#### Query Parameters (Obligatorios)

| Parámetro | Tipo | Obligatorio | Descripción | Ejemplo |
|-----------|------|-------------|-------------|---------|
| `username` | string | ✅ | Identificador de usuario para OAuth2 | `user@example.com` |
| `password` | string | ✅ | Contraseña del usuario | `SecurePass123!` |
| `client_id` | string | ✅ | Identificador de cliente OAuth2 | `app-client-id` |
| `client_secret` | string | ✅ | Secreto de cliente OAuth2 | `app-client-secret` |
| `grant_type` | string | ✅ | Tipo de verificación OAuth2 | `password` |

**Validaciones:**
- `grant_type` debe ser exactamente `"password"`
- Todos los parámetros son obligatorios
- `username` y `password` se validan contra AC y Cognito

### Respuestas

#### 200 OK - Autenticación Exitosa

**Estructura de Respuesta:**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 49999,
  "scope": "read write",
  "token_area": "AC"
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | string | Código de resultado (`"0000"` = éxito) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |
| `access_token` | string | Token JWT de acceso |
| `token_type` | string | Tipo de token (siempre `"bearer"`) |
| `refresh_token` | string | Token para renovación de sesión |
| `expires_in` | number | Tiempo de expiración en segundos (49999) |
| `scope` | string | Alcance del token (`"read write"`) |
| `token_area` | string | Origen del token (`"AC"` o `"CG"`) |

#### 400 Bad Request - Error de Autenticación

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

**Causas Comunes:**
- Credenciales inválidas en ambos backends
- Parámetros faltantes o incorrectos
- Formato de `grant_type` incorrecto
- Ambos backends fallaron simultáneamente

#### 500 Internal Server Error - Error del Sistema

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

**Causas Comunes:**
- Error de conectividad con backends
- Timeout en llamadas a AC o Cognito
- Error de configuración del sistema

### Autenticación

- **Tipo:** No requiere autenticación previa (este ES el endpoint de autenticación)
- **Salida:** Genera token OAuth 2.0 Bearer para uso en endpoints protegidos

### Timeouts

| Tipo | Valor |
|------|-------|
| **Request Timeout** | 30 segundos |
| **Connection Timeout** | 10 segundos |
| **Scatter-Gather Timeout** | 30 segundos (ambas rutas en paralelo) |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request]) --> LogStart[Log Entrada]
    LogStart --> SG{Scatter-Gather<br/>Paralelo}
    
    SG --> R1[Ruta 1: AC]
    SG --> R2[Ruta 2: Cognito]
    
    R1 --> Try1[Try-Catch AC]
    Try1 --> ReqAC[HTTP GET<br/>s-ac/login]
    ReqAC --> TransAC[Transform:<br/>Add token_area AC]
    TransAC --> ErrAC{Error?}
    ErrAC -->|No| VarAC[Set vars.loginAC]
    ErrAC -->|Sí| ErrTransAC[Error Transform:<br/>codResult -0001]
    ErrTransAC --> VarAC
    
    R2 --> Try2[Try-Catch Cognito]
    Try2 --> ReqCG[HTTP POST<br/>CognitoAPI/signinuser]
    ReqCG --> Choice{AccessToken<br/>válido?}
    Choice -->|Sí| TransOK[Transform OK:<br/>Normalize Response]
    Choice -->|No| TransErr[Transform Error:<br/>codResult -0001]
    TransOK --> VarCG[Set vars.loginOmega]
    TransErr --> VarCG
    
    VarAC --> Consolidate[Consolidar Respuesta]
    VarCG --> Consolidate
    
    Consolidate --> Prior{Priorización<br/>login_token_prioridad}
    Prior -->|AC prioritario| CheckAC{AC token<br/>válido?}
    Prior -->|CG prioritario| CheckCG{CG token<br/>válido?}
    
    CheckAC -->|Sí| UseAC[Usar Token AC]
    CheckAC -->|No| UseCG[Usar Token Cognito]
    CheckCG -->|Sí| UseCG2[Usar Token Cognito]
    CheckCG -->|No| UseAC2[Usar Token AC]
    
    UseAC --> Validate{codResult<br/>== 0000?}
    UseCG --> Validate
    UseCG2 --> Validate
    UseAC2 --> Validate
    
    Validate -->|No| ErrGen[Error Genérico<br/>HTTP 400]
    Validate -->|Sí| LogEnd[Log Salida]
    ErrGen --> LogEnd
    
    LogEnd --> End([HTTP Response])
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style End fill:#e8f5e9,stroke:#2e7d32
    style SG fill:#fff3e0,stroke:#f57c00
    style Prior fill:#e3f2fd,stroke:#1976d2
    style Consolidate fill:#f3e5f5,stroke:#7b1fa2
```

### Transformaciones DataWeave

#### 1. Transformación Request AC (Línea 15-23)
```dataweave
%dw 2.0
output application/java
---
{
	"username" : attributes.queryParams.username,
	"client_secret" : attributes.queryParams.client_secret,
	"grant_type" : attributes.queryParams.grant_type,
	"password" : attributes.queryParams.password,
	"client_id" : attributes.queryParams.client_id
}
```
**Propósito:** Mapea query params a parámetros de AC

#### 2. Transformación Request Cognito (Línea 58-66)
```dataweave
%dw 2.0
output application/json
---
{
  "user_pool_id": Mule::p("cognito.userPoolId"),
  "username" : attributes.queryParams.username,
  "password" : attributes.queryParams.password,
  "client_id": Mule::p("cognito.client_id")  
}
```
**Propósito:** Construye body para API de Cognito con configuración

#### 3. Normalización Respuesta AC (Línea 28-31)
```dataweave
%dw 2.0
output application/json
---
vars.loginAC ++ { "token_area": "AC" }
```
**Propósito:** Añade identificador de origen

#### 4. Normalización Respuesta Cognito (Línea 96-110)
```dataweave
%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0000",
        "msgResult": "No hay error"
    },	
   "access_token": vars.loginOmega.AccessToken,
   "token_type":"bearer",
   "refresh_token": vars.loginOmega.RefreshToken,
   "expires_in": 49999,
   "scope": "read write",
   "token_area": "CG"
}
```
**Propósito:** Convierte respuesta de Cognito a formato OAuth 2.0 estándar

#### 5. Consolidación con Priorización (Línea 139-153)
```dataweave
%dw 2.0
output application/json
var tokenPrioritario = p('login_token_prioridad')
---
if (tokenPrioritario == 'AC')
	if (vars.loginAC.access_token != null) 
		vars.loginAC
	else
		vars.loginOmega
else
	if (vars.loginOmega.access_token != null) 
		vars.loginOmega
	else
		vars.loginAC
```
**Propósito:** Implementa lógica de priorización con fallback automático

### Patrones de Diseño

**Patrón:** Scatter-Gather con Priorización Configurable

**Justificación:** 
- **Alta Disponibilidad**: Redundancia de backends garantiza servicio continuo
- **Performance**: Ejecución paralela reduce latencia total
- **Flexibilidad**: Priorización configurable permite cambios sin redeploy
- **Resiliencia**: Fallback automático sin intervención manual

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito (AC Prioritario)

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API
    participant AC as s-ac
    participant CG as Cognito API
    
    C->>P: GET /login?username=user&password=pass
    Note over P: Scatter-Gather Inicia
    
    par Ruta AC
        P->>AC: GET /login (query params)
        AC-->>P: Token AC + metadata
        Note over P: Transform: Add token_area=AC
    and Ruta Cognito
        P->>CG: POST /signinuser (body)
        CG-->>P: AccessToken + RefreshToken
        Note over P: Transform: Normalize to OAuth2
    end
    
    Note over P: Consolidar Respuestas
    Note over P: Prioridad: AC
    Note over P: AC token válido ✓
    
    P-->>C: 200 OK<br/>{access_token, token_area: "AC"}
```

#### 2. Flujo con Fallback (AC Falla, Cognito OK)

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API
    participant AC as s-ac
    participant CG as Cognito API
    
    C->>P: GET /login?username=user&password=pass
    Note over P: Scatter-Gather Inicia
    
    par Ruta AC
        P->>AC: GET /login
        AC--xP: Error 401 Unauthorized
        Note over P: Catch Error<br/>Set codResult: -0001
    and Ruta Cognito
        P->>CG: POST /signinuser
        CG-->>P: AccessToken válido
        Note over P: Transform: Normalize
    end
    
    Note over P: Consolidar Respuestas
    Note over P: Prioridad: AC
    Note over P: AC token = null ✗
    Note over P: Fallback a Cognito ✓
    
    P-->>C: 200 OK<br/>{access_token, token_area: "CG"}
```

#### 3. Flujo con Error Total (Ambos Fallan)

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API
    participant AC as s-ac
    participant CG as Cognito API
    
    C->>P: GET /login?username=wrong&password=wrong
    Note over P: Scatter-Gather Inicia
    
    par Ruta AC
        P->>AC: GET /login
        AC--xP: Error 401
        Note over P: Catch Error<br/>codResult: -0001
    and Ruta Cognito
        P->>CG: POST /signinuser
        CG--xP: Error 401
        Note over P: Catch Error<br/>codResult: -0001
    end
    
    Note over P: Consolidar Respuestas
    Note over P: Ambos tokens = null
    Note over P: codResult != 0000
    
    P-->>C: 400 Bad Request<br/>{codResult: "-0001"}
```

### Manejo de Errores

#### Estrategia por Ruta (Try-Catch Independiente)

**Ruta AC (Línea 34-52):**
```xml
<on-error-continue>
  <ee:transform doc:name="Error AC">
    <ee:set-variable variableName="loginAC">
      {
        "result": {
          "codResult": "-0001",
          "msgResult": "Error técnico"
        }
      }
    </ee:set-variable>
  </ee:transform>
</on-error-continue>
```

**Ruta Cognito (Línea 115-133):**
```xml
<on-error-continue>
  <ee:transform doc:name="Error Omega">
    <ee:set-variable variableName="loginOmega">
      {
        "result": {
          "codResult": "-0001",
          "msgResult": "Error técnico"
        }
      }
    </ee:set-variable>
  </ee:transform>
</on-error-continue>
```

**Beneficio:** Cada ruta maneja sus errores sin afectar la otra

#### Error Handler Global (Línea 177-196)

```xml
<on-error-propagate type="HTTP:*, EXPRESSION, STREAM_MAXIMUM_SIZE_EXCEEDED">
  <logger level="ERROR" message="logErrorGenerico" />
  <ee:transform>
    <ee:set-payload>
      {
        "result": {
          "codResult": "-0001",
          "msgResult": "Error técnico"
        }
      }
    </ee:set-payload>
    <ee:set-variable variableName="httpStatus">400</ee:set-variable>
  </ee:transform>
</on-error-propagate>
```

**Tipos de Error Capturados:**
- `HTTP:BAD_GATEWAY`, `HTTP:CONNECTIVITY`, `HTTP:TIMEOUT`
- `HTTP:UNAUTHORIZED`, `HTTP:FORBIDDEN`
- `EXPRESSION`, `STREAM_MAXIMUM_SIZE_EXCEEDED`

#### Validación Post-Consolidación (Línea 156-175)

```xml
<choice>
  <when expression='#[payload.result.codResult != "0000"]'>
    <ee:transform doc:name="Error generico">
      <ee:set-payload>
        {
          "result": {
            "codResult": "-0001",
            "msgResult": "Error técnico"
          }
        }
      </ee:set-payload>
      <ee:set-variable variableName="httpStatus">400</ee:set-variable>
    </ee:transform>
  </when>
</choice>
```

**Propósito:** Valida que al menos un backend haya autenticado exitosamente

### Configuración de Propiedades

**Propiedades Requeridas:**

```yaml
# AC Configuration
s-ac:
  path:
    login: "/login"

# Cognito Configuration
cognito:
  userPoolId: "us-east-1_XXXXXXXXX"
  client_id: "cognito-client-id"
  apiKey: "cognito-api-key"

# Priorización
login_token_prioridad: "AC"  # Valores: "AC" o "CG"
```

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Llamada Exitosa con cURL

#### Request

```bash
curl -X GET "https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/login?username=user@example.com&password=SecurePass123&client_id=app-client-id&client_secret=app-client-secret&grant_type=password" \
  -H "Content-Type: application/json"
```

#### Respuesta (200 OK) - Token AC

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyQGV4YW1wbGUuY29tIiwiaWF0IjoxNjk1MDAwMDAwLCJleHAiOjE2OTUwNDk5OTl9.signature",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyQGV4YW1wbGUuY29tIiwiaWF0IjoxNjk1MDAwMDAwfQ.refresh_signature",
  "expires_in": 49999,
  "scope": "read write",
  "token_area": "AC"
}
```

### Ejemplo 2: JavaScript (Fetch API)

```javascript
const loginUser = async (username, password) => {
  const params = new URLSearchParams({
    username: username,
    password: password,
    client_id: 'app-client-id',
    client_secret: 'app-client-secret',
    grant_type: 'password'
  });

  try {
    const response = await fetch(
      `https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/login?${params}`,
      {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json'
        }
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    
    // Guardar token para uso posterior
    localStorage.setItem('access_token', data.access_token);
    localStorage.setItem('refresh_token', data.refresh_token);
    localStorage.setItem('token_area', data.token_area);
    
    console.log(`Autenticado exitosamente con ${data.token_area}`);
    return data;
    
  } catch (error) {
    console.error('Error de autenticación:', error);
    throw error;
  }
};

// Uso
loginUser('user@example.com', 'SecurePass123')
  .then(data => console.log('Token obtenido:', data.access_token))
  .catch(error => console.error('Falló autenticación:', error));
```

### Ejemplo 3: Python (Requests)

```python
import requests
from typing import Dict, Optional

def login_user(
    username: str,
    password: str,
    client_id: str,
    client_secret: str,
    base_url: str = "https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io"
) -> Optional[Dict]:
    """
    Autentica usuario y obtiene token OAuth 2.0
    
    Args:
        username: Email o identificador de usuario
        password: Contraseña del usuario
        client_id: ID de cliente OAuth2
        client_secret: Secreto de cliente OAuth2
        base_url: URL base del API
        
    Returns:
        Dict con token y metadata, o None si falla
    """
    
    params = {
        'username': username,
        'password': password,
        'client_id': client_id,
        'client_secret': client_secret,
        'grant_type': 'password'
    }
    
    try:
        response = requests.get(
            f'{base_url}/api/login',
            params=params,
            headers={'Content-Type': 'application/json'},
            timeout=30
        )
        
        response.raise_for_status()
        data = response.json()
        
        # Verificar autenticación exitosa
        if data.get('result', {}).get('codResult') == '0000':
            print(f"✓ Autenticado con {data.get('token_area')}")
            return data
        else:
            print(f"✗ Error: {data.get('result', {}).get('msgResult')}")
            return None
            
    except requests.exceptions.RequestException as e:
        print(f"✗ Error de conexión: {e}")
        return None

# Uso
if __name__ == "__main__":
    token_data = login_user(
        username='user@example.com',
        password='SecurePass123',
        client_id='app-client-id',
        client_secret='app-client-secret'
    )
    
    if token_data:
        print(f"Access Token: {token_data['access_token'][:50]}...")
        print(f"Expira en: {token_data['expires_in']} segundos")
        print(f"Origen: {token_data['token_area']}")
```

### Ejemplo 4: Error de Autenticación

#### Request con Credenciales Inválidas

```bash
curl -X GET "https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/login?username=invalid@example.com&password=WrongPass&client_id=app-client-id&client_secret=app-client-secret&grant_type=password" \
  -H "Content-Type: application/json"
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

**Interpretación:** Ambos backends (AC y Cognito) rechazaron las credenciales, por lo que no se pudo generar ningún token válido.
