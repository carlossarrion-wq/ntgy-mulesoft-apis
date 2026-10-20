---
api: "ntgy-newco-s-salesforce-evo"
sidebar_position: 5
sidebar_label: "POST /sync"
---

# Endpoint: POST /sync

## 📋 Descripción General

Ejecuta una resincronización manual de eventos de integración entre Salesforce y SAP, procesando eventos pendientes de envío y publicándolos en Anypoint MQ para su procesamiento por sistemas downstream.

**Tipo de API:** System API  
**Capa:** System Layer (API-Led Connectivity)  
**Versión:** 1.30.0-SNAPSHOT  
**Última actualización:** 2026-05-21

---

## 🎯 Propósito

Este endpoint proporciona capacidad de resincronización manual bajo demanda, permitiendo:

- Resincronizar eventos de integración Salesforce-SAP manualmente
- Procesar eventos pendientes de envío
- Publicar eventos en Anypoint MQ para procesamiento asíncrono
- Actualizar estado de eventos en Salesforce
- Recuperar eventos fallidos o pendientes

**Casos de uso:**
- Resincronización manual tras incidencias
- Reprocesamiento de eventos fallidos
- Sincronización bajo demanda
- Testing de integración
- Recuperación de eventos pendientes

---

## 🔗 URL del Endpoint

```
POST /api/sync
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-salesforce-evo-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-salesforce-evo-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Request Body

```json
[
  {
    "Id": "a0X0900000123456",
    "Name": "INT-2026-001234",
    "NewCo_fld_Status__c": "Shipping Pending",
    "NewCo_fld_EventName__c": "ContratoActivado",
    "NewCo_fld_Mode__c": "Asynchronous",
    "NewCo_SAP_Intregations_Detail__r": [
      {
        "Id": "a0Y0900000123456",
        "NewCo_fld_TypeRecord__c": "Account",
        "NewCo_fld_Type__c": "Account",
        "NewCo_fld_Message__c": "{...}"
      }
    ]
  }
]
```

### Estructura del Request Body

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| Array de eventos | array | Sí | Lista de eventos de integración SAP a procesar |

**Cada evento debe contener:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Id` | string | ID del evento de integración SAP en Salesforce |
| `Name` | string | Nombre del evento |
| `NewCo_fld_Status__c` | string | Estado del evento |
| `NewCo_fld_EventName__c` | string | Nombre del tipo de evento |
| `NewCo_fld_Mode__c` | string | Modo de procesamiento (Asynchronous) |
| `NewCo_SAP_Intregations_Detail__r` | array | Detalles del evento (Account, Contract, etc.) |

### Headers

| Header | Valor | Requerido | Descripción |
|--------|-------|-----------|-------------|
| `Content-Type` | `application/json` | Sí | Tipo de contenido del request |
| `Accept` | `application/json` | No | Tipo de contenido esperado en la respuesta |

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

```json
{
  "resultadoEjecucion": "OK",
  "numMensaje": "200"
}
```

**Interpretación:** Los eventos fueron procesados y publicados exitosamente en Anypoint MQ.

### Estructura de Respuesta

| Campo | Tipo | Descripción | Valor |
|-------|------|-------------|-------|
| `resultadoEjecucion` | string | Resultado de la ejecución | `"OK"` |
| `numMensaje` | string | Código de mensaje | `"200"` |

---

### ❌ Errores Técnicos

#### 400 Bad Request

```json
{
  "message": "Bad request"
}
```

**Causas:**
- Request body vacío o malformado
- Formato JSON inválido
- Validación de APIKit fallida

#### 404 Not Found

```json
{
  "message": "Resource not found"
}
```

**Causas:**
- URL del endpoint incorrecta
- Path no existe en la API

#### 500 Internal Server Error

```json
{
  "errorCode": 500,
  "message": "SALESFORCE:CONNECTIVITY",
  "description": "Could not establish connection to Salesforce"
}
```

