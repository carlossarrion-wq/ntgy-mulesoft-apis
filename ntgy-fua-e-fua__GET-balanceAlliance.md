---
api: "ntgy-fua-e-fua"
sidebar_position: 7
sidebar_label: "GET /balanceAlliance"
description: "Consulta el saldo disponible del cliente en el programa de alianzas Moeve para acumulación de puntos por consumo energético"
keywords: [alianzas, moeve, saldo, balance, puntos, fidelización, gow]
tags: [Alianzas, Moeve, Consulta, Experience API, Web]
channel: web
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/balanceAlliance"
    purpose: "Orquesta la consulta de saldo con validación de cliente y manejo de errores específicos"
---

# GET /balanceAlliance

## 📌 Descripción General

### Propósito del Endpoint

Consulta el saldo disponible de un cliente en el programa de alianzas Moeve (anteriormente Repsol), permitiendo a los clientes de Naturgy visualizar los puntos o euros acumulados por su consumo energético. Este endpoint forma parte del ecosistema de fidelización que incentiva el consumo responsable y la permanencia del cliente.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para canal web del portal FUA (Frontal Único de Atención)

### Patrón de Integración

**Tipo:** Pass-Through con Propagación de Headers

**Descripción:** Este Experience API actúa como pass-through, propagando la petición al Process API sin transformar la respuesta. Añade headers específicos del contexto FUA (`vars.fuaHeaders`) para trazabilidad y seguridad.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Portal FUA Web<br/>Cliente Naturgy]
    E[Experience API<br/>ntgy-fua-e-fua<br/>GET /balanceAlliance]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>GET /balanceAlliance]
    S[System API<br/>ntgy-newco-s-alianzas<br/>GET /saldoAlianza]
    B[Backend REST<br/>Moeve API<br/>GET /customer/balance]
    
    C -->|idGow| E
    E -->|idGow + fuaHeaders| P
    P -->|customer_id| S
    S -->|customer_id + Bearer Token| B
    B -->|Saldo JSON| S
    S -->|Saldo JSON| P
    P -->|Saldo JSON| E
    E -->|Saldo JSON| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style B fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Consulta de Saldo por Cliente**
   - Cliente accede al portal FUA y consulta su saldo de alianza Moeve
   - Visualiza euros o puntos acumulados por consumo energético
   - Toma decisiones sobre canje o acumulación de puntos

2. **Verificación por Agente de Atención**
   - Agente de call center consulta saldo durante atención telefónica
   - Proporciona información actualizada al cliente
   - Resuelve dudas sobre acumulación de puntos

3. **Validación de Elegibilidad**
   - Sistema verifica si el cliente está registrado en el programa
   - Identifica clientes sin saldo o no registrados
   - Facilita proceso de alta en programa de alianzas

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web

**Características del Canal:**
- **Plataforma**: Portal FUA (Frontal Único de Atención) web responsive
- **Dispositivos**: Desktop, tablet, mobile browser
- **Conectividad**: Requiere conexión estable para consulta en tiempo real
- **Experiencia de Usuario**: Consulta rápida con respuesta en menos de 3 segundos

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Naturgy Residenciales**: Consultan su saldo de alianza para gestión personal
- **Clientes Naturgy Empresariales**: Verifican acumulación de puntos por consumo corporativo
- **Agentes de Atención al Cliente**: Consultan saldo durante interacciones de soporte

**Contexto de Uso:**
- **Autoservicio Web**: Cliente accede desde área personal del portal FUA
- **Atención Telefónica**: Agente consulta durante llamada de soporte
- **Gestión de Fidelización**: Cliente planifica canje de puntos acumulados

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Pass-Through Completo**: No se simplifica la respuesta, se entrega tal cual del Process API
- **Headers de Contexto**: Se añaden headers FUA para trazabilidad (`vars.fuaHeaders`)
- **Timeout Extendido**: 60 segundos para garantizar respuesta del backend externo
- **Propagación de Query Params**: Se mantienen todos los parámetros del cliente

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/balanceAlliance` | GET | Orquesta la consulta de saldo con validación de cliente y transformación de parámetros (idGow → customer_id) |

**Flujo de Dependencias Completo:**

```
Experience API (ntgy-fua-e-fua)
    ↓
Process API (ntgy-newco-p-fua-evo)
    ↓
System API (ntgy-newco-s-alianzas)
    ↓
Backend REST (Moeve API)
```

### Contexto Empresarial

- **Dominio:** Fidelización y Alianzas Comerciales
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Web (Portal FUA)
- **Programa:** Alianza Moeve (anteriormente Repsol)

### Stakeholders

- **Clientes Naturgy**: Necesitan transparencia sobre su saldo de alianza para tomar decisiones de canje
- **Equipo de Marketing**: Monitorea adopción del programa de fidelización
- **Agentes de Call Center**: Requieren información actualizada para atención al cliente
- **Moeve (Partner Externo)**: Proveedor del programa de alianzas y backend de saldos

### Valor de Negocio

- **Transparencia**: Cliente visualiza en tiempo real su saldo acumulado
- **Fidelización**: Incentiva permanencia del cliente mediante programa de puntos
- **Autoservicio**: Reduce carga de call center al permitir consulta autónoma
- **Integración Comercial**: Fortalece alianza estratégica con Moeve
- **Experiencia Omnicanal**: Información consistente en web, móvil y atención telefónica

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/balanceAlliance` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/balanceAlliance` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceAlliance` |

### Método HTTP

```
GET /fua/api/balanceAlliance
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Permite tiempo suficiente para consulta en backend externo Moeve |
| **Rate Limiting** | Según política API Manager | Protección contra abuso y garantía de disponibilidad |
| **Autenticación** | OAuth 2.0 requerido | Seguridad y trazabilidad de accesos |

#### Campos Específicos del Canal

**Headers Añadidos por el Experience API:**

| Header | Tipo | Descripción | Razón |
|--------|------|-------------|-------|
| `vars.fuaHeaders` | Object | Headers de contexto FUA (trazabilidad, seguridad) | Propagación de contexto de sesión y auditoría |

**Campos Propagados sin Modificación:**

Este endpoint es **pass-through completo**. Todos los campos de respuesta del Process API se entregan sin transformación al cliente.

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Obligatorio | Descripción | Ejemplo |
|-----------|------|-------------|-------------|---------|
| `idGow` | String | ✅ Sí | Identificador único del cliente en el sistema Moeve (anteriormente Repsol). Este ID vincula al cliente Naturgy con su cuenta en el programa de alianzas. | `"12345678"` |

**Validaciones:**
- `idGow` debe ser un string no vacío
- El formato es validado por el Process API

**Transformación en Capas:**
- **Experience API**: Recibe `idGow` del cliente
- **Process API**: Transforma `idGow` → `customer_id`
- **System API**: Envía `customer_id` al backend Moeve

### Respuestas

#### Respuesta Exitosa - Cliente con Saldo (200 OK)

```json
{
  "balance": 123.45,
  "currency": "EUR",
  "customer_id": "12345678"
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `balance` | Number | Saldo disponible en la cuenta de alianza del cliente |
| `currency` | String | Moneda del saldo (típicamente "EUR" para España) |
| `customer_id` | String | Identificador del cliente en sistema Moeve |

#### Respuesta - Cliente No Registrado (200 OK)

Cuando el cliente no existe en el sistema Moeve, el Process API devuelve una respuesta controlada:

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No existe cliente"
  }
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | String | Código "0000" indica cliente no encontrado (no es error técnico) |
| `result.msgResult` | String | Mensaje descriptivo del resultado |

**Nota:** Este caso NO es un error HTTP 404. Es una respuesta exitosa (200 OK) que indica que el cliente no está registrado en el programa de alianzas.

#### Respuesta de Error - Error Técnico (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al consultar saldo en sistema Moeve"
  }
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | String | Código "0001" indica error técnico |
| `result.msgResult` | String | Descripción del error técnico |

#### Códigos de Estado HTTP

| Código | Descripción | Cuándo Ocurre |
|--------|-------------|---------------|
| `200` | OK | Consulta exitosa (con o sin saldo) |
| `400` | Bad Request | Parámetros inválidos o faltantes |
| `401` | Unauthorized | Token de autenticación inválido o expirado |
| `403` | Forbidden | Usuario sin permisos para consultar saldo |
| `404` | Not Found | Endpoint no encontrado |
| `500` | Internal Server Error | Error técnico en backend Moeve o APIs intermedias |

### Autenticación

- **Tipo:** OAuth 2.0 con Bearer Token
- **Header:** `Authorization: Bearer {token}`
- **Scope:** Acceso a información de alianzas del cliente autenticado
- **Validación:** Realizada por API Manager antes de llegar al Experience API

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Experience API → Process API** | 60 segundos | Permite tiempo suficiente para consulta en backend externo |
| **Process API → System API** | Según configuración System API | Propagado desde configuración |
| **System API → Backend Moeve** | 30 segundos | Timeout del backend externo |

**Configuración en Código:**

```yaml
# e-fua-pro-config.yaml
newco-p-fua-evo:
  timeout: "60000"  # 60 segundos
```

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request<br/>Portal FUA]) --> Flow[Flow: get-alianza-balanceAlliance]
    Flow --> AddHeaders[Añadir vars.fuaHeaders]
    AddHeaders --> CallProcess[HTTP Request a Process API<br/>ntgy-newco-p-fua-evo]
    CallProcess --> Success{¿Éxito?}
    Success -->|Sí| PassThrough[Pass-Through Response]
    Success -->|No| ErrorHandler[Error Handler]
    ErrorHandler --> MapError[Mapear Código HTTP]
    MapError --> ErrorResponse[Respuesta de Error]
    PassThrough --> End([HTTP Response])
    ErrorResponse --> End
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style Flow fill:#fff4e1,stroke:#f57c00
    style CallProcess fill:#ffe1f5,stroke:#c2185b
```

### Código del Flow Principal

**Archivo:** `alliances.xml` (líneas 94-134)

```xml
<flow name="get-alianza-balanceAlliance" doc:id="bbc82609-1a1b-422b-8d2f-3ba4a523dba6">
    <http:request 
        method="GET" 
        doc:name="ntgy-newco-p-fua-evo" 
        config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
        path="/balanceAlliance" 
        responseTimeout="${newco-p-fua-evo.timeout}">
        <http:query-params>
            <![CDATA[#[output application/java
            ---
            attributes.queryParams ++ vars.fuaHeaders]]]>
        </http:query-params>
    </http:request>
    
    <error-handler>
        <on-error-propagate enableNotifications="true" logException="true">
            <ee:transform doc:name="error message">
                <ee:message>
                    <ee:set-payload>
                        <![CDATA[%dw 2.0
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
                        }]]>
                    </ee:set-payload>
                </ee:message>
                <ee:variables>
                    <ee:set-variable variableName="httpStatus">
                        <![CDATA[%dw 2.0
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
                            500]]>
                    </ee:set-variable>
                </ee:variables>
            </ee:transform>
        </on-error-propagate>
    </error-handler>
</flow>
```

### Transformaciones de Canal

**Este Experience API NO transforma la respuesta del Process API.**

⚠️ **Pass-Through Completo**: La respuesta del Process API se entrega sin modificaciones al cliente.

**Única Transformación Aplicada:**

| Elemento | Transformación | Propósito |
|----------|----------------|-----------|
| Query Params | `attributes.queryParams ++ vars.fuaHeaders` | Añade headers de contexto FUA para trazabilidad |

**Flujo de Respuesta (sin transformación):**

```
Backend Moeve (respuesta original)
    ↓ (sin transformación)
System API: extrae payload.response
    ↓ (sin transformación)
Process API: pass-through
    ↓ (sin transformación)
Experience API: pass-through
    ↓
Cliente (recibe respuesta original de Moeve)
```

**⚠️ IMPORTANTE - Estructura de Respuesta:**

La estructura de respuesta exitosa **NO está definida en el código MuleSoft** porque:
1. El Experience API hace pass-through completo (líneas 94-134 de `alliances.xml`)
2. El Process API hace pass-through en caso exitoso (líneas 73-78 de `alianzaBalanceAlliance.xml`)
3. El System API solo extrae `payload.response` del backend (línea 60 de `alianzaSaldo.xml`)
4. La estructura final depende del contrato del backend Moeve

**Para conocer la estructura exacta:**
- 📖 Consultar documentación del backend Moeve API
- 📖 Consultar documentación del System API: `ntgy-newco-s-alianzas` endpoint `/saldoAlianza`
- 🔍 Realizar pruebas en ambiente de desarrollo para ver respuesta real

**Estructuras SÍ definidas en código (errores):**
- ✅ Cliente no registrado: `{result: {codResult: "0000", msgResult: "No existe cliente"}}`
- ✅ Error técnico: `{result: {codResult: "0001", msgResult: "..."}}`

### Patrones de Diseño

**Patrón:** API Gateway con Pass-Through

**Justificación:** 
- El Experience API actúa como punto de entrada único para el canal web
- No requiere transformación de datos específica del canal
- Añade contexto de seguridad y trazabilidad mediante headers FUA
- Simplifica la arquitectura al delegar lógica de negocio al Process API

**Ventajas:**
- Menor latencia al evitar transformaciones innecesarias
- Facilita mantenimiento al centralizar lógica en Process API
- Permite evolución independiente de cada capa

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito - Cliente con Saldo

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant S as System API<br/>ntgy-newco-s-alianzas
    participant M as Backend Moeve

    C->>E: GET /balanceAlliance?idGow=12345
    Note over E: Añade vars.fuaHeaders
    E->>P: GET /balanceAlliance?idGow=12345<br/>+ fuaHeaders
    Note over P: Transforma idGow → customer_id
    P->>S: GET /saldoAlianza?customer_id=12345
    Note over S: Genera Bearer Token
    S->>M: GET /customer/balance?customer_id=12345<br/>Authorization: Bearer {token}
    M-->>S: 200 OK<br/>{balance: 123.45, currency: "EUR"}
    S-->>P: 200 OK<br/>{balance: 123.45, currency: "EUR"}
    P-->>E: 200 OK<br/>{balance: 123.45, currency: "EUR"}
    E-->>C: 200 OK<br/>{balance: 123.45, currency: "EUR"}
```

#### 2. Flujo - Cliente No Registrado

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant S as System API<br/>ntgy-newco-s-alianzas
    participant M as Backend Moeve

    C->>E: GET /balanceAlliance?idGow=99999
    E->>P: GET /balanceAlliance?idGow=99999
    P->>S: GET /saldoAlianza?customer_id=99999
    S->>M: GET /customer/balance?customer_id=99999
    M-->>S: 404 Not Found<br/>"Cannot identify the customer"
    Note over S: Error del backend
    S-->>P: Error: "Cannot identify the customer"
    Note over P: Detecta error específico<br/>Transforma a respuesta controlada
    P-->>E: 200 OK<br/>{result: {codResult: "0000",<br/>msgResult: "No existe cliente"}}
    E-->>C: 200 OK<br/>{result: {codResult: "0000",<br/>msgResult: "No existe cliente"}}
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant C as Cliente Web
    participant E as Experience API<br/>ntgy-fua-e-fua
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    participant S as System API<br/>ntgy-newco-s-alianzas
    participant M as Backend Moeve

    C->>E: GET /balanceAlliance?idGow=12345
    E->>P: GET /balanceAlliance?idGow=12345
    P->>S: GET /saldoAlianza?customer_id=12345
    S->>M: GET /customer/balance?customer_id=12345
    M-->>S: 500 Internal Server Error
    S-->>P: Error: "Error backend"
    Note over P: Maneja error técnico
    P-->>E: 500 Internal Server Error<br/>{result: {codResult: "0001",<br/>msgResult: "Error consulta saldo"}}
    Note over E: Error Handler<br/>Mapea código HTTP
    E-->>C: 500 Internal Server Error<br/>{result: {codResult: "0001",<br/>msgResult: "Error consulta saldo"}}
```

### Lógica de Negocio en Process API

El Process API (`ntgy-newco-p-fua-evo`) implementa lógica de negocio específica:

**1. Transformación de Parámetros:**
```dataweave
{
    "customer_id": attributes.queryParams.'idGow'
}
```

**2. Manejo Especial de "Cliente No Existe":**
```dataweave
// Si el error contiene "identify the customer"
when expression='#[error.errorMessage.payload.result.msgResult contains("identify the customer")]'
// Entonces responde con código 0000 (no es error técnico)
{
    "result": {
        "codResult": "0000",
        "msgResult": "No existe cliente"
    }
}
```

**3. Manejo de Errores Técnicos:**
```dataweave
// Para cualquier otro error
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

### Manejo de Errores

#### Estrategia de Error Handling

**Nivel 1 - Experience API:**
- Captura errores del Process API
- Mapea códigos HTTP según contenido del error
- Propaga error con estructura estándar

**Nivel 2 - Process API:**
- Distingue entre "cliente no existe" (negocio) y errores técnicos
- Transforma error de "cliente no existe" a respuesta exitosa con código 0000
- Propaga errores técnicos con código 0001

**Nivel 3 - System API:**
- Captura errores del backend Moeve
- Transforma errores a formato estándar
- Propaga con información del error original

#### Mapeo de Códigos de Error

| Código Result | Tipo | Descripción | HTTP Status |
|---------------|------|-------------|-------------|
| `0000` | Negocio | Cliente no registrado en programa | 200 OK |
| `0001` | Técnico | Error en consulta al backend | 500 |
| `400` | Validación | Parámetros inválidos | 400 |
| `401` | Autenticación | Token inválido | 401 |
| `403` | Autorización | Sin permisos | 403 |
| `404` | No encontrado | Endpoint no existe | 404 |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta Exitosa - Cliente con Saldo

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceAlliance?idGow=12345678" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID"
```

#### Respuesta (200 OK)

```json
{
  "balance": 123.45,
  "currency": "EUR",
  "customer_id": "12345678"
}
```

**Interpretación:**
- El cliente tiene un saldo de 123.45 euros en su cuenta Moeve
- El saldo está en euros (EUR)
- El customer_id confirma la identidad del cliente en el sistema

---

### Ejemplo 2: Cliente No Registrado en Programa

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceAlliance?idGow=99999999" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID"
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

**Interpretación:**
- El cliente no está registrado en el programa de alianzas Moeve
- No es un error técnico (código 0000)
- La aplicación puede ofrecer al cliente registrarse en el programa

---

### Ejemplo 3: Error Técnico - Backend No Disponible

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceAlliance?idGow=12345678" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID"
```

#### Respuesta (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al consultar saldo en sistema Moeve"
  }
}
```

**Interpretación:**
- Error técnico en el backend Moeve o APIs intermedias
- La aplicación debe mostrar mensaje de error al usuario
- Reintentar la operación después de unos minutos

---

### Ejemplo 4: JavaScript (Fetch API para Web)

```javascript
async function consultarSaldoAlianza(idGow, token) {
  try {
    const response = await fetch(
      `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceAlliance?idGow=${idGow}`,
      {
        method: 'GET',
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json',
          'client_id': 'YOUR_CLIENT_ID'
        }
      }
    );

    const data = await response.json();

    if (response.ok) {
      if (data.balance !== undefined) {
        // Cliente con saldo
        console.log(`Saldo disponible: ${data.balance} ${data.currency}`);
        return {
          success: true,
          balance: data.balance,
          currency: data.currency
        };
      } else if (data.result && data.result.codResult === '0000') {
        // Cliente no registrado
        console.log('Cliente no registrado en programa de alianzas');
        return {
          success: true,
          notRegistered: true,
          message: data.result.msgResult
        };
      }
    } else {
      // Error técnico
      console.error('Error al consultar saldo:', data);
      return {
        success: false,
        error: data.result?.msgResult || 'Error desconocido'
      };
    }
  } catch (error) {
    console.error('Error de red:', error);
    return {
      success: false,
      error: 'Error de conexión'
    };
  }
}

// Uso del ejemplo
const resultado = await consultarSaldoAlianza('12345678', 'tu_token_aqui');
if (resultado.success) {
  if (resultado.notRegistered) {
    alert('No estás registrado en el programa de alianzas');
  } else {
    alert(`Tu saldo es: ${resultado.balance} ${resultado.currency}`);
  }
} else {
  alert(`Error: ${resultado.error}`);
}
```

---

### Ejemplo 5: Python (Requests)

```python
import requests

def consultar_saldo_alianza(id_gow, token):
    """
    Consulta el saldo de alianza Moeve de un cliente.
    
    Args:
        id_gow (str): Identificador del cliente en sistema Moeve
        token (str): Token de autenticación OAuth 2.0
        
    Returns:
        dict: Resultado de la consulta con saldo o error
    """
    url = "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/balanceAlliance"
    
    headers = {
        'Authorization': f'Bearer {token}',
        'Content-Type': 'application/json',
        'client_id': 'YOUR_CLIENT_ID'
    }
    
    params = {
        'idGow': id_gow
    }
    
    try:
        response = requests.get(url, headers=headers, params=params, timeout=65)
        data = response.json()
        
        if response.status_code == 200:
            if 'balance' in data:
                # Cliente con saldo
                return {
                    'success': True,
                    'balance': data['balance'],
                    'currency': data['currency'],
                    'customer_id': data['customer_id']
                }
            elif 'result' in data and data['result']['codResult'] == '0000':
                # Cliente no registrado
                return {
                    'success': True,
                    'not_registered': True,
                    'message': data['result']['msgResult']
                }
        else:
            # Error técnico
            return {
                'success': False,
                'error': data.get('result', {}).get('msgResult', 'Error desconocido'),
                'status_code': response.status_code
            }
            
    except requests.exceptions.Timeout:
        return {
            'success': False,
            'error': 'Timeout al consultar saldo'
        }
    except requests.exceptions.RequestException as e:
        return {
            'success': False,
            'error': f'Error de conexión: {str(e)}'
        }

# Uso del ejemplo
resultado = consultar_saldo_alianza('12345678', 'tu_token_aqui')

if resultado['success']:
    if resultado.get('not_registered'):
        print(f"Cliente no registrado: {resultado['message']}")
    else:
        print(f"Saldo: {resultado['balance']} {resultado['currency']}")
else:
    print(f"Error: {resultado['error']}")
```

---

### Ejemplo 6: Postman Collection

```json
{
  "info": {
    "name": "FUA - Consulta Saldo Alianza",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "GET Balance Alliance - Cliente con Saldo",
      "request": {
        "method": "GET",
        "header": [
          {
            "key": "Authorization",
            "value": "Bearer {{access_token}}",
            "type": "text"
          },
          {
            "key": "Content-Type",
            "value": "application/json",
            "type": "text"
          },
          {
            "key": "client_id",
            "value": "{{client_id}}",
            "type": "text"
          }
        ],
        "url": {
          "raw": "{{base_url}}/fua/api/balanceAlliance?idGow=12345678",
          "host": ["{{base_url}}"],
          "path": ["fua", "api", "balanceAlliance"],
          "query": [
            {
              "key": "idGow",
              "value": "12345678",
              "description": "ID del cliente en sistema Moeve"
            }
          ]
        }
      }
    }
  ]
}
```
