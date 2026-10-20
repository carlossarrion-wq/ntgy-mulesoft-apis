---
api: "ntgy-fua-s-ac-fua"
sidebar_position: 23
description: Consulta información de municipios y provincias por código postal desde la base de datos PostgreSQL
keywords:
  - home
  - municipios
  - provincias
  - código postal
  - PostgreSQL
  - consulta geográfica
  - System API
  - MUL-COM-FUA-S-AC-FUA
tags:
  - System API
  - PostgreSQL
  - Consulta
sql_metadata:
  data_sources:
    - database: "PostgreSQL"
      schema: "public"
      type: "PostgreSQL"
      tables:
        - name: "V_FUA_MUNICIP_BY_POSTAL_CODE"
          operations: ["READ"]
          description: "Vista que contiene información de municipios y provincias indexada por código postal"
---

# GET /home

## 📌 Descripción General

Endpoint de System API que consulta información geográfica de municipios y provincias a partir de un código postal. Realiza una consulta directa a la vista `V_FUA_MUNICIP_BY_POSTAL_CODE` en PostgreSQL para obtener datos de localización asociados al código postal proporcionado.

**Tipo de API:** System API  
**Protocolo:** REST/HTTP  
**Método:** GET  
**Backend:** PostgreSQL  
**Autenticación:** Requerida (según configuración del API)

### Características Principales

- ✅ Consulta directa a vista de PostgreSQL
- ✅ Búsqueda por código postal
- ✅ Respuesta estructurada con código de resultado
- ✅ Logging de entrada y salida
- ✅ Timeout configurable para la consulta

## 💼 Propósito del Negocio

### Objetivo

Proporcionar información geográfica precisa (municipio y provincia) basándose en el código postal ingresado por el usuario. Este endpoint es fundamental para:

- **Validación de direcciones:** Verificar que el código postal corresponde a un municipio válido
- **Autocompletado de formularios:** Rellenar automáticamente campos de municipio/provincia
- **Geolocalización:** Asociar códigos postales con ubicaciones geográficas
- **Validación de datos:** Asegurar la consistencia de información de direcciones

### Casos de Uso

1. **Registro de Clientes**
   - Usuario ingresa código postal
   - Sistema obtiene municipio y provincia automáticamente
   - Se valida la existencia del código postal

2. **Actualización de Direcciones**
   - Cliente modifica su dirección
   - Sistema verifica el código postal
   - Se actualizan los datos geográficos asociados

3. **Búsqueda Geográfica**
   - Aplicación necesita datos de localización
   - Consulta por código postal
   - Obtiene información completa del municipio

### Valor de Negocio

- **Mejora de UX:** Reduce errores en la captura de direcciones
- **Calidad de Datos:** Asegura consistencia en información geográfica
- **Eficiencia Operativa:** Automatiza la validación de códigos postales
- **Integración:** Facilita la integración con otros sistemas que requieren datos geográficos

## 🔧 Especificación Técnica

### Endpoint

```
GET /home?CP={codigoPostal}
```

### Parámetros de Consulta

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `CP` | string | Sí | Código postal a consultar | `28001` |

### Headers Requeridos

```http
Content-Type: application/json
Authorization: Bearer {token}
```

### Respuesta Exitosa (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  }
}
```

### Estructura de Respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result` | object | Objeto contenedor de la respuesta |
| `result.codResult` | string | Código de resultado de la operación |
| `result.msgResult` | string | Mensaje descriptivo del resultado |

### Códigos de Respuesta HTTP

| Código | Descripción | Escenario |
|--------|-------------|-----------|
| 200 | OK | Consulta ejecutada exitosamente |
| 400 | Bad Request | Parámetro CP faltante o inválido |
| 401 | Unauthorized | Token de autenticación inválido o ausente |
| 500 | Internal Server Error | Error en la consulta a base de datos |
| 504 | Gateway Timeout | Timeout en la consulta a PostgreSQL |

### Códigos de Resultado de Negocio

| Código | Mensaje | Descripción |
|--------|---------|-------------|
| `0000` | No hay error | Operación exitosa |

## 🏗️ Implementación Técnica

### Arquitectura del Flujo

```mermaid
sequenceDiagram
    participant Client as Cliente
    participant API as MuleSoft API
    participant Logger as Logger
    participant DB as PostgreSQL
    participant Transform as DataWeave

    Client->>API: GET /home?CP=28001
    API->>Logger: Log entrada (payload)
    Logger-->>API: Logged
    API->>API: Set Variable (limpiar queryParams)
    API->>DB: SELECT * FROM V_FUA_MUNICIP_BY_POSTAL_CODE<br/>WHERE POSTAL_CODE = :CP
    DB-->>API: Resultados de la consulta
    API->>Transform: Transform Message
    Transform->>Transform: Construir respuesta JSON
    Transform-->>API: Response estructurado
    API->>Logger: Log salida (payload)
    Logger-->>API: Logged
    API-->>Client: 200 OK + JSON Response
```

### Diagrama de Componentes

```mermaid
graph TB
    subgraph "MuleSoft Runtime"
        A[APIKit Router] --> B[get:\home Flow]
        B --> C[homeFlow]
        C --> D[Logger Entrada]
        D --> E[Set Variable]
        E --> F[DB Select]
        F --> G[Transform Message]
        G --> H[Logger Salida]
    end
    
    subgraph "PostgreSQL"
        I[(V_FUA_MUNICIP_BY_POSTAL_CODE)]
    end
    
    F -.->|Query SQL| I
    I -.->|Resultados| F
    
    style C fill:#e1f5ff
    style F fill:#ffe1e1
    style G fill:#e1ffe1
```

### Flujo de Datos

```mermaid
flowchart LR
    A[Request<br/>CP=28001] --> B{Validar<br/>Parámetros}
    B -->|Válido| C[Log Entrada]
    B -->|Inválido| Z[Error 400]
    C --> D[Limpiar<br/>QueryParams]
    D --> E[Ejecutar Query<br/>PostgreSQL]
    E -->|Success| F[Transform<br/>Response]
    E -->|Error| Y[Error 500]
    F --> G[Log Salida]
    G --> H[Response 200]
    
    style E fill:#ffcccc
    style F fill:#ccffcc
    style H fill:#ccccff
```

### Configuración de Base de Datos

**Archivo:** `src/main/resources/properties/ntgy-fua-s-ac-fua-global-config.yaml`

```yaml
dbAC:
  queryTimeout: 30000  # Timeout en milisegundos (30 segundos)
```

### Implementación del Flow Principal

**Archivo:** `src/main/mule/ntgy-fua-s-ac-fua.xml`

```xml
<flow name="get:\home:ntgy-fua-s-ac-fua-config">
    <logger level="DEBUG" doc:name="Logger" />
    <flow-ref doc:name="homeFlow" name="homeFlow" />
</flow>
```

### Implementación del homeFlow

**Archivo:** `src/main/mule/implementation/homeFlow.xml` (o similar)

```xml
<flow name="homeFlow">
    <!-- Logger de entrada -->
    <logger level="INFO" 
            doc:name="Logger entrada"
            message="s-ac-consumptions.logEntrada ## payload: #[payload]"/>
    
    <!-- Limpieza de parámetros de query -->
    <set-variable 
        value="" 
        doc:name="Set Variable"
        variableName='attributes.queryParams["**"]'/>
    
    <!-- Consulta a PostgreSQL -->
    <db:select 
        doc:name="Select"
        config-ref="Database_Config"
        queryTimeout="${dbAC.queryTimeout}">
        <db:sql><![CDATA[
            SELECT * 
            FROM V_FUA_MUNICIP_BY_POSTAL_CODE 
            WHERE POSTAL_CODE = :CP
        ]]></db:sql>
    </db:select>
    
    <!-- Transformación de respuesta -->
    <ee:transform doc:name="Transform Message">
        <ee:message>
            <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    result: {
        codResult: "0000",
        msgResult: "No hay error"
    }
}]]></ee:set-payload>
        </ee:message>
    </ee:transform>
    
    <!-- Logger de salida -->
    <logger level="INFO"
            doc:name="Logger salida"
            message="s-ac-consumptions.logSalida ## payload: #[payload]"/>
</flow>
```

### Query SQL

**Vista:** `V_FUA_MUNICIP_BY_POSTAL_CODE`

```sql
SELECT * 
FROM V_FUA_MUNICIP_BY_POSTAL_CODE 
WHERE POSTAL_CODE = :CP
```

**Parámetros:**
- `:CP` - Código postal proporcionado en el query parameter

**Nota:** La vista `V_FUA_MUNICIP_BY_POSTAL_CODE` debe contener al menos las siguientes columnas (estructura inferida):
- `POSTAL_CODE` - Código postal
- `MUNICIPALITY` - Nombre del municipio
- `PROVINCE` - Nombre de la provincia
- Otros campos geográficos relevantes

### Transformación DataWeave

La transformación actual es estática y retorna un objeto de resultado genérico. En una implementación completa, debería incluir los datos obtenidos de la base de datos:

```dataweave
%dw 2.0
output application/json
---
{
    result: {
        codResult: "0000",
        msgResult: "No hay error"
    },
    data: payload map {
        postalCode: $.POSTAL_CODE,
        municipality: $.MUNICIPALITY,
        province: $.PROVINCE
        // Otros campos según estructura de la vista
    }
}
```

### Manejo de Errores

El endpoint implementa manejo de errores a través de:

1. **Timeout de Query:** Configurado en `${dbAC.queryTimeout}` (30 segundos)
2. **Logging:** Captura de entrada y salida para trazabilidad
3. **Error Handling:** Gestión de errores de base de datos (implícito en MuleSoft)

### Consideraciones de Rendimiento

- **Timeout:** 30 segundos configurables
- **Índice:** La vista debe tener índice en `POSTAL_CODE` para búsquedas eficientes
- **Cache:** Considerar implementar cache para códigos postales frecuentes
- **Connection Pool:** Configurado en `Database_Config`

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta Exitosa

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/home?CP=28001" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (200 OK):**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  }
}
```

### Ejemplo 2: Código Postal de Barcelona

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/home?CP=08001" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (200 OK):**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  }
}
```

### Ejemplo 3: Código Postal de Valencia

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/home?CP=46001" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (200 OK):**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  }
}
```

### Ejemplo 4: Error - Parámetro Faltante

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/home" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (400 Bad Request):**

```json
{
  "error": {
    "code": "400",
    "message": "Bad Request - Missing required parameter: CP"
  }
}
```

### Ejemplo 5: Error - Token Inválido

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/home?CP=28001" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer invalid_token"
```

**Response (401 Unauthorized):**

```json
{
  "error": {
    "code": "401",
    "message": "Unauthorized - Invalid or expired token"
  }
}
```

### Ejemplo 6: Integración con JavaScript

```javascript
async function consultarMunicipioPorCP(codigoPostal) {
  const url = `https://api.naturgy.com/fua/s-ac/v1/home?CP=${codigoPostal}`;
  
  try {
    const response = await fetch(url, {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${getAuthToken()}`
      }
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    
    if (data.result.codResult === "0000") {
      console.log("Consulta exitosa:", data);
      return data;
    } else {
      console.error("Error en la consulta:", data.result.msgResult);
      return null;
    }
  } catch (error) {
    console.error("Error al consultar municipio:", error);
    throw error;
  }
}

// Uso
consultarMunicipioPorCP("28001")
  .then(resultado => {
    console.log("Municipio encontrado:", resultado);
  })
  .catch(error => {
    console.error("Error:", error);
  });
```

### Ejemplo 7: Integración con Python

```python
import requests
import json

def consultar_municipio_por_cp(codigo_postal, token):
    """
    Consulta información de municipio por código postal
    
    Args:
        codigo_postal (str): Código postal a consultar
        token (str): Token de autenticación
        
    Returns:
        dict: Respuesta del API o None si hay error
    """
    url = f"https://api.naturgy.com/fua/s-ac/v1/home"
    
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {token}"
    }
    
    params = {
        "CP": codigo_postal
    }
    
    try:
        response = requests.get(url, headers=headers, params=params, timeout=30)
        response.raise_for_status()
        
        data = response.json()
        
        if data.get("result", {}).get("codResult") == "0000":
            print(f"Consulta exitosa para CP: {codigo_postal}")
            return data
        else:
            print(f"Error en consulta: {data.get('result', {}).get('msgResult')}")
            return None
            
    except requests.exceptions.RequestException as e:
        print(f"Error en la petición: {e}")
        return None

# Uso
token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
resultado = consultar_municipio_por_cp("28001", token)

if resultado:
    print(json.dumps(resultado, indent=2))
```

### Ejemplo 8: Testing con Postman

**Pre-request Script:**

```javascript
// Generar timestamp para logging
pm.environment.set("timestamp", new Date().toISOString());

// Validar que existe el código postal
if (!pm.request.url.query.get("CP")) {
    console.error("Falta el parámetro CP");
}
```
