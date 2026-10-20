---
api: "ntgy-newco-s-postgresql-salesforce"
sidebar_position: 1
sidebar_label: "GET /cases"
sql_metadata:
  data_sources:
    - database: "newcosalesforce"
      schema: "newcosalesforce"
      type: "PostgreSQL"
      tables:
        - name: "case"
          operations: ["READ"]
---

# Endpoint: GET /cases

## 📋 Descripción General

Recupera todos los casos (gestiones) asociados a un cliente desde la base de datos PostgreSQL, incluyendo información del estado del caso, pasos del proceso FUA, contratos asociados, cambios de información de contacto y estado de adjuntos.

**Tipo de API:** System API  
**Capa:** System Layer (API-Led Connectivity)  
**Versión:** 1.42.0  
**Última actualización:** 2026-05-19

---

## 🎯 Propósito del Negocio

Este endpoint proporciona acceso a todos los casos de gestión del cliente, permitiendo a las APIs de proceso y experiencia consultar el estado de trámites, gestiones administrativas y cambios solicitados por el cliente.

### Casos de Uso

1. **Portal de Cliente:** Mostrar listado de gestiones en curso y finalizadas
2. **Atención al Cliente:** Consultar estado de trámites durante llamadas
3. **Seguimiento de Gestiones:** Verificar progreso de cambios solicitados
4. **Gestión Documental:** Identificar casos con documentación pendiente
5. **Auditoría:** Consultar histórico de gestiones del cliente

---

## 🔗 URL del Endpoint

```
GET /api/cases
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-salesforce-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-salesforce-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `clientId` | string | No* | ID de cuenta de Salesforce (SFID) | `001aA000001234567` |
| `clientCode` | string | No* | Número de documento del cliente | `12345678A` |

**\*Nota:** Al menos uno de los dos parámetros es requerido.

### Lógica de Parámetros

```
SI clientId está presente (no null/empty/"undefined"):
  ├─ Usar clientId directamente para consultar
  └─ Ignorar clientCode
SINO:
  ├─ Usar clientCode para buscar clientId
  └─ Consultar Account.acn_fld_documentnumber__c = clientCode
```

**Prioridad:** `clientId` > `clientCode`

### Validaciones

- **clientId:** Formato SFID (15-18 caracteres alfanuméricos)
- **clientCode:** Longitud 1-50 caracteres
- **Ambos vacíos:** Retorna error 400

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

#### Casos Encontrados (`codResult: "0000"`)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "cases": [
    {
      "caseCode": "CASE-2024-001234",
      "typeId": "Cambio de titular",
      "type": "Case",
      "step": "Pendiente documentación",
      "subStep": "Esperando firma",
      "state": "En proceso",
      "serviceAccountId": "001aA000001234568",
      "contractId": "800aA000001234567",
      "isSva": false,
      "updateDate": "2024-01-15 10:30:00.000",
      "startDate": "2024-01-15 09:00:00.000",
      "endDate": "",
      "channel": "Web",
      "isCancelable": false,
      "hasFiles": true,
      "caseId": "500aA000001234567",
      "IBAN": "ES1234567890123456789012",
      "categorySC": "Gestión administrativa",
      "cancellationStatus": "",
      "idDocumentum": null,
      "acn_fld_CaseLimitDate__c": "2024-02-15",
      "newMail": "nuevo@example.com",
      "newPhone": "+34912345678",
      "oldMail": "antiguo@example.com",
      "oldPhone": "+34987654321",
      "revocationDate": null,
      "confirmationDate": "2024-01-15",
      "isVisible": true,
      "productType": "01"
    }
  ]
}
```

#### Sin Casos (`codResult: "0000"`)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "cases": []
}
```

**⚠️ Importante:** Siempre retorna HTTP 200, incluso cuando no hay casos. El array `cases` estará vacío.

### ❌ Errores Técnicos

#### 400 Bad Request

```json
{
  "errorCode": 400,
  "message": "Bad request",
  "description": "Invalid query parameter format"
}
```

#### 404 Not Found

```json
{
  "errorCode": 404,
  "message": "Resource not found",
  "description": "The requested endpoint does not exist"
}
```

#### 500 Internal Server Error

```json
{
  "errorCode": 500,
  "message": "DB:CONNECTIVITY",
  "description": "Could not establish connection to PostgreSQL database"
}
```

---

## 🔍 Estructura de Datos Detallada

### Objeto: `result`

| Campo | Tipo | Descripción | Valores Posibles |
|-------|------|-------------|------------------|
| `codResult` | string | Código de resultado | `"0000"` (siempre éxito) |
| `msgResult` | string | Mensaje descriptivo | `"No hay error"` |

### Array: `cases`

Lista de casos asociados al cliente.

#### Objeto: Case

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `caseCode` | string | Código calculado del caso |
| `typeId` | string | Identificador del tipo de gestión (FUA Group Title) |
| `type` | string | Tipo de caso (siempre "Case") |
| `step` | string | Paso actual en el proceso FUA |
| `subStep` | string | Sub-paso actual en el proceso FUA |
| `state` | string | Estado del caso en FUA |
| `serviceAccountId` | string | ID de cuenta de servicio asociada |
| `contractId` | string | ID de contrato asociado |
| `isSva` | boolean | Si es un caso SVA (siempre false) |
| `updateDate` | string | Fecha de última modificación (formato: yyyy-MM-dd HH:mm:ss.SSS) |
| `startDate` | string | Fecha de creación del caso (formato: yyyy-MM-dd HH:mm:ss.SSS) |
| `endDate` | string | Fecha de cierre del caso (vacío si no está cerrado) |
| `channel` | string | Canal de origen del caso (Web, Phone, Email, etc.) |
| `isCancelable` | boolean | Si el caso puede ser cancelado (siempre false) |
| `hasFiles` | boolean | Si el caso tiene archivos adjuntos |
| `caseId` | string | ID del caso en Salesforce (Case SFID) |
| `IBAN` | string | IBAN de pago (para casos de cambio de cuenta bancaria) |
| `categorySC` | string | Categoría del servicio |
| `cancellationStatus` | string | Estado de cancelación |
| `idDocumentum` | null | ID de Documentum (no implementado) |
| `acn_fld_CaseLimitDate__c` | date | Fecha límite del caso |
| `newMail` | string | Nuevo email (para casos de cambio de contacto) |
| `newPhone` | string | Nuevo teléfono (para casos de cambio de contacto) |
| `oldMail` | string | Email anterior (para casos de cambio de contacto) |
| `oldPhone` | string | Teléfono anterior (para casos de cambio de contacto) |
| `revocationDate` | date | Fecha de revocación |
| `confirmationDate` | date | Fecha de confirmación |
| `isVisible` | boolean | Si es visible en Omega |
| `productType` | string | Tipo de producto (01=Electricidad, 02=Gas) |

---

## 🔧 Implementación Técnica

### Query SQL Ejecutada

```sql
SELECT 
  tableCase.sfid as caseId,
  tableCase.Origin,
  tableCase.CreatedDate,
  tableCase.ClosedDate,
  tableCase.NewCo_CalcCaseCode__c,
  tableCase.NewCo_Contract__c,
  tableCase.NewCo_FUAGroupTitle__c,
  tableCase.AccountId,
  tableCase.NewCo_CaseStatusFUA__c,
  tableCase.NewCo_hasAttachments__c,
  tableCase.vlocity_cmt__serviceaccountid__c,
  tableCase.LastModifiedDate,
  tableCase.NewCo_StepFUA__c,
  tableCase.NewCo_SubStepFUA__c,
  tableCase.NewCO_AccountDocumentNumber__c,
  tableCase.NewCo_Recordtype__c,
  tableCase.NewCo_PaymentIBAN__c,
  tableCase.NewCo_CategoriaSC__c,
  tableCase.NewCo_Subestado__c,
  tableCase.acn_fld_CaseLimitDate__c,
  tableCase.NewCo_EmailNuevo__c,
  tableCase.NewCo_TelefonoNuevo__c,
  tableCase.NewCo_EmailAntiguo__c,
  tableCase.NewCo_TelefonoAntiguo__c,
  tableCase.NewCo_FechaRevocacion__c,
  tableCase.NewCo_FechaConfirmacion__c,
  tableCase.NewCo_Visible_en_omega__c,
  tableCase.acn_fld_BusinessLine__c
FROM newcosalesforce.case as tableCase
WHERE tableCase.AccountId = :clientId
  AND tableCase.AccountId NOT IN ('0010900001DdWPVAA3','001aA00000FQlvVQAT','0010900001RvmLcAAJ','001aA0000041BC2QAM','001aA00000Od5LSQAZ')
  AND tableCase.newco_recordtype__c = 'Gestion'
```

**Filtros aplicados:**
- Solo casos del cliente especificado
- Excluye cuentas de sistema específicas
- Solo casos de tipo "Gestion" (gestiones administrativas)

### Transformación DataWeave

```dataweave
fun formatDate(date) = 
  if(isEmpty(date)) "" 
  else date as LocalDateTime as String {format: "uuuu-MM-dd HH:mm:ss.SSS"}
```

---

## 📊 Performance

### Métricas Esperadas

| Métrica | Valor Objetivo |
|---------|----------------|
| Tiempo de respuesta (p50) | < 300ms |
| Tiempo de respuesta (p95) | < 500ms |
| Throughput | 1000+ req/hora |
| Error rate | < 1% |

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consulta por clientId

```bash
curl -X GET "http://localhost:8081/api/cases?clientId=001aA000001234567" \
  -H "Content-Type: application/json"
```

### Ejemplo 2: Consulta por clientCode

```bash
curl -X GET "http://localhost:8081/api/cases?clientCode=12345678A" \
  -H "Content-Type: application/json"
```

### Ejemplo 3: Cliente sin casos

```bash
curl -X GET "http://localhost:8081/api/cases?clientId=001aA000001234567" \
  -H "Content-Type: application/json"
```

**Respuesta:**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "cases": []
}
```

---

## 🐛 Troubleshooting

### Problema: Array vacío de casos

**Causas posibles:**
- El cliente no tiene casos de tipo "Gestion"
- Todos los casos son de otros tipos (no gestiones)
- El clientId es de una cuenta de sistema excluida

**Solución:**
1. Verificar que el cliente tenga casos en Salesforce
2. Consultar directamente la tabla Case con recordtype='Gestion'
3. Verificar que el clientId no esté en la lista de exclusión

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-salesforce.raml`
- **Flow Implementation:** `src/main/mule/cases/getCases.xml`
- **Main Flow:** `src/main/mule/s-postgresql-salesforce.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`

---

**Última actualización:** 2026-05-19  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy