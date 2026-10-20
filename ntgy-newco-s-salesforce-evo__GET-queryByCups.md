---
api: "ntgy-newco-s-salesforce-evo"
sidebar_position: 3
sidebar_label: "GET /queryByCups"
description: "Consulta solicitudes de contratación del distribuidor por código CUPS en Salesforce con información completa de relaciones"
keywords: [salesforce, cups, query, soql, distributor, contract-request, system-api, acn_obj_ContractRequest__c]
tags: [Salesforce, CUPS, Query, Distributor, System API]
salesforce_metadata:
  objects:
    - name: "acn_obj_ContractRequest__c"
      operations: ["QUERY"]
      fields: 
        - "acn_fld_Status__c"
        - "acn_fld_ActivationStatus__c"
        - "acn_fld_Category__c"
        - "acn_fld_AnnulationStatus__c"
        - "acn_fld_Contract__c"
        - "acn_fld_Account__c"
        - "acn_fld_Sctype__c"
        - "acn_fld_resultreasondef__c"
        - "Name"
        - "Distribuidora__c"
      description: "Solicitudes de Contratación del Distribuidor"
      relationships:
        - name: "acn_fld_Contract__r"
          type: "Lookup"
          target_object: "Contract"
          fields: ["Status", "acn_fld_activatedDate__c", "NewCo_FechaBaja__c", "acn_fld_ContractCode2__c"]
          description: "Relación con objeto Contract"
        - name: "acn_fld_Account__r"
          type: "Lookup"
          target_object: "Account"
          fields: ["acn_fld_DocumentNumber__c"]
          description: "Relación con objeto Account"
        - name: "acn_fld_CUPS__r"
          type: "Lookup"
          target_object: "vlocity_cmt__ServicePoint__c"
          fields: ["Name"]
          description: "Relación con objeto CUPS"
        - name: "NewCo_fld_case__r"
          type: "Lookup"
          target_object: "Case"
          fields: ["NewCo_Canal_creador__c"]
          description: "Relación con objeto Case"
---

# Endpoint: GET /queryByCups

## 📋 Descripción General

Realiza una búsqueda en Salesforce de solicitudes de contrato (Contract Requests) asociadas a un código CUPS específico, retornando información detallada sobre el estado de las solicitudes, contratos relacionados y datos del cliente.

**Tipo de API:** System API  
**Capa:** System Layer (API-Led Connectivity)  
**Versión:** 1.30.0-SNAPSHOT  
**Última actualización:** 2026-05-21

---

## 🎯 Propósito

Este endpoint proporciona acceso a las solicitudes de contrato almacenadas en Salesforce mediante búsqueda por CUPS (Código Universal del Punto de Suministro), permitiendo:

- Consultar el estado de solicitudes de contrato en curso
- Obtener información del contrato asociado al CUPS
- Verificar datos del cliente titular
- Consultar información de la distribuidora
- Revisar estados de activación y anulación

**Casos de uso:**
- Validación de solicitudes de contrato en proceso
- Consulta de estado de activación de suministros
- Verificación de contratos asociados a un CUPS
- Auditoría de solicitudes de contrato
- Integración con sistemas de gestión de contratos

---

## 🔗 URL del Endpoint

```
GET /api/queryByCups?cups=<CUPS>
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-salesforce-evo-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-salesforce-evo-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `cups` | string | Sí | Código CUPS del punto de suministro | `ES0021000000000001JN` |

### Detalle del Parámetro

#### `cups` (Requerido)
- **Formato:** 20-22 caracteres alfanuméricos
- **Patrón:** ES + 4 dígitos (distribuidora) + 12 dígitos + 2 letras
- **Propósito:** Código Universal del Punto de Suministro español
- **Validación:** No puede ser null o vacío
- **Ejemplo:** `ES0021000000000001JN`

### Headers

| Header | Valor | Requerido | Descripción |
|--------|-------|-----------|-------------|
| `Accept` | `application/json` | No | Tipo de contenido esperado en la respuesta |

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

#### Solicitudes Encontradas

```json
[
  {
    "acn_fld_Status__c": "02",
    "acn_fld_ActivationStatus__c": "01",
    "acn_fld_Category__c": "Alta",
    "acn_fld_AnnulationStatus__c": null,
    "acn_fld_Contract__c": "8010900000123456",
    "acn_fld_Account__c": "001aA000001234567",
    "acn_fld_Contract__r": {
      "Status": "Activated",
      "acn_fld_activatedDate__c": "2026-01-15",
      "NewCo_FechaBaja__c": null
    },
    "acn_fld_Sctype__c": "01",
    "acn_fld_Account__r": {
      "acn_fld_DocumentNumber__c": "12345678A"
    },
    "acn_fld_resultreasondef__c": "Solicitud procesada correctamente",
    "acn_fld_CUPS__r": {
      "Name": "ES0021000000000001JN"
    },
    "Name": "CR-2026-001234",
    "acn_fld_Contract__r": {
      "acn_fld_ContractCode2__c": "CT-2026-001234"
    },
    "Distribuidora__c": "UFD Distribución",
    "NewCo_fld_case__r": {
      "NewCo_Canal_creador__c": "Web"
    }
  }
]
```

#### Sin Solicitudes (Array Vacío)

```json
[]
```

**⚠️ Importante:** Ambas respuestas retornan HTTP 200. Un array vacío indica que no hay solicitudes de contrato para el CUPS especificado.

---

### ❌ Errores Técnicos

#### 400 Bad Request

```json
{
  "message": "Bad request"
}
```

**Causas:**
- Parámetro `cups` faltante
- Formato de CUPS inválido
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
- Error en query SOQL
- Token OAuth expirado
- Excepción no controlada

---

## 🔍 Estructura de Datos Detallada

### Array de Solicitudes de Contrato

Cada elemento del array contiene:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `acn_fld_Status__c` | string | Estado de la solicitud (códigos: 01-05) |
| `acn_fld_ActivationStatus__c` | string | Estado de activación |
| `acn_fld_Category__c` | string | Categoría de solicitud (Alta, Baja, Modificación) |
| `acn_fld_AnnulationStatus__c` | string | Estado de anulación (null si no aplica) |
| `acn_fld_Contract__c` | string | ID del contrato Salesforce (SFID) |
| `acn_fld_Account__c` | string | ID de la cuenta del cliente (SFID) |
| `acn_fld_Sctype__c` | string | Tipo de solicitud de contrato |
| `acn_fld_resultreasondef__c` | string | Razón del resultado de la solicitud |
| `Name` | string | Nombre/código de la solicitud |
| `Distribuidora__c` | string | Nombre de la distribuidora |

### Objeto: `acn_fld_Contract__r` (Contrato Relacionado)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Status` | string | Estado del contrato (Activated, Draft, etc.) |
| `acn_fld_activatedDate__c` | date | Fecha de activación del contrato |
| `NewCo_FechaBaja__c` | date | Fecha de baja del contrato (null si activo) |
| `acn_fld_ContractCode2__c` | string | Código del contrato |

### Objeto: `acn_fld_Account__r` (Cuenta del Cliente)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `acn_fld_DocumentNumber__c` | string | Número de documento del cliente (NIF/NIE/CIF) |

### Objeto: `acn_fld_CUPS__r` (Punto de Suministro)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Name` | string | Código CUPS del punto de suministro |

### Objeto: `NewCo_fld_case__r` (Caso Relacionado)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `NewCo_Canal_creador__c` | string | Canal de creación del caso (Web, Teléfono, etc.) |

---

## 🔧 Implementación Técnica

### Arquitectura del Flow

```
HTTP GET Request → APIKit Router → s-salesforce-queryByCupsFlow →
Logger (Entrada) → Salesforce Query → Transform to JSON → Logger (Salida) → Response
```

### Código Fuente

**Archivo:** `src/main/mule/s-salesforce-queryByCups.xml`

**Flow:** `s-salesforce-queryByCupsFlow`

### Query SOQL Ejecutada

```sql
SELECT 
  acn_fld_Status__c, 
  acn_fld_ActivationStatus__c,
  acn_fld_Category__c,
  acn_fld_AnnulationStatus__c, 
  acn_fld_Contract__c,
  acn_fld_Account__c,
  acn_fld_Contract__r.Status,
  acn_fld_Contract__r.acn_fld_activatedDate__c, 
  acn_fld_Contract__r.NewCo_FechaBaja__c,
  acn_fld_Sctype__c, 
  acn_fld_Account__r.acn_fld_DocumentNumber__c,  
  acn_fld_resultreasondef__c, 
  acn_fld_CUPS__r.Name, 
  Name, 
  acn_fld_Contract__r.acn_fld_ContractCode2__c,
  Distribuidora__c,
  NewCo_fld_case__r.NewCo_Canal_creador__c 
FROM acn_obj_ContractRequest__c 
WHERE acn_fld_CUPS__r.Name = :cups
```

**Características:**
- **Objeto principal:** `acn_obj_ContractRequest__c` (Solicitudes de Contrato)
- **Relaciones (Joins):**
  - `acn_fld_Contract__r` - Contrato asociado
  - `acn_fld_Account__r` - Cuenta del cliente
  - `acn_fld_CUPS__r` - Punto de suministro
  - `NewCo_fld_case__r` - Caso relacionado
- **Filtro:** Por nombre del CUPS (campo `Name` en objeto `vlocity_cmt__ServicePoint__c`)
- **Parámetro:** `:cups` (parametrizado para seguridad)

### Transformación DataWeave

```dataweave
%dw 2.0
output application/json
---
payload
```

**Características:**
- Transformación simple: conversión directa a JSON
- No hay mapeo de campos
- Retorna la estructura tal como viene de Salesforce
- Mantiene todos los campos y relaciones

### Logging

**Entrada:**
```
Request queryByCups contractID: <cups>
```

**Salida:**
```
Response queryByCups: <payload>
```

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consultar solicitudes de un CUPS

```bash
curl -X GET "http://localhost:8081/api/queryByCups?cups=ES0021000000000001JN" \
  -H "Accept: application/json"
```

**Respuesta:**
```json
[
  {
    "acn_fld_Status__c": "02",
    "acn_fld_ActivationStatus__c": "01",
    "acn_fld_Category__c": "Alta",
    "acn_fld_Contract__c": "8010900000123456",
    "acn_fld_Account__c": "001aA000001234567",
    "acn_fld_Contract__r": {
      "Status": "Activated",
      "acn_fld_activatedDate__c": "2026-01-15"
    },
    "acn_fld_CUPS__r": {
      "Name": "ES0021000000000001JN"
    },
    "Name": "CR-2026-001234"
  }
]
```

---

### Ejemplo 2: CUPS sin solicitudes

```bash
curl -X GET "http://localhost:8081/api/queryByCups?cups=ES0021000000000999ZZ" \
  -H "Accept: application/json"
```

**Respuesta:**
```json
[]
```

---

### Ejemplo 3: CUPS con formato inválido

```bash
curl -X GET "http://localhost:8081/api/queryByCups?cups=INVALID" \
  -H "Accept: application/json"
```

**Respuesta (400):**
```json
{
  "message": "Bad request"
}
```

---

## 🐛 Troubleshooting

### Problema: Array vacío pero el CUPS existe

**Causas posibles:**
- No hay solicitudes de contrato para ese CUPS
- Las solicitudes fueron eliminadas o archivadas
- El CUPS no tiene contratos asociados
- Error en la relación CUPS-ContractRequest en Salesforce

**Solución:**
1. Verificar en Salesforce que existan registros en `acn_obj_ContractRequest__c`
2. Comprobar que el campo `acn_fld_CUPS__r` esté correctamente relacionado
3. Revisar permisos del usuario de integración en Salesforce

---

### Problema: Error 500 - SALESFORCE:CONNECTIVITY

**Causas posibles:**
- Salesforce no disponible
- Token OAuth expirado
- Credenciales incorrectas
- Límites de API de Salesforce alcanzados

**Solución:**
1. Verificar conectividad a Salesforce
2. Revisar configuración OAuth en `global.xml`
3. Comprobar límites de API en Salesforce
4. Verificar logs para detalles del error

---

### Problema: Respuesta lenta (> 5 segundos)

**Causas posibles:**
- Query SOQL compleja con múltiples joins
- Gran volumen de solicitudes para el CUPS
- Salesforce bajo carga
- Índices faltantes en Salesforce

**Solución:**
1. Revisar explain plan de la query SOQL en Salesforce
2. Verificar índices en campos de búsqueda
3. Considerar limitar resultados con LIMIT
4. Optimizar relaciones en Salesforce

---

## 🔒 Seguridad y Controles

### Autenticación

- **Tipo:** OAuth 2.0 User-Password Flow
- **Configuración:** `Salesforce_Config` en `global.xml`
- **Token Endpoint:** Configurable por ambiente
- **Refresh Token:** Automático por el conector

### Autorización

- **Permisos Salesforce:** El usuario de integración debe tener acceso a:
  - Objeto `acn_obj_ContractRequest__c` (lectura)
  - Objeto `Contract` (lectura)
  - Objeto `Account` (lectura)
  - Objeto `vlocity_cmt__ServicePoint__c` (lectura)
  - Objeto `Case` (lectura)

### Protección de Datos

- **Datos sensibles:** Número de documento del cliente
- **GDPR:** Cumplimiento según políticas de Salesforce
- **Logging:** No se registran datos sensibles en logs

### Performance

- **Timeout:** Configurado en Salesforce Connector
- **Rate Limiting:** Sujeto a límites de API de Salesforce
- **Connection Pooling:** Gestionado por el conector

---

## 📊 Performance

### Métricas Esperadas

| Métrica | Valor Objetivo |
|---------|----------------|
| Tiempo de respuesta (p50) | < 1s |
| Tiempo de respuesta (p95) | < 3s |
| Throughput | 500+ req/hora |
| Error rate | < 1% |

**Nota:** El performance depende de la disponibilidad y carga de Salesforce.

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/s-salesforce-evo.raml`
- **Flow Implementation:** `src/main/mule/s-salesforce-queryByCups.xml`
- **Main Flow:** `src/main/mule/s-salesforce-evo.xml`
- **Salesforce Config:** `src/main/mule/global.xml`

---

## 🔗 Endpoints Relacionados

- **GET /distributorOngoingRequest** - Validar solicitudes en curso por contractID
- **GET /getRecCasesByInvoiceId** - Obtener casos por ID de factura
- **POST /sync** - Sincronización Salesforce-SAP

---

## 📌 Notas Importantes

### Objeto Salesforce: acn_obj_ContractRequest__c

Este endpoint consulta el objeto personalizado `acn_obj_ContractRequest__c` que representa las solicitudes de contrato en Salesforce. Este objeto es central en el proceso de contratación y gestión de suministros.

### Relaciones en Salesforce

La query utiliza múltiples relaciones (lookup/master-detail):
- **acn_fld_Contract__r:** Relación con el objeto Contract
- **acn_fld_Account__r:** Relación con el objeto Account
- **acn_fld_CUPS__r:** Relación con vlocity_cmt__ServicePoint__c
- **NewCo_fld_case__r:** Relación con el objeto Case

### Códigos de Estado

**acn_fld_Status__c:**
- `01` - Pendiente
- `02` - En proceso
- `03` - Completado
- `04` - Cancelado
- `05` - Error

**acn_fld_Category__c:**
- `Alta` - Nueva contratación
- `Baja` - Cancelación de contrato
- `Modificación` - Cambio en contrato existente

### Consideraciones de Integración

- Este endpoint NO transforma los datos de Salesforce
- La estructura de respuesta depende completamente del modelo de datos de Salesforce
- Cambios en Salesforce pueden afectar la respuesta sin cambios en la API
- Se recomienda validar la estructura de respuesta en cada ambiente

---

**Última actualización:** 2026-05-21  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy