---
api: "ntgy-newco-s-postgresql-salesforce"
sidebar_position: 3
sidebar_label: "GET /client"
sql_metadata:
  data_sources:
    - database: "newcosalesforce"
      schema: "newcosalesforce"
      type: "PostgreSQL"
      tables:
        - name: "account"
          operations: ["READ"]
---

# Endpoint: GET /client

## 📋 Descripción General

Recupera información completa del cliente desde la base de datos PostgreSQL, incluyendo datos personales, dirección fiscal, preferencias GDPR, contactos asociados, alianzas y gestión personalizada.

**Tipo de API:** System API  
**Capa:** System Layer (API-Led Connectivity)  
**Versión:** 1.42.0  
**Última actualización:** 2026-05-12

---

## 🎯 Propósito del Negocio

Este endpoint proporciona acceso centralizado a toda la información del cliente almacenada en Salesforce (PostgreSQL), permitiendo a las APIs de proceso y experiencia obtener datos completos del cliente de manera eficiente y estandarizada.

### Casos de Uso

1. **Portal de Cliente:** Mostrar información completa del perfil del cliente
2. **CRM:** Consultar datos del cliente para gestión comercial
3. **Atención al Cliente:** Acceso rápido a información del cliente durante llamadas
4. **Facturación:** Obtener datos de contacto y dirección para facturación
5. **Marketing:** Consultar preferencias GDPR para campañas

---

## 🔗 URL del Endpoint

```
GET /api/client
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

#### Cliente Encontrado (`codResult: "0000"`)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "client": {
    "clientId": "001aA000001234567",
    "clientType": "Residencial",
    "addressId": "a0X0900000123456",
    "isIndividualClient": true,
    "name": "María",
    "email": "maria.gonzalez@example.com",
    "phone": "+34912345678",
    "surname": "González",
    "surname2": "López",
    "docTypeId": "nif",
    "docNumber": "12345678A",
    "nationalityId": "ES",
    "birthDate": "1985-03-15",
    "businessName": "",
    "cnae": "",
    "idEnergyUseType": "Doméstico",
    "preferredLanguage": "es",
    "areacliente": true,
    "bypassSign": false,
    "gdprPreferences": {
      "keepData": true,
      "commercialCommunications": false,
      "dataTransfer": false,
      "communications": true,
      "profile": true,
      "advertising": false,
      "survey": false,
      "market": false,
      "alianzas": true
    },
    "naturgyManagement": true,
    "mainManagement": {
      "name": "Juan García",
      "phone": "+34912345678",
      "email": "juan.garcia@naturgy.com"
    },
    "backUpManagement": {
      "name": "Ana Martínez",
      "phone": "+34912345679",
      "email": "ana.martinez@naturgy.com"
    },
    "contacts": [
      {
        "contactId": "003aA000001234567",
        "clientId": "001aA000001234567",
        "name": "María",
        "surname": "González",
        "surname2": "López",
        "docTypeId": "nif",
        "docNumber": "12345678A",
        "nationalityId": "ES",
        "birthdate": "1985-03-15",
        "scheduleId": "morning",
        "preferredPhone": {
          "prefix": "+34",
          "number": "912345678"
        },
        "phone2": {
          "prefix": "+34",
          "number": "912345679"
        },
        "channel": "mail",
        "preferredEmail": "maria.gonzalez@example.com",
        "mailStatus": "Validado",
        "isValidated": true,
        "vlocity_cmt__Type__c": "01",
        "isHolder": true,
        "isPreferred": true,
        "representationTypeId": "holder",
        "activeAgent": true,
        "newcoEstado": "Activo",
        "preferredLanguage": "es"
      }
    ],
    "address": {
      "zipCode": "28001",
      "province": "Madrid",
      "city": "Madrid",
      "way": "Calle Gran Vía",
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
    "alianzas": [
      {
        "id": "ALI-001",
        "name": "Alianza Energía Verde",
        "size": "L",
        "idGow": "GOW123",
        "startDate": "2024-01-01",
        "endDate": "2025-12-31",
        "type": "Alianza_Comercial",
        "updateDate": "2024-03-15T10:30:00Z",
        "accountId": "001aA000001234567"
      }
    ]
  }
}
```

#### Cliente No Encontrado (`codResult: "1001"`)

```json
{
  "result": {
    "codResult": "1001",
    "msgResult": "Cliente Inexistente"
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
- Parámetros con formato inválido
- Ambos parámetros vacíos
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
| `codResult` | string | Código de resultado | `"0000"` (éxito), `"1001"` (no encontrado) |
| `msgResult` | string | Mensaje descriptivo | `"No hay error"`, `"Cliente Inexistente"` |

### Objeto: `client`

#### Información Básica

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `clientId` | string | Sí | ID de cuenta Salesforce (SFID) |
| `clientType` | string | No | Tipo de cliente (ej: "Residencial") |
| `addressId` | string | No | ID de dirección fiscal (Premises SFID) |
| `isIndividualClient` | boolean | Sí | true = Consumidor, false = Empresa |
| `name` | string | Sí | Nombre (del contacto principal) |
| `email` | string | No | Email principal |
| `phone` | string | No | Teléfono principal |
| `surname` | string | Sí | Primer apellido |
| `surname2` | string | No | Segundo apellido |

#### Identificación

| Campo | Tipo | Descripción | Valores |
|-------|------|-------------|---------|
| `docTypeId` | string | Tipo de documento (minúsculas) | `"nif"`, `"nie"`, `"passport"`, `"cif"` |
| `docNumber` | string | Número de documento | Ej: `"12345678A"` |
| `nationalityId` | string | Código de nacionalidad | Ej: `"ES"` |
| `birthDate` | date | Fecha de nacimiento | Formato: `"YYYY-MM-DD"` |

#### Información Empresarial

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `businessName` | string | Razón social (para empresas) |
| `cnae` | string | Código CNAE de actividad económica |
| `idEnergyUseType` | string | Tipo de uso de energía |

#### Preferencias

| Campo | Tipo | Descripción | Valores |
|-------|------|-------------|---------|
| `preferredLanguage` | string | Idioma preferido | `"es"`, `"ca"`, `"eu"`, `"gl"`, `"en"` |
| `areacliente` | boolean | Usuario de área cliente activo | |
| `bypassSign` | boolean | Omitir firma Omega | |

#### GDPR Preferences

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `keepData` | boolean | Consentimiento para conservar datos |
| `commercialCommunications` | boolean | Comunicaciones comerciales |
| `dataTransfer` | boolean | Transferencia de datos a terceros |
| `communications` | boolean | Comunicaciones generales |
| `profile` | boolean | Perfilado |
| `advertising` | boolean | Publicidad |
| `survey` | boolean | Encuestas |
| `market` | boolean | Estudios de mercado |
| `alianzas` | boolean | Alianzas con partners |

#### Gestión Personalizada

**mainManagement / backUpManagement:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `name` | string | Nombre del gestor |
| `phone` | string | Teléfono del gestor |
| `email` | string | Email del gestor |

### Array: `contacts`

Lista de contactos asociados al cliente.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `contactId` | string | ID del contacto Salesforce |
| `clientId` | string | ID del cliente asociado |
| `name` | string | Nombre del contacto |
| `surname` | string | Primer apellido |
| `surname2` | string | Segundo apellido |
| `docTypeId` | string | Tipo de documento |
| `docNumber` | string | Número de documento |
| `nationalityId` | string | Código de nacionalidad |
| `birthdate` | date | Fecha de nacimiento |
| `scheduleId` | string | Horario preferente: `"morning"`, `"afternoon"` |
| `preferredPhone` | object | `{prefix, number}` - Teléfono preferido |
| `phone2` | object | `{prefix, number}` - Teléfono secundario |
| `channel` | string | Canal preferente: `"mail"`, `"phone"` |
| `preferredEmail` | string | Email preferido |
| `mailStatus` | string | Estado del cambio de email |
| `isValidated` | boolean | Si el contacto está validado |
| `vlocity_cmt__Type__c` | string | Código de rol (01-05) |
| `isHolder` | boolean | Si es titular de la cuenta |
| `isPreferred` | boolean | Si es el contacto preferido |
| `representationTypeId` | string | Tipo de representación |
| `activeAgent` | boolean | Si el agente está activo |
| `newcoEstado` | string | Estado de login en Omega |
| `preferredLanguage` | string | Idioma preferido |

#### Mapeo de Roles (vlocity_cmt__Type__c)

| Código | representationTypeId | Descripción |
|--------|---------------------|-------------|
| `"01"` | `"holder"` | Titular |
| `"02"` | `"accountHolder"` | Titular de cuenta |
| `"03"` | `"representative"` | Representante |
| `"04"` | `"authorizedAll"` | Autorizado total |
| `"05"` | `"authorizedRead"` | Autorizado lectura |

### Objeto: `address`

Dirección fiscal del cliente.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `zipCode` | string | Código postal |
| `province` | string | Provincia |
| `city` | string | Ciudad |
| `way` | string | Nombre de la vía |
| `number` | string | Número de la vía |
| `floor` | string | Piso |
| `door` | string | Puerta |
| `portal` | string | Portal |
| `stairs` | string | Escalera |
| `provinceCode` | string | Código de provincia |
| `codMunIne` | string | Código INE del municipio |
| `premiseTypeCode` | string | Código de tipo de inmueble |
| `viaId` | string | Identificador de vía |

### Array: `alianzas`

Lista de alianzas del cliente.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | string | Código de alianza |
| `name` | string | Nombre de la alianza |
| `size` | string | Tamaño/nivel de la alianza |
| `idGow` | string | Identificador en sistema GOW |
| `startDate` | date | Fecha de inicio |
| `endDate` | date | Fecha de fin |
| `type` | string | Tipo de alianza |
| `updateDate` | datetime | Fecha de última actualización |
| `accountId` | string | ID de cuenta asociada |

---

## 🔧 Implementación Técnica

### Arquitectura

```
┌─────────────────┐
│   API Consumer  │
└────────┬────────┘
         │ GET /api/client?clientId=xxx
         ▼
┌─────────────────────────────────────┐
│  APIKit Router                      │
│  (s-postgresql-salesforce-main)     │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Flow: get:\client:config           │
│  - Extrae parámetros                │
│  - Llama a sub-flow                 │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Sub-flow: getClientInfoSub_Flow    │
│  - Valida clientId/clientCode       │
│  - Ejecuta 4 queries paralelas      │
│  - Transforma respuesta             │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Scatter-Gather (4 queries)         │
│  ├─ Account + Premises              │
│  ├─ GDPR Consents                   │
│  ├─ Contacts + Relations            │
│  └─ Alianzas                        │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  DataWeave Transformation           │
│  - Mapeo de campos                  │
│  - Formateo de datos                │
│  - Construcción de JSON             │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Response (HTTP 200)                │
│  - codResult: "0000" o "1001"       │
└─────────────────────────────────────┘
```

### Queries SQL Ejecutadas

#### 1. Query Principal (Account + Premises)

```sql
SELECT
  account.sfid as accountId,
  account.Name,
  account.newco_cnaecode__c,
  account.acn_fld_DocumentTypeG__c,
  account.acn_fld_DocumentNumber__c,
  -- ... más campos ...
  premises.vlocity_cmt__PostalCode__c,
  premises.vlocity_cmt__State__c,
  -- ... más campos de premises ...
FROM newcosalesforce.Account AS account
LEFT JOIN newcosalesforce.vlocity_cmt__Premises__c AS premises 
  ON premises.sfid = account.newco_fiscaladdress__c
WHERE account.sfid = :clientId
```

#### 2. Query GDPR

```sql
SELECT 
  NewCo_Tipo_de_consentimiento__c,
  NewCo_Estado__c
FROM newcosalesforce.NewCo_Consentimiento_GDPR__c 
WHERE newco_account__c = :clientId
```

#### 3. Query Contacts

```sql
SELECT
  contact.sfid as contactId,
  contact.NewCo_Apellido1__c,
  -- ... más campos ...
  accountcontactrelation.roles as rol,
  accountcontactrelation.newco_estadologinomega__c as loginStatus
FROM newcosalesforce.accountcontactrelation as accountcontactrelation
INNER JOIN newcosalesforce.Contact as contact 
  ON contact.sfid = accountcontactrelation.contactid 
WHERE accountcontactrelation.accountid = :clientId
```

#### 4. Query Alianzas

```sql
SELECT
  alianzas.newco_codigoalianza__c,
  alianzas.newco_nombrealianza__c,
  -- ... más campos ...
  tableRecordType.developername  
FROM newcosalesforce.newco_cuenta_alianza__c as alianzas
INNER JOIN newcosalesforce.recordtype tableRecordType 
  ON tableRecordType.sfid = alianzas.recordtypeid
WHERE alianzas.newco_account__c = :clientId
```

### Transformaciones DataWeave

#### Mapeo de Idiomas

```dataweave
content.vlocity_cmt__preferredlanguage__c match {
  case "Castellano" -> "es"
  case "Catalán" -> "ca"
  case "Euskera" -> "eu"
  case "Gallego" -> "gl"
  case "Inglés" -> "en"
  else -> "es"
}
```

#### Formateo de Teléfonos

```dataweave
fun updatePhone(phoneDict) =
  {
    "prefix": dw::core::Strings::substring(phoneDict, 0, 3) default "",
    "number": dw::core::Strings::substring(phoneDict, 3, sizeOf(phoneDict)) default ""
  }
```

#### Mapeo de Horarios

```dataweave
item.newco_horariopreferente__c match {
  case "01" -> "morning"
  case "02" -> "afternoon"
  else -> null
}
```

---

## 📊 Performance

### Métricas Esperadas

| Métrica | Valor Objetivo | Valor Actual |
|---------|----------------|--------------|
| Tiempo de respuesta (p50) | < 500ms | ~300ms |
| Tiempo de respuesta (p95) | < 1s | ~600ms |
| Throughput | 1000+ req/hora | Validar en producción |
| Error rate | < 1% | < 0.5% |
| Disponibilidad | 99.9% | Validar en producción |

### Optimizaciones Implementadas

1. **Scatter-Gather Pattern:** 4 queries ejecutadas en paralelo
2. **Connection Pooling:** Pool de 5-30 conexiones PostgreSQL
3. **Prepared Statement Cache:** 100 statements cacheados
4. **Fetch Size:** 100 registros por batch

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consulta por clientId

```bash
curl -X GET "http://localhost:8081/api/client?clientId=001aA000001234567" \
  -H "Content-Type: application/json"
```

### Ejemplo 2: Consulta por clientCode

```bash
curl -X GET "http://localhost:8081/api/client?clientCode=12345678A" \
  -H "Content-Type: application/json"
```

### Ejemplo 3: Con ambos parámetros (clientId tiene prioridad)

```bash
curl -X GET "http://localhost:8081/api/client?clientId=001aA000001234567&clientCode=12345678A" \
  -H "Content-Type: application/json"
```

### Ejemplo 4: Cliente no encontrado

```bash
curl -X GET "http://localhost:8081/api/client?clientId=999999999999999" \
  -H "Content-Type: application/json"
```

**Respuesta:**
```json
{
  "result": {
    "codResult": "1001",
    "msgResult": "Cliente Inexistente"
  }
}
```

---

## 🐛 Troubleshooting

### Problema: "Cliente Inexistente" (codResult: 1001)

**Causas posibles:**
- El clientId no existe en la base de datos
- El clientCode no corresponde a ningún cliente
- Error de tipeo en los parámetros

**Solución:**
1. Verificar que el clientId/clientCode sea correcto
2. Consultar directamente en la base de datos
3. Revisar logs para ver la query ejecutada

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

### Problema: Respuesta lenta (> 2 segundos)

**Causas posibles:**
- Pool de conexiones saturado
- Queries SQL no optimizadas
- Alto volumen de datos en respuesta
- Red lenta

**Solución:**
1. Revisar utilización del connection pool
2. Analizar explain plan de las queries
3. Considerar paginación para grandes volúmenes
4. Verificar latencia de red

---

## 🔐 Seguridad

### Consideraciones de Seguridad

1. **Datos Sensibles:**
   - Números de documento
   - Emails
   - Teléfonos
   - Direcciones

2. **GDPR Compliance:**
   - Respeta preferencias de consentimiento
   - Auditoría de accesos recomendada
   - Retención de datos según normativa

3. **Autenticación:**
   - ⚠️ Actualmente sin autenticación
   - Recomendado: Implementar API Key o OAuth 2.0

4. **Autorización:**
   - Validar que el consumidor tiene permisos
   - Implementar rate limiting

---

## 📝 Notas Adicionales

### Patrón de Respuesta Legacy

⚠️ **Importante:** Este endpoint usa un patrón legacy donde:
- HTTP 200 se retorna tanto para éxito como para "cliente no encontrado"
- El código de resultado real está en `result.codResult`
- Esto NO sigue el estándar REST

**Recomendación futura:** Migrar a:
- HTTP 200 para éxito
- HTTP 404 para cliente no encontrado

### Lógica de Fallback para Contactos

Si la tabla `AccountContactRelation` no tiene datos, el sistema:
1. Consulta directamente la tabla `Contact`
2. Usa `loginStatus: 'Pendiente'` como valor por defecto
3. Usa `vlocity_cmt__Type__c` en lugar de `roles`

### Validación de "undefined"

El código valida explícitamente el string `"undefined"`:
```
vars.clientId == "undefined"
```

Esto sugiere que algunos clientes envían el literal "undefined" en lugar de omitir el parámetro.

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-salesforce.raml`
- **Flow Implementation:** `src/main/mule/clients/getClients.xml`
- **Main Flow:** `src/main/mule/s-postgresql-salesforce.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`

---

## 📞 Soporte

Para preguntas o issues relacionados con este endpoint:

1. **Documentación técnica:** `docs/memory-bank/`
2. **Logs:** Revisar CloudHub logs o logs locales
3. **APIKit Console:** `http://localhost:8081/console/` (local)

---

**Última actualización:** 2026-05-12  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy
