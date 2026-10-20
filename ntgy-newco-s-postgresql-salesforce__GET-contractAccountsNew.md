---
api: "ntgy-newco-s-postgresql-salesforce"
sidebar_position: 4
sidebar_label: "GET /contractAccountsNew"
sql_metadata:
  data_sources:
    - database: "newcosalesforce"
      schema: "newcosalesforce"
      type: "PostgreSQL"
      tables:
        - name: "vlocity_cmt__contractaccount__c"
          operations: ["READ"]
---

# Endpoint: GET /contractAccountsNew

## 📋 Descripción General

Recupera información completa de cuentas contrato desde la base de datos PostgreSQL, incluyendo cuentas de servicio, cuentas de facturación, direcciones de suministro y facturación, métodos de pago, productos/contratos, puntos de suministro (CUPS) y datos de autoconsumo.

**Tipo de API:** System API  
**Capa:** System Layer (API-Led Connectivity)  
**Versión:** 1.42.0  
**Última actualización:** 2026-05-19

---

## 🎯 Propósito del Negocio

Este endpoint proporciona acceso centralizado a toda la información de cuentas contrato del cliente, permitiendo a las APIs de proceso y experiencia obtener datos completos de contratos, productos energéticos y datos técnicos de manera eficiente y estandarizada.

### Casos de Uso

1. **Portal de Cliente:** Mostrar listado completo de contratos y productos del cliente
2. **Gestión de Contratos:** Consultar estado y detalles de contratos activos
3. **Facturación:** Obtener información de cuentas de facturación y métodos de pago
4. **Atención al Cliente:** Acceso rápido a información de productos durante llamadas
5. **Gestión Técnica:** Consultar datos técnicos de puntos de suministro (CUPS, potencias, tarifas)

---

## 🔗 URL del Endpoint

