---
api: "ntgy-fua-s-cognito"
sidebar_position: 6
sidebar_label: "POST /logout"
description: "Cierra la sesión de un usuario en AWS Cognito revocando globalmente todos sus tokens de acceso"
keywords: [logout, cognito, aws, sesión, cierre, revocación, tokens, seguridad, autenticación]
tags: [Autenticación, AWS Cognito, Seguridad, Gestión de Sesiones]
aws_cognito_metadata:
  operation: "GlobalSignOut"
  service: "Cognito Identity Provider"
  authentication: "Access Token"
  scope: "Global (todas las sesiones del usuario)"
  token_invalidation: "Access Tokens, ID Tokens"
---

# POST /logout

## 📌 Descripción General

### Propósito del Endpoint

Cierra la sesión de un usuario en AWS Cognito User Pool mediante la revocación global de todos sus tokens de acceso. Este endpoint invoca la operación `GlobalSignOut` de AWS Cognito a través de API Gateway y Lambda, invalidando inmediatamente todas las sesiones activas del usuario en todos los dispositivos.

El endpoint soporta:
- **Revocación Global de Tokens**: Invalida todos los Access Tokens e ID Tokens del usuario
- **Cierre Multi-Dispositivo**: Cierra sesión en todos los dispositivos donde el usuario esté autenticado
- **Seguridad Inmediata**: Los tokens quedan inválidos inmediatamente tras la operación
- **Logout Forzado**: Permite cerrar sesión de forma administrativa o por seguridad

### Tipo de API

**Capa:** System API (API-Led Connectivity)

**Función:** Integración directa con AWS Cognito User Pools vía API Gateway y Lambda para gestión de sesiones y revocación de tokens.

### Patrón de Integración

**Tipo:** Request-Reply Síncrono con AWS Cognito GlobalSignOut

**Descripción:** Recibe un Access Token válido, lo transforma al formato esperado por Lambda, invoca la operación GlobalSignOut de AWS Cognito a través de API Gateway, y retorna confirmación de cierre de sesión exitoso.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    Client[Cliente<br/>Portal Web/App Móvil]
    EA[Experience API<br/>ntgy-fua-e-*]
    PA[Process API<br/>ntgy-fua-p-*]
    SA[System API<br/>ntgy-fua-s-cognito<br/>POST /logout]
    GW[AWS API Gateway<br/>X-API-Key Auth]
    Lambda[AWS Lambda<br/>Logout Handler]
    Cognito[AWS Cognito<br/>User Pools<br/>GlobalSignOut]
    
    Client -->|Access Token| EA
    EA -->|Request JSON| PA
    PA -->|Request JSON| SA
    SA -->|HTTPS POST| GW
    GW -->|Invoke| Lambda
    Lambda -->|GlobalSignOut| Cognito
    Cognito -->|Tokens Revoked| Lambda
    Lambda -->|Response| GW
    GW -->|Response| SA
    SA -->|Response JSON| PA
    PA -->|Confirmation| EA
    EA -->|Logout Success| Client
    
    style SA fill:#e1f5ff,stroke:#01579b,stroke-width:3px
    style Cognito fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style Lambda fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Logout Normal de Usuario**
   - Usuario cierra sesión desde portal web o aplicación móvil
   - Se invalidan todos los tokens del usuario
   - Usuario debe autenticarse nuevamente para acceder

2. **Cierre de Sesión por Seguridad**
   - Sospecha de compromiso de cuenta
   - Cambio de contraseña requiere re-autenticación
   - Cierre preventivo de todas las sesiones activas

3. **Logout Forzado Administrativo**
   - Administrador cierra sesión de usuario por políticas de seguridad
   - Deshabilitación temporal de acceso
   - Gestión de sesiones sospechosas

4. **Cierre Multi-Dispositivo**
   - Usuario solicita "Cerrar sesión en todos los dispositivos"
   - Revocación de acceso en dispositivos perdidos o robados
   - Limpieza de sesiones antiguas

### Contexto Empresarial

- **Dominio:** Gestión de Identidad y Acceso (IAM)
- **Sector:** Energía - Utilities
- **División:** Naturgy España - Front Único de Atención (FUA)
- **Proceso de Negocio:** Autenticación y Gestión de Sesiones
- **Servicio Cloud:** AWS Cognito User Pools

### Stakeholders

- **Usuarios Finales**: Clientes de Naturgy que necesitan cerrar sesión de forma segura
- **Equipo de Seguridad**: Responsables de políticas de seguridad y gestión de accesos
- **Administradores de Sistema**: Gestión de sesiones y soporte a usuarios
- **Desarrolladores de Aplicaciones**: Implementación de logout en portales y apps

### Valor de Negocio

- **Seguridad Mejorada**: Revocación inmediata de acceso en caso de compromiso
- **Cumplimiento Normativo**: Gestión adecuada de sesiones según políticas de seguridad
- **Experiencia de Usuario**: Logout confiable y predecible en todos los canales
- **Control de Acceso**: Capacidad de cerrar sesiones de forma centralizada

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Process APIs de autenticación y gestión de usuarios
- Experience APIs de portales web de autoservicio
- Aplicaciones móviles (iOS, Android)
- Sistemas de agentes de call center
- Herramientas administrativas de gestión de usuarios

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/logout` |
| **Development** | `https://ntgy-fua-s-cognito-dev.us-e2.cloudhub.io` | `https://ntgy-fua-s-cognito-dev.us-e2.cloudhub.io/api/logout` |
| **UAT** | `https://ntgy-fua-s-cognito-uat.us-e2.cloudhub.io` | `https://ntgy-fua-s-cognito-uat.us-e2.cloudhub.io/api/logout` |
| **Production** | `https://ntgy-fua-s-cognito-pro.us-e2.cloudhub.io` | `https://ntgy-fua-s-cognito-pro.us-e2.cloudhub.io/api/logout` |

### Método HTTP

```
POST /api/logout
```

### Parámetros de Entrada

#### Headers HTTP

| Header | Tipo | Requerido | Descripción | Ejemplo |
|--------|------|-----------|-------------|---------|
| `Content-Type` | string | ✅ Sí | Tipo de contenido del request | `application/json` |
| `Accept` | string | ❌ No | Tipo de contenido esperado en response | `application/json` |

#### Body Parameters

**Tipo RAML:** `LogoutRequest`

| Campo | Tipo | Requerido | Descripción | Ejemplo |
|-------|------|-----------|-------------|---------|
| `accessToken` | string | ✅ Sí | Token de acceso JWT obtenido durante el login | `"eyJraWQiOiJxxx...xxxxx"` |

**Ejemplo de Request:**

```json
{
  "accessToken": "eyJraWQiOiJxdXNlcjEiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
}
```

### Respuestas

#### Respuesta Exitosa (200 OK)

**Tipo RAML:** `LogoutResponse`

```json
{
  "statusCode": 200,
  "message": "Logout successful",
  "data": {
    "timestamp": "2026-06-12T10:55:00Z"
  }
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `statusCode` | integer | Código de estado HTTP (200 para éxito) |
| `message` | string | Mensaje descriptivo del resultado |
| `data` | object | Datos adicionales (puede incluir timestamp) |

#### Respuesta de Error - Token Inválido (400 Bad Request)

```json
{
  "statusCode": 400,
  "message": "Invalid access token",
  "data": {}
}
```

#### Respuesta de Error - Token Expirado (401 Unauthorized)

```json
{
  "error": {
    "code": "NotAuthorizedException",
    "message": "Access Token has expired or is invalid"
  }
}
```

#### Respuesta de Error Técnico (500 Internal Server Error)

```json
{
  "statusCode": 500,
  "message": "Internal server error",
  "data": {}
}
```

### Códigos de Estado HTTP

| Código | Descripción | Cuándo Ocurre |
|--------|-------------|---------------|
| `200` | OK | Logout exitoso, tokens revocados |
| `400` | Bad Request | Token inválido o request malformado |
| `401` | Unauthorized | Token expirado o no autorizado |
| `500` | Internal Server Error | Error en AWS Cognito o Lambda |

### Autenticación y Autorización

- **Tipo:** Client ID & Client Secret (Anypoint Platform)
- **AWS API Gateway:** Autenticación mediante X-API-Key header
- **Autorización AWS:** El Access Token debe:
  - Ser válido y no estar expirado
  - Pertenecer al User Pool configurado
  - Estar asociado a un usuario existente en Cognito

### Timeouts

| Tipo | Valor | Descripción |
|------|-------|-------------|
| **Request Timeout** | 30 segundos | Tiempo máximo de espera para la respuesta de AWS |
| **Connection Timeout** | 10 segundos | Tiempo máximo para establecer conexión con API Gateway |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request]) --> LogStart[Log: Inicio del flow logout]
    LogStart --> Transform[Transform Message<br/>Prepare Lambda Input<br/>Extract accessToken]
    Transform --> LogInput[Log: Input Data]
    LogInput --> HTTPReq[HTTP Request<br/>POST to API Gateway<br/>X-API-Key Authentication]
    HTTPReq --> Lambda[AWS Lambda<br/>Logout Handler<br/>GlobalSignOut Operation]
    Lambda --> Cognito[AWS Cognito<br/>Revoke All Tokens<br/>Close All Sessions]
    Cognito --> LambdaResp[Lambda Response<br/>Logout Confirmation]
    LambdaResp --> PrepareResp[Transform Message<br/>Build Standard Response<br/>statusCode, message, data]
    PrepareResp --> LogResp[Log: Response Data]
    LogResp --> LogEnd[Log: Fin del flow logout]
    LogEnd --> End([HTTP 200 Response])
    
    HTTPReq -.->|Error| ErrorHandler[Error Handler<br/>Transform Error Response]
    ErrorHandler -.-> ErrorEnd([HTTP 400/500 Response])
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style Cognito fill:#fff3e0,stroke:#e65100,stroke-width:3px
    style Lambda fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style End fill:#e8f5e9,stroke:#2e7d32
    style ErrorEnd fill:#ffebee,stroke:#c62828
```

### Componentes del Flow

#### 1. HTTP Listener
- **Configuración:** `HTTP_Listener_config`
- **Path:** `/logout`
- **Método:** POST
- **Puerto:** 8081 (local), 8081 (CloudHub)

#### 2. Logger - Inicio
```xml
<logger level="INFO" message="post:\logout:application\json:s-cognito-config" />
```
- Registra el inicio del flow de logout

#### 3. Transform Message - Prepare Lambda Input
```dataweave
%dw 2.0
output application/json
---
{
    "AccessToken": payload.accessToken
}
```
- Extrae el `accessToken` del payload
- Transforma al formato esperado por Lambda (campo `AccessToken`)

#### 4. HTTP Request - API Gateway
- **Configuración:** `HTTP_Cognito`
- **Método:** POST
- **Path:** `${cognito.path.logout}` (típicamente `/logout`)
- **Headers:**
  ```dataweave
  {
    "X-API-Key": p('cognito.apikey')
  }
  ```
- **Timeout:** 30 segundos

#### 5. AWS Lambda - Logout Handler
- **Operación:** GlobalSignOut
- **Proceso:**
  1. Recibe Access Token
  2. Valida token contra Cognito
  3. Ejecuta GlobalSignOut
  4. Retorna confirmación

#### 6. AWS Cognito - GlobalSignOut
- **Operación:** `GlobalSignOut`
- **Efecto:**
  - Invalida todos los Access Tokens del usuario
  - Invalida todos los ID Tokens del usuario
  - Cierra todas las sesiones activas
  - Los Refresh Tokens no pueden usarse para obtener nuevos tokens

#### 7. Transform Message - Build Response
```dataweave
%dw 2.0
output application/json
---
{
    "statusCode": 200,
    "message": "Logout successful",
    "data": payload.data default {}
}
```
- Construye respuesta estandarizada
- Incluye statusCode, message y data

#### 8. Logger - Response
```xml
<logger level="INFO" message="#[%dw 2.0 output application/json --- payload]" />
```
- Registra la respuesta completa (sin datos sensibles)

#### 9. Logger - Fin
```xml
<logger level="INFO" message="Fin del flow logout" />
```
- Registra el fin exitoso del flow

### Configuración de Propiedades

**Archivo:** `s-cognito-[env]-config.yaml`

```yaml
cognito:
  host: "api-gateway-url.execute-api.us-east-1.amazonaws.com"
  path:
    globalpath: "/prod"
    logout: "/logout"
  apikey: "secure-api-key-value"
  timeout: "30000"
```

### Diagrama de Secuencia Detallado

```mermaid
sequenceDiagram
    participant Client as Cliente
    participant SA as System API<br/>(MuleSoft)
    participant GW as API Gateway<br/>(AWS)
    participant Lambda as Lambda Function
    participant Cognito as AWS Cognito<br/>User Pools
    
    Client->>SA: POST /logout<br/>{accessToken}
    activate SA
    SA->>SA: Log: Inicio flow
    SA->>SA: Transform: Extract accessToken
    SA->>SA: Log: Input data
    
    SA->>GW: POST /logout<br/>X-API-Key: ***<br/>{AccessToken}
    activate GW
    GW->>GW: Validate API Key
    
    GW->>Lambda: Invoke Logout Handler<br/>{AccessToken}
    activate Lambda
    
    Lambda->>Cognito: GlobalSignOut<br/>(AccessToken)
    activate Cognito
    Cognito->>Cognito: Validate Token
    Cognito->>Cognito: Revoke All Access Tokens
    Cognito->>Cognito: Revoke All ID Tokens
    Cognito->>Cognito: Close All Sessions
    Cognito-->>Lambda: Logout Confirmed
    deactivate Cognito
    
    Lambda-->>GW: {status: "success"}
    deactivate Lambda
    
    GW-->>SA: 200 OK<br/>{logout confirmation}
    deactivate GW
    
    SA->>SA: Transform: Build Response
    SA->>SA: Log: Response data
    SA->>SA: Log: Fin flow
    SA-->>Client: 200 OK<br/>{statusCode, message, data}
    deactivate SA
    
    Note over Cognito: Todos los tokens<br/>del usuario quedan<br/>inmediatamente inválidos
```

### Manejo de Errores

#### Error Handler - APIKit

```xml
<on-error-propagate type="APIKIT:BAD_REQUEST">
    <ee:transform>
        <ee:message>
            <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    message: "Bad request"
}]]></ee:set-payload>
        </ee:message>
        <ee:variables>
            <ee:set-variable variableName="httpStatus">400</ee:set-variable>
        </ee:variables>
    </ee:transform>
</on-error-propagate>
```

#### Errores Comunes de AWS Cognito

| Error | Código | Descripción | Solución |
|-------|--------|-------------|----------|
| `NotAuthorizedException` | 401 | Token inválido o expirado | Usuario debe hacer login nuevamente |
| `InvalidParameterException` | 400 | Access Token malformado | Verificar formato del token |
| `TooManyRequestsException` | 429 | Rate limit excedido | Implementar retry con backoff |
| `InternalErrorException` | 500 | Error interno de AWS | Reintentar operación |

### Transformaciones DataWeave

#### Input Transformation
```dataweave
%dw 2.0
output application/json
---
{
    "AccessToken": payload.accessToken
}
```

**Propósito:** Extraer y mapear el Access Token al formato esperado por Lambda.

#### Output Transformation
```dataweave
%dw 2.0
output application/json
---
{
    "statusCode": 200,
    "message": "Logout successful",
    "data": payload.data default {}
}
```

**Propósito:** Construir respuesta estandarizada con statusCode, mensaje y datos adicionales.

### Diagrama de Transformación de Datos

```mermaid
graph LR
    Input[Request JSON<br/>accessToken: string] --> DW1[DataWeave Transform 1<br/>Extract accessToken]
    DW1 --> Lambda[Lambda Format<br/>AccessToken: string]
    Lambda --> Cognito[AWS Cognito<br/>GlobalSignOut]
    Cognito --> LambdaResp[Lambda Response<br/>status, data]
    LambdaResp --> DW2[DataWeave Transform 2<br/>Build Standard Response]
    DW2 --> Output[Response JSON<br/>statusCode, message, data]
    
    style Input fill:#e3f2fd,stroke:#1976d2
    style DW1 fill:#fff3e0,stroke:#e65100
    style Lambda fill:#f3e5f5,stroke:#4a148c
    style Cognito fill:#fff3e0,stroke:#e65100,stroke-width:3px
    style DW2 fill:#fff3e0,stroke:#e65100
    style Output fill:#e8f5e9,stroke:#2e7d32
```

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Logout Exitoso

