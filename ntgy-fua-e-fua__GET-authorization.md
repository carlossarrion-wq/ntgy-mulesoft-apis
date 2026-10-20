---
api: "ntgy-fua-e-fua"
sidebar_position: 6
sidebar_label: "GET /authorization"
description: "Obtiene la lista de cuentas autorizadas para un cliente con información de roles, contratos y métodos de validación"
keywords: [authorization, autorizacion, roles, cuentas, cliente, validacion, firma, contratos, iban, salesforce, heroku, postgresql]
tags: [Authorization, Customer Management, Experience API, Web]
channel: web
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/authorization"
    purpose: "Orquesta la consulta de autorización del cliente contra base de datos Salesforce en Heroku"
---

# GET /authorization

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint obtiene la lista completa de cuentas autorizadas para un cliente específico, incluyendo información detallada sobre roles, permisos de firma, estado de contratos y métodos de validación disponibles. Es fundamental para el proceso de autorización y gestión de permisos en el portal FUA.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Orquestación de consulta de autorización con adaptación para canal web

### Patrón de Integración

**Tipo:** Pass-Through con Enriquecimiento Condicional

**Descripción:** El Experience API actúa como orquestador que:
1. Recibe la petición del cliente web
2. Valida headers de canal
3. Enruta condicionalmente según configuración `split-services.authorization`:
   - Si `true`: Llama a `ntgy-newco-p-authentication` (nuevo servicio de autenticación)
   - Si `false`: Llama a `ntgy-newco-p-fua-evo` (servicio actual)
4. Retorna la respuesta sin transformación adicional

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente Web<br/>Portal FUA]
    E[Experience API<br/>ntgy-fua-e-fua<br/>GET /authorization]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>GET /authorization]
    S[System API<br/>ntgy-newco-s-postgresql-salesforce<br/>POST /ejecutarQuery]
    DB[(PostgreSQL Heroku<br/>Réplica Salesforce)]
    
    C -->|Request con x-client-doc-number| E
    E -->|Choice: split-services.authorization| P
    P -->|Query SQL 1: accountcontactrelation| S
    P -->|Query SQL 2: contact alternativo| S
    S -->|Ejecuta Query| DB
    DB -->|Resultados| S
    S -->|Response| P
    P -->|Transform + Enrich| P
    P -->|Response| E
    E -->|Pass-Through| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style DB fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Validación de Autorización de Cliente**
   - Verificar qué cuentas puede gestionar un cliente específico
   - Determinar los roles y permisos asociados a cada cuenta
   - Identificar el estado de autorización (Activo, Pendiente)

2. **Gestión de Permisos de Firma**
   - Obtener roles permitidos para firma de documentos
   - Diferenciar permisos según tipo de documento (NIF/CIF)
   - Validar capacidad de firma del cliente

3. **Determinación de Método de Validación**
   - Identificar si el cliente tiene IBAN registrado
   - Determinar método de validación: "iban" o "importe"
   - Verificar existencia de contratos activos

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web

**Características del Canal:**
- **Plataforma**: Portal web FUA (Frontal Único de Atención)
- **Dispositivos**: Navegadores web (desktop y responsive)
- **Conectividad**: Conexión estable requerida
- **Experiencia de Usuario**: Interfaz de gestión de cuentas y autorizaciones

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Gestión de cuentas personales de energía
- **Clientes Empresariales**: Gestión de múltiples cuentas corporativas
- **Representantes Autorizados**: Gestión de cuentas de terceros con permisos específicos

**Contexto de Uso:**
- **Inicio de Sesión**: Validación de permisos tras autenticación
- **Gestión de Cuentas**: Selección de cuenta para operaciones
- **Firma de Documentos**: Validación de capacidad de firma
- **Cambio de Titular**: Verificación de roles autorizados

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Pass-Through**: No se aplican transformaciones adicionales, respuesta directa del Process API
- **Timeout Extendido**: 60 segundos para permitir consultas complejas en Salesforce
- **Validación de Headers**: Verificación de headers de canal (x-client-doc-number)

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/authorization` | GET | Orquesta la consulta de autorización del cliente contra base de datos Salesforce en Heroku |

**Nota:** Existe una configuración condicional (`split-services.authorization`) que permite enrutar a un servicio alternativo (`ntgy-newco-p-authentication`) cuando está habilitado.

### Contexto Empresarial

- **Dominio:** Gestión de Clientes y Autorizaciones
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Web (Portal FUA)

### Stakeholders

- **Clientes Finales**: Necesitan conocer sus permisos y cuentas autorizadas
- **Equipo de Desarrollo FUA**: Requiere información de autorización para habilitar funcionalidades
- **Equipo de Seguridad**: Valida que los permisos se gestionen correctamente
- **Equipo de Salesforce**: Mantiene la información de roles y relaciones en CRM

### Valor de Negocio

- **Seguridad**: Garantiza que solo usuarios autorizados accedan a cuentas específicas
- **Experiencia de Usuario**: Permite al cliente ver todas sus cuentas autorizadas en un solo lugar
- **Cumplimiento**: Asegura trazabilidad de permisos y roles
- **Eficiencia Operativa**: Automatiza la validación de autorizaciones sin intervención manual

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/authorization` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/authorization` |
| **UAT** | `https://ntgy-fua-e-fua-uat.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-uat.de-c1.cloudhub.io/fua/api/authorization` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/authorization` |

### Método HTTP

```
GET /fua/api/authorization
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Consultas complejas en Salesforce pueden requerir tiempo adicional |
| **Rate Limiting** | Según política API Gateway | Protección contra abuso |
| **Autenticación** | OAuth 2.0 requerido | Seguridad de datos sensibles |

#### Campos Específicos del Canal

**Campos Añadidos por el Process API (no por Experience):**

El Experience API actúa como pass-through, por lo que todos los campos provienen del Process API:

| Campo | Tipo | Descripción | Origen |
|-------|------|-------------|--------|
| `allowedSignRol` | String | Roles permitidos para firma | Properties del Process API |
| `allowedSignRolNif` | String | Roles permitidos para firma con NIF | Properties del Process API |
| `allowedSignRolCif` | String | Roles permitidos para firma con CIF | Properties del Process API |
| `authorization` | Array | Lista de cuentas autorizadas | Transformación de resultados de Heroku |

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `x-client-doc-number` | String | ✅ Sí | Número de documento del cliente (DNI/NIE/CIF) | `12345678A` |

**Nota:** El parámetro se envía como query parameter y es propagado al Process API.

### Estructura de Respuesta

#### Respuesta Exitosa (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "allowedSignRol": "01,03,04",
  "allowedSignRolNif": "01,03",
  "allowedSignRolCif": "01,04",
  "authorization": [
    {
      "nombre": "EMPRESA EJEMPLO SL",
      "rol": "01",
      "docnumber": "B12345678",
      "docType": "CIF",
      "isIndividualClient": false,
      "accountid": "0015I00000XxXxXXXX",
      "contactid": "0035I00000YyYyYYYY",
      "status": "Activo",
      "hasContracts": true,
      "validateMethod": "iban"
    },
    {
      "nombre": "JUAN PEREZ GARCIA",
      "rol": "03",
      "docnumber": "12345678A",
      "docType": "NIF",
      "isIndividualClient": true,
      "accountid": "0015I00000ZzZzZZZZ",
      "contactid": "0035I00000WwWwWWWW",
      "status": "Pendiente",
      "hasContracts": false,
      "validateMethod": "importe"
    }
  ]
}
```

#### Campos de Respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | String | Código de resultado ("0000" = éxito) |
| `result.msgResult` | String | Mensaje descriptivo del resultado |
| `allowedSignRol` | String | Roles permitidos para firma (separados por coma) |
| `allowedSignRolNif` | String | Roles permitidos para firma con NIF |
| `allowedSignRolCif` | String | Roles permitidos para firma con CIF |
| `authorization` | Array | Lista de cuentas autorizadas |
| `authorization[].nombre` | String | Nombre de la cuenta |
| `authorization[].rol` | String | Código de rol (01=Titular, 03=Autorizado, 04=Representante, 05=Consulta, 06=Otro) |
| `authorization[].docnumber` | String | Número de documento de la cuenta |
| `authorization[].docType` | String | Tipo de documento (NIF/CIF/NIE) |
| `authorization[].isIndividualClient` | Boolean | Indica si es cliente individual (true) o empresa (false) |
| `authorization[].accountid` | String | ID de la cuenta en Salesforce |
| `authorization[].contactid` | String | ID del contacto en Salesforce |
| `authorization[].status` | String | Estado de autorización (Activo/Pendiente) |
| `authorization[].hasContracts` | Boolean | Indica si tiene contratos asociados |
| `authorization[].validateMethod` | String | Método de validación ("iban" si tiene IBAN, "importe" si no) |

#### Respuesta de Error - Cliente Sin Cuentas (200 OK con error de negocio)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "No hay datos asociados al docNumber"
  }
}
```

#### Respuesta de Error - Error Técnico (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0005",
    "msgResult": "Error en la consulta a base de datos"
  }
}
```

### Códigos de Respuesta HTTP

| Código | Descripción | Escenario |
|--------|-------------|-----------|
| `200 OK` | Solicitud exitosa | Cliente con cuentas autorizadas encontradas |
| `200 OK` | Error de negocio | Cliente sin cuentas asociadas (codResult != "0000") |
| `400 Bad Request` | Parámetros inválidos | Falta x-client-doc-number o formato inválido |
| `401 Unauthorized` | No autenticado | Token OAuth 2.0 inválido o expirado |
| `500 Internal Server Error` | Error técnico | Error en Process API o System API |
| `504 Gateway Timeout` | Timeout | Consulta excede 60 segundos |

### Autenticación

- **Tipo:** OAuth 2.0
- **Header:** `Authorization: Bearer {token}`
- **Scope:** Acceso a información de autorización del cliente

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Consultas complejas en Salesforce pueden requerir tiempo adicional |
| **Connection Idle Timeout** | 60 segundos | Mantener conexión para operaciones largas |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request<br/>x-client-doc-number]) --> LogStart[Log: get:authorization:e-fua-config]
    LogStart --> FlowRef[Flow-Ref: get-authorization]
    FlowRef --> Choice{Choice:<br/>split-services.authorization}
    
    Choice -->|true| AuthNew[HTTP Request<br/>ntgy-newco-p-authentication<br/>GET /authorization]
    Choice -->|false| AuthEvo[HTTP Request<br/>ntgy-newco-p-fua-evo<br/>GET /authorization]
    
    AuthNew --> Response[Return Response]
    AuthEvo --> Response
    Response --> End([HTTP Response])
    
    FlowRef -.->|On Error| ErrorHandler[Error Handler]
    ErrorHandler --> LogError[Log Error]
    LogError --> TransformError[Transform Error Response]
    TransformError --> ErrorResponse([Error Response])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style ErrorResponse fill:#ffebee,stroke:#c62828
    style Choice fill:#fff9c4,stroke:#f57f17
```

### Transformaciones de Canal

⚠️ **Este Experience API NO transforma la respuesta del Process API.**

El endpoint actúa como **pass-through puro** con enrutamiento condicional. La respuesta del Process API se retorna directamente al cliente sin modificaciones.

Para el schema completo y transformaciones de datos, consultar:
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `/authorization`**

### Patrones de Diseño

**Patrón:** Content-Based Routing (Enrutamiento Basado en Contenido)

**Justificación:** 
- Permite migración gradual entre servicios de autenticación
- Configuración dinámica mediante property `split-services.authorization`
- Facilita testing A/B y rollback en caso de problemas
- Mantiene compatibilidad con ambos servicios simultáneamente

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito (Ruta Principal: ntgy-newco-p-fua-evo)

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant S as System API<br/>s-postgresql-salesforce
    participant DB as PostgreSQL Heroku<br/>(Salesforce Replica)
    
    C->>E: GET /authorization?x-client-doc-number=12345678A
    Note over E: Log: get:authorization:e-fua-config
    Note over E: Flow-Ref: get-authorization
    Note over E: Choice: split-services.authorization = false
    
    E->>P: GET /authorization?x-client-doc-number=12345678A
    Note over P: Log: Inicio authorization
    Note over P: Construir Query SQL 1<br/>(accountcontactrelation)
    
    P->>S: POST /ejecutarQuery<br/>{query: "SELECT..."}
    S->>DB: Execute SQL Query
    DB-->>S: Query Results
    S-->>P: Response con datos
    
    alt Resultados vacíos
        Note over P: Construir Query SQL 2<br/>(contact)
        P->>S: POST /ejecutarQuery<br/>{query: "SELECT..."}
        S->>DB: Execute SQL Query
        DB-->>S: Query Results
        S-->>P: Response con datos
    end
    
    alt Sin resultados en ambas queries
        Note over P: Raise Error:<br/>CLIENT_NOT_ASSOCIATED_ACCOUNTS
    else Con resultados
        Note over P: Transform Response<br/>+ Add allowedSignRol properties
        P-->>E: Response con authorization[]
    end
    
    E-->>C: Pass-Through Response
    Note over C: Display authorized accounts
```

