---
api: "ntgy-fua-e-fua"
sidebar_position: 93
sidebar_label: "PUT /contractAccounts"
description: "Actualiza cuentas contrato con diferentes operaciones: cambio de cuenta bancaria, modificación de método de pago o modificaciones generales"
keywords: [contractAccounts, cuenta contrato, cambio cuenta bancaria, método de pago, firma digital, modificaciones, FUA]
tags: [Experience API, FUA, Cuentas Contrato, Web, Móvil]
channel: web
endpoint_dependencies:
  - method: "POST"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/firmas/procesos/:processType"
    purpose: "Proceso de firma digital para modificaciones de cuenta contrato (CambioCuentaBancaria o ModificacionMetodoPago según parámetro processType)"
  - method: "POST"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/modificaciones"
    purpose: "Modificaciones generales de cuenta contrato sin proceso de firma"
---

# PUT /contractAccounts

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite actualizar información de cuentas contrato de clientes de energía mediante diferentes tipos de operaciones. Implementa un patrón de enrutamiento condicional que dirige las solicitudes a diferentes procesos del Process API según el tipo de operación solicitada, incluyendo procesos con firma digital para operaciones críticas como cambios de cuenta bancaria o método de pago.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Orquestación y enrutamiento de modificaciones de cuenta contrato para canales web y móvil del portal FUA (Frontal Único de Atención)

### Patrón de Integración

**Tipo:** Content-Based Routing con Transformación de Canal

**Descripción:** El endpoint analiza el campo `operationType` del request para determinar el flujo de procesamiento apropiado. Las operaciones que requieren firma digital (`bankAccount`, `changePaymentMethod`) se enrutan a endpoints de firma, mientras que otras modificaciones se procesan directamente.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente FUA<br/>Portal Web/Móvil]
    E[Experience API<br/>ntgy-fua-e-fua<br/>PUT /contractAccounts]
    P1[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /firmas/procesos/:processType]
    P2[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /modificaciones]
    S[System APIs<br/>Backend Systems]
    
    C -->|Request con operationType| E
    E -->|bankAccount o changePaymentMethod| P1
    E -->|Otras operaciones| P2
    P1 -->|Consume| S
    P2 -->|Consume| S
    S -->|Response| P1
    S -->|Response| P2
    P1 -->|Response| E
    P2 -->|Response| E
    E -->|Adapted Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P1 fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style P2 fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Cambio de Cuenta Bancaria con Firma Digital**
   - Cliente solicita cambiar la cuenta bancaria asociada a su contrato de energía
   - Requiere proceso de firma digital para validación y seguridad
   - Beneficio: Proceso seguro y trazable de cambio de domiciliación bancaria

2. **Modificación de Método de Pago con Firma Digital**
   - Cliente cambia su método de pago (ej: de domiciliación a transferencia)
   - Requiere firma digital para confirmar el cambio
   - Beneficio: Flexibilidad en la gestión de pagos con seguridad jurídica

3. **Modificaciones Generales de Cuenta Contrato**
   - Actualizaciones de datos de cuenta contrato que no requieren firma digital
   - Incluye cambios de datos de contacto, preferencias, etc.
   - Beneficio: Autoservicio rápido para modificaciones no críticas

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web y Móvil

**Características del Canal:**
- **Plataforma**: Portal FUA (Frontal Único de Atención) - Web responsive y aplicación móvil
- **Dispositivos**: Navegadores web (desktop/tablet) y smartphones iOS/Android
- **Conectividad**: Requiere conexión estable para procesos de firma digital
- **Experiencia de Usuario**: Interfaz intuitiva con validación en tiempo real y feedback inmediato

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Clientes particulares que gestionan sus contratos de energía de forma autónoma
- **Clientes Empresariales**: Pequeñas y medianas empresas que administran sus cuentas de energía
- **Agentes de Servicio**: Personal de atención al cliente que realiza modificaciones en nombre de clientes

**Contexto de Uso:**
- **Autoservicio Web/Móvil**: Cliente accede desde su área personal para realizar modificaciones
- **Asistencia Telefónica**: Agente realiza cambios mientras atiende al cliente por teléfono
- **Gestión Proactiva**: Cliente actualiza datos antes de vencimientos o cambios de situación

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Enriquecimiento de Headers**: Añade headers de contexto FUA (x-interaction-id, x-user-id, etc.) para trazabilidad
- **Extracción de IP**: Captura la dirección IP del cliente para auditoría y seguridad
- **Validación de Entrada**: Valida estructura del request antes de enrutar al Process API
- **Manejo de Errores Simplificado**: Transforma errores técnicos en mensajes comprensibles para el usuario

### 🔗 Endpoints Dependientes

Este Experience API consume los siguientes endpoints del Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/firmas/procesos/:processType` | POST | Proceso de firma digital para modificaciones críticas (CambioCuentaBancaria o ModificacionMetodoPago) |
| `ntgy-newco-p-fua-evo` | `/modificaciones` | POST | Modificaciones generales de cuenta contrato sin firma digital |

### Contexto Empresarial

- **Dominio:** Gestión de Cuentas Contrato y Autoservicio de Clientes
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Web y Móvil (Portal FUA)

### Stakeholders

- **Clientes de Energía**: Necesitan gestionar sus cuentas contrato de forma autónoma y segura
- **Agentes de Atención al Cliente**: Requieren herramientas para asistir a clientes en modificaciones
- **Departamento de Facturación**: Necesita garantizar la correcta actualización de datos de pago
- **Departamento Legal**: Requiere trazabilidad y firma digital para operaciones críticas

### Valor de Negocio

- **Reducción de Llamadas al Call Center**: Autoservicio para modificaciones comunes reduce carga operativa
- **Seguridad Jurídica**: Firma digital proporciona validez legal a cambios críticos
- **Mejora de Experiencia de Cliente**: Proceso rápido y sencillo desde cualquier dispositivo
- **Trazabilidad Completa**: Registro detallado de todas las modificaciones con IP y usuario
- **Flexibilidad Operativa**: Soporte para múltiples tipos de modificaciones en un solo endpoint

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8083` | `http://localhost:8083/fua/api/contractAccounts` |
| **Development** | `https://ntgy-fua-e-fua-des.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-des.de-c1.cloudhub.io/fua/api/contractAccounts` |
| **UAT** | `https://ntgy-fua-e-fua-cc.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-cc.de-c1.cloudhub.io/fua/api/contractAccounts` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/contractAccounts` |

### Método HTTP

```
PUT /fua/api/contractAccounts
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Procesos de firma digital pueden requerir tiempo adicional |
| **Tamaño de Respuesta** | 5 MB máximo | Respuestas incluyen datos de confirmación y trazabilidad |
| **Rate Limiting** | 100 requests/minuto por usuario | Prevención de abuso y protección del backend |
| **Autenticación** | JWT obligatorio | Todas las operaciones requieren usuario autenticado |

#### Campos Específicos del Canal

**Campos Añadidos por el Experience API:**

| Campo | Tipo | Descripción | Razón |
|-------|------|-------------|-------|
| `DirIp` | String | Dirección IP del cliente | Auditoría y seguridad - trazabilidad de origen de la solicitud |
| `x-interaction-id` | String (Header) | ID único de interacción | Trazabilidad end-to-end de la operación |
| `x-user-id` | String (Header) | ID del usuario autenticado | Identificación del usuario que realiza la modificación |
| `x-client-id` | String (Header) | ID del cliente | Identificación del cliente afectado |
| `x-origin` | String (Header) | Origen de la petición (web/mobile) | Análisis de uso por canal |

**Campos del Request (No Omitidos):**

Este Experience API actúa como pass-through para el payload, añadiendo únicamente headers de contexto y la IP del cliente. Todos los campos del request se envían al Process API.

### Parámetros de Entrada

#### Headers Requeridos

| Header | Tipo | Obligatorio | Descripción |
|--------|------|-------------|-------------|
| `Content-Type` | String | Sí | Debe ser `application/json` |
| `Authorization` | String | Sí | Token JWT del usuario autenticado |
| `x-interaction-id` | String | Sí | ID único de la interacción para trazabilidad |
| `x-user-id` | String | Sí | ID del usuario que realiza la operación |
| `x-client-id` | String | No | ID del cliente (opcional para agentes) |
| `x-origin` | String | Sí | Origen: `web` o `mobile` |

#### Body Parameters

| Campo | Tipo | Obligatorio | Descripción |
|-------|------|-------------|-------------|
| `operationType` | String | No | Tipo de operación: `bankAccount`, `changePaymentMethod`, u otros. Determina el enrutamiento |
| `digitalSignatureType` | String | No | Código postal para firma digital (ej: "28023") |
| `reason` | String | No | Código del motivo de baja FOL (ej: "03") |
| `text` | String | No | Motivo de baja FOL en texto libre |
| `serviceAccountList` | Array[ServiceAccount] | No | Lista de cuentas de servicio a modificar |

##### Estructura de ServiceAccount

| Campo | Tipo | Obligatorio | Descripción |
|-------|------|-------------|-------------|
| `serviceAccountId` | String | Sí | Identificador de la cuenta de servicio (ej: "14479766") |
| `clientId` | String | Sí | Identificador del cliente |
| `address` | Address | No | Dirección del punto de suministro |
| `billingAccountList` | Array[BillingAccount] | Sí | Lista de cuentas contrato asociadas |
| `lastInvoice` | Invoice | No | Última factura disponible |

##### Estructura de Address (anidado en ServiceAccount)

| Campo | Tipo | Obligatorio | Descripción |
|-------|------|-------------|-------------|
| `addressId` | String | No | Identificador de dirección |
| `zipCode` | String | Sí | Código postal (ej: "28023") |
| `province` | String | Sí | Provincia |
| `city` | String | Sí | Ciudad |
| `town` | String | No | Población |
| `way` | String | No | Vía |
| `wayId` | String | No | Identificador de vía |
| `wayType` | String | No | Tipo de vía (calle, avenida, etc.) |
| `number` | String | No | Número del inmueble |
| `floor` | String | No | Piso |
| `door` | String | No | Puerta |
| `portal` | String | No | Bloque/Portal |
| `stairs` | String | No | Escalera |
| `placeId` | String | No | ID de dirección horizontal |

##### Estructura de BillingAccount (anidado en ServiceAccount)

| Campo | Tipo | Obligatorio | Descripción |
|-------|------|-------------|-------------|
| `refNum` | String | No | Número de referencia (ROW_ID de Siebel) |
| `billingAccountId` | String | No | Identificador de cuenta contrato |
| `isDirectPayment` | Boolean | No | Flag de pago domiciliado |
| `billDeliveryMethod` | String | No | Método de entrega de factura (factura online FOL) |
| `bankAccount` | BankAccount | No | Cuenta bancaria asociada |
| `products` | Array[Product] | No | Lista de productos contratados |
| `mailingAddress` | Address | No | Dirección de correspondencia |

##### Estructura de Invoice (anidado en ServiceAccount)

| Campo | Tipo | Obligatorio | Descripción |
|-------|------|-------------|-------------|
| `invoiceId` | String | No | Identificador de factura |
| `issuedDate` | String | No | Fecha de emisión de factura |
| `billingDate` | String | No | Fecha de facturación (fecha vencimiento) |
| `status` | String | No | Estado de pago de la factura |
| `pendingAmount` | Number | No | Importe pendiente de la factura |
| `amount` | Number | Sí | Importe total de la factura (ej: 31.22) |
| `interactiveInvoice` | String | No | URL encriptada con la factura interactiva |
| `pdfDocumentId` | String | No | ID del documento en gestor documental |
| `periodEnd` | String | No | Fecha de último periodo |
| `type` | String | No | Indicador si es factura NC o NI |

#### Ejemplo de Request

```json
{
  "operationType": "bankAccount",
  "digitalSignatureType": "28023",
  "reason": "03",
  "text": "Cambio de entidad bancaria",
  "serviceAccountList": [
    {
      "serviceAccountId": "14479766",
      "clientId": "12345",
      "address": {
        "addressId": "14479766",
        "zipCode": "28023",
        "province": "Madrid",
        "city": "Madrid",
        "town": "Madrid",
        "way": "Calle Mayor",
        "wayType": "Calle",
        "number": "123",
        "floor": "3",
        "door": "A"
      },
      "billingAccountList": [
        {
          "billingAccountId": "9876543210",
          "isDirectPayment": true,
          "billDeliveryMethod": "ONLINE"
        }
      ],
      "lastInvoice": {
        "invoiceId": "INV-2024-001",
        "issuedDate": "2024-01-15",
        "billingDate": "2024-02-01",
        "status": "PAID",
        "amount": 85.50
      }
    }
  ]
}
```

### Respuestas

#### Respuesta Exitosa (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Operación realizada correctamente"
  }
}
```

#### Respuesta de Error (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al procesar la solicitud: [descripción del error]"
  }
}
```

#### Códigos de Estado HTTP

| Código | Descripción | Cuándo Ocurre |
|--------|-------------|---------------|
| `200` | OK | Operación completada exitosamente |
| `400` | Bad Request | Request malformado o campos inválidos |
| `401` | Unauthorized | Token JWT inválido o expirado |
| `500` | Internal Server Error | Error en el procesamiento del Process API |
| `504` | Gateway Timeout | Timeout en la llamada al Process API (>60s) |

### Autenticación

- **Tipo:** OAuth 2.0 con JWT (JSON Web Token)
- **Validación:** El endpoint `jwt-validation` valida el token antes de procesar la solicitud
- **Headers:** Token incluido en header `Authorization: Bearer {token}`
- **Roles:** Soporta roles de cliente y agente con diferentes permisos

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Procesos de firma digital pueden requerir tiempo adicional de procesamiento |
| **Connection Idle Timeout** | 60 segundos | Mantiene conexión activa durante procesos largos |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP Request<br/>PUT /contractAccounts]) --> Log[Log Request - Kinesis]
    Log --> JWTVal[JWT Validation]
    JWTVal --> SetHeaders[Set FUA Headers Variable]
    SetHeaders --> Transform[Transform: Add Headers + IP]
    Transform --> Choice{operationType?}
    
    Choice -->|bankAccount| Route1[POST /firmas/procesos/<br/>CambioCuentaBancaria]
    Choice -->|changePaymentMethod| Route2[POST /firmas/procesos/<br/>ModificacionMetodoPago]
    Choice -->|Otros| Route3[POST /modificaciones]
    
    Route1 --> Response[HTTP Response]
    Route2 --> Response
    Route3 --> Response
    
    Response --> End([200 OK])
    
    Choice -.->|Error| ErrorHandler[Error Handler]
    ErrorHandler --> ErrorTransform[Transform Error Response]
    ErrorTransform --> ErrorLog[Log Error]
    ErrorLog --> ErrorEnd([500 Error])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#c8e6c9,stroke:#2e7d32
    style ErrorEnd fill:#ffcdd2,stroke:#c62828
    style Choice fill:#fff9c4,stroke:#f57f17
```

### Transformaciones de Canal

#### Transformación Principal: Enriquecimiento de Request

**Ubicación en el código:** `put-contract-accounts.xml` (líneas 13-28)

**Lógica de Transformación:**

```dataweave
%dw 2.2
import * from dw::core::Strings
output application/json
---
payload
++
vars.fuaHeaders 
++ 
{
    "DirIp": substringBefore(substringAfter(attributes.remoteAddress,"/"),":")
}
```

**Descripción:**
1. **Preserva el payload original**: Mantiene todos los campos del request del cliente
2. **Añade headers FUA**: Incluye headers de contexto (x-interaction-id, x-user-id, etc.) desde `vars.fuaHeaders`
3. **Extrae IP del cliente**: Parsea `attributes.remoteAddress` para obtener la IP limpia

**Ejemplo de Transformación:**

| Campo Original | Campo Transformado | Transformación |
|----------------|-------------------|----------------|
| `payload.*` | `payload.*` | Pass-through (sin cambios) |
| `vars.fuaHeaders` | Headers de contexto | Añadidos al payload |
| `attributes.remoteAddress` | `DirIp` | Extracción de IP: `/192.168.1.100:54321` → `192.168.1.100` |

#### Variable fuaHeaders

**Ubicación:** `e-fua.xml` (líneas 15-37)

**Campos Incluidos:**

```dataweave
{
  "x-interaction-id": attributes.headers['x-interaction-id'],
  "x-interaction-origin": attributes.headers['x-interaction-origin'],
  "x-user-id": attributes.headers['x-user-id'],
  "x-client-id": attributes.headers['x-client-id'],
  "x-client-doc-number": attributes.headers['x-client-doc-number'],
  "x-client-code": attributes.headers['x-client-code'],
  "x-origin": attributes.headers['x-origin'],
  "x-user-role": attributes.headers['x-user-role'],
  "traceparent": attributes.headers['traceparent'],
  "x-agent-mail": attributes.headers['x-agent-mail'],
  "x-language": attributes.headers['x-language'],
  "x-contact-id": attributes.headers['x-contact-id']
}
```

**Nota:** Solo se incluyen headers que no son null y no están vacíos.

### Patrones de Diseño

**Patrón:** Content-Based Router (Enterprise Integration Pattern)

**Justificación:** 
- Permite enrutar solicitudes a diferentes endpoints del Process API según el tipo de operación
- Centraliza la lógica de enrutamiento en el Experience API
- Facilita la adición de nuevos tipos de operaciones sin modificar el Process API
- Proporciona un único punto de entrada para el cliente, simplificando la integración

**Implementación:**
```xml
<choice>
    <when expression="#[payload.operationType == 'bankAccount']">
        <!-- Ruta a proceso de firma para cambio de cuenta bancaria -->
    </when>
    <when expression="#[payload.operationType == 'changePaymentMethod']">
        <!-- Ruta a proceso de firma para cambio de método de pago -->
    </when>
    <otherwise>
        <!-- Ruta a modificaciones generales -->
    </otherwise>
</choice>
```

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito - Cambio de Cuenta Bancaria

```mermaid
sequenceDiagram
    participant C as Cliente Web/Móvil
    participant E as Experience API
    participant JWT as JWT Validation
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    
    C->>E: PUT /contractAccounts<br/>{operationType: "bankAccount"}
    Note over E: Log Request (Kinesis)
    E->>JWT: Validate JWT Token
    JWT-->>E: Token Valid
    Note over E: Set FUA Headers
    Note over E: Transform: Add Headers + IP
    Note over E: Choice: operationType == "bankAccount"
    E->>P: POST /firmas/procesos/CambioCuentaBancaria
    Note over P: Process Digital Signature
    P-->>E: 200 OK {codResult: "0000"}
    E-->>C: 200 OK {result: {...}}
```

#### 2. Flujo con Error de Negocio

```mermaid
sequenceDiagram
    participant C as Cliente Web/Móvil
    participant E as Experience API
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    
    C->>E: PUT /contractAccounts<br/>{operationType: "changePaymentMethod"}
    Note over E: Validate & Transform
    E->>P: POST /firmas/procesos/ModificacionMetodoPago
    Note over P: Business Validation Failed
    P-->>E: 400 Bad Request
    Note over E: Error Handler
    Note over E: Transform Error
    E-->>C: 500 Internal Server Error<br/>{codResult: "0001", msgResult: "..."}
```

#### 3. Flujo con Error Técnico - Timeout

```mermaid
sequenceDiagram
    participant C as Cliente Web/Móvil
    participant E as Experience API
    participant P as Process API<br/>ntgy-newco-p-fua-evo
    
    C->>E: PUT /contractAccounts<br/>{operationType: "bankAccount"}
    Note over E: Validate & Transform
    E->>P: POST /firmas/procesos/CambioCuentaBancaria
    Note over P: Processing...
    Note over P: Processing... (>60s)
    P--xE: Timeout (60s exceeded)
    Note over E: Error Handler
    Note over E: Log Error
    E-->>C: 504 Gateway Timeout<br/>{codResult: "0001", msgResult: "Timeout"}
```

### Configuración del Connector

**Connector:** `HTTP_FUA_PROCESS_NEWCO_EVO`

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

**Properties (e-fua-local-config.yaml, líneas 25-29):**

```yaml
newco-p-fua-evo:
  host: "ntgy-newco-p-fua-evo-des.de-c1.cloudhub.io"
  port: "80"
  timeout: "60000"
  recordtypeid: "0125I00000056X4QAI"
```

### Manejo de Errores

**Error Handler:** `on-error-propagate` (put-contract-accounts.xml, líneas 30-45)

**Estrategia:**
1. **Captura cualquier error** durante el procesamiento
2. **Transforma el error** a formato estándar con código `0001`
3. **Limpia el mensaje** de error (elimina comillas)
4. **Registra el error** en logs con nivel ERROR
5. **Propaga el error** al cliente con estructura consistente

**Transformación de Error:**

```dataweave
%dw 2.0
output application/json
---
{
  "result": {
    "codResult": "0001",
    "msgResult": error.description as String replace /\"/ with("")
  }
}
```

**Códigos de Error:**

| Código | Significado | Acción Recomendada |
|--------|-------------|-------------------|
| `0001` | Error genérico | Revisar logs para detalles específicos |
| `0000` | Éxito | Operación completada correctamente |

### Optimizaciones del Canal

#### 1. Logging Estructurado con Kinesis

**Ubicación:** `put-contract-accounts.xml` (líneas 3-11)

```dataweave
{
  headers: {
    method: attributes.method,
    requestUri: attributes.requestUri,
    correlationId: correlationId
  },
  message: {
    payload: payload
  }
}
```

**Beneficio:** Trazabilidad completa de requests para análisis y debugging

#### 2. Validación JWT Centralizada

**Flow:** `jwt-validation` (referenciado en e-fua.xml, línea 14)

**Beneficio:** Seguridad consistente en todos los endpoints del Experience API

#### 3. Headers de Contexto Reutilizables

**Variable:** `vars.fuaHeaders` (e-fua.xml, líneas 15-37)

**Beneficio:** Evita duplicación de lógica de extracción de headers en cada endpoint

#### 4. Timeout Extendido

**Valor:** 60 segundos (vs 30 segundos estándar)

**Razón:** Procesos de firma digital pueden requerir tiempo adicional de procesamiento

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Cambio de Cuenta Bancaria (Exitoso)

#### cURL

```bash
curl -X PUT "https://ntgy-fua-e-fua-des.de-c1.cloudhub.io/fua/api/contractAccounts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-interaction-id: 12345678-1234-1234-1234-123456789012" \
  -H "x-user-id: user123" \
  -H "x-client-id: client456" \
  -H "x-origin: web" \
  -d '{
    "operationType": "bankAccount",
    "digitalSignatureType": "28023",
    "reason": "01",
    "text": "Cambio de entidad bancaria por mejores condiciones",
    "serviceAccountList": [
      {
        "serviceAccountId": "14479766",
        "clientId": "12345",
        "address": {
          "zipCode": "28023",
          "province": "Madrid",
          "city": "Madrid",
          "way": "Calle Mayor",
          "wayType": "Calle",
          "number": "123",
          "floor": "3",
          "door": "A"
        },
        "billingAccountList": [
          {
            "billingAccountId": "9876543210",
            "isDirectPayment": true,
            "billDeliveryMethod": "ONLINE"
          }
        ]
      }
    ]
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Cambio de cuenta bancaria procesado correctamente. Se ha enviado el proceso de firma digital."
  }
}
```

---

### Ejemplo 2: Modificación de Método de Pago (JavaScript)

#### JavaScript (Fetch API)

```javascript
const updateContractAccount = async () => {
  const response = await fetch(
    'https://ntgy-fua-e-fua-des.de-c1.cloudhub.io/fua/api/contractAccounts',
    {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + localStorage.getItem('jwtToken'),
        'x-interaction-id': generateUUID(),
        'x-user-id': getCurrentUserId(),
        'x-client-id': getCurrentClientId(),
        'x-origin': 'web'
      },
      body: JSON.stringify({
        operationType: 'changePaymentMethod',
        digitalSignatureType: '28001',
        reason: '02',
        text: 'Cambio a transferencia bancaria',
        serviceAccountList: [
          {
            serviceAccountId: '14479766',
            clientId: '67890',
            address: {
              zipCode: '28001',
              province: 'Madrid',
              city: 'Madrid',
              way: 'Gran Vía',
              wayType: 'Calle',
              number: '45'
            },
            billingAccountList: [
              {
                billingAccountId: '9876543210',
                isDirectPayment: false,
                billDeliveryMethod: 'POSTAL'
              }
            ]
          }
        ]
      })
    }
  );

  const data = await response.json();
  
  if (response.ok) {
    console.log('Modificación exitosa:', data);
    return data;
  } else {
    console.error('Error en modificación:', data);
    throw new Error(data.result.msgResult);
  }
};

// Función auxiliar para generar UUID
function generateUUID() {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
    const r = Math.random() * 16 | 0;
    const v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
}

// Uso
updateContractAccount()
  .then(result => {
    alert('Método de pago actualizado correctamente');
  })
  .catch(error => {
    alert('Error: ' + error.message);
  });
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Modificación de método de pago procesada correctamente. Se ha enviado el proceso de firma digital."
  }
}
```

---

### Ejemplo 3: Modificación General (Python)

#### Python (requests)

```python
import requests
import uuid

def update_contract_account_general():
    """
    Actualiza datos generales de cuenta contrato sin firma digital
    """
    url = "https://ntgy-fua-e-fua-des.de-c1.cloudhub.io/fua/api/contractAccounts"
    
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {get_jwt_token()}",
        "x-interaction-id": str(uuid.uuid4()),
        "x-user-id": "user123",
        "x-client-id": "client456",
        "x-origin": "mobile"
    }
    
    payload = {
        "operationType": "updateContactInfo",
        "reason": "03",
        "text": "Actualización de datos de contacto",
        "serviceAccountList": [
            {
                "serviceAccountId": "14479766",
                "clientId": "11223",
                "address": {
                    "zipCode": "08001",
                    "province": "Barcelona",
                    "city": "Barcelona",
                    "way": "Paseo de Gracia",
                    "wayType": "Paseo",
                    "number": "100",
                    "floor": "2",
                    "door": "B"
                },
                "billingAccountList": [
                    {
                        "billingAccountId": "1122334455",
                        "isDirectPayment": True,
                        "billDeliveryMethod": "ONLINE"
                    }
                ]
            }
        ]
    }
    
    try:
        response = requests.put(url, json=payload, headers=headers, timeout=60)
        response.raise_for_status()
        
        result = response.json()
        print(f"✅ Modificación exitosa: {result['result']['msgResult']}")
        return result
        
    except requests.exceptions.Timeout:
        print("❌ Error: Timeout en la solicitud (>60s)")
        raise
    except requests.exceptions.HTTPError as e:
        error_data = e.response.json()
        print(f"❌ Error HTTP {e.response.status_code}: {error_data['result']['msgResult']}")
        raise
    except Exception as e:
        print(f"❌ Error inesperado: {str(e)}")
        raise

def get_jwt_token():
    """Obtiene el token JWT del usuario autenticado"""
    # Implementación específica de autenticación
    return "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# Ejecutar
if __name__ == "__main__":
    try:
        result = update_contract_account_general()
        print(f"✅ Operación completada: {result}")
    except Exception as e:
        print(f"❌ Error: {e}")
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Modificación procesada correctamente"
  }
}
```

---

### Ejemplo 4: Manejo de Error (Timeout)

#### cURL

```bash
curl -X PUT "https://ntgy-fua-e-fua-des.de-c1.cloudhub.io/fua/api/contractAccounts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-interaction-id: 12345678-1234-1234-1234-123456789012" \
  -H "x-user-id: user123" \
  -H "x-origin: web" \
  -d '{
    "operationType": "bankAccount",
    "digitalSignatureType": "28023",
    "serviceAccountList": [
      {
        "accountId": "1234567890",
        "newBankAccount": "ES7921000813610123456789"
      }
    ]
  }'
```

#### Respuesta de Error (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "HTTP POST on resource 'https://ntgy-newco-p-fua-evo-des.de-c1.cloudhub.io:80/api/firmas/procesos/CambioCuentaBancaria' failed: timeout exceeded."
  }
}
```

**Nota:** El mensaje de error incluye detalles técnicos para facilitar el debugging, pero en producción podría simplificarse para el usuario final.