#### Request

```bash
curl -X POST "https://ntgy-fua-s-cognito-dev.us-e2.cloudhub.io/api/logout" \
  -H "Content-Type: application/json" \
  -d '{
    "accessToken": "eyJraWQiOiJxdXNlcjEiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
  }'
```

#### Response (200 OK)

```json
{
  "statusCode": 200,
  "message": "Logout successful",
  "data": {
    "timestamp": "2026-06-12T10:55:00Z"
  }
}
```

**Descripción:** El usuario ha cerrado sesión exitosamente. Todos sus tokens han sido revocados y debe autenticarse nuevamente para acceder.

---

### Ejemplo 2: Token Inválido

#### Request

```bash
curl -X POST "https://ntgy-fua-s-cognito-dev.us-e2.cloudhub.io/api/logout" \
  -H "Content-Type: application/json" \
  -d '{
    "accessToken": "invalid-token-format"
  }'
```

#### Response (400 Bad Request)

```json
{
  "statusCode": 400,
  "message": "Invalid access token",
  "data": {}
}
```

**Descripción:** El token proporcionado no tiene un formato válido de JWT.

---

### Ejemplo 3: Token Expirado

#### Request

```bash
curl -X POST "https://ntgy-fua-s-cognito-dev.us-e2.cloudhub.io/api/logout" \
  -H "Content-Type: application/json" \
  -d '{
    "accessToken": "eyJraWQiOiJleHBpcmVkIiwiYWxnIjoiUlMyNTYifQ.eyJzdWIiOiIxMjM0NTY3ODkwIiwiZXhwIjoxNTE2MjM5MDIyfQ.expired_signature"
  }'
```

#### Response (401 Unauthorized)

```json
{
  "error": {
    "code": "NotAuthorizedException",
    "message": "Access Token has expired or is invalid"
  }
}
```

**Descripción:** El token ha expirado y ya no es válido. El usuario debe hacer login nuevamente.

---

### Ejemplo 4: Error del Servidor

#### Request

```bash
curl -X POST "https://ntgy-fua-s-cognito-dev.us-e2.cloudhub.io/api/logout" \
  -H "Content-Type: application/json" \
  -d '{
    "accessToken": "eyJraWQiOiJ2YWxpZCIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiIxMjM0NTY3ODkwIn0.valid_signature"
  }'
```

#### Response (500 Internal Server Error)

```json
{
  "statusCode": 500,
  "message": "Internal server error",
  "data": {}
}
```

**Descripción:** Error interno en AWS Cognito o Lambda. Se recomienda reintentar la operación.

---

### Ejemplo 5: Logout desde JavaScript (Frontend)

```javascript
// Función para cerrar sesión
async function logout(accessToken) {
  try {
    const response = await fetch('https://ntgy-fua-s-cognito-dev.us-e2.cloudhub.io/api/logout', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        accessToken: accessToken
      })
    });

    const data = await response.json();

    if (response.ok) {
      console.log('Logout exitoso:', data.message);
      // Limpiar tokens del localStorage
      localStorage.removeItem('accessToken');
      localStorage.removeItem('idToken');
      localStorage.removeItem('refreshToken');
      // Redirigir a página de login
      window.location.href = '/login';
    } else {
      console.error('Error en logout:', data);
      // Manejar error
      if (response.status === 401) {
        // Token ya expirado, limpiar y redirigir
        localStorage.clear();
        window.location.href = '/login';
      }
    }
  } catch (error) {
    console.error('Error de red:', error);
  }
}

// Uso
const token = localStorage.getItem('accessToken');
logout(token);
```

---

### Ejemplo 6: Logout desde Postman

**Configuración:**

1. **Método:** POST
2. **URL:** `https://ntgy-fua-s-cognito-dev.us-e2.cloudhub.io/api/logout`
3. **Headers:**
   - `Content-Type: application/json`
4. **Body (raw JSON):**
   ```json
   {
     "accessToken": "{{access_token}}"
   }
   ```

**Variables de Entorno en Postman:**
- `access_token`: Token obtenido del endpoint `/login`

