---
api: "ntgy-fua-s-cognito"
sidebar_position: 2
description: Valida tokens de acceso JWT de AWS Cognito mediante Lambda y obtiene información del usuario autenticado
keywords:
  - cognito
  - token validation
  - jwt
  - authentication
  - lambda
  - access token
tags:
  - System API
  - AWS Cognito
  - Authentication
  - Token Validation
  - Lambda
cognito_metadata:
  service: "AWS Cognito User Pools"
  lambda_function: "Token Validation Handler"
  method: "POST"
  authentication: "X-API-Key"
---

# POST /checkToken

## 📌 Descripción General

### Tipo de API
**System API** - Capa de integración directa con AWS Cognito User Pools vía Lambda

### Patrón de Integración
**Request-Reply Síncrono** con validación de token mediante Lambda

### Arquitectura

```mermaid
graph LR
    A[Client] -->|POST /checkToken| B[System API]
    B -->|POST Lambda| C[API Gateway]
    C -->|Invoke| D[Lambda Function]
    D -->|Validate Token| E[AWS Cognito]
    E -->|Token Info| D
    D -->|User Data| C
    C -->|Response| B
    B -->|User Info| A
    
    style B fill:#e1f5ff
    style D fill:#fff4e1
    style E fill:#ffe1e1
```

**Flujo de datos:**
1. Cliente envía accessToken y username opcional
2. System API transforma datos al formato Lambda
3. Invoca API Gateway con X-API-Key
4. Lambda valida token contra Cognito
5. Cognito verifica validez y expiración
6. Lambda retorna información del usuario
7. System API transforma y retorna respuesta

### Backend
- **Servicio**: AWS Cognito User Pools
- **Integración**: API Gateway + Lambda
- **Método HTTP**: POST
- **Autenticación**: X-API-Key (API Gateway)
- **Protocolo**: HTTPS

---

### Propósito
Validar tokens de acceso JWT emitidos por AWS Cognito y obtener información del usuario autenticado, permitiendo verificar sesiones activas y autorizar operaciones en aplicaciones.

### Casos de Uso

#### 1. Validación de Sesión Activa
**Escenario**: Una aplicación web necesita verificar si el usuario sigue autenticado antes de mostrar contenido protegido.

**Flujo**:
1. Usuario accede a página protegida
2. Aplicación envía accessToken almacenado
3. Sistema valida token contra Cognito vía Lambda
4. Si válido, muestra contenido; si no, redirige a login

**Beneficio**: Seguridad mejorada y experiencia de usuario fluida

#### 2. Middleware de Autenticación
**Escenario**: API Gateway necesita validar tokens antes de permitir acceso a endpoints protegidos.

**Flujo**:
1. Request llega con token
2. Middleware extrae token
3. Valida token con /checkToken
4. Si válido, permite acceso; si no, retorna 401

**Beneficio**: Centralización de validación de autenticación

#### 3. Verificación Pre-Operación
**Escenario**: Antes de ejecutar operación sensible, verificar autenticación.

**Flujo**:
1. Usuario intenta operación sensible
2. Sistema valida token actual
3. Si válido y no expirado, permite operación
4. Si expirado, solicita re-autenticación

**Beneficio**: Seguridad adicional en operaciones críticas

### Stakeholders
- **Usuarios Finales**: Clientes de Naturgy que usan portales y apps
- **Desarrolladores**: Equipos que implementan autenticación
- **Seguridad**: Equipo que gestiona políticas de autenticación
- **Operaciones**: Equipo que monitorea sesiones y accesos

---

## 🔧 Especificación Técnica

### Endpoint
```
POST /checkToken
```

### Headers
```http
Content-Type: application/json
```

### Request Body

```json
{
  "accessToken": "eyJraWQiOiJxxx...xxxxx",
  "username": "user@example.com"
}
```

#### Parámetros

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `accessToken` | string | ✅ Sí | Token JWT de acceso emitido por Cognito durante login |
| `username` | string | ❌ No | Username opcional para validación adicional |

**Validaciones**:
- `accessToken`: Debe ser un JWT válido (mínimo 100 caracteres)
- `username`: Si se proporciona, debe coincidir con el usuario del token

### Response Body

#### Respuesta Exitosa (200 OK)

```json
{
  "valid": true,
  "username": "user@example.com",
  "sub": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "email": "user@example.com",
  "email_verified": true,
  "custom:confirmedUser": "true"
}
```