**Causas:**
- Error de conexión a Salesforce
- Error al publicar en Anypoint MQ
- Error en transformación DataWeave
- Excepción no controlada

---

## 🔧 Implementación Técnica

### Arquitectura del Flow

```
HTTP POST Request → APIKit Router → s-salesforce-evo-main →
s-salesforce-syncFlow (Sub-flow) →
Parallel For Each (por cada evento) →
  ├─ Set Variables (eventPayload, contract)
  ├─ Choice: ¿Tiene detalles?
  │   ├─ SÍ: For Each (por cada detalle)
  │   │   └─ map-sub-flow (mapea Account/Contract/ContractCT)
  │   ├─ Transform: formedEvent (construye evento CloudEvents)
  │   ├─ Publish to Anypoint MQ (exchangeContAt)
  │   └─ Update Salesforce (Composite: Status = Completed)
  │   
  └─ NO: Update Salesforce (Composite: Status = Error)
→ Transform: Response OK
```

### Código Fuente

**Archivos:**
- `src/main/mule/s-salesforce-evo.xml` (flow principal, líneas 125-128)
- `src/main/mule/s-salesforce-sync.xml` (sub-flow `s-salesforce-syncFlow`, líneas 230-341)
- `src/main/mule/s-salesforce-sync.xml` (sub-flow `map-sub-flow`, líneas 342-834)

**Flow Principal:** `post:\sync:application\json:s-salesforce-evo-config`

**Sub-Flow Principal:** `s-salesforce-syncFlow`

**Sub-Flow de Mapeo:** `map-sub-flow`

---

## 🔄 Procesamiento de Eventos

### 1. Parallel For Each

Procesa cada evento del array en paralelo para mejorar performance.

### 2. Mapeo de Objetos (map-sub-flow)

El sub-flow `map-sub-flow` mapea diferentes tipos de objetos:

#### A. Account (Cliente)

**Condición:** `payload.NewCo_fld_TypeRecord__c contains "Account"`

**Mapeo:**
```dataweave
{
  "tipoObjeto": "Account",
  "nombreObjeto": "Account",
  "detalleObjeto": {
    "fechaModificacion": messageToJson.LastModifiedDate,
    "tipoDocumento": messageToJson.acn_fld_DocumentTypeG__c,
    "numeroDocumento": messageToJson.acn_fld_DocumentNumber__c,
    "idCRM": messageToJson.Id,
    "razonSocial": messageToJson.Name,
    "tipoCliente": messageToJson.NewCo_TipoCliente__c,
    "contactos": [...],
    "datosBancarios": [...],
    "direccionFiscal": {...},
    "alianzas": [...]
  }
}
```

#### B. CommercialAgreement (Cuenta Contrato)

**Condición:** `payload.NewCo_fld_TypeRecord__c contains "CommercialAgreement"`

**Mapeo:**
```dataweave
{
  "tipoObjeto": "CommercialAgreement",
  "nombreObjeto": "CommercialAgreement",
  "detalleObjeto": {
    "id": messageToJson.Id,
    "codigo": messageToJson.NewCo_CodeCuentaContrato__c,
    "metodoPago": messageToJson.acn_fld_PaymentMethod__c,
    "direccionCorrespondencia": {...}
  }
}
```

#### C. ContractCT (Contrato CT)

**Condición:** `payload.NewCo_fld_Type__c == "ContractCT"`

**Mapeo:**
```dataweave
{
  "tipoObjeto": "ContractCT",
  "nombreObjeto": "ContractCT",
  "desbloqueoCC": vars.eventPayload.NewCo_fld_UnlockCommercialAgreement__c,
  "detalleObjeto": {
    "codigo": messageToJson.acn_fld_ContractCode2__c,
    "versionContrato": {...},
    "suministro": {...},
    "promociones": [...]
  }
}
```

#### D. Contract (Contrato)

**Condición:** `payload.NewCo_fld_Type__c == "Contract"`

**Mapeo similar a ContractCT con estructura completa del contrato**

### 3. Construcción del Evento CloudEvents

```dataweave
{
  "specversion": "1.0",
  "type": vars.eventPayload.NewCo_fld_EventName__c,
  "source": "newco/contratacion/salesforce",
  "id": vars.eventPayload.Id,
  "time": vars.eventPayload.NewCo_fld_GenerationDate__c,
  "datacontenttype": "application/json",
  "version": "v2",
  "traceparent": traceParent,
  "data": [
    vars.clienteMap,
    vars.cuentaContratoMap,
    vars.contract,
    vars.contractCT
  ] filter ((item) -> item != null and !isEmpty(item))
}
```

**Características:**
- **Formato:** CloudEvents 1.0
- **Trazabilidad:** Incluye `traceparent` con MD5 del ID
- **Data:** Array de objetos mapeados (filtrados por null/empty)

### 4. Publicación en Anypoint MQ

**Destino:** `${anypointMQ.exchangeContAt}`

**Configuración:** `Anypoint_MQ_Config`

### 5. Actualización en Salesforce

**Operación:** Salesforce Composite Request (PATCH)

**Actualización exitosa:**
```json
{
  "NewCo_fld_SendDate__c": "20260521143000",
  "NewCo_fld_Status__c": "Completed",
  "NewCo_fld_ErrorDescription__c": ""
}
```

**Actualización con error:**
```json
{
  "NewCo_fld_SendDate__c": "20260521143000",
  "NewCo_fld_Status__c": "Error",
  "NewCo_fld_ErrorDescription__c": ""
}
```

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Resincronizar un evento

```bash
curl -X POST "http://localhost:8081/api/sync" \
  -H "Content-Type: application/json" \
  -d '[
    {
      "Id": "a0X0900000123456",
      "Name": "INT-2026-001234",
      "NewCo_fld_Status__c": "Shipping Pending",
      "NewCo_fld_EventName__c": "ContratoActivado",
      "NewCo_fld_Mode__c": "Asynchronous",
      "NewCo_fld_GenerationDate__c": "2026-05-21T14:30:00Z",
      "NewCo_SAP_Intregations_Detail__r": [
        {
          "Id": "a0Y0900000123456",
          "NewCo_fld_TypeRecord__c": "Account",
          "NewCo_fld_Type__c": "Account",
          "NewCo_fld_Message__c": "{\"Id\":\"001...\",\"Name\":\"Cliente Test\"}"
        }
      ]
    }
  ]'
```

**Respuesta:**
```json
{
  "resultadoEjecucion": "OK",
  "numMensaje": "200"
}
```

---

### Ejemplo 2: Resincronizar múltiples eventos

```bash
curl -X POST "http://localhost:8081/api/sync" \
  -H "Content-Type: application/json" \
  -d '[
    {
      "Id": "a0X0900000123456",
      "NewCo_fld_EventName__c": "ContratoActivado",
      "NewCo_SAP_Intregations_Detail__r": [...]
    },
    {
      "Id": "a0X0900000123457",
      "NewCo_fld_EventName__c": "ClienteModificado",
      "NewCo_SAP_Intregations_Detail__r": [...]
    }
  ]'
```

---

## 🐛 Troubleshooting

### Problema: Respuesta OK pero eventos no procesados

**Causas posibles:**
- Eventos sin detalles (`NewCo_SAP_Intregations_Detail__r` null)
- Error en transformación DataWeave
- Error al publicar en Anypoint MQ (silencioso)

**Solución:**
1. Verificar logs de la aplicación
2. Comprobar que los eventos tengan detalles
3. Verificar conectividad con Anypoint MQ
4. Revisar estado de eventos en Salesforce

---

### Problema: Error al publicar en Anypoint MQ

**Causas posibles:**
- Anypoint MQ no disponible
- Credenciales incorrectas
- Exchange no existe
- Límites de mensajes alcanzados

**Solución:**
1. Verificar configuración de Anypoint MQ en `global.xml`
2. Comprobar que el exchange `${anypointMQ.exchangeContAt}` existe
3. Verificar credenciales y permisos
4. Revisar límites de mensajes en Anypoint MQ

---

### Problema: Eventos marcados como Error en Salesforce

**Causas posibles:**
- Eventos sin detalles
- Error en mapeo de objetos
- Formato de mensaje inválido

**Solución:**
1. Revisar campo `NewCo_fld_ErrorDescription__c` en Salesforce
2. Verificar que `NewCo_SAP_Intregations_Detail__r` no sea null
3. Validar formato JSON en `NewCo_fld_Message__c`
4. Revisar logs para detalles del error

---

## 🔒 Seguridad y Controles

### Autenticación

- **Tipo:** OAuth 2.0 User-Password Flow (Salesforce)
- **Anypoint MQ:** Client Credentials
- **Configuración:** `Salesforce_Config` y `Anypoint_MQ_Config`

### Autorización

- **Permisos Salesforce:**
  - Objeto `NewCo_SAP_Integration__c` (lectura/escritura)
  - Salesforce Composite API (ejecución)
- **Permisos Anypoint MQ:**
  - Publicar en exchange `exchangeContAt`

### Protección de Datos

- **Datos sensibles:** Información completa de clientes y contratos
- **GDPR:** Cumplimiento según políticas de Salesforce
- **Logging:** Se registran IDs de eventos (no datos sensibles)

---

## 📊 Performance

### Métricas Esperadas

| Métrica | Valor Objetivo |
|---------|----------------|
| Tiempo de respuesta | Variable (depende de cantidad de eventos) |
| Throughput | 10-50 eventos/minuto |
| Error rate | < 5% |

**Nota:** El performance depende de:
- Cantidad de eventos en el request
- Complejidad de los eventos (cantidad de detalles)
- Disponibilidad de Salesforce y Anypoint MQ
- Procesamiento paralelo (Parallel For Each)

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/s-salesforce-evo.raml`
- **Flow Implementation:** `src/main/mule/s-salesforce-sync.xml`
- **Main Flow:** `src/main/mule/s-salesforce-evo.xml` (líneas 125-128)
- **Salesforce Config:** `src/main/mule/global.xml`
- **Anypoint MQ Config:** `src/main/mule/global.xml`

---

## 🔗 Endpoints Relacionados

- **GET /schedulerSync** - Sincronización programada automática
- **GET /queryByCups** - Búsqueda de solicitudes por CUPS
- **GET /distributorOngoingRequest** - Validar solicitudes en curso

---

## 📌 Notas Importantes

### Procesamiento Paralelo

El endpoint usa `parallel-foreach` para procesar múltiples eventos simultáneamente:
- ✅ **Ventaja:** Mejor performance con múltiples eventos
- ⚠️ **Consideración:** Puede generar carga alta en Salesforce/MQ

### Formato CloudEvents 1.0

Los eventos publicados siguen el estándar CloudEvents:
- Facilita interoperabilidad
- Incluye trazabilidad con `traceparent`
- Formato estándar para event-driven architectures

### Actualización de Estado en Salesforce

El endpoint actualiza el estado del evento en Salesforce:
- `Completed`: Evento procesado exitosamente
- `Error`: Evento sin detalles o con error

### Diferencia con /schedulerSync

| Aspecto | /sync (Manual) | /schedulerSync (Automático) |
|---------|----------------|----------------------------|
| **Trigger** | Request HTTP | Scheduler (cada 100s) |
| **Input** | Request body | Query Salesforce |
| **Filtros** | Ninguno | Filtros complejos por evento |
| **Uso** | Resincronización manual | Sincronización continua |

---

**Última actualización:** 2026-05-21  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy