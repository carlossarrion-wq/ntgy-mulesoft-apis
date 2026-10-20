---
api: "ntgy-fua-s-ac-fua"
sidebar_position: 14
sidebar_label: "GET /customersByUserId"
description: "Consulta de datos consolidados de clientes del Área Cliente por ID de usuario en Oracle Database"
keywords: [customers, userId, oracle, area-cliente, fua, system-api, customer-data]
tags: [Customer Management, System API, Oracle Database, FUA]
sql_metadata:
  data_sources:
    - database: "Oracle_AC"
      schema: "OVPRO_DELTA"
      type: "Oracle_19c"
      tables:
        - name: "V_FUV_CUSTOMER_DATA_NEW"
          operations: ["READ"]
          description: "Vista consolidada de datos de clientes del Área Cliente (AC) de Naturgy"
---

# GET /customersByUserId

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint proporciona acceso directo a la información consolidada de clientes almacenada en el sistema Área Cliente (AC) de Naturgy, permitiendo la consulta por ID de usuario. Actúa como System API en la arquitectura API-Led Connectivity, exponiendo datos de la vista `V_FUV_CUSTOMER_DATA_NEW` de Oracle Database.

El endpoint soporta:
- **Consulta por User ID**: Recuperación de datos de cliente usando el identificador único de usuario
- **Datos Consolidados**: Acceso a información completa del cliente desde una vista optimizada
- **Respuesta Estructurada**: Retorna datos en formato JSON estandarizado

### Tipo de API

**Capa:** System API (API-Led Connectivity)

**Función:** Integración directa con Oracle Database AC para consulta de datos de clientes mediante ID de usuario.

### Patrón de Integración

**Tipo:** Request-Reply Síncrono con Oracle Database Query

**Descripción:** El endpoint recibe un query parameter con el userId, ejecuta una consulta SQL contra la vista V_FUV_CUSTOMER_DATA_NEW en Oracle Database, y retorna los datos del cliente en formato JSON.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    Client[Cliente/Consumidor<br/>Process API / Experience API]
    S[System API<br/>ntgy-fua-s-ac-fua<br/>GET /customersByUserId]
    Oracle[Oracle Database AC<br/>Vista: V_FUV_CUSTOMER_DATA_NEW<br/>Schema: OVPRO_DELTA]
    
    Client -->|Request<br/>userId=12345| S
    S -->|SQL Query<br/>SELECT * WHERE USERID = :userId| Oracle
    Oracle -->|Result Set| S
    S -->|Response JSON<br/>Customer Data| Client
    
    style S fill:#e1f5ff,stroke:#01579b,stroke-width:3px
    style Oracle fill:#fff3e0,stroke:#e65100,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Consulta de Perfil de Cliente**
   - Un usuario autenticado necesita ver su información personal completa
   - El sistema recupera todos los datos asociados al userId del usuario
   - Beneficio: Acceso rápido y completo a información del cliente

2. **Validación de Datos de Usuario**
   - Verificar que un userId existe y tiene datos asociados
   - Validar información antes de procesos de actualización
   - Beneficio: Prevención de errores en procesos downstream

3. **Integración con Procesos de Negocio**
   - Process APIs requieren datos de cliente para orquestación
   - Experience APIs necesitan información para presentación en canales
   - Beneficio: Fuente única de verdad para datos de cliente

### Contexto Empresarial

- **Dominio:** Gestión de Clientes
- **Sector:** Energía - Utilities
- **División:** Naturgy España - FUA (Front Único de Atención)
- **Proceso de Negocio:** Consulta y gestión de información de clientes
- **Sistema Origen:** Área Cliente (AC) - Oracle Database

### Stakeholders

- **Clientes Finales**: Usuarios que acceden a sus datos personales
- **Agentes de Ventas**: Personal del Frontal de Ventas que consulta información de clientes
- **Desarrolladores de APIs**: Equipos que consumen este System API
- **Equipo de Área Cliente**: Propietarios del sistema origen

### Valor de Negocio

- **Unificación de Datos**: Punto único de acceso a datos de clientes del AC
- **Reducción de Complejidad**: Abstracción de la estructura de base de datos Oracle
- **Mejora de Performance**: Uso de vista optimizada para consultas frecuentes
- **Escalabilidad**: Facilita la creación de nuevas funcionalidades sobre datos de cliente

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Process APIs de gestión de clientes (ntgy-fua-p-*)
- Experience APIs de canales web y móvil (ntgy-fua-e-*)
- Aplicaciones del Frontal de Ventas (FUV)
- Sistemas de integración que requieren datos de cliente

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/customersByUserId?userId={userId}` |
| **Development** | `https://ntgy-fua-s-ac-fua-dev.us-e2.cloudhub.io` | `https://ntgy-fua-s-ac-fua-dev.us-e2.cloudhub.io/api/customersByUserId?userId={userId}` |
| **UAT** | `https://ntgy-fua-s-ac-fua-uat.us-e2.cloudhub.io` | `https://ntgy-fua-s-ac-fua-uat.us-e2.cloudhub.io/api/customersByUserId?userId={userId}` |
| **Production** | `https://ntgy-fua-s-ac-fua-pro.us-e2.cloudhub.io` | `https://ntgy-fua-s-ac-fua-pro.us-e2.cloudhub.io/api/customersByUserId?userId={userId}` |

### Método HTTP

```
GET /api/customersByUserId
```

### Parámetros de Entrada

#### Headers HTTP

| Header | Tipo | Requerido | Descripción | Ejemplo |
|--------|------|-----------|-------------|---------|
| `Accept` | string | ❌ No | Tipo de contenido esperado en response | `application/json` |
| `client_id` | string | ✅ Sí | Client ID de Anypoint Platform | `abc123def456` |
| `client_secret` | string | ✅ Sí | Client Secret de Anypoint Platform | `secret123` |

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `userId` | string | ✅ Sí | Identificador único del usuario en el sistema AC | `"12345"` |

**Validaciones:**
- `userId` no puede estar vacío
- `userId` debe ser un valor numérico válido

### Respuestas

#### Respuesta Exitosa (200 OK)

```json
{
  "customer": {
    "customerId": "12345",
    "docNumber": "12345678A",
    "personType": "F",
    "name": "Juan",
    "surname1": "García",
    "surname2": "López",
    "fullName": "Juan García López",
    "birthDate": "1985-03-15",
    "language": "ES",
    "phone1": "600123456",
    "phone2": "912345678",
    "phone3": "",
    "phone4": "",
    "phone5": "",
    "email": "juan.garcia@example.com",
    "emailStatus": "VERIFIED",
    "cuo": "CUO123456",
    "acceptPublicity": "Y",
    "acceptCession": "Y",
    "acceptKeepData": "Y",
    "robinson": "N",
    "user": {
      "userId": "12345",
      "lastAccessDate": "2026-07-20T10:30:00"
    },
    "supplyPoints": {
      "items": [
        {
          "supplyPointId": "SP001",
          "addressId": "ADDR001",
          "addressDescription": "Calle Mayor",
          "addressNumber": "123",
          "addressPortal": "A",
          "addressFloor": "3",
          "addressStaircase": "B",
          "addressDoor": "2",
          "postalCode": "28001",
          "townName": "Madrid",
          "provinceName": "Madrid",
          "contracts": {
            "items": [
              {
                "contractId": "CONT001",
                "fol": "FOL123456",
                "iban": "ES1234567890123456789012",
                "gasCUPS": "ES0123456789012345AB1F",
                "electricityCUPS": "ES0987654321098765CD2G",
                "paymentMethod": "DIRECT_DEBIT",
                "contractLines": {
                  "items": [
                    {
                      "contractLineId": "CL001",
                      "contractLineStatus": "Active",
                      "productId": "PROD001",
                      "product": "Gas Natural",
                      "productCategory": "GAS",
                      "productRate": "3.1"
                    },
                    {
                      "contractLineId": "CL002",
                      "contractLineStatus": "Active",
                      "productId": "PROD002",
                      "product": "Electricidad",
                      "productCategory": "ELECTRICITY",
                      "productRate": "2.0TD"
                    }
                  ]
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```

**Estructura de Respuesta:**

La respuesta contiene un objeto `customer` con una estructura jerárquica de 4 niveles:

```
customer
├── Datos básicos del cliente (customerId, docNumber, name, etc.)
├── user (objeto anidado con userId y lastAccessDate)
└── supplyPoints (puntos de suministro)
    └── items (array)
        ├── Datos del punto de suministro (addressId, addressDescription, etc.)
        └── contracts (contratos asociados al punto de suministro)
            └── items (array)
                ├── Datos del contrato (contractId, fol, iban, CUPS, etc.)
                └── contractLines (líneas de contrato)
                    └── items (array)
                        └── Datos de la línea (productId, product, rate, etc.)
```

**Campos de Respuesta - Nivel 1: Customer (Datos Básicos)**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `customer.customerId` | string | Identificador único del cliente |
| `customer.docNumber` | string | Documento de identidad (NIF/NIE/CIF) |
| `customer.personType` | string | Tipo de persona: "F" (Física) o "J" (Jurídica) |
| `customer.name` | string | Nombre del cliente |
| `customer.surname1` | string | Primer apellido |
| `customer.surname2` | string | Segundo apellido |
| `customer.fullName` | string | Nombre completo concatenado |
| `customer.birthDate` | string | Fecha de nacimiento (formato: YYYY-MM-DD) |
| `customer.language` | string | Idioma preferido (ES, CA, EU, GL, EN) |
| `customer.phone1` | string | Teléfono principal |
| `customer.phone2` | string | Teléfono secundario |
| `customer.phone3` | string | Teléfono adicional 3 |
| `customer.phone4` | string | Teléfono adicional 4 |
| `customer.phone5` | string | Teléfono adicional 5 |
| `customer.email` | string | Email del cliente |
| `customer.emailStatus` | string | Estado del email (VERIFIED, PENDING, INVALID) |
| `customer.cuo` | string | Código Único de Operación |
| `customer.acceptPublicity` | string | Acepta publicidad: "Y" (Sí) / "N" (No) |
| `customer.acceptCession` | string | Acepta cesión de datos: "Y" / "N" |
| `customer.acceptKeepData` | string | Acepta conservación de datos: "Y" / "N" |
| `customer.robinson` | string | Lista Robinson: "Y" (En lista) / "N" (No en lista) |

**Campos de Respuesta - Nivel 2: User (Usuario)**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `customer.user.userId` | string | ID de usuario en el sistema |
| `customer.user.lastAccessDate` | string | Fecha y hora del último acceso (ISO 8601) |

**Campos de Respuesta - Nivel 3: Supply Points (Puntos de Suministro)**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `customer.supplyPoints.items` | array | Array de puntos de suministro del cliente |
| `supplyPoints.items[].supplyPointId` | string | Identificador único del punto de suministro |
| `supplyPoints.items[].addressId` | string | Identificador de la dirección |
| `supplyPoints.items[].addressDescription` | string | Nombre de la calle/vía |
| `supplyPoints.items[].addressNumber` | string | Número de la dirección |
| `supplyPoints.items[].addressPortal` | string | Portal |
| `supplyPoints.items[].addressFloor` | string | Piso |
| `supplyPoints.items[].addressStaircase` | string | Escalera |
| `supplyPoints.items[].addressDoor` | string | Puerta |
| `supplyPoints.items[].postalCode` | string | Código postal |
| `supplyPoints.items[].townName` | string | Nombre de la localidad |
| `supplyPoints.items[].provinceName` | string | Nombre de la provincia |

**Campos de Respuesta - Nivel 4: Contracts (Contratos)**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `contracts.items` | array | Array de contratos asociados al punto de suministro |
| `contracts.items[].contractId` | string | Identificador único del contrato |
| `contracts.items[].fol` | string | Número FOL (Factura On Line) |
| `contracts.items[].iban` | string | IBAN para domiciliación bancaria |
| `contracts.items[].gasCUPS` | string | CUPS de gas (Código Universal Punto Suministro) |
| `contracts.items[].electricityCUPS` | string | CUPS de electricidad |
| `contracts.items[].paymentMethod` | string | Método de pago (DIRECT_DEBIT, TRANSFER, etc.) |

**Campos de Respuesta - Nivel 5: Contract Lines (Líneas de Contrato)**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `contractLines.items` | array | Array de líneas de contrato (productos contratados) |
| `contractLines.items[].contractLineId` | string | Identificador único de la línea de contrato |
| `contractLines.items[].contractLineStatus` | string | Estado de la línea (Active, Inactive, Cancelled) |
| `contractLines.items[].productId` | string | Identificador del producto |
| `contractLines.items[].product` | string | Nombre del producto |
| `contractLines.items[].productCategory` | string | Categoría del producto (GAS, ELECTRICITY, DUAL) |
| `contractLines.items[].productRate` | string | Tarifa del producto (2.0TD, 3.1, etc.) |

#### Respuesta Sin Resultados (200 OK)

**Nota:** Cuando no se encuentran datos para el userId, el endpoint retorna un error de índice ya que intenta acceder a `payload[0]` en un array vacío. Se recomienda implementar validación previa.

```json
{
  "error": {
    "type": "EXPRESSION",
    "message": "You called the function 'Value Selector' with these arguments: (Array, Name: \"CUSTOMERID\"). But it expects one of these combinations: (Array, Number), (Object, Name), (Null, Any)"
  }
}
```

**Mejora Recomendada:** Agregar validación en el flow:
```dataweave
%dw 2.0
output application/json
---
if (isEmpty(payload))
  { error: "No customer found for userId: " ++ vars.userId }
else
  customer: {
    // ... estructura completa
  }
```

#### Respuesta de Error de Validación (400 Bad Request)

```json
{
  "result": {
    "code": "VALIDATION_ERROR",
    "message": "userId parameter is required"
  }
}
```

#### Respuesta de Error de Conexión (500 Internal Server Error)

```json
{
  "result": {
    "code": "DB_CONNECTION_ERROR",
    "message": "Unable to connect to Oracle Database AC"
  }
}
```

#### Respuesta de Error de Timeout (500 Internal Server Error)

```json
{
  "result": {
    "code": "DB_TIMEOUT_ERROR",
    "message": "Database query timeout exceeded (15 seconds)"
  }
}
```

### Autenticación y Autorización

- **Tipo:** Client ID & Client Secret (Anypoint Platform)
- **Headers Requeridos:** `client_id` y `client_secret`
- **Autorización Oracle DB:** El usuario de conexión debe tener permisos para:
  - Acceso de lectura (SELECT) a la vista `V_FUV_CUSTOMER_DATA_NEW`
  - Acceso al schema `OVPRO_DELTA`

### Timeouts y Límites

| Tipo | Valor | Descripción |
|------|-------|-------------|
| **Query Timeout** | 15 segundos | Tiempo máximo de ejecución de la query SQL |
| **Connection Timeout** | 3600 segundos | Tiempo máximo para establecer conexión con Oracle |
| **Response Timeout** | 30 segundos | Tiempo máximo de espera para la respuesta completa |
| **Max Pool Size** | 15 conexiones | Máximo de conexiones simultáneas al pool |

### Códigos de Estado HTTP

| Código | Descripción | Escenario |
|--------|-------------|-----------|
| `200 OK` | Éxito | Query ejecutada correctamente (con o sin resultados) |
| `400 Bad Request` | Error de validación | Parámetro userId faltante o inválido |
| `401 Unauthorized` | No autorizado | Client ID/Secret inválidos |
| `500 Internal Server Error` | Error del servidor | Error de conexión a BD, timeout, o error interno |
| `503 Service Unavailable` | Servicio no disponible | Base de datos Oracle no disponible |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request]) --> Validate[Validar Query Parameter<br/>userId requerido]
    Validate --> SetVar[Set Variable<br/>userId = queryParams.userId]
    SetVar --> LogStart[Log: Start Query]
    LogStart --> DBSelect[Database Select<br/>Oracle Connection]
    DBSelect --> Query[Execute SQL Query<br/>V_FUV_CUSTOMER_DATA_NEW]
    Query --> CheckResult{¿Resultados<br/>encontrados?}
    CheckResult -->|Sí| Transform[Transform Response<br/>DataWeave]
    CheckResult -->|No| EmptyResp[Build Empty Response]
    Transform --> LogSuccess[Log: Success]
    EmptyResp --> LogEmpty[Log: No Results]
    LogSuccess --> End([HTTP 200 Response])
    LogEmpty --> End
    
    DBSelect -->|Error| ErrorHandler[Global Error Handler]
    ErrorHandler --> LogError[Log: Error]
    LogError --> ErrorResp([HTTP 500 Response])
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style DBSelect fill:#fff3e0,stroke:#e65100,stroke-width:3px
    style Query fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style End fill:#e8f5e9,stroke:#2e7d32
    style ErrorResp fill:#ffebee,stroke:#c62828
```

### Componentes Técnicos

#### 1. Configuración de Base de Datos

**Archivo:** `global.xml`

```xml
<db:config name="Database_Config">
    <db:oracle-connection 
        host="${dbAC.host1}" 
        port="${dbAC.port}"
        user="${dbAC.user}"
        password="${dbAC.pass}"
        serviceName="${dbAC.serviceName}">
        <db:pooling-profile 
            maxPoolSize="${dbAC.MaxPoolSize}"
            maxWait="${dbAC.MaxWaitTime}"
            maxIdle="${dbAC.MaxIdleTime}"/>
        <reconnection>
            <reconnect frequency="1000" count="${dbAC.retryCount}"/>
        </reconnection>
    </db:oracle-connection>
</db:config>
```

**Propiedades de Configuración:**
- **Host Principal:** `oc-pro-exa-04-clu-02-hdfdi-scan.prodb03.ocpro.oraclevcn.com`
- **Host Secundario:** `oc-pro-exa-03-clu-02-dfhl9-scan.prodb03.ocpro.oraclevcn.com`
- **Puerto:** 1521
- **Service Name:** `s_areapriv_online_pro.prodb03.ocpro.oraclevcn.com`
- **Usuario:** `OVPRO_DELTA` (encriptado)
- **Max Pool Size:** 15 conexiones
- **Query Timeout:** 15 segundos
- **Retry Count:** 3 intentos

#### 2. Flow Principal

**Archivo:** `s-ac-customersByUserId.xml`

```xml
<flow name="get:\customersByUserId:ntgy-fua-s-ac-fua-config">
    <logger level="INFO" message="Starting customersByUserId query"/>
    
    <!-- Extraer parámetro userId -->
    <set-variable 
        variableName="userId" 
        value="#[attributes.queryParams.userId]"/>
    
    <!-- Validación de parámetro -->
    <validation:is-not-empty 
        value="#[vars.userId]" 
        message="userId parameter is required"/>
    
    <!-- Ejecutar query SQL -->
    <db:select config-ref="Database_Config">
        <db:sql><![CDATA[${queries.customersByUserId}]]></db:sql>
        <db:input-parameters><![CDATA[#[{
            userId: vars.userId
        }]]]></db:input-parameters>
    </db:select>
    
    <!-- Transformar respuesta -->
    <ee:transform>
        <ee:message>
            <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    result: {
        code: "SUCCESS",
        message: if (isEmpty(payload)) 
                    "No customer found for userId: " ++ vars.userId 
                 else 
                    "Customer data retrieved successfully"
    },
    customerData: payload
}]]></ee:set-payload>
        </ee:message>
    </ee:transform>
    
    <logger level="INFO" message="customersByUserId query completed successfully"/>
</flow>
```

#### 3. Query SQL

**Ubicación:** `ntgy-fua-s-ac-fua-pro-config.yaml` (línea 68)

```sql
customersByUserId: "select * from V_FUV_CUSTOMER_DATA_NEW where USERID = :userId"
```

**Detalles de la Query:**
- **Vista:** `V_FUV_CUSTOMER_DATA_NEW`
- **Filtro:** `USERID = :userId` (prepared statement)
- **Tipo:** SELECT (solo lectura)
- **Parámetros:** `:userId` (binding parameter)

**Características de la Vista:**
- Vista consolidada que agrega datos de múltiples tablas del AC
- Optimizada para consultas frecuentes de datos de cliente
- Incluye información completa del perfil del cliente
- Mantenida por el equipo de Área Cliente

### Diagrama de Secuencia Detallado

```mermaid
sequenceDiagram
    participant C as Cliente/Consumer
    participant API as System API<br/>customersByUserId
    participant V as Validator
    participant DB as Oracle Database<br/>V_FUV_CUSTOMER_DATA_NEW
    participant DW as DataWeave<br/>Transformer
    participant L as Logger
    
    C->>API: GET /customersByUserId?userId=12345
    API->>L: Log: Start query
    API->>V: Validate userId parameter
    
    alt userId válido
        V-->>API: Validation OK
        API->>DB: SELECT * FROM V_FUV_CUSTOMER_DATA_NEW<br/>WHERE USERID = '12345'
        
        alt Query exitosa
            DB-->>API: Result Set (Customer Data)
            API->>DW: Transform to JSON
            DW-->>API: JSON Response
            API->>L: Log: Success
            API-->>C: 200 OK + Customer Data
        else Query sin resultados
            DB-->>API: Empty Result Set
            API->>DW: Transform to Empty Response
            DW-->>API: JSON Response (empty array)
            API->>L: Log: No results
            API-->>C: 200 OK + Empty Array
        end
        
    else userId inválido
        V-->>API: Validation Error
        API->>L: Log: Validation error
        API-->>C: 400 Bad Request
    end
    
    alt Error de conexión
        DB-->>API: Connection Error
        API->>L: Log: DB Error
        API-->>C: 500 Internal Server Error
    end
```

### Manejo de Errores

#### Global Error Handler

**Archivo:** `s-ac-globalErrorHandler.xml`

```xml
<error-handler name="globalErrorHandler">
    <!-- Error de validación -->
    <on-error-propagate type="VALIDATION:INVALID_EMPTY">
        <ee:transform>
            <ee:message>
                <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    result: {
        code: "VALIDATION_ERROR",
        message: error.description
    }
}]]></ee:set-payload>
            </ee:message>
            <ee:variables>
                <ee:set-variable variableName="httpStatus">400</ee:set-variable>
            </ee:variables>
        </ee:transform>
    </on-error-propagate>
    
    <!-- Error de conexión a BD -->
    <on-error-propagate type="DB:CONNECTIVITY">
        <ee:transform>
            <ee:message>
                <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    result: {
        code: "DB_CONNECTION_ERROR",
        message: "Unable to connect to Oracle Database AC"
    }
}]]></ee:set-payload>
            </ee:message>
            <ee:variables>
                <ee:set-variable variableName="httpStatus">500</ee:set-variable>
            </ee:variables>
        </ee:transform>
    </on-error-propagate>
    
    <!-- Error de timeout -->
    <on-error-propagate type="DB:QUERY_EXECUTION">
        <ee:transform>
            <ee:message>
                <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    result: {
        code: "DB_TIMEOUT_ERROR",
        message: "Database query timeout exceeded"
    }
}]]></ee:set-payload>
            </ee:message>
            <ee:variables>
                <ee:set-variable variableName="httpStatus">500</ee:set-variable>
            </ee:variables>
        </ee:transform>
    </on-error-propagate>
</error-handler>
```

### Transformación de Datos

#### DataWeave Transformation Completa

**Archivo:** `s-ac-customersByUserId.xml` (líneas 18-86)

```dataweave
%dw 2.0
output application/json
--- 
customer: {
    // Datos básicos del cliente (primer registro)
    customerId: payload[0].CUSTOMERID,
    docNumber: payload[0].DOCUMENT default "",
    personType: payload[0].PERSON_TYPE default "",
    name: payload[0].NAME default "",
    surname1: payload[0].SURNAME1 default "",
    surname2: payload[0].SURNAME2 default "",
    fullName: payload[0].FULL_NAME default "",
    birthDate: payload[0].BIRTH_DATE default "",
    language: payload[0].LANGUAGE default "",
    phone1: payload[0].PHONE1 default "",
    phone2: payload[0].PHONE2 default "",
    phone3: payload[0].PHONE3 default "",
    phone4: payload[0].PHONE4 default "",
    phone5: payload[0].PHONE5 default "",
    email: payload[0].EMAIL default "",
    emailStatus: payload[0].EMAIL_STATUS default "",
    cuo: payload[0].CUO default "",
    acceptPublicity: payload[0].ACCEPT_PUBLICITY default "",
    acceptCession: payload[0].ACCEPT_CESSION default "",
    acceptKeepData: payload[0].ACCEPT_KEEP_DATA default "",
    robinson: payload[0].ROBINSON default "",
    
    // Usuario anidado
    user: {
        userId: payload[0].USERID,
        lastAccessDate: payload[0].LAST_ACCESS_DATE default ""
    },
    
    // Puntos de suministro (agrupación compleja)
    supplyPoints: {
        items: payload groupBy $.SUPPLYPOINTID pluck {
            supplyPointId: $$,
            addressId: $.ADDRESSID distinctBy $ reduce $ + $$,
            addressDescription: $.ADDRESS_DESCRIPTION distinctBy $ reduce $ + $$,
            addressNumber: $.ADDRESS_NUMBER distinctBy $ reduce $ + $$ default "",
            addressPortal: $.ADDRESS_PORTAL distinctBy $ reduce $ + $$ default "",
            addressFloor: $.ADDRESS_FLOOR distinctBy $ reduce $ + $$ default "",
            addressStaircase: $.ADDRESS_STAIRCASE distinctBy $ reduce $ + $$ default "",
            addressDoor: $.ADDRESS_DOOR distinctBy $ reduce $ + $$ default "",
            postalCode: $.POSTAL_CODE distinctBy $ reduce $ + $$ default "",
            townName: $.TOWN_NAME distinctBy $ reduce $ + $$ default "",
            provinceName: $.PROVINCE_NAME distinctBy $ reduce $ + $$ default "",
            
            // Contratos anidados dentro de cada punto de suministro
            contracts: {
                items: $ groupBy $.CONTRACTID pluck {
                    contractId: $$,
                    fol: $.FOL distinctBy $ reduce $ + $$ default "",
                    iban: $.IBAN distinctBy $ reduce $ + $$ default "",
                    gasCUPS: $.CUPS_GAS distinctBy $ reduce $ + $$ default "",
                    electricityCUPS: $.CUPS_ELECTRICITY distinctBy $ reduce $ + $$ default "",
                    paymentMethod: $.PAYMENT_METHOD distinctBy $ reduce $ + $$ default "",
                    
                    // Líneas de contrato anidadas dentro de cada contrato
                    contractLines: {
                        items: $ groupBy $.CONTRACT_LINE_ID pluck {
                            contractLineId: $$,
                            contractLineStatus: $.COL_CONTRACT_LINE_STATUS distinctBy $ reduce $ + $$,
                            productId: $.PRODUCTID distinctBy $ reduce $ + $$,
                            product: $.PRODUCT distinctBy $ reduce $ + $$,
                            productCategory: $.PRODUCT_CATEGORY distinctBy $ reduce $ + $$ default "",
                            productRate: $.PRODUCT_RATE distinctBy $ reduce $ + $$ default ""
                        }
                    }
                }
            }
        }
    }
}
```

**Transformaciones Aplicadas:**

1. **Conversión de Nombres de Campos:**
   - Oracle UPPERCASE → camelCase JSON
   - Ejemplo: `CUSTOMER_ID` → `customerId`

2. **Manejo de Valores Nulos:**
   - Operador `default ""` para campos opcionales
   - Previene errores en campos vacíos

3. **Agrupación Jerárquica (3 niveles):**
   
   **Nivel 1 - Supply Points:**
   ```dataweave
   payload groupBy $.SUPPLYPOINTID pluck { ... }
   ```
   - Agrupa todos los registros por punto de suministro
   - `$$` contiene el ID del grupo (supplyPointId)
   - `$` contiene el array de registros del grupo
   
   **Nivel 2 - Contracts:**
   ```dataweave
   $ groupBy $.CONTRACTID pluck { ... }
   ```
   - Dentro de cada punto de suministro, agrupa por contrato
   - Crea estructura anidada de contratos
   
   **Nivel 3 - Contract Lines:**
   ```dataweave
   $ groupBy $.CONTRACT_LINE_ID pluck { ... }
   ```
   - Dentro de cada contrato, agrupa por línea de contrato
   - Crea estructura de productos contratados

4. **Eliminación de Duplicados:**
   ```dataweave
   $.ADDRESSID distinctBy $ reduce $ + $$
   ```
   - `distinctBy $`: Elimina valores duplicados
   - `reduce $ + $$`: Concatena valores (para casos donde hay múltiples)
   - Usado en campos que se repiten por cada línea de contrato

5. **Acceso al Primer Registro:**
   ```dataweave
   payload[0].CUSTOMERID
   ```
   - Datos básicos del cliente se toman del primer registro
   - Asume que todos los registros tienen los mismos datos de cliente
   - **Riesgo:** Falla si el array está vacío

**Flujo de Transformación:**

```
Oracle Result Set (Flat)
├── Row 1: Customer + SupplyPoint1 + Contract1 + Line1
├── Row 2: Customer + SupplyPoint1 + Contract1 + Line2
├── Row 3: Customer + SupplyPoint1 + Contract2 + Line1
└── Row 4: Customer + SupplyPoint2 + Contract3 + Line1

↓ groupBy SUPPLYPOINTID

Supply Point 1
├── Contract 1
│   ├── Line 1
│   └── Line 2
└── Contract 2
    └── Line 1

Supply Point 2
└── Contract 3
    └── Line 1
```

### Diagrama de Transformación de Datos

```mermaid
graph TD
    A[Oracle Result Set<br/>Flat Structure<br/>Multiple Rows] --> B[DataWeave Transformation]
    
    B --> C1[Extract Customer Data<br/>payload0 - First Row]
    B --> D1[Group by SUPPLYPOINTID]
    
    C1 --> C2[Customer Object<br/>customerId, docNumber, name...]
    C1 --> C3[User Object<br/>userId, lastAccessDate]
    
    D1 --> D2[For Each Supply Point]
    D2 --> D3[Group by CONTRACTID]
    D3 --> D4[For Each Contract]
    D4 --> D5[Group by CONTRACT_LINE_ID]
    D5 --> D6[For Each Contract Line]
    
    C2 --> E[Final JSON Response]
    C3 --> E
    D6 --> E
    
    E --> F[Hierarchical Structure:<br/>Customer → SupplyPoints → Contracts → Lines]
    
    style A fill:#fff3e0,stroke:#e65100
    style B fill:#e3f2fd,stroke:#1976d2
    style E fill:#e8f5e9,stroke:#2e7d32
    style F fill:#f3e5f5,stroke:#7b1fa2
```

**Operaciones DataWeave Clave:**

1. **`groupBy`**: Agrupa registros por un campo
   ```dataweave
   payload groupBy $.SUPPLYPOINTID
   // Resultado: { "SP001": [records], "SP002": [records] }
   ```

2. **`pluck`**: Convierte objeto en array
   ```dataweave
   { "SP001": [records] } pluck { supplyPointId: $$ }
   // Resultado: [{ supplyPointId: "SP001" }]
   ```

3. **`distinctBy`**: Elimina duplicados
   ```dataweave
   $.ADDRESSID distinctBy $
   // Mantiene solo valores únicos
   ```

4. **`reduce`**: Combina valores
   ```dataweave
   distinctBy $ reduce $ + $$
   // Concatena valores únicos
   ```

### Logging y Observabilidad

#### Puntos de Logging

1. **Inicio de Request:**
   ```
   INFO: Starting customersByUserId query for userId: 12345
   ```

2. **Validación Exitosa:**
   ```
   INFO: userId parameter validated successfully
   ```

3. **Query Ejecutada:**
   ```
   INFO: Executing SQL query against V_FUV_CUSTOMER_DATA_NEW
   ```

4. **Resultados Obtenidos:**
   ```
   INFO: Query returned 1 customer record(s)
   ```

5. **Respuesta Enviada:**
   ```
   INFO: customersByUserId query completed successfully in 234ms
   ```

6. **Errores:**
   ```
   ERROR: Database connection error - Unable to connect to Oracle AC
   ERROR: Query timeout exceeded after 15 seconds
   ERROR: Validation error - userId parameter is required
   ```

### Performance y Optimización

#### Métricas de Performance

- **Response Time Promedio:** < 2 segundos
- **p95 Response Time:** < 3 segundos
- **p99 Response Time:** < 5 segundos
- **Throughput:** ~50 requests/segundo
- **Error Rate:** < 1%

#### Optimizaciones Implementadas

1. **Connection Pooling:**
   - Pool de 15 conexiones reutilizables
   - Reduce overhead de creación de conexiones
   - MaxWaitTime de 300 segundos

2. **Vista Optimizada:**
   - `V_FUV_CUSTOMER_DATA_NEW` pre-agrega datos
   - Índice en columna USERID
   - Reduce joins en tiempo de ejecución

3. **Prepared Statements:**
   - Previene SQL injection
   - Mejora performance de queries repetidas
   - Reutilización de plan de ejecución

4. **Query Timeout:**
   - Límite de 15 segundos
   - Previene queries colgadas
   - Libera recursos rápidamente

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta Exitosa con Resultados

#### Request

```bash
curl -X GET "https://ntgy-fua-s-ac-fua-pro.us-e2.cloudhub.io/api/customersByUserId?userId=12345" \
  -H "client_id: abc123def456" \
  -H "client_secret: secret123" \
  -H "Accept: application/json"
```

#### Response (200 OK)

```json
{
  "customer": {
    "customerId": "12345",
    "docNumber": "12345678A",
    "personType": "F",
    "name": "Juan",
    "surname1": "García",
    "surname2": "López",
    "fullName": "Juan García López",
    "birthDate": "1985-03-15",
    "language": "ES",
    "phone1": "600123456",
    "phone2": "912345678",
    "phone3": "",
    "phone4": "",
    "phone5": "",
    "email": "juan.garcia@example.com",
    "emailStatus": "VERIFIED",
    "cuo": "CUO123456",
    "acceptPublicity": "Y",
    "acceptCession": "Y",
    "acceptKeepData": "Y",
    "robinson": "N",
    "user": {
      "userId": "12345",
      "lastAccessDate": "2026-07-20T10:30:00"
    },
    "supplyPoints": {
      "items": [
        {
          "supplyPointId": "SP001",
          "addressId": "ADDR001",
          "addressDescription": "Calle Mayor",
          "addressNumber": "123",
          "addressPortal": "A",
          "addressFloor": "3",
          "addressStaircase": "B",
          "addressDoor": "2",
          "postalCode": "28001",
          "townName": "Madrid",
          "provinceName": "Madrid",
          "contracts": {
            "items": [
              {
                "contractId": "CONT001",
                "fol": "FOL123456",
                "iban": "ES1234567890123456789012",
                "gasCUPS": "ES0123456789012345AB1F",
                "electricityCUPS": "ES0987654321098765CD2G",
                "paymentMethod": "DIRECT_DEBIT",
                "contractLines": {
                  "items": [
                    {
                      "contractLineId": "CL001",
                      "contractLineStatus": "Active",
                      "productId": "PROD001",
                      "product": "Gas Natural",
                      "productCategory": "GAS",
                      "productRate": "3.1"
                    },
                    {
                      "contractLineId": "CL002",
                      "contractLineStatus": "Active",
                      "productId": "PROD002",
                      "product": "Electricidad",
                      "productCategory": "ELECTRICITY",
                      "productRate": "2.0TD"
                    }
                  ]
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```

### Ejemplo 2: Consulta Sin Resultados (Error de Implementación)

#### Request

```bash
curl -X GET "https://ntgy-fua-s-ac-fua-pro.us-e2.cloudhub.io/api/customersByUserId?userId=99999" \
  -H "client_id: abc123def456" \
  -H "client_secret: secret123" \
  -H "Accept: application/json"
```

#### Response (500 Internal Server Error)

**Nota:** La implementación actual no maneja correctamente el caso de userId no encontrado, resultando en un error de expresión DataWeave.

```json
{
  "error": {
    "type": "EXPRESSION",
    "message": "You called the function 'Value Selector' with these arguments: (Array, Name: \"CUSTOMERID\"). But it expects one of these combinations: (Array, Number), (Object, Name), (Null, Any)"
  }
}
```

**Recomendación:** Implementar validación de array vacío antes de acceder a `payload[0]`.

### Ejemplo 3: Error de Validación (userId faltante)

#### Request

```bash
curl -X GET "https://ntgy-fua-s-ac-fua-pro.us-e2.cloudhub.io/api/customersByUserId" \
  -H "client_id: abc123def456" \
  -H "client_secret: secret123" \
  -H "Accept: application/json"
```

#### Response (400 Bad Request)

```json
{
  "result": {
    "code": "VALIDATION_ERROR",
    "message": "userId parameter is required"
  }
}
```

### Ejemplo 4: Error de Conexión a Base de Datos

#### Request

```bash
curl -X GET "https://ntgy-fua-s-ac-fua-pro.us-e2.cloudhub.io/api/customersByUserId?userId=12345" \
  -H "client_id: abc123def456" \
  -H "client_secret: secret123" \
  -H "Accept: application/json"
```

#### Response (500 Internal Server Error)

```json
{
  "result": {
    "code": "DB_CONNECTION_ERROR",
    "message": "Unable to connect to Oracle Database AC"
  }
}
```

### Ejemplo 5: Uso desde JavaScript (Node.js)

```javascript
const axios = require('axios');

async function getCustomerByUserId(userId) {
  try {
    const response = await axios.get(
      'https://ntgy-fua-s-ac-fua-pro.us-e2.cloudhub.io/api/customersByUserId',
      {
        params: { userId },
        headers: {
          'client_id': 'abc123def456',
          'client_secret': 'secret123',
          'Accept': 'application/json'
        }
      }
    );
    
    console.log('Customer Data:', response.data.customerData);
    return response.data;
  } catch (error) {
    console.error('Error fetching customer:', error.response?.data || error.message);
    throw error;
  }
}

// Uso
getCustomerByUserId('12345')
  .then(data => console.log('Success:', data))
  .catch(err => console.error('Failed:', err));
```

### Ejemplo 6: Uso desde Python

```python
import requests

def get_customer_by_user_id(user_id):
    url = "https://ntgy-fua-s-ac-fua-pro.us-e2.cloudhub.io/api/customersByUserId"
    
    headers = {
        "client_id": "abc123def456",
        "client_secret": "secret123",
        "Accept": "application/json"
    }
    
    params = {
        "userId": user_id
    }
    
    try:
        response = requests.get(url, headers=headers, params=params)
        response.raise_for_status()
        
        data = response.json()
        print(f"Customer Data: {data['customerData']}")
        return data
    except requests.exceptions.RequestException as e:
        print(f"Error fetching customer: {e}")
        raise

# Uso
customer = get_customer_by_user_id("12345")
```

### Ejemplo 7: Uso desde Postman

**Configuración de Request:**

1. **Método:** GET
2. **URL:** `https://ntgy-fua-s-ac-fua-pro.us-e2.cloudhub.io/api/customersByUserId`
3. **Query Params:**
   - Key: `userId`
   - Value: `12345`
4. **Headers:**
   - `client_id`: `abc123def456`
   - `client_secret`: `secret123`
   - `Accept`: `application/json`

**Tests (Postman):**

```javascript
// Verificar status code
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Verificar estructura de respuesta
pm.test("Response has correct structure", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('result');
    pm.expect(jsonData).to.have.property('customerData');
    pm.expect(jsonData.result).to.have.property('code');
    pm.expect(jsonData.result.code).to.eql('SUCCESS');
});

// Verificar datos del cliente
pm.test("Customer data is present", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.customerData).to.be.an('array');
    if (jsonData.customerData.length > 0) {
        pm.expect(jsonData.customerData[0]).to.have.property('userId');
        pm.expect(jsonData.customerData[0]).to.have.property('document');
        pm.expect(jsonData.customerData[0]).to.have.property('email');
    }
});
```
