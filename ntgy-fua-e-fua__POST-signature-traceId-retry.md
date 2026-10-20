---
api: "ntgy-fua-e-fua"
sidebar_position: 85
sidebar_label: "POST /signature/:traceId/retry"
description: "Reenvía el SMS de firma digital al cliente - Orquesta el reenvío mediante Process API"
keywords:
  - reenviar sms
  - firma digital
  - traceid
  - retry
  - experience api
  - canal agente
  - orquestacion
tags:
  - Experience API
  - Firma Digital
  - Reenvío SMS
  - Canal Agente
channel: web
endpoint_dependencies:
  - method: "POST"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/firmas/:traceId/relanzar"
    purpose: "Reenvía SMS de firma, actualiza caso en Salesforce con comentario del reenvío"
---

# POST /signature/:traceId/retry

## 📌 Descripción General

### Propósito del Endpoint

Endpoint de Experience API que permite reenviar el SMS con el enlace de firma digital al cliente. Actúa como orquestador que delega la lógica de reenvío al Process API correspondiente, adaptando la experiencia para el canal web/agente.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Orquestación de reenvío de SMS de firma para canal agente

### Patrón de Integración

**Tipo:** Pass-Through con Propagación de Headers

**Descripción:** Este Experience API actúa como pass-through, delegando toda la lógica de negocio al Process API. Su función principal es:
- Extraer parámetros de la URI
- Propagar headers de contexto FUA
- Invocar el Process API correspondiente
- Retornar la respuesta sin transformación

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente Web/Agente<br/>Canal Agente]
    E[Experience API<br/>ntgy-fua-e-fua<br/>POST /signature/:traceId/retry]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /firmas/:traceId/relanzar]
    S1[System API<br/>ntgy-fua-s-rds-postgres]
    S2[System API<br/>ntgy-newco-s-aviva-evo]
    S3[System API<br/>ntgy-newco-s-salesforce]
    
    C -->|Request| E
    E -->|Consume| P
    P -->|Consulta Request Original| S1
    P -->|Reenvía SMS| S2
    P -->|Actualiza Caso| S3
    S1 -->|Response| P
    S2 -->|Response| P
    S3 -->|Response| P
    P -->|Response| E
    E -->|Pass-Through Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S3 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **SMS No Recibido**
   - Cliente no recibió el SMS original
   - Problemas de cobertura o entrega

2. **SMS Eliminado**
   - Cliente eliminó el SMS por error
   - Necesita un nuevo enlace de firma

3. **Cambio de Número**
   - Se actualizó el número de teléfono del cliente
   - Reenvío al nuevo número

4. **Solicitud del Agente**
   - Agente reenvía por solicitud del cliente
   - Gestión proactiva de firmas pendientes

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web (Portal de Agentes)

**Características del Canal:**
- **Plataforma**: Portal web de atención al cliente
- **Dispositivos**: Navegadores web en estaciones de trabajo
- **Conectividad**: Red corporativa estable
- **Experiencia de Usuario**: Interfaz de gestión para agentes

#### Audiencia Objetivo

**Usuarios Finales:**
- **Agentes de Call Center**: Reenvían SMS por solicitud del cliente
- **Supervisores**: Gestionan procesos de firma pendientes

**Contexto de Uso:**
- **Atención Telefónica**: Cliente llama porque no recibió el SMS
- **Seguimiento Proactivo**: Agente reenvía SMS de firmas pendientes
- **Corrección de Datos**: Reenvío tras actualizar número de teléfono

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Simplificación de Headers**: Solo propaga headers necesarios del contexto FUA
- **Timeout Extendido**: 60 segundos para operaciones de firma
- **Sin Transformación**: Respuesta directa del Process API

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/firmas/:traceId/relanzar` | POST | Reenvía SMS de firma, actualiza caso en Salesforce con comentario del reenvío |

### Contexto Empresarial

- **Dominio:** Gestión de Firma Digital
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Web (Portal de Agentes)

### Stakeholders

- **Agentes de Call Center**: Necesitan reenviar SMS por solicitud del cliente
- **Clientes**: Requieren recibir nuevamente el enlace de firma
- **Supervisores**: Supervisan la gestión de procesos de firma

### Valor de Negocio

- **Recuperación de Procesos**: Evita perder procesos de firma iniciados
- **Experiencia del Cliente**: No requiere reiniciar el proceso completo
- **Eficiencia Operativa**: Reduce tiempo de gestión de agentes
- **Trazabilidad**: Registro de cada reenvío en Salesforce

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/signature/:traceId/retry` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/signature/:traceId/retry` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/signature/:traceId/retry` |

### Método HTTP

```
POST /fua/api/signature/:traceId/retry
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Operaciones de firma pueden ser lentas |
| **Rate Limiting** | Según política API Gateway | Control de acceso corporativo |
| **Autenticación** | JWT obligatorio | Seguridad del canal agente |

#### Headers Específicos del Canal

**Headers Propagados por el Experience API:**

| Header | Tipo | Descripción | Obligatorio |
|--------|------|-------------|-------------|
| `x-client-code` | string | Código del cliente | Sí |
| `x-user-id` | string | ID del usuario/agente | Sí |
| `x-interaction-id` | string | ID de interacción | No |
| `x-user-role` | string | Rol del usuario | No |
| `traceparent` | string | Trazabilidad distribuida W3C | No |

### Parámetros de Entrada

#### Path Parameters

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `traceId` | string | Sí | Identificador único del proceso de firma a relanzar |

#### Headers Requeridos

| Header | Tipo | Descripción |
|--------|------|-------------|
| `Content-Type` | string | `application/json` |
| `Authorization` | string | JWT token del usuario/agente |
| `x-client-code` | string | Código del cliente |

### Respuestas

#### Respuesta Exitosa (200)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  }
}
```

#### Respuesta de Error (400)

```json
{
  "code": "APP:FIRMA_NOT_FOUND",
  "message": "No se encontró el proceso de firma",
  "details": "El traceId proporcionado no existe",
  "timestamp": "2026-10-20T11:00:00Z"
}
```

#### Respuesta de Error (500)

```json
{
  "code": "APP:AVIVA_SMS_FORWARDING_ERROR",
  "message": "Error al reenviar SMS de firma",
  "details": "Error en el servicio de firma digital",
  "timestamp": "2026-10-20T11:00:00Z"
}
```

### Autenticación

- **Tipo:** JWT (JSON Web Token)
- **Validación:** Mediante flow `jwt-validation`
- **Headers:** Authorization header con Bearer token

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Operaciones de firma pueden ser lentas |
| **Connection Timeout** | 60 segundos | Configurado en HTTP connector |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request<br/>Canal Agente]) --> JWT[JWT Validation]
    JWT --> SetHeaders[Set fuaHeaders Variable]
    SetHeaders --> Router[APIKit Router]
    Router --> ExtractVars[Extract Variables:<br/>traceId, clientCode]
    ExtractVars --> CallProcess[Call Process API<br/>POST /firmas/:traceId/relanzar]
    CallProcess --> End([HTTP Response<br/>Pass-Through])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style CallProcess fill:#ffe1f5,stroke:#c2185b
```

### Transformaciones de Canal

**⚠️ Este Experience API NO transforma la respuesta del Process API.**

Es un **pass-through puro** que:
1. Extrae el `traceId` de la URI
2. Extrae el `clientCode` de los headers FUA
3. Propaga los headers de contexto
4. Invoca el Process API
5. Retorna la respuesta sin modificación

Para el schema completo de request/response, consultar:
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `POST /firmas/:traceId/relanzar`**

### Código de Implementación

**Ubicación:** `src/main/mule/e-fua.xml` (líneas 63-72)

```xml
<flow name="post:\signature\(traceId)\retry:application\json:e-fua-config" doc:id="de8a60c2-a150-4e05-ac48-752b6f94c2dc">
    <ee:transform doc:name="Vars" doc:id="e061b065-6ca2-4292-99c6-edf9bbd60ed5">
        <ee:variables>
            <ee:set-variable variableName="traceId"><![CDATA[attributes.uriParams.'traceId']]></ee:set-variable>
            <ee:set-variable variableName="clientCode"><![CDATA[vars.fuaHeaders.'x-client-code']]></ee:set-variable>
        </ee:variables>
    </ee:transform>
    <flow-ref doc:name="retrySignature" doc:id="c0c36394-af4e-437a-8921-7e2cff0ae2d2" name="post-signature-retryFlow"/>
</flow>
```

**Ubicación:** `src/main/mule/signature.xml` (líneas 20-29)

```xml
<flow name="post-signature-retryFlow" doc:id="3ec808b7-5e18-4eea-a3bf-e11b6776a94f">
    <http:request method="POST" 
                  doc:name="Newco-evo Post Firma" 
                  doc:id="dd90f862-f38d-4035-919e-ec1343f92fc8" 
                  config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
                  path="/firmas/{traceId}/relanzar" 
                  outputMimeType="application/json" 
                  outputEncoding="UTF-8" 
                  sendBodyMode="ALWAYS">
        <http:headers><![CDATA[#[vars.fuaHeaders]]]></http:headers>
        <http:uri-params><![CDATA[#[output application/java
---
{
    "traceId" : vars.traceId
}]]]></http:uri-params>
    </http:request>
</flow>
```

### Patrones de Diseño

**Patrón:** API Gateway / Pass-Through

**Justificación:** 
- Separa la capa de experiencia de la lógica de negocio
- Permite evolucionar el Process API sin afectar al cliente
- Centraliza la gestión de headers y contexto FUA
- Facilita el cambio de backend sin impactar al consumidor

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente Web/Agente
    participant E as Experience API
    participant P as Process API
    
    C->>E: POST /signature/:traceId/retry
    Note over E: JWT Validation
    Note over E: Extract traceId & clientCode
    E->>P: POST /firmas/:traceId/relanzar<br/>(con headers FUA)
    Note over P: Consulta request original<br/>Reenvía SMS en Aviva<br/>Actualiza caso en Salesforce
    P-->>E: Response OK
    E-->>C: Pass-Through Response
```

#### 2. Flujo con Error de Negocio

```mermaid
sequenceDiagram
    participant C as Cliente Web/Agente
    participant E as Experience API
    participant P as Process API
    
    C->>E: POST /signature/:traceId/retry
    Note over E: JWT Validation
    E->>P: POST /firmas/:traceId/relanzar
    Note over P: Firma no encontrada
    P-->>E: 400 Bad Request
    E-->>C: Pass-Through Error
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant C as Cliente Web/Agente
    participant E as Experience API
    participant P as Process API
    
    C->>E: POST /signature/:traceId/retry
    Note over E: JWT Validation
    E->>P: POST /firmas/:traceId/relanzar
    Note over P: Error en Aviva
    P-->>E: 500 Internal Error
    E-->>C: Pass-Through Error
```

### Configuración HTTP Connector

**Connector:** `HTTP_FUA_PROCESS_NEWCO_EVO`

**Configuración (global.xml líneas 30-32):**
```xml
<http:request-config name="HTTP_FUA_PROCESS_NEWCO_EVO" 
                     basePath="/api" 
                     responseTimeout="${newco-p-fua-evo.timeout}">
    <http:request-connection host="${newco-p-fua-evo.host}" 
                             port="${newco-p-fua-evo.port}" 
                             connectionIdleTimeout="${newco-p-fua-evo.timeout}"/>
</http:request-config>
```

**Properties (e-fua-pro-config.yaml líneas 25-28):**
```yaml
newco-p-fua-evo:
  host: "mule-worker-internal-ntgy-newco-p-fua-evo.de-c1.cloudhub.io"
  port: "8091"
  timeout: "60000"  # 60 segundos
```

### Manejo de Errores

**Error Handler:** Global error handler de APIKit

**Errores Manejados:**
- `APIKIT:BAD_REQUEST` → 400
- `APIKIT:NOT_FOUND` → 404
- `APIKIT:METHOD_NOT_ALLOWED` → 405
- Errores del Process API → Pass-through

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Llamada Exitosa desde Portal de Agentes

#### cURL

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/signature/abc123-def456-ghi789/retry" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-client-code: 12345678" \
  -H "x-user-id: agent001"
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

### Ejemplo 2: JavaScript (para Portal Web)

```javascript
const reenviarSMSFirma = async (traceId, authToken, clientCode) => {
  try {
    const response = await fetch(
      `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/signature/${traceId}/retry`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${authToken}`,
          'x-client-code': clientCode,
          'x-user-id': getUserId()
        }
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    console.log('SMS reenviado:', data);
    return data;
  } catch (error) {
    console.error('Error al reenviar SMS:', error);
    throw error;
  }
};

// Uso
reenviarSMSFirma('abc123-def456-ghi789', getAuthToken(), '12345678');
```

### Ejemplo 3: React Component para Agentes

```javascript
import React, { useState } from 'react';

const ReenviarSMSButton = ({ traceId, clientCode }) => {
  const [loading, setLoading] = useState(false);
  const [message, setMessage] = useState('');

  const handleReenviar = async () => {
    setLoading(true);
    setMessage('');
    
    try {
      const response = await fetch(
        `/fua/api/signature/${traceId}/retry`,
        {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${localStorage.getItem('token')}`,
            'x-client-code': clientCode,
            'x-user-id': localStorage.getItem('userId')
          }
        }
      );

      if (response.ok) {
        setMessage('✅ SMS reenviado correctamente');
      } else {
        const error = await response.json();
        setMessage(`❌ Error: ${error.message}`);
      }
    } catch (error) {
      setMessage('❌ Error de conexión');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="reenviar-sms-container">
      <button 
        onClick={handleReenviar} 
        disabled={loading}
        className="btn btn-primary"
      >
        {loading ? 'Reenviando...' : 'Reenviar SMS'}
      </button>
      {message && <div className="alert mt-2">{message}</div>}
    </div>
  );
};

export default ReenviarSMSButton;
```

### Ejemplo 4: Error - Firma No Encontrada

#### Request

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/signature/invalid-trace-id/retry" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-client-code: 12345678"
```

#### Response (400 Bad Request)

```json
{
  "code": "APP:FIRMA_NOT_FOUND",
  "message": "No se encontró el proceso de firma",
  "details": "El traceId proporcionado no existe en la base de datos",
  "timestamp": "2026-10-20T11:00:00Z"
}
```
