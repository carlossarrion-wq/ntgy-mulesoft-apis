---
api: "ntgy-fua-e-fua"
sidebar_position: 78
sidebar_label: "POST /recoveryPasswordUpdate"
description: "Establece una nueva contraseña usando el token de recuperación enviado por email"
keywords:
  - recovery password
  - reset password
  - password update
  - authentication
  - token validation
tags:
  - Experience API
  - Authentication
  - Password Recovery
channel: "web"
endpoint_dependencies:
  - method: "POST"
    api_name: "ntgy-newco-p-authentication"
    path: "/recoveryPasswordUpdate"
    purpose: "Actualización de contraseña mediante token (servicio nuevo - actualmente inactivo según split-services.recoveryPasswordUpdate=false)"
  - method: "POST"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/recoveryPasswordUpdate"
    purpose: "Actualización de contraseña mediante token (servicio legacy - actualmente activo según split-services.recoveryPasswordUpdate=false)"
---

# POST /recoveryPasswordUpdate

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite a los usuarios establecer una nueva contraseña después de haber solicitado la recuperación mediante el endpoint `/recoveryPasswordMail`. El usuario recibe un token temporal por email que debe proporcionar junto con la nueva contraseña deseada.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Orquestación de recuperación de contraseña con enrutamiento condicional entre servicios nuevo y legacy

### Patrón de Integración

**Tipo:** Pass-Through con Enriquecimiento de Datos

**Descripción:** El Experience API enriquece la petición con la dirección IP del cliente (`DirIp`) y enruta condicionalmente a uno de dos Process APIs según la configuración de feature flags (`split-services.recoveryPasswordUpdate`).

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente Web<br/>Portal FUA]
    E[Experience API<br/>ntgy-fua-e-fua<br/>POST /recoveryPasswordUpdate]
    P1[Process API<br/>ntgy-newco-p-authentication<br/>Servicio Nuevo]
    P2[Process API<br/>ntgy-newco-p-fua-evo<br/>Servicio Legacy]
    S1[System API<br/>ntgy-fua-s-cognito]
    S2[System API<br/>ntgy-fua-s-dynamodb]
    S3[System API<br/>ntgy-fua-s-rds-postgres]
    
    C -->|Request + Token| E
    E -->|split-services=true| P1
    E -->|split-services=false<br/>ACTUAL| P2
    P1 -.->|Valida Token| S2
    P1 -.->|Actualiza Password| S1
    P1 -.->|Auditoría| S3
    P2 -->|Valida Token| S2
    P2 -->|Actualiza Password| S1
    P2 -->|Auditoría| S3
    S1 -.->|Response| P1
    S2 -.->|Response| P1
    S3 -.->|Response| P1
    S1 -->|Response| P2
    S2 -->|Response| P2
    S3 -->|Response| P2
    P1 -.->|Response| E
    P2 -->|Response| E
    E -->|Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P1 fill:#ffe1f5,stroke:#c2185b,stroke-width:2px,stroke-dasharray: 5 5
    style P2 fill:#ffe1f5,stroke:#c2185b,stroke-width:3px
    style S1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S3 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

**Nota:** La línea sólida indica el flujo actualmente configurado (split-services=false → ntgy-newco-p-fua-evo). La línea punteada indica el flujo alternativo disponible pero inactivo.

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Recuperación de Contraseña Olvidada**
   - Usuario olvidó su contraseña y solicitó recuperación
   - Recibió token temporal por email
   - Establece nueva contraseña de forma segura

2. **Autoservicio sin Intervención de Soporte**
   - Reduce carga en call center
   - Disponible 24/7
   - Proceso completamente automatizado

3. **Seguridad y Auditoría**
   - Token temporal con tiempo de expiración
   - Validación de token antes de actualización
   - Registro de auditoría de cambios de contraseña
   - Captura de IP del cliente para seguridad

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web

**Características del Canal:**
- **Plataforma**: Portal web FUA (Frontal Único de Atención)
- **Dispositivos**: Desktop y mobile browsers
- **Conectividad**: Requiere conexión estable para validación de token
- **Experiencia de Usuario**: Formulario simple con validación en tiempo real

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Usuarios que olvidaron su contraseña de acceso al portal
- **Clientes Empresariales**: Usuarios corporativos que necesitan recuperar acceso

**Contexto de Uso:**
- **Escenario 1**: Usuario recibió email con token y accede desde el enlace
- **Escenario 2**: Usuario tiene el token y accede directamente al formulario

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Enriquecimiento de Datos**: Captura automática de IP del cliente (`DirIp`)
- **Formato de Respuesta**: JSON simplificado con códigos de resultado
- **Validación**: Token validado en backend antes de actualización
- **Seguridad**: HTTPS obligatorio para proteger credenciales en tránsito

### 🔗 Endpoints Dependientes

Este Experience API consume uno de los siguientes Process APIs (enrutamiento condicional):

| Process API | Endpoint | Método | Estado | Propósito |
|-------------|----------|--------|--------|-----------|
| `ntgy-newco-p-authentication` | `/recoveryPasswordUpdate` | POST | Inactivo | Servicio nuevo de autenticación (split-services=true) |
| `ntgy-newco-p-fua-evo` | `/recoveryPasswordUpdate` | POST | **Activo** | Servicio legacy de autenticación (split-services=false) |

**Configuración Actual:**
```yaml
split-services:
  recoveryPasswordUpdate: "false"  # Usa ntgy-newco-p-fua-evo
```

**Consumidores Potenciales:**
Este endpoint es consumido por:
- Portal web FUA (Frontal Único de Atención)
- Aplicaciones móviles de Naturgy
- Flujos de recuperación de contraseña automatizados

### Contexto Empresarial

- **Dominio:** Gestión de Identidad y Acceso (IAM)
- **Sector:** Energía - Utilities
- **División:** Naturgy España - Área Digital
- **Canal:** Web (Portal FUA)

### Stakeholders

- **Clientes Finales**: Necesitan recuperar acceso a su cuenta de forma rápida y segura
- **Equipo de Soporte**: Reducción de tickets relacionados con recuperación de contraseña
- **Equipo de Seguridad**: Auditoría y trazabilidad de cambios de contraseña
- **Equipo de Desarrollo**: Migración gradual entre servicios legacy y nuevos

### Valor de Negocio

- **Reducción de Costos**: Menos llamadas al call center por recuperación de contraseña
- **Mejora de Experiencia**: Proceso de autoservicio rápido y disponible 24/7
- **Seguridad**: Validación de token temporal con expiración
- **Auditoría**: Registro completo de cambios de contraseña con IP del cliente
- **Flexibilidad**: Migración gradual entre servicios mediante feature flags

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/recoveryPasswordUpdate` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/recoveryPasswordUpdate` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/recoveryPasswordUpdate` |

### Método HTTP

```
POST /fua/api/recoveryPasswordUpdate
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Validación de token y actualización en múltiples sistemas |
| **Rate Limiting** | No especificado | Controlado por API Gateway |
| **Tamaño de Request** | < 1 KB | Solo contiene token y nueva contraseña |
| **Seguridad** | HTTPS obligatorio | Protección de credenciales en tránsito |

#### Campos Específicos del Canal

**Campos Añadidos por el Experience API:**

| Campo | Tipo | Descripción | Razón |
|-------|------|-------------|-------|
| `DirIp` | string | Dirección IP del cliente | Auditoría y seguridad - capturada automáticamente del request |

**Transformación Aplicada:**
```dataweave
%dw 2.0
import * from dw::core::Strings
output application/json
---
payload
++ 
{
    "DirIp": substringBefore(substringAfter(attributes.remoteAddress,"/"),":")
}
```

### Parámetros de Entrada

#### Headers

| Header | Tipo | Requerido | Descripción |
|--------|------|-----------|-------------|
| `Content-Type` | string | Sí | Debe ser `application/json` |
| `client_id` | string | Sí | Client ID para autenticación OAuth 2.0 |
| `client_secret` | string | Sí | Client Secret para autenticación OAuth 2.0 |

#### Body (application/json)

| Campo | Tipo | Requerido | Descripción | Validaciones |
|-------|------|-----------|-------------|--------------|
| `newPassword` | string | Sí | Nueva contraseña deseada | Debe cumplir política de contraseñas |
| `token` | string | Sí | Token de recuperación recibido por email | Token temporal válido y no expirado |

**Ejemplo de Request Body:**
```json
{
  "newPassword": "NuevaPassword123!",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Nota:** El campo `DirIp` es agregado automáticamente por el Experience API y no debe ser enviado por el cliente.

### Respuestas

#### 200 OK - Actualización Exitosa

**Descripción:** La contraseña fue actualizada correctamente

**Body:**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Contraseña actualizada correctamente"
  }
}
```

#### 400 Bad Request - Token Inválido o Expirado

**Descripción:** El token proporcionado no es válido o ha expirado

**Body:**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Token inválido o expirado"
  }
}
```

#### 400 Bad Request - Contraseña No Cumple Política

**Descripción:** La nueva contraseña no cumple con los requisitos de seguridad

**Body:**
```json
{
  "result": {
    "codResult": "0002",
    "msgResult": "La contraseña no cumple con los requisitos de seguridad"
  }
}
```

#### 500 Internal Server Error - Error del Sistema

**Descripción:** Error en el procesamiento de la solicitud

**Body:**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al procesar la solicitud"
  }
}
```

### Códigos de Resultado

| Código | Descripción | Acción Recomendada |
|--------|-------------|-------------------|
| `0000` | Éxito | Contraseña actualizada correctamente |
| `0001` | Error genérico | Revisar logs para detalles |
| `0002` | Validación fallida | Verificar requisitos de contraseña |
| `0003` | Token inválido | Solicitar nuevo token de recuperación |
| `0004` | Token expirado | Solicitar nuevo token de recuperación |

### Autenticación

- **Tipo:** OAuth 2.0 Client Credentials
- **Headers Requeridos:** `client_id`, `client_secret`
- **Scope:** Acceso a operaciones de recuperación de contraseña

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Validación de token en DynamoDB + actualización en Cognito + auditoría en RDS |
| **Connection Timeout** | 30 segundos | Establecimiento de conexión con Process API |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request]) --> LogStart[Log: Inicio de Recuperación]
    LogStart --> Transform1[Transform: Agregar DirIp]
    Transform1 --> Choice{split-services<br/>recoveryPasswordUpdate?}
    
    Choice -->|true<br/>INACTIVO| CallAuth[HTTP Request:<br/>p-authentication]
    Choice -->|false<br/>ACTIVO| CallEvo[HTTP Request:<br/>p-fua-evo]
    
    CallAuth --> Success1[Response OK]
    CallEvo --> Success2[Response OK]
    
    Success1 --> End([HTTP Response])
    Success2 --> End
    
    CallAuth -.->|Error| ErrorHandler[Error Handler]
    CallEvo -.->|Error| ErrorHandler
    
    ErrorHandler --> TransformError[Transform: Error Response]
    TransformError --> LogError[Log: Error Details]
    LogError --> End
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style Choice fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style CallAuth fill:#f3e5f5,stroke:#7b1fa2,stroke-dasharray: 5 5
    style CallEvo fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    style ErrorHandler fill:#ffebee,stroke:#c62828
```

### Transformaciones de Canal

#### Transformación de Entrada: Enriquecimiento con IP

**Ubicación en el código:** Línea 8-18 de `post-recovery-password-update.xml`

**Propósito:** Agregar la dirección IP del cliente para auditoría y seguridad

| Campo Original (Cliente) | Campo Transformado (Process API) | Transformación |
|--------------------------|----------------------------------|----------------|
| `newPassword` | `newPassword` | Pass-through |
| `token` | `token` | Pass-through |
| N/A | `DirIp` | Extraído de `attributes.remoteAddress` |

**Lógica de Transformación:**

```dataweave
%dw 2.0
import * from dw::core::Strings
output application/json
---
payload
++ 
{
    "DirIp": substringBefore(substringAfter(attributes.remoteAddress,"/"),":")
}
```

**Explicación:**
1. `attributes.remoteAddress` contiene la IP en formato `/127.0.0.1:12345`
2. `substringAfter(attributes.remoteAddress,"/")` extrae `127.0.0.1:12345`
3. `substringBefore(...,":")` extrae solo `127.0.0.1`
4. Se agrega al payload original con el operador `++`

#### Transformación de Salida: Pass-Through

⚠️ **Este Experience API NO transforma la respuesta del Process API.**

La respuesta del Process API se devuelve directamente al cliente sin modificaciones.

Para el schema completo de la respuesta, consultar:
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `/recoveryPasswordUpdate`**

### Enrutamiento Condicional

#### Configuración de Feature Flag

**Archivo:** `e-fua-pro-config.yaml`

```yaml
split-services:
  recoveryPasswordUpdate: "false"  # false = usa p-fua-evo, true = usa p-authentication
```

#### Lógica de Enrutamiento

**Ubicación en el código:** Línea 19-26 de `post-recovery-password-update.xml`

```xml
<choice>
  <when expression="#[(Mule::p('split-services.recoveryPasswordUpdate') as Boolean default false)]">
    <!-- Servicio NUEVO (actualmente inactivo) -->
    <http:request method="POST" 
                  config-ref="HTTP_Request_newco-p-authentication" 
                  path="/recoveryPasswordUpdate" />
  </when>
  <otherwise>
    <!-- Servicio LEGACY (actualmente activo) -->
    <http:request method="POST" 
                  config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
                  path="/recoveryPasswordUpdate" />
  </otherwise>
</choice>
```

**Estrategia de Migración:**
1. **Fase Actual**: `split-services.recoveryPasswordUpdate = false` → Usa servicio legacy
2. **Fase de Pruebas**: Cambiar a `true` en ambiente de desarrollo/test
3. **Fase de Migración**: Cambiar a `true` en producción cuando el nuevo servicio esté validado
4. **Fase Final**: Eliminar servicio legacy cuando la migración esté completa

### Configuración de Conectores HTTP

#### Conector 1: ntgy-newco-p-authentication (Nuevo - Inactivo)

**Configuración:**
```yaml
p-authentication:
  host: "ntgy-newco-p-authentication-vx8fhl.internal-6bnlu3.irl-e1.cloudhub.io"
  port: "443"
  protocol: "HTTPS"
  timeout: "600000"  # 10 minutos
  basePath: "/api/v1"
```

**Características:**
- Protocolo HTTPS
- Timeout extendido (10 minutos)
- Base path versionado (`/api/v1`)

#### Conector 2: ntgy-newco-p-fua-evo (Legacy - Activo)

**Configuración:**
```yaml
newco-p-fua-evo:
  host: "mule-worker-internal-ntgy-newco-p-fua-evo.de-c1.cloudhub.io"
  port: "8091"
  timeout: "60000"  # 60 segundos
```

**Características:**
- Protocolo HTTP interno
- Timeout estándar (60 segundos)
- Base path `/api`

### Patrones de Diseño

**Patrón:** Feature Toggle (Feature Flag) con Enrutamiento Condicional

**Justificación:** 
- Permite migración gradual entre servicios sin downtime
- Facilita rollback inmediato en caso de problemas
- Permite testing en producción con tráfico real controlado
- Reduce riesgo de migración big-bang

**Implementación:**
- Feature flag en archivo de configuración
- Evaluación en tiempo de ejecución
- Sin necesidad de redespliegue para cambiar de servicio

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito (Servicio Legacy Activo)

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant D as System API<br/>DynamoDB
    participant Cog as System API<br/>Cognito
    participant RDS as System API<br/>RDS PostgreSQL
    
    C->>E: POST /recoveryPasswordUpdate<br/>{newPassword, token}
    Note over E: Transform: Agregar DirIp
    Note over E: Choice: split-services=false
    E->>P: POST /recoveryPasswordUpdate<br/>{newPassword, token, DirIp}
    
    P->>D: Validar Token
    D-->>P: Token Válido
    
    P->>Cog: Actualizar Password
    Cog-->>P: Password Actualizada
    
    P->>RDS: Registrar Auditoría
    RDS-->>P: Auditoría Registrada
    
    P-->>E: 200 OK<br/>{codResult: "0000"}
    E-->>C: 200 OK<br/>{codResult: "0000"}
```

#### 2. Flujo con Token Inválido

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant D as System API<br/>DynamoDB
    
    C->>E: POST /recoveryPasswordUpdate<br/>{newPassword, token}
    Note over E: Transform: Agregar DirIp
    E->>P: POST /recoveryPasswordUpdate<br/>{newPassword, token, DirIp}
    
    P->>D: Validar Token
    D-->>P: Token Inválido/Expirado
    
    P-->>E: 400 Bad Request<br/>{codResult: "0003"}
    E-->>C: 400 Bad Request<br/>{codResult: "0003"}
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant Cog as System API<br/>Cognito
    
    C->>E: POST /recoveryPasswordUpdate<br/>{newPassword, token}
    Note over E: Transform: Agregar DirIp
    E->>P: POST /recoveryPasswordUpdate<br/>{newPassword, token, DirIp}
    
    P->>Cog: Actualizar Password
    Cog--xP: Error de Conexión
    
    P-->>E: 500 Internal Error<br/>{codResult: "0001"}
    Note over E: Error Handler
    Note over E: Transform Error Response
    Note over E: Log Error
    E-->>C: 500 Internal Error<br/>{codResult: "0001"}
```

### Manejo de Errores

#### Error Handler Global

**Ubicación en el código:** Línea 30-45 de `post-recovery-password-update.xml`

```xml
<error-handler>
  <on-error-propagate enableNotifications="true" logException="true">
    <ee:transform doc:name="Set Error Message">
      <ee:message>
        <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
if ( !isEmpty(error.errorMessage.payload) ) error.errorMessage.payload
else 
{
  "result": {
    "codResult": "0001",
    "msgResult": error.description as String replace /\"/ with("")
  }
}]]></ee:set-payload>
      </ee:message>
    </ee:transform>
    <logger level="ERROR" doc:name="Error" />
  </on-error-propagate>
</error-handler>
```

#### Estrategia de Manejo de Errores

1. **Errores de Negocio (del Process API)**
   - Se propagan directamente al cliente
   - Mantienen el payload de error original
   - Incluyen códigos de resultado específicos

2. **Errores Técnicos (timeout, conexión, etc.)**
   - Se transforman a formato estándar
   - Código genérico `0001`
   - Mensaje descriptivo del error
   - Se registran en logs con nivel ERROR

3. **Logging**
   - Todos los errores se registran
   - Incluye descripción completa del error
   - Facilita troubleshooting y auditoría

#### Tipos de Errores Manejados

| Tipo de Error | Código | Acción |
|---------------|--------|--------|
| Token inválido/expirado | 0003/0004 | Propagado desde Process API |
| Contraseña no válida | 0002 | Propagado desde Process API |
| Timeout de conexión | 0001 | Transformado por error handler |
| Error de sistema | 0001 | Transformado por error handler |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Actualización Exitosa de Contraseña

#### cURL

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/recoveryPasswordUpdate" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "newPassword": "NuevaPassword123!",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwiZW1haWwiOiJ1c2VyQGV4YW1wbGUuY29tIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Contraseña actualizada correctamente"
  }
}
```

### Ejemplo 2: Token Inválido o Expirado

#### cURL

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/recoveryPasswordUpdate" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "newPassword": "NuevaPassword123!",
    "token": "token_invalido_o_expirado"
  }'
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "0003",
    "msgResult": "El token de recuperación es inválido o ha expirado. Por favor, solicite un nuevo token."
  }
}
```

### Ejemplo 3: Contraseña No Cumple Requisitos

#### cURL

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/recoveryPasswordUpdate" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "newPassword": "123",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }'
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "0002",
    "msgResult": "La contraseña debe tener al menos 8 caracteres, incluir mayúsculas, minúsculas, números y caracteres especiales"
  }
}
```

### Ejemplo 4: Error del Sistema

#### cURL

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/recoveryPasswordUpdate" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "newPassword": "NuevaPassword123!",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }'
```

#### Respuesta (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al procesar la solicitud. Por favor, intente nuevamente más tarde."
  }
}
```

### Ejemplo 5: JavaScript (Aplicación Web)

```javascript
async function updatePassword(newPassword, token) {
  try {
    const response = await fetch('https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/recoveryPasswordUpdate', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'client_id': 'YOUR_CLIENT_ID',
        'client_secret': 'YOUR_CLIENT_SECRET'
      },
      body: JSON.stringify({
        newPassword: newPassword,
        token: token
      })
    });

    const data = await response.json();
    
    if (data.result.codResult === '0000') {
      console.log('Contraseña actualizada correctamente');
      // Redirigir al login
      window.location.href = '/login';
    } else {
      console.error('Error:', data.result.msgResult);
      alert(data.result.msgResult);
    }
  } catch (error) {
    console.error('Error de red:', error);
    alert('Error al conectar con el servidor. Por favor, intente nuevamente.');
  }
}

// Uso en formulario
document.getElementById('resetPasswordForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  const newPassword = document.getElementById('newPassword').value;
  const token = new URLSearchParams(window.location.search).get('token');
  await updatePassword(newPassword, token);
});
```

### Ejemplo 6: Python

```python
import requests
import json

def update_password(new_password, token):
    """
    Actualiza la contraseña usando el token de recuperación
    """
    url = 'https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/recoveryPasswordUpdate'
    
    headers = {
        'Content-Type': 'application/json',
        'client_id': 'YOUR_CLIENT_ID',
        'client_secret': 'YOUR_CLIENT_SECRET'
    }
    
    payload = {
        'newPassword': new_password,
        'token': token
    }
    
    try:
        response = requests.post(url, headers=headers, json=payload)
        data = response.json()
        
        if data['result']['codResult'] == '0000':
            print('✓ Contraseña actualizada correctamente')
            return True
        else:
            print(f'✗ Error: {data["result"]["msgResult"]}')
            return False
            
    except requests.exceptions.RequestException as e:
        print(f'✗ Error de conexión: {e}')
        return False

# Uso
if __name__ == '__main__':
    token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
    new_password = 'NuevaPassword123!'
    update_password(new_password, token)
```
