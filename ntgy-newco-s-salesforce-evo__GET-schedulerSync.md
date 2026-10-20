---
api: "ntgy-newco-s-salesforce-evo"
sidebar_position: 4
sidebar_label: "GET /schedulerSync"
---

# Endpoint: GET /schedulerSync

## 📋 Descripción General

Ejecuta una sincronización programada automática de eventos de integración entre Salesforce y SAP, consultando eventos pendientes en Salesforce y publicándolos en Anypoint MQ para su procesamiento asíncrono. Este endpoint se ejecuta automáticamente mediante un scheduler cada 100 segundos.

**Tipo de API:** System API  
**Capa:** System Layer (API-Led Connectivity)  
**Versión:** 1.30.0-SNAPSHOT  
**Última actualización:** 2026-05-21

---

## 🎯 Propósito

Este endpoint proporciona sincronización automática continua, permitiendo:

- Sincronización automática de eventos Salesforce-SAP
- Procesamiento de eventos pendientes de forma programada
- Publicación de eventos en Anypoint MQ según tipo de evento
- Actualización automática de estado de eventos en Salesforce
- Filtrado inteligente de eventos según horario y tipo

**Casos de uso:**
- Sincronización continua automática
- Procesamiento de eventos en background
- Integración asíncrona Salesforce-SAP
- Distribución de eventos a diferentes colas/exchanges
- Gestión automática de eventos pendientes

---

## 🔗 URL del Endpoint

```
GET /api/schedulerSync
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-salesforce-evo-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-salesforce-evo-pro.cloudhub.io`

---

## ⏰ Configuración del Scheduler

### Frecuencia de Ejecución

```xml
<scheduler>
  <scheduling-strategy>
    <fixed-frequency frequency="100000" />
  </scheduling-strategy>
</scheduler>
```

**Características:**
- **Frecuencia:** Cada 100,000 milisegundos (100 segundos)
- **Tipo:** Fixed frequency
- **Inicio:** Automático al desplegar la aplicación
- **Ejecución:** Continua mientras la aplicación esté activa

---

## 📥 Parámetros de Entrada

### Sin Parámetros

Este endpoint **NO requiere parámetros** ya que:
- Se ejecuta automáticamente por el scheduler
- Consulta eventos directamente desde Salesforce
- Aplica filtros predefinidos en la query SOQL

---

## 📤 Respuestas

### ✅ Ejecución Exitosa

**Nota:** Este endpoint NO retorna respuesta HTTP ya que es ejecutado por el scheduler, no por una petición HTTP externa.

**Comportamiento:**
- Procesa eventos encontrados en Salesforce
- Publica eventos en Anypoint MQ
- Actualiza estado de eventos en Salesforce
- Registra logs de ejecución

---

## 🔧 Implementación Técnica

### Arquitectura del Flow

```
Scheduler (cada 100s) → s-salesforce-evo-main →
s-salesforce-scheduler-syncFlow (Sub-flow) →
  ├─ Set Variable: limitQuery (150 o 500 según horario)
  ├─ Salesforce Query (eventos pendientes con filtros)
  ├─ Transform: Filtrado complejo de eventos
  ├─ Logger: payload
  └─ Parallel For Each (por cada evento) →
      ├─ Set Variables (eventPayload, contracts)
      ├─ Choice: ¿Tiene detalles?
      │   ├─ SÍ: For Each (por cada detalle)
      │   │   └─ map-sub-flow (mapea Account/Contract/ContractCT)
      │   ├─ Transform: formedEvent (construye evento CloudEvents)
      │   ├─ Choice: Tipo de evento
      │   │   ├─ PedidoActivado/AlineamientoATRNoActivado
      │   │   │   └─ Publish to trastiendaQueue
      │   │   ├─ NewCo_fld_NoPending__c == "true"
      │   │   │   └─ Publish to exchangeContAt
      │   │   └─ Otros
      │   │       └─ Publish to trastiendaQueue (con "Pendiente")
      │   └─ Update Salesforce (Composite: Status = Completed)
      │   
      └─ NO: Update Salesforce (Composite: Status = Error)
```

### Código Fuente

**Archivos:**
- `src/main/mule/s-salesforce-evo.xml` (flow principal, líneas 129-137)
- `src/main/mule/s-salesforce-sync.xml` (sub-flow `s-salesforce-scheduler-syncFlow`, líneas 13-229)
- `src/main/mule/s-salesforce-sync.xml` (sub-flow `map-sub-flow`, líneas 342-834)

**Flow Principal:** `get:\schedulerSync:application\json:s-salesforce-evo-config`

**Sub-Flow Principal:** `s-salesforce-scheduler-syncFlow`

---

## 🔍 Lógica de Límite Dinámico

### Cálculo del Límite según Horario

```dataweave
var hour = ((now() >> "Europe/Madrid") as DateTime).hour
---
if(hour >= 21 and hour < 23) "150" else "500"
```

**Comportamiento:**
- **21:00 - 22:59 (Madrid):** Límite de 150 eventos
- **Resto del día:** Límite de 500 eventos

**Razón:** Reducir carga durante horario de menor actividad.

---

## 📊 Query SOQL Ejecutada

```sql
SELECT 
  Id, 
  Name, 
  NewCo_fld_Status__c, 
  NewCo_fld_EventParent__c, 
  NewCo_fld_EventName__c, 
  NewCo_fld_Mode__c, 
  NewCo_fld_GenerationDate__c, 
  NewCo_fld_SendDate__c, 
  NewCo_CorrelationId__c, 
  NewCo_fld_ErrorDescription__c,
  NewCo_fld_UnlockCommercialAgreement__c, 
  NewCo_fld_NoPending__c, 
  CreatedDate, 
  NewCo_fld_System__c,
  NewCo_fld_ErrorCode__c,
  NewCo_RequestCode__c,
  NewCo_fld_TraceId__c,
  NewCo_ErrorMessage__c,
  acn_fld_Comments__c,
  acn_fld_ClaimResult__c,
  acn_fld_ResultDetail__c,
  acn_fld_resultreason__c,
  (SELECT 
    Id, 
    Name, 
    NewCo_fld_TypeRecord__c, 
    NewCo_fld_Type__c, 
    NewCo_fld_Message__c, 
    NewCo_fld_IntegracionSAPId__c 
   FROM NewCo_SAP_Intregations_Detail__r)
FROM NewCo_SAP_Integration__c 
WHERE NewCo_fld_Status__c = 'Shipping Pending' 
  AND NewCo_fld_EventName__c NOT IN (
    'RepricingContratoActivado',
    'ReubicacionPSSolicitada',
    'ModificacionAliadoRegistrada'
  ) 
  AND NewCo_fld_Mode__c = 'Asynchronous' 
ORDER BY createddate 
LIMIT :limitQuery
```

**Filtros aplicados:**
1. **Estado:** Solo eventos con estado "Shipping Pending"
2. **Eventos excluidos:** Repricing, Reubicación PS, Modificación Aliado
3. **Modo:** Solo eventos asíncronos
4. **Orden:** Por fecha de creación (más antiguos primero)
5. **Límite:** Dinámico (150 o 500 según horario)

---

## 🔄 Filtrado Complejo de Eventos

### Transformación DataWeave de Filtrado

```dataweave
// Filtra PedidoActivado/AlineamientoATRNoActivado/pending false
var filtPedidoAct = payload filter ((i) -> 
  (i.NewCo_fld_EventName__c == "PedidoActivado" or 
   i.NewCo_fld_EventName__c == "AlineamientoATRNoActivado" or 
   i.NewCo_fld_NoPending__c == "false"))

// Filtra no PedidoActivado/AlineamientoATRNoActivado con NoPending = true
var filtNoPedidoAct = payload filter ((i) -> 
  (i.NewCo_fld_EventName__c != "PedidoActivado" and 
   i.NewCo_fld_EventName__c != "AlineamientoATRNoActivado" and 
   i.NewCo_fld_NoPending__c == "true"))

// Filtra Account sin duplicados por ID
var filtAccountNoAct = filtNoPedidoAct 
  filter ((item) -> sizeOf(item.NewCo_SAP_Intregations_Detail__r[?($.NewCo_fld_Type__c == "Account")] default []) > 0) 
  distinctBy ($.NewCo_SAP_Intregations_Detail__r..NewCo_fld_Message__c[0] substringBy $ == "\"")[3]

// Filtra eventos sin Account
var filtNoAccountNoActt = filtNoPedidoAct 
  filter ((item) -> sizeOf(item.NewCo_SAP_Intregations_Detail__r[?($.NewCo_fld_Type__c == "Account")] default []) == 0)

---
filtPedidoAct ++ filtAccountNoAct ++ filtNoAccountNoActt orderBy($.CreatedDate)
```

**Lógica de filtrado:**
1. **Grupo 1:** PedidoActivado, AlineamientoATRNoActivado, o NoPending=false
2. **Grupo 2:** Otros eventos con NoPending=true y Account (sin duplicados)
3. **Grupo 3:** Otros eventos con NoPending=true sin Account
4. **Resultado:** Concatenación ordenada por fecha de creación

---

## 📨 Distribución de Eventos por Tipo

### 1. PedidoActivado / AlineamientoATRNoActivado

**Destino:** `${anypointMQ.trastiendaQueue}`

**Condición:**
```
NewCo_fld_EventName__c == "PedidoActivado" OR
NewCo_fld_EventName__c == "AlineamientoATRNoActivado"
```

### 2. Eventos con NoPending = "true"

**Destino:** `${anypointMQ.exchangeContAt}`

**Condición:**
```
NewCo_fld_NoPending__c == "true"
```

### 3. Otros Eventos

**Destino:** `${anypointMQ.trastiendaQueue}`

**Modificación:** Se añade "Pendiente" al tipo de evento
```dataweave
vars.formedEvent update {
  case ."type" -> vars.formedEvent."type" ++ "Pendiente"
}
```

---

## 🔧 Construcción del Evento CloudEvents

```dataweave
var source = "newco/contratacion/salesforce"
var version = "00"
var traceId = Crypto::MD5(vars.eventPayload.Id as Binary)
var traceparentId = if(vars.eventPayload.NewCo_fld_EventParent__c != null) 
                      Crypto::MD5(vars.eventPayload.NewCo_fld_EventParent__c as Binary)
                    else if(vars.eventPayload.NewCo_fld_TraceId__c != null) 
                      Crypto::MD5(vars.eventPayload.NewCo_fld_TraceId__c as Binary)
                    else ""
var parentId = "0000000000000001"
var traceFlags = "00"
var traceParent = version ++ "-" ++ traceId ++ "-" ++ parentId ++ "-" ++ traceFlags

var infoAd = {
  "origen": vars.eventPayload.NewCo_fld_System__c,
  "idSap": vars.eventPayload.NewCo_fld_TraceId__c,
  "scCode": vars.eventPayload.NewCo_RequestCode__c,
  "codigoError": vars.eventPayload.NewCo_fld_ErrorCode__c,
  "descError": vars.eventPayload.NewCo_ErrorMessage__c,
  "comentarios": vars.eventPayload.acn_fld_Comments__c,
  "reason": vars.eventPayload.acn_fld_resultreason__c,
  "detRes": vars.eventPayload.acn_fld_ResultDetail__c,
  "codRes": vars.eventPayload.acn_fld_ClaimResult__c,
  "fechaCreacion": vars.eventPayload.CreatedDate,
  "idEventoSF": vars.eventPayload.Id
}
---
{
  "specversion": "1.0",
  "type": vars.eventPayload.NewCo_fld_EventName__c,
  "source": source,
  "id": vars.eventPayload.Id,
  "time": vars.eventPayload.NewCo_fld_GenerationDate__c,
  "datacontenttype": "application/json",
  "version": "v2",
  "traceparent": traceParent,
  "parenttraceid": traceparentId,
  "data": [
    vars.clienteMap,
    vars.cuentaContratoMap,
    vars.contract,
    vars.contractCT,
    infoAd
  ] filter ((item) -> item != null and !isEmpty(item))
}
```

**Características:**
- **Formato:** CloudEvents 1.0
- **Trazabilidad:** Incluye `traceparent` y `parenttraceid`
- **Información adicional:** Incluye `infoAd` con datos de error/resultado
- **Data:** Array de objetos mapeados + información adicional

---

## 📊 Performance

### Métricas Esperadas

| Métrica | Valor |
|---------|-------|
| Frecuencia de ejecución | Cada 100 segundos |
| Eventos procesados por ejecución | 150-500 (según horario) |
| Tiempo de ejecución | Variable (depende de cantidad de eventos) |
| Throughput | ~180-300 eventos/hora |

**Nota:** El performance depende de:
- Cantidad de eventos pendientes en Salesforce
- Horario de ejecución (límite dinámico)
- Complejidad de los eventos
- Disponibilidad de Salesforce y Anypoint MQ

---

## 🔒 Seguridad y Controles

### Autenticación

- **Salesforce:** OAuth 2.0 User-Password Flow
- **Anypoint MQ:** Client Credentials
- **Configuración:** `Salesforce_Config` y `Anypoint_MQ_Config`

### Autorización

- **Permisos Salesforce:**
  - Objeto `NewCo_SAP_Integration__c` (lectura/escritura)
  - Salesforce Composite API (ejecución)
- **Permisos Anypoint MQ:**
  - Publicar en `trastiendaQueue`
  - Publicar en `exchangeContAt`

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/s-salesforce-evo.raml`
- **Flow Implementation:** `src/main/mule/s-salesforce-sync.xml`
- **Main Flow:** `src/main/mule/s-salesforce-evo.xml` (líneas 129-137)
- **Salesforce Config:** `src/main/mule/global.xml`
- **Anypoint MQ Config:** `src/main/mule/global.xml`

---

## 🔗 Endpoints Relacionados

- **POST /sync** - Sincronización manual bajo demanda
- **GET /queryByCups** - Búsqueda de solicitudes por CUPS
- **GET /distributorOngoingRequest** - Validar solicitudes en curso

---

## 📌 Notas Importantes

### Ejecución Automática

Este endpoint se ejecuta automáticamente:
- ✅ **No requiere invocación manual**
- ✅ **Se inicia al desplegar la aplicación**
- ✅ **Continúa ejecutándose cada 100 segundos**
- ⚠️ **No retorna respuesta HTTP**

### Límite Dinámico por Horario

El límite de eventos se ajusta según la hora:
- **21:00-22:59:** 150 eventos (horario de baja actividad)
- **Resto del día:** 500 eventos (horario normal)

**Zona horaria:** Europe/Madrid (UTC+1/UTC+2)

### Eventos Excluidos

Tres tipos de eventos NO son procesados por este scheduler:
1. `RepricingContratoActivado` - Procesado por scheduler específico
2. `ReubicacionPSSolicitada` - Procesado por scheduler específico
3. `ModificacionAliadoRegistrada` - Procesado por scheduler específico

**Razón:** Estos eventos tienen schedulers dedicados con lógica específica.

### Distribución Inteligente de Eventos

Los eventos se distribuyen a diferentes destinos según su tipo:
- **trastiendaQueue:** Eventos de pedidos y eventos pendientes
- **exchangeContAt:** Eventos sin pendientes (NoPending=true)

### Filtrado Complejo

El filtrado de eventos es complejo para:
- Evitar duplicados de Account
- Priorizar ciertos tipos de eventos
- Mantener orden cronológico
- Optimizar procesamiento

### Diferencia con POST /sync

| Aspecto | GET /schedulerSync | POST /sync |
|---------|-------------------|------------|
| **Trigger** | Scheduler automático | Request HTTP manual |
| **Frecuencia** | Cada 100 segundos | Bajo demanda |
| **Input** | Query Salesforce | Request body |
| **Filtros** | Complejos y específicos | Ninguno |
| **Límite** | Dinámico (150/500) | Sin límite |
| **Destinos MQ** | Múltiples (según tipo) | Uno solo |

---

**Última actualización:** 2026-05-21  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy