---
api: "ntgy-newco-s-postgresql-salesforce"
sidebar_position: 7
sidebar_label: "GET /productDetail"
sql_metadata:
  data_sources:
    - database: "newcosalesforce"
      schema: "newcosalesforce"
      type: "PostgreSQL"
      tables:
        - name: "vlocity_cmt__contractaccount__c"
          operations: ["READ"]
        - name: "account"
          operations: ["READ"]
---

# Endpoint: GET /productDetail

## 📋 Descripción General

Recupera información detallada de un producto específico desde la base de datos PostgreSQL, incluyendo datos del contrato, precios históricos, equipamiento fotovoltaico, servicios de renting, datos de energía y balance.

**Tipo de API:** System API  
**Capa:** System Layer (API-Led Connectivity)  
**Versión:** 1.42.0  
**Última actualización:** 2026-05-19

---

## 🎯 Propósito del Negocio

Este endpoint proporciona acceso detallado a la información completa de un producto/asset, incluyendo histórico de precios, equipamiento, servicios adicionales y datos de consumo energético.

### Casos de Uso

1. **Portal de Cliente:** Mostrar detalles completos de productos contratados
2. **Gestión de Contratos:** Consultar información detallada de productos
3. **Facturación:** Obtener precios históricos y vigencias
4. **Gestión Técnica:** Consultar equipamiento fotovoltaico y servicios
5. **Análisis de Consumo:** Datos de energía y balance

---

## 🔗 URL del Endpoint

```
GET /api/productDetail
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-salesforce-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-salesforce-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `assetId` | string | No* | ID del Asset de Salesforce (SFID) | `02i0900000123456` |
| `contractId` | string | No* | ID del Contrato de Salesforce (SFID) | `800aA000001234567` |

**\*Nota:** Al menos uno de los dos parámetros es requerido.

### Lógica de Parámetros

```
SI assetId está presente (no null/empty):
  ├─ Usar assetId directamente para consultar
  └─ Ignorar contractId
SINO:
  ├─ Usar contractId para buscar todos los assets del contrato
  └─ Retornar el primer asset encontrado
```

**Prioridad:** `assetId` > `contractId`

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

#### Producto Encontrado (`codResult: "0000"`)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "productDetail": {
    "contractId": "800aA000001234567",
    "productTypeId": "01t0900000123456",
    "promotionDesc": "Promo Verano 2024",
    "promotionStartDate": "2024-06-01",
    "promotionEndDate": "2024-09-30",
    "productOfferType": "Online",
    "isVariable": false,
    "tollType": "2.0TD",
    "equipmentData": {
      "panelType": "Monocristalino",
      "numberPanels": 12,
      "photovoltaicPower": 5.5,
      "battery": true,
      "chargingPoint": false,
      "inverterPower": 5.0
    },
    "rentingData": {
      "type": "Renting Básico",
      "brand": "Marca X",
      "model": "Modelo Y",
      "numberInstallments": 24
    },
    "oneshotData": {
      "emergency": false,
      "schedule": "Mañana",
      "serviceType": "Mantenimiento"
    },
    "prices": [
      {
        "directriz": "DIR-2024-001",
        "serviceAmount": 0,
        "flatRateAmount": 0,
        "solarPayment": 0,
        "active": true,
        "reason": "02",
        "dateFrom": "2024-01-01",
        "dateTo": "2024-12-31",
        "energy": [0.15, 0.12, 0.10, 0.08, 0.06, 0.05],
        "power": [3.45, 3.45, 0, 0, 0, 0],
        "fixedTerm": 0,
        "variableTerm": 0
      }
    ],
    "choice": {
      "type": null,
      "days": "L,M,X,J,V",
      "hours": "08:00-14:00",
      "recommendedDays": null,
      "recommendedHours": null
    },
    "energy": {
      "consumedEnergy": null,
      "contractedEnergy": "10",
      "lastBillDate": null,
      "validityDate": "2024-01-01",
      "anualMaxConsumption": "3650",
      "textTP": null,
      "estimatedConsumption": "3500"
    },
    "balance": {
      "availableBalance": null,
      "consumedBalance": null,
      "initialBalance": null,
      "actualValidity": null,
      "currency": null,
      "history": null
    },
    "visits": null,
    "marketingInfo": [
      {
        "text": "Información promocional del producto",
        "icon": "https://example.com/icon.png"
      }
    ]
  }
}
```

#### Producto No Encontrado (`codResult: "1003"`)

```json
{
  "result": {
    "codResult": "1003",
    "msgResult": "Contrato inexistente"
  }
}
```

**⚠️ Importante:** Ambas respuestas retornan HTTP 200. Verificar `result.codResult` para determinar éxito/error.

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
| `codResult` | string | Código de resultado | `"0000"` (éxito), `"1003"` (no encontrado) |
| `msgResult` | string | Mensaje descriptivo | `"No hay error"`, `"Contrato inexistente"` |

### Objeto: `productDetail`

#### Información del Contrato y Producto

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `contractId` | string | ID del contrato (Contract SFID) |
| `productTypeId` | string | ID del tipo de producto (Product2 SFID) |
| `promotionDesc` | string | Descripción de promociones (separadas por comas) |
| `promotionStartDate` | string | Fecha inicio de promociones (separadas por comas) |
| `promotionEndDate` | string | Fecha fin de promociones (separadas por comas) |
| `productOfferType` | string | Tipo de oferta del producto |
| `isVariable` | boolean | Si el producto tiene precios indexados |
| `tollType` | string | Tipo de tarifa ATR |

#### Objeto: `equipmentData`

Datos de equipamiento fotovoltaico (para productos solares).

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `panelType` | string | Tipo de panel solar |
| `numberPanels` | number | Número de paneles instalados |
| `photovoltaicPower` | number | Potencia fotovoltaica en kW |
| `battery` | boolean | Si tiene batería |
| `chargingPoint` | boolean | Si tiene punto de carga |
| `inverterPower` | number | Potencia del inversor en kW |

#### Objeto: `rentingData`

Datos de servicios de renting.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `type` | string | Tipo de renting |
| `brand` | string | Marca del equipo |
| `model` | string | Modelo del equipo |
| `numberInstallments` | number | Número de cuotas |

#### Objeto: `oneshotData`

Datos de servicios puntuales.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `emergency` | boolean | Si es servicio de urgencia |
| `schedule` | string | Horario del servicio |
| `serviceType` | string | Tipo de servicio |

#### Array: `prices`

Histórico de precios del producto.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `directriz` | string | Código de directriz de precios |
| `serviceAmount` | number | Importe de servicios |
| `flatRateAmount` | number | Importe de tarifa plana |
| `solarPayment` | number | Pago por instalación solar |
| `active` | boolean | Si el precio está activo |
| `reason` | string | Código de motivo del cambio |
| `dateFrom` | date | Fecha inicio de vigencia |
| `dateTo` | date | Fecha fin de vigencia |
| `energy` | array | Precios de energía (6 periodos) |
| `power` | array | Precios de potencia (6 periodos) |
| `fixedTerm` | number | Término fijo |
| `variableTerm` | number | Término variable |

#### Objeto: `choice`

Datos de elección de días y horarios.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `type` | string | Tipo de elección |
| `days` | string | Días seleccionados |
| `hours` | string | Horas seleccionadas |
| `recommendedDays` | string | Días recomendados |
| `recommendedHours` | string | Horas recomendadas |

#### Objeto: `energy`

Datos de energía y consumo.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `consumedEnergy` | string | Energía consumida |
| `contractedEnergy` | string | Energía contratada diaria |
| `lastBillDate` | date | Fecha última factura |
| `validityDate` | date | Fecha de vigencia del contrato |
| `anualMaxConsumption` | string | Consumo máximo anual |
| `textTP` | string | Texto TP |
| `estimatedConsumption` | string | Consumo estimado anual |

#### Objeto: `balance`

Información de balance.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `availableBalance` | number | Balance disponible |
| `consumedBalance` | number | Balance consumido |
| `initialBalance` | number | Balance inicial |
| `actualValidity` | date | Vigencia actual |
| `currency` | string | Código de moneda |
| `history` | array | Histórico de balance |

#### Array: `marketingInfo`

Información de marketing y adjuntos.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `text` | string | Texto descriptivo |
| `icon` | string | URL del icono/imagen |

---

## 🔧 Implementación Técnica

### Complejidad del Endpoint

Este es uno de los endpoints **más complejos** de la API:
- **1096 líneas de código** en el sub-flow
- **6-8 rutas paralelas** en Scatter-Gather
- **Múltiples queries condicionales** según tipo de producto
- **Cálculos complejos de precios** con DataWeave

### Tipos de Precios Calculados

El endpoint calcula 6 tipos diferentes de precios:

1. **Indexados:** Precios variables según mercado
2. **Fotovoltaicas:** Instalaciones solares
3. **Tarifas Planas:** Precio fijo mensual
4. **Servicios:** Servicios adicionales
5. **Compuestos:** Combinación de varios tipos
6. **Batería Virtual:** Almacenamiento virtual de energía

---

## 📊 Performance

### Métricas Esperadas

| Métrica | Valor Objetivo |
|---------|----------------|
| Tiempo de respuesta (p50) | < 500ms |
| Tiempo de respuesta (p95) | < 2s |
| Throughput | 300+ req/hora |
| Error rate | < 1% |

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consulta por assetId

```bash
curl -X GET "http://localhost:8081/api/productDetail?assetId=02i0900000123456" \
  -H "Content-Type: application/json"
```

### Ejemplo 2: Consulta por contractId

```bash
curl -X GET "http://localhost:8081/api/productDetail?contractId=800aA000001234567" \
  -H "Content-Type: application/json"
```

---

## 🐛 Troubleshooting

### Problema: Error 500 - EXPRESSION "You called the function 'AnonymousFunction' with these arguments: 1: Null"

**Causas posibles:**
- El producto tiene el campo `vlocity_cmt__AttributeSelectedValues__c` NULL

**Solución:**
✅ **CORREGIDO:** Este bug fue identificado y corregido en la versión actual (2026-05-19).

**Código corregido:**
```dataweave
var attributeselectedvalues = if (!isEmpty(content.vlocity_cmt__attributeselectedvalues__c))
    read(content.vlocity_cmt__attributeselectedvalues__c, "application/json")
else
    {}
```

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-salesforce.raml`
- **Flow Implementation:** `src/main/mule/products/getProducts.xml`
- **Main Flow:** `src/main/mule/s-postgresql-salesforce.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`

---

**Última actualización:** 2026-05-19  
**Versión del documento:** 2.0  
**Autor:** Equipo MuleSoft Naturgy