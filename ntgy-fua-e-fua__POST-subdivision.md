---
api: "ntgy-fua-e-fua"
sidebar_position: 86
sidebar_label: "POST /subdivision"
description: "Solicita un nuevo fraccionamiento de pago para facturas del cliente"
keywords: [fraccionamiento, subdivision, solicitud, cuotas, pago, SAP ISU]
tags: [experience-api, fua, subdivision, creacion]
channel: web-app
endpoint_dependencies:
  - method: "POST"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/subdivision"
    purpose: "Creación de fraccionamiento en Process API"
---

# POST /subdivision

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite solicitar un nuevo fraccionamiento de pago para una o más facturas del cliente. El fraccionamiento divide el monto total de las facturas en cuotas mensuales, facilitando el pago al cliente y mejorando la cobranza.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Orquestación y adaptación de experiencia para canal web/móvil

### Patrón de Integración

**Tipo:** Transformación con Enriquecimiento de Datos

**Descripción:** El Experience API enriquece el payload con información del cliente (docNumber, clientId) y headers de contexto FUA antes de invocar el Process API para crear el fraccionamiento en SAP ISU.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente Web/App<br/>Portal FUA]
    E[Experience API<br/>ntgy-fua-e-fua<br/>POST /subdivision]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /subdivision]
    S[System API<br/>ntgy-newco-s-sap-isu<br/>RFC: ZISU_FRAC_CREAR]
    SAP[SAP ISU<br/>Tablas: DFKKOP, FKKVKP]
    
    C -->|Request| E
    E -->|Enriquece Payload| P
    P -->|Invoca RFC| S
    S -->|Crea| SAP
    SAP -->|numRef Generado| S
    S -->|Response| P
    P -->|Response| E
    E -->|Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style SAP fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Solicitud de Fraccionamiento por Dificultades Financieras**
   - El cliente tiene dificultades temporales para pagar facturas
   - Solicita dividir el pago en cuotas mensuales
   - Evita cortes de suministro y mejora la relación con el cliente

2. **Fraccionamiento de Facturas Elevadas**
   - El cliente recibe una factura con monto superior al habitual
   - Solicita fraccionar para distribuir el pago en el tiempo
   - Facilita la gestión del presupuesto familiar/empresarial

3. **Consolidación de Múltiples Facturas**
   - El cliente tiene varias facturas pendientes
   - Solicita un fraccionamiento único para todas ellas
   - Simplifica la gestión de pagos y reduce la carga administrativa

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web / Mobile App

**Características del Canal:**
- **Plataforma**: Portal FUA (Frontal Único de Atención)
- **Dispositivos**: Navegadores web, aplicaciones móviles iOS/Android
- **Conectividad**: Requiere conexión a internet estable
- **Experiencia de Usuario**: Interfaz de autoservicio para solicitud de fraccionamientos

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Necesitan fraccionar pagos por dificultades temporales
- **Clientes Empresariales**: Gestionan flujo de caja y necesitan flexibilidad de pago
- **Agentes de Atención al Cliente**: Procesan solicitudes en nombre del cliente

**Contexto de Uso:**
- **Autoservicio**: Cliente solicita desde el portal web/móvil
- **Gestión Asistida**: Agente solicita en nombre del cliente
- **Prevención de Morosidad**: Solicitud proactiva antes de impago

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Enriquecimiento de Payload**: Se añaden docNumber y clientId del contexto FUA
- **Enriquecimiento de Headers**: Se añade x-real-ip para auditoría
- **Formato de Respuesta**: JSON optimizado con numRef generado
- **Timeout**: 60 segundos para garantizar respuesta en operaciones complejas
- **Logging**: Registro en Kinesis para auditoría completa

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/subdivision` | POST | Creación de fraccionamiento en SAP ISU |

**Flujo de Dependencias:**
```
Experience API → Process API → System API → SAP ISU (RFC: ZISU_FRAC_CREAR)
```

### Contexto Empresarial

- **Dominio:** Gestión Financiera de Clientes
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Web / Mobile App

### Stakeholders

- **Clientes de Energía**: Necesitan flexibilidad en el pago de facturas
- **Departamento de Cobranzas**: Mejora la recuperación de deuda
- **Servicio de Atención al Cliente**: Procesa solicitudes de fraccionamiento
- **Departamento Financiero**: Monitorea flujo de caja y cobranza

### Valor de Negocio

- **Reducción de Morosidad**: Facilita el pago y reduce impagos
- **Mejora de Satisfacción**: El cliente tiene control sobre sus pagos
- **Optimización de Cobranza**: Aumenta la tasa de recuperación de deuda
- **Prevención de Cortes**: Evita suspensión de suministro por impago
- **Fidelización**: Mejora la relación cliente-empresa

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/subdivision` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/subdivision` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision` |

### Método HTTP

```
POST /fua/api/subdivision
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Creación de fraccionamiento en SAP ISU puede ser compleja |
| **Rate Limiting** | Según política API Gateway | Protección del backend SAP |
| **Autenticación** | OAuth 2.0 requerido | Seguridad de operaciones financieras |
| **Auditoría** | Registro obligatorio en Kinesis | Trazabilidad completa de solicitudes |
| **Tamaño Payload** | Máximo 10 facturas por solicitud | Límite de procesamiento SAP |

#### Headers Específicos del Canal

**Headers Añadidos por el Experience API:**

| Header | Tipo | Descripción | Razón |
|--------|------|-------------|-------|
| `x-real-ip` | string | IP real del cliente | Auditoría y seguridad de la operación |

#### Campos Añadidos al Payload

**Campos Enriquecidos por el Experience API:**

| Campo | Tipo | Descripción | Origen |
|-------|------|-------------|--------|
| `docNumber` | string | Documento del cliente | Header `x-client-code` |
| `clientId` | string | ID del cliente en el sistema | Header `x-client-id` |

### Parámetros de Entrada

#### Request Body (JSON)

| Campo | Tipo | Obligatorio | Descripción | Ejemplo |
|-------|------|-------------|-------------|---------|
| `contractAccountId` | string | ✅ Sí | ID de la cuenta contrato | `"12345"` |
| `createReason` | string | ✅ Sí | ID del motivo de alta del fraccionamiento | `"01"` |
| `invoices` | array | ✅ Sí | Listado de facturas a fraccionar | `["INV001", "INV002"]` |
| `startDate` | date | ✅ Sí | Fecha de inicio del fraccionamiento | `"2024-03-01"` |
| `numDues` | number | ✅ Sí | Número de cuotas | `6` |
| `contactId` | string | ✅ Sí | ID de contacto preferente | `"123"` |

**Validaciones:**
- Todos los campos son obligatorios
- `contractAccountId` debe corresponder a una cuenta contrato válida
- `createReason` debe ser un código válido en SAP ISU
- `invoices` debe contener al menos 1 factura y máximo 10
- `startDate` debe ser fecha futura o actual
- `numDues` debe estar entre 2 y 12 cuotas
- `contactId` debe corresponder a un contacto válido del cliente

**Códigos de Motivo de Creación (Ejemplos):**
- `"01"`: Dificultades temporales de pago
- `"02"`: Factura elevada
- `"03"`: Consolidación de deuda
- `"04"`: Acuerdo de pago
- `"05"`: Prevención de corte

**Formato de Fecha:**
- Formato: `YYYY-MM-DD` (ISO 8601)
- Ejemplo: `"2024-03-01"`

### Respuestas

#### 200 OK - Fraccionamiento Creado

**Descripción:** Fraccionamiento creado exitosamente en SAP ISU

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Fraccionamiento creado exitosamente"
  },
  "numRef": "FRA-2024-001"
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | string | Código de resultado ("0000" = éxito) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |
| `numRef` | string | Número de referencia del fraccionamiento creado |

#### 400 Bad Request - Parámetros Inválidos

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Campos obligatorios faltantes: contractAccountId, createReason, invoices, startDate, numDues, contactId"
  }
}
```

#### 400 Bad Request - Validación de Negocio

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "El número de cuotas debe estar entre 2 y 12"
  }
}
```

#### 404 Not Found - Cuenta No Encontrada

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "No se encontró la cuenta contrato especificada"
  }
}
```

#### 409 Conflict - Fraccionamiento Ya Existe

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Ya existe un fraccionamiento activo para esta cuenta"
  }
}
```

#### 500 Internal Server Error - Error del Sistema

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al crear fraccionamiento en SAP ISU"
  }
}
```

### Autenticación

- **Tipo:** OAuth 2.0
- **Headers Requeridos:**
  - `Authorization: Bearer {token}`
  - `client_id: {client_id}`
  - `x-client-id: {client_id}` (para enriquecimiento)
  - `x-client-code: {document}` (para enriquecimiento)

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Creación de fraccionamiento en SAP ISU puede ser compleja |
| **Connection Timeout** | 30 segundos | Establecimiento de conexión con Process API |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request<br/>Portal FUA]) --> LogKinesis[Log Request Kinesis]
    LogKinesis --> Transform[Transform Payload<br/>Enriquece con docNumber, clientId]
    Transform --> EnrichHeaders[Enrich Header<br/>x-real-ip]
    EnrichHeaders --> CallProcess[HTTP Request<br/>Process API POST /subdivision]
    CallProcess --> Success{Success?}
    Success -->|Yes| LogSuccess[Log Success]
    Success -->|No| ErrorHandler[Error Handler]
    LogSuccess --> End([HTTP Response<br/>numRef Generado])
    ErrorHandler --> LogErrorKinesis[Log Error Kinesis]
    LogErrorKinesis --> TransformError[Transform Error Response]
    TransformError --> EndError([HTTP Error Response])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#c8e6c9,stroke:#2e7d32
    style EndError fill:#ffcdd2,stroke:#c62828
    style CallProcess fill:#fff9c4,stroke:#f57f17
    style Transform fill:#e1bee7,stroke:#8e24aa
```

### Código del Flow (Experience API)

**Ubicación:** `MUL-COM-FUA-E-FUA/src/main/mule/implementation/subdivisions.xml`

**Flow Principal:** `post-subdivisionsFlow`

```xml
<flow name="post-subdivisionsFlow" doc:id="83f17e87-a416-4a22-a8c1-b4ccb1e1e800">
    <!-- Logging Kinesis -->
    <logger level="INFO" doc:name="LoggerKinesis" category="kinesisLogger" 
        message="#[%dw 2.0
output application/json
---
{
    headers:
    {
        method: attributes.method,
        requestUri: attributes.requestUri,
        correlationId: correlationId
    },
    message:
    {
        payload: payload
    }
}]" />
    
    <!-- Transformación: Enriquecimiento del Payload -->
    <ee:transform doc:name="Transform Message">
        <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
payload
++ 
vars.fuaHeaders
++ 
{
    "docNumber": attributes.headers['x-client-code'],
    "clientId": attributes.headers['x-client-id']
}]]></ee:set-payload>
    </ee:transform>
    
    <!-- Llamada al Process API -->
    <http:request 
        method="POST" 
        doc:name="request post/subdivisions Newco-evo" 
        config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
        path="/subdivision">
        <http:headers><![CDATA[#[import * from dw::core::Strings
output application/java
---
{
    "x-real-ip" : attributes.headers['x-real-ip']
}]]]></http:headers>
    </http:request>
    
    <!-- Manejo de Errores -->
    <error-handler>
        <on-error-propagate enableNotifications="true" logException="true">
            <logger level="ERROR" doc:name="Error KO" 
                message='#[{
    "result": {
        "codResult": "0001",
        "msgResult": error.description as String replace /\"/ with("")
    }
}]'/>
            <ee:transform doc:name="Transform Message">
                <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0001",
        "msgResult": error.description as String replace /\"/ with("")
    }
}]]></ee:set-payload>
            </ee:transform>
        </on-error-propagate>
    </error-handler>
</flow>
```

### Transformaciones de Canal

**Este Experience API SÍ transforma el payload antes de enviarlo al Process API.**

**Transformación Aplicada:**

| Campo Original | Campo Transformado | Transformación |
|----------------|-------------------|----------------|
| `payload` (body original) | `payload` | Se mantiene |
| `vars.fuaHeaders` | Campos adicionales | Se añaden headers FUA al payload |
| `attributes.headers['x-client-code']` | `docNumber` | Se extrae del header |
| `attributes.headers['x-client-id']` | `clientId` | Se extrae del header |

**Lógica de Transformación DataWeave:**

```dataweave
%dw 2.0
output application/json
---
payload                                          // Body original del cliente
++ 
vars.fuaHeaders                                  // Headers FUA como campos
++ 
{
    "docNumber": attributes.headers['x-client-code'],  // Documento del cliente
    "clientId": attributes.headers['x-client-id']      // ID del cliente
}
```

**Payload Resultante Enviado al Process API:**

```json
{
  "contractAccountId": "12345",
  "createReason": "01",
  "invoices": ["INV001", "INV002"],
  "startDate": "2024-03-01",
  "numDues": 6,
  "contactId": "123",
  "docNumber": "12345678A",
  "clientId": "cliente123"
}
```

### Configuración HTTP

**Connector:** `HTTP_FUA_PROCESS_NEWCO_EVO`

**Configuración (global.xml):**
```xml
<http:request-config 
    name="HTTP_FUA_PROCESS_NEWCO_EVO" 
    basePath="/api" 
    responseTimeout="${newco-p-fua-evo.timeout}">
    <http:request-connection 
        host="${newco-p-fua-evo.host}" 
        port="${newco-p-fua-evo.port}" 
        connectionIdleTimeout="${newco-p-fua-evo.timeout}"/>
</http:request-config>
```

**Properties (e-fua-pro-config.yaml):**
```yaml
newco-p-fua-evo:
  host: "mule-worker-internal-ntgy-newco-p-fua-evo.de-c1.cloudhub.io"
  port: "8091"
  timeout: "60000"
```

### Patrones de Diseño

**Patrón:** Transformación con Enriquecimiento de Contexto

**Justificación:** 
- El Experience API enriquece el payload con información del cliente
- Añade contexto de auditoría (x-real-ip)
- Registra la solicitud en Kinesis para trazabilidad completa
- Mantiene la separación de responsabilidades entre capas
- Facilita el mantenimiento y evolución independiente

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente Web/App
    participant E as Experience API
    participant K as AWS Kinesis
    participant P as Process API
    participant S as System API
    participant SAP as SAP ISU
    
    C->>E: POST /subdivision<br/>Body con datos fraccionamiento
    E->>K: Log Request (auditoría)
    Note over E: Enriquece payload con<br/>docNumber, clientId
    E->>P: POST /subdivision<br/>+ Payload enriquecido
    P->>S: Invoca RFC ZISU_FRAC_CREAR
    S->>SAP: Crea fraccionamiento<br/>DFKKOP, FKKVKP
    SAP-->>S: numRef generado
    S-->>P: Response con numRef
    P-->>E: Response sin transformar
    E-->>C: 200 OK<br/>numRef del fraccionamiento
```

#### 2. Flujo con Error de Validación

```mermaid
sequenceDiagram
    participant C as Cliente Web/App
    participant E as Experience API
    participant P as Process API
    participant S as System API
    participant SAP as SAP ISU
    
    C->>E: POST /subdivision<br/>numDues = 15 (inválido)
    E->>P: POST /subdivision
    P->>S: Invoca RFC
    S->>SAP: Valida parámetros
    SAP-->>S: Error: numDues fuera de rango
    S-->>P: Error de validación
    P-->>E: 400 Bad Request
    E-->>C: 400 Bad Request<br/>Validación fallida
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant C as Cliente Web/App
    participant E as Experience API
    participant K as AWS Kinesis
    participant P as Process API
    participant S as System API
    participant SAP as SAP ISU
    
    C->>E: POST /subdivision
    E->>K: Log Request
    E->>P: POST /subdivision
    P->>S: Invoca RFC
    S->>SAP: Crea (timeout)
    Note over SAP: SAP no responde<br/>en tiempo esperado
    S-->>P: Timeout Error
    P-->>E: 500 Internal Server Error
    Note over E: Error Handler<br/>Transforma error
    E->>K: Log Error
    E-->>C: 500 Internal Server Error<br/>codResult: "0001"
```

### Manejo de Errores

**Estrategia:** Global Error Handler con Transformación Estándar y Logging

**Tipos de Errores Manejados:**

1. **Errores de Conectividad (HTTP:TIMEOUT, HTTP:CONNECTIVITY)**
   - Se capturan en el error handler
   - Se transforman a formato estándar con codResult "0001"
   - Se registran en logs con nivel ERROR
   - Se registran en Kinesis para auditoría

2. **Errores de Validación (400)**
   - Se propagan desde el Process API
   - Mantienen el código de estado HTTP original
   - Se registran para auditoría

3. **Errores de Negocio (404, 409)**
   - Se propagan desde el Process API
   - Mantienen el código de estado HTTP original
   - Se registran para análisis

4. **Errores del Sistema (500)**
   - Se transforman a formato estándar
   - Se ocultan detalles técnicos al cliente
   - Se registran con stack trace completo en Kinesis

**Formato de Error Estándar:**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Descripción del error"
  }
}
```

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Creación Exitosa de Fraccionamiento

#### cURL

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "client_id: your-client-id" \
  -H "x-client-id: cliente123" \
  -H "x-client-code: 12345678A" \
  -H "x-real-ip: 192.168.1.100" \
  -d '{
    "contractAccountId": "12345",
    "createReason": "01",
    "invoices": ["INV-2024-001", "INV-2024-002"],
    "startDate": "2024-03-01",
    "numDues": 6,
    "contactId": "123"
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Fraccionamiento creado exitosamente"
  },
  "numRef": "FRA-2024-001"
}
```

### Ejemplo 2: JavaScript (React/Vue para Web)

```javascript
async function createSubdivision(subdivisionData) {
  try {
    const response = await fetch(
      'https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision',
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${getAuthToken()}`,
          'client_id': 'your-client-id',
          'x-client-id': getUserClientId(),
          'x-client-code': getUserDocument(),
          'x-real-ip': getUserIP()
        },
        body: JSON.stringify(subdivisionData)
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    
    if (data.result.codResult === "0000") {
      console.log('✓ Fraccionamiento creado:', data.numRef);
      return data.numRef;
    } else {
      console.error('✗ Error al crear:', data.result.msgResult);
      return null;
    }
  } catch (error) {
    console.error('✗ Error al crear fraccionamiento:', error);
    throw error;
  }
}

// Uso
const subdivisionData = {
  contractAccountId: '12345',
  createReason: '01',
  invoices: ['INV-2024-001', 'INV-2024-002'],
  startDate: '2024-03-01',
  numDues: 6,
  contactId: '123'
};

const numRef = await createSubdivision(subdivisionData);
if (numRef) {
  alert(`Fraccionamiento creado con referencia: ${numRef}`);
}
```

### Ejemplo 3: Python (para Integraciones Backend)

```python
import requests
from typing import Optional, List
from datetime import date

def create_subdivision(
    contract_account_id: str,
    create_reason: str,
    invoices: List[str],
    start_date: date,
    num_dues: int,
    contact_id: str,
    auth_token: str,
    client_id: str,
    user_client_id: str,
    user_document: str,
    user_ip: str
) -> Optional[str]:
    """
    Crea un nuevo fraccionamiento de pago.
    
    Args:
        contract_account_id: ID de la cuenta contrato
        create_reason: Código del motivo de creación
        invoices: Lista de IDs de facturas a fraccionar
        start_date: Fecha de inicio del fraccionamiento
        num_dues: Número de cuotas (2-12)
        contact_id: ID del contacto preferente
        auth_token: Token de autenticación OAuth 2.0
        client_id: Client ID de la aplicación
        user_client_id: ID del cliente en el sistema
        user_document: Documento del cliente
        user_ip: IP del cliente
        
    Returns:
        Número de referencia del fraccionamiento creado o None si hay error
    """
    url = "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision"
    
    headers = {
        'Content-Type': 'application/json',
        'Authorization': f'Bearer {auth_token}',
        'client_id': client_id,
        'x-client-id': user_client_id,
        'x-client-code': user_document,
        'x-real-ip': user_ip
    }
    
    payload = {
        'contractAccountId': contract_account_id,
        'createReason': create_reason,
        'invoices': invoices,
        'startDate': start_date.isoformat(),
        'numDues': num_dues,
        'contactId': contact_id
    }
    
    try:
        response = requests.post(url, headers=headers, json=payload, timeout=60)
        response.raise_for_status()
        
        data = response.json()
        
        if data['result']['codResult'] == '0000':
            num_ref = data['numRef']
            print(f"✓ Fraccionamiento creado exitosamente: {num_ref}")
            return num_ref
        else:
            print(f"✗ Error al crear: {data['result']['msgResult']}")
            return None
            
    except requests.exceptions.Timeout:
        print("✗ Error: Timeout al crear fraccionamiento")
        return None
    except requests.exceptions.RequestException as e:
        print(f"✗ Error de conexión: {str(e)}")
        return None
    except Exception as e:
        print(f"✗ Error inesperado: {str(e)}")
        return None

# Uso
from datetime import date, timedelta

num_ref = create_subdivision(
    contract_account_id='12345',
    create_reason='01',  # Dificultades temporales
    invoices=['INV-2024-001', 'INV-2024-002'],
    start_date=date.today() + timedelta(days=7),  # Inicia en 7 días
    num_dues=6,
    contact_id='123',
    auth_token='eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
    client_id='your-client-id',
    user_client_id='cliente123',
    user_document='12345678A',
    user_ip='192.168.1.100'
)

if num_ref:
    print(f"Fraccionamiento creado con referencia: {num_ref}")
else:
    print("No se pudo crear el fraccionamiento")
```

### Ejemplo 4: Error - Validación de Cuotas

#### Request

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token" \
  -H "client_id: your-client-id" \
  -d '{
    "contractAccountId": "12345",
    "createReason": "01",
    "invoices": ["INV001"],
    "startDate": "2024-03-01",
    "numDues": 15,
    "contactId": "123"
  }'
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "El número de cuotas debe estar entre 2 y 12"
  }
}
```

### Ejemplo 5: Error - Campos Faltantes

#### Request

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token" \
  -H "client_id: your-client-id" \
  -d '{
    "contractAccountId": "12345",
    "createReason": "01"
  }'
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Campos obligatorios faltantes: invoices, startDate, numDues, contactId"
  }
}
```

### Ejemplo 6: Fraccionamiento de Múltiples Facturas

#### Request

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token" \
  -H "client_id: your-client-id" \
  -H "x-client-id: cliente123" \
  -H "x-client-code: 12345678A" \
  -d '{
    "contractAccountId": "12345",
    "createReason": "03",
    "invoices": [
      "INV-2023-12-001",
      "INV-2024-01-001",
      "INV-2024-02-001"
    ],
    "startDate": "2024-03-15",
    "numDues": 12,
    "contactId": "123"
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Fraccionamiento creado exitosamente"
  },
  "numRef": "FRA-2024-002"
}
```
