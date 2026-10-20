---
api: "ntgy-fua-e-fua"
sidebar_position: 56
sidebar_label: "POST /createDelegation"
description: "Orquesta la creación de delegaciones de permisos en el área de cliente con enrutamiento condicional entre servicios de autenticación"
keywords: [delegacion, permisos, autorizacion, area-cliente, fua, enrutamiento, migracion, cognito, salesforce]
tags: [Delegaciones, Autenticación, Experience API, Portal FUA, Enrutamiento Condicional]
channel: web
endpoint_dependencies:
  - method: "POST"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/createDelegation"
    purpose: "Orquesta el proceso completo de creación de delegación con validación de cliente, registro en Cognito, Salesforce y envío de notificaciones"
  - method: "POST"
    api_name: "ntgy-newco-p-authentication"
    path: "/createDelegation"
    purpose: "Servicio alternativo de autenticación (actualmente no activo, preparado para migración futura)"
---

# POST /createDelegation

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint del Experience API orquesta la creación de delegaciones de permisos en el área de cliente FUA (Frontal Único de Atención). Actúa como punto de entrada único para el portal web, implementando una estrategia de enrutamiento condicional que permite migrar gradualmente entre diferentes servicios de autenticación sin impacto en los clientes.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Orquestador con enrutamiento condicional y estrategia de migración

### Patrón de Integración

**Tipo:** Pass-Through con Enrutamiento Condicional

**Descripción:** El endpoint no transforma datos, sino que actúa como router inteligente que:
- Propaga el payload original sin modificaciones
- Añade headers de contexto FUA (`vars.fuaHeaders`)
- Enruta condicionalmente según configuración (`split-services.createDelegation`)
- Permite cambio de backend sin modificar código (solo configuración)

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente<br/>Portal FUA Web]
    E[Experience API<br/>ntgy-fua-e-fua<br/>POST /createDelegation]
    P1[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /createDelegation]
    P2[Process API<br/>ntgy-newco-p-authentication<br/>POST /createDelegation]
    S1[System APIs<br/>DynamoDB, Cognito,<br/>Salesforce, PostgreSQL,<br/>Marketing Cloud]
    
    C -->|Request + token| E
    E -->|split-services = false<br/>ACTIVO| P1
    E -.->|split-services = true<br/>FUTURO| P2
    P1 -->|Orquestación completa| S1
    P2 -.->|Servicio nuevo| S1
    S1 -->|Response| P1
    P1 -->|Response| E
    E -->|Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P1 fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style P2 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,stroke-dasharray: 5 5
    style S1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Alta de Usuario Autorizado desde Portal Web**
   - Cliente titular accede al portal FUA para autorizar a otra persona
   - Sistema valida token de registro y crea usuario con permisos específicos
   - Beneficio: Proceso automatizado sin intervención de agentes

2. **Migración Gradual entre Servicios**
   - Permite cambiar entre servicios de autenticación mediante configuración
   - No requiere cambios en el código ni redespliegue
   - Beneficio: Rollback inmediato en caso de problemas

3. **Propagación de Contexto FUA**
   - Mantiene trazabilidad completa con headers específicos del canal
   - Preserva información de sesión y usuario
   - Beneficio: Auditoría completa y troubleshooting eficiente

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web (Portal FUA)

**Características del Canal:**
- **Plataforma**: Portal web responsive de autoservicio
- **Dispositivos**: Desktop y tablets
- **Conectividad**: Conexión estable requerida
- **Experiencia de Usuario**: Flujo guiado con validación en tiempo real

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Titulares**: Personas que desean delegar gestión de su cuenta de energía
- **Usuarios Autorizados**: Familiares o representantes que gestionarán la cuenta

**Contexto de Uso:**
- **Registro Inicial**: Completar proceso de alta en área de cliente
- **Delegación de Permisos**: Autorizar a terceros para gestión de cuenta
- **Autoservicio**: Proceso sin intervención de call center

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Propagación de Headers**: Mantiene contexto de sesión FUA
- **Timeout Extendido**: 60 segundos para operaciones complejas
- **Enrutamiento Transparente**: Cliente no percibe cambio de backend
- **Manejo de Errores**: Transformación a formato estándar FUA

### 🔗 Endpoints Dependientes

Este Experience API consume los siguientes Process APIs:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/createDelegation` | POST | Orquesta el proceso completo de creación de delegación (ACTIVO) |
| `ntgy-newco-p-authentication` | `/createDelegation` | POST | Servicio alternativo de autenticación (PREPARADO, no activo) |

**Nota sobre Dependencias:** El Process API activo (`ntgy-newco-p-fua-evo`) consume múltiples System APIs:
- DynamoDB: Gestión de tokens de registro
- PostgreSQL-Salesforce: Validación de contactos existentes
- Cognito: Gestión de identidades de usuario
- Salesforce: Creación/actualización de contactos y relaciones
- Marketing Cloud: Envío de emails de bienvenida

### Contexto Empresarial

- **Dominio:** Autoservicio de Clientes - Gestión de Delegaciones
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Portal Web FUA

### Stakeholders

- **Clientes Finales**: Necesitan delegar gestión de cuenta de forma autónoma
- **Equipo de Producto FUA**: Requiere flexibilidad para migrar servicios
- **Equipo de Operaciones**: Necesita capacidad de rollback sin redespliegue
- **Equipo de Desarrollo**: Mantiene dos servicios en paralelo durante migración

### Valor de Negocio

- **Flexibilidad Operacional**: Cambio de backend sin impacto en cliente
- **Reducción de Riesgo**: Migración gradual con rollback inmediato
- **Autonomía del Cliente**: Proceso completamente automatizado
- **Trazabilidad Completa**: Headers FUA para auditoría y troubleshooting

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/createDelegation?token=:token` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/createDelegation?token=:token` |
| **UAT** | `https://ntgy-fua-e-fua-uat.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-uat.de-c1.cloudhub.io/fua/api/createDelegation?token=:token` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/createDelegation?token=:token` |

### Método HTTP

```
POST /fua/api/createDelegation
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Operaciones complejas en múltiples sistemas backend |
| **Rate Limiting** | Según política API Gateway | Protección contra abuso |
| **Autenticación** | JWT token requerido | Validado en `jwt-validation` flow |

#### Campos Específicos del Canal

**Headers Propagados por el Experience API:**

| Header | Tipo | Descripción | Propósito |
|--------|------|-------------|-----------|
| `x-interaction-id` | string | ID único de interacción | Trazabilidad end-to-end |
| `x-interaction-origin` | string | Origen de la interacción | Identificar canal (web/mobile) |
| `x-user-id` | string | ID del usuario autenticado | Contexto de usuario |
| `x-client-id` | string | ID del cliente | Identificación del cliente |
| `x-client-doc-number` | string | Documento del cliente | Validación de identidad |
| `x-client-code` | string | Código del cliente | Referencia interna |
| `x-origin` | string | Origen de la petición | Auditoría |
| `x-user-role` | string | Rol del usuario | Control de acceso |
| `traceparent` | string | W3C Trace Context | Distributed tracing |
| `x-agent-mail` | string | Email del agente (si aplica) | Contexto de agente |
| `x-language` | string | Idioma preferido | Localización |
| `x-contact-id` | string | ID del contacto | Referencia Salesforce |

**Nota:** Estos headers se extraen del request original y se propagan al Process API en `vars.fuaHeaders`.

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `token` | string | Sí | Token de registro para recuperar datos de DynamoDB |

#### Request Body (application/json)

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `docNumber` | string | No | Documento del autorizado |
| `docType` | string | No | Tipo de documento (NIF, NIE, etc.) |
| `nombre` | string | No | Nombre del autorizado |
| `apellido1` | string | No | Primer apellido del autorizado |
| `apellido2` | string | No | Segundo apellido del autorizado |
| `email` | string | No | Email del autorizado |
| `password` | string | No | Contraseña a asignar del usuario |

**Ejemplo de Request Body:**

```json
{
  "docNumber": "12345678A",
  "docType": "NIF",
  "nombre": "Juan",
  "apellido1": "García",
  "apellido2": "López",
  "email": "juan.garcia@example.com",
  "password": "SecurePass123!"
}
```

### Respuestas

#### 200 OK - Delegación Creada Exitosamente

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  }
}
```

#### 400 Bad Request - Error de Validación

```json
{
  "message": "Bad request"
}
```

#### 500 Internal Server Error - Error del Sistema

```json
{
  "result": {
    "codResult": "0005",
    "msgResult": "KO"
  }
}
```

**Nota:** La respuesta es pass-through del Process API. Para detalles completos de códigos de error, consultar documentación del Process API `ntgy-newco-p-fua-evo`.

### Headers Requeridos

| Header | Valor | Descripción |
|--------|-------|-------------|
| `Content-Type` | `application/json` | Formato del body |
| `Authorization` | `Bearer {JWT_TOKEN}` | Token JWT validado por `jwt-validation` flow |

**Headers Opcionales (Contexto FUA):**
- `x-interaction-id`: ID de interacción
- `x-user-id`: ID del usuario
- `x-client-doc-number`: Documento del cliente
- `traceparent`: Trace context W3C
- Otros headers FUA (ver tabla anterior)

### Autenticación

- **Tipo:** JWT Bearer Token
- **Validación:** Flow `jwt-validation` ejecutado antes del enrutamiento
- **Ubicación:** Header `Authorization`

### Timeouts

| Tipo | Valor | Configuración |
|------|-------|---------------|
| **Request Timeout (fua-evo)** | 60 segundos | `${newco-p-fua-evo.timeout}` |
| **Request Timeout (authentication)** | 600 segundos | `${p-authentication.timeout}` |
| **Connection Idle Timeout** | 60 segundos | Configurado en HTTP request config |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request]) --> JWT[JWT Validation]
    JWT --> Headers[Extract FUA Headers]
    Headers --> Router[APIKit Router]
    Router --> Logger[Log Request]
    Logger --> FlowRef[Flow-Ref:<br/>post-createDelegation]
    
    FlowRef --> Choice{split-services<br/>.createDelegation?}
    
    Choice -->|true<br/>FUTURO| Auth[HTTP Request:<br/>p-authentication]
    Choice -->|false<br/>ACTIVO| FuaEvo[HTTP Request:<br/>p-fua-evo]
    
    Auth --> AddHeaders1[Add fuaHeaders<br/>+ token param]
    FuaEvo --> AddHeaders2[Add fuaHeaders<br/>+ token param]
    
    AddHeaders1 --> Response1[Response]
    AddHeaders2 --> Response2[Response]
    
    Response1 --> End([HTTP Response])
    Response2 --> End
    
    FlowRef -.->|Error| ErrorHandler[Error Handler]
    ErrorHandler --> LogError[Log Error]
    LogError --> TransformError[Transform Error<br/>to Standard Format]
    TransformError --> ErrorResponse([HTTP Error Response])
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style Choice fill:#fff3e0,stroke:#f57c00,stroke-width:3px
    style Auth fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,stroke-dasharray: 5 5
    style FuaEvo fill:#e1f5fe,stroke:#0277bd,stroke-width:2px
    style ErrorHandler fill:#ffebee,stroke:#c62828,stroke-width:2px
    style End fill:#e8f5e9,stroke:#2e7d32
```

### Transformaciones de Canal

**⚠️ Este Experience API NO transforma la respuesta del Process API.**

El endpoint actúa como pass-through puro, solo añadiendo contexto:

| Operación | Descripción | Ubicación en Código |
|-----------|-------------|---------------------|
| **Extracción de Headers** | Extrae headers FUA del request original | `e-fua.xml` líneas 15-37 |
| **Propagación de Payload** | Envía payload sin modificaciones | `post-createDelegation.xml` líneas 14-21, 25-32 |
| **Propagación de Token** | Añade query param `token` | `post-createDelegation.xml` líneas 18-21, 29-32 |
| **Propagación de Headers** | Añade `vars.fuaHeaders` al body | `post-createDelegation.xml` líneas 14-17, 25-28 |

**Transformación de Headers FUA:**

```dataweave
%dw 2.2
output application/java
---
{
  ("x-interaction-id": attributes.headers['x-interaction-id']) if (attributes.headers['x-interaction-id'] != "null" and !isEmpty(attributes.headers['x-interaction-id'])),
  ("x-interaction-origin": attributes.headers['x-interaction-origin']) if (attributes.headers['x-interaction-origin'] != "null" and !isEmpty(attributes.headers['x-interaction-origin'])),
  ("x-user-id": attributes.headers['x-user-id']) if (attributes.headers['x-user-id'] != "null" and !isEmpty(attributes.headers['x-user-id'])),
  // ... otros headers FUA
}
```

**Para el schema completo de request/response, consultar:**
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `/createDelegation`**

### Patrones de Diseño

**Patrón 1: Enrutamiento Condicional (Feature Toggle)**

**Justificación:** Permite migración gradual entre servicios sin cambios de código. El flag `split-services.createDelegation` controla qué backend se usa.

**Implementación:**
```xml
<choice>
  <when expression="#[(Mule::p('split-services.createDelegation') as Boolean default false)]">
    <!-- Ruta a p-authentication (FUTURO) -->
  </when>
  <otherwise>
    <!-- Ruta a p-fua-evo (ACTIVO) -->
  </otherwise>
</choice>
```

**Patrón 2: Pass-Through con Enriquecimiento de Contexto**

**Justificación:** Mantiene simplicidad del Experience API mientras añade contexto necesario para trazabilidad.

**Patrón 3: Estrategia de Migración Blue-Green**

**Justificación:** Dos backends en paralelo permiten:
- Testing en producción con usuarios seleccionados
- Rollback inmediato cambiando configuración
- Validación gradual del nuevo servicio

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito (Ruta Activa: fua-evo)

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API
    participant JWT as JWT Validation
    participant P as Process API<br/>(fua-evo)
    participant S as System APIs
    
    C->>E: POST /createDelegation?token=xyz
    Note over E: Extract FUA Headers
    E->>JWT: Validate JWT Token
    JWT-->>E: Token Valid
    
    E->>E: Check split-services.createDelegation
    Note over E: Value = false (ACTIVO)
    
    E->>P: POST /createDelegation<br/>+ payload<br/>+ fuaHeaders<br/>+ token param
    
    Note over P: Orquestación completa
    P->>S: DynamoDB, Cognito,<br/>Salesforce, etc.
    S-->>P: Responses
    
    P-->>E: 200 OK<br/>{result: {codResult: "0000"}}
    E-->>C: 200 OK<br/>(pass-through)
```

#### 2. Flujo Alternativo (Ruta Futura: authentication)

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API
    participant P2 as Process API<br/>(authentication)
    
    C->>E: POST /createDelegation?token=xyz
    
    E->>E: Check split-services.createDelegation
    Note over E: Value = true (FUTURO)
    
    E->>P2: POST /createDelegation<br/>+ payload<br/>+ fuaHeaders<br/>+ token param
    
    Note over P2: Nuevo servicio<br/>de autenticación
    P2-->>E: 200 OK
    E-->>C: 200 OK<br/>(pass-through)
```

#### 3. Flujo con Error

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API
    participant P as Process API
    participant EH as Error Handler
    
    C->>E: POST /createDelegation?token=invalid
    
    E->>P: POST /createDelegation
    P-->>E: 500 Error
    
    E->>EH: Handle Error
    Note over EH: Log error message
    
    EH->>EH: Transform Error
    Note over EH: if (!isEmpty(error.errorMessage.payload))<br/>  return error.errorMessage.payload<br/>else<br/>  return {codResult: "0005", msgResult: "KO"}
    
    EH-->>C: 500 Internal Server Error<br/>{result: {codResult: "0005"}}
```

### Configuración de Enrutamiento

**Archivo:** `e-fua-pro-config.yaml`

```yaml
# Configuración de enrutamiento condicional
split-services:
  createDelegation: "false"  # false = fua-evo (ACTIVO)
                             # true = authentication (FUTURO)

# Configuración del Process API activo
newco-p-fua-evo:
  host: "mule-worker-internal-ntgy-newco-p-fua-evo.de-c1.cloudhub.io"
  port: "8091"
  timeout: "60000"

# Configuración del Process API futuro
p-authentication:
  host: "ntgy-newco-p-authentication-vx8fhl.internal-6bnlu3.irl-e1.cloudhub.io"
  port: "443"
  protocol: "HTTPS"
  timeout: "600000"
  basePath: "/api/v1"
```

**Para cambiar de backend:**
1. Modificar `split-services.createDelegation` a `"true"`
2. Redesplegar configuración (sin cambios de código)
3. Validar funcionamiento
4. Rollback inmediato si hay problemas (cambiar a `"false"`)

### Manejo de Errores

**Error Handler:** `on-error-propagate`

**Estrategia:**

1. **Log del Error**: Registra el error completo para troubleshooting
2. **Transformación Condicional**: 
   - Si existe `error.errorMessage.payload` → Propagar tal cual
   - Si no existe → Crear error estándar con código "0005"
3. **Propagación**: El error se propaga al cliente (no se captura)

**Código del Error Handler:**

```xml
<error-handler>
  <on-error-propagate>
    <logger level="ERROR" message='#[if (!isEmpty(error.errorMessage.payload)) 
      error.errorMessage.payload
    else 
      {
        "result": {
          "codResult": "0005",
          "msgResult": error.description as String replace /"/ with("")
        }
      }]' />
    
    <ee:transform>
      <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
if (!isEmpty(error.errorMessage.payload)) 
  error.errorMessage.payload
else 
  {
    "result": {
      "codResult": "0005",
      "msgResult": "KO"
    }
  }]]></ee:set-payload>
    </ee:transform>
  </on-error-propagate>
</error-handler>
```

**Códigos de Error Comunes:**

| Código | Descripción | Origen | Acción |
|--------|-------------|--------|--------|
| `0000` | Sin error | Process API | Operación exitosa |
| `0005` | Error genérico | Experience API | Error no específico del Process |
| Otros | Errores específicos | Process API | Ver documentación del Process API |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Llamada Exitosa desde Portal Web

#### cURL

```bash
curl -X POST "https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/createDelegation?token=abc123xyz" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-interaction-id: 550e8400-e29b-41d4-a716-446655440000" \
  -H "x-user-id: user123" \
  -H "x-client-doc-number: 12345678A" \
  -d '{
    "docNumber": "87654321B",
    "docType": "NIF",
    "nombre": "María",
    "apellido1": "Martínez",
    "apellido2": "Sánchez",
    "email": "maria.martinez@example.com",
    "password": "SecurePass456!"
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  }
}
```

**Resultado:**
- ✅ Request enrutado a `ntgy-newco-p-fua-evo`
- ✅ Headers FUA propagados
- ✅ Delegación creada exitosamente
- ✅ Usuario registrado en Cognito
- ✅ Email de bienvenida enviado

---

### Ejemplo 2: JavaScript (Fetch API) - Portal Web

```javascript
const createDelegation = async (token, delegationData) => {
  try {
    const response = await fetch(
      `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/createDelegation?token=${token}`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${localStorage.getItem('jwt_token')}`,
          'x-interaction-id': generateInteractionId(),
          'x-user-id': getCurrentUserId(),
          'x-client-doc-number': getCurrentUserDoc()
        },
        body: JSON.stringify(delegationData)
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    
    if (data.result.codResult === '0000') {
      console.log('✅ Delegación creada exitosamente');
      return { success: true, data };
    } else {
      console.error('❌ Error en la creación:', data.result.msgResult);
      return { success: false, error: data.result };
    }
  } catch (error) {
    console.error('❌ Error de red:', error);
    throw error;
  }
};

// Uso en el portal
const delegationData = {
  docNumber: '87654321B',
  docType: 'NIF',
  nombre: 'María',
  apellido1: 'Martínez',
  apellido2: 'Sánchez',
  email: 'maria.martinez@example.com',
  password: 'SecurePass456!'
};

createDelegation('abc123xyz', delegationData)
  .then(result => {
    if (result.success) {
      showSuccessMessage('Delegación creada correctamente');
      redirectToConfirmation();
    } else {
      showErrorMessage(result.error.msgResult);
    }
  })
  .catch(error => {
    showErrorMessage('Error de conexión. Por favor, inténtelo más tarde.');
  });
```

---

### Ejemplo 3: Python (Requests) - Testing/Automation

```python
import requests
import json
from typing import Dict, Optional

def create_delegation(
    token: str,
    delegation_data: Dict,
    jwt_token: str,
    base_url: str = "https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io"
) -> Dict:
    """
    Crea una delegación en el área de cliente FUA
    
    Args:
        token: Token de registro
        delegation_data: Datos del usuario autorizado
        jwt_token: Token JWT de autenticación
        base_url: URL base del API
    
    Returns:
        dict: Respuesta del API
    """
    url = f"{base_url}/fua/api/createDelegation"
    
    headers = {
        'Content-Type': 'application/json',
        'Authorization': f'Bearer {jwt_token}',
        'x-interaction-id': 'test-interaction-001',
        'x-user-id': 'test-user-123'
    }
    
    params = {'token': token}
    
    try:
        response = requests.post(
            url,
            headers=headers,
            params=params,
            json=delegation_data,
            timeout=60
        )
        
        response.raise_for_status()
        result = response.json()
        
        if result.get('result', {}).get('codResult') == '0000':
            print(f"✅ Delegación creada exitosamente")
            return {'success': True, 'data': result}
        else:
            print(f"❌ Error: {result.get('result', {}).get('msgResult')}")
            return {'success': False, 'error': result}
            
    except requests.exceptions.Timeout:
        print("❌ Timeout: El servidor tardó demasiado en responder")
        return {'success': False, 'error': 'TIMEOUT'}
    except requests.exceptions.RequestException as e:
        print(f"❌ Error de red: {str(e)}")
        return {'success': False, 'error': str(e)}

# Ejemplo de uso
if __name__ == "__main__":
    delegation_data = {
        "docNumber": "87654321B",
        "docType": "NIF",
        "nombre": "María",
        "apellido1": "Martínez",
        "apellido2": "Sánchez",
        "email": "maria.martinez@example.com",
        "password": "SecurePass456!"
    }
    
    jwt_token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    token = "abc123xyz"
    
    result = create_delegation(token, delegation_data, jwt_token)
    
    if result['success']:
        print("✅ Proceso completado exitosamente")
    else:
        print(f"❌ Error en el proceso: {result['error']}")
```

---

### Ejemplo 4: Error - Token Inválido

#### Request

```bash
curl -X POST "https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/createDelegation?token=invalid_token" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -d '{
    "docNumber": "99887766D",
    "docType": "NIF",
    "nombre": "Ana",
    "apellido1": "López",
    "email": "ana.lopez@example.com",
    "password": "Pass123!"
  }'
```

#### Respuesta (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0005",
    "msgResult": "KO"
  }
}
```

**Causa:** Token no encontrado en DynamoDB o expirado.

---

### Ejemplo 5: Postman Collection Snippet

```json
{
  "info": {
    "name": "FUA - Create Delegation",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Create Delegation",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          },
          {
            "key": "Authorization",
            "value": "Bearer {{jwt_token}}"
          },
          {
            "key": "x-interaction-id",
            "value": "{{$guid}}"
          },
          {
            "key": "x-user-id",
            "value": "{{user_id}}"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"docNumber\": \"87654321B\",\n  \"docType\": \"NIF\",\n  \"nombre\": \"María\",\n  \"apellido1\": \"Martínez\",\n  \"apellido2\": \"Sánchez\",\n  \"email\": \"maria.martinez@example.com\",\n  \"password\": \"SecurePass456!\"\n}"
        },
        "url": {
          "raw": "{{base_url}}/fua/api/createDelegation?token={{token}}",
          "host": ["{{base_url}}"],
          "path": ["fua", "api", "createDelegation"],
          "query": [
            {
              "key": "token",
              "value": "{{token}}"
            }
          ]
        }
      }
    }
  ]
}
```

**Variables de Entorno Postman:**
```json
{
  "base_url": "https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io",
  "jwt_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token": "abc123xyz",
  "user_id": "user123"
}
```
