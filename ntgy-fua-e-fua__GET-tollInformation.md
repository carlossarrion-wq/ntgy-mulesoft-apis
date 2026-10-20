---
api: "ntgy-fua-e-fua"
sidebar_position: 39
sidebar_label: "GET /tollInformation"
description: "Recupera datos de peaje de lecturas ATR (Acceso de Terceros a Red) para agentes"
keywords: [peaje, ATR, lecturas, agentes, consumo, distribuidora, CUPS]
tags: [Peajes, ATR, Lecturas, Experience API, Agentes]
channel: voice
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/tollInformation"
    purpose: "Orquestación de consulta de información de peaje desde SAP ISU"
---

# GET /tollInformation

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite a los agentes de atención al cliente recuperar información detallada de peajes de lecturas ATR (Acceso de Terceros a Red) para contratos de energía. Proporciona datos de consumo, lecturas y facturación de peajes asociados a un CUPS específico dentro de un rango de fechas.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para canal de agentes (voz/ATC)

### Patrón de Integración

**Tipo:** Pass-Through con propagación de headers

**Descripción:** Este Experience API actúa como pass-through, propagando los query parameters y headers de autenticación al Process API sin transformación de datos. La respuesta del Process API se devuelve directamente al cliente.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Agente<br/>AgentVoice/ATC]
    E[Experience API<br/>ntgy-fua-e-fua<br/>GET /tollInformation]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>GET /tollInformation]
    S[System API<br/>ntgy-newco-s-sap-isu<br/>GET /tollInformation]
    SAP[SAP ISU<br/>Backend]
    
    C -->|Request| E
    E -->|Consume| P
    P -->|Consume| S
    S -->|RFC Call| SAP
    SAP -->|Response| S
    S -->|Response| P
    P -->|Response| E
    E -->|Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style SAP fill:#fce4ec,stroke:#880e4f,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Consulta de Peajes por Agente**
   - Agentes de call center necesitan consultar información de peajes ATR
   - Permite verificar lecturas y consumos facturados por la distribuidora
   - Facilita la resolución de consultas sobre facturación de peajes

2. **Análisis de Consumo ATR**
   - Revisión de lecturas ATR en un período específico
   - Validación de datos de consumo reportados por la distribuidora
   - Soporte en reclamaciones relacionadas con peajes

3. **Soporte a Clientes**
   - Proporciona información detallada para explicar cargos de peaje
   - Ayuda en la resolución de dudas sobre facturación ATR
   - Facilita la transparencia en los costos de acceso a red

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Voice (AgentVoice/ATC)

**Características del Canal:**
- **Plataforma**: Sistema de atención telefónica para agentes
- **Dispositivos**: Terminales de agente en call center
- **Conectividad**: Red interna corporativa de alta disponibilidad
- **Experiencia de Usuario**: Interfaz optimizada para consultas rápidas durante llamadas

#### Audiencia Objetivo

**Usuarios Finales:**
- **Agentes de Call Center**: Personal de atención al cliente que necesita consultar información de peajes durante llamadas
- **Agentes de Soporte Técnico**: Personal especializado en resolución de consultas técnicas sobre facturación

**Contexto de Uso:**
- **Llamadas de Clientes**: Durante atención telefónica para resolver dudas sobre peajes
- **Reclamaciones**: Validación de datos para gestión de reclamaciones relacionadas con ATR

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Pass-Through**: No se aplican transformaciones, se devuelve la respuesta completa del Process API
- **Headers de Contexto**: Propagación de headers de agente (x-agent-mail, x-user-role)
- **Timeout**: Configurado para respuestas rápidas en contexto de llamada telefónica

### 🔗 Endpoints Dependientes

Este Experience API consume los siguientes Process APIs:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/tollInformation` | GET | Orquestación de consulta de información de peaje desde SAP ISU |

**Flujo de Dependencias:**
```
Experience API (e-fua)
    ↓
Process API (p-fua-evo)
    ↓
System API (s-sap-isu)
    ↓
SAP ISU Backend
```

### Contexto Empresarial

- **Dominio:** Atención al Cliente - Consultas de Facturación
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Agentes (Voice/ATC)

### Stakeholders

- **Agentes de Call Center**: Necesitan acceso rápido a información de peajes para atender consultas
- **Supervisores de Atención**: Requieren que los agentes tengan información completa y precisa
- **Clientes Finales**: Beneficiarios indirectos de un servicio de atención informado

### Valor de Negocio

- **Mejora en Atención al Cliente**: Agentes con información completa para resolver consultas
- **Reducción de Tiempos**: Acceso rápido a datos de peajes sin necesidad de consultas manuales
- **Transparencia**: Información detallada sobre cargos de acceso a red
- **Resolución de Reclamaciones**: Datos precisos para gestionar reclamaciones relacionadas con ATR

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/tollInformation` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/tollInformation` |
| **UAT** | `https://ntgy-fua-e-fua-uat.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-uat.de-c1.cloudhub.io/fua/api/tollInformation` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/tollInformation` |

### Método HTTP

```
GET /fua/api/tollInformation
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 30 segundos | Optimizado para consultas durante llamadas telefónicas |
| **Rate Limiting** | Según política de API Manager | Control de uso por agente |
| **Autenticación** | JWT con rol de agente | Solo accesible para usuarios con rol `newcoAgent` |

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `cupsGas` | string | No* | CUPS de gas | `ES0021000000000001JN` |
| `cupsElec` | string | No* | CUPS de electricidad | `ES0031406000001234AB` |
| `startDate` | date-only / string | Sí | Fecha inicio de consulta (formato: YYYY-MM-DD) | `2024-01-01` |
| `endDate` | date-only / string | Sí | Fecha fin de consulta (formato: YYYY-MM-DD) | `2024-12-31` |
| `contractNumber` | string | Sí | Número de contrato | `1234567890` |

**Nota:** * Al menos uno de `cupsGas` o `cupsElec` debe ser proporcionado.

#### Headers Requeridos

| Header | Tipo | Requerido | Descripción | Ejemplo |
|--------|------|-----------|-------------|---------|
| `Authorization` | string | Sí | Token JWT de autenticación | `Bearer eyJhbGc...` |
| `x-interaction-id` | string | Sí | ID único de interacción | `550e8400-e29b-41d4-a716-446655440000` |
| `x-interaction-origin` | string | Sí | Origen de la interacción | `AgentVoice` |
| `x-user-id` | string | Sí | ID del usuario agente | `agent123` |
| `x-client-id` | string | Sí | ID del cliente | `client456` |
| `x-client-doc-number` | string | No | Número de documento del cliente | `12345678A` |
| `x-client-code` | string | No | Código del cliente | `CLI001` |
| `x-origin` | string | No | Sistema origen | `ATC` |
| `x-user-role` | string | Sí | Rol del usuario (debe ser `newcoAgent`) | `newcoAgent` |
| `x-agent-mail` | string | No | Email del agente | `agent@naturgy.com` |
| `x-language` | string | No | Idioma preferido | `es` |
| `x-contact-id` | string | No | ID del contacto | `contact789` |

### Respuestas

#### 200 OK - Respuesta Exitosa

**Estructura de Respuesta:**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "OK"
  },
  "tollInformationsList": [
    {
      "atrDate": "2024-01-15",
      "endDate": "2024-01-31",
      "startDate": "2024-01-01",
      "billingReason": "Lectura mensual",
      "invoiceType": "Normal",
      "cups": "ES0031406000001234AB",
      "atrRate": "2.0TD",
      "atrInvoiceId": "FAC123456",
      "coversionFactor": "1.0",
      "distribuitor": "UFD Distribución",
      "autoCons": "Sin autoconsumo",
      "device": "Contador digital",
      "tollInformationDetailList": [
        {
          "magnitude": "Activa",
          "period": "P1",
          "procedenceTo": "Real",
          "procedenceFrom": "Real",
          "readingValueTo": "1234.56",
          "readingValueFrom": "1111.11",
          "adjust": "0",
          "integer": "123",
          "decimal": "45",
          "registerDate": "2024-01-31",
          "consumption": "123.45",
          "energyRevert": "0"
        }
      ]
    }
  ]
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result` | object | Objeto con resultado de la operación |
| `result.codResult` | string | Código de resultado (`0000` = éxito, `0001` = error) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |
| `tollInformationsList` | array | Lista de información de peajes (opcional, solo para agentes) |
| `tollInformationsList[].atrDate` | date-only / string | Fecha de carga ATR |
| `tollInformationsList[].endDate` | date-only / string | Fecha hasta |
| `tollInformationsList[].startDate` | date-only / string | Fecha desde |
| `tollInformationsList[].billingReason` | string | Motivo de facturación |
| `tollInformationsList[].invoiceType` | string | Tipo de factura |
| `tollInformationsList[].cups` | string | CUPS del suministro |
| `tollInformationsList[].atrRate` | string | Tarifa ATR |
| `tollInformationsList[].atrInvoiceId` | string | ID de factura ATR |
| `tollInformationsList[].coversionFactor` | string | Factor de conversión |
| `tollInformationsList[].distribuitor` | string | Distribuidora |
| `tollInformationsList[].autoCons` | string | Modalidad de autoconsumo |
| `tollInformationsList[].device` | string | Tipo de aparato/contador |
| `tollInformationsList[].tollInformationDetailList` | array | Detalle de consumos y lecturas ATR |
| `tollInformationDetailList[].magnitude` | string | Magnitud de la lectura |
| `tollInformationDetailList[].period` | string | Período de la lectura |
| `tollInformationDetailList[].procedenceTo` | string | Procedencia lectura hasta |
| `tollInformationDetailList[].procedenceFrom` | string | Procedencia lectura desde |
| `tollInformationDetailList[].readingValueTo` | string | Valor de lectura hasta |
| `tollInformationDetailList[].readingValueFrom` | string | Valor de lectura desde |
| `tollInformationDetailList[].adjust` | string | Ajuste aplicado |
| `tollInformationDetailList[].integer` | string | Parte entera del consumo |
| `tollInformationDetailList[].decimal` | string | Parte decimal del consumo |
| `tollInformationDetailList[].registerDate` | string | Fecha de registro |
| `tollInformationDetailList[].consumption` | string | Consumo de lectura |
| `tollInformationDetailList[].energyRevert` | string | Energía excedente (kW) |

#### 400 Bad Request - Error de Validación

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Parámetros inválidos"
  }
}
```

#### 500 Internal Server Error - Error del Sistema

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error interno del servidor"
  }
}
```

### Autenticación

- **Tipo:** OAuth 2.0 con JWT
- **Rol Requerido:** `newcoAgent`
- **Validación:** El token JWT debe contener el rol de agente para acceder a este endpoint

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 30 segundos | Optimizado para consultas durante llamadas telefónicas |
| **Process API Timeout** | 30 segundos | Configurado en HTTP connector |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request<br/>Canal Agente]) --> Validate[JWT Validation]
    Validate --> SetHeaders[Set FUA Headers]
    SetHeaders --> Router[APIKit Router]
    Router --> FlowRef[Flow: get-toll-information]
    FlowRef --> HTTPReq[HTTP Request to Process API]
    HTTPReq --> Response([HTTP Response])
    
    HTTPReq --> ErrorHandler{Error?}
    ErrorHandler -->|Yes| SetError[Set Error Message]
    SetError --> LogError[Log Error]
    LogError --> ErrorResponse([Error Response])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style Response fill:#e1f5ff,stroke:#0277bd
    style ErrorResponse fill:#ffebee,stroke:#c62828
```

### Implementación del Flow

**Archivo:** `e-fua.xml` (línea 479-481)

```xml
<flow name="get:\tollInformation:e-fua-config" doc:id="4c941cd7-453d-4823-801b-611774185aca">
    <flow-ref doc:name="get-toll-information" doc:id="e4e7a200-53d3-4714-baa1-dd774d6e3461" name="get-toll-information" />
</flow>
```

**Archivo:** `get-toll-information.xml`

```xml
<flow name="get-toll-information" doc:id="6ed96aaa-b6f3-4424-af68-107d6f281692">
    <http:request method="GET" 
                  doc:name="Newco-evo GET tollInformation Request" 
                  doc:id="d66bf296-4e6a-42eb-8c39-b1321d1fd577" 
                  config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
                  path="/tollInformation">
        <http:query-params><![CDATA[#[output application/java
---
attributes.queryParams ++ vars.fuaHeaders]]]></http:query-params>
    </http:request>
    <error-handler>
        <on-error-propagate enableNotifications="true" logException="true" 
                           doc:name="On Error Propagate" 
                           doc:id="d8e20580-d2c2-4555-b3e2-cae634c0a1dd">
            <ee:transform doc:name="Set Error Message" doc:id="60a9d580-6976-429c-b9e3-2c0964899219">
                <ee:message>
                    <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0001",
        "msgResult": error.description as String replace /\"/ with("")
    }
}]]></ee:set-payload>
                </ee:message>
            </ee:transform>
            <logger level="ERROR" doc:name="Error" doc:id="66005f3c-0720-4722-abc1-76f0a71290ca" 
                    message='#[{
    "result": {
        "codResult": "0001",
        "msgResult": error.description as String replace /\"/ with("")
    }
}]' />
        </on-error-propagate>
    </error-handler>
</flow>
```

### Transformaciones de Canal

⚠️ **Este Experience API NO transforma la respuesta del Process API.**

**Patrón:** Pass-Through con propagación de headers

**Implementación:**
- Los query parameters del request se propagan directamente al Process API
- Los headers de contexto FUA (establecidos en el flow principal) se añaden a los query parameters
- La respuesta del Process API se devuelve sin modificaciones al cliente

Para el schema completo de la respuesta, consultar:
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `/tollInformation`**

### Configuración HTTP

**Connector:** `HTTP_FUA_PROCESS_NEWCO_EVO`

**Configuración (desde properties):**
```yaml
newco-p-fua-evo:
  host: "mule-worker-internal-ntgy-newco-p-fua-evo.de-c1.cloudhub.io"
  port: "8091"
  basePath: "/api"
  protocol: "HTTP"
  timeout: "30000"
```

### Patrones de Diseño

**Patrón:** API Gateway con Pass-Through

**Justificación:** 
- El Experience API actúa como punto de entrada unificado para agentes
- Proporciona capa de seguridad y validación de JWT
- Propaga contexto de agente mediante headers
- No requiere transformación de datos ya que la respuesta del Process API es adecuada para el canal

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant A as Agente (ATC)
    participant E as Experience API
    participant P as Process API
    participant S as System API
    participant SAP as SAP ISU
    
    A->>E: GET /tollInformation<br/>(JWT + Headers + Query Params)
    Note over E: Validate JWT<br/>Extract FUA Headers
    E->>P: GET /tollInformation<br/>(Query Params + FUA Headers)
    Note over P: Extract Variables<br/>(clientId, cups, dates, sector)
    P->>S: GET /tollInformation<br/>(Transformed Params)
    Note over S: Call SAP RFC
    S->>SAP: RFC Call
    SAP-->>S: RFC Response
    S-->>P: Toll Information Data
    Note over P: Log Response Count
    P-->>E: Toll Information Response
    E-->>A: 200 OK<br/>Toll Information Data
```

#### 2. Flujo con Error de Negocio

```mermaid
sequenceDiagram
    participant A as Agente (ATC)
    participant E as Experience API
    participant P as Process API
    
    A->>E: GET /tollInformation<br/>(Parámetros inválidos)
    Note over E: Validate JWT
    E->>P: GET /tollInformation
    P-->>E: 400 Bad Request
    Note over E: Error Handler<br/>Transform Error
    E-->>A: 400 Bad Request<br/>Error Message
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant A as Agente (ATC)
    participant E as Experience API
    participant P as Process API
    
    A->>E: GET /tollInformation
    Note over E: Validate JWT
    E->>P: GET /tollInformation
    Note over P: Timeout o Error
    P--xE: Connection Error
    Note over E: Error Handler<br/>Set Error Message<br/>Log Error
    E-->>A: 500 Internal Server Error<br/>{"codResult": "0001"}
```

### Manejo de Errores

**Error Handler Global:**
- Captura cualquier error durante la llamada al Process API
- Transforma el error en formato estándar con `codResult: "0001"`
- Limpia caracteres especiales del mensaje de error
- Registra el error en logs para troubleshooting

**Códigos de Error:**

| Código | Descripción | Acción |
|--------|-------------|--------|
| `0000` | Operación exitosa | Continuar |
| `0001` | Error en la operación | Revisar mensaje de error |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta de Peajes por CUPS Eléctrico

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/tollInformation?cupsElec=ES0031406000001234AB&startDate=2024-01-01&endDate=2024-12-31&contractNumber=1234567890" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-interaction-id: 550e8400-e29b-41d4-a716-446655440000" \
  -H "x-interaction-origin: AgentVoice" \
  -H "x-user-id: agent123" \
  -H "x-client-id: client456" \
  -H "x-user-role: newcoAgent" \
  -H "x-agent-mail: agent@naturgy.com"
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "OK"
  },
  "tollInformationsList": [
    {
      "atrDate": "2024-01-15",
      "endDate": "2024-01-31",
      "startDate": "2024-01-01",
      "billingReason": "Lectura mensual",
      "invoiceType": "Normal",
      "cups": "ES0031406000001234AB",
      "atrRate": "2.0TD",
      "atrInvoiceId": "FAC123456",
      "coversionFactor": "1.0",
      "distribuitor": "UFD Distribución",
      "autoCons": "Sin autoconsumo",
      "device": "Contador digital",
      "tollInformationDetailList": [
        {
          "magnitude": "Activa",
          "period": "P1",
          "procedenceTo": "Real",
          "procedenceFrom": "Real",
          "readingValueTo": "1234.56",
          "readingValueFrom": "1111.11",
          "consumption": "123.45"
        }
      ]
    }
  ]
}
```

### Ejemplo 2: Consulta de Peajes por CUPS Gas

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/tollInformation?cupsGas=ES0021000000000001JN&startDate=2024-06-01&endDate=2024-06-30&contractNumber=9876543210" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-interaction-id: 660e8400-e29b-41d4-a716-446655440001" \
  -H "x-interaction-origin: ATC" \
  -H "x-user-id: agent456" \
  -H "x-client-id: client789" \
  -H "x-user-role: newcoAgent"
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "OK"
  },
  "tollInformationsList": [
    {
      "atrDate": "2024-06-15",
      "endDate": "2024-06-30",
      "startDate": "2024-06-01",
      "billingReason": "Lectura bimestral",
      "invoiceType": "Normal",
      "cups": "ES0021000000000001JN",
      "atrRate": "RL.1",
      "atrInvoiceId": "FAC789012",
      "coversionFactor": "11.2",
      "distribuitor": "Nedgia",
      "autoCons": "No aplica",
      "device": "Contador gas",
      "tollInformationDetailList": [
        {
          "magnitude": "Volumen",
          "period": "Único",
          "procedenceTo": "Real",
          "procedenceFrom": "Real",
          "readingValueTo": "5678.90",
          "readingValueFrom": "5111.01",
          "consumption": "567.89"
        }
      ]
    }
  ]
}
```

### Ejemplo 3: Error - Parámetros Faltantes

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/tollInformation?startDate=2024-01-01" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-interaction-id: 770e8400-e29b-41d4-a716-446655440002" \
  -H "x-interaction-origin: AgentVoice" \
  -H "x-user-id: agent789" \
  -H "x-client-id: client012" \
  -H "x-user-role: newcoAgent"
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Parámetros requeridos faltantes: endDate, contractNumber, cups"
  }
}
```

### Ejemplo 4: JavaScript (Fetch API)

```javascript
const getTollInformation = async () => {
  const params = new URLSearchParams({
    cupsElec: 'ES0031406000001234AB',
    startDate: '2024-01-01',
    endDate: '2024-12-31',
    contractNumber: '1234567890'
  });

  const response = await fetch(
    `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/tollInformation?${params}`,
    {
      method: 'GET',
      headers: {
        'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
        'x-interaction-id': '550e8400-e29b-41d4-a716-446655440000',
        'x-interaction-origin': 'AgentVoice',
        'x-user-id': 'agent123',
        'x-client-id': 'client456',
        'x-user-role': 'newcoAgent',
        'x-agent-mail': 'agent@naturgy.com'
      }
    }
  );

  const data = await response.json();
  console.log('Toll Information:', data);
  return data;
};

// Uso
getTollInformation()
  .then(data => {
    if (data.result.codResult === '0000') {
      console.log('Lecturas ATR:', data.tollInformationsList);
    } else {
      console.error('Error:', data.result.msgResult);
    }
  })
  .catch(error => console.error('Error de red:', error));