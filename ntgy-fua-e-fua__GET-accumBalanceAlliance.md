---
api: "ntgy-fua-e-fua"
sidebar_position: 4
sidebar_label: "GET /accumBalanceAlliance"
description: "Consulta el saldo acumulado de puntos en el programa de alianzas del cliente"
keywords: [alianzas, saldo acumulado, puntos, programa fidelización, GOW]
tags: [Alianzas, Consultas, Experience API, FUA]
channel: web
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/accumBalanceAlliance"
    purpose: "Orquesta la consulta del saldo acumulado de alianzas validando el cliente"
---

# GET /accumBalanceAlliance

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite consultar el saldo acumulado de puntos que un cliente tiene en el programa de alianzas de Naturgy. El endpoint orquesta la consulta a través de las capas Process y System API, validando la existencia del cliente en el sistema de alianzas externo.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para canal web (FUA - Frontal Único de Atención)

### Patrón de Integración

**Tipo:** Pass-Through con Manejo de Errores Especial

**Descripción:** Este endpoint actúa como pass-through, propagando la respuesta del Process API sin transformaciones adicionales. Implementa un manejo especial de errores para el caso de "cliente no existe", que se trata como un caso de éxito controlado.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Portal FUA<br/>Agente]
    E[Experience API<br/>ntgy-fua-e-fua<br/>GET /accumBalanceAlliance]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>GET /accumBalanceAlliance]
    S[System API<br/>ntgy-newco-s-alianzas<br/>GET /saldo-acumulado]
    B[API REST Externa<br/>Sistema de Alianzas]
    
    C -->|Request con idGow| E
    E -->|Propaga queryParams + fuaHeaders| P
    P -->|Transforma a customer_id| S
    S -->|Consulta REST| B
    B -->|Response| S
    S -->|Response| P
    P -->|Response| E
    E -->|Response sin transformar| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style B fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Consulta de Saldo de Puntos por Agente**
   - Agente de FUA consulta el saldo acumulado de un cliente en el programa de alianzas
   - Permite verificar los puntos disponibles para canje o beneficios
   - Facilita la gestión de consultas sobre el programa de fidelización

2. **Validación de Participación en Programa**
   - Verifica si el cliente está registrado en el sistema de alianzas
   - Identifica clientes no registrados para posible alta en el programa
   - Proporciona información para campañas de captación

3. **Soporte a Gestión de Beneficios**
   - Información base para gestión de canjes de puntos
   - Soporte a consultas sobre estado del programa de fidelización
   - Facilita la resolución de incidencias relacionadas con puntos

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web (Portal FUA)

**Características del Canal:**
- **Plataforma**: Portal web interno para agentes de atención al cliente
- **Dispositivos**: Estaciones de trabajo de agentes (desktop)
- **Conectividad**: Red corporativa interna de alta disponibilidad
- **Experiencia de Usuario**: Interfaz de agente con múltiples consultas simultáneas

#### Audiencia Objetivo

**Usuarios Finales:**
- **Agentes de Atención al Cliente**: Personal de call center que atiende consultas sobre el programa de alianzas
- **Supervisores de Atención**: Personal que supervisa y resuelve casos escalados

**Contexto de Uso:**
- **Consulta durante llamada**: Agente consulta saldo mientras atiende al cliente por teléfono
- **Gestión de incidencias**: Verificación de saldo para resolver reclamaciones o dudas
- **Información para canjes**: Consulta previa a gestión de canje de puntos

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Propagación de Headers FUA**: Incluye headers de autorización y tracking específicos del portal
- **Manejo de Errores Controlado**: Caso especial para "cliente no existe" tratado como respuesta válida
- **Timeout Extendido**: 60 segundos para permitir consulta a sistema externo
- **Sin Transformación**: Respuesta directa del Process API para máxima fidelidad de datos

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/accumBalanceAlliance` | GET | Orquesta la consulta del saldo acumulado validando el cliente en el sistema de alianzas |

**Cadena de Dependencias Completa:**
```
Experience API (ntgy-fua-e-fua)
    ↓
Process API (ntgy-newco-p-fua-evo)
    ↓
System API (ntgy-newco-s-alianzas)
    ↓
API REST Externa (Sistema de Alianzas)
```

### Contexto Empresarial

- **Dominio:** Programa de Fidelización y Alianzas
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Web (Portal FUA - Agentes)

### Stakeholders

- **Agentes de Atención al Cliente**: Necesitan consultar rápidamente el saldo de puntos durante la atención
- **Clientes Finales**: Beneficiarios indirectos que reciben información sobre su saldo de puntos
- **Gestores del Programa de Alianzas**: Utilizan la información para análisis y gestión del programa

### Valor de Negocio

- **Mejora de Atención al Cliente**: Información inmediata sobre saldo de puntos durante la llamada
- **Reducción de Tiempos de Gestión**: Consulta integrada en el portal FUA sin necesidad de acceder a sistemas externos
- **Soporte a Programa de Fidelización**: Facilita la gestión y promoción del programa de alianzas
- **Trazabilidad**: Headers FUA permiten seguimiento completo de consultas realizadas

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/accumBalanceAlliance` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/accumBalanceAlliance` |
| **UAT** | `https://ntgy-fua-e-fua-uat.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-uat.de-c1.cloudhub.io/fua/api/accumBalanceAlliance` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/accumBalanceAlliance` |

### Método HTTP

```
GET /fua/api/accumBalanceAlliance
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Consulta a sistema externo de alianzas puede requerir tiempo adicional |
| **Rate Limiting** | Según política API Gateway | Control de uso por agente/sesión |
| **Autenticación** | OAuth 2.0 + Headers FUA | Seguridad y trazabilidad de consultas de agentes |

#### Campos Específicos del Canal

**Headers FUA Propagados:**

| Header | Tipo | Descripción | Razón |
|--------|------|-------------|-------|
| `Authorization` | string | Token OAuth 2.0 del agente | Autenticación y autorización |
| `X-Correlation-ID` | string | ID de correlación de la sesión | Trazabilidad end-to-end |
| `X-Agent-ID` | string | Identificador del agente | Auditoría de consultas |

**Respuesta (Pass-Through):**

⚠️ **Este Experience API NO transforma la respuesta del Process API.**

**La estructura de respuesta NO está definida en este nivel.**

Para conocer el schema completo de la respuesta, consultar la documentación de:
- 📖 **Process API:** `ntgy-newco-p-fua-evo` endpoint `/accumBalanceAlliance`
- 📖 **System API:** `ntgy-newco-s-alianzas` (endpoint de saldo acumulado)

**Única estructura verificable en el código del Experience API:**
```json
{
  "result": {
    "codResult": "0000" | "0001",
    "msgResult": "string"
  }
}
```

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `idGow` | string | ✅ Sí | Identificador del cliente en sistema GOW | `"12345678"` |

**Notas:**
- El parámetro `idGow` se propaga directamente al Process API
- Los headers FUA se añaden automáticamente desde `vars.fuaHeaders`

#### Headers

| Header | Tipo | Requerido | Descripción |
|--------|------|-----------|-------------|
| `Authorization` | string | ✅ Sí | Token Bearer OAuth 2.0 |
| `Content-Type` | string | ✅ Sí | `application/json` |
| `X-Correlation-ID` | string | ⚠️ Recomendado | ID de correlación para trazabilidad |

### Respuestas

⚠️ **IMPORTANTE: Este Experience API actúa como pass-through completo.**

**La estructura de respuesta NO se transforma en este nivel y NO está definida en el código del Experience API.**

Para conocer la estructura exacta de la respuesta, consultar la documentación de:
- **Process API:** `ntgy-newco-p-fua-evo` endpoint `/accumBalanceAlliance`
- **System API:** `ntgy-newco-s-alianzas` (endpoint de saldo acumulado)

#### Estructura Mínima Garantizada

**Lo único verificable en el código del Experience API es la estructura de error:**

```json
{
  "result": {
    "codResult": "0000" | "0001",
    "msgResult": "string"
  }
}
```

**Códigos de Resultado Verificables:**
- `"0000"` - Operación exitosa (incluye caso especial "No existe cliente")
- `"0001"` - Error en la operación

#### 200 OK - Respuesta Exitosa

**Descripción:** La consulta se completó correctamente.

**Nota:** La estructura completa de datos depende del Process API y System API downstream. Este Experience API propaga la respuesta sin modificaciones.

#### 200 OK - Cliente No Existe (Caso Especial)

**Descripción:** Cliente no registrado en sistema de alianzas (tratado como éxito controlado en el Process API).

**Estructura Verificable:**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No existe cliente"
  }
}
```

**Nota:** Este caso especial se maneja en el Process API (`ntgy-newco-p-fua-evo`) y se propaga como respuesta válida.

#### 400 Bad Request

**Descripción:** Parámetros de entrada inválidos o error de validación.

**Estructura de Error (verificable en código):**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Mensaje de error del Process API"
  }
}
```

#### 401 Unauthorized

**Descripción:** Token de autenticación inválido o expirado.

**Estructura de Error (verificable en código):**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Mensaje de error del Process API"
  }
}
```

#### 403 Forbidden

**Descripción:** Acceso no autorizado al recurso.

**Estructura de Error (verificable en código):**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Mensaje de error del Process API"
  }
}
```

#### 404 Not Found

**Descripción:** Recurso no encontrado.

**Estructura de Error (verificable en código):**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Mensaje de error del Process API"
  }
}
```

#### 500 Internal Server Error

**Descripción:** Error interno del servidor o del sistema de alianzas.

**Estructura de Error (verificable en código):**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Mensaje de error del Process API o error.description"
  }
}
```

**Nota:** El mensaje de error (`msgResult`) se extrae del Process API si está disponible, o del `error.description` en caso contrario (verificable en alliances.xml, líneas 23-30).

### Autenticación

- **Tipo:** OAuth 2.0 Bearer Token
- **Scope:** Acceso a consultas de alianzas
- **Headers Requeridos:** `Authorization: Bearer {token}`

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Consulta a sistema externo de alianzas puede requerir tiempo adicional |
| **Connection Idle Timeout** | 60 segundos | Mantiene conexión activa para múltiples consultas |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request<br/>Portal FUA]) --> Flow1[Flow: get:\accumBalanceAlliance:e-fua-config]
    Flow1 --> FlowRef[flow-ref: get-alianza-accumBalanceAlliance]
    FlowRef --> HTTPReq[HTTP Request: ntgy-newco-p-fua-evo<br/>GET /accumBalanceAlliance]
    HTTPReq --> BuildParams[Build Query Params<br/>attributes.queryParams ++ vars.fuaHeaders]
    BuildParams --> CallProcess[Call Process API<br/>Timeout: 60s]
    CallProcess --> Success{Success?}
    Success -->|Yes| Response[Return Response<br/>Sin transformar]
    Success -->|No| ErrorHandler[Error Handler]
    ErrorHandler --> CheckError{Error Type?}
    CheckError -->|400| Set400[Set httpStatus = 400]
    CheckError -->|401| Set401[Set httpStatus = 401]
    CheckError -->|403| Set403[Set httpStatus = 403]
    CheckError -->|404| Set404[Set httpStatus = 404]
    CheckError -->|Other| Set500[Set httpStatus = 500]
    Set400 --> BuildError[Build Error Response]
    Set401 --> BuildError
    Set403 --> BuildError
    Set404 --> BuildError
    Set500 --> BuildError
    BuildError --> LogError[Log Error]
    LogError --> ErrorResponse[Return Error Response]
    Response --> End([HTTP Response])
    ErrorResponse --> End
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style ErrorHandler fill:#ffebee,stroke:#c62828
    style HTTPReq fill:#fff4e1,stroke:#f57c00
```

### Transformaciones de Canal

**Este endpoint NO transforma datos:**

⚠️ **Pass-Through Completo**

Este Experience API actúa como pass-through, propagando la respuesta del Process API sin modificaciones. La única transformación aplicada es en el manejo de errores.

**Propagación de Parámetros:**

```dataweave
%dw 2.0
output application/java
---
attributes.queryParams ++ vars.fuaHeaders
```

**Descripción:**
- Combina los query parameters de la request original
- Añade los headers FUA almacenados en variables
- Propaga todo al Process API

### Patrones de Diseño

**Patrón:** Pass-Through con Error Handling Enriquecido

**Justificación:** 
- El canal FUA requiere la respuesta completa del Process API sin modificaciones
- El manejo de errores se enriquece con códigos HTTP específicos para el portal
- Los headers FUA se propagan para trazabilidad completa

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant A as Agente FUA
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant S as System API<br/>ntgy-newco-s-alianzas
    participant B as API Alianzas<br/>Externa
    
    A->>E: GET /accumBalanceAlliance?idGow=12345678
    Note over E: Extrae queryParams<br/>Añade fuaHeaders
    E->>P: GET /accumBalanceAlliance<br/>queryParams + fuaHeaders
    Note over P: Procesa consulta<br/>(ver doc Process API)
    P->>S: Consulta a System API<br/>(ver doc Process API)
    S->>B: Consulta a API Externa
    B-->>S: Response
    S-->>P: Response
    Note over P: Log Output
    P-->>E: Response sin transformar
    E-->>A: Response pass-through<br/>(estructura definida en Process API)
```

#### 2. Flujo con Cliente No Existe (Caso Especial)

```mermaid
sequenceDiagram
    participant A as Agente FUA
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant S as System API<br/>ntgy-newco-s-alianzas
    participant B as API Alianzas<br/>Externa
    
    A->>E: GET /accumBalanceAlliance?idGow=99999999
    E->>P: GET /accumBalanceAlliance<br/>queryParams + fuaHeaders
    P->>S: GET /saldo-acumulado?customer_id=99999999
    S->>B: GET /api/balance?customer_id=99999999
    B-->>S: 404 Not Found<br/>"Cannot identify the customer"
    Note over S: Error Handler
    S-->>P: Error Response
    Note over P: Choice: Error contains<br/>"identify the customer"
    Note over P: Transform to Success<br/>codResult: "0000"<br/>msgResult: "No existe cliente"
    Note over P: Raise NTGY:ALIANZAS_CONTINUE
    P-->>E: 200 OK<br/>{result: {codResult: "0000",<br/>msgResult: "No existe cliente"}}
    E-->>A: 200 OK<br/>{result: {codResult: "0000",<br/>msgResult: "No existe cliente"}}
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant A as Agente FUA
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant S as System API<br/>ntgy-newco-s-alianzas
    
    A->>E: GET /accumBalanceAlliance?idGow=12345678
    E->>P: GET /accumBalanceAlliance<br/>queryParams + fuaHeaders
    P->>S: GET /saldo-acumulado?customer_id=12345678
    Note over S: Timeout o Error<br/>Sistema Externo
    S-->>P: 500 Internal Server Error
    Note over P: Error Handler<br/>Transform Error
    P-->>E: 500 Internal Server Error<br/>{result: {codResult: "0001",<br/>msgResult: "Error..."}}
    Note over E: Error Handler<br/>Extract Error Message
    Note over E: Determine HTTP Status<br/>from Error Message
    Note over E: Log Error
    E-->>A: 500 Internal Server Error<br/>{result: {codResult: "0001",<br/>msgResult: "Error..."}}
```

### Manejo de Errores

#### Error Handler del Experience API

**Ubicación:** Flow `get-alianza-accumBalanceAlliance` (alliances.xml, líneas 18-42)

**Estrategia:**

1. **Captura de Error:**
   ```xml
   <on-error-propagate enableNotifications="true" logException="true">
   ```

2. **Transformación de Error:**
   ```dataweave
   %dw 2.0
   output application/json
   ---
   {
     "result": {
       "codResult": "0001",
       "msgResult": if(error.errorMessage.payload.result.msgResult != null) 
                      error.errorMessage.payload.result.msgResult 
                    else 
                      error.description
     }
   }
   ```

3. **Determinación de HTTP Status:**
   ```dataweave
   %dw 2.0
   output application/java
   ---
   if((error.errorMessage.payload.result.msgResult contains ("400")) or (error.description contains ("400")))
     400
   else if((error.errorMessage.payload.result.msgResult contains ("401")) or (error.description contains ("401")))
     401
   else if((error.errorMessage.payload.result.msgResult contains ("403")) or (error.description contains ("403")))
     403
   else if((error.errorMessage.payload.result.msgResult contains ("404")) or (error.description contains ("404")))
     404
   else
     500
   ```

4. **Logging:**
   ```xml
   <logger level="INFO" message="Error output: #[payload]"/>
   ```

#### Manejo Especial en Process API

**Caso: Cliente No Existe**

El Process API implementa un manejo especial para el error "Cannot identify the customer":

```xml
<choice>
  <when expression='#[error.errorMessage.payload.result.msgResult contains("identify the customer")]'>
    <!-- Transforma a éxito controlado -->
    <ee:transform>
      <ee:set-payload>
        {
          "result": {
            "codResult": "0000",
            "msgResult": "No existe cliente"
          }
        }
      </ee:set-payload>
    </ee:transform>
    <raise-error type="NTGY:ALIANZAS_CONTINUE" description="No existe cliente"/>
  </when>
  <otherwise>
    <!-- Error normal -->
  </otherwise>
</choice>
```

**Rationale:** Este caso se considera un resultado válido del negocio, no un error técnico.

### Configuración de Conectores

#### HTTP Request Configuration

**Nombre:** `HTTP_FUA_PROCESS_NEWCO_EVO`

**Configuración (global.xml, líneas 30-32):**

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

**Properties (e-fua-pro-config.yaml, líneas 25-28):**

```yaml
newco-p-fua-evo:
  host: "mule-worker-internal-ntgy-newco-p-fua-evo.de-c1.cloudhub.io"
  port: "8091"
  timeout: "60000"
```

**Características:**
- **Timeout:** 60 segundos (extendido para consulta a sistema externo)
- **Connection Idle Timeout:** 60 segundos
- **Base Path:** `/api`
- **Protocol:** HTTP (interno CloudHub)

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta Exitosa desde Portal FUA

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/accumBalanceAlliance?idGow=12345678" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "X-Correlation-ID: 550e8400-e29b-41d4-a716-446655440000" \
  -H "X-Agent-ID: AGENT001"
```

#### Respuesta (200 OK)

⚠️ **La estructura de respuesta depende del Process API downstream.**

**Estructura mínima garantizada:**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "string"
  }
}
```

**Nota:** Para la estructura completa de datos, consultar la documentación del Process API `ntgy-newco-p-fua-evo`.

### Ejemplo 2: Cliente No Existe (Caso Especial)

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/accumBalanceAlliance?idGow=99999999" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "X-Correlation-ID: 550e8400-e29b-41d4-a716-446655440001"
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No existe cliente"
  }
}
```

### Ejemplo 3: Error de Parámetros

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/accumBalanceAlliance" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Respuesta (400 Bad Request)

**Estructura de error verificable:**
```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Mensaje de error del Process API"
  }
}
```

**Nota:** El mensaje específico depende de la validación realizada en el Process API.

### Ejemplo 4: JavaScript (Fetch API)

```javascript
// Consulta de saldo acumulado desde portal FUA
const consultarSaldoAcumulado = async (idGow, token) => {
  try {
    const response = await fetch(
      `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/accumBalanceAlliance?idGow=${idGow}`,
      {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`,
          'X-Correlation-ID': generateCorrelationId(),
          'X-Agent-ID': 'AGENT001'
        }
      }
    );

    const data = await response.json();
    
    // Verificar código de resultado
    if (data.result.codResult === '0000') {
      // Caso especial: cliente no existe
      if (data.result.msgResult === 'No existe cliente') {
        console.log('Cliente no registrado en programa de alianzas');
        return { exists: false, message: data.result.msgResult };
      }
      
      // Éxito - la estructura de datos depende del Process API
      console.log('Consulta exitosa');
      return { exists: true, data: data };
    } else {
      // Error
      console.error('Error:', data.result.msgResult);
      throw new Error(data.result.msgResult);
    }
  } catch (error) {
    console.error('Error en consulta:', error);
    throw error;
  }
};

// Función auxiliar para generar correlation ID
const generateCorrelationId = () => {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, (c) => {
    const r = Math.random() * 16 | 0;
    const v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
};

// Uso
consultarSaldoAcumulado('12345678', 'your-token-here')
  .then(result => {
    if (result.exists) {
      console.log('Datos del cliente:', result.data);
      // Procesar según estructura definida en Process API
    } else {
      console.log('Cliente no existe:', result.message);
    }
  })
  .catch(error => console.error('Error:', error));
```

**Nota:** La estructura de `result.data` depende del Process API `ntgy-newco-p-fua-evo`. Consultar su documentación para conocer los campos disponibles.

### Ejemplo 5: Python

```python
import requests
from typing import Optional, Dict

def consultar_saldo_acumulado(
    id_gow: str,
    token: str,
    base_url: str = "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io"
) -> Optional[Dict]:
    """
    Consulta el saldo acumulado de alianzas de un cliente.
    
    Args:
        id_gow: Identificador del cliente en sistema GOW
        token: Token de autenticación OAuth 2.0
        base_url: URL base del API (por defecto producción)
    
    Returns:
        Dict con la respuesta completa o None si el cliente no existe
        La estructura de datos depende del Process API downstream
    
    Raises:
        requests.exceptions.RequestException: Error en la petición HTTP
        ValueError: Error en la respuesta del API
    """
    url = f"{base_url}/fua/api/accumBalanceAlliance"
    
    headers = {
        'Content-Type': 'application/json',
        'Authorization': f'Bearer {token}',
        'X-Correlation-ID': '550e8400-e29b-41d4-a716-446655440000',
        'X-Agent-ID': 'AGENT001'
    }
    
    params = {
        'idGow': id_gow
    }
    
    try:
        response = requests.get(url, headers=headers, params=params, timeout=65)
        response.raise_for_status()
        
        data = response.json()
        
        # Verificar código de resultado
        if data['result']['codResult'] == '0000':
            # Caso especial: cliente no existe
            if data['result']['msgResult'] == 'No existe cliente':
                print(f"Cliente {id_gow} no registrado en programa de alianzas")
                return None
            
            # Éxito - retornar respuesta completa
            print(f"Consulta exitosa para cliente {id_gow}")
            return data
        else:
            raise ValueError(f"Error en respuesta: {data['result']['msgResult']}")
            
    except requests.exceptions.Timeout:
        print("Timeout en consulta de saldo acumulado")
        raise
    except requests.exceptions.RequestException as e:
        print(f"Error en petición HTTP: {e}")
        raise

# Uso
if __name__ == "__main__":
    try:
        resultado = consultar_saldo_acumulado(
            id_gow='12345678',
            token='your-token-here'
        )
        
        if resultado:
            print("Respuesta completa:", resultado)
            # Procesar según estructura definida en Process API
            # Consultar documentación de ntgy-newco-p-fua-evo para campos disponibles
        else:
            print("Cliente no existe en programa de alianzas")
    except Exception as e:
        print(f"Error: {e}")
```

**Nota:** La estructura de datos en la respuesta depende del Process API `ntgy-newco-p-fua-evo`. Este Experience API actúa como pass-through sin transformar la respuesta.
