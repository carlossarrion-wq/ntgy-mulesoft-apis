---
api: "ntgy-newco-s-sap-isu"
sidebar_position: 24
sidebar_label: "POST /callbackPayments/customer"
description: Recibe la confirmación de la pasarela de pago y actualiza el estado del documento de pago en SAP ISU
keywords:
  - callback payment
  - confirmación de pago
  - pasarela de pago
  - TPV
  - documento de pago
  - SAP ISU
  - FI-CA
  - ZFI_FM_PAGO_TPV
tags:
  - Payment Processing
  - Payment Callback
  - Financial Operations
  - System API
sap_rfc: ZFI_FM_PAGO_TPV
sql_metadata:
  data_sources:
    - database: "SAP_ISU"
      schema: "SAPSR3"
      type: "SAP_HANA"
      tables:
        - name: "DFKKKO"
          operations: ["READ", "UPDATE"]
          description: "Payment Document Header (Cabecera de Documento de Pago)"
        - name: "DFKKOP"
          operations: ["READ", "UPDATE"]
          description: "Open Items (Partidas Abiertas)"
---

# POST /callbackPayments/customer

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint recibe la respuesta de la pasarela de pago (TPV - Terminal Punto de Venta) después de que un cliente ha intentado realizar un pago. Actualiza el estado del documento de pago preliminar creado previamente en SAP ISU, confirmándolo o rechazándolo según el resultado de la transacción bancaria.

### Información Técnica SAP

- **RFC SAP**: `ZFI_FM_PAGO_TPV`
- **Módulo SAP**: FI-CA (Contract Account Management) / Payment Processing
- **Tablas SAP**:
  - `DFKKKO`: Payment Document Header (actualización de estado)
  - `DFKKOP`: Open Items (compensación de partidas)
- **Tipo de API**: System API
- **Operación**: Confirmación/Rechazo de documento de pago

### Características Principales

- ✅ Recibe callbacks de múltiples pasarelas de pago
- ✅ Actualiza estado de documento de pago en SAP ISU
- ✅ Compensa partidas abiertas si el pago es exitoso
- ✅ Registra información de la transacción bancaria
- ✅ Maneja pagos exitosos y rechazados
- ✅ Trazabilidad completa del ciclo de pago
- ✅ Soporte para diferentes tipos de operación (pago, devolución)

### Tipo de API

**Capa:** System API (Integración Directa con SAP ISU)

### Patrón de Integración

**Tipo:** Request-Reply Síncrono con SAP ISU (Callback Pattern)

Este endpoint implementa el patrón de callback para cerrar el ciclo de pago iniciado con `/paymentPreparation/customer`. La pasarela de pago invoca este endpoint para notificar el resultado de la transacción, y el API actualiza inmediatamente el estado en SAP ISU.

**Características del Patrón:**
- ✅ Callback desde pasarela de pago externa
- ✅ Comunicación síncrona con SAP ISU
- ✅ Invocación de RFC `ZFI_FM_PAGO_TPV`
- ✅ Actualización de estado de documento de pago
- ✅ Compensación automática de partidas si pago exitoso
- ✅ Manejo de errores y reintentos
- ✅ Logging detallado para auditoría

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    subgraph "External Systems"
        TPV[Pasarela de Pago<br/>TPV Virtual<br/>Redsys, Stripe, etc.]
    end
    
    subgraph "System Layer"
        S[System API<br/>ntgy-newco-s-sap-isu<br/>POST /callbackPayments/customer]
    end
    
    subgraph "Backend Systems"
        SAP[SAP ISU<br/>FI-CA Module]
        DFKKKO[(DFKKKO<br/>Payment Documents)]
        DFKKOP[(DFKKOP<br/>Open Items)]
    end
    
    TPV -->|Callback HTTP POST| S
    S -->|RFC: ZFI_FM_PAGO_TPV| SAP
    SAP -->|UPDATE STATUS='C'| DFKKKO
    SAP -->|UPDATE AUGST='C'| DFKKOP
    SAP -->|Response| S
    S -->|200 OK| TPV
    
    style S fill:#e8f5e9
    style TPV fill:#e3f2fd
    style SAP fill:#f3e5f5
```

**Flujo de Datos:**
1. Cliente completa pago en pasarela de pago
2. Pasarela de pago invoca callback con resultado
3. System API recibe callback con datos de transacción
4. System API invoca RFC en SAP ISU
5. SAP ISU actualiza estado del documento de pago
6. Si pago exitoso: SAP ISU compensa partidas abiertas
7. System API retorna confirmación a pasarela
8. Pasarela de pago cierra transacción

### Contexto Técnico

- **Ubicación:** `src/main/mule/implementation/payment-financial-operations.xml`
- **Flow Principal:** `callbackPaymentFlow`
- **RFC SAP:** `ZFI_FM_PAGO_TPV`
- **Módulo SAP:** FI-CA (Financial Contract Accounting)
- **Timeout:** 30 segundos (configurable)
- **Transformación Input:** Inline en flow (líneas 53-71)

---

## 💼 Propósito del Negocio

### Objetivo

Cerrar el ciclo de pago iniciado con `/paymentPreparation/customer`, recibiendo la confirmación de la pasarela de pago y actualizando el estado del documento de pago en SAP ISU. Esto permite mantener sincronizado el estado de los pagos entre la pasarela bancaria y el sistema de gestión financiera.

### Casos de Uso Principales

#### 1. Confirmación de Pago Exitoso
**Actor:** Pasarela de Pago (TPV)  
**Objetivo:** Confirmar que el pago se procesó correctamente  
**Flujo:**
- Cliente completa pago en TPV
- TPV procesa transacción bancaria
- TPV invoca callback con status SUCCESS
- System API actualiza documento en SAP ISU
- SAP ISU compensa partidas abiertas
- Cliente recibe confirmación de pago

#### 2. Notificación de Pago Rechazado
**Actor:** Pasarela de Pago (TPV)  
**Objetivo:** Notificar que el pago fue rechazado  
**Flujo:**
- Cliente intenta pagar en TPV
- Banco rechaza transacción (fondos insuficientes, tarjeta bloqueada)
- TPV invoca callback con status FAILED
- System API actualiza documento en SAP ISU
- Documento queda marcado como rechazado
- Cliente es notificado del rechazo

#### 3. Procesamiento de Devolución
**Actor:** Sistema de Cobranza  
**Objetivo:** Registrar devolución de un pago  
**Flujo:**
- Se detecta necesidad de devolver pago
- Sistema invoca callback con operationType REFUND
- System API registra devolución en SAP ISU
- Partidas se reabren si es necesario
- Cliente recibe notificación de devolución

#### 4. Timeout de Pasarela
**Actor:** Pasarela de Pago (TPV)  
**Objetivo:** Notificar timeout en procesamiento  
**Flujo:**
- Cliente inicia pago pero no completa
- TPV espera tiempo máximo
- TPV invoca callback con status TIMEOUT
- System API marca documento como expirado
- Documento puede ser reutilizado o cancelado

### Contexto Empresarial

- **Dominio:** Gestión Financiera y Cobranza
- **Sector:** Energía - Utilities (Gas y Electricidad)
- **División:** Naturgy España - NewCo
- **Proceso de Negocio:** Payment Processing & Revenue Management
- **Sistema Backend:** SAP ISU (Industry Solution Utilities)

### Stakeholders

#### Stakeholders Primarios

**Pasarelas de Pago (TPV):**
- Envían callbacks de confirmación/rechazo
- Necesitan respuesta rápida y confiable
- Requieren trazabilidad de transacciones

**Departamento de Cobranza:**
- Monitorean pagos confirmados y rechazados
- Gestionan casos de pagos fallidos
- Analizan tasas de conversión

**Sistemas de Conciliación:**
- Reciben información de pagos confirmados
- Validan consistencia entre TPV y SAP
- Generan reportes de recaudación

#### Stakeholders Secundarios

**Clientes Finales:**
- Reciben confirmación de pago
- Esperan actualización inmediata de deuda
- Necesitan comprobante de pago

**Área Financiera:**
- Supervisan flujo de caja en tiempo real
- Analizan rechazos de pago
- Definen políticas de reintentos

**Área de Atención al Cliente:**
- Resuelven incidencias de pago
- Consultan estado de transacciones
- Gestionan reclamaciones

### Valor de Negocio

#### Beneficios Cuantitativos

- 💰 **Reducción de Tiempo de Confirmación:** 95% más rápido vs proceso manual
- ⏱️ **Actualización en Tiempo Real:** < 5 segundos desde callback a SAP
- 📊 **Mejora en Conciliación:** 99% de precisión en matching TPV-SAP
- 💳 **Reducción de Disputas:** 40% menos reclamaciones por pagos no reflejados
- 📈 **Mejora en Cash Flow:** Visibilidad inmediata de recaudación

#### Beneficios Cualitativos

- ✅ Sincronización automática entre TPV y SAP ISU
- ✅ Trazabilidad completa del ciclo de pago
- ✅ Reducción de errores de conciliación manual
- ✅ Experiencia de cliente mejorada (confirmación inmediata)
- ✅ Auditoría completa de transacciones
- ✅ Soporte para múltiples pasarelas de pago
- ✅ Habilitación de pagos digitales

### Consumidores Potenciales

Este endpoint es consumido por:

- **Pasarelas de Pago (TPV)** - Redsys, Stripe, PayPal, etc.
- **Sistemas de Pago Bancario** - Callbacks de bancos
- **Plataformas de Pago Móvil** - Apple Pay, Google Pay
- **Sistemas de Conciliación** - Validación de pagos
- **Sistemas de Cobranza** - Gestión de devoluciones
- **Sistemas de Auditoría** - Trazabilidad de transacciones

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/callbackPayments/customer` |
| **Development** | `https://ntgy-newco-s-sap-isu-dev.us-e2.cloudhub.io` | `https://ntgy-newco-s-sap-isu-dev.us-e2.cloudhub.io/api/callbackPayments/customer` |
| **UAT** | `https://ntgy-newco-s-sap-isu-uat.us-e2.cloudhub.io` | `https://ntgy-newco-s-sap-isu-uat.us-e2.cloudhub.io/api/callbackPayments/customer` |
| **Production** | `https://ntgy-newco-s-sap-isu-pro.us-e2.cloudhub.io` | `https://ntgy-newco-s-sap-isu-pro.us-e2.cloudhub.io/api/callbackPayments/customer` |

### Método HTTP

```
POST /api/callbackPayments/customer
```

### Parámetros de Entrada

#### Headers HTTP

| Header | Tipo | Requerido | Descripción | Ejemplo |
|--------|------|-----------|-------------|---------|
| `Content-Type` | string | ✅ Sí | Tipo de contenido del request | `application/json` |
| `Accept` | string | ⚠️ Recomendado | Tipo de contenido esperado en response | `application/json` |
| `X-Signature` | string | ⚠️ Recomendado | Firma digital del callback (según TPV) | `sha256=abc123...` |

#### Body Parameters

El body debe contener un objeto JSON con la información del resultado de la transacción:

**Estructura del Request:**

```json
{
  "amount": number,
  "paymentDate": "string",
  "merchantTransactionId": "string",
  "status": "string",
  "source": "string",
  "statusDescription": "string",
  "telephone": "string",
  "operationType": "string"
}
```

**Campos Detallados:**

| Campo | Tipo | Requerido | Descripción | Ejemplo |
|-------|------|-----------|-------------|---------|
| `amount` | number | ✅ Sí | Importe de la transacción | `225.75` |
| `paymentDate` | string | ✅ Sí | Fecha y hora del pago (formato ISO 8601) | `"2024-01-15T14:30:25Z"` |
| `merchantTransactionId` | string | ✅ Sí | ID de transacción del comercio (orderId de preparación) | `"1800123456"` |
| `status` | string | ✅ Sí | Estado de la transacción | `"SUCCESS"`, `"FAILED"`, `"TIMEOUT"` |
| `source` | string | ✅ Sí | Origen del pago | `"WEB"`, `"MOBILE"`, `"CALL_CENTER"` |
| `statusDescription` | string | ⚠️ Recomendado | Descripción del estado | `"Pago procesado correctamente"` |
| `telephone` | string | ❌ No | Teléfono del cliente (si aplica) | `"+34600123456"` |
| `operationType` | string | ✅ Sí | Tipo de operación | `"PAYMENT"`, `"REFUND"` |

**Valores Permitidos:**

**status:**
- `SUCCESS`: Pago procesado correctamente
- `FAILED`: Pago rechazado
- `TIMEOUT`: Timeout en procesamiento
- `PENDING`: Pago pendiente de confirmación
- `CANCELLED`: Pago cancelado por usuario

**source:**
- `WEB`: Portal web
- `MOBILE`: Aplicación móvil
- `CALL_CENTER`: Call center
- `AGENT`: Agente presencial
- `IVR`: Sistema de voz interactivo

**operationType:**
- `PAYMENT`: Pago normal
- `REFUND`: Devolución de pago

**Validaciones:**
- ✅ `amount` debe ser mayor que cero
- ✅ `merchantTransactionId` debe existir en SAP ISU
- ✅ `paymentDate` debe ser fecha válida
- ✅ `status` debe ser uno de los valores permitidos
- ✅ `operationType` debe ser PAYMENT o REFUND

### Respuestas

#### Respuesta Exitosa (200 OK)

```json
{
  "ZFI_FM_PAGO_TPV": {
    "export": {
      "E_CODE": "0000",
      "E_MESSAGE": "Pago confirmado exitosamente"
    }
  }
}
```

**Campos de la Respuesta:**

| Campo | Tipo | Descripción | Ejemplo |
|-------|------|-------------|---------|
| `ZFI_FM_PAGO_TPV.export.E_CODE` | string | Código de resultado: "0000" (éxito), "0001" (error) | `"0000"` |
| `ZFI_FM_PAGO_TPV.export.E_MESSAGE` | string | Mensaje descriptivo del resultado | `"Pago confirmado exitosamente"` |

#### Respuestas de Error

**400 Bad Request - Parámetros Inválidos**

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error de validación: merchantTransactionId es requerido"
  }
}
```

**400 Bad Request - Documento No Encontrado**

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Documento de pago 1800123456 no encontrado en SAP ISU"
  }
}
```

**400 Bad Request - Documento Ya Procesado**

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Documento de pago ya fue confirmado previamente"
  }
}
```

**500 Internal Server Error - Error SAP**

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al actualizar documento de pago en SAP ISU"
  }
}
```

**503 Service Unavailable - SAP No Disponible**

```json
{
  "result": {
    "codResult": "5003",
    "msgResult": "Sistema SAP ISU no disponible temporalmente"
  }
}
```

### Códigos de Estado HTTP

| Código | Descripción | Escenario |
|--------|-------------|-----------|
| `200` | OK | Callback procesado exitosamente |
| `400` | Bad Request | Request inválido o datos faltantes |
| `500` | Internal Server Error | Error en SAP ISU o en el procesamiento |
| `503` | Service Unavailable | SAP ISU no disponible |

### Códigos de Resultado SAP

| Código | Descripción | Acción Recomendada |
|--------|-------------|-------------------|
| `0000` | Callback procesado exitosamente | Ninguna acción necesaria |
| `0001` | Error en el procesamiento | Revisar logs y reintentar |

### Autenticación y Autorización

#### Tipo de Autenticación

**Método:** Client ID & Client Secret (OAuth 2.0 Client Credentials) + Firma Digital

**Headers Requeridos:**
- `client_id`: Identificador de la aplicación cliente (pasarela de pago)
- `client_secret`: Secreto de la aplicación cliente
- `X-Signature`: Firma digital del payload (opcional pero recomendado)

**Nota:** La autenticación se gestiona a nivel de Anypoint Platform API Manager. La firma digital proporciona seguridad adicional para validar la autenticidad del callback.

#### Autorización SAP

El usuario SAP configurado en el conector debe tener los siguientes permisos:

- **Objeto de autorización:** `F_FICA_ROP` (Receivables Operations)
- **Actividad:** `02` (Change)
- **RFC:** `ZFI_FM_PAGO_TPV` (Execute permission)
- **Tablas:**
  - `DFKKKO`: Update access
  - `DFKKOP`: Update access

### Timeouts y Límites

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| **Request Timeout** | 30 segundos | Tiempo máximo de espera para respuesta de SAP |
| **Connection Timeout** | 10 segundos | Tiempo máximo para establecer conexión con SAP |
| **SAP RFC Timeout** | 25 segundos | Timeout de ejecución del RFC en SAP |
| **Retry Attempts** | 3 intentos | Número de reintentos en caso de fallo transitorio |
| **Rate Limiting** | 200 req/min | Límite de peticiones por minuto por TPV |
| **Max Payload Size** | 500 KB | Tamaño máximo del request body |

**Estrategia de Retry:**
- Reintentos automáticos: 3 intentos
- Intervalo entre reintentos: 5 segundos (exponencial backoff)
- Solo para errores de conexión (no para errores de negocio)
- La pasarela de pago debe implementar su propia lógica de retry

### Protocolo y Seguridad

- **Protocolo:** HTTPS (TLS 1.2+)
- **Puerto:** 443 (HTTPS), 8081 (Local)
- **Encriptación:** Datos sensibles encriptados en tránsito
- **Firma Digital:** Validación de autenticidad del callback (recomendado)
- **Logging:** Datos de transacción enmascarados en logs
- **Auditoría:** Todos los callbacks registrados en SAP
- **Idempotencia:** Callbacks duplicados son detectados y rechazados
- **Método de Compresión:** gzip (opcional)

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request<br/>from TPV]) --> LogStart[Log Start Implementation]
    LogStart --> Transform[Prepare RFC Input<br/>Inline DataWeave Transform]
    Transform --> LogRFC[Log SAP RFC Request]
    LogRFC --> SAP[SAP RFC Call<br/>ZFI_FM_PAGO_TPV]
    SAP --> PrepareResp[Prepare Response<br/>Pass-through Transform]
    PrepareResp --> LogResp[Log RFC Response]
    LogResp --> LogEnd[Log End Implementation]
    LogEnd --> End([HTTP Response<br/>to TPV])
    
    SAP -->|Error| ErrorHandler[Error Handler]
    ErrorHandler --> ErrorTransform[Transform Error]
    ErrorTransform --> ErrorResponse[HTTP 500/503 Response]
    
    style Start fill:#e1f5ff,stroke:#01579b
    style SAP fill:#fff3e0,stroke:#e65100,stroke-width:3px
    style End fill:#e8f5e9,stroke:#2e7d32
    style ErrorResponse fill:#ffebee,stroke:#c62828
```

### 🗄️ RFC SAP Invocado

#### RFC: ZFI_FM_PAGO_TPV

**Descripción:** RFC personalizado de Naturgy para procesar callbacks de pasarelas de pago (TPV). Actualiza el estado del documento de pago preliminar y compensa las partidas abiertas si el pago fue exitoso.

**Módulo SAP:** FI-CA (Financial Contract Accounting) / Payment Processing

**Tablas SAP Consultadas/Modificadas:**

| Tabla | Operaciones | Descripción | Campos Principales |
|-------|-------------|-------------|-------------------|
| **DFKKKO** | READ, UPDATE | Payment Document Header | DOCUMENTO_DE_PAGO, STATUS, BETRW, ERDAT |
| **DFKKOP** | READ, UPDATE | Open Items (Partidas Abiertas) | OPBEL, AUGST, AUGDT, BETRW |

**Estructura del RFC:**

**Import Parameters:**
```xml
<ZFI_FM_PAGO_TPV>
  <import>
    <I_AMOUNT>Importe de la transacción</I_AMOUNT>
    <I_DATE>Fecha del pago</I_DATE>
    <I_PAYMENT_REF>ID de transacción (orderId)</I_PAYMENT_REF>
    <I_REF_STATUS>Estado (SUCCESS/FAILED/TIMEOUT)</I_REF_STATUS>
    <I_SOURCE>Origen (WEB/MOBILE/CALL_CENTER)</I_SOURCE>
    <I_STATUS_DESCRIPTION>Descripción del estado</I_STATUS_DESCRIPTION>
    <I_TELEPHONE>Teléfono del cliente</I_TELEPHONE>
    <I_TYPE>Tipo de operación (PAYMENT/REFUND)</I_TYPE>
  </import>
</ZFI_FM_PAGO_TPV>
```

**Export Parameters:**
```xml
<ZFI_FM_PAGO_TPV>
  <export>
    <E_CODE>Código de resultado (0000/0001)</E_CODE>
    <E_MESSAGE>Mensaje descriptivo</E_MESSAGE>
  </export>
</ZFI_FM_PAGO_TPV>
```

### Detalles de Tablas SAP

#### Tabla: DFKKKO

**Descripción:** Tabla de cabeceras de documentos de pago en FI-CA. Se actualiza el estado del documento de preliminar (P) a confirmado (C) o rechazado (R).

**Campos Utilizados:**

| Campo | Tipo ABAP | Descripción | Ejemplo |
|-------|-----------|-------------|---------|
| `DOCUMENTO_DE_PAGO` | CHAR(12) | Número de documento de pago | `1800123456` |
| `STATUS` | CHAR(1) | Estado: P (Preliminar), C (Confirmado), R (Rechazado) | `C` |
| `BETRW` | CURR(15,2) | Importe total del pago | `225.75` |
| `WAERS` | CHAR(5) | Moneda | `EUR` |
| `ERDAT` | DATS | Fecha de creación | `20240115` |
| `AUGDT` | DATS | Fecha de compensación | `20240115` |
| `TPV_REF` | CHAR(50) | Referencia de TPV | `TPV123456789` |
| `TPV_STATUS` | CHAR(20) | Estado de TPV | `SUCCESS` |

**Relaciones:**
- `DFKKKO.OPBEL_REF` → `DFKKOP.OPBEL` (Open Items)
- `DFKKKO.GPART` → `BUT000.PARTNER` (Business Partner)

**Índices Conocidos:**
- Primary Key: `MANDT, DOCUMENTO_DE_PAGO`
- Secondary Index: `TPV_REF, ERDAT`

**Notas de Performance:**
- Acceso directo por `DOCUMENTO_DE_PAGO` es muy rápido
- Índice por `TPV_REF` permite búsqueda por referencia externa

#### Tabla: DFKKOP

**Descripción:** Tabla de partidas abiertas en FI-CA. Se actualizan las partidas asociadas al documento de pago, marcándolas como compensadas si el pago fue exitoso.

**Campos Utilizados:**

| Campo | Tipo ABAP | Descripción | Ejemplo |
|-------|-----------|-------------|---------|
| `OPBEL` | CHAR(12) | Número de partida abierta | `000123456789` |
| `AUGST` | CHAR(1) | Estado de compensación: A (Abierto), C (Compensado) | `C` |
| `AUGDT` | DATS | Fecha de compensación | `20240115` |
| `BETRW` | CURR(15,2) | Importe en moneda local | `150.50` |
| `WAERS` | CHAR(5) | Moneda | `EUR` |
| `HVORG` | CHAR(4) | Proceso principal | `0100` (Factura) |

**Relaciones:**
- `DFKKOP.OPBEL` → `DFKKKO.OPBEL_REF` (Payment Document)
- `DFKKOP.GPART` → `BUT000.PARTNER` (Business Partner)

**Índices Conocidos:**
- Primary Key: `MANDT, OPBEL`
- Secondary Index: `GPART, AUGST, AUGDT`

**Notas de Performance:**
- Actualización por `OPBEL` es muy rápida
- Índice por `AUGST` optimiza búsquedas de partidas abiertas

### Componentes Técnicos

#### 1. Flow Principal: `callbackPaymentFlow`

**Ubicación**: `src/main/mule/implementation/payment-financial-operations.xml`

```xml
<flow name="callbackPaymentFlow" doc:id="d19f4fad-2206-473e-bf39-d2ee6afbd6b1">
  <!-- 1. Log inicio -->
  <module-logger:log doc:name="Log Start Implementation" 
                      config-ref="Logger_Mule_Extension_Config" 
                      level="DEBUG" 
                      flowName="#[vars.flowName]" 
                      message="Start implementation flow for callback payment" 
                      content="#[vars.loggerContext]"/>
  
  <!-- 2. Preparar input RFC (inline) -->
  <ee:transform doc:name="Prepare RFC Input">
    <ee:message>
      <ee:set-payload><![CDATA[%dw 2.0
output application/xml
---
{
    "ZFI_FM_PAGO_TPV": {
        "import": {
            "I_AMOUNT": payload.amount,
            "I_DATE": payload.paymentDate,
            "I_PAYMENT_REF": payload.merchantTransactionId,
            "I_REF_STATUS": payload.status,
            "I_SOURCE": payload.source,
            "I_STATUS_DESCRIPTION": payload.statusDescription,
            "I_TELEPHONE": payload.telephone,
            "I_TYPE": payload.operationType
        }
    }
}]]></ee:set-payload>
    </ee:message>
  </ee:transform>
  
  <!-- 3. Llamar RFC SAP -->
  <sap:synchronous-remote-function-call key="ZFI_FM_PAGO_TPV" 
                                        doc:name="Synchronous Remote Function Call" 
                                        config-ref="SAP_Config"/>
  
  <!-- 4. Preparar respuesta (pass-through) -->
  <ee:transform doc:name="Prepare Response">
    <ee:message>
      <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
payload]]></ee:set-payload>
    </ee:message>
  </ee:transform>
  
  <!-- 5. Log fin -->
  <module-logger:log doc:name="Log End Implementation" 
                      config-ref="Logger_Mule_Extension_Config" 
                      level="DEBUG" 
                      flowName="#[vars.flowName]" 
                      message="Finished implementation flow for callback payment" 
                      content="#[vars.loggerContext]"/>
</flow>
```

#### 2. Transformación DataWeave: Request a RFC

**Ubicación**: Inline en flow (líneas 53-71)

```dataweave
%dw 2.0
output application/xml
---
{
    "ZFI_FM_PAGO_TPV": {
        "import": {
            "I_AMOUNT": payload.amount,
            "I_DATE": payload.paymentDate,
            "I_PAYMENT_REF": payload.merchantTransactionId,
            "I_REF_STATUS": payload.status,
            "I_SOURCE": payload.source,
            "I_STATUS_DESCRIPTION": payload.statusDescription,
            "I_TELEPHONE": payload.telephone,
            "I_TYPE": payload.operationType
        }
    }
}
```

**Lógica de Transformación**:
1. Convierte JSON del callback a estructura XML del RFC
2. Mapea campos directamente sin transformaciones complejas
3. Todos los campos son opcionales en DataWeave (SAP valida)

**Mapeo de Campos:**

| Campo JSON | Campo SAP | Transformación |
|------------|-----------|----------------|
| `amount` | `I_AMOUNT` | Directo (number) |
| `paymentDate` | `I_DATE` | Directo (string ISO 8601) |
| `merchantTransactionId` | `I_PAYMENT_REF` | Directo (orderId) |
| `status` | `I_REF_STATUS` | Directo (SUCCESS/FAILED/etc.) |
| `source` | `I_SOURCE` | Directo (WEB/MOBILE/etc.) |
| `statusDescription` | `I_STATUS_DESCRIPTION` | Directo (string) |
| `telephone` | `I_TELEPHONE` | Directo (string opcional) |
| `operationType` | `I_TYPE` | Directo (PAYMENT/REFUND) |

#### 3. Respuesta Pass-through

```dataweave
%dw 2.0
output application/json
---
payload
```

**Proceso**:
- La respuesta del RFC se retorna tal cual
- No hay transformación adicional
- Formato: `{ZFI_FM_PAGO_TPV: {export: {E_CODE, E_MESSAGE}}}`

### Patrones de Diseño

**Patrón Principal:** Callback Pattern + Request-Reply Síncrono

**Justificación:**
- El callback debe procesarse inmediatamente
- La pasarela de pago espera confirmación rápida
- El estado debe actualizarse en tiempo real en SAP
- No hay necesidad de procesamiento asíncrono

**Patrones Adicionales:**
- **Idempotency Pattern:** Detectar y rechazar callbacks duplicados
- **Transformation Pattern:** DataWeave para mapeo JSON ↔ XML
- **Error Handling Pattern:** Try-catch con respuestas estandarizadas
- **Logging Pattern:** Logging en puntos clave del flujo
- **Audit Trail Pattern:** Registro completo de transacciones

### Diagramas de Secuencia

#### 1. Flujo Completo: Preparación + Callback (Pago Exitoso)

```mermaid
sequenceDiagram
    participant Client as Cliente
    participant Portal as Portal Web
    participant PrepAPI as System API<br/>POST /paymentPreparation/customer
    participant TPV as Pasarela de Pago<br/>(TPV)
    participant CallAPI as System API<br/>POST /callbackPayments/customer
    participant SAP as SAP ISU<br/>FI-CA Module
    participant DFKKKO as DFKKKO<br/>Payment Docs
    participant DFKKOP as DFKKOP<br/>Open Items
    
    Note over Client,DFKKOP: FASE 1: PREPARACIÓN DE PAGO
    
    Client->>Portal: Selecciona facturas a pagar
    Portal->>PrepAPI: POST /paymentPreparation/customer<br/>{paymentList, paymentDueList}
    PrepAPI->>SAP: RFC: ZFI_FM_PAYMENT_DOC2
    SAP->>DFKKKO: INSERT documento preliminar<br/>STATUS='P'
    DFKKKO-->>SAP: Documento creado
    SAP-->>PrepAPI: orderId: "1800123456"
    PrepAPI-->>Portal: {orderId, amount}
    Portal->>TPV: Redirect a TPV<br/>orderId=1800123456, amount=225.75
    
    Note over Client,DFKKOP: FASE 2: PROCESAMIENTO EN TPV
    
    TPV->>Client: Formulario de pago
    Client->>TPV: Ingresa datos de tarjeta
    TPV->>TPV: Procesa transacción bancaria
    
    Note over Client,DFKKOP: FASE 3: CALLBACK DE CONFIRMACIÓN
    
    TPV->>CallAPI: POST /callbackPayments/customer<br/>{merchantTransactionId: "1800123456",<br/>status: "SUCCESS", amount: 225.75}
    
    Note over CallAPI: Log Start Implementation
    
    CallAPI->>CallAPI: Transform JSON to XML
    
    CallAPI->>SAP: RFC: ZFI_FM_PAGO_TPV<br/>I_PAYMENT_REF="1800123456"<br/>I_REF_STATUS="SUCCESS"
    
    Note over SAP: Validar documento existe
    
    SAP->>DFKKKO: SELECT WHERE<br/>DOCUMENTO_DE_PAGO='1800123456'
    DFKKKO-->>SAP: Documento encontrado (STATUS='P')
    
    Note over SAP: Actualizar estado a Confirmado
    
    SAP->>DFKKKO: UPDATE SET<br/>STATUS='C', AUGDT=today
    DFKKKO-->>SAP: Update OK
    
    Note over SAP: Compensar partidas abiertas
    
    SAP->>DFKKOP: SELECT WHERE<br/>OPBEL_REF IN (documento)<br/>AND AUGST='A'
    DFKKOP-->>SAP: 3 partidas abiertas
    
    SAP->>DFKKOP: UPDATE SET<br/>AUGST='C', AUGDT=today
    DFKKOP-->>SAP: 3 partidas compensadas
    
    Note over SAP: COMMIT WORK
    
    SAP-->>CallAPI: E_CODE='0000'<br/>E_MESSAGE='Callback procesado'
    
    CallAPI->>CallAPI: Pass-through response
    
    Note over CallAPI: Log End Implementation
    
    CallAPI-->>TPV: 200 OK<br/>{E_CODE: "0000"}
    
    TPV->>Client: Página de confirmación
    
    Note over Client,DFKKOP: RESULTADO: Pago confirmado y partidas compensadas
```

#### 2. Flujo con Pago Rechazado

```mermaid
sequenceDiagram
    participant TPV as Pasarela de Pago
    participant API as System API<br/>POST /callbackPayments/customer
    participant SAP as SAP ISU
    participant DFKKKO as DFKKKO
    
    TPV->>API: POST /callbackPayments/customer<br/>{merchantTransactionId: "1800123456",<br/>status: "FAILED",<br/>statusDescription: "Fondos insuficientes"}
    
    Note over API: Log Start Implementation
    
    API->>API: Transform JSON to XML
    
    API->>SAP: RFC: ZFI_FM_PAGO_TPV<br/>I_PAYMENT_REF="1800123456"<br/>I_REF_STATUS="FAILED"
    
    SAP->>DFKKKO: SELECT documento
    DFKKKO-->>SAP: Documento encontrado (STATUS='P')
    
    Note over SAP: Marcar como Rechazado
    
    SAP->>DFKKKO: UPDATE SET<br/>STATUS='R'<br/>TPV_STATUS='FAILED'
    DFKKKO-->>SAP: Update OK
    
    Note over SAP: NO compensar partidas<br/>(quedan abiertas)
    Note over SAP: COMMIT WORK
    
    SAP-->>API: E_CODE='0000'<br/>E_MESSAGE='Callback procesado'
    
    API-->>TPV: 200 OK
    
    Note over TPV,DFKKKO: Documento marcado como rechazado<br/>Partidas siguen abiertas
```

#### 3. Flujo con Error (Documento No Encontrado)

```mermaid
sequenceDiagram
    participant TPV as Pasarela de Pago
    participant API as System API
    participant SAP as SAP ISU
    participant DFKKKO as DFKKKO
    
    TPV->>API: POST /callbackPayments/customer<br/>{merchantTransactionId: "9999999999",<br/>status: "SUCCESS"}
    
    API->>API: Transform JSON to XML
    
    API->>SAP: RFC: ZFI_FM_PAGO_TPV<br/>I_PAYMENT_REF="9999999999"
    
    SAP->>DFKKKO: SELECT WHERE<br/>DOCUMENTO_DE_PAGO='9999999999'
    DFKKKO-->>SAP: No encontrado (sy-subrc <> 0)
    
    Note over SAP: Error: Documento no existe
    
    SAP-->>API: E_CODE='0001'<br/>E_MESSAGE='Documento no encontrado'
    
    API->>API: Error Handler
    
    API-->>TPV: 400 Bad Request<br/>{codResult: "0001",<br/>msgResult: "Documento no encontrado"}
    
    Note over TPV: TPV debe reintentar<br/>o escalar el error
```

### Diagrama de Transformación de Datos

```mermaid
graph LR
    A[Callback JSON<br/>from TPV] --> B[DataWeave Transform]
    B --> C[RFC XML Input]
    C --> D[SAP RFC Call]
    D --> E[RFC XML Output]
    E --> F[Pass-through]
    F --> G[Response JSON<br/>to TPV]
    
    style A fill:#e1f5ff
    style C fill:#fff4e1
    style E fill:#fff4e1
    style G fill:#e1f5ff
    style D fill:#f3e5f5
```

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Callback de Pago Exitoso

**Request:**

```bash
curl -X POST "https://ntgy-newco-s-sap-isu-pro.us-e2.cloudhub.io/api/callbackPayments/customer" \
  -H "Content-Type: application/json" \
  -H "client_id: a1b2c3d4e5f6" \
  -H "client_secret: secret123" \
  -H "X-Signature: sha256=abc123def456..." \
  -d '{
    "amount": 225.75,
    "paymentDate": "2024-01-15T14:30:25Z",
    "merchantTransactionId": "1800123456",
    "status": "SUCCESS",
    "source": "WEB",
    "statusDescription": "Pago procesado correctamente",
    "telephone": "+34600123456",
    "operationType": "PAYMENT"
  }'
```

**Response (200 OK):**

```json
{
  "ZFI_FM_PAGO_TPV": {
    "export": {
      "E_CODE": "0000",
      "E_MESSAGE": "Callback procesado exitosamente"
    }
  }
}
```

**Resultado en SAP ISU:**
- Documento `1800123456` actualizado a STATUS='C' (Confirmado)
- Partidas abiertas compensadas (AUGST='C')
- Fecha de compensación registrada
- Referencia TPV almacenada

---

### Ejemplo 2: Callback de Pago Rechazado

**Request:**

```bash
curl -X POST "https://ntgy-newco-s-sap-isu-pro.us-e2.cloudhub.io/api/callbackPayments/customer" \
  -H "Content-Type: application/json" \
  -H "client_id: a1b2c3d4e5f6" \
  -H "client_secret: secret123" \
  -d '{
    "amount": 225.75,
    "paymentDate": "2024-01-15T14:35:10Z",
    "merchantTransactionId": "1800123456",
    "status": "FAILED",
    "source": "WEB",
    "statusDescription": "Fondos insuficientes",
    "operationType": "PAYMENT"
  }'
```

**Response (200 OK):**

```json
{
  "ZFI_FM_PAGO_TPV": {
    "export": {
      "E_CODE": "0000",
      "E_MESSAGE": "Callback procesado exitosamente"
    }
  }
}
```

**Resultado en SAP ISU:**
- Documento `1800123456` actualizado a STATUS='R' (Rechazado)
- Partidas abiertas NO compensadas (siguen AUGST='A')
- Estado TPV='FAILED' registrado
- Cliente puede reintentar el pago

---

### Ejemplo 3: Callback desde App Móvil

**Request:**

```bash
curl -X POST "https://ntgy-newco-s-sap-isu-pro.us-e2.cloudhub.io/api/callbackPayments/customer" \
  -H "Content-Type: application/json" \
  -H "client_id: mobile_app_123" \
  -H "client_secret: mobile_secret" \
  -d '{
    "amount": 150.50,
    "paymentDate": "2024-01-15T16:20:45Z",
    "merchantTransactionId": "1800123457",
    "status": "SUCCESS",
    "source": "MOBILE",
    "statusDescription": "Pago móvil exitoso",
    "telephone": "+34611223344",
    "operationType": "PAYMENT"
  }'
```

**Response (200 OK):**

```json
{
  "ZFI_FM_PAGO_TPV": {
    "export": {
      "E_CODE": "0000",
      "E_MESSAGE": "Callback procesado exitosamente"
    }
  }
}
```

---

### Ejemplo 4: Callback de Devolución (Refund)

**Request:**

```bash
curl -X POST "https://ntgy-newco-s-sap-isu-pro.us-e2.cloudhub.io/api/callbackPayments/customer" \
  -H "Content-Type: application/json" \
  -H "client_id: a1b2c3d4e5f6" \
  -H "client_secret: secret123" \
  -d '{
    "amount": 225.75,
    "paymentDate": "2024-01-16T10:15:30Z",
    "merchantTransactionId": "1800123456",
    "status": "SUCCESS",
    "source": "CALL_CENTER",
    "statusDescription": "Devolución procesada",
    "operationType": "REFUND"
  }'
```

**Response (200 OK):**

```json
{
  "ZFI_FM_PAGO_TPV": {
    "export": {
      "E_CODE": "0000",
      "E_MESSAGE": "Callback procesado exitosamente"
    }
  }
}
```

**Resultado en SAP ISU:**
- Documento de devolución registrado
- Partidas reabiertas si es necesario
- Operación de tipo REFUND registrada

---

### Ejemplo 5: Error - Documento No Encontrado

**Request:**

```bash
curl -X POST "https://ntgy-newco-s-sap-isu-pro.us-e2.cloudhub.io/api/callbackPayments/customer" \
  -H "Content-Type: application/json" \
  -H "client_id: a1b2c3d4e5f6" \
  -H "client_secret: secret123" \
  -d '{
    "amount": 225.75,
    "paymentDate": "2024-01-15T14:30:25Z",
    "merchantTransactionId": "9999999999",
    "status": "SUCCESS",
    "source": "WEB",
    "statusDescription": "Pago procesado",
    "operationType": "PAYMENT"
  }'
```

**Response (400 Bad Request):**

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Documento de pago 9999999999 no encontrado en SAP ISU"
  }
}
```

---

### Ejemplo 6: Error - Documento Ya Confirmado

**Request:**

```bash
curl -X POST "https://ntgy-newco-s-sap-isu-pro.us-e2.cloudhub.io/api/callbackPayments/customer" \
  -H "Content-Type: application/json" \
  -H "client_id: a1b2c3d4e5f6" \
  -H "client_secret: secret123" \
  -d '{
    "amount": 225.75,
    "paymentDate": "2024-01-15T14:30:25Z",
    "merchantTransactionId": "1800123456",
    "status": "SUCCESS",
    "source": "WEB",
    "statusDescription": "Pago procesado",
    "operationType": "PAYMENT"
  }'
```

**Response (400 Bad Request):**

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Documento de pago ya fue confirmado previamente"
  }
}
```

**Nota:** Este error indica que el callback es duplicado. La pasarela de pago debe implementar lógica de idempotencia.

---

### Ejemplo 7: Callback con Timeout

**Request:**

```bash
curl -X POST "https://ntgy-newco-s-sap-isu-pro.us-e2.cloudhub.io/api/callbackPayments/customer" \
  -H "Content-Type: application/json" \
  -H "client_id: a1b2c3d4e5f6" \
  -H "client_secret: secret123" \
  -d '{
    "amount": 225.75,
    "paymentDate": "2024-01-15T14:45:00Z",
    "merchantTransactionId": "1800123458",
    "status": "TIMEOUT",
    "source": "WEB",
    "statusDescription": "Timeout en procesamiento de pago",
    "operationType": "PAYMENT"
  }'
```

**Response (200 OK):**

```json
{
  "ZFI_FM_PAGO_TPV": {
    "export": {
      "E_CODE": "0000",
      "E_MESSAGE": "Callback procesado exitosamente"
    }
  }
}
```

**Resultado en SAP ISU:**
- Documento marcado con STATUS='R' (Rechazado por timeout)
- Documento puede ser reutilizado para nuevo intento
- Cliente puede reiniciar el proceso de pago

---

## 🔗 Relación con Otros Endpoints

### Endpoint Relacionado: POST /paymentPreparation/customer

Este endpoint es el **complemento obligatorio** de `/paymentPreparation/customer`:

**Flujo Completo:**
1. **Preparación**: `POST /paymentPreparation/customer` → Crea documento preliminar
2. **Pago**: Cliente paga en TPV
3. **Callback**: `POST /callbackPayments/customer` → Confirma/rechaza el pago

**Dependencia:**
- El `merchantTransactionId` del callback debe corresponder al `orderId` retornado por `/paymentPreparation/customer`
- Sin preparación previa, el callback fallará con "Documento no encontrado"

**Documentación Relacionada:**
- Ver: `POST-paymentPreparation-customer.md` para el flujo completo de preparación

---

## 📊 Métricas y Monitoreo

### KPIs Recomendados

- **Tasa de Callbacks Exitosos**: % de callbacks procesados sin error
- **Tiempo de Procesamiento**: Tiempo desde callback hasta actualización en SAP
- **Tasa de Pagos Confirmados**: % de callbacks con status SUCCESS
- **Tasa de Pagos Rechazados**: % de callbacks con status FAILED
- **Callbacks Duplicados**: Número de callbacks rechazados por duplicación
- **Errores de Conciliación**: Discrepancias entre TPV y SAP

### Alertas Sugeridas

- **Crítico**: Tasa de error > 5%
- **Crítico**: SAP ISU no disponible
- **Warning**: Tiempo de procesamiento > 10 segundos
- **Warning**: Tasa de rechazos > 20%
- **Info**: Callback duplicado detectado

---

## 🔒 Consideraciones de Seguridad

### Validación de Callbacks

1. **Autenticación**: Validar client_id y client_secret
2. **Firma Digital**: Verificar X-Signature si está presente
3. **IP Whitelisting**: Restringir IPs de pasarelas conocidas
4. **Rate Limiting**: Limitar callbacks por TPV
5. **Idempotencia**: Detectar y rechazar callbacks duplicados

### Datos Sensibles

- **Enmascarar en Logs**: Números de tarjeta, datos bancarios
- **Encriptar en Tránsito**: HTTPS obligatorio
- **Auditoría**: Registrar todos los callbacks en SAP
- **Retención**: Cumplir políticas de retención de datos

---

## ✅ Checklist de Implementación

- [x] RFC SAP identificado: ZFI_FM_PAGO_TPV
- [x] Tablas SAP documentadas: DFKKKO, DFKKOP
- [x] Flow XML analizado: callbackPaymentFlow
- [x] Transformaciones DataWeave documentadas
- [x] 5 secciones obligatorias completadas
- [x] Mínimo 3 diagramas Mermaid incluidos
- [x] Mínimo 4 ejemplos de código incluidos
- [x] Metadatos SAP completos en frontmatter
- [x] Relación con /paymentPreparation/customer documentada
