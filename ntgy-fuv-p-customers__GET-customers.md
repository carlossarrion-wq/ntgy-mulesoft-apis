---
api: "ntgy-fuv-p-customers"
sidebar_position: 2
sidebar_label: "GET /customers"
description: "Obtiene información completa del cliente y sus puntos de suministro mediante validación de token multi-origen y consulta de tenencia en Imperial y NewCo"
keywords: [customers, cliente, tenencia, multi-origen, token, validación, Imperial, NewCo, contratos, suministros]
tags: [Customers, Process API, Orquestación, Multi-Backend, Autenticación]
endpoint_dependencies:
  - method: "GET"
    api_name: "MUL-COM-CLI-S-AC"
    path: "/validateToken"
    purpose: "Validación de token de autenticación para origen AC (Área Cliente)"
  - method: "GET"
    api_name: "MUL-COM-CLI-S-AC"
    path: "/customersRols"
    purpose: "Obtención de roles del usuario para determinar permisos (US_CC)"
  - method: "GET"
    api_name: "MUL-COM-CLI-S-AC"
    path: "/customersByDoc"
    purpose: "Consulta de tenencia Imperial por documento (para usuarios US_CC)"
  - method: "GET"
    api_name: "MUL-COM-CLI-S-AC"
    path: "/customersByUserId"
    purpose: "Consulta de tenencia Imperial por userId (para usuarios no US_CC)"
  - method: "GET"
    api_name: "ntgy-newco-s-postgresql-salesforce"
    path: "/client"
    purpose: "Consulta de datos del cliente NewCo desde PostgreSQL/Salesforce"
  - method: "GET"
    api_name: "ntgy-newco-s-postgresql-salesforce"
    path: "/contractAccountsNew"
    purpose: "Consulta de contratos y productos activos del cliente NewCo"
  - method: "POST"
    api_name: "validateToken-main"
    path: "/validateToken"
    purpose: "Validación de tokens Auth0 (AT), AWS Cognito (CG) y Azure AD (AZ)"
---

# GET /customers

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint obtiene la información completa del cliente, incluyendo sus puntos de suministro, contratos, líneas de contrato y productos asociados. Implementa un sistema de **validación de tokens multi-origen** (AC, Auth0, Cognito, Azure AD) y **consulta de tenencia dual** (Imperial y NewCo), agregando y normalizando los datos de ambas comercializadoras en una única respuesta.

### Tipo de API

**Capa:** Process API (API-Led Connectivity)

**Función:** Orquestación de consultas de tenencia con validación de autenticación multi-origen

### Patrón de Integración

**Tipo:** Orquestación Compleja con Enrutamiento Condicional

**Descripción:** El endpoint implementa un patrón de orquestación sofisticado que:
1. Valida el token de autenticación según su origen (4 orígenes soportados)
2. Determina permisos del usuario (rol US_CC para AC)
3. Consulta tenencia en paralelo para ambas comercializadoras (Imperial y NewCo)
4. Agrega y normaliza los datos en una respuesta unificada

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    E[Experience APIs<br/>Portal FUA, Mobile Apps]
    P[Process API<br/>ntgy-fuv-p-customers<br/>GET /customers]
    S1[System API<br/>MUL-COM-CLI-S-AC<br/>Área Cliente Imperial]
    S2[System API<br/>ntgy-newco-s-postgresql-salesforce<br/>Datos NewCo]
    S3[System API<br/>validateToken-main<br/>Validación Tokens]
    
    E -->|Request con Token| P
    P -->|Validar Token AC| S1
    P -->|Validar Token AT/CG/AZ| S3
    P -->|Consultar Tenencia Imperial| S1
    P -->|Consultar Tenencia NewCo| S2
    S1 -->|Datos Imperial| P
    S2 -->|Datos NewCo| P
    S3 -->|Token Válido| P
    P -->|Response Agregada| E
    
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:3px
    style S1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style S3 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Consulta de Tenencia Multi-Comercializadora**
   - Cliente consulta sus contratos en ambas comercializadoras (Imperial y NewCo)
   - Sistema agrega información de múltiples fuentes en una vista unificada
   - Beneficio: Visión completa del cliente sin múltiples consultas

2. **Validación de Autenticación Multi-Origen**
   - Soporte para 4 orígenes de token: AC (Área Cliente), Auth0, AWS Cognito, Azure AD
   - Validación específica según el origen del token
   - Beneficio: Flexibilidad para diferentes canales de autenticación

3. **Gestión de Permisos por Rol**
   - Usuarios US_CC (Customer Care) consultan por documento
   - Usuarios estándar consultan por userId
   - Beneficio: Control de acceso granular según rol del usuario

### 🔗 Endpoints Dependientes

Este Process API consume los siguientes System APIs:

| System API | Endpoint | Método | Propósito |
|------------|----------|--------|-----------|
| `MUL-COM-CLI-S-AC` | `/validateToken` | GET | Validación de token AC |
| `MUL-COM-CLI-S-AC` | `/customersRols` | GET | Obtención de roles del usuario |
| `MUL-COM-CLI-S-AC` | `/customersByDoc` | GET | Tenencia Imperial por documento |
| `MUL-COM-CLI-S-AC` | `/customersByUserId` | GET | Tenencia Imperial por userId |
| `ntgy-newco-s-postgresql-salesforce` | `/client` | GET | Datos del cliente NewCo |
| `ntgy-newco-s-postgresql-salesforce` | `/contractAccountsNew` | GET | Contratos y productos NewCo |
| `validateToken-main` | `/validateToken` | POST | Validación tokens Auth0/Cognito/Azure |

### Lógica de Orquestación

```mermaid
graph TB
    Start([GET /customers]) --> ValidateOrigin{Origen del Token?}
    
    ValidateOrigin -->|AC| ValidateAC[Validar Token AC]
    ValidateOrigin -->|AT| ValidateAT[Validar Token Auth0]
    ValidateOrigin -->|CG| ValidateCG[Validar Token Cognito]
    ValidateOrigin -->|AZ| ValidateAZ[Validar Token Azure AD]
    
    ValidateAC --> GetRoles[Obtener Roles Usuario]
    GetRoles --> CheckRole{Rol US_CC?}
    
    CheckRole -->|Sí| QueryByDoc[Consultar por Documento]
    CheckRole -->|No| QueryByUser[Consultar por UserId]
    
    ValidateAT --> QueryTenure
    ValidateCG --> QueryTenure
    ValidateAZ --> QueryTenure
    QueryByDoc --> QueryTenure
    QueryByUser --> QueryTenure
    
    QueryTenure{Comercializadora?}
    QueryTenure -->|Imperial| GetImperial[Consultar S-AC]
    QueryTenure -->|NewCo| GetNewCo[Consultar S-PostgreSQL-Salesforce]
    QueryTenure -->|Ambas| GetBoth[Consultar Ambas en Paralelo]
    
    GetImperial --> Aggregate[Agregar Respuestas]
    GetNewCo --> Aggregate
    GetBoth --> Aggregate
    
    Aggregate --> Transform[Transformar y Normalizar]
    Transform --> End([Response JSON])
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style End fill:#e8f5e9,stroke:#2e7d32
    style ValidateOrigin fill:#fff3e0,stroke:#f57c00
    style QueryTenure fill:#fff3e0,stroke:#f57c00
```

**Pasos del Proceso:**

1. **Validación de Token según Origen:**
   - **AC (Área Cliente):** Valida token + obtiene roles + determina tipo de consulta
   - **AT (Auth0):** Valida token JWT con validateToken-main
   - **CG (AWS Cognito):** Valida token con validateToken-main
   - **AZ (Azure AD):** Valida token con validateToken-main

2. **Consulta de Tenencia:**
   - **Imperial:** Consulta a MUL-COM-CLI-S-AC (por documento o userId)
   - **NewCo:** Consulta a ntgy-newco-s-postgresql-salesforce (cliente + contratos)

3. **Agregación y Transformación:**
   - Combina datos de ambas comercializadoras
   - Normaliza estados de líneas de contrato
   - Filtra CUPS por tipo de producto (Electricidad/Gas)
   - Ofusca datos sensibles (teléfonos)

### Transformaciones Aplicadas

| Campo Original | Campo Transformado | Transformación |
|----------------|-------------------|----------------|
| `phone1` (NewCo) | `phone1` | Ofuscación: últimos 9 dígitos |
| `contractLineStatus` (NewCo) | `contractLineStatus` | Mapeo de códigos: 01→"Documentación recibida", 02→"Activo", etc. |
| `electricityCUPS` | `electricityCUPS` | Filtrado: solo si existen líneas de Electricidad |
| `gasCUPS` | `gasCUPS` | Filtrado: solo si existen líneas de Gas |
| `supplyPoints` | `supplyPoints` | Normalización de estructura entre Imperial y NewCo |

### Contexto Empresarial

- **Dominio:** Gestión de Clientes y Contratos
- **Sector:** Energía - Utilities
- **División:** Naturgy España - Imperial y NewCo
- **Proceso de Negocio:** Consulta unificada de tenencia multi-comercializadora

### Stakeholders

- **Clientes finales:** Consultan su información de contratos y suministros
- **Agentes de atención al cliente:** Validan información del cliente
- **Sistemas de autoservicio:** Portales web y aplicaciones móviles
- **Equipos de desarrollo:** Consumen API para integraciones

### Valor de Negocio

- **Unificación de datos:** Vista única del cliente en ambas comercializadoras
- **Flexibilidad de autenticación:** Soporte para múltiples orígenes de token
- **Control de acceso:** Gestión de permisos por rol de usuario
- **Experiencia mejorada:** Respuesta única en lugar de múltiples consultas

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Portales web de autoservicio (FUA)
- Aplicaciones móviles de clientes
- Sistemas de agentes de call center
- Integraciones de terceros autorizados
- Sistemas internos de gestión

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/customers` |
| **Development** | `https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io` | `https://ntgy-fuv-p-customers-dev.us-e2.cloudhub.io/api/customers` |
| **UAT** | `https://ntgy-fuv-p-customers-uat.us-e2.cloudhub.io` | `https://ntgy-fuv-p-customers-uat.us-e2.cloudhub.io/api/customers` |
| **Production** | `https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io` | `https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io/api/customers` |

### Método HTTP

```
GET /api/customers
```

### Parámetros de Entrada

#### Headers

| Header | Tipo | Requerido | Descripción | Ejemplo |
|--------|------|-----------|-------------|---------|
| `Authorization` | string | ✅ Sí | Token de autenticación (Bearer token) | `Bearer ca901dae-f60c-406d-9519-8e6ce5c674af` |

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Valores Permitidos | Ejemplo |
|-----------|------|-----------|-------------|-------------------|---------|
| `docNumber` | string | ❌ No | Número de documento del cliente (NIF) | Cualquier NIF válido | `00000000T` |
| `origin` | string | ✅ Sí | Origen del token de autenticación | `AC`, `AT`, `CG`, `AZ` | `AC` |
| `comercializadora` | string | ✅ Sí | Comercializadora de destino | `imperial`, `newco` | `imperial` |

**Descripción de Orígenes:**
- **AC:** Área de Cliente (sistema legacy)
- **AT:** Auth0 Naturgy
- **CG:** AWS Cognito
- **AZ:** Azure AD

### Respuestas

#### 200 OK - Éxito

**Estructura de Respuesta:**

⚠️ **NOTA IMPORTANTE:** La estructura de respuesta depende de qué comercializadora(s) devuelven datos. El array `customer` puede contener 0, 1 o 2 elementos según disponibilidad de datos en Imperial y/o NewCo.

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "customer": [
    {
      "origin": "imperial",
      "tenure": {
        "customerId": "string",
        "customerName": "string",
        "documentType": "string",
        "documentNumber": "string",
        "email": "string",
        "phone1": "string",
        "phone2": "string (opcional)",
        "address": "string",
        "postalCode": "string",
        "city": "string",
        "province": "string",
        "country": "string",
        "supplyPoints": [
          {
            "supplyPointId": "string",
            "address": "string",
            "postalCode": "string",
            "city": "string",
            "province": "string",
            "country": "string",
            "latitude": "string (opcional)",
            "longitude": "string (opcional)",
            "contracts": [
              {
                "contractId": "string",
                "contractNumber": "string",
                "contractStatus": "string",
                "contractType": "string",
                "startDate": "string (formato: yyyy-MM-dd)",
                "endDate": "string (formato: yyyy-MM-dd, opcional)",
                "electricityCUPS": "string (condicional - solo si hay productos de Electricidad)",
                "gasCUPS": "string (condicional - solo si hay productos de Gas)",
                "contractLines": {
                  "items": [
                    {
                      "contractLineId": "string",
                      "lineNumber": "string",
                      "productId": "string",
                      "productName": "string",
                      "productCategory": "Electricidad | Gas",
                      "productType": "string",
                      "contractLineStatus": "string",
                      "startDate": "string (formato: yyyy-MM-dd)",
                      "endDate": "string (formato: yyyy-MM-dd, opcional)",
                      "power": "string (solo para Electricidad)",
                      "tariff": "string"
                    }
                  ]
                }
              }
            ]
          }
        ]
      }
    },
    {
      "origin": "newco",
      "tenure": {
        "customerId": "string",
        "firstName": "string",
        "lastName": "string",
        "documentType": "string",
        "documentNumber": "string",
        "email": "string",
        "emailStatus": "boolean",
        "phone1": "string (últimos 9 dígitos)",
        "preferredLanguage": "string",
        "supplyPoints": [
          {
            "supplyPointId": "string",
            "address": {
              "street": "string",
              "number": "string",
              "floor": "string",
              "door": "string",
              "postalCode": "string",
              "city": "string",
              "province": "string",
              "country": "string"
            },
            "contracts": [
              {
                "contractId": "string",
                "contractAccountId": "string",
                "contractStatus": "string",
                "startDate": "string",
                "endDate": "string",
                "electricityCUPS": "string (condicional)",
                "gasCUPS": "string (condicional)",
                "contractLines": {
                  "items": [
                    {
                      "lineId": "string",
                      "productId": "string",
                      "productName": "string",
                      "productCategory": "Electricidad | Gas",
                      "contractLineStatus": "Documentación recibida | Activo | Alta Activa | Anulado | Inactivo",
                      "startDate": "string",
                      "endDate": "string"
                    }
                  ]
                }
              }
            ]
          }
        ]
      },
      "partnerships": [
        {
          "partnershipId": "string",
          "partnerName": "string",
          "partnershipType": "string",
          "status": "string",
          "startDate": "string"
        }
      ]
    }
  ]
}
```

**Casos Especiales de Respuesta:**

1. **Cliente solo en Imperial:**
```json
{
  "result": {"codResult": "0000", "msgResult": "No hay error"},
  "customer": [
    {"origin": "imperial", "tenure": {...}}
  ]
}
```

2. **Cliente solo en NewCo:**
```json
{
  "result": {"codResult": "0000", "msgResult": "No hay error"},
  "customer": [
    {"origin": "newco", "tenure": {...}, "partnerships": [...]}
  ]
}
```

3. **Cliente en ambas comercializadoras:**
```json
{
  "result": {"codResult": "0000", "msgResult": "No hay error"},
  "customer": [
    {"origin": "imperial", "tenure": {...}},
    {"origin": "newco", "tenure": {...}, "partnerships": [...]}
  ]
}
```

4. **Cliente no encontrado en ninguna:**
```json
{
  "result": {"codResult": "0000", "msgResult": "No hay error"},
  "customer": []
}
```

5. **Cliente NewCo sin productos activos:**
```json
{
  "result": {"codResult": "0000", "msgResult": "No hay error"},
  "customer": [
    {
      "origin": "newco",
      "tenure": null,
      "partnerships": null
    }
  ]
}
```

**Campos Condicionales Importantes:**

| Campo | Condición | Descripción |
|-------|-----------|-------------|
| `electricityCUPS` | Solo si `contractLines` contiene productos de Electricidad | CUPS de electricidad filtrado |
| `gasCUPS` | Solo si `contractLines` contiene productos de Gas | CUPS de gas filtrado |
| `phone1` (NewCo) | Siempre ofuscado | Solo últimos 9 dígitos visibles |
| `partnerships` | Solo para NewCo | Array de alianzas del cliente |
| `tenure` (NewCo) | `null` si no hay productos activos | Validado por `hasActiveProducts()` |

**Fuentes de Datos:**

- **Imperial:** Respuesta directa de System API `MUL-COM-CLI-S-AC` (endpoints `/customersByDoc` o `/customersByUserId`)
- **NewCo:** Construida por funciones DataWeave:
  - `buildResponseGetCustomerInfo(customer, productlist, emailStatus)` - cuando hay productos activos
  - `buildResponseCustomerNotExist()` - cuando no hay productos activos o cliente no existe
  - Validación: `hasActiveProducts(productlist)` determina qué función usar

#### 401 Unauthorized - Token Inválido

```json
{
  "result": {
    "codResult": "AUTH-0001",
    "msgResult": "El token de autenticación recibido es incorrecto o ha caducado",
    "detail": "El token de autenticación recibido es incorrecto o ha caducado"
  }
}
```

#### 400 Bad Request - Error Técnico

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Error técnico"
  }
}
```

#### 400 Bad Request - Parámetro No Encontrado

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Parámetro no encontrado"
  }
}
```

### Autenticación

- **Tipo:** Bearer Token (multi-origen)
- **Header:** `Authorization: Bearer {token}`
- **Orígenes Soportados:**
  - **AC:** Token de Área Cliente (validado por MUL-COM-CLI-S-AC)
  - **AT:** Token JWT de Auth0 (validado por validateToken-main)
  - **CG:** Token de AWS Cognito (validado por validateToken-main)
  - **AZ:** Token de Azure AD (validado por validateToken-main)

### Timeouts

| Tipo | Valor |
|------|-------|
| **Request Timeout** | 30 segundos |
| **Connection Timeout** | 10 segundos |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request]) --> LogStart[Log Entrada]
    LogStart --> ExtractParams[Extraer Parámetros<br/>docNumber, origin, token, comercializadora]
    ExtractParams --> LogToken[Log Token]
    LogToken --> ValidateOrigin{Validar Origen Token}
    
    ValidateOrigin -->|AC| ValidateTokenAC[Validar Token AC<br/>GET /validateToken]
    ValidateTokenAC --> CheckResult{Result = RESULT_00?}
    CheckResult -->|Sí| GetRoles[Obtener Roles<br/>GET /customersRols]
    CheckResult -->|No| ErrorAuth[Error AUTH-0001]
    
    GetRoles --> ExtractUserId[Extraer userId]
    ExtractUserId --> CheckRole{Rol US_CC?}
    CheckRole -->|Sí| SetUSCC1[IS_US_CC = true]
    CheckRole -->|No| SetUSCC2[IS_US_CC = false]
    
    ValidateOrigin -->|AT| ValidateTokenAT[Validar Token Auth0<br/>POST validateToken-main]
    ValidateOrigin -->|CG| ValidateTokenCG[Validar Token Cognito<br/>POST validateToken-main]
    ValidateOrigin -->|AZ| ValidateTokenAZ[Validar Token Azure AD<br/>POST validateToken-main]
    
    ValidateTokenAT --> CheckValid1{Status = valid?}
    ValidateTokenCG --> CheckValid2{Status = valid?}
    ValidateTokenAZ --> CheckValid3{Status = valid?}
    
    CheckValid1 -->|Sí| SetUSCC3[IS_US_CC = true]
    CheckValid2 -->|Sí| SetUSCC4[IS_US_CC = true]
    CheckValid3 -->|Sí| SetUSCC5[IS_US_CC = true]
    
    CheckValid1 -->|No| ErrorAuth
    CheckValid2 -->|No| ErrorAuth
    CheckValid3 -->|No| ErrorAuth
    
    SetUSCC1 --> QueryTenure[Flow-ref:<br/>getCustomerByDoc-obtenerTenencia]
    SetUSCC2 --> QueryTenure
    SetUSCC3 --> QueryTenure
    SetUSCC4 --> QueryTenure
    SetUSCC5 --> QueryTenure
    
    QueryTenure --> CheckComer{Comercializadora?}
    CheckComer -->|Imperial| QueryImperial[Subflow:<br/>getCustomerByDoc-obtenerTenenciaImperial]
    CheckComer -->|NewCo| QueryNewCo[Subflow:<br/>getCustomerByDoc-obtenerTenenciaNewCo]
    
    QueryImperial --> CheckRoleImperial{IS_US_CC?}
    CheckRoleImperial -->|Sí| GetByDoc[GET /customersByDoc]
    CheckRoleImperial -->|No| GetByUserId[GET /customersByUserId]
    
    QueryNewCo --> GetClient[GET /client]
    GetClient --> CheckExists{Cliente existe?}
    CheckExists -->|Sí| GetContracts[GET /contractAccountsNew]
    CheckExists -->|No| BuildNotExist[Build Response Not Exist]
    
    GetContracts --> CheckProducts{Productos activos?}
    CheckProducts -->|Sí| BuildResponse[Build Response Exist]
    CheckProducts -->|No| BuildNotExist
    
    GetByDoc --> Aggregate[Agregar y Transformar]
    GetByUserId --> Aggregate
    BuildResponse --> Aggregate
    BuildNotExist --> Aggregate
    
    Aggregate --> LogEnd[Log Salida]
    LogEnd --> End([HTTP Response])
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style End fill:#e8f5e9,stroke:#2e7d32
    style ErrorAuth fill:#ffebee,stroke:#c62828
    style ValidateOrigin fill:#fff3e0,stroke:#f57c00
    style CheckComer fill:#fff3e0,stroke:#f57c00
```

### Transformaciones DataWeave

#### Funciones DataWeave Reales (Código Fuente Verificado)

**Ubicación:** `src/main/resources/dataweave/customer/`

##### 1. buildResponseGetCustomerInfo() - Mappings.dwl

**Propósito:** Construir respuesta completa cuando el cliente tiene productos activos

**Firma:**
```dataweave
fun buildResponseGetCustomerInfo(customer, productlist, emailStatus)
```

**Parámetros:**
- `customer`: Respuesta de `/client` (datos del cliente desde Salesforce)
- `productlist`: Respuesta de `/contractAccountsNew` (contratos y productos)
- `emailStatus`: Boolean indicando si el email está validado

**Estructura de Salida:**
```dataweave
{
    customer: {
        customerId: customer.client.clientId,
        firstName: customer.client.firstName,
        lastName: customer.client.lastName,
        documentType: customer.client.documentType,
        documentNumber: customer.client.documentNumber,
        email: customer.client.contacts[0].email,
        emailStatus: emailStatus,
        phone1: customer.client.contacts[0].phone,
        preferredLanguage: customer.client.preferredLanguage,
        supplyPoints: [mapeo de productlist.contractAccounts.items]
    },
    partnerships: customer.client.partnerships
}
```

##### 2. buildResponseCustomerNotExist() - Mappings.dwl

**Propósito:** Construir respuesta cuando el cliente no existe o no tiene productos activos

**Firma:**
```dataweave
fun buildResponseCustomerNotExist()
```

**Estructura de Salida:**
```dataweave
{
    customer: null,
    partnerships: null
}
```

##### 3. hasActiveProducts() - Utils.dwl

**Propósito:** Validar si el cliente tiene productos/contratos activos

**Firma:**
```dataweave
fun hasActiveProducts(productlist)
```

**Lógica:**
```dataweave
!isEmpty(productlist.contractAccounts.items) and
productlist.contractAccounts.items[0].contracts != null and
!isEmpty(productlist.contractAccounts.items[0].contracts.items)
```

**Retorna:** `true` si hay contratos activos, `false` en caso contrario

#### Transformación Principal (Agregación de Tenencias)

**Ubicación:** `getCustomerByDoc-main.xml` (líneas 174-264)

**Propósito:** Combinar y normalizar datos de Imperial y NewCo en una respuesta unificada

**Código DataWeave:**

```dataweave
%dw 2.0
output application/json
---
{
    "result": {
        "codResult": "0000",
        "msgResult": "No hay error"
    },
    "customer": [
        // Tenencia Imperial
        ({
            "origin": "imperial",
            "tenure": vars.tenenciaImperial.customer mapObject ((value, key, index) ->
                (if (key ~= "supplyPoints" and value != null) 
                    (key): value mapObject(value, key, index) ->
                        (key): value map(supplyPoint, key) ->
                            supplyPoint mapObject(value, key, index) ->
                                (if (key ~= "contracts" and value != null) 
                                    (key): value mapObject(value, key, index) ->
                                        (key): value map(contract, key) ->
                                            contract mapObject(value, key, index) ->
                                                // Filtrar CUPS por tipo de producto
                                                (if (key ~= "electricityCUPS" and value != null) 
                                                    ((key):value) if(!isEmpty(contract..contractLines.items[0] filter($.productCategory == 'Electricidad')))
                                                else if (key ~= "gasCUPS" and value != null) 
                                                    ((key):value) if(!isEmpty(contract..contractLines.items[0] filter($.productCategory == 'Gas')))
                                                else (key): value)
                                else (key): value)
                else (key): value))
        }) if (vars.tenenciaImperial != null and vars.tenenciaImperial.customer != null),
        
        // Tenencia NewCo
        ({
            "origin": "newco",
            "tenure": vars.tenenciaNewCo.customer mapObject ((value, key, index) ->
                {
                    // Ofuscar teléfono (últimos 9 dígitos)
                    (if (key ~= "phone1" and value != null) 
                        (key): value[-9 to -1]
                    else (
                        (if (key ~= "supplyPoints" and value != null)
                            (key): value mapObject(value, key, index) ->
                                (key): value map(supplyPoint, key) ->
                                    supplyPoint mapObject(value, key, index) ->
                                        (if (key ~= "contracts" and value != null)
                                            (key): value mapObject(value, key, index) ->
                                                (key): value map(contract, key) ->
                                                    contract mapObject(value, key, index) ->
                                                        (if (key ~= "contractLines" and value != null)
                                                            (key): value mapObject(value, key, index) ->
                                                                (key): value map(value, key) ->
                                                                    value mapObject(value, key, index) ->
                                                                        // Mapear estados de líneas de contrato
                                                                        if (key ~= "contractLineStatus" and value ~= "01") 
                                                                            (key): "Documentación recibida"
                                                                        else if (key ~= "contractLineStatus" and value ~= "02") 
                                                                            (key): "Activo"
                                                                        else if (key ~= "contractLineStatus" and value ~= "03") 
                                                                            (key): "Alta Activa"
                                                                        else if (key ~= "contractLineStatus" and value ~= "04") 
                                                                            (key): "Anulado"
                                                                        else if (key ~= "contractLineStatus" and value ~= "07") 
                                                                            (key): "Inactivo"
                                                                        else (key): value
                                                        else if (key ~= "electricityCUPS" and value != null) 
                                                            ((key):value) if(!isEmpty(contract..contractLines.items[0] filter($.productCategory == 'Electricidad')))
                                                        else if (key ~= "gasCUPS" and value != null) 
                                                            ((key):value) if(!isEmpty(contract..contractLines.items[0] filter($.productCategory == 'Gas')))
                                                        else (key): value)
                                        else (key): value)
                        else (key): value)
                    ))
                }),
            "partnerships": vars.tenenciaNewCo.partnerships
        }) if (vars.tenenciaNewCo != null and vars.tenenciaNewCo.customer != null)
    ]
}
```

**Transformaciones Aplicadas:**
1. **Ofuscación de teléfono:** Extrae últimos 9 dígitos para NewCo
2. **Mapeo de estados:** Convierte códigos numéricos a texto descriptivo
3. **Filtrado de CUPS:** Solo incluye CUPS si existen productos del tipo correspondiente
4. **Normalización de estructura:** Unifica formato entre Imperial y NewCo

#### Funciones DataWeave Utilizadas

**Ubicación:** `src/main/resources/dataweave/customer/`

**Funciones Importadas:**
- `hasActiveProducts()`: Valida si el cliente tiene productos activos
- `buildResponseGetCustomerInfo()`: Construye respuesta con datos del cliente
- `buildResponseCustomerNotExist()`: Construye respuesta cuando cliente no existe

### Patrones de Diseño

**Patrón 1: Content-Based Routing**
- **Implementación:** Choice router basado en `vars.origin` y `vars.comercializadora`
- **Beneficio:** Enrutamiento dinámico sin duplicación de código

**Patrón 2: API Composition**
- **Implementación:** Agregación de múltiples System APIs en una respuesta
- **Beneficio:** Vista unificada del cliente

**Patrón 3: Error Handling Centralizado**
- **Implementación:** Error handlers específicos por tipo de error
- **Beneficio:** Respuestas de error consistentes

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito (Origen AC + Imperial)

```mermaid
sequenceDiagram
    participant E as Experience API
    participant P as Process API
    participant AC as S-AC (Imperial)
    
    E->>P: GET /customers?origin=AC&comercializadora=imperial
    Note over P: Extraer token del header
    P->>AC: GET /validateToken?token=xxx
    AC-->>P: {result: RESULT_00, userid: "12345"}
    P->>AC: GET /customersRols?userId=12345
    AC-->>P: {items: [{rolename: "US_CC"}]}
    Note over P: IS_US_CC = true
    P->>AC: GET /customersByDoc?docNumber=00000000T
    AC-->>P: {customer: {...}}
    Note over P: Transformar y normalizar
    P-->>E: {result: {codResult: "0000"}, customer: [...]}
```

#### 2. Flujo Normal de Éxito (Origen AT + NewCo)

```mermaid
sequenceDiagram
    participant E as Experience API
    participant P as Process API
    participant VT as validateToken-main
    participant NC as S-PostgreSQL-Salesforce
    
    E->>P: GET /customers?origin=AT&comercializadora=newco
    Note over P: Extraer token del header
    P->>VT: POST /validateToken {issuer: "AT", token: "xxx"}
    VT-->>P: {status: "valid"}
    Note over P: IS_US_CC = true
    P->>NC: GET /client?clientCode=00000000T
    NC-->>P: {result: {codResult: "0000"}, client: {...}}
    P->>NC: GET /contractAccountsNew?clientId=xxx
    NC-->>P: {contracts: [...]}
    Note over P: Validar productos activos
    Note over P: Transformar y normalizar
    P-->>E: {result: {codResult: "0000"}, customer: [...]}
```

#### 3. Flujo con Error de Autenticación

```mermaid
sequenceDiagram
    participant E as Experience API
    participant P as Process API
    participant AC as S-AC
    
    E->>P: GET /customers?origin=AC&comercializadora=imperial
    Note over P: Extraer token del header
    P->>AC: GET /validateToken?token=invalid
    AC-->>P: {result: {CodResult: "ERROR"}}
    Note over P: Validación falla (regex no match)
    Note over P: Error Handler: VALIDATION:MISMATCH
    P-->>E: 400 {result: {codResult: "-0001", msgResult: "ERROR-..."}}
```

#### 4. Flujo con Cliente No Encontrado (NewCo)

```mermaid
sequenceDiagram
    participant E as Experience API
    participant P as Process API
    participant NC as S-PostgreSQL-Salesforce
    
    E->>P: GET /customers?origin=AC&comercializadora=newco
    Note over P: Validación exitosa
    P->>NC: GET /client?clientCode=99999999X
    NC-->>P: {result: {codResult: "0001"}, client: null}
    Note over P: Cliente no existe
    Note over P: buildResponseCustomerNotExist()
    P-->>E: 200 {result: {codResult: "0000"}, customer: null}
```

### Manejo de Errores

**Error Handler Principal:**

```xml
<error-handler>
    <!-- Errores HTTP y técnicos -->
    <on-error-propagate type="HTTP:*, EXPRESSION, STREAM_MAXIMUM_SIZE_EXCEEDED">
        <ee:transform>
            {
                "result": {
                    "codResult": "-0001",
                    "msgResult": "Error técnico"
                }
            }
        </ee:transform>
    </on-error-propagate>
    
    <!-- Error de validación de token -->
    <on-error-propagate type="VALIDATION:MISMATCH">
        <ee:transform>
            {
                "result": {
                    "codResult": "-0001",
                    "msgResult": payload.result.CodResult + '-' + payload.result.MsgResult
                }
            }
        </ee:transform>
    </on-error-propagate>
    
    <!-- Error de parámetros -->
    <on-error-propagate type="VALIDATION:INVALID_BOOLEAN">
        <ee:transform>
            {
                "result": {
                    "codResult": "-0001",
                    "msgResult": "Parámetro no encontrado"
                }
            }
        </ee:transform>
    </on-error-propagate>
</error-handler>
```

**Estrategia de Recuperación:**
- **Errores de conectividad:** Retry automático (configurado en HTTP connector)
- **Errores de autenticación:** Respuesta 401 inmediata
- **Errores de negocio:** Respuesta con código específico
- **Errores técnicos:** Respuesta genérica 400

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta con Token AC + Imperial

#### cURL

```bash
curl -X GET "https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io/api/customers?docNumber=12345678Z&origin=AC&comercializadora=imperial" \
  -H "Authorization: Bearer ca901dae-f60c-406d-9519-8e6ce5c674af" \
  -H "Content-Type: application/json"
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "customer": [
    {
      "origin": "imperial",
      "tenure": {
        "customerId": "0001234567",
        "customerName": "GARCIA LOPEZ, JUAN",
        "documentType": "NIF",
        "documentNumber": "12345678Z",
        "email": "juan.garcia@example.com",
        "phone1": "912345678",
        "phone2": "654321098",
        "address": "CALLE MAYOR 123 PISO 3 PUERTA B",
        "postalCode": "28013",
        "city": "MADRID",
        "province": "MADRID",
        "country": "ESPAÑA",
        "supplyPoints": [
          {
            "supplyPointId": "PS0001234567",
            "address": "CALLE MAYOR 123 PISO 3 PUERTA B",
            "postalCode": "28013",
            "city": "MADRID",
            "province": "MADRID",
            "country": "ESPAÑA",
            "latitude": "40.4168",
            "longitude": "-3.7038",
            "contracts": [
              {
                "contractId": "CT0001234567",
                "contractNumber": "1234567890",
                "contractStatus": "ACTIVO",
                "contractType": "SUMINISTRO",
                "startDate": "2020-01-15",
                "endDate": null,
                "electricityCUPS": "ES0031406759647001JN0F",
                "contractLines": {
                  "items": [
                    {
                      "contractLineId": "CL0001234567",
                      "lineNumber": "001",
                      "productId": "PROD_ELEC_001",
                      "productName": "TARIFA 2.0TD",
                      "productCategory": "Electricidad",
                      "productType": "ELECTRICIDAD_DOMESTICA",
                      "contractLineStatus": "ACTIVO",
                      "startDate": "2020-01-15",
                      "endDate": null,
                      "power": "4.6",
                      "tariff": "2.0TD"
                    }
                  ]
                }
              },
              {
                "contractId": "CT0001234568",
                "contractNumber": "1234567891",
                "contractStatus": "ACTIVO",
                "contractType": "SUMINISTRO",
                "startDate": "2021-03-01",
                "endDate": null,
                "gasCUPS": "ES0021000012345678AB1F",
                "contractLines": {
                  "items": [
                    {
                      "contractLineId": "CL0001234568",
                      "lineNumber": "001",
                      "productId": "PROD_GAS_001",
                      "productName": "GAS NATURAL",
                      "productCategory": "Gas",
                      "productType": "GAS_DOMESTICO",
                      "contractLineStatus": "ACTIVO",
                      "startDate": "2021-03-01",
                      "endDate": null,
                      "tariff": "RL.1"
                    }
                  ]
                }
              }
            ]
          }
        ]
      }
    }
  ]
}
```

**Nota:** Esta estructura es la REAL devuelta por el System API `MUL-COM-CLI-S-AC` (endpoint `/customersByDoc`). El Process API aplica únicamente el filtrado de CUPS condicional:
- `electricityCUPS` solo aparece si `contractLines` contiene productos de categoría "Electricidad"
- `gasCUPS` solo aparece si `contractLines` contiene productos de categoría "Gas"

**Fuente:** Basado en `getCustomersByDocResponse.json` del proyecto MUL-COM-CLI-S-AC

### Ejemplo 2: Consulta con Token Auth0 + NewCo (Con Productos Activos)

#### cURL

```bash
curl -X GET "https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io/api/customers?docNumber=11111111A&origin=AT&comercializadora=newco" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json"
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "customer": [
    {
      "origin": "newco",
      "tenure": {
        "customerId": "a0X7Q000000AbCdEFG",
        "firstName": "María",
        "lastName": "López Fernández",
        "documentType": "01",
        "documentNumber": "11111111A",
        "email": "maria.lopez@example.com",
        "emailStatus": true,
        "phone1": "612345678",
        "preferredLanguage": "es_ES",
        "supplyPoints": [
          {
            "supplyPointId": "a0Y7Q000000XyZabc",
            "address": {
              "street": "Avenida Libertad",
              "number": "45",
              "floor": "3",
              "door": "B",
              "postalCode": "08001",
              "city": "Barcelona",
              "province": "Barcelona",
              "country": "España"
            },
            "contracts": [
              {
                "contractId": "a0Z7Q000000PqRstu",
                "contractAccountId": "8017Q000000MnOpqr",
                "contractStatus": "Activo",
                "startDate": "2024-03-01",
                "endDate": null,
                "gasCUPS": "ES0021000012345678AB1F",
                "contractLines": {
                  "items": [
                    {
                      "lineId": "a1A7Q000000WxYzab",
                      "productId": "01t7Q000000VuWxyz",
                      "productName": "Gas Natural Estándar",
                      "productCategory": "Gas",
                      "contractLineStatus": "Activo",
                      "startDate": "2024-03-01",
                      "endDate": null
                    }
                  ]
                }
              }
            ]
          }
        ]
      },
      "partnerships": [
        {
          "partnershipId": "a2B7Q000000CdEfgh",
          "partnerName": "Alianza Energía Verde",
          "partnershipType": "Energía Renovable",
          "status": "Activa",
          "startDate": "2024-01-15"
        }
      ]
    }
  ]
}
```

**Nota:** Esta respuesta es construida por la función `buildResponseGetCustomerInfo()` cuando `hasActiveProducts()` retorna `true`. Los campos están basados en la estructura real definida en `dataweave/customer/Mappings.dwl`.

### Ejemplo 3: Consulta NewCo Sin Productos Activos

#### cURL

```bash
curl -X GET "https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io/api/customers?docNumber=22222222B&origin=AT&comercializadora=newco" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json"
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "customer": [
    {
      "origin": "newco",
      "tenure": null,
      "partnerships": null
    }
  ]
}
```

**Nota:** Esta respuesta es construida por la función `buildResponseCustomerNotExist()` cuando:
- El cliente existe en Salesforce PERO `hasActiveProducts()` retorna `false`
- O cuando el cliente no existe en absoluto (codResult != '0000')

**Lógica de Validación (Utils.dwl):**
```dataweave
fun hasActiveProducts(productlist) = (
    !isEmpty(productlist.contractAccounts.items) and
    productlist.contractAccounts.items[0].contracts != null and
    !isEmpty(productlist.contractAccounts.items[0].contracts.items)
)
```

### Ejemplo 4: Consulta Ambas Comercializadoras

#### JavaScript (Fetch API)

```javascript
// Función para consultar ambas comercializadoras
async function getCustomerAllComercializadoras(docNumber, token, origin) {
  const baseUrl = 'https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io/api/customers';
  
  // Consultar Imperial
  const imperialResponse = await fetch(
    `${baseUrl}?docNumber=${docNumber}&origin=${origin}&comercializadora=imperial`,
    {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    }
  );
  const imperialData = await imperialResponse.json();
  
  // Consultar NewCo
  const newcoResponse = await fetch(
    `${baseUrl}?docNumber=${docNumber}&origin=${origin}&comercializadora=newco`,
    {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    }
  );
  const newcoData = await newcoResponse.json();
  
  // Combinar resultados
  return {
    imperial: imperialData.customer.find(c => c.origin === 'imperial'),
    newco: newcoData.customer.find(c => c.origin === 'newco')
  };
}

// Uso
const customerData = await getCustomerAllComercializadoras(
  '00000000T',
  'ca901dae-f60c-406d-9519-8e6ce5c674af',
  'AC'
);

console.log('Imperial:', customerData.imperial);
console.log('NewCo:', customerData.newco);
```

### Ejemplo 4: Manejo de Errores

#### Python

```python
import requests

def get_customer_with_error_handling(doc_number, token, origin, comercializadora):
    url = 'https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io/api/customers'
    
    headers = {
        'Authorization': f'Bearer {token}',
        'Content-Type': 'application/json'
    }
    
    params = {
        'docNumber': doc_number,
        'origin': origin,
        'comercializadora': comercializadora
    }
    
    try:
        response = requests.get(url, headers=headers, params=params, timeout=30)
        
        if response.status_code == 200:
            data = response.json()
            if data['result']['codResult'] == '0000':
                return {'success': True, 'data': data}
            else:
                return {
                    'success': False,
                    'error': 'Business error',
                    'code': data['result']['codResult'],
                    'message': data['result']['msgResult']
                }
        
        elif response.status_code == 401:
            return {
                'success': False,
                'error': 'Authentication error',
                'message': 'Token inválido o expirado'
            }
        
        elif response.status_code == 400:
            error_data = response.json()
            return {
                'success': False,
                'error': 'Bad request',
                'code': error_data['result']['codResult'],
                'message': error_data['result']['msgResult']
            }
        
        else:
            return {
                'success': False,
                'error': 'HTTP error',
                'status_code': response.status_code
            }
    
    except requests.exceptions.Timeout:
        return {
            'success': False,
            'error': 'Timeout',
            'message': 'La solicitud excedió el tiempo de espera'
        }
    
    except requests.exceptions.RequestException as e:
        return {
            'success': False,
            'error': 'Connection error',
            'message': str(e)
        }

# Uso
result = get_customer_with_error_handling(
    doc_number='00000000T',
    token='ca901dae-f60c-406d-9519-8e6ce5c674af',
    origin='AC',
    comercializadora='imperial'
)

if result['success']:
    print('Cliente encontrado:', result['data'])
else:
    print(f"Error: {result['error']} - {result.get('message', 'Sin mensaje')}")
```

### Ejemplo 5: Error de Token Inválido

#### Request

```bash
curl -X GET "https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io/api/customers?docNumber=00000000T&origin=AC&comercializadora=imperial" \
  -H "Authorization: Bearer token_invalido_o_expirado" \
  -H "Content-Type: application/json"
```

#### Response (401 Unauthorized)

```json
{
  "result": {
    "codResult": "AUTH-0001",
    "msgResult": "El token de autenticación recibido es incorrecto o ha caducado",
    "detail": "El token de autenticación recibido es incorrecto o ha caducado"
  }
}
```

### Ejemplo 6: Error de Parámetro Faltante

#### Request

```bash
curl -X GET "https://ntgy-fuv-p-customers-pro.us-e2.cloudhub.io/api/customers?docNumber=00000000T&origin=AC" \
  -H "Authorization: Bearer ca901dae-f60c-406d-9519-8e6ce5c674af" \
  -H "Content-Type: application/json"
```

#### Response (400 Bad Request)

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Parámetro no encontrado"
  }
}
```
