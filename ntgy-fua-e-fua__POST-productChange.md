---
api: "ntgy-fua-e-fua"
sidebar_position: 75
sidebar_label: "POST /productChange"
description: "Inicia el proceso de cambio de producto energético con firma electrónica digital"
keywords:
  - productChange
  - cambio producto
  - firma electrónica
  - proceso digital
  - FUA
tags:
  - Experience API
  - Firma Digital
channel: "Web/Móvil"
endpoint_dependencies:
  - method: "POST"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/firmas/procesos/:processType"
    purpose: "Gestión del proceso de firma electrónica para cambio de producto (processType=CambioDeProducto) con integración a servicios de firma digital"
---

# POST /productChange

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint inicia el proceso de cambio de producto energético para clientes de Naturgy, gestionando todo el flujo de firma electrónica digital necesario para formalizar el cambio. El endpoint actúa como punto de entrada desde los canales web y móvil, delegando la lógica compleja de firma al Process API correspondiente.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para canales web y móvil, orquestando el proceso de cambio de producto con firma electrónica.

### Patrón de Integración

**Tipo:** Pass-Through con Enriquecimiento de Headers

**Descripción:** El Experience API recibe la solicitud del cliente, enriquece el payload con headers FUA necesarios para el contexto de la operación, y delega toda la lógica de procesamiento al Process API `ntgy-newco-p-fua-evo`.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente Web/Móvil]
    E[Experience API<br/>ntgy-fua-e-fua<br/>POST /productChange]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /firmas/procesos/:processType]
    S1[System API<br/>s-postgresql-salesforce]
    S2[System API<br/>ntgy-newco-s-salesforce]
    S3[System API<br/>newco-s-aviva-evo]
    S4[System API<br/>s-rds-postgres]
    
    C -->|Request| E
    E -->|Enriquece Headers| P
    P -->|Consulta Cliente| S1
    P -->|Crea Caso| S2
    P -->|Envía Firma| S3
    P -->|Motor Estados| S4
    S4 -->|Response| P
    S3 -->|Response| P
    S2 -->|Response| P
    S1 -->|Response| P
    P -->|Response| E
    E -->|Adapted Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S3 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S4 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Cambio de Tarifa Energética**
   - Cliente solicita cambiar de tarifa regulada a tarifa de mercado libre
   - Requiere firma electrónica para formalizar el cambio contractual
   - Proceso completamente digital sin necesidad de visita presencial

2. **Cambio de Producto con Servicios Adicionales**
   - Cliente desea añadir o modificar servicios asociados al contrato
   - Gestión de firma electrónica con validación de identidad
   - Notificación inmediata al cliente del proceso iniciado

3. **Modificación de Condiciones Contractuales**
   - Cambio de producto que implica modificación de términos y condiciones
   - Generación automática de documentación legal
   - Envío de SMS con enlace seguro para firma digital

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web y Móvil

**Características del Canal:**
- **Plataforma**: Portal FUA (Frontal Único de Atención) y aplicaciones móviles iOS/Android
- **Dispositivos**: Navegadores web modernos, smartphones y tablets
- **Conectividad**: Requiere conexión estable para proceso de firma
- **Experiencia de Usuario**: Proceso guiado paso a paso con confirmación por SMS

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Particulares que desean cambiar su producto energético
- **Clientes Business**: Pequeñas empresas que gestionan sus contratos de forma autónoma

**Contexto de Uso:**
- **Autoservicio Digital**: Cliente inicia el proceso desde su área privada
- **Firma Electrónica**: Recibe SMS con enlace para firmar digitalmente el cambio
- **Seguimiento en Tiempo Real**: Puede consultar el estado del proceso en cualquier momento

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Simplificación de Datos**: Solo se solicita información esencial al cliente
- **Formato de Respuesta**: Respuesta simple con código de resultado
- **Notificaciones**: Integración con sistema de notificaciones push y SMS
- **Validación Asíncrona**: El proceso de firma se gestiona de forma asíncrona

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/firmas/procesos/:processType` | POST | Gestión completa del proceso de firma electrónica (processType=CambioDeProducto) |

### Contexto Empresarial

- **Dominio:** Autoservicio de Clientes - Gestión de Contratos
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Web/Móvil

### Stakeholders

- **Clientes Finales**: Necesitan cambiar su producto energético de forma rápida y digital
- **Departamento Comercial**: Requiere seguimiento de solicitudes de cambio de producto
- **Departamento Legal**: Necesita garantizar la validez legal de las firmas electrónicas
- **Atención al Cliente**: Debe poder consultar el estado de los procesos de firma

### Valor de Negocio

- **Digitalización Completa**: Eliminación de procesos en papel y visitas presenciales
- **Reducción de Tiempos**: Proceso de cambio de producto en minutos en lugar de días
- **Mejora de Experiencia**: Cliente gestiona todo desde su dispositivo móvil o web
- **Cumplimiento Legal**: Firma electrónica con validez legal completa
- **Trazabilidad**: Registro completo del proceso en Motor de Estados

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/productChange` |
| **Development** | `https://e-fua-dev.de-c1.cloudhub.io` | `https://e-fua-dev.de-c1.cloudhub.io/fua/api/productChange` |
| **Production** | `https://e-fua-pro.de-c1.cloudhub.io` | `https://e-fua-pro.de-c1.cloudhub.io/fua/api/productChange` |

### Método HTTP

```
POST /fua/api/productChange
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Proceso complejo que incluye múltiples integraciones |
| **Tamaño de Respuesta** | < 5 KB | Respuesta simple con código de resultado |
| **Rate Limiting** | 10 requests/minuto por usuario | Prevención de solicitudes duplicadas |
| **Disponibilidad** | 24/7 | Proceso crítico de negocio |

#### Campos Específicos del Canal

**Campos Añadidos por el Experience API:**

| Campo | Tipo | Descripción | Razón |
|-------|------|-------------|-------|
| `x-client-id` | String | ID del cliente en Salesforce | Contexto de sesión del usuario |
| `x-contact-id` | String | ID del contacto en Salesforce | Identificación del usuario |
| `x-client-code` | String | Código del cliente (NIF/CIF) | Identificación fiscal |
| `x-client-doc-number` | String | Número de documento | Validación de identidad |
| `x-language` | String | Idioma preferido del cliente | Personalización de comunicaciones |
| `x-trading-company` | String | Compañía comercializadora | Routing de proceso |

### Parámetros de Entrada

#### Headers Obligatorios

```json
{
  "Content-Type": "application/json",
  "x-client-id": "0015I00000XxXxXXXX",
  "x-contact-id": "0035I00000YyYyYYYY",
  "x-client-code": "12345678A",
  "x-client-doc-number": "12345678A",
  "x-language": "es",
  "x-trading-company": "newco"
}
```

#### Body Request

**⚠️ Nota sobre Pass-Through:** Este Experience API actúa como pass-through, enviando todos los campos al Process API. Sin embargo, el Process API (`ntgy-newco-p-fua-evo`) solo utiliza un subconjunto de estos campos para el proceso de firma.

```json
{
  "contractId": "8007Y0000031FbfQAE",
  "serviceAccountId": "a1X7Y000000AbCdEFG",
  "billingAccountId": "a1Y7Y000000XyZaBCD",
  "digitalSignatureType": "01"
}
```

**Campos Realmente Utilizados por el Process API:**

| Campo | Tipo | Obligatorio | Descripción | Uso en Process API |
|-------|------|-------------|-------------|-------------------|
| `contractId` | String | Sí | ID del contrato en Salesforce | ✅ Usado para crear caso de gestión y motor de estados |
| `serviceAccountId` | String | Sí | ID de la cuenta de servicio | ✅ Usado para crear caso de gestión |
| `billingAccountId` | String | Sí | ID de la cuenta de facturación | ✅ Usado para crear caso de gestión y motor de estados |
| `digitalSignatureType` | String | Sí | Tipo de firma digital (01=SMS, 02=Email) | ✅ Usado para determinar el canal de envío de firma |

**Campos Adicionales del RAML (Pass-Through):**

Estos campos están definidos en el RAML de la Experience API pero **no son procesados** por el Process API en el flujo de firma. Se envían como pass-through pero no afectan el proceso:

| Campo | Tipo | Obligatorio | Descripción | Estado |
|-------|------|-------------|-------------|--------|
| `productId` | String | Sí | ID del producto energético en Salesforce | ⚠️ No procesado en firma |
| `priceBookEntryId` | String | Sí | ID de la entrada en el catálogo de precios | ⚠️ No procesado en firma |
| `cups` | String \| null | Sí | Código Universal del Punto de Suministro | ⚠️ No procesado en firma |
| `bankAccount` | Object | Sí | Datos de la cuenta bancaria | ⚠️ No procesado en firma |
| `contactId` | String | Sí | ID del contacto/cliente en Salesforce | ⚠️ No procesado (se usa header) |
| `addressId` | String | Sí | ID de la dirección del suministro | ⚠️ No procesado en firma |
| `mailingAddress` | String | No | Dirección postal para correspondencia | ⚠️ No procesado en firma |
| `promotionIdList` | Array[String] | Sí | Lista de IDs de promociones | ⚠️ No procesado en firma |
| `price` | Object | No | Estructura de precios del nuevo producto | ⚠️ No procesado en firma |
| `techDataId` | String \| null | No | ID de datos técnicos del suministro | ⚠️ No procesado en firma |
| `contractNumber` | String | No | Número de contrato | ⚠️ No procesado en firma |
| `startDate` | String \| null | No | Fecha de inicio del contrato | ⚠️ No procesado en firma |

**📝 Nota Importante:** El Process API obtiene la información adicional necesaria (datos del cliente, idioma, teléfono, etc.) mediante consultas a PostgreSQL-Salesforce usando los IDs proporcionados, no del payload directo.

### Respuestas

#### Respuesta Exitosa (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "caseCode": "CASE-2024-001234"
}
```

**Campos de la Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | String | Código de resultado (0000=éxito) |
| `result.msgResult` | String | Mensaje descriptivo del resultado |
| `caseCode` | String | Código del caso de gestión creado en Salesforce |

#### Respuesta de Error (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error en validación de datos"
  }
}
```

#### Respuesta de Error (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "9999",
    "msgResult": "Error interno del servidor"
  }
}
```

### Autenticación

- **Tipo:** OAuth 2.0 con JWT tokens
- **Scope:** Requiere autenticación de cliente
- **Headers:** Token de acceso en header `Authorization`

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Proceso complejo con múltiples integraciones |
| **Connection Timeout** | 30 segundos | Tiempo máximo para establecer conexión |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP Request<br/>POST /productChange]) --> LogStart[Log Inicio]
    LogStart --> Transform[Transform: Merge Payload + Headers FUA]
    Transform --> CallProcess[HTTP Request: Process API<br/>POST /firmas/procesos/CambioDeProducto]
    CallProcess --> Response[Response del Process API]
    Response --> End([HTTP Response<br/>Código de Resultado])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style Transform fill:#fff3e0,stroke:#f57c00
    style CallProcess fill:#f3e5f5,stroke:#8e24aa
```

### Transformaciones de Canal

**Transformación de Enriquecimiento:**

El Experience API enriquece el payload original con los headers FUA necesarios para el contexto:

```dataweave
%dw 2.0
output application/json
---
payload 
++
vars.fuaHeaders
```

**Mapeo de Headers a Payload:**

| Header Original | Campo en Payload Enriquecido | Propósito |
|-----------------|------------------------------|-----------|
| `x-client-id` | `x-client-id` | ID del cliente en Salesforce |
| `x-contact-id` | `x-contact-id` | ID del contacto en Salesforce |
| `x-client-code` | `x-client-code` | Código del cliente (NIF/CIF) |
| `x-client-doc-number` | `x-client-doc-number` | Número de documento |
| `x-language` | `x-language` | Idioma preferido |
| `x-trading-company` | `x-trading-company` | Compañía comercializadora |

### Patrones de Diseño

**Patrón:** Delegación con Enriquecimiento de Contexto

**Justificación:** 
- El Experience API no contiene lógica de negocio compleja
- Se enfoca en adaptar la experiencia del canal (web/móvil)
- Enriquece el contexto con información de sesión del usuario
- Delega toda la lógica de firma al Process API especializado

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente Web/Móvil
    participant E as Experience API
    participant P as Process API
    participant S as System APIs
    
    C->>E: POST /productChange
    Note over E: Valida Headers FUA
    E->>E: Transform: Merge Payload + Headers
    E->>P: POST /firmas/procesos/:processType
    Note over P: Proceso de Firma Completo
    P->>S: Consultas y Operaciones
    S-->>P: Respuestas
    P-->>E: {"codResult": "0000"}
    E-->>C: Proceso Iniciado Exitosamente
    Note over C: Cliente recibe SMS<br/>con enlace de firma
```

#### 2. Flujo con Error de Validación

```mermaid
sequenceDiagram
    participant C as Cliente Web/Móvil
    participant E as Experience API
    participant P as Process API
    
    C->>E: POST /productChange<br/>(datos incompletos)
    Note over E: Valida Headers FUA
    E->>P: POST /firmas/procesos/CambioDeProducto
    Note over P: Validación de Datos
    P-->>E: Error de Validación
    E-->>C: {"codResult": "0001",<br/>"msgResult": "Error validación"}
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant C as Cliente Web/Móvil
    participant E as Experience API
    participant P as Process API
    participant S as System API
    
    C->>E: POST /productChange
    E->>P: POST /firmas/procesos/CambioDeProducto
    P->>S: Operación en System API
    S-->>P: Error de Conexión
    Note over P: Manejo de Error
    P-->>E: {"codResult": "9999"}
    E-->>C: Error Interno del Servidor
```

### Proceso Completo en el Process API

El Process API `ntgy-newco-p-fua-evo` ejecuta el siguiente flujo (subflow `firmaSub_Flow`):

1. **Inicialización del Motor de Estados**
   - Mapeo del tipo de proceso: `CambioDeProducto`
   - Creación de intención en Motor de Estados
   - Generación de evento: `SolicitudCambioProductoIniciada`

2. **Consulta de Datos del Cliente**
   - Query a PostgreSQL-Salesforce para obtener:
     - Idioma preferido del cliente
     - Flag de bypass de firma (si aplica)
     - Datos del titular (nombre, apellidos, documento)
     - Teléfono móvil

3. **Validación de Mandato** (si aplica)
   - Verificación de mandato SEPA existente
   - Generación de nuevo mandato si es necesario

4. **Creación de Caso de Gestión en Salesforce**
   - Tipo de caso: "Gestion"
   - Subject: "Cambio producto"
   - Status: "03" (En proceso)
   - Vinculación con contrato y cuenta de facturación

5. **Generación de Documento para Firma**
   - Obtención de plantilla CLM (Contract Lifecycle Management)
   - Generación de documento PDF con datos del cliente
   - Tipo de documento: "contratoComun"

6. **Envío de SMS con Firma Electrónica**
   - Integración con Aviva (proveedor de firma electrónica)
   - Envío de SMS al teléfono del cliente
   - Generación de enlace seguro con expiración

7. **Creación de Notificación**
   - Registro en DynamoDB de notificación de firma iniciada
   - Visible en el área privada del cliente

8. **Actualización de Estados**
   - Actualización en Motor de Estados: `FIRMA_SOLICITUD`
   - Actualización en RDS Omega: `ContratoPendienteFirmaEnviado`
   - Comentario en Caso de Salesforce

### Optimizaciones del Canal

**Optimizaciones Aplicadas:**

1. **Respuesta Inmediata**: El endpoint responde inmediatamente sin esperar el proceso completo de firma
2. **Procesamiento Asíncrono**: La firma se gestiona de forma asíncrona mediante eventos
3. **Notificaciones Push**: El cliente recibe notificaciones del progreso en tiempo real
4. **Caché de Configuración**: Datos de configuración cacheados para mejorar rendimiento

### Manejo de Errores

**Estrategia de Manejo de Errores:**

1. **Errores de Validación**: Retornados inmediatamente al cliente con código específico
2. **Errores de Integración**: Manejados con reintentos automáticos en el Process API
3. **Errores de Firma**: Notificados al cliente mediante SMS y notificación push
4. **Errores Críticos**: Escalados a soporte con registro completo en logs

**Códigos de Error Comunes:**

| Código | Descripción | Acción Recomendada |
|--------|-------------|-------------------|
| `0000` | Sin error | Proceso exitoso |
| `0001` | Error de validación | Verificar datos de entrada |
| `0002` | Cliente no encontrado | Verificar ID de cliente |
| `0003` | Contrato no válido | Verificar ID de contrato |
| `9999` | Error interno | Contactar soporte |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Llamada Exitosa desde Web

#### cURL

```bash
curl -X POST "https://e-fua-pro.de-c1.cloudhub.io/fua/api/productChange" \
  -H "Content-Type: application/json" \
  -H "x-client-id: 0015I00000XxXxXXXX" \
  -H "x-contact-id: 0035I00000YyYyYYYY" \
  -H "x-client-code: 12345678A" \
  -H "x-client-doc-number: 12345678A" \
  -H "x-language: es" \
  -H "x-trading-company: newco" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -d '{
    "contractId": "8007Y0000031FbfQAE",
    "serviceAccountId": "a1X7Y000000AbCdEFG",
    "billingAccountId": "a1Y7Y000000XyZaBCD",
    "digitalSignatureType": "01"
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "caseCode": "CASE-2024-001234"
}
```

**Nota:** El cliente recibirá un SMS con un enlace para completar la firma electrónica del cambio de producto.

### Ejemplo 2: JavaScript (para Web)

```javascript
const productChange = async (contractData) => {
  const response = await fetch('https://e-fua-pro.de-c1.cloudhub.io/fua/api/productChange', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-client-id': '0015I00000XxXxXXXX',
      'x-contact-id': '0035I00000YyYyYYYY',
      'x-client-code': '12345678A',
      'x-client-doc-number': '12345678A',
      'x-language': 'es',
      'x-trading-company': 'newco',
      'Authorization': 'Bearer ' + getAccessToken()
    },
    body: JSON.stringify({
      contractId: contractData.contractId,
      serviceAccountId: contractData.serviceAccountId,
      billingAccountId: contractData.billingAccountId,
      digitalSignatureType: '01'
    })
  });

  const data = await response.json();
  
  if (data.result.codResult === '0000') {
    console.log('Proceso de firma iniciado exitosamente');
    console.log('Código de caso:', data.caseCode);
    showNotification('Recibirás un SMS para completar la firma');
  } else {
    console.error('Error:', data.result.msgResult);
  }
  
  return data;
};

// Ejemplo de uso
const contractData = {
  contractId: '8007Y0000031FbfQAE',
  serviceAccountId: 'a1X7Y000000AbCdEFG',
  billingAccountId: 'a1Y7Y000000XyZaBCD'
};

productChange(contractData);
```

### Ejemplo 3: Python

```python
import requests
import json

def iniciar_cambio_producto(access_token, client_data, contract_data):
    url = 'https://e-fua-pro.de-c1.cloudhub.io/fua/api/productChange'
    
    headers = {
        'Content-Type': 'application/json',
        'x-client-id': client_data['client_id'],
        'x-contact-id': client_data['contact_id'],
        'x-client-code': client_data['client_code'],
        'x-client-doc-number': client_data['doc_number'],
        'x-language': 'es',
        'x-trading-company': 'newco',
        'Authorization': f'Bearer {access_token}'
    }
    
    payload = {
        'contractId': contract_data['contract_id'],
        'serviceAccountId': contract_data['service_account_id'],
        'billingAccountId': contract_data['billing_account_id'],
        'digitalSignatureType': '01'
    }
    
    response = requests.post(url, headers=headers, json=payload)
    
    if response.status_code == 200:
        result = response.json()
        if result['result']['codResult'] == '0000':
            print('Proceso de firma iniciado exitosamente')
            print(f"Código de caso: {result['caseCode']}")
            return result
        else:
            print(f"Error: {result['result']['msgResult']}")
            return None
    else:
        print(f"Error HTTP: {response.status_code}")
        return None

# Ejemplo de uso
client_data = {
    'client_id': '0015I00000XxXxXXXX',
    'contact_id': '0035I00000YyYyYYYY',
    'client_code': '12345678A',
    'doc_number': '12345678A'
}

contract_data = {
    'contract_id': '8007Y0000031FbfQAE',
    'service_account_id': 'a1X7Y000000AbCdEFG',
    'billing_account_id': 'a1Y7Y000000XyZaBCD'
}

iniciar_cambio_producto('your_access_token', client_data, contract_data)
```

### Ejemplo 4: Error de Validación

#### Request con Datos Incompletos

```bash
curl -X POST "https://e-fua-pro.de-c1.cloudhub.io/fua/api/productChange" \
  -H "Content-Type: application/json" \
  -H "x-client-id: 0015I00000XxXxXXXX" \
  -H "x-contact-id: 0035I00000YyYyYYYY" \
  -H "x-client-code: 12345678A" \
  -H "x-language: es" \
  -d '{
    "contractId": "8007Y0000031FbfQAE",
    "digitalSignatureType": "01"
  }'
```

#### Respuesta (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Faltan campos obligatorios: serviceAccountId, billingAccountId"
  }
}
