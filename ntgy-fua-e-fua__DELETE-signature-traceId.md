---
api: "ntgy-fua-e-fua"
sidebar_position: 2
sidebar_label: "DELETE /signature/:traceId"
description: "Cancela un proceso de firma digital en curso mediante el traceId - Orquesta la cancelación en Process API"
keywords:
  - cancelar firma
  - firma digital
  - traceid
  - experience api
  - canal agente
  - orquestacion
tags:
  - Experience API
  - Firma Digital
  - Cancelación
  - Canal Agente
channel: web
endpoint_dependencies:
  - method: "DELETE"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/firmas/:traceId"
    purpose: "Cancela la firma en Aviva, actualiza estado en RDS y registra evento en Motor de Estados"
---

# DELETE /signature/:traceId

## 📌 Descripción General

### Propósito del Endpoint

Endpoint de Experience API que permite cancelar un proceso de firma digital en curso. Actúa como orquestador que delega la lógica de cancelación al Process API correspondiente, adaptando la experiencia para el canal web/agente.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Orquestación de cancelación de firma para canal agente

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
    E[Experience API<br/>ntgy-fua-e-fua<br/>DELETE /signature/:traceId]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>DELETE /firmas/:traceId]
    S1[System API<br/>ntgy-fua-s-rds-postgres]
    S2[System API<br/>ntgy-newco-s-aviva-evo]
    
    C -->|Request| E
    E -->|Consume| P
    P -->|Consulta/Actualiza| S1
    P -->|Cancela Firma| S2
    S1 -->|Response| P
    S2 -->|Response| P
    P -->|Response| E
    E -->|Pass-Through Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Cancelación por Cliente**
   - Cliente decide no completar el proceso de firma
   - Evita confusión con firmas pendientes

2. **Cancelación por Agente**
   - Agente de call center cancela por solicitud del cliente
   - Corrección de errores en el proceso

3. **Gestión de Errores**
   - Cancelación preventiva ante detección de errores
   - Limpieza de procesos obsoletos

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
- **Agentes de Call Center**: Gestionan cancelaciones por solicitud del cliente
- **Supervisores**: Supervisan y gestionan procesos de firma

**Contexto de Uso:**
- **Atención Telefónica**: Cliente llama para cancelar una firma
- **Corrección de Errores**: Se detecta un error en el proceso iniciado
- **Gestión Proactiva**: Limpieza de firmas pendientes obsoletas

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Simplificación de Headers**: Solo propaga headers necesarios del contexto FUA
- **Timeout Extendido**: 60 segundos para operaciones de firma
- **Sin Transformación**: Respuesta directa del Process API

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/firmas/:traceId` | DELETE | Cancela firma en Aviva, actualiza estado en RDS y registra evento en Motor de Estados |

### Contexto Empresarial

- **Dominio:** Gestión de Firma Digital
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Web (Portal de Agentes)

### Stakeholders

- **Agentes de Call Center**: Necesitan cancelar firmas por solicitud del cliente
- **Clientes**: Requieren poder cancelar procesos de firma iniciados
- **Supervisores**: Supervisan la gestión de procesos de firma

### Valor de Negocio

- **Control del Proceso**: Permite cancelar firmas que no se completarán
- **Experiencia del Cliente**: Evita confusión con procesos pendientes
- **Eficiencia Operativa**: Limpieza de procesos obsoletos
- **Trazabilidad**: Registro completo de cancelaciones

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/signature/:traceId` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/signature/:traceId` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/signature/:traceId` |

### Método HTTP

```
DELETE /fua/api/signature/:traceId
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
| `traceId` | string | Sí | Identificador único del proceso de firma a cancelar |

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
  "code": "APP:AVIVA_REQUEST_CANCELATION_ERROR",
  "message": "Error al cancelar la firma",
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
    Start([HTTP DELETE Request<br/>Canal Agente]) --> JWT[JWT Validation]
    JWT --> SetHeaders[Set fuaHeaders Variable]
    SetHeaders --> Router[APIKit Router]
    Router --> ExtractVars[Extract Variables:<br/>traceId, clientCode]
    ExtractVars --> CallProcess[Call Process API<br/>DELETE /firmas/:traceId]
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
📖 **Process API: `ntgy-newco-p-fua-evo` endpoint `DELETE /firmas/:traceId`**

### Código de Implementación

**Ubicación:** `src/main/mule/e-fua.xml` (líneas 53-62)

```xml
<flow name="delete:\signature\(traceId):e-fua-config" doc:id="53212566-d0a9-402f-852b-044a4f34bbec">
    <ee:transform doc:name="Vars" doc:id="525a0042-9f3b-4a5b-bc52-26beb745b2ae">
        <ee:variables>
            <ee:set-variable variableName="traceId"><![CDATA[attributes.uriParams.'traceId']]></ee:set-variable>
            <ee:set-variable variableName="clientCode"><![CDATA[vars.fuaHeaders.'x-client-code']]></ee:set-variable>
        </ee:variables>
    </ee:transform>
    <flow-ref doc:name="deleteSignature" doc:id="92d7cc4e-ae03-4cff-9c6b-9a1befb7d343" name="delete-signatureFlow"/>
</flow>
```

**Ubicación:** `src/main/mule/signature.xml` (líneas 7-18)

```xml
<flow name="delete-signatureFlow" doc:id="40e26b35-10c3-4085-bdf6-a87a8d28cd16">
    <http:request method="DELETE" 
                  doc:name="Newco-evo Post Firma" 
                  doc:id="4e3bfe93-a059-4e77-9910-4d7088eaa0c1" 
                  config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
                  path="/firmas/{traceId}" 
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
    
    C->>E: DELETE /signature/:traceId
    Note over E: JWT Validation
    Note over E: Extract traceId & clientCode
    E->>P: DELETE /firmas/:traceId<br/>(con headers FUA)
    Note over P: Cancela en Aviva<br/>Actualiza RDS<br/>Registra evento
    P-->>E: Response OK
    E-->>C: Pass-Through Response
```

#### 2. Flujo con Error de Negocio

```mermaid
sequenceDiagram
    participant C as Cliente Web/Agente
    participant E as Experience API
    participant P as Process API
    
    C->>E: DELETE /signature/:traceId
    Note over E: JWT Validation
    E->>P: DELETE /firmas/:traceId
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
    
    C->>E: DELETE /signature/:traceId
    Note over E: JWT Validation
    E->>P: DELETE /firmas/:traceId
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
curl -X DELETE "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/signature/abc123-def456-ghi789" \
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
const cancelarFirma = async (traceId, authToken, clientCode) => {
  try {
    const response = await fetch(
      `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/signature/${traceId}`,
      {
        method: 'DELETE',
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
    console.log('Firma cancelada:', data);
    return data;
  } catch (error) {
    console.error('Error al cancelar firma:', error);
    throw error;
  }
};

// Uso
cancelarFirma('abc123-def456-ghi789', getAuthToken(), '12345678');
```

### Ejemplo 3: React Component para Agentes

```javascript
import React, { useState } from 'react';

const CancelarFirmaButton = ({ traceId, clientCode }) => {
  const [loading, setLoading] = useState(false);
  const [message, setMessage] = useState('');

  const handleCancelar = async () => {
    setLoading(true);
    setMessage('');
    
    try {
      const response = await fetch(
        `/fua/api/signature/${traceId}`,
        {
          method: 'DELETE',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${localStorage.getItem('token')}`,
            'x-client-code': clientCode,
            'x-user-id': localStorage.getItem('userId')
          }
        }
      );

      if (response.ok) {
        setMessage('✅ Firma cancelada correctamente');
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
    <div className="cancelar-firma-container">
      <button 
        onClick={handleCancelar} 
        disabled={loading}
        className="btn btn-danger"
      >
        {loading ? 'Cancelando...' : 'Cancelar Firma'}
      </button>
      {message && <div className="alert mt-2">{message}</div>}
    </div>
  );
};

export default CancelarFirmaButton;
```

### Ejemplo 4: Error - Firma No Encontrada

#### Request

```bash
curl -X DELETE "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/signature/invalid-trace-id" \
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