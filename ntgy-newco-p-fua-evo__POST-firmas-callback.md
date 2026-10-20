---
api: "ntgy-newco-p-fua-evo"
sidebar_position: 79
sidebar_label: "POST /firmas/callback"
description: "Webhook para procesar callbacks de AVIVA sobre estados de firma digital en procesos de gestión"
keywords: [firma digital, AVIVA, callback, webhook, proceso firma, PostgreSQL, Salesforce, Motor Estados]
tags: [Firma Digital, Webhook, Process API, Orquestación, AVIVA]
endpoint_dependencies:
  - method: "POST"
    api_name: "ntgy-fua-s-rds-postgres"
    path: "/ejecutarQuery"
  - method: "POST"
    api_name: "ntgy-newco-s-salesforce"
    path: "/postComposite"
  - method: "GET"
    api_name: "ntgy-newco-s-aviva-evo"
    path: "/firmas/:traceId"
  - method: "PUT"
    api_name: "ntgy-newco-s-s3-small"
    path: "/bucket/{bucket}/object"
sql_metadata:
  data_sources:
    - database: "PostgreSQL_RDS"
      schema: "public"
      type: "PostgreSQL"
      tables:
        - api_name: "sign"
          operations: ["READ", "UPDATE"]
          description: "Registro de procesos de firma digital con AVIVA"
---

# POST /firmas/callback

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint actúa como **webhook/callback** para recibir notificaciones de AVIVA sobre cambios de estado en procesos de firma digital. Orquesta la actualización de múltiples sistemas (PostgreSQL, Salesforce, S3) según el estado recibido, garantizando la sincronización completa del proceso de firma.

### Tipo de API

**Capa:** Process API (API-Led Connectivity)

**Función:** Orquestación de procesos de firma digital y sincronización de estados entre sistemas

### Patrón de Integración

**Tipo:** Event-Driven Orchestration + Webhook Handler

**Descripción:** Recibe eventos asíncronos de AVIVA y orquesta actualizaciones en múltiples sistemas backend según el estado del proceso de firma. Implementa lógica condicional compleja para manejar 6 estados diferentes de firma.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    AVIVA[AVIVA<br/>Plataforma de Firma Digital]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /firmas/callback]
    RDS[(PostgreSQL RDS<br/>Tabla: sign)]
    SF[Salesforce<br/>Cases + GDPR]
    ME[Motor de Estados<br/>Intenciones]
    S3[AWS S3<br/>Documentos]
    MQ[AnypointMQ<br/>Eventos]
    
    AVIVA -->|Callback| P
    P -->|Query/Update| RDS
    P -->|Composite API| SF
    P -->|Actualizar Intención| ME
    P -->|Subir PDF| S3
    P -->|Publicar Evento| MQ
    
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:3px
    style AVIVA fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style RDS fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style SF fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style ME fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S3 fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    style MQ fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Firma Confirmada por Cliente (Estado 20)**
   - Cliente completa exitosamente la firma digital en AVIVA
   - Se actualiza Salesforce con el estado de firma confirmada
   - Se registra en Motor de Estados el evento FIRMA_REALIZADA
   - Se ejecuta la lógica específica del proceso (cambio cuenta, cambio titular, etc.)
   - **Beneficio:** Automatización completa del proceso post-firma

2. **Firma Expirada (Estado 7)**
   - El enlace de firma caduca sin ser completado
   - Se cancela automáticamente el caso en Salesforce
   - Se notifica al Motor de Estados (LINK_EXPIRADO)
   - Se elimina la notificación del cliente
   - **Beneficio:** Gestión automática de procesos no completados

3. **Firma Cancelada (Estados 8, 9, &lt;0, 4)**
   - Cliente o sistema cancela el proceso de firma
   - Se actualiza el estado a FirmaContratoCancelada
   - Se cancela el caso con motivo específico
   - Se registra en Motor de Estados (SOLICITA_CANCELACION)
   - **Beneficio:** Trazabilidad completa de cancelaciones

4. **Documentación Disponible (Estado 0)**
   - AVIVA notifica que el documento firmado está listo
   - Se descarga el PDF firmado desde AVIVA
   - Se sube el documento a S3
   - Se vincula con el Case de Salesforce
   - **Beneficio:** Almacenamiento automático de documentación legal

### 🔗 Endpoints Dependientes

Este Process API consume los siguientes System APIs:

| System API | Endpoint | Método | Propósito |
|------------|----------|--------|-----------|
| `ntgy-fua-s-rds-postgres` | `/ejecutarQuery` | POST | Consultar y actualizar tabla `sign` en PostgreSQL + actualizar intenciones Motor de Estados |
| `ntgy-newco-s-salesforce` | `/postComposite` | POST | Actualizar Cases y registros GDPR en Salesforce mediante Composite API |
| `ntgy-newco-s-aviva-evo` | `/firmas/:traceId` | GET | Descargar documento PDF firmado desde AVIVA |
| `ntgy-newco-s-s3-small` | `/bucket/{bucket}/object` | PUT | Subir documentos PDF firmados a AWS S3 |

### Lógica de Orquestación

```mermaid
graph TB
    Start[Callback AVIVA] --> Query[Consultar RDS<br/>tabla sign]
    Query --> Extract[Extraer datos<br/>del proceso]
    Extract --> Choice{Estado AVIVA?}
    
    Choice -->|7: Expirada| Exp[Actualizar: ErrorCallbackCaducado]
    Choice -->|8,9,&lt;0,4: Cancelada| Can[Actualizar: FirmaContratoCancelada]
    Choice -->|20: Confirmada| Conf[Actualizar: FirmaContratoConfirmado]
    Choice -->|0: Doc Disponible| Doc[Descargar y subir a S3]
    
    Exp --> ME1[Motor Estados:<br/>LINK_EXPIRADO]
    Can --> ME2[Motor Estados:<br/>SOLICITA_CANCELACION]
    Conf --> SF[Actualizar Salesforce<br/>Case + GDPR]
    SF --> Logic[Ejecutar lógica<br/>por tipo proceso]
    Logic --> ME3[Motor Estados:<br/>FIRMA_REALIZADA]
    Doc --> S3[Subir a S3]
    
    ME1 --> MQ1[Publicar evento<br/>AnypointMQ]
    ME2 --> MQ2[Publicar evento<br/>AnypointMQ]
    ME3 --> MQ3[Publicar evento<br/>AnypointMQ]
    S3 --> MQ4[Publicar evento<br/>AnypointMQ]
    
    MQ1 --> End[Response OK]
    MQ2 --> End
    MQ3 --> End
    MQ4 --> End
    
    style Start fill:#fff3e0,stroke:#e65100
    style End fill:#e8f5e9,stroke:#2e7d32
    style Choice fill:#fff9c4,stroke:#f57f17
```

**Pasos del Proceso:**

1. **Recepción del Callback**: AVIVA envía notificación con `traceId` y `state`
2. **Consulta de Contexto**: Se consulta PostgreSQL (`ntgy-fua-s-rds-postgres`) para obtener datos del proceso
3. **Enrutamiento por Estado**: Se ejecuta lógica específica según el estado recibido
4. **Actualización de Sistemas**: Se sincronizan PostgreSQL (tabla sign + Motor de Estados) y Salesforce
5. **Gestión de Documentación**: Si aplica, se descarga desde AVIVA y se almacena en S3

### Transformaciones Aplicadas

Este endpoint NO transforma la estructura de datos de AVIVA, pero sí realiza **mapeos de estado** y **enriquecimiento de datos**:

| Estado AVIVA | Estado Interno | Acción Principal |
|--------------|----------------|------------------|
| `"7"` | `ErrorCallbackCaducado` | Cancelar caso por expiración |
| `"8"`, `"9"`, `&lt;0`, `"4"` | `FirmaContratoCancelada` | Cancelar caso por rechazo |
| `"20"` | `FirmaContratoConfirmado` | Confirmar firma y ejecutar lógica SF |
| `"0"` | `GuardadoDocumentoFirmado` | Descargar y almacenar documento |

**Enriquecimiento de Datos:**

- Se obtienen datos del proceso desde PostgreSQL (tabla `sign`) vía `ntgy-fua-s-rds-postgres`
- Se mapea `processType` a eventos del Motor de Estados (misma API `ntgy-fua-s-rds-postgres`)
- Se construyen payloads específicos para Salesforce según tipo de proceso

### Contexto Empresarial

- **Dominio:** Gestión de Procesos de Firma Digital
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Proceso de Negocio:** Automatización de procesos contractuales con firma digital

### Stakeholders

- **Clientes Naturgy**: Necesitan completar procesos de firma de forma digital y segura
- **Agentes de Call Center**: Requieren visibilidad del estado de firma en tiempo real
- **Equipo de Operaciones**: Necesitan trazabilidad completa de procesos de firma
- **Equipo Legal**: Requieren almacenamiento seguro de documentos firmados

### Valor de Negocio

- **Automatización Completa**: Elimina intervención manual en el proceso post-firma
- **Trazabilidad Total**: Registro completo de estados en múltiples sistemas
- **Cumplimiento Legal**: Almacenamiento seguro de documentación firmada
- **Experiencia de Cliente**: Proceso de firma digital sin fricciones
- **Eficiencia Operativa**: Reducción de tiempos de procesamiento de contratos

### Consumidores Potenciales

Este endpoint es consumido exclusivamente por:
- **AVIVA** (Plataforma de Firma Digital) - Webhook callback

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8091` | `http://localhost:8091/api/firmas/callback` |
| **Development** | `https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io` | `https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/firmas/callback` |
| **UAT** | `https://ntgy-newco-p-fua-evo-uat.us-e2.cloudhub.io` | `https://ntgy-newco-p-fua-evo-uat.us-e2.cloudhub.io/api/firmas/callback` |
| **Production** | `https://ntgy-newco-p-fua-evo-pro.us-e2.cloudhub.io` | `https://ntgy-newco-p-fua-evo-pro.us-e2.cloudhub.io/api/firmas/callback` |

### Método HTTP

```
POST /api/firmas/callback
```

### Parámetros de Entrada

#### Request Body (JSON)

```json
{
  "traceId": "string",
  "state": "string",
  "data1": "string (opcional)",
  "data2": "string (opcional)",
  "data3": "string (opcional)"
}
```

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `traceId` | string | ✅ Sí | Identificador único del proceso de firma |
| `state` | string | ✅ Sí | Estado del proceso en AVIVA (ver tabla de estados) |
| `data1` | string | ❌ No | Consentimiento GDPR tipo 02 (on/off) |
| `data2` | string | ❌ No | Consentimiento GDPR tipo 01 (on/off) |
| `data3` | string | ❌ No | Consentimiento GDPR tipo 00 (on/off) |

#### Estados AVIVA Soportados

| Estado | Significado | Acción |
|--------|-------------|--------|
| `"0"` | Documentación disponible | Descargar y almacenar PDF |
| `"4"` | Firma cancelada | Cancelar proceso |
| `"7"` | Firma expirada | Cancelar por expiración |
| `"8"` | Firma cancelada por usuario | Cancelar proceso |
| `"9"` | Firma cancelada por sistema | Cancelar proceso |
| `"20"` | Firma confirmada | Procesar firma exitosa |
| `< 0` | Error en firma | Cancelar proceso |

### Respuestas

#### 200 OK - Callback Procesado Exitosamente

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  }
}
```

#### 500 Internal Server Error - Error en Procesamiento

```json
{
  "code": "ERROR_CODE",
  "message": "Descripción del error",
  "details": "Detalles adicionales"
}
```

### Autenticación

- **Tipo:** Client ID & Client Secret (Anypoint Platform)
- **Nota:** AVIVA debe estar configurado con las credenciales correctas para invocar este webhook

### Timeouts

| Tipo | Valor |
|------|-------|
| **Request Timeout** | 120 segundos (proceso complejo) |
| **Connection Timeout** | 30 segundos |
| **Timeout PostgreSQL** | 60 segundos |
| **Timeout Salesforce** | 60 segundos |
| **Timeout AVIVA** | 30 segundos |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([POST /firmas/callback]) --> SavePayload[Guardar entryPayload]
    SavePayload --> QueryRDS[Query PostgreSQL<br/>SELECT * FROM sign<br/>WHERE traceid = traceId]
    QueryRDS --> ParseResult[Parsear resultRDSQuery]
    ParseResult --> ExtractVars[Extraer variables:<br/>caseId, traceId,<br/>paymentMethodId, traceparent]
    ExtractVars --> InitMotor[Inicializar datos<br/>Motor de Estados]
    InitMotor --> Choice{Estado AVIVA?}
    
    Choice -->|Estado 7| Exp[Estado: ErrorCallbackCaducado]
    Choice -->|Estados 8,9,<0,4| Can[Estado: FirmaContratoCancelada]
    Choice -->|Estado 20| Conf[Estado: FirmaContratoConfirmado]
    Choice -->|Estado 0| Doc[Estado: ProcesadoDocEnCurso]
    
    Exp --> UpdateExp[UPDATE sign<br/>SET status = ErrorCallbackCaducado]
    UpdateExp --> EventExp[Evento: ErrorCallbackCaducado]
    EventExp --> MotorExp[Motor: LINK_EXPIRADO]
    
    Can --> UpdateCan[UPDATE sign<br/>SET status = FirmaContratoCancelada]
    UpdateCan --> EventCan[Evento: FirmaContratoCancelada]
    EventCan --> MotorCan[Motor: SOLICITA_CANCELACION]
    
    Conf --> MotorConf[Motor: FIRMA_REALIZADA]
    MotorConf --> LogicaSF[logicaSFSub_Flow]
    LogicaSF --> UpdateCase[Salesforce Composite:<br/>Update Case + GDPR]
    UpdateCase --> UpdateConf[UPDATE sign<br/>SET status = FirmaContratoConfirmado]
    UpdateConf --> ProcessType{Tipo Proceso?}
    ProcessType -->|CambioCuentaBancaria| Bank[bankAccountFlow]
    ProcessType -->|CambioDeProducto| Product[productChangeFlow]
    ProcessType -->|CambioTitular| Owner[ownerChangeFlow]
    ProcessType -->|ModificacionMetodoPago| Payment[paymentMethodFlow]
    ProcessType -->|CambioDePrecio| Price[priceChangeFlow]
    Bank --> MotorPetSF[Motor: PET_REG_SF]
    Product --> MotorPetSF
    Owner --> MotorPetSF
    Payment --> MotorPetSF
    Price --> MotorPetSF
    MotorPetSF --> CloseSign[Cerrar firma en SF<br/>NewCo_ServiceSignSepaRest]
    CloseSign --> UpdateFinal[UPDATE sign<br/>SET status = CreacionSalesforceFinalizada]
    UpdateFinal --> CheckSW{Proceso con<br/>Switching?}
    CheckSW -->|No| MotorFin[Motor: PROCESO_FINALIZADO]
    CheckSW -->|Sí| SkipFin[Continuar proceso]
    
    Doc --> CheckStatus{status ==<br/>CreacionSalesforceFinalizada?}
    CheckStatus -->|Sí| UpdateDoc[UPDATE sign<br/>SET status = ProcesadoDocEnCurso]
    UpdateDoc --> DownloadDoc[GET AVIVA<br/>Descargar PDF firmado]
    DownloadDoc --> UploadS3[Subir documento a S3]
    UploadS3 --> UpdateGuardado[UPDATE sign<br/>SET status = GuardadoDocumentoFirmado]
    UpdateGuardado --> EventDoc[Evento: GuardadoDocumentoFirmado]
    CheckStatus -->|No| LogNoFinalizado[Log: SF no finalizada]
    
    MotorExp --> Response[Response 200 OK]
    MotorCan --> Response
    MotorFin --> Response
    SkipFin --> Response
    EventDoc --> Response
    LogNoFinalizado --> Response
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style Response fill:#e8f5e9,stroke:#2e7d32
    style Choice fill:#fff9c4,stroke:#f57f17
    style ProcessType fill:#fff9c4,stroke:#f57f17
    style CheckStatus fill:#fff9c4,stroke:#f57f17
    style CheckSW fill:#fff9c4,stroke:#f57f17
```

### Operaciones PostgreSQL

#### Query SELECT (Línea 21 de callbacksAviva.xml)

```sql
SELECT * FROM public.sign WHERE traceid = '[traceId]'
```

**Campos Retornados:**
- `traceid`, `channel`, `processtype`, `callcenteragent`
- `account`, `contact`, `templateid`, `templatepayload`
- `caseid`, `status`, `expiresin`, `doctype`, `docurl`
- `target`, `language`, `parenttraceid`, `accountidsf`
- `entrypayload`, `paymentmethodid`, `traceparent`
- `createdat`, `updatedat`, `orderid`, `avivarequest`

#### Query UPDATE (Múltiples ubicaciones)

```sql
UPDATE public.sign 
SET status = :status, 
    updatedat = (current_timestamp AT TIME ZONE 'Europe/Madrid') 
WHERE traceid = :traceid
```

**Estados Posibles:**
- `ErrorCallbackCaducado`
- `FirmaContratoCancelada`
- `FirmaContratoConfirmado`
- `ProcesadoDocEnCurso`
- `GuardadoDocumentoFirmado`
- `CreacionSalesforceFinalizada`

### Transformaciones DataWeave

#### 1. Inicialización de Datos para Motor de Estados (Línea 65)

```dataweave
%dw 2.0
output application/java

var tipoProceso = vars.resultRDSQuery.processtype
var entryPayloadResultRDS = vars.resultRDSQuery.entrypayload

// Mapeo de proceso MULE/SF a código proceso Motor de Estados
var proceso = tipoProceso match {
    case "ModificacionMetodoPago" -> "ModificacionMetodoPago"
    case "CambioDeProducto" -> "CambioDeProducto"
    case "CambioCuentaBancaria" -> "CambioIBAN"
    case "CambioTitularConSubrogacion" -> "CambioTitular"
    case "CambioTitularSinSubrogacion" -> "CambioTitular"
    else -> ""
}

var tieneSW = tipoProceso match {
    case "ModificacionMetodoPago" -> false
    case "CambioDeProducto" -> false
    case "CambioCuentaBancaria" -> false
    case "CambioTitularSinSubrogacion" -> true
    case "CambioTitularConSubrogacion" -> true
    else -> ""
}
---
{
    procesoConSW: tieneSW,
    evento: "",
    proceso: proceso,
    nif: vars.resultRDSQuery.account,
    billingAccount: billingAccount,
    contrato: contrato,
    casoGestion: "",
    jsonPayload: {}
}
```

#### 2. Construcción de Payload para Salesforce Composite (Línea 475)

```dataweave
%dw 2.0
output application/json
---
{
    "body": {
        "allOrNone": true,
        "collateSubrequests": false,
        "compositeRequest": [
            {
                "method": "PATCH",
                "url": "/services/data/v52.0/sobjects/Case/" ++ vars.caseId,
                "referenceId": "updatedCase",
                "body": {
                    "status": "02",
                    "comments": "Firma realizada con exito"
                }
            }
        ] ++ (vars.inputGDPR default [])
    },
    "operationSalesforce": "composite"
}
```

### Patrones de Diseño

**Patrón 1: Event-Driven Architecture**

**Justificación:** El endpoint actúa como receptor de eventos asíncronos de AVIVA, procesando cada estado de forma independiente y publicando eventos a sistemas downstream.

**Patrón 2: Saga Pattern (Orquestación)**

**Justificación:** Coordina transacciones distribuidas entre PostgreSQL, Salesforce, Motor de Estados y S3, con compensación en caso de error.

**Patrón 3: Content-Based Router**

**Justificación:** Enruta el flujo de procesamiento según el estado recibido de AVIVA, ejecutando lógica específica para cada caso.

### Diagramas de Secuencia

#### 1. Flujo Estado "20" - Firma Confirmada (Caso Principal)

```mermaid
sequenceDiagram
    participant AVIVA as AVIVA
    participant API as Process API
    participant RDS as PostgreSQL
    participant ME as Motor Estados
    participant SF as Salesforce
    participant MQ as AnypointMQ
    
    AVIVA->>API: POST /firmas/callback<br/>{traceId, state: "20"}
    API->>RDS: SELECT * FROM sign<br/>WHERE traceid = traceId
    RDS-->>API: Datos del proceso
    API->>ME: Actualizar Intención<br/>Evento: FIRMA_REALIZADA
    ME-->>API: OK
    API->>SF: Composite API<br/>Update Case + GDPR
    SF-->>API: Case actualizado
    API->>SF: Ejecutar lógica específica<br/>(bankAccount/productChange/etc)
    SF-->>API: Proceso completado
    API->>ME: Actualizar Intención<br/>Evento: PET_REG_SF
    ME-->>API: OK
    API->>SF: Cerrar firma<br/>NewCo_ServiceSignSepaRest
    SF-->>API: Firma cerrada
    API->>RDS: UPDATE sign<br/>SET status = CreacionSalesforceFinalizada
    RDS-->>API: OK
    API->>MQ: Publicar evento<br/>CreacionSalesforceFinalizada
    MQ-->>API: Evento publicado
    alt Proceso sin Switching
        API->>ME: Actualizar Intención<br/>Evento: PROCESO_FINALIZADO
        ME-->>API: OK
    end
    API-->>AVIVA: 200 OK<br/>{codResult: "0000"}
```

#### 2. Flujo Estado "7" - Firma Expirada

```mermaid
sequenceDiagram
    participant AVIVA as AVIVA
    participant API as Process API
    participant RDS as PostgreSQL
    participant ME as Motor Estados
    participant MQ as AnypointMQ
    participant DDB as DynamoDB
    
    AVIVA->>API: POST /firmas/callback<br/>{traceId, state: "7"}
    API->>RDS: SELECT * FROM sign<br/>WHERE traceid = traceId
    RDS-->>API: Datos del proceso
    API->>RDS: UPDATE sign<br/>SET status = ErrorCallbackCaducado
    RDS-->>API: OK
    API->>MQ: Publicar evento<br/>ErrorCallbackCaducado
    MQ-->>API: Evento publicado
    API->>DDB: DELETE notificación<br/>NOTICE_ID = traceId
    DDB-->>API: Notificación eliminada
    API->>ME: Actualizar Intención<br/>Evento: LINK_EXPIRADO<br/>Comentario: Firma cancelada por expiración
    ME-->>API: OK
    API-->>AVIVA: 200 OK<br/>{codResult: "0000"}
```

#### 3. Flujo Estado "0" - Documentación Disponible

```mermaid
sequenceDiagram
    participant AVIVA as AVIVA
    participant API as Process API
    participant RDS as PostgreSQL
    participant AVIVA_API as AVIVA API
    participant S3 as AWS S3
    participant SF as Salesforce
    participant MQ as AnypointMQ
    
    AVIVA->>API: POST /firmas/callback<br/>{traceId, state: "0"}
    API->>RDS: SELECT * FROM sign<br/>WHERE traceid = traceId
    RDS-->>API: Datos del proceso
    alt status == CreacionSalesforceFinalizada
        API->>RDS: UPDATE sign<br/>SET status = ProcesadoDocEnCurso
        RDS-->>API: OK
        API->>AVIVA_API: GET /firma/{transactionId}<br/>?managerType=agent
        AVIVA_API-->>API: PDF firmado (base64)
        API->>S3: Subir documento<br/>uploadCaseAttachmentS3
        S3-->>API: URL del documento
        API->>SF: Update Case<br/>comments: Documentación subida
        SF-->>API: Case actualizado
        API->>RDS: UPDATE sign<br/>SET status = GuardadoDocumentoFirmado<br/>SET docurl = pathFile
        RDS-->>API: OK
        API->>MQ: Publicar evento<br/>GuardadoDocumentoFirmado
        MQ-->>API: Evento publicado
    else status != CreacionSalesforceFinalizada
        API->>API: Log: SF no finalizada
    end
    API-->>AVIVA: 200 OK<br/>{codResult: "0000"}
```

### Manejo de Errores

#### Estrategia de Error Handling

1. **Try-Catch Global**: Todo el sub-flow está envuelto en un bloque try-catch
2. **Error Handler Específico**: `firmaCallback-errorsError_Handler`
3. **Compensación**: En caso de error, se registra en logs y se notifica al Motor de Estados
4. **Idempotencia**: El endpoint puede recibir el mismo callback múltiples veces sin efectos adversos

#### Errores Comunes

| Error | Causa | Acción |
|-------|-------|--------|
| `APP:POSTGRESQL_OMEGA_QUERY` | No se encuentra el registro en PostgreSQL (ntgy-fua-s-rds-postgres) | Retornar error 500 |
| `APP:AVIVA_GET_DOCUMENT_ERROR` | Error al descargar documento de AVIVA (ntgy-newco-s-aviva-evo) | Reintentar o registrar error |
| `APP:COMPOSITE_SALESFORCE` | Error al actualizar Salesforce (ntgy-newco-s-salesforce) | Registrar error y continuar |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Callback de Firma Confirmada (Estado 20)

#### cURL

```bash
curl -X POST "https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/firmas/callback" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "traceId": "abc123-def456-ghi789",
    "state": "20",
    "data1": "on",
    "data2": "off",
    "data3": "on"
  }'
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

### Ejemplo 2: Callback de Firma Expirada (Estado 7)

#### cURL

```bash
curl -X POST "https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/firmas/callback" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "traceId": "abc123-def456-ghi789",
    "state": "7"
  }'
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

### Ejemplo 3: Callback de Firma Cancelada (Estado 8)

#### cURL

```bash
curl -X POST "https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/firmas/callback" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "traceId": "abc123-def456-ghi789",
    "state": "8"
  }'
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

### Ejemplo 4: Callback de Documentación Disponible (Estado 0)

#### cURL

```bash
curl -X POST "https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/firmas/callback" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "traceId": "abc123-def456-ghi789",
    "state": "0"
  }'
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

### Ejemplo 5: JavaScript (Simulación de Callback)

```javascript
// Simulación de callback desde AVIVA
const callbackData = {
  traceId: 'abc123-def456-ghi789',
  state: '20',
  data1: 'on',
  data2: 'off',
  data3: 'on'
};

const response = await fetch('https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/firmas/callback', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'client_id': 'YOUR_CLIENT_ID',
    'client_secret': 'YOUR_CLIENT_SECRET'
  },
  body: JSON.stringify(callbackData)
});

const result = await response.json();
console.log('Callback procesado:', result);
```

### Ejemplo 6: Python (Simulación de Callback)

```python
import requests
import json

# Simulación de callback desde AVIVA
callback_data = {
    'traceId': 'abc123-def456-ghi789',
    'state': '20',
    'data1': 'on',
    'data2': 'off',
    'data3': 'on'
}

response = requests.post(
    'https://ntgy-newco-p-fua-evo-dev.us-e2.cloudhub.io/api/firmas/callback',
    headers={
        'Content-Type': 'application/json',
        'client_id': 'YOUR_CLIENT_ID',
        'client_secret': 'YOUR_CLIENT_SECRET'
    },
    json=callback_data
)

print('Status Code:', response.status_code)
print('Response:', response.json())
```

### Tipos de Procesos Soportados

Este callback maneja los siguientes tipos de procesos de firma:

| Tipo de Proceso | Descripción | Switching |
|-----------------|-------------|-----------|
| `ModificacionMetodoPago` | Cambio de método de pago | No |
| `CambioDeProducto` | Cambio de producto energético | No |
| `CambioCuentaBancaria` | Cambio de IBAN | No |
| `CambioTitularConSubrogacion` | Cambio de titular con subrogación de deuda | Sí |
| `CambioTitularSinSubrogacion` | Cambio de titular sin subrogación | Sí |
| `CambioDePrecio` | Cambio de precio del contrato | No |
