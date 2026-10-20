---
api: "ntgy-fuv-p-customers"
sidebar_position: 12
sidebar_label: "GET /towns"
description: "Servicio no protegido para obtener listado de municipios filtrado por ID de municipalidad o código postal"
keywords: [towns, municipios, localidades, callejero, geografía, direcciones, Process API, orquestación]
tags: [Geographic Data, Callejero, Process API, Pass-Through]
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-fuv-s-callejero"
    path: "/towns"
    purpose: "Obtención de listado de municipios desde System API de Callejero"
---

# GET /towns

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint proporciona un servicio de consulta de municipios (towns/localidades) que permite obtener un listado filtrado por ID de municipalidad o código postal. Actúa como capa de orquestación que delega la consulta al System API de Callejero, facilitando el acceso a datos geográficos para formularios de direcciones y validaciones de ubicación.

### Tipo de API

**Capa:** Process API (API-Led Connectivity)

**Función:** Orquestación de consulta de datos geográficos - Municipios

### Patrón de Integración

**Tipo:** Pass-Through con Orquestación Simple

**Descripción:** El Process API recibe la petición, valida los parámetros de entrada y los reenvía directamente al System API S-Callejero sin transformación de datos. La respuesta del System API se devuelve tal cual al consumidor.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    E[Experience APIs<br/>Portales Web, Apps Móviles]
    P[Process API<br/>ntgy-fuv-p-customers<br/>GET /towns]
    S[System API<br/>ntgy-fuv-s-callejero<br/>GET /towns]
    DB[(Base de Datos<br/>Callejero)]
    
    E -->|Request<br/>municipalityId o postalCodeId| P
    P -->|Pass-through| S
    S -->|Query| DB
    DB -->|Datos de municipios| S
    S -->|Response| P
    P -->|Response| E
    
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:3px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style DB fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Formularios de Direcciones en Portales Web**
   - Los usuarios seleccionan provincia y municipalidad, luego necesitan elegir el municipio específico
   - Facilita la captura precisa de direcciones en procesos de alta de clientes
   - Mejora la experiencia de usuario con listas desplegables dinámicas

2. **Validación de Direcciones en Aplicaciones Móviles**
   - Las apps móviles validan que el municipio ingresado existe y corresponde a la provincia/código postal
   - Previene errores en la captura de datos de ubicación
   - Asegura consistencia en la información geográfica

3. **Autocompletado de Direcciones**
   - Sistemas de autocompletado utilizan este servicio para sugerir municipios válidos
   - Reduce tiempo de captura de datos
   - Minimiza errores tipográficos en nombres de municipios

### 🔗 Endpoints Dependientes

Este Process API consume el siguiente System API:

| System API | Endpoint | Método | Propósito |
|------------|----------|--------|-----------|
| `ntgy-fuv-s-callejero` | `/towns` | GET | Consulta de municipios desde base de datos de callejero |

### Lógica de Orquestación

```mermaid
graph LR
    P[Process API<br/>p-customers] --> S[System API<br/>s-callejero<br/>/towns]
    S --> DB[(Base de Datos<br/>Callejero)]
    
    style P fill:#ffe1f5
    style S fill:#e8f5e9
    style DB fill:#e3f2fd
```

**Pasos del Proceso:**

1. **Recepción de Request**: Process API recibe petición con `municipalityId` o `postalCodeId`
2. **Validación de Parámetros**: Verifica que al menos uno de los parámetros opcionales esté presente
3. **Delegación a System API**: Reenvía los query parameters al System API S-Callejero
4. **Respuesta Directa**: Devuelve la respuesta del System API sin transformación

### Transformaciones Aplicadas

⚠️ **Este Process API NO transforma la respuesta del System API.**

Los query parameters se pasan directamente mediante mapeo 1:1:
- `municipalityId` → `municipalityId`
- `postalCodeId` → `postalCodeId`

Para el schema completo de la respuesta, consultar:
📖 **System API: `ntgy-fuv-s-callejero` endpoint `/towns`**

### Contexto Empresarial

- **Dominio:** Gestión de Datos Geográficos
- **Sector:** Energía - Utilities
- **División:** Naturgy España - FUV (Fuerza de Ventas)
- **Proceso de Negocio:** Captura y validación de direcciones de clientes

### Stakeholders

- **Desarrolladores de Portales Web**: Necesitan datos de municipios para formularios de direcciones
- **Equipos de Aplicaciones Móviles**: Requieren validación de ubicaciones en apps
- **Agentes de Call Center**: Utilizan sistemas que consultan municipios para registrar direcciones
- **Usuarios Finales**: Clientes que ingresan sus direcciones en portales de autoservicio

### Valor de Negocio

- **Calidad de Datos**: Asegura que las direcciones capturadas sean válidas y consistentes
- **Experiencia de Usuario**: Facilita la captura de direcciones con listas desplegables dinámicas
- **Eficiencia Operativa**: Reduce errores en direcciones que requieren corrección posterior
- **Cumplimiento Normativo**: Mantiene datos de ubicación precisos para cumplimiento regulatorio

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Portales web de autoservicio (alta de clientes, cambio de dirección)
- Aplicaciones móviles de Naturgy
- Sistemas de agentes de call center
- Herramientas internas de validación de datos
- Integraciones con sistemas de terceros autorizados

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/towns` |
| **Development** | `https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io` | `https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/towns` |
| **UAT** | `https://ntgy-fuv-p-customers-uat.us-e2.cloudhub.io` | `https://ntgy-fuv-p-customers-uat.us-e2.cloudhub.io/api/towns` |
| **Production** | `https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io` | `https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io/api/towns` |

### Método HTTP

```
GET /api/towns
```

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `municipalityId` | string | No* | ID del municipio para filtrar localidades | `01003` |
| `postalCodeId` | string | No* | Código postal para filtrar localidades | `08924` |

**Nota:** Al menos uno de los dos parámetros debe ser proporcionado.

#### Headers

| Header | Tipo | Requerido | Descripción |
|--------|------|-----------|-------------|
| `Content-Type` | string | Sí | Debe ser `application/json` |

**⚠️ Autenticación:** Este es un servicio **NO protegido** - no requiere autenticación.

### Respuestas

#### 200 OK - Consulta Exitosa

**Descripción:** Listado de municipios obtenido correctamente.

**Estructura de Respuesta:**

```json
{
  "items": [
    {
      "townId": "string",
      "townName": "string"
    }
  ]
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `items` | array | Array de objetos con información de municipios |
| `items[].townId` | string | Identificador único del municipio |
| `items[].townName` | string | Nombre del municipio |

#### 400 Bad Request - Error en Parámetros

**Descripción:** Parámetros de entrada inválidos o faltantes.

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

#### 500 Internal Server Error - Error del Sistema

**Descripción:** Error interno del servidor o del System API.

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

### Códigos de Estado HTTP

| Código | Descripción | Escenario |
|--------|-------------|-----------|
| `200` | OK | Consulta exitosa, municipios encontrados |
| `400` | Bad Request | Parámetros inválidos o faltantes |
| `500` | Internal Server Error | Error en System API o error técnico |
| `503` | Service Unavailable | System API no disponible |

### Timeouts

| Tipo | Valor |
|------|-------|
| **Request Timeout** | 30 segundos |
| **Connection Timeout** | 10 segundos |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request]) --> LogEntrada[Log Entrada<br/>logEntrada get:\towns]
    LogEntrada --> BuildParams[Build Query Params<br/>municipalityId, postalCodeId]
    BuildParams --> HTTPRequest[HTTP Request<br/>GET S-Callejero /towns]
    HTTPRequest --> LogSalida[Log Salida<br/>logSalida get:\towns]
    LogSalida --> End([HTTP Response 200])
    
    HTTPRequest -->|Error| ErrorHandler[Error Handler]
    ErrorHandler --> LogError[Log Error<br/>logErrorGenerico]
    LogError --> TransformError[Transform Error<br/>codResult: -0001]
    TransformError --> ErrorResponse([HTTP Response 400])
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style End fill:#e8f5e9,stroke:#2e7d32
    style ErrorResponse fill:#ffebee,stroke:#c62828
    style HTTPRequest fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
```

### Código de Implementación

**Flow Principal:** `p-customers-townsFlow`

**Ubicación:** `src/main/mule/implementation/p-customers-townsFlow.xml`

**Componentes Clave:**

1. **Logger de Entrada** (línea 8):
   ```xml
   <logger level="INFO" doc:name="logEntrada" 
           message="logEntrada get:\towns"/>
   ```

2. **HTTP Request a System API** (línea 9-17):
   ```xml
   <http:request method="GET" doc:name="Request" 
                 path="${s-callejero.path.towns}" 
                 config-ref="S-Callejero_HTTP_Request_configuration">
       <http:query-params><![CDATA[#[output application/java
       ---
       {
           municipalityId : attributes.queryParams["municipalityId"],
           postalCodeId : attributes.queryParams["postalCodeId"]
       }]]]></http:query-params>
   </http:request>
   ```

3. **Logger de Salida** (línea 18):
   ```xml
   <logger level="INFO" doc:name="logSalida" 
           message="logSalida get:\towns"/>
   ```

### Transformaciones DataWeave

**Transformación de Query Parameters:**

```dataweave
%dw 2.0
output application/java
---
{
    municipalityId : attributes.queryParams["municipalityId"],
    postalCodeId : attributes.queryParams["postalCodeId"]
}
```

**Descripción:** Extrae los query parameters del request HTTP y los mapea 1:1 para el System API.

### Configuración HTTP

**Config Reference:** `S-Callejero_HTTP_Request_configuration`

**Path Dinámico:** `${s-callejero.path.towns}`

**Valor del Path:** `/towns` (configurado en properties)

### Patrones de Diseño

**Patrón:** Pass-Through con Logging

**Justificación:** 
- Los datos geográficos no requieren transformación
- El System API ya proporciona el formato correcto
- Se mantiene trazabilidad con logs de entrada/salida
- Minimiza latencia al evitar transformaciones innecesarias

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant Client as Cliente<br/>(Portal/App)
    participant P as Process API<br/>p-customers
    participant S as System API<br/>s-callejero
    participant DB as Base de Datos<br/>Callejero
    
    Client->>P: GET /towns?municipalityId=01003
    Note over P: Log: logEntrada get:\towns
    P->>S: GET /towns?municipalityId=01003
    S->>DB: Query municipios
    DB-->>S: Resultados
    S-->>P: 200 OK + Lista de municipios
    Note over P: Log: logSalida get:\towns
    P-->>Client: 200 OK + Lista de municipios
```

#### 2. Flujo con Error de Conectividad

```mermaid
sequenceDiagram
    participant Client as Cliente<br/>(Portal/App)
    participant P as Process API<br/>p-customers
    participant S as System API<br/>s-callejero
    
    Client->>P: GET /towns?postalCodeId=08924
    Note over P: Log: logEntrada get:\towns
    P->>S: GET /towns?postalCodeId=08924
    S--xP: HTTP:CONNECTIVITY Error
    Note over P: Error Handler activado
    Note over P: Log: logErrorGenerico
    Note over P: Transform Error Response
    P-->>Client: 400 Bad Request<br/>codResult: -0001
```

#### 3. Flujo con Timeout

```mermaid
sequenceDiagram
    participant Client as Cliente<br/>(Portal/App)
    participant P as Process API<br/>p-customers
    participant S as System API<br/>s-callejero
    
    Client->>P: GET /towns?municipalityId=01003
    Note over P: Log: logEntrada get:\towns
    P->>S: GET /towns?municipalityId=01003
    Note over S: Procesamiento lento...
    Note over P: Timeout (30s)
    S--xP: HTTP:TIMEOUT
    Note over P: Error Handler activado
    Note over P: Log: logErrorGenerico
    P-->>Client: 400 Bad Request<br/>codResult: -0001
```

### Manejo de Errores

**Error Handler Global** (líneas 19-36):

**Tipos de Errores Capturados:**
- `HTTP:BAD_GATEWAY`
- `HTTP:BAD_REQUEST`
- `HTTP:CLIENT_SECURITY`
- `HTTP:CONNECTIVITY`
- `HTTP:FORBIDDEN`
- `HTTP:INTERNAL_SERVER_ERROR`
- `HTTP:METHOD_NOT_ALLOWED`
- `HTTP:NOT_ACCEPTABLE`
- `HTTP:NOT_FOUND`
- `HTTP:PARSING`
- `HTTP:RETRY_EXHAUSTED`
- `HTTP:SECURITY`
- `HTTP:SERVICE_UNAVAILABLE`
- `HTTP:TIMEOUT`
- `HTTP:TOO_MANY_REQUESTS`
- `HTTP:UNAUTHORIZED`
- `HTTP:UNSUPPORTED_MEDIA_TYPE`
- `EXPRESSION`
- `STREAM_MAXIMUM_SIZE_EXCEEDED`

**Estrategia de Manejo:**

1. **Logging del Error:**
   ```xml
   <logger level="ERROR" doc:name="logErrorGenerico" 
           message="#['logErrorGenerico get:\towns Error Description: ' ++ 
                     (error.description default 'null' as String) ++ 
                     'Error Response: ' ++ 
                     if (!isEmpty(error.errorMessage.payload)) 
                       (write(error.errorMessage.payload, 'application/json') as String) 
                     else 'null']" />
   ```

2. **Transformación de Error Estándar:**
   ```dataweave
   %dw 2.0
   output application/json
   ---
   {
       "result": {
           "codResult": "-0001",
           "msgResult": "Error técnico"
       }
   }
   ```

3. **HTTP Status Code:** 400 (Bad Request)

**Política de Reintentos:** No implementada (delegada al System API)

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta por Municipality ID

#### cURL

```bash
curl -X GET "https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/towns?municipalityId=01003" \
  -H "Content-Type: application/json"
```

#### Respuesta (200 OK)

```json
{
  "items": [
    {
      "townId": "01003000000",
      "townName": "ARAMAIO"
    },
    {
      "townId": "01003000100",
      "townName": "AREXOLA"
    }
  ]
}
```

### Ejemplo 2: Consulta por Código Postal

#### cURL

```bash
curl -X GET "https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/towns?postalCodeId=08924" \
  -H "Content-Type: application/json"
```

#### Respuesta (200 OK)

```json
{
  "items": [
    {
      "townId": "08924000000",
      "townName": "SANTA COLOMA DE GRAMENET"
    }
  ]
}
```

### Ejemplo 3: JavaScript (Fetch API)

```javascript
// Consulta por municipality ID
const getTownsByMunicipality = async (municipalityId) => {
  try {
    const response = await fetch(
      `https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/towns?municipalityId=${municipalityId}`,
      {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json'
        }
      }
    );
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    console.log('Towns:', data.items);
    return data.items;
  } catch (error) {
    console.error('Error fetching towns:', error);
    throw error;
  }
};

// Uso
getTownsByMunicipality('01003');
```

### Ejemplo 4: Python (Requests)

```python
import requests

def get_towns_by_postal_code(postal_code_id):
    """
    Obtiene municipios por código postal
    
    Args:
        postal_code_id (str): Código postal
        
    Returns:
        list: Lista de municipios
    """
    url = "https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/towns"
    
    params = {
        'postalCodeId': postal_code_id
    }
    
    headers = {
        'Content-Type': 'application/json'
    }
    
    try:
        response = requests.get(url, params=params, headers=headers, timeout=30)
        response.raise_for_status()
        
        data = response.json()
        return data.get('items', [])
        
    except requests.exceptions.RequestException as e:
        print(f"Error al consultar municipios: {e}")
        raise

# Uso
towns = get_towns_by_postal_code('08924')
for town in towns:
    print(f"ID: {town['townId']}, Nombre: {town['townName']}")
```

### Ejemplo 5: Error - Parámetros Faltantes

#### cURL

```bash
curl -X GET "https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/towns" \
  -H "Content-Type: application/json"
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

### Ejemplo 6: Error - System API No Disponible

#### cURL

```bash
curl -X GET "https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/towns?municipalityId=01003" \
  -H "Content-Type: application/json"
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

**Nota:** En caso de error de conectividad con el System API, el Process API devuelve un error genérico para no exponer detalles internos de la arquitectura.