#### Campos de Respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `valid` | boolean | Indica si el token es válido |
| `username` | string | Nombre de usuario en Cognito |
| `sub` | string | Subject identifier único del usuario (UUID) |
| `email` | string | Email del usuario |
| `email_verified` | boolean | Indica si el email está verificado |
| `custom:confirmedUser` | string | Atributo custom de confirmación |

#### Respuesta de Error (401 Unauthorized)

```json
{
  "error": {
    "code": "NotAuthorizedException",
    "message": "Invalid or expired token"
  }
}
```

#### Respuesta de Error (500 Internal Server Error)

```json
{
  "error": {
    "code": "InternalErrorException",
    "message": "Internal server error"
  }
}
```

### Códigos de Estado HTTP

| Código | Descripción | Cuándo Ocurre |
|--------|-------------|---------------|
| 200 | OK | Token válido, información del usuario retornada |
| 401 | Unauthorized | Token inválido, expirado o malformado |
| 500 | Internal Server Error | Error en AWS Cognito, Lambda o System API |

---

## 🏗️ Implementación Técnica

### Diagrama de Secuencia

```mermaid
sequenceDiagram
    participant Client
    participant SystemAPI as System API<br/>(MuleSoft)
    participant Gateway as API Gateway
    participant Lambda as Lambda Function
    participant Cognito as AWS Cognito
    
    Client->>SystemAPI: POST /checkToken<br/>{accessToken, username}
    
    Note over SystemAPI: Logger: Inicio del flow checkToken
    
    SystemAPI->>SystemAPI: Transform to Lambda format<br/>{AccessToken, UserPoolId, Username}
    
    SystemAPI->>Gateway: POST /checktoken<br/>X-API-Key: {apikey}
    
    Gateway->>Lambda: Invoke Lambda Function
    
    Lambda->>Cognito: Validate Token
    
    alt Token Válido
        Cognito-->>Lambda: Token Valid<br/>{user info}
        Lambda-->>Gateway: 200 OK<br/>{valid: true, user data}
        Gateway-->>SystemAPI: 200 OK<br/>{user info}
        Note over SystemAPI: Logger: Response payload
        SystemAPI-->>Client: 200 OK<br/>{user info}
    else Token Inválido/Expirado
        Cognito-->>Lambda: Token Invalid
        Lambda-->>Gateway: 401 Unauthorized
        Gateway-->>SystemAPI: 401 Unauthorized
        SystemAPI-->>Client: 401 Unauthorized<br/>{error details}
    else Error de Cognito
        Cognito-->>Lambda: Error
        Lambda-->>Gateway: 500 Error
        Gateway-->>SystemAPI: 500 Error
        SystemAPI-->>Client: 500 Internal Server Error
    end
    
    Note over SystemAPI: Logger: Fin del flow checkToken
```

### Diagrama de Arquitectura

```mermaid
graph TB
    subgraph "API-Led Connectivity"
        E[Experience APIs<br/>Mobile, Web]
        P[Process APIs<br/>Business Logic]
    end
    
    subgraph "System API Layer"
        S[ntgy-fua-s-cognito<br/>System API]
    end
    
    subgraph "AWS Cloud"
        AG[API Gateway<br/>X-API-Key Auth]
        L[Lambda Function<br/>Token Validation]
        C[AWS Cognito<br/>User Pools]
    end
    
    E -->|Token Validation| P
    P -->|POST /checkToken| S
    S -->|POST + X-API-Key| AG
    AG -->|Invoke| L
    L -->|Validate Token| C
    C -->|Token Info| L
    L -->|User Data| AG
    AG -->|Response| S
    S -->|Validated User Data| P
    P -->|User Info| E
    
    style S fill:#e1f5ff
    style L fill:#fff4e1
    style C fill:#ffe1e1
```

### Diagrama de Transformación de Datos

```mermaid
graph LR
    subgraph "Input"
        I1[accessToken: string]
        I2[username: string optional]
    end
    
    subgraph "Transformation"
        T1[Extract Fields]
        T2[Add UserPoolId]
        T3[Format for Lambda]
    end
    
    subgraph "Lambda Request"
        L1[AccessToken]
        L2[UserPoolId]
        L3[Username]
    end
    
    subgraph "Lambda Response"
        R1[valid: boolean]
        R2[username: string]
        R3[sub: UUID]
        R4[email: string]
        R5[custom attributes]
    end
    
    subgraph "Output"
        O1[User Information JSON]
    end
    
    I1 --> T1
    I2 --> T1
    T1 --> T2
    T2 --> T3
    T3 --> L1
    T3 --> L2
    T3 --> L3
    L1 --> R1
    L2 --> R1
    L3 --> R1
    R1 --> O1
    R2 --> O1
    R3 --> O1
    R4 --> O1
    R5 --> O1
    
    style T1 fill:#e1f5ff
    style T2 fill:#e1f5ff
    style T3 fill:#e1f5ff
```

### Flow MuleSoft

**Archivo**: `s-cognito.xml`

**Flow Name**: `post:\checkToken:application\json:s-cognito-config`

**Componentes**:

1. **HTTP Listener** (APIKit Router)
   - Recibe POST request en `/checkToken`
   - Valida request contra RAML spec

2. **Logger - Inicio**
   ```xml
   <logger level="INFO" message="post:\checkToken:application\json:s-cognito-config" />
   ```

3. **Transform Message - Prepare Lambda Request**
   ```dataweave
   %dw 2.0
   output application/json
   ---
   {
     "AccessToken": payload.accessToken,
     "UserPoolId": p('cognito.userPoolId'),
     "Username": payload.username
   }
   ```
   - Extrae accessToken y username del body
   - Inyecta UserPoolId desde properties
   - Prepara formato esperado por Lambda

4. **HTTP Request - API Gateway**
   ```xml
   <http:request 
       method="POST" 
       config-ref="HTTP_Cognito" 
       path="${cognito.path.checktoken}">
       <http:headers>
           <![CDATA[#[output application/java
           ---
           {
               "X-API-Key": p('cognito.apikey')
           }]]]>
       </http:headers>
   </http:request>
   ```
   - Invoca API Gateway con método POST
   - Incluye X-API-Key para autenticación
   - Path configurado en properties
   - Timeout: 30 segundos

5. **Logger - Response**
   ```xml
   <logger level="INFO" doc:name="Logger" />
   ```

### Configuración

**Properties** (`s-cognito-{env}-config.yaml`):

```yaml
cognito:
  host: "{api-gateway-url}.execute-api.{region}.amazonaws.com"
  path:
    globalpath: "/prod"
    checktoken: "/checktoken"
  userPoolId: "{user-pool-id}"
  apikey: "{api-key}"
  timeout: "30000"
```

**Ejemplo de configuración por ambiente**:
- **Dev**: `https://abc123.execute-api.us-east-1.amazonaws.com/dev/checktoken`
- **UAT**: `https://def456.execute-api.us-east-1.amazonaws.com/uat/checktoken`
- **Prod**: `https://ghi789.execute-api.us-east-1.amazonaws.com/prod/checktoken`

### Manejo de Errores

#### Error: Token Expirado
```json
{
  "error": {
    "code": "NotAuthorizedException",
    "message": "Access token has expired"
  }
}
```
**Solución**: Cliente debe obtener nuevo token con /login o refresh token

#### Error: Token Inválido
```json
{
  "error": {
    "code": "NotAuthorizedException",
    "message": "Invalid access token"
  }
}
```
**Solución**: Verificar que el token sea correcto y pertenezca al User Pool

#### Error: Username No Coincide
```json
{
  "error": {
    "code": "InvalidParameterException",
    "message": "Username does not match token"
  }
}
```
**Solución**: Verificar que el username proporcionado coincida con el del token

### Consideraciones de Performance

- **Response Time**: < 3 segundos (p95)
- **Timeout**: 30 segundos configurado
- **Caching**: No se cachea (tokens pueden expirar)
- **Rate Limiting**: Limitado por AWS API Gateway y Cognito

### Consideraciones de Seguridad

1. **Token en HTTPS**: Siempre transmitir tokens sobre HTTPS
2. **No Logging de Tokens**: Tokens no se loguean en producción
3. **Validación en Cognito**: Validación criptográfica del JWT
4. **Expiración**: Tokens expiran automáticamente (típicamente 1 hora)
5. **API Key Rotation**: Rotar X-API-Key cada 90 días

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Validación de Token Válido

#### Request
```bash
curl -X POST https://ntgy-fua-s-cognito-dev.us-e2.cloudhub.io/api/checkToken \
  -H "Content-Type: application/json" \
  -d '{
    "accessToken": "eyJraWQiOiJxNzVGR1wvXC9cL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQnhcL0RcL0pqQ