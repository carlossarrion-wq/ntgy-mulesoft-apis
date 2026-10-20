---
api: "ntgy-fua-e-fua"
sidebar_position: 51
sidebar_label: "POST /clientRegistrationConfirmation"
description: "Confirma el registro de un cliente validando el token recibido por email y registrando su dirección IP para auditoría"
keywords:
  - confirmación
  - registro
  - token
  - validación
  - email
  - activación
  - cognito
  - autenticación
tags:
  - Experience API
  - Autenticación
  - Registro
  - AWS Cognito
channel: "Web/Mobile"
endpoint_dependencies:
  - method: "POST"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/clientRegistrationConfirmation"
    purpose: "Procesamiento de confirmación de registro - validación de token en Cognito y actualización de estado en PostgreSQL (ruta activa en producción)"
  - method: "POST"
    api_name: "newco-p-authentication"
    path: "/clientRegistrationConfirmation"
    purpose: "Procesamiento alternativo de confirmación de registro (ruta inactiva, disponible mediante configuración)"
---

# POST /clientRegistrationConfirmation

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint confirma el registro de un nuevo cliente en el sistema FUA (Frontal Único de Atención) mediante la validación de un token único enviado previamente por email. El proceso verifica la autenticidad del token, activa la cuenta del cliente y registra la dirección IP desde la cual se realizó la confirmación para fines de auditoría y seguridad.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Orquestación de confirmación de registro con enriquecimiento de datos de auditoría

### Patrón de Integración

**Tipo:** Pass-Through con Enriquecimiento de Datos

**Descripción:** El Experience API añade información contextual (dirección IP del cliente) al payload original y delega el procesamiento de la confirmación al Process API correspondiente mediante routing condicional.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente Web/Mobile]
    E[Experience API<br/>ntgy-fua-e-fua<br/>POST /clientRegistrationConfirmation]
    P1[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /clientRegistrationConfirmation]
    P2[Process API<br/>newco-p-authentication<br/>POST /clientRegistrationConfirmation]
    SC[System API<br/>ntgy-fua-s-cognito]
    SP[System API<br/>ntgy-fua-s-rds-postgres]
    COG[AWS Cognito]
    PG[PostgreSQL RDS]
    
    C -->|Request con token| E
    E -->|Ruta Activa<br/>split-services=false| P1
    E -.->|Ruta Alternativa<br/>split-services=true| P2
    P1 -->|Validar Token| SC
    P1 -->|Actualizar Estado| SP
    SC -->|confirmSignUp| COG
    SP -->|UPDATE fua_client_registration| PG
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P1 fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style P2 fill:#ffe1f5,stroke:#c2185b,stroke-width:2px,stroke-dasharray: 5 5
    style SC fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style SP fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style COG fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style PG fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Confirmación de Registro de Cliente**
   - El cliente recibe un email con un enlace que contiene un token único
   - Al hacer clic en el enlace, se invoca este endpoint con el token
   - El sistema valida el token y activa la cuenta del cliente

2. **Validación de Identidad por Email**
   - Verifica que el cliente tiene acceso al email proporcionado durante el registro
   - Previene registros fraudulentos o con emails incorrectos
   - Cumple con requisitos de verificación de identidad

3. **Auditoría de Confirmaciones**
   - Registra la dirección IP desde la cual se realizó la confirmación
   - Permite rastrear intentos de confirmación sospechosos
   - Facilita investigaciones de seguridad si es necesario

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web / Mobile

**Características del Canal:**
- **Plataforma**: Aplicaciones web y móviles del portal FUA
- **Dispositivos**: Navegadores web, aplicaciones iOS/Android
- **Conectividad**: Requiere conexión a internet estable
- **Experiencia de Usuario**: Proceso de un solo clic desde el email

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Nuevos**: Personas que acaban de registrarse en el portal FUA y necesitan activar su cuenta
- **Contexto**: Proceso de onboarding inicial, crítico para el acceso al portal

**Contexto de Uso:**
- **Escenario Principal**: Cliente hace clic en enlace de confirmación desde su email
- **Escenario Alternativo**: Cliente copia y pega el token manualmente en la aplicación

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Enriquecimiento de Datos**: Añade dirección IP del cliente automáticamente
- **Routing Inteligente**: Selecciona el Process API apropiado según configuración
- **Timeout Extendido**: 60 segundos para permitir validaciones complejas
- **Manejo de Errores**: Respuestas estandarizadas para diferentes tipos de error

### 🔗 Endpoints Dependientes

Este Experience API consume los siguientes Process APIs mediante routing condicional:

| Process API | Endpoint | Método | Estado | Propósito |
|-------------|----------|--------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/clientRegistrationConfirmation` | POST | **Activo** | Procesamiento principal de confirmación |
| `newco-p-authentication` | `/clientRegistrationConfirmation` | POST | Inactivo | Procesamiento alternativo (disponible por configuración) |

**Configuración de Routing:**
- **Parámetro**: `split-services.clientRegistrationConfirmation`
- **Valor Actual (Producción)**: `false` → Usa `ntgy-newco-p-fua-evo`
- **Valor Alternativo**: `true` → Usa `newco-p-authentication`

### Contexto Empresarial

- **Dominio:** Gestión de Clientes y Autenticación
- **Sector:** Energía - Utilities
- **División:** Naturgy España - FUA (Frontal Único de Atención)
- **Canal:** Web/Móvil

### Stakeholders

- **Clientes Finales**: Necesitan activar su cuenta para acceder a servicios de autogestión
- **Equipo de Seguridad**: Requiere auditoría de confirmaciones para prevenir fraudes
- **Equipo de Soporte**: Necesita visibilidad de problemas en el proceso de confirmación

### Valor de Negocio

- **Reducción de Fraude**: Validación de email previene registros fraudulentos
- **Cumplimiento Normativo**: Verificación de identidad según regulaciones del sector energético
- **Experiencia de Usuario**: Proceso simple y rápido de activación de cuenta
- **Auditoría y Seguridad**: Registro de IPs permite investigaciones de seguridad

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/clientRegistrationConfirmation` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/clientRegistrationConfirmation` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/clientRegistrationConfirmation` |

### Método HTTP

```
POST /fua/api/clientRegistrationConfirmation
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Permite validaciones en Cognito y actualizaciones en base de datos |
| **Rate Limiting** | No especificado | Controlado por API Gateway |
| **Reintentos** | No recomendado | El token puede ser de un solo uso |

#### Campos Específicos del Canal

**Campos Añadidos por el Experience API:**

| Campo | Tipo | Descripción | Razón |
|-------|------|-------------|-------|
| `DirIp` | string | Dirección IP del cliente | Auditoría y seguridad - permite rastrear origen de confirmaciones |

**Extracción de DirIp:**
```dataweave
substringBefore(substringAfter(attributes.remoteAddress,"/"),":")
```

### Parámetros de Entrada

#### Headers

| Header | Tipo | Requerido | Descripción |
|--------|------|-----------|-------------|
| `Content-Type` | string | Sí | Debe ser `application/json` |

#### Body Parameters

| Campo | Tipo | Requerido | Descripción | Ejemplo |
|-------|------|-----------|-------------|---------|
| `token` | string | Sí | Token único de confirmación enviado por email | `"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."` |

**Ejemplo de Request Body:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
}
```

### Respuestas

#### 200 OK - Confirmación Exitosa

**Descripción:** El token fue validado correctamente y la cuenta del cliente ha sido activada.

**Body:**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "docNumber": "12345678A"
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result` | object | Objeto con información del resultado de la operación |
| `result.codResult` | string | Código de resultado (`"0000"` = éxito) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |
| `docNumber` | string | Número de documento del cliente confirmado |

#### 400 Bad Request - Token Inválido

**Descripción:** El token proporcionado no es válido o está mal formado.

**Body:**

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Token inválido o mal formado"
  }
}
```

#### 400 Bad Request - Token Expirado

**Descripción:** El token ha expirado y ya no puede ser utilizado.

**Body:**

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "El token ha expirado"
  }
}
```

#### 500 Internal Server Error

**Descripción:** Error interno del servidor durante el procesamiento.

**Body:**

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error interno del servidor"
  }
}
```

### Autenticación

- **Tipo:** No requiere autenticación previa
- **Razón:** El token en sí mismo actúa como mecanismo de autenticación

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Permite validación en Cognito y actualización en PostgreSQL |
| **Connection Timeout** | 60 segundos | Configurado en `newco-p-fua-evo.timeout` |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request]) --> Transform1[Transform: Añadir DirIp]
    Transform1 --> Choice{Routing<br/>Condicional}
    Choice -->|split-services=false<br/>ACTIVO| Route1[HTTP Request a<br/>P-FUA-EVO]
    Choice -.->|split-services=true<br/>INACTIVO| Route2[HTTP Request a<br/>P-AUTHENTICATION]
    Route1 --> Response[Response]
    Route2 -.-> Response
    Response --> End([HTTP Response])
    
    Transform1 -.->|Error| ErrorHandler[Error Handler]
    Route1 -.->|Error| ErrorHandler
    Route2 -.->|Error| ErrorHandler
    ErrorHandler --> ErrorTransform[Transform Error]
    ErrorTransform --> ErrorLog[Log Error]
    ErrorLog --> ErrorResponse([Error Response])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style ErrorResponse fill:#ffebee,stroke:#c62828
    style Choice fill:#fff9c4,stroke:#f57f17
    style Route1 fill:#c8e6c9,stroke:#388e3c,stroke-width:3px
    style Route2 fill:#f5f5f5,stroke:#9e9e9e,stroke-dasharray: 5 5
```

### Transformaciones de Canal

#### Transformación de Entrada (Experience API)

**Propósito:** Enriquecer el payload con información de auditoría

**Archivo:** Inline en `post-client-registration-confirmation.xml`

**Transformación DataWeave:**

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
1. Mantiene todos los campos del payload original (`payload ++`)
2. Añade el campo `DirIp` extrayendo la IP de `attributes.remoteAddress`
3. `attributes.remoteAddress` tiene formato: `/192.168.1.100:54321`
4. `substringAfter(attributes.remoteAddress,"/")` → `192.168.1.100:54321`
5. `substringBefore(...,":")` → `192.168.1.100`

**Payload Transformado:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "DirIp": "192.168.1.100"
}
```

### Routing Condicional

**Configuración:** `split-services.clientRegistrationConfirmation`

**Lógica de Decisión:**

```xml
<choice>
    <when expression="#[(Mule::p('split-services.clientRegistrationConfirmation') as Boolean default false)]">
        <!-- Ruta A: newco-p-authentication -->
        <http:request method="POST" 
            config-ref="HTTP_Request_newco-p-authentication" 
            path="/clientRegistrationConfirmation" />
    </when>
    <otherwise>
        <!-- Ruta B: ntgy-newco-p-fua-evo (ACTIVA) -->
        <http:request method="POST" 
            config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
            path="/clientRegistrationConfirmation" />
    </otherwise>
</choice>
```

**Configuración Actual (Producción):**

```yaml
split-services:
  clientRegistrationConfirmation: "false"  # Usa ntgy-newco-p-fua-evo
```

### Procesamiento en Process API (P-FUA-EVO)

**Archivo:** `ntgy-newco-p-fua-evo/src/main/mule/services/login/clientRegistrationConfirmation.xml`

**Flow Principal:**

```xml
<flow name="clientRegistrationConfirmation">
    <!-- 1. Validar token en AWS Cognito -->
    <flow-ref name="getClientRegistrationConfirmation" />
    
    <!-- 2. Actualizar estado en PostgreSQL -->
    <flow-ref name="updateClientRegistrationConfirmation" />
    
    <!-- 3. Construir respuesta -->
    <ee:transform doc:name="Build Response">
        {
            "result": {
                "codResult": "0000",
                "msgResult": "No hay error"
            },
            "docNumber": payload.docNumber
        }
    </ee:transform>
</flow>
```

**Subflows:**

1. **getClientRegistrationConfirmation**
   - Llama a System API: `ntgy-fua-s-cognito`
   - Operación: Validar token con AWS Cognito
   - Verifica que el token es válido y no ha expirado

2. **updateClientRegistrationConfirmation**
   - Llama a System API: `ntgy-fua-s-rds-postgres`
   - Operación: Actualizar estado de registro en PostgreSQL
   - Tabla: `fua_client_registration`
   - Acción: Marcar registro como confirmado

### Backends Involucrados

#### 1. AWS Cognito (via S-COGNITO)

**Operación:** `confirmSignUp`

**Propósito:** Validar el token de confirmación

**Validaciones:**
- Token es válido y bien formado
- Token no ha expirado
- Token no ha sido usado previamente
- Usuario existe en Cognito

#### 2. PostgreSQL RDS (via S-RDS-POSTGRES)

**Operación:** `UPDATE`

**Tabla:** `fua_client_registration`

**Propósito:** Actualizar estado del registro

**Campos Actualizados:**
- `status`: Cambia a "CONFIRMED"
- `confirmation_date`: Timestamp de confirmación
- `confirmation_ip`: Dirección IP del cliente (DirIp)

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente Web/Mobile
    participant E as E-FUA<br/>Experience API
    participant P as P-FUA-EVO<br/>Process API
    participant SC as S-COGNITO<br/>System API
    participant COG as AWS Cognito
    participant SP as S-RDS-POSTGRES<br/>System API
    participant PG as PostgreSQL RDS
    
    C->>E: POST /clientRegistrationConfirmation<br/>{token}
    Note over E: Extrae IP: 192.168.1.100
    E->>E: Transform: Añade DirIp
    Note over E: {token, DirIp}
    E->>P: POST /clientRegistrationConfirmation<br/>{token, DirIp}
    
    P->>SC: Validar token
    SC->>COG: confirmSignUp(token)
    COG-->>SC: Token válido + userData
    SC-->>P: {valid: true, docNumber}
    
    P->>SP: Actualizar estado
    SP->>PG: UPDATE fua_client_registration<br/>SET status='CONFIRMED'
    PG-->>SP: 1 row updated
    SP-->>P: {success: true}
    
    P->>P: Build Response
    P-->>E: {result: {codResult: "0000"}, docNumber}
    E-->>C: {result: {codResult: "0000"}, docNumber}
    
    Note over C: Cuenta activada ✓
```

#### 2. Flujo con Token Inválido

```mermaid
sequenceDiagram
    participant C as Cliente
    participant E as E-FUA
    participant P as P-FUA-EVO
    participant SC as S-COGNITO
    participant COG as AWS Cognito
    
    C->>E: POST /clientRegistrationConfirmation<br/>{token: "invalid"}
    E->>E: Transform: Añade DirIp
    E->>P: POST /clientRegistrationConfirmation
    
    P->>SC: Validar token
    SC->>COG: confirmSignUp(token)
    COG-->>SC: Error: Token inválido
    SC-->>P: Error: Token inválido
    
    P-->>E: Error Response
    E->>E: Transform Error
    E->>E: Log Error
    E-->>C: {result: {codResult: "0001",<br/>msgResult: "Token inválido"}}
    
    Note over C: Confirmación fallida ✗
```

#### 3. Flujo con Token Expirado

```mermaid
sequenceDiagram
    participant C as Cliente
    participant E as E-FUA
    participant P as P-FUA-EVO
    participant SC as S-COGNITO
    participant COG as AWS Cognito
    
    C->>E: POST /clientRegistrationConfirmation<br/>{token: "expired"}
    E->>E: Transform: Añade DirIp
    E->>P: POST /clientRegistrationConfirmation
    
    P->>SC: Validar token
    SC->>COG: confirmSignUp(token)
    COG-->>SC: Error: Token expirado
    SC-->>P: Error: Token expirado
    
    P-->>E: Error Response
    E->>E: Transform Error
    E->>E: Log Error
    E-->>C: {result: {codResult: "0001",<br/>msgResult: "El token ha expirado"}}
    
    Note over C: Solicitar nuevo token
```

### Manejo de Errores

#### Error Handler del Experience API

**Estrategia:** Propagación de errores con transformación estandarizada

**Implementación:**

```xml
<error-handler>
    <on-error-propagate enableNotifications="true" logException="true">
        <ee:transform doc:name="Set Error Message">
            <ee:set-payload>
                %dw 2.0
                output application/json
                ---
                if (!isEmpty(error.errorMessage.payload)) 
                    error.errorMessage.payload
                else 
                {
                    "result": {
                        "codResult": "0001",
                        "msgResult": error.description as String replace /\"/ with("")
                    }
                }
            </ee:set-payload>
        </ee:transform>
        <logger level="ERROR" doc:name="Error" />
    </on-error-propagate>
</error-handler>
```

**Lógica:**
1. Si el error del Process API tiene payload → Propagar tal cual
2. Si no → Construir respuesta estándar con código "0001"
3. Registrar error en logs para auditoría

#### Códigos de Resultado

| Código | Significado | Origen |
|--------|-------------|--------|
| `0000` | Éxito | Process API |
| `0001` | Error genérico | Experience API (error handler) |
| Otros | Errores específicos | Process API (según lógica de negocio) |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Confirmación Exitosa desde Web

#### cURL

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/clientRegistrationConfirmation" \
  -H "Content-Type: application/json" \
  -d '{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "docNumber": "12345678A"
}
```

### Ejemplo 2: JavaScript (para Aplicación Web)

```javascript
async function confirmarRegistro(token) {
  try {
    const response = await fetch(
      'https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/clientRegistrationConfirmation',
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ token })
      }
    );

    const data = await response.json();

    if (data.result.codResult === '0000') {
      console.log('✓ Cuenta activada correctamente');
      console.log('Número de documento:', data.docNumber);
      // Redirigir al login
      window.location.href = '/login';
    } else {
      console.error('✗ Error en confirmación:', data.result.msgResult);
      // Mostrar mensaje de error al usuario
      alert('Error: ' + data.result.msgResult);
    }
  } catch (error) {
    console.error('✗ Error de red:', error);
    alert('Error de conexión. Por favor, intente nuevamente.');
  }
}

// Uso: Extraer token de URL y confirmar
const urlParams = new URLSearchParams(window.location.search);
const token = urlParams.get('token');
if (token) {
  confirmarRegistro(token);
}
```

### Ejemplo 3: Python (para Testing)

```python
import requests
import json

def confirmar_registro(token):
    """
    Confirma el registro de un cliente usando el token recibido por email.
    
    Args:
        token (str): Token de confirmación
        
    Returns:
        dict: Respuesta del API
    """
    url = 'https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/clientRegistrationConfirmation'
    
    headers = {
        'Content-Type': 'application/json'
    }
    
    payload = {
        'token': token
    }
    
    try:
        response = requests.post(url, headers=headers, json=payload, timeout=60)
        response.raise_for_status()
        
        data = response.json()
        
        if data['result']['codResult'] == '0000':
            print(f"✓ Confirmación exitosa")
            print(f"  Documento: {data['docNumber']}")
            return data
        else:
            print(f"✗ Error: {data['result']['msgResult']}")
            return data
            
    except requests.exceptions.Timeout:
        print("✗ Error: Timeout - El servidor tardó demasiado en responder")
        return None
    except requests.exceptions.RequestException as e:
        print(f"✗ Error de conexión: {e}")
        return None

# Ejemplo de uso
if __name__ == "__main__":
    token_ejemplo = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    resultado = confirmar_registro(token_ejemplo)
```

### Ejemplo 4: Error - Token Inválido

#### Request

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/clientRegistrationConfirmation" \
  -H "Content-Type: application/json" \
  -d '{
    "token": "token_invalido_123"
  }'
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Token inválido o mal formado"
  }
}
```

### Ejemplo 5: Error - Token Expirado

#### Request

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/clientRegistrationConfirmation" \
  -H "Content-Type: application/json" \
  -d '{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.expired_token..."
  }'
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "El token ha expirado"
  }
}
```

**Nota:** En caso de token expirado, el cliente debe solicitar un nuevo email de confirmación mediante el endpoint correspondiente de reenvío de confirmación.
