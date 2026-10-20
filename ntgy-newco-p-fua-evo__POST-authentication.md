---
api: "ntgy-newco-p-fua-evo"
sidebar_position: 58
sidebar_label: "POST /authentication"
description: "Autenticación de usuarios contra AWS Cognito con validación de roles en PostgreSQL y actualización de estado en Salesforce"
keywords: [authentication, cognito, login, AWS, PostgreSQL, Salesforce, autenticación, tokens, JWT, roles]
tags: [Autenticación, AWS Cognito, Process API, Orquestación, Seguridad]
endpoint_dependencies:
  - method: POST
    path: /getUser
    api_name: ntgy-fua-s-cognito
  - method: POST
    path: /login
    api_name: ntgy-fua-s-cognito
  - method: POST
    path: /ejecutarQuery
    api_name: ntgy-newco-s-postgresql-salesforce
  - method: POST
    path: /postComposite
    api_name: ntgy-newco-s-salesforce
backend_integration:
  primary: AWS Cognito
  secondary:
    - PostgreSQL Heroku (roles validation)
    - Salesforce (async status update)
---

# POST /authentication

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint orquesta el proceso completo de autenticación de usuarios del portal FUA contra AWS Cognito, validando roles y permisos en PostgreSQL Heroku, y actualizando el estado de login en Salesforce de forma asíncrona. Es el punto de entrada crítico para el acceso seguro al portal.

### Tipo de API

**Capa:** Process API (API-Led Connectivity)

**Función:** Orquestación del proceso de autenticación multi-backend con validación de roles, generación de tokens JWT y actualización de estado

### Patrón de Integración

**Tipo:** Orquestación Compleja con Transformación y Procesamiento Asíncrono

**Descripción:** El endpoint coordina tres backends diferentes (AWS Cognito, PostgreSQL Heroku y Salesforce) en un flujo secuencial con validaciones de negocio, transformaciones de datos y actualizaciones asíncronas de estado.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    E[Experience APIs<br/>Portal FUA, Apps Móviles]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /authentication]
    S1[System API<br/>ntgy-fua-s-cognito<br/>AWS Cognito]
    S2[System API<br/>s-postgresql-salesforce<br/>PostgreSQL Heroku]
    S3[System API<br/>ntgy-newco-s-salesforce<br/>Salesforce CRM]
    C[(AWS Cognito<br/>User Pool)]
    PG[(PostgreSQL Heroku<br/>newcosalesforce)]
    SF[(Salesforce<br/>Account/Contact)]
    
    E -->|username + password| P
    P -->|1. getUser| S1
    S1 -->|verify| C
    C -->|user exists| S1
    S1 -->|response| P
    
    P -->|2. query roles| S2
    S2 -->|SQL query| PG
    PG -->|roles data| S2
    S2 -->|response| P
    
    P -->|3. login| S1
    S1 -->|authenticate| C
    C -->|JWT tokens| S1
    S1 -->|tokens| P
    
    P -.->|4. async update| S3
    S3 -.->|composite| SF
    
    P -->|tokens + user info| E
    
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:3px
    style S1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S3 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style C fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    style PG fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style SF fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Autenticación Segura de Clientes**
   - Validación de credenciales contra AWS Cognito
   - Generación de tokens JWT (AccessToken, IdToken, RefreshToken)
   - Soporte para múltiples cuentas por usuario

2. **Validación de Roles y Permisos**
   - Consulta de roles en PostgreSQL Heroku
   - Validación de roles permitidos (01, 03, 04, 05, 06)
   - Detección de usuarios externos (>50 cuentas)
   - Verificación de estado de login en Omega

3. **Trazabilidad de Accesos**
   - Actualización de fecha de último login en Salesforce
   - Marcado de contacto como registrado
   - Registro de actividad de usuario

4. **Prevención de Fraude**
   - Bloqueo de CIF (solo DNI/NIE permitidos)
   - Validación de existencia de usuario en Cognito
   - Verificación de coherencia entre username y token JWT

### 🔗 Endpoints Dependientes

Este Process API consume los siguientes System APIs:

| System API | Endpoint | Método | Propósito | Secuencia |
|------------|----------|--------|-----------|-----------|
| `ntgy-fua-s-cognito` | `/getUser` | POST | Verificar existencia del usuario en AWS Cognito | 1 |
| `s-postgresql-salesforce` | `/ejecutarQuery` | POST | Consulta de roles y permisos en PostgreSQL Heroku | 2 |
| `ntgy-fua-s-cognito` | `/login` | POST | Autenticar usuario y obtener tokens JWT (AccessToken, IdToken, RefreshToken) | 3 |
| `ntgy-newco-s-salesforce` | `/postComposite` | POST | Actualización asíncrona de estado en Salesforce (fecha último login, marca registrado) | 4 (async) |

### Lógica de Orquestación

```mermaid
graph TD
    Start([POST /authentication]) --> ValidateCIF{¿Es CIF?}
    ValidateCIF -->|Sí| ErrorCIF[Error: CIF no permitido]
    ValidateCIF -->|No| GetUser[Cognito: getUser]
    
    GetUser --> QueryRoles[PostgreSQL: Query Roles]
    QueryRoles --> CheckRoles{Validar Roles}
    
    CheckRoles -->|Sin roles| ErrorNoRoles[Error: Usuario no encontrado]
    CheckRoles -->|Roles válidos| CountAccounts{Contar Cuentas}
    
    CountAccounts -->|> 50 cuentas| SetExternal[Rol = Externo]
    CountAccounts -->|≤ 50 cuentas| BuildRoles[Construir string roles]
    
    SetExternal --> Login[Cognito: Login]
    BuildRoles --> Login
    
    Login --> DecodeToken[Decodificar JWT]
    DecodeToken --> ValidateToken{¿Token válido?}
    
    ValidateToken -->|No| ErrorToken[Error: Token inválido]
    ValidateToken -->|Sí| CheckStatus{¿Estado Validado?}
    
    CheckStatus -->|Sí| AsyncFull[Async: Update Login + Registrado]
    CheckStatus -->|No| AsyncPartial[Async: Update Registrado]
    
    AsyncFull --> Response[Response con Tokens]
    AsyncPartial --> Response
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style Response fill:#e8f5e9,stroke:#2e7d32
    style ErrorCIF fill:#ffebee,stroke:#c62828
    style ErrorNoRoles fill:#ffebee,stroke:#c62828
    style ErrorToken fill:#ffebee,stroke:#c62828
```

**Pasos del Proceso:**

1. **Validación de Formato**: Verificar que el usuario NO sea un CIF (solo DNI/NIE permitidos)
2. **Verificación en Cognito**: Confirmar que el usuario existe en AWS Cognito (operación `getUser`)
3. **Consulta de Roles**: Obtener roles y permisos del usuario desde PostgreSQL Heroku
4. **Validación de Roles**: 
   - Si no tiene roles → Error (usuario no encontrado o en proceso de creación)
   - Si tiene > 50 cuentas → Marcar como "Externo"
   - Si tiene ≤ 50 cuentas → Construir string de roles
5. **Autenticación**: Login en AWS Cognito con username, password y roles
6. **Validación de Token**: Decodificar JWT y verificar coherencia con username
7. **Actualización Asíncrona**: Actualizar estado en Salesforce según estado del usuario
8. **Respuesta**: Retornar tokens JWT y datos del usuario

### Transformaciones Aplicadas

#### Transformación 1: Preparación para getUser (líneas 27-35)

| Campo Original (Input) | Campo Transformado (Cognito) | Transformación |
|------------------------|------------------------------|----------------|
| `payload.user` | `username` | Pass-through |
| N/A | `operation` | Hardcoded: `"getUser"` |

#### Transformación 2: Query SQL para Roles (líneas 37-47)

| Campo Original | Campo Transformado | Transformación |
|----------------|-------------------|----------------|
| `payload.user` | `params[0]` | Pass-through como parámetro SQL |
| N/A | `query` | Query SQL complejo con JOINs |

**Query SQL Ejecutado:**
```sql
SELECT 
  accountcontactrelation.accountid,
  accountcontactrelation.roles,
  accountcontactrelation.newco_estadologinomega__c,
  account.acn_fld_documentnumber__c as docNumber,
  contact.sfid as contactid
FROM newcosalesforce.accountcontactrelation
INNER JOIN newcosalesforce.Contact 
  ON contact.sfid = accountcontactrelation.contactid
INNER JOIN newcosalesforce.account 
  ON account.sfid = accountcontactrelation.accountid
WHERE accountcontactrelation.roles IN ('01','03','04','05','06')
  AND accountcontactrelation.roles IS NOT NULL
  AND contact.newco_documentnumber__c = :param1
```

#### Transformación 3: Construcción de Roles (líneas 68-76)

| Condición | Transformación |
|-----------|----------------|
| `sizeOf(herokuResponse) > 50` | `roles = "Externo"` |
| `sizeOf(herokuResponse) ≤ 50` | `roles = accountid,docnumber,roles,estado # accountid,docnumber,roles,estado # ...` |

#### Transformación 4: Preparación para Login (líneas 78-88)

| Campo Original | Campo Transformado (Cognito) | Transformación |
|----------------|------------------------------|----------------|
| `payload.user` | `username` | Pass-through |
| `payload.password` | `password` | Pass-through |
| `vars.roles` | `roles` | String construido en paso anterior |
| N/A | `operation` | Hardcoded: `"login"` |

#### Transformación 5: Decodificación de Token JWT (líneas 93-103)

| Campo Original | Transformación |
|----------------|----------------|
| `loginOutput.AccessToken` | Split por ".", extraer parte [1], decodificar Base64, parsear JSON |

#### Transformación 6: Respuesta Final (líneas 107-123)

| Campo Original | Campo Transformado | Transformación |
|----------------|-------------------|----------------|
| `loginOutput.AccessToken` | `authResponse.token` | Pass-through |
| `loginOutput.IdToken` | `authResponse.idToken` | Pass-through |
| `loginOutput.RefreshToken` | `authResponse.refreshToken` | Pass-through |
| `tokenOutput.body.username` | `authResponse.user.userId` | Uppercase |
| `tokenOutput.body.username` | `authResponse.user.clientCode` | Uppercase |
| N/A | `authResponse.user.roleTypeId` | Hardcoded: `"newcoClient"` |
| N/A | `result.codResult` | Hardcoded: `"0000"` |
| N/A | `result.msgResult` | Hardcoded: `"No hay error"` |

### Contexto Empresarial

- **Dominio:** Autenticación y Gestión de Accesos
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Proceso de Negocio:** Acceso seguro al portal FUA

### Stakeholders

- **Clientes Finales**: Usuarios del portal FUA que necesitan acceso seguro
- **Equipo de Seguridad**: Gestión de políticas de autenticación y prevención de fraude
- **Equipo de Desarrollo FUA**: Mantenimiento del flujo de login
- **Administradores de Cognito**: Gestión del User Pool de AWS
- **Administradores de Salesforce**: Seguimiento de actividad de usuarios

### Valor de Negocio

- **Seguridad**: Autenticación robusta con AWS Cognito y validación multi-capa
- **Experiencia de Usuario**: Login rápido con soporte para múltiples cuentas
- **Trazabilidad**: Registro completo de accesos y actividad
- **Prevención de Fraude**: Validaciones de formato, roles y coherencia de tokens
- **Escalabilidad**: Uso de servicios cloud (AWS Cognito) para alto volumen

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Portal web FUA (página de login)
- Aplicaciones móviles de Naturgy
- Sistemas de agentes de call center
- Aplicaciones de terceros autorizados (con OAuth)

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/authentication` |
| **Development** | `https://ntgy-newco-p-fua-evo-des.de-c1.cloudhub.io` | `https://ntgy-newco-p-fua-evo-des.de-c1.cloudhub.io/api/authentication` |
| **UAT** | `https://ntgy-newco-p-fua-evo-uat.de-c1.cloudhub.io` | `https://ntgy-newco-p-fua-evo-uat.de-c1.cloudhub.io/api/authentication` |
| **Production** | `https://ntgy-newco-p-fua-evo-pro.de-c1.cloudhub.io` | `https://ntgy-newco-p-fua-evo-pro.de-c1.cloudhub.io/api/authentication` |

### Método HTTP

```
POST /api/authentication
```

### Headers Requeridos

| Header | Tipo | Obligatorio | Descripción |
|--------|------|-------------|-------------|
| `Content-Type` | string | ✅ Sí | Debe ser `application/json` |

### Parámetros de Entrada

#### Request Body (application/json)

```json
{
  "user": "string",
  "password": "string"
}
```

#### Descripción de Campos

| Campo | Tipo | Obligatorio | Descripción | Validación |
|-------|------|-------------|-------------|------------|
| `user` | string | ✅ Sí | Número de documento del usuario (DNI/NIE) | NO debe ser CIF (regex: `/^[ABCDEFGHJKLMNPQRSUVW][0-9]{7}[0-9A-J]$/`) |
| `password` | string | ✅ Sí | Contraseña del usuario en AWS Cognito | Mínimo 8 caracteres (validado por Cognito) |

**⚠️ IMPORTANTE - Validación de CIF:**
- Los CIF (Certificados de Identificación Fiscal) NO están permitidos
- Solo se aceptan DNI (Documento Nacional de Identidad) y NIE (Número de Identidad de Extranjero)
- Patrón de CIF: Letra inicial + 7 dígitos + dígito/letra de control
- Si se detecta un CIF, se retorna error `APP:COGNITO_USER_NOT_VALID`

### Respuestas

#### 200 OK - Autenticación Exitosa

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "authResponse": {
    "token": "eyJraWQiOiJ...",
    "idToken": "eyJraWQiOiJ...",
    "refreshToken": "eyJjdHkiOiJ...",
    "user": {
      "userId": "12345678A",
      "clientCode": "12345678A",
      "roleTypeId": "newcoClient"
    }
  }
}
```

**Descripción de Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | string | Código de resultado (`"0000"` = éxito) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |
| `authResponse.token` | string | JWT AccessToken para autenticación en APIs |
| `authResponse.idToken` | string | JWT IdToken con información del usuario |
| `authResponse.refreshToken` | string | JWT RefreshToken para renovar tokens |
| `authResponse.user.userId` | string | ID del usuario (documento en mayúsculas) |
| `authResponse.user.clientCode` | string | Código de cliente (documento en mayúsculas) |
| `authResponse.user.roleTypeId` | string | Tipo de rol (`"newcoClient"`) |

#### 400 Bad Request - CIF No Permitido

```json
{
  "error": "APP:COGNITO_USER_NOT_VALID",
  "message": "Los CIF no están permitidos para autenticación"
}
```

#### 400 Bad Request - Usuario No Encontrado

```json
{
  "error": "APP:POSTGRESQL_SALESFORCE_CLIENT_NOT_FOUND",
  "message": "Usuario no encontrado en el sistema"
}
```

#### 400 Bad Request - Usuario en Proceso de Creación

```json
{
  "error": "APP:POSTGRESQL_SALESFORCE_CREATION_IN_PROGRESS",
  "message": "Usuario en proceso de creación, intente más tarde"
}
```

#### 401 Unauthorized - Credenciales Inválidas

```json
{
  "error": "COGNITO:INVALID_CREDENTIALS",
  "message": "Usuario o contraseña incorrectos"
}
```

#### 500 Internal Server Error - Error del Sistema

```json
{
  "error": "SYSTEM_ERROR",
  "message": "Error al procesar la autenticación"
}
```

### Autenticación

- **Tipo:** Ninguna (este ES el endpoint de autenticación)
- **Descripción:** Este endpoint genera los tokens de autenticación que se usarán en otros endpoints

### Timeouts

| Tipo | Valor | Descripción |
|------|-------|-------------|
| **Request Timeout** | 30 segundos | Timeout general del endpoint |
| **Cognito Timeout** | 30 segundos | Timeout para operaciones en AWS Cognito |
| **PostgreSQL Timeout** | 240 segundos | Timeout para consultas en Heroku |
| **Salesforce Timeout** | Asíncrono | No bloquea la respuesta |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request]) --> LogStart[Log: Inicio authentication]
    LogStart --> ValidateCIF{Validar CIF<br/>Regex Match}
    
    ValidateCIF -->|Es CIF| ErrorCIF[Raise Error:<br/>COGNITO_USER_NOT_VALID]
    ValidateCIF -->|No es CIF| SetUser[Set Variable: user]
    
    SetUser --> Transform1[Transform:<br/>inputCognito getUser]
    Transform1 --> CallCognito1[Call: cognitoSub_Flow<br/>Operation: getUser]
    
    CallCognito1 --> Transform2[Transform:<br/>query SQL roles]
    Transform2 --> CallPostgres[HTTP POST:<br/>s-postgresql-salesforce<br/>/ejecutarQuery]
    
    CallPostgres --> CheckRoles{Validar<br/>Respuesta}
    CheckRoles -->|Sin roles + error| ErrorNotFound[Raise Error:<br/>CLIENT_NOT_FOUND]
    CheckRoles -->|Sin roles| ErrorCreation[Raise Error:<br/>CREATION_IN_PROGRESS]
    CheckRoles -->|> 50 cuentas| SetExternal[Set roles:<br/>Externo]
    CheckRoles -->|≤ 50 cuentas| BuildRoles[Transform:<br/>Construir string roles]
    
    SetExternal --> Transform3[Transform:<br/>inputCognito login]
    BuildRoles --> Transform3
    
    Transform3 --> CallCognito2[Call: cognitoSub_Flow<br/>Operation: login]
    CallCognito2 --> LogSuccess[Log: Usuario Contraseña<br/>Correctos]
    
    LogSuccess --> CheckStatus{Estado<br/>Validado?}
    CheckStatus -->|Sí| AsyncFull[Async:<br/>Update Login + Registrado]
    CheckStatus -->|No| AsyncPartial[Async:<br/>Update Registrado]
    
    AsyncFull --> Transform4[Transform:<br/>Decodificar JWT]
    AsyncPartial --> Transform4
    
    Transform4 --> ValidateToken{Username ==<br/>Token.username?}
    ValidateToken -->|No| ErrorToken[Raise Error:<br/>USER_NOT_VALID]
    ValidateToken -->|Sí| Transform5[Transform:<br/>Response Final]
    
    Transform5 --> LogEnd[Log: Fin authentication]
    LogEnd --> End([HTTP Response 200 OK])
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style End fill:#e8f5e9,stroke:#2e7d32
    style ErrorCIF fill:#ffebee,stroke:#c62828
    style ErrorNotFound fill:#ffebee,stroke:#c62828
    style ErrorCreation fill:#fff3e0,stroke:#f57c00
    style ErrorToken fill:#ffebee,stroke:#c62828
    style AsyncFull fill:#e1f5fe,stroke:#0277bd
    style AsyncPartial fill:#e1f5fe,stroke:#0277bd
```

### Transformaciones DataWeave

#### Transformación 1: Preparación para getUser (líneas 27-35)

```dataweave
%dw 2.0
output application/json
---
{
	"username": payload.user,
	"operation": "getUser"
}
```

#### Transformación 2: Query SQL para Roles (líneas 37-47)

```dataweave
%dw 2.0
output application/json
---
{
    "query": "select accountcontactrelation.accountid, accountcontactrelation.roles, accountcontactrelation.newco_estadologinomega__c, account.acn_fld_documentnumber__c as docNumber, contact.sfid as contactid from newcosalesforce.accountcontactrelation as accountcontactrelation inner join newcosalesforce.Contact as contact on contact.sfid = accountcontactrelation.contactid inner join newcosalesforce.account as account on account.sfid = accountcontactrelation.accountid where (accountcontactrelation.roles in ('01','03','04','05','06') and accountcontactrelation.roles is not null) and contact.newco_documentnumber__c = :param1",
    "params": [payload.user]
}
```

#### Transformación 3: Construcción de Roles (líneas 68-76)

**Caso 1: Usuario Externo (>50 cuentas)**
```dataweave
%dw 2.0
output text/plain
---
"Externo"
```

**Caso 2: Usuario Normal (≤50 cuentas)**
```dataweave
%dw 2.0
output text/plain
---
(vars.herokuResponse map ( $.accountid ++ "," ++ $.docnumber ++ "," ++ $.roles ++ "," ++ $.newco_estadologinomega__c) joinBy "#")
```

**Ejemplo de Output:**
```
0011234567890ABC,12345678A,01,Validado#0019876543210XYZ,12345678A,03,Pendiente
```

#### Transformación 4: Preparación para Login (líneas 78-88)

```dataweave
%dw 2.0
output application/json
---
{
  "username": payload.user,
  "password": payload.password,
  "roles": vars.roles,
  "operation": "login"
}
```

#### Transformación 5: Decodificación de Token JWT (líneas 93-103)

```dataweave
%dw 2.0
import fromBase64 from dw::core::Binaries
output application/json
var AccessToken = (vars.loginOutput.AccessToken splitBy ".")

---
{
    body:read(fromBase64(AccessToken[1]),"application/json")
}
```

**Explicación:**
1. Split del JWT por "." → `[header, payload, signature]`
2. Extraer parte [1] (payload)
3. Decodificar Base64
4. Parsear como JSON

#### Transformación 6: Respuesta Final (líneas 107-123)

```dataweave
%dw 2.0
output application/json
---
{
	"result": {
		"codResult": "0000",
		"msgResult": "No hay error"
	},
	"authResponse": {
		"token": vars.loginOutput.AccessToken,
		"idToken": vars.loginOutput.IdToken,
		"refreshToken": vars.loginOutput.RefreshToken,
		"user": {
			"userId": upper(vars.tokenOutput.body.username),
			"clientCode": upper(vars.tokenOutput.body.username),
			"roleTypeId": "newcoClient"
		}
	}
}
```

### Patrones de Diseño

**Patrón 1: Orchestration Pattern**

**Justificación:** El endpoint coordina múltiples servicios (Cognito, PostgreSQL, Salesforce) en un flujo secuencial con lógica de negocio compleja.

**Patrón 2: Async Fire-and-Forget**

**Justificación:** Las actualizaciones en Salesforce se ejecutan de forma asíncrona para no bloquear la respuesta al usuario, mejorando el tiempo de respuesta.

**Patrón 3: Validation Chain**

**Justificación:** Múltiples validaciones en cascada (CIF, usuario existe, roles válidos, token coherente) aseguran la integridad del proceso de autenticación.

**Patrón 4: Token Verification**

**Justificación:** Decodificación y validación del JWT generado por Cognito para asegurar coherencia entre el username solicitado y el token emitido.

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito - Usuario con Cuenta Única

```mermaid
sequenceDiagram
    participant C as Cliente/Portal FUA
    participant P as Process API<br/>authenticationFlow
    participant CG as System API<br/>ntgy-fua-s-cognito
    participant COG as AWS Cognito
    participant PG as System API<br/>s-postgresql-salesforce
    participant DB as PostgreSQL Heroku
    participant SF as System API<br/>ntgy-newco-s-salesforce
    participant SFC as Salesforce CRM
    
    C->>P: POST /authentication<br/>{user, password}
    activate P
    Note over P: Validar NO es CIF
    
    P->>CG: POST /cognito<br/>{username, operation: getUser}
    activate CG
    CG->>COG: GetUser
    COG-->>CG: User exists
    CG-->>P: User data
    deactivate CG
    
    P->>PG: POST /ejecutarQuery<br/>{query, params}
    activate PG
    PG->>DB: SQL Query (roles)
    DB-->>PG: 1 row (accountId, roles, estado)
    PG-->>P: herokuResponse[1]
    deactivate PG
    
    Note over P: Construir string roles
    
    P->>CG: POST /cognito<br/>{username, password, roles, operation: login}
    activate CG
    CG->>COG: InitiateAuth
    COG-->>CG: JWT Tokens
    CG-->>P: AccessToken, IdToken, RefreshToken
    deactivate CG
    
    Note over P: Decodificar JWT<br/>Validar username
    
    par Async Update Salesforce
        P->>SF: POST /salesforce<br/>{composite: update Account + Contact}
        activate SF
        SF->>SFC: Composite API
        SFC-->>SF: Success
        SF-->>P: OK
        deactivate SF
    end
    
    P-->>C: 200 OK<br/>{result, authResponse}
    deactivate P
```

#### 2. Flujo con Usuario Externo (>50 Cuentas)

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API
    participant CG as Cognito API
    participant PG as PostgreSQL API
    participant DB as PostgreSQL
    
    C->>P: POST /authentication
    activate P
    
    P->>CG: getUser
    CG-->>P: User exists
    
    P->>PG: Query roles
    activate PG
    PG->>DB: SQL Query
    DB-->>PG: 51 rows
    PG-->>P: herokuResponse[51]
    deactivate PG
    
    Note over P: sizeOf > 50<br/>roles = "Externo"
    
    P->>CG: login (roles: Externo)
    CG-->>P: JWT Tokens
    
    Note over P: Usuario marcado<br/>como Externo
    
    P-->>C: 200 OK (Externo)
    deactivate P
```

#### 3. Flujo con Error - CIF No Permitido

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API
    
    C->>P: POST /authentication<br/>{user: "A12345678", password}
    activate P
    
    Note over P: Validar formato<br/>Regex match CIF
    
    P-->>C: 400 Bad Request<br/>COGNITO_USER_NOT_VALID
    deactivate P
```

#### 4. Flujo con Error - Usuario No Encontrado

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API
    participant CG as Cognito API
    participant PG as PostgreSQL API
    participant DB as PostgreSQL
    
    C->>P: POST /authentication
    activate P
    
    P->>CG: getUser
    CG-->>P: User exists
    
    P->>PG: Query roles
    activate PG
    PG->>DB: SQL Query
    DB-->>PG: 0 rows
    PG-->>P: herokuResponse = []
    deactivate PG
    
    Note over P: isEmpty(herokuResponse)<br/>+ isUser.error == true
    
    P-->>C: 400 Bad Request<br/>CLIENT_NOT_FOUND
    deactivate P
```

#### 5. Flujo con Error - Credenciales Inválidas

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API
    participant CG as Cognito API
    participant COG as AWS Cognito
    
    C->>P: POST /authentication<br/>{user, password: "wrong"}
    activate P
    
    P->>CG: getUser
    CG-->>P: User exists
    
    P->>CG: login (wrong password)
    activate CG
    CG->>COG: InitiateAuth
    COG-->>CG: NotAuthorizedException
    CG-->>P: Error
    deactivate CG
    
    P-->>C: 401 Unauthorized<br/>INVALID_CREDENTIALS
    deactivate P
```

### Manejo de Errores

**Estrategia:** Error handler global `login-errorsError_Handler`

**Tipos de Errores Manejados:**

1. **Errores de Validación (400)**
   - `APP:COGNITO_USER_NOT_VALID`: CIF detectado o token inválido
   - `APP:POSTGRESQL_SALESFORCE_CLIENT_NOT_FOUND`: Usuario sin roles en PostgreSQL
   - `APP:POSTGRESQL_SALESFORCE_CREATION_IN_PROGRESS`: Usuario en proceso de creación

2. **Errores de Autenticación (401)**
   - `COGNITO:INVALID_CREDENTIALS`: Usuario o contraseña incorrectos
   - `COGNITO:USER_NOT_FOUND`: Usuario no existe en Cognito
   - `COGNITO:USER_NOT_CONFIRMED`: Usuario no confirmado

3. **Errores de Conectividad (500)**
   - `APP:POSTGRESQL_SALESFORCE`: Error de conexión con PostgreSQL
   - `COGNITO:SERVICE_ERROR`: Error del servicio AWS Cognito
   - `SALESFORCE:SERVICE_ERROR`: Error del servicio Salesforce (no bloquea respuesta)

**Respuesta de Error Estandarizada:**

```json
{
  "error": "ERROR_CODE",
  "message": "Descripción del error",
  "timestamp": "2026-01-24T14:30:00Z"
}
```

### Configuración de Propiedades

**Archivo:** `properties-dev.yaml`

```yaml
s-cognito:
  host: "mule-worker-internal-ntgy-fua-s-cognito-dev.de-c1.cloudhub.io"
  port: "8091"
  timeout: "30000"
  basePath: "api"
  userPoolId: "eu-west-1_WWZbMSd5k"

s-postgresql-salesforce:
  host: "mule-worker-internal-ntgy-newco-s-postgresql-salesforce-des.de-c1.cloudhub.io"
  port: "8091"
  timeout: "240000"
  pathejecutarQuery: "/ejecutarQuery"

ntgy-newco-s-salesforce:
  host: "mule-worker-internal-ntgy-newco-s-salesforce-des.de-c1.cloudhub.io"
  port: "8091"
  protocol: "HTTP"
  timeout: "60000"
  basePath: "/api"
```

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Login Exitoso - Usuario con Cuenta Única

#### cURL

```bash
curl -X POST "https://ntgy-newco-p-fua-evo-des.de-c1.cloudhub.io/api/authentication" \
  -H "Content-Type: application/json" \
  -d '{
    "user": "12345678A",
    "password": "MySecurePassword123!"
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "authResponse": {
    "token": "eyJraWQiOiJxN3JYc0RcL0pqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tEQnNcL0tqXC9cL0tE