#### 2. Flujo con Error de Negocio (Cliente Sin Cuentas)

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API
    participant P as Process API
    participant S as System API
    participant DB as PostgreSQL Heroku
    
    C->>E: GET /authorization?x-client-doc-number=99999999X
    E->>P: GET /authorization
    
    P->>S: POST /ejecutarQuery (Query 1)
    S->>DB: Execute Query
    DB-->>S: Empty Result
    S-->>P: []
    
    P->>S: POST /ejecutarQuery (Query 2)
    S->>DB: Execute Query
    DB-->>S: Empty Result
    S-->>P: []
    
    Note over P: isEmpty(queryResponse) = true
    Note over P: Raise Error:<br/>CLIENT_NOT_ASSOCIATED_ACCOUNTS
    
    P-->>E: Error Response<br/>{codResult: "0001"}
    E-->>C: Pass-Through Error
    Note over C: Display "No hay datos asociados"
```

#### 3. Flujo con Error Técnico (Timeout en Base de Datos)

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API
    participant P as Process API
    participant S as System API
    participant DB as PostgreSQL Heroku
    
    C->>E: GET /authorization?x-client-doc-number=12345678A
    E->>P: GET /authorization
    
    P->>S: POST /ejecutarQuery (Query 1)
    S->>DB: Execute Query
    Note over DB: Timeout after 240s
    DB--XS: Timeout
    S--XP: Error: Timeout
    
    Note over P: Error Handler
    P-->>E: Error Response<br/>{codResult: "0005"}
    E-->>C: Pass-Through Error
    Note over C: Display error message
```

### Manejo de Errores

#### Tipos de Errores

| Tipo de Error | Código | Descripción | Acción |
|---------------|--------|-------------|--------|
| **Cliente Sin Cuentas** | `APP:CLIENT_NOT_ASSOCIATED_ACCOUNTS` | No se encontraron cuentas asociadas al documento | Retornar codResult "0001" |
| **Error PostgreSQL** | `APP:POSTGRESQL_SALESFORCE` | Error en consulta a base de datos | Retornar codResult "0005" |
| **Timeout** | `MULE:TIMEOUT` | Timeout en consulta (>60s) | Retornar codResult "0005" |
| **Error Genérico** | `ANY` | Cualquier otro error no controlado | Retornar codResult "0005" |

#### Estrategia de Error Handling

```xml
<error-handler>
    <on-error-propagate>
        <logger level="ERROR" message="Error en authorization"/>
        <ee:transform>
            <ee:set-payload>
                if (!isEmpty(error.errorMessage.payload)) 
                    error.errorMessage.payload
                else {
                    "result": {
                        "codResult": "0005",
                        "msgResult": "KO"
                    }
                }
            </ee:set-payload>
        </ee:transform>
    </on-error-propagate>
</error-handler>
```

### Optimizaciones del Canal

**Configuración de Timeout:**
- Experience API: 60 segundos
- Process API: 60 segundos  
- System API: 240 segundos (4 minutos)

**Razón:** Las consultas SQL en PostgreSQL/Heroku pueden ser complejas debido a múltiples JOINs entre tablas de Salesforce.

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Llamada Exitosa con Múltiples Cuentas

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/authorization?x-client-doc-number=12345678A" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "allowedSignRol": "01#03#04#06",
  "allowedSignRolNif": "01#04",
  "allowedSignRolCif": "01#03#04#06",
  "authorization": [
    {
      "nombre": "EMPRESA EJEMPLO SL",
      "rol": "01",
      "docnumber": "B12345678",
      "docType": "CIF",
      "isIndividualClient": false,
      "accountid": "0015I00000XxXxXXXX",
      "contactid": "0035I00000YyYyYYYY",
      "status": "Activo",
      "hasContracts": true,
      "validateMethod": "iban"
    },
    {
      "nombre": "JUAN PEREZ GARCIA",
      "rol": "03",
      "docnumber": "12345678A",
      "docType": "NIF",
      "isIndividualClient": true,
      "accountid": "0015I00000ZzZzZZZZ",
      "contactid": "0035I00000WwWwWWWW",
      "status": "Activo",
      "hasContracts": true,
      "validateMethod": "iban"
    }
  ]
}
```

### Ejemplo 2: Cliente Sin Cuentas Asociadas

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/authorization?x-client-doc-number=99999999X" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "No hay datos asociados al docNumber"
  }
}
```

### Ejemplo 3: JavaScript (Frontend Web)

```javascript
async function getAuthorization(docNumber) {
  try {
    const response = await fetch(
      `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/authorization?x-client-doc-number=${docNumber}`,
      {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${getAuthToken()}`
        }
      }
    );
    
    const data = await response.json();
    
    if (data.result.codResult === "0000") {
      console.log('Cuentas autorizadas:', data.authorization);
      return data.authorization;
    } else {
      console.error('Error:', data.result.msgResult);
      return [];
    }
  } catch (error) {
    console.error('Error en la petición:', error);
    throw error;
  }
}

// Uso
const cuentas = await getAuthorization('12345678A');
```

### Ejemplo 4: Python

```python
import requests

def get_authorization(doc_number, token):
    """
    Obtiene las cuentas autorizadas para un cliente
    
    Args:
        doc_number (str): Número de documento del cliente
        token (str): Token de autenticación OAuth 2.0
    
    Returns:
        list: Lista de cuentas autorizadas
    """
    url = "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/authorization"
    
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {token}"
    }
    
    params = {
        "x-client-doc-number": doc_number
    }
    
    try:
        response = requests.get(url, headers=headers, params=params, timeout=60)
        response.raise_for_status()
        
        data = response.json()
        
        if data.get("result", {}).get("codResult") == "0000":
            return data.get("authorization", [])
        else:
            print(f"Error: {data.get('result', {}).get('msgResult')}")
            return []
            
    except requests.exceptions.Timeout:
        print("Error: Timeout en la petición")
        return []
    except requests.exceptions.RequestException as e:
        print(f"Error en la petición: {e}")
        return []

# Uso
cuentas = get_authorization("12345678A", "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...")
for cuenta in cuentas:
    print(f"Cuenta: {cuenta['nombre']} - Rol: {cuenta['rol']}")
```

### Ejemplo 5: Error de Timeout

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/authorization?x-client-doc-number=12345678A" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  --max-time 60
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

---

## 📊 Información Adicional

### Roles de Autorización

| Código | Descripción | Permisos |
|--------|-------------|----------|
| `01` | Titular | Permisos completos sobre la cuenta |
| `03` | Autorizado | Permisos de gestión y consulta |
| `04` | Representante | Permisos de representación legal |
| `05` | Consulta | Solo permisos de lectura |
| `06` | Otro | Permisos específicos definidos |

### Métodos de Validación

| Método | Descripción | Cuándo se Usa |
|--------|-------------|---------------|
| `iban` | Validación por IBAN | Cliente tiene IBAN registrado en Salesforce |
| `importe` | Validación por importe | Cliente NO tiene IBAN registrado |

### Estados de Autorización

| Estado | Descripción |
|--------|-------------|
| `Activo` | Autorización activa y operativa |
| `Pendiente` | Autorización pendiente de activación |

### Properties de Firma

Las properties de firma se obtienen del archivo de configuración del Process API:

```yaml
firma:
  allowedSignRol: "01#03#04#06"
  allowedSignRolNif: "01#04"
  allowedSignRolCif: "01#03#04#06"
```

Estas properties determinan qué roles pueden firmar documentos según el tipo de documento (NIF o CIF).