```
GET /api/contractAccountsNew
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-salesforce-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-salesforce-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `clientId` | string | Sí | ID de cuenta de Salesforce (SFID) del consumidor | `001aA000001234567` |
| `language` | string | No | Código de idioma para traducciones de productos | `ES` |

### Detalle de Parámetros

#### `clientId` (Requerido)
- **Formato:** SFID de Salesforce (15-18 caracteres alfanuméricos)
- **Propósito:** Identificador de la cuenta consumidor (Consumer Account)
- **Uso:** Se utiliza para buscar todas las cuentas de servicio asociadas
- **Validación:** No puede ser null o vacío
- **Ejemplo:** `001aA000001234567`

#### `language` (Opcional)
- **Formato:** Código de idioma de 2 letras en mayúsculas
- **Propósito:** Idioma para traducciones de nombres de productos
- **Default:** `ES` (Español)
- **Valores permitidos:** `ES`, `CA`, `EU`, `GL`, `EN`
- **Impacto:** Afecta el campo `productTypeName` en la respuesta
- **Tabla consultada:** `NewCo_ProductTranslation__c`
- **Ejemplo:** `CA` (Catalán)

### Validaciones

- **clientId:** Formato SFID válido (patrón: `^[a-zA-Z0-9]{15,18}$`)
- **language:** Debe ser uno de los valores permitidos (enum)
- **clientId vacío:** Retorna error 400

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

#### Cuentas Contrato Encontradas (`codResult: "0000"`)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "serviceAccountList": [
    {
      "clientId": "001aA000001234567",
      "serviceAccountId": "001aA000001234568",
      "supplyAddress": {
        "fullAddress": "Calle Gran Vía 123, 3A, 28001 Madrid",
        "addressStreet": "Calle Gran Vía",
        "placeId": "PLACE123",
        "masterId": "MASTER123",
        "addressId": "a0X0900000123456",
        "zipCode": "28001",
        "province": "Madrid",
        "city": "Madrid",
        "town": "Madrid",
        "wayId": "",
        "way": "Calle Gran Vía",
        "wayType": "",
        "number": "123",
        "floor": "3",
        "door": "A",
        "portal": "1",
        "stairs": "Izquierda",
        "provinceCode": "28",
        "codMunIne": "28079",
        "premiseTypeCode": "01",
        "viaId": "12345"
      },
      "billingAccountList": [
        {
          "refNum": "CC123456",
          "billingAccountId": "001aA000001234569",
          "isDirectPayment": true,
          "billDeliveryMethod": "Email",
          "lastInvoice": null,
          "payDay": 5,
          "mailingAddress": {
            "fullAddress": "Calle Mayor 45, 2B, 28013 Madrid",
            "placeId": "PLACE456",
            "masterId": "MASTER456",
            "addressId": "a0X0900000123457",
            "zipCode": "28013",
            "province": "Madrid",
            "city": "Madrid",
            "town": "Madrid",
            "wayId": "",
            "way": "Calle Mayor",
            "wayType": "",
            "number": "45",
            "floor": "2",
            "door": "B",
            "portal": "1",
            "stairs": "",
            "provinceCode": "28",
            "codMunIne": "28079",
            "premiseTypeCode": "01",
            "viaId": "12346"
          },
          "bankAccount": {
            "bankAccountId": "a1X0900000123456",
            "contactId": "003aA000001234567",
            "holderName": "María González López",
            "holderType": "",
            "iban": "ES1234567890123456789012"
          },
          "products": [
            {
              "contractId": "800aA000001234567",
              "contractStatus": "Activated",
              "contractNumber": "CT123456",
              "productClientType": "Residencial",
              "isPortability": false,
              "contractPdfNumber": "DOC123456",
              "isValidPdfDocument": true,
              "serviceContractPdfNumber": "12345",
              "renewedContract": null,
              "statusBonification": "Active",
              "productType": {
                "isEco": true,
                "productTypeId": "01t0900000123456",
                "productTypeName": "Tarifa Luz Eco",
                "productOfferType": "Online",
                "Assetid": "02i0900000123456",
                "isActiveFUV": "Activo",
                "svaFamily": "",
                "productTypeFamily": "luz",
                "priceBookEntryId": "01u0900000123456",
                "ProductServiceCRMId": "PROD123",
                "productCode": "LUZ-ECO-001",
                "productRate": "2.0TD",
                "productTypeNameExtended": "Tarifa Luz Eco 2.0TD",
                "productStatus": "03",
                "countEquipment": null
              },
              "isActive": true,
              "contractSignatureDate": "2024-01-15",
              "startDate": "2024-02-01T00:00:00Z",
              "endDate": "2025-02-01T00:00:00Z",
              "dropOutDate": null,
              "isPending": false,
              "techData": {
                "IPDate": "2024-01-20",
                "isValidCIE": true,
                "changePower": false,
                "techDataId": "ES0021000000000001JN",
                "cups": "ES0021000000000001JN",
                "distribution": "UFD Distribución",
                "distributorCNMC": "0021",
                "rate": "2.0TD",
                "rateId": "2.0TD",
                "voltage": "Baja Tensión",
                "voltageCod": "BT",
                "selfConsumption": false,
                "powerBie": "5.750000",
                "installation": "Individual",
                "installationType": "Maxímetro",
                "remoteManagement": true,
                "serviceType": "Electricidad",
                "p1": "3.450000",
                "p2": "3.450000",
                "p3": "0.000000",
                "p4": "0.000000",
                "p5": "0.000000",
                "p6": "0.000000",
                "recommendedPowerPlain": "",
                "recommendedPowerValley": "",
                "cnae": "9820",
                "essentialType": false,
                "protectedType": false,
                "type": "00",
                "subsection": null,
                "cupsType": null,
                "collective": null,
                "installationPower": 0,
                "selfConsumptionData": []
              },
              "pendingUnsubscribe": false,
              "cancelIndicator": "",
              "hasGasReadingEmail": false,
              "productHistoricalList": []
            }
          ]
        }
      ]
    }
  ]
}
```

#### Cuentas Contrato No Encontradas (`codResult: "1002"`)

```json
{
  "result": {
    "codResult": "1002",
    "msgResult": "Cuenta Contrato inexistente"
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

**Causas:**
- Parámetro `clientId` con formato inválido
- Parámetro `clientId` vacío o null
- Parámetro `language` con valor no permitido
- Validación de APIKit fallida

#### 404 Not Found

```json
{
  "errorCode": 404,
  "message": "Resource not found",
  "description": "The requested endpoint does not exist"
}
```

**Causas:**
- URL del endpoint incorrecta
- Path no existe en la API

#### 500 Internal Server Error

```json
{
  "errorCode": 500,
  "message": "DB:CONNECTIVITY",
  "description": "Could not establish connection to PostgreSQL database"
}
```

**Causas:**
- Error de conexión a base de datos
- Error en query SQL
- Error en transformación DataWeave
- Excepción no controlada

---

## 🔍 Estructura de Datos Detallada

### Objeto: `result`

| Campo | Tipo | Descripción | Valores Posibles |
|-------|------|-------------|------------------|
| `codResult` | string | Código de resultado | `"0000"` (éxito), `"1002"` (no encontrado) |
| `msgResult` | string | Mensaje descriptivo | `"No hay error"`, `"Cuenta Contrato inexistente"` |

### Array: `serviceAccountList`

Lista de cuentas de servicio asociadas al cliente.

#### Objeto: Service Account

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `clientId` | string | ID de cuenta consumidor (Consumer Account SFID) |
| `serviceAccountId` | string | ID de cuenta de servicio (Service Account SFID) |
| `supplyAddress` | object | Dirección del punto de suministro |
| `billingAccountList` | array | Lista de cuentas de facturación |

#### Objeto: `supplyAddress`

Dirección del punto de suministro de energía.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `fullAddress` | string | Dirección completa formateada |
| `addressStreet` | string | Nombre de la calle |
| `placeId` | string | Identificador de lugar |
| `masterId` | string | Identificador maestro |
| `addressId` | string | ID de dirección (Premises SFID) |
| `zipCode` | string | Código postal |
| `province` | string | Provincia |
| `city` | string | Ciudad |
| `town` | string | Localidad |
| `wayId` | string | Identificador de vía |
| `way` | string | Nombre de la vía |
| `wayType` | string | Tipo de vía |
| `number` | string | Número de la vía |
| `floor` | string | Piso |
| `door` | string | Puerta |
| `portal` | string | Portal |
| `stairs` | string | Escalera |
| `provinceCode` | string | Código de provincia |
| `codMunIne` | string | Código INE del municipio |
| `premiseTypeCode` | string | Código de tipo de inmueble |
| `viaId` | string | Identificador de vía |

#### Array: `billingAccountList`

Lista de cuentas de facturación asociadas a la cuenta de servicio.

#### Objeto: Billing Account

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `refNum` | string | Número de referencia de cuenta contrato |
| `billingAccountId` | string | ID de cuenta de facturación (Billing Account SFID) |
| `isDirectPayment` | boolean | Si el pago es domiciliado (true) o no (false) |
| `billDeliveryMethod` | string | Método de entrega de factura (Email, Postal, etc.) |
| `lastInvoice` | null | Última factura (no implementado) |
| `payDay` | integer | Día fijo de pago del mes (1-31) |
| `mailingAddress` | object | Dirección de facturación (misma estructura que supplyAddress) |
| `bankAccount` | object | Datos de cuenta bancaria |
| `products` | array | Lista de productos/contratos |

#### Objeto: `bankAccount`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `bankAccountId` | string | ID del método de pago (PaymentMethod SFID) |
| `contactId` | string | ID del contacto asociado |
| `holderName` | string | Nombre del titular de la cuenta |
| `holderType` | string | Tipo de titular (no implementado) |
| `iban` | string | Número IBAN de la cuenta bancaria |

#### Array: `products`

Lista de productos/contratos asociados a la cuenta de facturación.

#### Objeto: Product

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `contractId` | string | ID del contrato (Contract SFID) |
| `contractStatus` | string | Estado del contrato (Activated, Pending, etc.) |
| `contractNumber` | string | Número de contrato |
| `productClientType` | string | Tipo de cliente del producto (Residencial, Empresa, etc.) |
| `isPortability` | boolean | Si es portabilidad |
| `contractPdfNumber` | string | Número de documento PDF del contrato |
| `isValidPdfDocument` | boolean | Si el documento PDF es válido |
| `serviceContractPdfNumber` | string | Número de línea de contrato Siebel |
| `renewedContract` | string | Contrato renovado |
| `statusBonification` | string | Estado de bonificación |
| `productType` | object | Información del tipo de producto |
| `isActive` | boolean | Si el producto está activo |
| `contractSignatureDate` | date | Fecha de firma del contrato |
| `startDate` | datetime | Fecha de inicio (formato ISO 8601) |
| `endDate` | datetime | Fecha de fin (formato ISO 8601) |
| `dropOutDate` | date | Fecha de baja |
| `isPending` | boolean | Si está pendiente |
| `techData` | object | Datos técnicos del punto de suministro |
| `pendingUnsubscribe` | boolean | Si tiene baja pendiente |
| `cancelIndicator` | string | Indicador de cancelación |
| `hasGasReadingEmail` | boolean | Si tiene email de aviso de lectura de gas |
| `productHistoricalList` | array | Lista de productos históricos |

#### Objeto: `productType`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `isEco` | boolean | Si es producto ecológico |
| `productTypeId` | string | ID del tipo de producto (Product2 SFID) |
| `productTypeName` | string | Nombre del tipo de producto (traducido según language) |
| `productOfferType` | string | Tipo de oferta (Online, Offline, etc.) |
| `Assetid` | string | ID del asset (Asset SFID) |
| `isActiveFUV` | string | Estado activo FUV (Activo, En curso, Inactivo) |
| `svaFamily` | string | Familia SVA |
| `productTypeFamily` | string | Familia del producto (luz, gas, servicios, recarga, solar) |
| `priceBookEntryId` | string | ID de entrada en libro de precios |
| `ProductServiceCRMId` | string | ID de servicio de producto en CRM |
| `productCode` | string | Código del producto |
| `productRate` | string | Tarifa del producto |
| `productTypeNameExtended` | string | Nombre extendido del tipo de producto |
| `productStatus` | string | Estado del producto (código 01-29) |
| `countEquipment` | null | Contador de equipamiento (no implementado) |

#### Mapeo de Estados de Producto (productStatus)

| Código | isActiveFUV | isActive | isPending | Descripción |
|--------|-------------|----------|-----------|-------------|
| 02, 01 | - | false | true | Pendiente de activación |
| 03, 04, 05, 06, 07, 09, 11, 14, 18, 23 | Activo | true | false | Activo |
| 10, 11 | Activo | true | true | Activo con baja pendiente |
| 08, 17 | Activo | - | - | Activo (estados especiales) |
| 12, 13, 15, 16 | Inactivo | false | false | Inactivo/Cancelado |

#### Objeto: `techData`

Datos técnicos del punto de suministro (CUPS). Para productos de tipo "servicios", todos los campos son null.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `IPDate` | date | Fecha de instalación/puesta en marcha |
| `isValidCIE` | boolean | Si el CIE es válido |
| `changePower` | boolean | Si hay cambio de potencia |
| `techDataId` | string | ID técnico (CUPS) |
| `cups` | string | Código CUPS del punto de suministro |
| `distribution` | string | Nombre de la distribuidora |
| `distributorCNMC` | string | Código CNMC de la distribuidora |
| `rate` | string | Tarifa contratada |
| `rateId` | string | ID de la tarifa |
| `voltage` | string | Nivel de tensión (Baja Tensión, Media Tensión) |
| `voltageCod` | string | Código de tensión |
| `selfConsumption` | boolean | Si tiene autoconsumo |
| `powerBie` | string | Potencia BIE (formato decimal con 6 decimales) |
| `installation` | string | Tipo de instalación |
| `installationType` | string | Tipo de control de potencia |
| `remoteManagement` | boolean | Si tiene telegestión |
| `serviceType` | string | Tipo de servicio (Electricidad, Gas) |
| `p1` - `p6` | string | Potencias contratadas por periodo (formato decimal) |
| `recommendedPowerPlain` | string | Potencia recomendada llano |
| `recommendedPowerValley` | string | Potencia recomendada valle |
| `cnae` | string | Código CNAE |
| `essentialType` | boolean | Si es suministro esencial |
| `protectedType` | boolean | Si es suministro protegido |
| `type` | string | Tipo de autoconsumo (código) |
| `subsection` | null | Subsección (en selfConsumptionData) |
| `cupsType` | null | Tipo de CUPS (en selfConsumptionData) |
| `collective` | null | Si es colectivo (en selfConsumptionData) |
| `installationPower` | integer | Potencia instalada |
| `selfConsumptionData` | array | Datos detallados de autoconsumo (CAU) |

#### Array: `selfConsumptionData`

Datos de autoconsumo (CAU - Configuración de Autoconsumo). Solo presente si el punto de suministro tiene autoconsumo.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `CAU` | string | Código CAU |
| `subsection` | string | Tipo de subsección |
| `cupsType` | string | Tipo de CUPS |
| `collective` | boolean | Si es autoconsumo colectivo |
| `type` | string | Tipo de autoconsumo |
| `datosGenInst` | object | Datos de generación instalada (1, 2, 3) |
| `installationPower` | object | Potencias instaladas (1, 2, 3) |
| `tecGenerador` | object | Tecnologías de generador (1, 2, 3) |
| `refCatastro` | object | Referencias catastrales (1, 2, 3) |
| `esquemaMedida` | object | Esquemas de medida (1, 2, 3) |
| `combustible` | object | Tipos de combustible (1, 2, 3) |
| `ssaa` | object | Sistemas auxiliares (1, 2, 3) |
| `unicoContrato` | object | Único contrato (1, 2, 3) |

#### Mapeo de Familias de Producto (productTypeFamily)

| Código Salesforce | productTypeFamily | svaFamily |
|-------------------|-------------------|-----------|
| Electricidad | luz | (vacío o luz según tipo) |
| Gas | gas | (vacío o gas según tipo) |
| Hogar | servicios | hogar |
| Seguros | servicios | servicios |
| Service | servicios | servicios |
| Recarga | recarga | (vacío) |
| Solar | solar | (vacío o solar según tipo) |

---

## 🔧 Implementación Técnica

### Arquitectura

```
┌─────────────────┐
│   API Consumer  │
└────────┬────────┘
         │ GET /api/contractAccountsNew?clientId=xxx&language=ES
         ▼
┌─────────────────────────────────────┐
│  APIKit Router                      │
│  (s-postgresql-salesforce-main)     │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Flow: get:\contractAccountsNew     │
│  - Extrae parámetros                │
│  - Llama a sub-flow                 │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Sub-flow:                          │
│  getContractAccountNewSub_Flow      │
│  - Ejecuta 4 queries secuenciales   │
│  - Transforma respuesta             │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Query 1: Service Account +         │
│           Supply Address            │
│  ├─ Valida si hay resultados        │
│  └─ Si vacío → Error 1002           │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Query 2: Billing Account +         │
│           Billing Address +         │
│           Payment Method            │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Query 3: Asset + Contract +        │
│           Service Point             │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Query 4 (condicional):             │
│  CAU (Self-consumption data)        │
│  - Solo si hay service points       │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  DataWeave Transformation           │
│  - Mapeo jerárquico complejo        │
│  - Formateo de decimales            │
│  - Construcción de JSON             │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Response (HTTP 200)                │
│  - codResult: "0000" o "1002"       │
└─────────────────────────────────────┘
```

### Queries SQL Ejecutadas

#### 1. Query Service Account + Supply Address

```sql
SELECT 
  serviceAccount.sfid as serviceSfid,
  supplyAddress.sfid as supplyAddressID,
  supplyAddress.NewCo_PlaceId__c,
  supplyAddress.NewCo_MasterId__c,
  supplyAddress.vlocity_cmt__PostalCode__c,
  supplyAddress.vlocity_cmt__State__c,
  supplyAddress.vlocity_cmt__City__c,
  supplyAddress.NewCo_Localidad__c,
  supplyAddress.vlocity_cmt__StreetAddress__c,
  supplyAddress.vlocity_cmt__PremisesNumber__c,
  supplyAddress.NewCo_Portal__c,
  supplyAddress.NewCo_Escalera__c,
  supplyAddress.vlocity_cmt__Floor__c,
  supplyAddress.NewCo_Puerta__c,
  supplyAddress.NewCo_DireccionCompleta__c,
  supplyAddress.NewCo_ProvinceCode__c,
  supplyAddress.NewCo_codMunIne__c,
  supplyAddress.NewCo_PremiseTypeCode__c,
  supplyAddress.NewCo_ViaId__c
FROM 
  newcosalesforce.vlocity_cmt__Premises__c as supplyAddress,
  newcosalesforce.account consumerAccount
INNER JOIN newcosalesforce.account serviceAccount 
  ON serviceAccount.parentid = consumerAccount.sfid 
WHERE 
  serviceAccount.newco_recordtype__c = 'Service'
  AND serviceAccount.vlocity_cmt__premisesid__c = supplyAddress.sfid 
  AND consumerAccount.sfid = :clientId
```

**Validación:** Si no hay resultados → retorna `codResult: "1002"`

#### 2. Query Billing Account + Billing Address + Payment Method

```sql
SELECT 
  billingaccount.sfid as billingSfid,
  billingAccount.newco_serviceaccount__c as serviceSfid,
  billingaccount.newco_payment_method__c,
  billingaccount.vlocity_cmt__billdeliverymethod__c,
  billingaccount.vlocity_cmt__accountpaymenttype__c,
  billingaccount.newco_codecuentacontrato__c,
  billingaccount.newco_diafijopago__c,
  billingaddress.sfid as billingaddressID,
  -- ... más campos de billingaddress ...
  paymentmethod.sfid as paymentmethodID,
  paymentmethod.vlocity_cmt__BankAccountHolderName__c,
  paymentmethod.vlocity_cmt__IBAN__c,
  paymentmethod.NewCo_Contact__c
FROM 
  newcosalesforce.vlocity_cmt__Premises__c as billingaddress,
  newcosalesforce.vlocity_cmt__paymentmethod__c paymentmethod,
  newcosalesforce.account billingAccount
WHERE 
  billingAccount.newco_recordtype__c = 'Billing' 
  AND billingAccount.vlocity_cmt__premisesid__c = billingaddress.sfid
  AND billingAccount.newco_payment_method__c = paymentmethod.sfid
  AND billingAccount.newco_serviceaccount__c IN (:serviceAccountIDs)
```

#### 3. Query Asset + Contract + Service Point

```sql
SELECT 
  contract.sfid as contractID,
  contract.newco_billingaccount__c as billingSfid,
  contract.Status as contractStatus,
  contract.acn_fld_ContractCode2__c,
  -- ... más campos de contract ...
  tableasset.sfid as assetID,
  tableasset.status as assetStatus,
  tableasset.NewCo_Specification_Type__c,
  tableasset.ProductFamily,
  -- ... más campos de asset ...
  servicepoint.sfid as servicepointID,
  servicepoint.acn_fld_SelfConsumption__c,
  servicepoint.Name as servicepointName,
  servicepoint.NewCo_CNAECode__c,
  -- ... más campos de servicepoint ...
  (SELECT newco_descripcion__c 
   FROM newcosalesforce.NewCo_ProductTranslation__c 
   WHERE name = tableasset.vlocity_cmt__ItemName__c 
     AND newco_idioma__c = :language) as productTypeName
FROM newcosalesforce.asset tableasset 
FULL JOIN newcosalesforce.vlocity_cmt__servicepoint__c as servicePoint 
  ON tableasset.vlocity_cmt__ServicePointId__c = servicePoint.sfid
INNER JOIN newcosalesforce.contract as contract 
  ON contract.sfid = tableasset.acn_fld_contract__c
LEFT OUTER JOIN newcosalesforce.acn_obj_CNAE__c as tableCNAE 
  ON servicepoint.NewCo_CNAECode__c = tableCNAE.name
INNER JOIN newcosalesforce.product2 as tableproduct 
  ON tableasset.product2id = tableproduct.sfid
WHERE tableasset.parentid IS NULL
  AND (tableasset.status IN ('02','03','04','06','07','09','10','11','12','16','18','20','22','23','24','25','26','27','29') 
       OR (tableasset.status = '08' AND tableasset.acn_fld_EndDateVersion__c IS NULL)) 
  AND contract.newco_billingaccount__c IN (:billingAccountIDs)
```

#### 4. Query CAU (Condicional)

Solo se ejecuta si existen service points en la query anterior.

```sql
SELECT 
  ncc.newco_cau__c,
  ncc.newco_tiposubseccion__c,
  ncc.newco_tipoautoconsumo__c,
  ncc.newco_colectivo__c,
  ncc.newco_potinstaladagen1__c,
  ncc.newco_potinstaladagen2__c,
  ncc.newco_potinstaladagen3__c,
  ncc.newco_servicepoint__c,
  ncc.newco_tipoinstalacion1__c,
  ncc.newco_tipoinstalacion2__c,
  ncc.newco_tipoinstalacion3__c,
  -- ... más campos CAU ...
FROM newcosalesforce.NewCo_CAU__c ncc
WHERE ncc.newco_servicepoint__c IN (:servicePointIDs)
```

### Transformaciones DataWeave

#### Formateo de Decimales

```dataweave
fun formateoDecimales(valor) =
  if (isEmpty(valor)) '0'
  else
    if (valor==0) "0" 
    else valor as String { format: "0.000000" }
```

#### Formateo de Fechas

```dataweave
fun formatDate(date) = 
  if (isEmpty(date)) "" 
  else (date as String) ++ "T00:00:00Z"
```

#### Mapeo de Estado de Producto

```dataweave
var isActiveFUV = productItem.assetstatus match {
  case str if ['03', '05', '08', '17'] contains(str as String) -> 'Activo'
  case str if ['01', '02', '04', '06', '07', '09', '10', '11', '18'] contains(str as String) -> 'En curso'
  case str if ['12', '13', '15', '16'] contains(str as String) -> 'Inactivo'
  else -> ''
}
```

#### Mapeo de Familia de Producto

```dataweave
var family = productItem.newco_specification_type__c match {
  case str if str == 'Product' -> {
    "product_family": productItem.productfamily match {
      case str if str == 'Electricidad' -> 'luz'
      case str if str == 'Gas' -> 'gas'
      case str if ['Hogar', 'Seguros', 'Service'] contains(str as String) -> 'servicios'
      case str if str == 'Recarga' -> 'recarga'
      case str if str == 'Solar' -> 'solar'
      else -> ''
    },
    "svaFamily": ''
  }
  case str if str == 'Service' -> {
    "product_family": 'servicios',
    "svaFamily": productItem.productfamily match {
      case str if str == 'Electricidad' -> 'luz'
      case str if str == 'Gas' -> 'gas'
      case str if ['Seguros', 'Service'] contains(str as String) -> 'servicios'
      case str if str == 'Hogar' -> 'hogar'
      case str if str == 'Solar' -> 'solar'
      else -> ''
    }
  }
  else -> ''
}
```

---

## 📊 Performance

### Métricas Esperadas

| Métrica | Valor Objetivo | Notas |
|---------|----------------|-------|
| Tiempo de respuesta (p50) | < 1s | Depende del volumen de datos |
| Tiempo de respuesta (p95) | < 2s | Con múltiples productos |
| Throughput | 500+ req/hora | Validar en producción |
| Error rate | < 1% | Excluye errores de negocio (1002) |
| Disponibilidad | 99.9% | Validar en producción |

### Optimizaciones Implementadas

1. **Queries Secuenciales Optimizadas:** 4 queries ejecutadas en secuencia con filtros eficientes
2. **Connection Pooling:** Pool de 5-30 conexiones PostgreSQL
3. **Prepared Statement Cache:** 100 statements cacheados
4. **Fetch Size:** 100 registros por batch
5. **Filtrado Temprano:** Validación de service accounts antes de queries costosas

### Consideraciones de Performance

- **Volumen de Datos:** El tiempo de respuesta aumenta con el número de productos/contratos
- **Complejidad de Transformación:** DataWeave procesa estructuras jerárquicas complejas
- **Query CAU Condicional:** Solo se ejecuta si hay service points, ahorrando tiempo
- **Productos Históricos:** Se filtran y procesan por separado

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consulta básica con idioma por defecto

```bash
curl -X GET "http://localhost:8081/api/contractAccountsNew?clientId=001aA000001234567" \
  -H "Content-Type: application/json"
```

### Ejemplo 2: Consulta con idioma catalán

```bash
curl -X GET "http://localhost:8081/api/contractAccountsNew?clientId=001aA000001234567&language=CA" \
  -H "Content-Type: application/json"
```

### Ejemplo 3: Cliente sin cuentas contrato

```bash
curl -X GET "http://localhost:8081/api/contractAccountsNew?clientId=999999999999999" \
  -H "Content-Type: application/json"
```

**Respuesta:**
```json
{
  "result": {
    "codResult": "1002",
    "msgResult": "Cuenta Contrato inexistente"
  }
}
```

### Ejemplo 4: Error de parámetro inválido

```bash
curl -X GET "http://localhost:8081/api/contractAccountsNew?clientId=INVALID" \
  -H "Content-Type: application/json"
```

**Respuesta (400):**
```json
{
  "errorCode": 400,
  "message": "Bad request",
  "description": "Invalid query parameter format"
}
```

---

## 🐛 Troubleshooting

### Problema: "Cuenta Contrato inexistente" (codResult: 1002)

**Causas posibles:**
- El clientId no tiene cuentas de servicio asociadas
- El cliente no tiene contratos activos
- Error de tipeo en el clientId

**Solución:**
1. Verificar que el clientId sea correcto
2. Consultar directamente la tabla Account con recordtype='Service'
3. Revisar logs para ver la query ejecutada
4. Verificar que el cliente tenga contratos en Salesforce

### Problema: Error 500 - DB:CONNECTIVITY

**Causas posibles:**
- Base de datos PostgreSQL no disponible
- VPN desconectada (en desarrollo local)
- Pool de conexiones agotado
- Timeout de conexión

**Solución:**
1. Verificar conectividad a PostgreSQL
2. Revisar estado del pool de conexiones
3. Verificar VPN en entorno local
4. Revisar logs de error detallados

### Problema: Respuesta lenta (> 3 segundos)

**Causas posibles:**
- Cliente con muchos contratos/productos
- Pool de conexiones saturado
- Queries SQL no optimizadas
- Alto volumen de datos de autoconsumo (CAU)

**Solución:**
1. Revisar utilización del connection pool
2. Analizar explain plan de las queries
3. Considerar paginación para clientes con muchos contratos
4. Verificar índices en tablas de base de datos

### Problema: Nombres de productos no traducidos

**Causas posibles:**
- Parámetro `language` incorrecto
- Traducciones no disponibles en tabla `NewCo_ProductTranslation__c`
- Código de idioma no soportado

**Solución:**
1. Verificar que el parámetro `language` sea válido (ES, CA, EU, GL, EN)
2. Consultar tabla `NewCo_ProductTranslation__c` para verificar traducciones
3. Usar idioma por defecto (ES) si no hay traducción disponible

---

## 🔐 Seguridad

### Consideraciones de Seguridad

1. **Datos Sensibles:**
   - Números IBAN de cuentas bancarias
   - Direcciones de suministro y facturación
   - Datos de contratos y productos
   - Información técnica de CUPS

2. **GDPR Compliance:**
   - Datos personales del titular de cuenta bancaria
   - Direcciones personales
   - Auditoría de accesos recomendada
   - Retención de datos según normativa

3. **Autenticación:**
   - ⚠️ Actualmente sin autenticación
   - Recomendado: Implementar API Key o OAuth 2.0

4. **Autorización:**
   - Validar que el consumidor tiene permisos para acceder a los datos del cliente
   - Implementar rate limiting
   - Logging de accesos para auditoría

---

## 📝 Notas Adicionales

### Patrón de Respuesta Legacy

⚠️ **Importante:** Este endpoint usa un patrón legacy donde:
- HTTP 200 se retorna tanto para éxito como para "cuenta contrato no encontrada"
- El código de resultado real está en `result.codResult`
- Esto NO sigue el estándar REST

**Recomendación futura:** Migrar a:
- HTTP 200 para éxito
- HTTP 404 para cuenta contrato no encontrada

### Diferencias con GET /client

| Aspecto | GET /client | GET /contractAccountsNew |
|---------|-------------|--------------------------|
| **Queries** | 4 paralelas (Scatter-Gather) | 4 secuenciales |
| **Complejidad** | Media | Alta (jerárquica) |
| **Volumen de datos** | Moderado | Alto (múltiples productos) |
| **Tiempo de respuesta** | ~300ms | ~1s (variable) |
| **Código de error** | "1001" | "1002" |

### Productos Históricos

El endpoint incluye `productHistoricalList` dentro de cada producto activo, que contiene:
- Productos con status '16' o '20' (históricos)
- Asociados al mismo `contractId`
- Útil para ver historial de cambios de producto

### Validación de Productos Activos

Los productos se filtran por múltiples criterios:
- `parentid IS NULL` (solo productos padre, no hijos)
- Status en lista específica de códigos activos
- Excepción para status '08' si `acn_fld_EndDateVersion__c` es null

### Datos de Autoconsumo (CAU)

- Solo se consultan si existen service points
- Pueden haber hasta 3 configuraciones de generación por CAU
- Incluye datos técnicos detallados de instalaciones fotovoltaicas

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-salesforce.raml`
- **Flow Implementation:** `src/main/mule/contract/getContractAccountNew.xml`
- **Main Flow:** `src/main/mule/s-postgresql-salesforce.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`

---

## 📞 Soporte

Para preguntas o issues relacionados con este endpoint:

1. **Documentación técnica:** `docs/memory-bank/`
2. **Logs:** Revisar CloudHub logs o logs locales
3. **APIKit Console:** `http://localhost:8081/console/` (local)

---

**Última actualización:** 2026-05-19  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy
