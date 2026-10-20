---
api: "ntgy-fua-p-imp-customer-queries"
sidebar_position: 10
sidebar_label: "GET /documentsByCase"
description: "Obtiene un archivo ZIP con los documentos adjuntos a una gestión mediante delegación al System API AC"
keywords:
  - documentsByCase
  - documentos
  - gestión
  - case management
  - ZIP
  - archivos adjuntos
  - System API AC
  - pass-through
tags:
  - Case Management
  - Documents
  - Pass-Through
channel: "Process API"
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-fua-s-ac-fua"
    path: "/documentsByCase"
    purpose: "Obtención de documentos adjuntos a una gestión en formato ZIP"
---

# GET /documentsByCase

## 📌 Descripción General

### Tipo de API
**Process API** - Capa de Orquestación (API-Led Connectivity)

### Patrón de Integración
**Pass-Through (Delegación Directa)**

Este endpoint implementa un patrón de delegación pura al System API AC, actuando como proxy transparente sin aplicar transformaciones ni lógica de negocio adicional. Simplemente reenvía la petición y devuelve la respuesta tal cual.

### Arquitectura

```mermaid
graph LR
    A[Experience API<br/>Portal/Mobile/Agent] -->|GET /documentsByCase| B[Process API<br/>ntgy-fua-p-imp-customer-queries]
    B -->|GET /documentsByCase| C[System API AC<br/>ntgy-fua-s-ac-fua]
    C -->|ZIP File| B
    B -->|ZIP File| A
    
    style B fill:#e1f5ff
    style C fill:#fff4e1
```

### Propósito
Proporcionar acceso unificado a los documentos adjuntos a una gestión o caso de atención al cliente, delegando la operación al System API AC que gestiona el repositorio documental.

---

## 💼 Propósito del Negocio

### Problema que Resuelve

#### 1. Acceso a Documentación de Gestiones
**Contexto**: Los agentes de atención al cliente y los clientes finales necesitan acceder a los documentos adjuntos a sus gestiones (reclamaciones, solicitudes, incidencias).

**Problema**: Los documentos están almacenados en el sistema AC y requieren un mecanismo de descarga eficiente.

**Solución**: Este endpoint proporciona acceso directo a un archivo ZIP que contiene todos los documentos asociados a un caso específico.

#### 2. Consolidación de Documentos
**Contexto**: Una gestión puede tener múltiples documentos adjuntos (facturas, contratos, fotografías, justificantes).

**Problema**: Descargar documentos individualmente es ineficiente y genera múltiples llamadas.

**Solución**: El System API AC consolida todos los documentos en un único archivo ZIP, optimizando la descarga.

### Casos de Uso

#### Caso de Uso 1: Agente Consulta Documentos de Gestión
```
Actor: Agente de Atención al Cliente
Precondición: El agente tiene acceso al ID del caso
Flujo:
1. El agente busca el caso del cliente
2. Solicita los documentos adjuntos mediante caseId
3. El sistema descarga un ZIP con todos los documentos
4. El agente revisa los documentos para resolver la consulta
Postcondición: El agente tiene acceso a toda la documentación del caso
```

#### Caso de Uso 2: Cliente Descarga Documentos de su Gestión
```
Actor: Cliente Final
Precondición: El cliente tiene una gestión activa con documentos
Flujo:
1. El cliente accede a su portal de autoservicio
2. Navega a "Mis Gestiones"
3. Selecciona una gestión específica
4. Descarga los documentos adjuntos
5. El sistema proporciona un ZIP con todos los archivos
Postcondición: El cliente tiene los documentos de su gestión
```

#### Caso de Uso 3: Auditoría de Gestiones
```
Actor: Equipo de Auditoría/Calidad
Precondición: Se requiere revisar documentación de gestiones
Flujo:
1. El auditor identifica el caso a revisar
2. Solicita los documentos mediante caseId
3. Descarga el ZIP con toda la documentación
4. Revisa la completitud y calidad de los documentos
Postcondición: Auditoría completada con evidencia documental
```

### Stakeholders

#### Usuarios Directos
- **Agentes de Call Center**: Consultan documentos para resolver gestiones
- **Clientes Finales**: Descargan documentos de sus propias gestiones
- **Back Office**: Revisan documentación para procesamiento de gestiones

#### Usuarios Indirectos
- **Equipo de Calidad**: Auditan documentación de gestiones
- **Equipo Legal**: Acceden a documentación para casos legales
- **Supervisores**: Revisan gestiones escaladas

### Valor de Negocio

#### Beneficios Cuantitativos
- **Reducción de Tiempo**: Descarga consolidada vs múltiples descargas individuales
- **Eficiencia Operativa**: Menos llamadas al backend (1 vs N documentos)
- **Ancho de Banda**: Compresión ZIP reduce transferencia de datos

#### Beneficios Cualitativos
- **Experiencia de Usuario**: Descarga simple y rápida
- **Completitud**: Garantiza que se obtienen todos los documentos
- **Trazabilidad**: Registro de acceso a documentación de gestiones

---

## 🔧 Especificación Técnica

### Endpoint

#### URLs por Ambiente

| Ambiente | URL Completa |
|----------|--------------|
| **Local** | `http://localhost:8091/api/documentsByCase` |
| **Development** | `https://ntgy-fua-p-imp-customer-queries-dev.de-c1.cloudhub.io/api/documentsByCase` |
| **UAT** | `https://ntgy-fua-p-imp-customer-queries-uat.de-c1.cloudhub.io/api/documentsByCase` |
| **Production** | `https://ntgy-fua-p-imp-customer-queries-pro.de-c1.cloudhub.io/api/documentsByCase` |

#### Método HTTP
```
GET /api/documentsByCase
```

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `caseId` | string | ✅ Sí | Identificador único del caso o gestión | `"156894"` |

**Validaciones:**
- `caseId`: Debe ser un string no vacío
- El System API AC valida la existencia del caso

### Parámetros de Salida

#### Response Exitoso (200 OK)

**Content-Type:** `application/json`

**Estructura:**
```json
{
  "result": {
    "codResult": "string",
    "msgResult": "string"
  },
  "documents": "file"
}
```

**Descripción de Campos:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | string | Código de resultado de la operación (ej: "0000" = éxito) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |
| `documents` | file | Archivo ZIP con los documentos adjuntos al caso |

**Nota:** El campo `documents` contiene el archivo ZIP directamente. El tipo de archivo es `*/*` (cualquier tipo MIME).

#### Respuestas de Error

**400 Bad Request**
```json
{
  "message": "Bad request",
  "details": "Missing required parameter: caseId"
}
```

**404 Not Found**
```json
{
  "message": "Resource not found",
  "details": "Case not found with ID: 156894"
}
```

**500 Internal Server Error**
```json
{
  "message": "Internal server error",
  "details": "Error retrieving documents from System API"
}
```

**503 Service Unavailable**
```json
{
  "message": "Service temporarily unavailable",
  "details": "System API AC is not responding"
}
```

### Códigos de Estado HTTP

| Código | Descripción | Cuándo Ocurre |
|--------|-------------|---------------|
| 200 | OK | Documentos recuperados exitosamente |
| 400 | Bad Request | Parámetro caseId faltante o inválido |
| 404 | Not Found | Caso no encontrado o sin documentos |
| 500 | Internal Server Error | Error en el procesamiento interno |
| 503 | Service Unavailable | System API AC no disponible |

---

## 🏗️ Implementación Técnica

### Diagrama de Secuencia

```mermaid
sequenceDiagram
    participant Client as Cliente/Portal
    participant ProcessAPI as Process API<br/>ntgy-fua-p-imp-customer-queries
    participant SystemAPI as System API AC<br/>ntgy-fua-s-ac-fua
    participant DocRepo as Repositorio<br/>Documental

    Client->>ProcessAPI: GET /documentsByCase?caseId=156894
    
    Note over ProcessAPI: Logging de entrada<br/>(headers, query params)
    
    ProcessAPI->>SystemAPI: GET /documentsByCase?caseId=156894
    
    SystemAPI->>DocRepo: Consultar documentos del caso
    DocRepo-->>SystemAPI: Lista de documentos
    
    SystemAPI->>SystemAPI: Generar ZIP con documentos
    SystemAPI->>SystemAPI: Codificar ZIP en base64
    
    SystemAPI-->>ProcessAPI: 200 OK<br/>{fileName, fileContent, mimeType}
    
    ProcessAPI-->>Client: 200 OK<br/>{fileName, fileContent, mimeType}
    
    Note over Client: Cliente decodifica base64<br/>y guarda archivo ZIP
```

### Diagrama de Arquitectura

```mermaid
graph TB
    subgraph "Experience Layer"
        A[Portal Web]
        B[Mobile App]
        C[Agent Desktop]
    end
    
    subgraph "Process Layer"
        D[Process API<br/>ntgy-fua-p-imp-customer-queries]
        D1[APIKit Router]
        D2[HTTP Request<br/>Pass-Through]
    end
    
    subgraph "System Layer"
        E[System API AC<br/>ntgy-fua-s-ac-fua]
        E1[Document Service]
        E2[ZIP Generator]
    end
    
    subgraph "Data Layer"
        F[Repositorio<br/>Documental]
    end
    
    A --> D
    B --> D
    C --> D
    
    D --> D1
    D1 --> D2
    D2 --> E
    
    E --> E1
    E1 --> E2
    E2 --> F
    
    style D fill:#e1f5ff
    style E fill:#fff4e1
    style F fill:#f0f0f0
```

### Flujo de Datos

```mermaid
graph LR
    A[Request<br/>caseId: 156894] -->|Query Param| B[Process API<br/>Pass-Through]
    B -->|Delegación| C[System API AC]
    C -->|Consulta| D[Repositorio<br/>Documental]
    D -->|Documentos| C
    C -->|Genera ZIP| E[ZIP File]
    E -->|Base64| C
    C -->|Response| B
    B -->|Response| F[Cliente]
    F -->|Decodifica| G[Archivo ZIP<br/>Local]
    
    style B fill:#e1f5ff
    style C fill:#fff4e1
```

### Configuración

#### HTTP Request Configuration

**Archivo:** `global.xml`

```xml
<http:request-config 
    name="HTTP_API_SistemaAC" 
    doc:name="HTTP Request configuration" 
    basePath="/api" 
    responseTimeout="${s-ac.timeout}">
    <http:request-connection 
        host="${s-ac.host}" 
        port="${s-ac.port}" 
        connectionIdleTimeout="${s-ac.timeout}" 
        protocol="${s-ac.protocol}"/>
</http:request-config>
```

**Propiedades (Production):**

```yaml
s-ac:
  host: "ntgy-fua-s-ac-fua-bqyi65.internal-6bnlu3.irl-e1.cloudhub.io"
  port: "443"
  protocol: "HTTPS"
  timeout: "30000"  # 30 segundos
```

### Implementación del Flow

**Archivo:** `p-imp-customer-queries.xml` (línea 156-158)

```xml
<flow name="get:\documentsByCase:p-imp-customer-queries-config">
    <http:request 
        method="GET" 
        doc:name="Request" 
        config-ref="HTTP_API_SistemaAC" 
        path="/documentsByCase" />
</flow>
```

**Características de la Implementación:**

1. **Pass-Through Puro**: No hay transformaciones DataWeave
2. **Delegación Directa**: Reenvío inmediato al System API AC
3. **Query Parameters**: Se pasan automáticamente por el HTTP connector
4. **Sin Lógica de Negocio**: Implementación minimalista
5. **Timeout**: 30 segundos (configurado en properties)

### Manejo de Errores

**Error Handler Global** (heredado del flow principal):

```xml
<error-handler>
    <on-error-propagate type="HTTP:TIMEOUT">
        <!-- Timeout del System API AC -->
        <set-variable variableName="httpStatus" value="504"/>
        <set-payload value='{"message": "Gateway Timeout"}'/>
    </on-error-propagate>
    
    <on-error-propagate type="HTTP:CONNECTIVITY">
        <!-- System API AC no disponible -->
        <set-variable variableName="httpStatus" value="503"/>
        <set-payload value='{"message": "Service Unavailable"}'/>
    </on-error-propagate>
    
    <on-error-propagate type="ANY">
        <!-- Error genérico -->
        <set-variable variableName="httpStatus" value="500"/>
        <set-payload value='{"message": "Internal Server Error"}'/>
    </on-error-propagate>
</error-handler>
```

### Logging

**Entrada del Request** (línea 15 del archivo principal):

```xml
<logger level="INFO" doc:name="Logger INFO entrada request" message="#[output application/json
---
{
    &quot;method&quot;:attributes.method,
    &quot;requestUri&quot;:attributes.requestUri,
    (&quot;queryParams&quot;: attributes.queryParams) if (!isEmpty(attributes.queryParams))
}]"/>
```

**Información Registrada:**
- Método HTTP: `GET`
- URI completa: `/api/documentsByCase`
- Query parameters: `{caseId: "156894"}`
- Headers de cliente (si están presentes)

### Performance

#### Métricas Esperadas

| Métrica | Valor Objetivo | Notas |
|---------|----------------|-------|
| Response Time (P50) | < 3 segundos | Depende del tamaño del ZIP |
| Response Time (P95) | < 5 segundos | Casos con múltiples documentos |
| Response Time (P99) | < 8 segundos | Casos excepcionales |
| Timeout | 30 segundos | Configurado en properties |
| Throughput | 50 req/min | Limitado por generación de ZIP |

#### Factores de Performance

1. **Tamaño del ZIP**: Más documentos = mayor tiempo de generación
2. **Latencia de Red**: Comunicación Process API ↔ System API AC
3. **Carga del System API AC**: Disponibilidad y capacidad
4. **Ancho de Banda**: Transferencia de archivos base64

### Seguridad

#### Autenticación
- **OAuth 2.0**: Definido en RAML, aplicado por API Gateway
- **Token Bearer**: Requerido en header `Authorization`

#### Autorización
- **Validación de Acceso**: El System API AC valida que el usuario tenga acceso al caso
- **Datos Sensibles**: Los documentos pueden contener información confidencial

#### Encriptación
- **TLS 1.2+**: Comunicación cifrada en producción (HTTPS)
- **Base64**: Codificación del contenido del ZIP (no es encriptación)

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Request Básico

#### cURL
```bash
curl -X GET "https://ntgy-fua-p-imp-customer-queries-pro.de-c1.cloudhub.io/api/documentsByCase?caseId=156894" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json"
```

#### Response Exitoso (200 OK)
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "documents": "file.zip"
}
```

**Nota:** El campo `documents` contiene el archivo ZIP. En la respuesta real, este campo contendrá el archivo binario o una referencia al mismo, dependiendo de la implementación del System API AC.

### Ejemplo 2: Manejo de Respuesta (JavaScript)

```javascript
// Función para descargar documentos
async function downloadDocuments(caseId) {
  try {
    const response = await fetch(
      `https://api.example.com/api/documentsByCase?caseId=${caseId}`,
      {
        method: 'GET',
        headers: {
          'Authorization': `Bearer ${accessToken}`,
          'Content-Type': 'application/json'
        }
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    
    // Verificar resultado
    if (data.result.codResult === "0000") {
      console.log('Documentos recuperados exitosamente');
      console.log('Archivo:', data.documents);
      
      // El campo documents contiene el archivo ZIP
      // La implementación específica depende del System API AC
      // Puede ser un archivo binario, base64, o una URL de descarga
      
    } else {
      console.error('Error:', data.result.msgResult);
    }
    
  } catch (error) {
    console.error('Error al descargar documentos:', error);
  }
}

// Uso
downloadDocuments('156894');
```

**Nota:** La estructura exacta del campo `documents` depende de la implementación del System API AC. Consultar la documentación del System API para detalles específicos sobre el formato del archivo.

### Ejemplo 3: Manejo de Respuesta (Python)

```python
import requests

def download_documents(case_id, access_token):
    """
    Descarga documentos de un caso
    """
    url = f"https://api.example.com/api/documentsByCase"
    params = {"caseId": case_id}
    headers = {
        "Authorization": f"Bearer {access_token}",
        "Content-Type": "application/json"
    }
    
    try:
        response = requests.get(url, params=params, headers=headers)
        response.raise_for_status()
        
        data = response.json()
        
        # Verificar resultado
        if data['result']['codResult'] == "0000":
            print(f"Documentos recuperados exitosamente")
            print(f"Archivo: {data['documents']}")
            
            # El campo documents contiene el archivo ZIP
            # La implementación específica depende del System API AC
            
        else:
            print(f"Error: {data['result']['msgResult']}")
        
    except requests.exceptions.HTTPError as e:
        print(f"Error HTTP: {e}")
        print(f"Response: {e.response.text}")
    except Exception as e:
        print(f"Error: {e}")

# Uso
access_token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
download_documents("156894", access_token)
```

**Nota:** Consultar la documentación del System API AC (MUL-COM-FUA-S-AC-FUA) para detalles sobre el formato exacto del campo `documents`.

### Ejemplo 4: Error - Caso No Encontrado

#### Request
```bash
curl -X GET "https://ntgy-fua-p-imp-customer-queries-pro.de-c1.cloudhub.io/api/documentsByCase?caseId=999999" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json"
```

#### Response (404 Not Found)
```json
{
  "message": "Resource not found",
  "details": "Case not found with ID: 999999"
}
```

### Ejemplo 5: Error - Parámetro Faltante

#### Request
```bash
curl -X GET "https://ntgy-fua-p-imp-customer-queries-pro.de-c1.cloudhub.io/api/documentsByCase" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json"
```

#### Response (400 Bad Request)
```json
{
  "message": "Bad request",
  "details": "Missing required parameter: caseId"
}
```

### Ejemplo 6: Integración en Portal Web (React)

```jsx
import React, { useState } from 'react';
import { Button, Alert } from 'react-bootstrap';

function DocumentDownloader({ caseId }) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [success, setSuccess] = useState(false);

  const handleDownload = async () => {
    setLoading(true);
    setError(null);
    setSuccess(false);

    try {
      const response = await fetch(
        `/api/documentsByCase?caseId=${caseId}`,
        {
          headers: {
            'Authorization': `Bearer ${localStorage.getItem('token')}`
          }
        }
      );

      if (!response.ok) {
        throw new Error('Error al descargar documentos');
      }

      const data = await response.json();
      
      // Verificar resultado
      if (data.result.codResult === "0000") {
        setSuccess(true);
        console.log('Documentos disponibles:', data.documents);
        
        // El manejo específico del archivo depende de la implementación
        // del System API AC. Consultar documentación para detalles.
        
      } else {
        throw new Error(data.result.msgResult);
      }
      
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <Button 
        onClick={handleDownload} 
        disabled={loading}
        variant="primary"
      >
        {loading ? 'Descargando...' : 'Descargar Documentos'}
      </Button>
      
      {error && (
        <Alert variant="danger" className="mt-2">
          {error}
        </Alert>
      )}
      
      {success && (
        <Alert variant="success" className="mt-2">
          Documentos recuperados exitosamente
        </Alert>
      )}
    </div>
  );
}

export default DocumentDownloader;
```

**Nota:** La implementación específica del manejo del archivo ZIP depende del System API AC. Consultar la documentación de MUL-COM-FUA-S-AC-FUA para detalles.

---

## 📊 Métricas y Monitorización

### KPIs del Endpoint

| Métrica | Objetivo | Crítico Si |
|---------|----------|------------|
| Disponibilidad | > 99.5% | < 99% |
| Tiempo de Respuesta (P95) | < 5s | > 10s |
| Tasa de Error | < 1% | > 5% |
| Throughput | 50 req/min | < 20 req/min |

### Alertas Recomendadas

1. **Alta Latencia**: Response time > 10 segundos
2. **Tasa de Error Elevada**: Error rate > 5% en 5 minutos
3. **System API AC Down**: Múltiples errores 503 consecutivos
4. **Timeout Frecuente**: > 10 timeouts en 10 minutos

### Logs Importantes

```
INFO: GET /documentsByCase - caseId: 156894
INFO: Delegating to System API AC
INFO: Response received - Status: 200, Size: 2.5MB
```

---

## 🔗 System API Consumido

Este endpoint delega completamente al System API AC:

**API:** MUL-COM-FUA-S-AC-FUA (ntgy-fua-s-ac-fua)  
**Endpoint:** GET /documentsByCase  
**Documentación:** Para el schema completo y detalles de implementación del campo `documents`, consultar la documentación del System API AC, que es el SOURCE OF TRUTH de este endpoint.

**Nota sobre el campo `documents`:** El tipo y formato exacto del archivo ZIP en el campo `documents` está definido en el System API AC. Este Process API actúa como pass-through transparente, devolviendo la respuesta tal cual la proporciona el System API.

---

**Documentación generada para:** ntgy-fua-p-imp-customer-queries v1.15.0-SNAPSHOT  
**Tipo de API:** Process API (Orchestration Layer)  
**Patrón:** Pass-Through (Delegación Directa)  
**System API Consumido:** ntgy-fua-s-ac-fua (MUL-COM-FUA-S-AC-FUA)
