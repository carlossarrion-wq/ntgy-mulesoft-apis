---
api: "ntgy-fua-e-fua"
sidebar_position: 3
sidebar_label: "DELETE /subdivision"
description: "Cancela un fraccionamiento activo del cliente"
keywords: [fraccionamiento, subdivision, cancelar, eliminar, SAP ISU]
tags: [experience-api, fua, subdivision, cancelacion]
channel: web-app
endpoint_dependencies:
  - method: "DELETE"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/subdivision"
    purpose: "Cancelación de fraccionamiento en Process API"
---

# DELETE /subdivision

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite cancelar un fraccionamiento activo del cliente. La cancelación implica la anulación del plan de pago fraccionado, requiriendo un motivo de cancelación y el identificador del fraccionamiento a cancelar.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Orquestación y adaptación de experiencia para canal web/móvil

### Patrón de Integración

**Tipo:** Pass-Through con Enriquecimiento de Headers

**Descripción:** El Experience API actúa como pass-through hacia el Process API, enriqueciendo la petición con headers de contexto FUA (x-real-ip) antes de invocar el servicio de cancelación de fraccionamientos.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente Web/App<br/>Portal FUA]
    E[Experience API<br/>ntgy-fua-e-fua<br/>DELETE /subdivision]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>DELETE /subdivision]
    S[System API<br/>ntgy-newco-s-sap-isu<br/>RFC: ZISU_FRAC_CANCELAR]
    SAP[SAP ISU<br/>Tablas: DFKKOP, FKKVKP]
    
    C -->|Request| E
    E -->|Enriquece Headers| P
    P -->|Invoca RFC| S
    S -->|Cancela| SAP
    SAP -->|Confirmación| S
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

1. **Cancelación por Solicitud del Cliente**
   - El cliente decide cancelar un fraccionamiento activo
   - Puede ser por cambio en su situación financiera
   - Permite al cliente gestionar sus compromisos de pago

2. **Cancelación por Pago Anticipado**
   - El cliente ha pagado el total de la deuda fraccionada
   - Se cancela el fraccionamiento al no ser necesario
   - Mejora la situación crediticia del cliente

3. **Cancelación por Incumplimiento**
   - El cliente no ha cumplido con las cuotas acordadas
   - El departamento de cobranzas cancela el fraccionamiento
   - Se activan procesos de cobranza alternativos

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web / Mobile App

**Características del Canal:**
- **Plataforma**: Portal FUA (Frontal Único de Atención)
- **Dispositivos**: Navegadores web, aplicaciones móviles iOS/Android
- **Conectividad**: Requiere conexión a internet estable
- **Experiencia de Usuario**: Interfaz de autoservicio para gestión de pagos

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes de Energía**: Necesitan cancelar fraccionamientos activos
- **Agentes de Cobranzas**: Gestionan cancelaciones por incumplimiento
- **Servicio de Atención al Cliente**: Procesan solicitudes de cancelación

**Contexto de Uso:**
- **Autoservicio**: Cliente cancela desde el portal web/móvil
- **Gestión Asistida**: Agente cancela en nombre del cliente
- **Proceso de Cobranza**: Cancelación automática por incumplimiento

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Enriquecimiento de Headers**: Se añade header x-real-ip para auditoría
- **Formato de Respuesta**: JSON optimizado para consumo web/móvil
- **Timeout**: 60 segundos para garantizar respuesta en operaciones complejas

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/subdivision` | DELETE | Cancelación de fraccionamiento en SAP ISU |

**Flujo de Dependencias:**
```
Experience API → Process API → System API → SAP ISU (RFC: ZISU_FRAC_CANCELAR)
```

### Contexto Empresarial

- **Dominio:** Gestión Financiera de Clientes
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Web / Mobile App

### Stakeholders

- **Clientes de Energía**: Necesitan cancelar fraccionamientos cuando cambia su situación
- **Departamento de Cobranzas**: Gestiona cancelaciones por incumplimiento
- **Servicio de Atención al Cliente**: Procesa solicitudes de cancelación
- **Departamento Legal**: Supervisa cancelaciones por motivos regulatorios

### Valor de Negocio

- **Flexibilidad Financiera**: El cliente puede ajustar sus compromisos de pago
- **Reducción de Morosidad**: Cancelación temprana evita acumulación de deuda
- **Mejora de Cobranza**: Permite activar estrategias alternativas de cobro
- **Transparencia**: El cliente controla sus obligaciones financieras

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/subdivision?numRef=12345&cancellationReason=01&contactId=123` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/subdivision?numRef=12345&cancellationReason=01&contactId=123` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision?numRef=12345&cancellationReason=01&contactId=123` |

### Método HTTP

```
DELETE /fua/api/subdivision
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Operación de cancelación en SAP ISU puede ser compleja |
| **Rate Limiting** | Según política API Gateway | Protección del backend SAP |
| **Autenticación** | OAuth 2.0 requerido | Seguridad de operaciones financieras |
| **Auditoría** | Registro obligatorio | Trazabilidad de cancelaciones |

#### Headers Específicos del Canal

**Headers Añadidos por el Experience API:**

| Header | Tipo | Descripción | Razón |
|--------|------|-------------|-------|
| `x-real-ip` | string | IP real del cliente | Auditoría y seguridad de la operación |

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Obligatorio | Descripción | Ejemplo |
|-----------|------|-------------|-------------|---------|
| `cancellationReason` | string | ✅ Sí | ID del motivo de cancelación | `"01"` |
| `numRef` | string | ✅ Sí | ID del fraccionamiento a cancelar | `"12345"` |
| `contactId` | string | ✅ Sí | ID de contacto preferente | `"123"` |

**Validaciones:**
- Todos los parámetros son obligatorios
- `numRef` debe corresponder a un fraccionamiento activo
- `cancellationReason` debe ser un código válido en SAP ISU
- `contactId` debe corresponder a un contacto válido del cliente

**Códigos de Motivo de Cancelación (Ejemplos):**
- `"01"`: Solicitud del cliente
- `"02"`: Pago anticipado
- `"03"`: Incumplimiento
- `"04"`: Error en la solicitud
- `"05"`: Cambio de condiciones

### Respuestas

#### 200 OK - Cancelación Exitosa

**Descripción:** Fraccionamiento cancelado correctamente

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Fraccionamiento cancelado exitosamente"
  }
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | string | Código de resultado ("0000" = éxito) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |

#### 400 Bad Request - Parámetros Inválidos

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Parámetros obligatorios faltantes: numRef, cancellationReason, contactId"
  }
}
```

#### 404 Not Found - Fraccionamiento No Encontrado

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "No se encontró el fraccionamiento especificado"
  }
}
```

#### 409 Conflict - Fraccionamiento Ya Cancelado

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "El fraccionamiento ya está cancelado"
  }
}
```

#### 500 Internal Server Error - Error del Sistema

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al cancelar fraccionamiento en SAP ISU"
  }
}
```

### Autenticación

- **Tipo:** OAuth 2.0
- **Headers Requeridos:**
  - `Authorization: Bearer {token}`
  - `client_id: {client_id}`

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Operaciones de cancelación en SAP ISU pueden ser complejas |
| **Connection Timeout** | 30 segundos | Establecimiento de conexión con Process API |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP DELETE Request<br/>Portal FUA]) --> LogStart[Log Request]
    LogStart --> EnrichHeaders[Enrich Header<br/>x-real-ip]
    EnrichHeaders --> BuildQueryParams[Build Query Params<br/>+ fuaHeaders]
    BuildQueryParams --> CallProcess[HTTP Request<br/>Process API DELETE /subdivision]
    CallProcess --> Success{Success?}
    Success -->|Yes| LogSuccess[Log Success]
    Success -->|No| ErrorHandler[Error Handler]
    LogSuccess --> End([HTTP Response<br/>Cancelación Confirmada])
    ErrorHandler --> TransformError[Transform Error Response]
    TransformError --> LogError[Log Error]
    LogError --> EndError([HTTP Error Response])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#c8e6c9,stroke:#2e7d32
    style EndError fill:#ffcdd2,stroke:#c62828
    style CallProcess fill:#fff9c4,stroke:#f57f17
```

### Código del Flow (Experience API)

**Ubicación:** `MUL-COM-FUA-E-FUA/src/main/mule/implementation/subdivisions.xml`

**Flow Principal:** `delete-subdivisionsFlow`

```xml
<flow name="delete-subdivisionsFlow" doc:id="45714f08-765d-442c-aec3-b4d4ec0476ce">
    <!-- Llamada al Process API -->
    <http:request 
        method="DELETE" 
        doc:name="fua_newCo delete /subdivisions" 
        config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
        path="/subdivision">
        <http:headers><![CDATA[#[import * from dw::core::Strings
output application/java
---
{
    "x-real-ip" : attributes.headers['x-real-ip']
}]]]></http:headers>
        <http:query-params><![CDATA[#[output application/java
---
attributes.queryParams ++ vars.fuaHeaders]]]></http:query-params>
    </http:request>
    
    <!-- Manejo de Errores -->
    <error-handler>
        <on-error-propagate enableNotifications="true" logException="true">
            <logger level="ERROR" doc:name="Error" 
                message='#[%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0001",
        "msgResult": error.description as String replace /\"/ with("")
    }
}]' />
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

**⚠️ Este Experience API NO transforma la respuesta del Process API.**

Es un patrón **Pass-Through con Enriquecimiento de Headers**:
- Enriquece la petición con header x-real-ip para auditoría
- Combina query params originales con headers FUA
- Propaga la respuesta del Process API sin modificaciones

Para el schema completo de la respuesta, consultar:
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `/subdivision`**

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

**Patrón:** Pass-Through con Enriquecimiento de Contexto y Auditoría

**Justificación:** 
- El Experience API no necesita transformar la lógica de negocio
- Se enfoca en añadir contexto de auditoría (x-real-ip)
- Mantiene la separación de responsabilidades entre capas
- Facilita el mantenimiento y evolución independiente
- Garantiza trazabilidad de operaciones críticas

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente Web/App
    participant E as Experience API
    participant P as Process API
    participant S as System API
    participant SAP as SAP ISU
    
    C->>E: DELETE /subdivision<br/>numRef, cancellationReason, contactId
    Note over E: Enriquece con<br/>x-real-ip
    E->>P: DELETE /subdivision<br/>+ FUA Headers
    P->>S: Invoca RFC ZISU_FRAC_CANCELAR
    S->>SAP: Cancela fraccionamiento<br/>DFKKOP, FKKVKP
    SAP-->>S: Confirmación cancelación
    S-->>P: Response exitoso
    P-->>E: Response sin transformar
    E-->>C: 200 OK<br/>Cancelación confirmada
```

#### 2. Flujo con Error de Negocio (Fraccionamiento No Encontrado)

```mermaid
sequenceDiagram
    participant C as Cliente Web/App
    participant E as Experience API
    participant P as Process API
    participant S as System API
    participant SAP as SAP ISU
    
    C->>E: DELETE /subdivision<br/>numRef=99999
    E->>P: DELETE /subdivision + Headers
    P->>S: Invoca RFC
    S->>SAP: Busca fraccionamiento
    SAP-->>S: Fraccionamiento no existe
    S-->>P: Error: No encontrado
    P-->>E: 404 Not Found
    E-->>C: 404 Not Found<br/>Fraccionamiento no encontrado
```

#### 3. Flujo con Error Técnico (Timeout SAP)

```mermaid
sequenceDiagram
    participant C as Cliente Web/App
    participant E as Experience API
    participant P as Process API
    participant S as System API
    participant SAP as SAP ISU
    
    C->>E: DELETE /subdivision
    E->>P: DELETE /subdivision + Headers
    P->>S: Invoca RFC
    S->>SAP: Cancela (timeout)
    Note over SAP: SAP no responde<br/>en tiempo esperado
    S-->>P: Timeout Error
    P-->>E: 500 Internal Server Error
    Note over E: Error Handler<br/>Transforma error
    E-->>C: 500 Internal Server Error<br/>codResult: "0001"
```

### Manejo de Errores

**Estrategia:** Global Error Handler con Transformación Estándar

**Tipos de Errores Manejados:**

1. **Errores de Conectividad (HTTP:TIMEOUT, HTTP:CONNECTIVITY)**
   - Se capturan en el error handler
   - Se transforman a formato estándar con codResult "0001"
   - Se registran en logs con nivel ERROR

2. **Errores de Negocio (404, 400, 409)**
   - Se propagan desde el Process API
   - Mantienen el código de estado HTTP original
   - Se registran para auditoría

3. **Errores del Sistema (500)**
   - Se transforman a formato estándar
   - Se ocultan detalles técnicos al cliente
   - Se registran con stack trace completo

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

### Ejemplo 1: Cancelación Exitosa

#### cURL

```bash
curl -X DELETE "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision?numRef=FRA-2024-001&cancellationReason=01&contactId=123" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "client_id: your-client-id" \
  -H "x-real-ip: 192.168.1.100"
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Fraccionamiento cancelado exitosamente"
  }
}
```

### Ejemplo 2: JavaScript (React/Vue para Web)

```javascript
async function cancelSubdivision(numRef, cancellationReason, contactId) {
  try {
    const params = new URLSearchParams({
      numRef: numRef,
      cancellationReason: cancellationReason,
      contactId: contactId
    });

    const response = await fetch(
      `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision?${params}`,
      {
        method: 'DELETE',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${getAuthToken()}`,
          'client_id': 'your-client-id',
          'x-real-ip': getUserIP()
        }
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    
    if (data.result.codResult === "0000") {
      console.log('✓ Fraccionamiento cancelado exitosamente');
      return true;
    } else {
      console.error('✗ Error al cancelar:', data.result.msgResult);
      return false;
    }
  } catch (error) {
    console.error('✗ Error al cancelar fraccionamiento:', error);
    throw error;
  }
}

// Uso
const success = await cancelSubdivision('FRA-2024-001', '01', '123');
if (success) {
  alert('Fraccionamiento cancelado correctamente');
}
```

### Ejemplo 3: Python (para Integraciones Backend)

```python
import requests
from typing import Optional

def cancel_subdivision(
    num_ref: str,
    cancellation_reason: str,
    contact_id: str,
    auth_token: str,
    client_id: str,
    user_ip: str
) -> bool:
    """
    Cancela un fraccionamiento activo.
    
    Args:
        num_ref: ID del fraccionamiento a cancelar
        cancellation_reason: Código del motivo de cancelación
        contact_id: ID del contacto preferente
        auth_token: Token de autenticación OAuth 2.0
        client_id: Client ID de la aplicación
        user_ip: IP del cliente
        
    Returns:
        True si la cancelación fue exitosa, False en caso contrario
    """
    url = "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision"
    
    headers = {
        'Content-Type': 'application/json',
        'Authorization': f'Bearer {auth_token}',
        'client_id': client_id,
        'x-real-ip': user_ip
    }
    
    params = {
        'numRef': num_ref,
        'cancellationReason': cancellation_reason,
        'contactId': contact_id
    }
    
    try:
        response = requests.delete(url, headers=headers, params=params, timeout=60)
        response.raise_for_status()
        
        data = response.json()
        
        if data['result']['codResult'] == '0000':
            print(f"✓ Fraccionamiento {num_ref} cancelado exitosamente")
            return True
        else:
            print(f"✗ Error al cancelar: {data['result']['msgResult']}")
            return False
            
    except requests.exceptions.Timeout:
        print("✗ Error: Timeout al cancelar fraccionamiento")
        return False
    except requests.exceptions.RequestException as e:
        print(f"✗ Error de conexión: {str(e)}")
        return False
    except Exception as e:
        print(f"✗ Error inesperado: {str(e)}")
        return False

# Uso
success = cancel_subdivision(
    num_ref='FRA-2024-001',
    cancellation_reason='01',  # Solicitud del cliente
    contact_id='123',
    auth_token='eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
    client_id='your-client-id',
    user_ip='192.168.1.100'
)

if success:
    print("Fraccionamiento cancelado correctamente")
else:
    print("No se pudo cancelar el fraccionamiento")
```

### Ejemplo 4: Error - Fraccionamiento No Encontrado

#### Request

```bash
curl -X DELETE "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision?numRef=99999&cancellationReason=01&contactId=123" \
  -H "Authorization: Bearer token" \
  -H "client_id: your-client-id"
```

#### Respuesta (404 Not Found)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "No se encontró el fraccionamiento especificado"
  }
}
```

### Ejemplo 5: Error - Parámetros Faltantes

#### Request

```bash
curl -X DELETE "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/subdivision?numRef=12345" \
  -H "Authorization: Bearer token" \
  -H "client_id: your-client-id"
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Parámetros obligatorios faltantes: cancellationReason, contactId"
  }
}
```