**Tests (opcional):**
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Logout successful", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.statusCode).to.eql(200);
    pm.expect(jsonData.message).to.include("successful");
});

// Limpiar variables de entorno
pm.environment.unset("access_token");
pm.environment.unset("id_token");
pm.environment.unset("refresh_token");
```

---

## 📊 Consideraciones de Implementación

### Mejores Prácticas

1. **Limpieza de Tokens en Cliente**
   - Siempre limpiar tokens del almacenamiento local tras logout exitoso
   - Eliminar cookies de sesión si se utilizan
   - Limpiar cualquier estado de autenticación en memoria

2. **Manejo de Errores**
   - Si el token ya expiró (401), considerar el logout como exitoso
   - Limpiar tokens locales incluso si el logout falla
   - Redirigir al usuario a la página de login

3. **Experiencia de Usuario**
   - Mostrar mensaje de confirmación tras logout exitoso
   - Proporcionar opción de "Cerrar sesión en todos los dispositivos"
   - Informar al usuario que debe autenticarse nuevamente

4. **Seguridad**
   - No almacenar tokens en localStorage si es posible (usar httpOnly cookies)
   - Implementar logout automático tras inactividad
   - Registrar eventos de logout para auditoría

### Limitaciones Conocidas

1. **GlobalSignOut es Irreversible**
   - No se pueden restaurar los tokens una vez revocados
   - El usuario debe hacer login completo nuevamente
   - Afecta a TODAS las sesiones del usuario

2. **Refresh Tokens**
   - Los Refresh Tokens permanecen técnicamente válidos
   - Sin embargo, no pueden usarse para obtener nuevos Access Tokens
   - Se recomienda invalidarlos también en el cliente

3. **Latencia**
   - La operación puede tardar hasta 30 segundos en completarse
   - Implementar indicador de carga en la UI
   - Considerar timeout del cliente

### Monitoreo y Métricas

**Métricas Recomendadas:**
- Tasa de logout exitoso vs fallido
- Tiempo de respuesta del endpoint
- Errores por tipo (401, 400, 500)
- Volumen de logouts por hora/día

**Alertas Sugeridas:**
- Tasa de error > 5%
- Tiempo de respuesta > 10 segundos
- Picos inusuales de logouts (posible ataque)

---

## 🔐 Seguridad y Cumplimiento

### Consideraciones de Seguridad

1. **Protección de Tokens**
   - Nunca registrar tokens completos en logs
   - Usar HTTPS para todas las comunicaciones
   - Implementar rate limiting para prevenir abuso

2. **Auditoría**
   - Registrar todos los eventos de logout
   - Incluir timestamp, username, IP origen
   - Mantener logs por al menos 90 días

3. **Cumplimiento GDPR**
   - El logout es parte del derecho de acceso
   - Documentar proceso de revocación de tokens
   - Proporcionar transparencia al usuario

### Políticas de Sesión

- **Duración de Access Token:** 1 hora
- **Duración de ID Token:** 1 hora
- **Duración de Refresh Token:** 30 días
- **Logout Automático:** Tras 30 minutos de inactividad (recomendado)

---

## 🧪 Testing

### Test Cases Recomendados

1. **Happy Path**
   - Logout con token válido
   - Verificar que tokens quedan inválidos
   - Confirmar que usuario no puede acceder a recursos protegidos

2. **Error Cases**
   - Token inválido (400)
   - Token expirado (401)
   - Token de otro User Pool (401)
   - Request sin token (400)
   - Request malformado (400)

3. **Edge Cases**
   - Logout múltiple (idempotencia)
   - Logout con token ya revocado
   - Logout durante mantenimiento de AWS

### Ejemplo de Test MUnit (Futuro)

```xml
<munit:test name="logout-success-test">
    <munit:execution>
        <set-payload value='{"accessToken": "valid-token"}' />
        <flow-ref name="post:\logout:application\json:s-cognito-config" />
    </munit:execution>
    <munit:validation>
        <munit-tools:assert-that expression="#[payload.statusCode]" is="#[MunitTools::equalTo(200)]" />
        <munit-tools:assert-that expression="#[payload.message]" is="#[MunitTools::containsString('successful')]" />
    </munit:validation>
</munit:test>
```
