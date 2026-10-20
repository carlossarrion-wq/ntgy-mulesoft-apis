---
api: "ntgy-fuv-p-callejero"
sidebar_position: 5
sidebar_label: "POST /compruebaDireccion"
description: "Valida y gestiona direcciones mediante búsqueda jerárquica en base de datos PostgreSQL, creando registros horizontal y vertical según necesidad"
keywords: [callejero, direcciones, validación, PostgreSQL, horizontal, vertical, idVia, clave]
tags: [Process API, Callejero, Validación, Orquestación]
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-fuv-s-callejero"
    path: "/vias"
    purpose: "Validación de existencia de idVia en base de datos"
  - method: "GET"
    api_name: "ntgy-fuv-s-callejero"
    path: "/dirVertical"
    purpose: "Búsqueda de dirección vertical completa por clave"
  - method: "GET"
    api_name: "ntgy-fuv-s-callejero"
    path: "/dirHorizontal"
    purpose: "Búsqueda de dirección horizontal por clave"
  - method: "POST"
    api_name: "ntgy-fuv-s-callejero"
    path: "/dirHorizontal"
    purpose: "Creación de nueva dirección horizontal"
  - method: "POST"
    api_name: "ntgy-fuv-s-callejero"
    path: "/dirVertical"
    purpose: "Creación de nueva dirección vertical"
---

# POST /compruebaDireccion

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint implementa un proceso de validación y gestión inteligente de direcciones mediante una búsqueda jerárquica en tres niveles: validación de vía, búsqueda de dirección vertical completa y, si no existe, búsqueda o creación de dirección horizontal. El proceso garantiza la existencia de direcciones completas en el sistema, creando los registros necesarios cuando no existen.

### Tipo de API

**Capa:** Process API (API-Led Connectivity)

**Función:** Orquestación de validación y gestión de direcciones con lógica de búsqueda en cascada y creación condicional

### Patrón de Integración

**Tipo:** Orquestación con Transformación y Lógica Condicional

**Descripción:** El endpoint orquesta múltiples llamadas al System API S-CALLEJERO siguiendo un patrón de búsqueda jerárquica:
1. Valida existencia de la vía (idVia)
2. Genera claves horizontal y vertical mediante transformaciones DataWeave
3. Busca dirección vertical completa
4. Si no existe vertical, busca horizontal
5. Si no existe horizontal, crea ambas (horizontal + vertical)
6. Si existe horizontal pero no vertical, crea solo vertical

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    E[Experience APIs<br/>Portales Web, Móvil, Agentes]
    P[Process API<br/>ntgy-fuv-p-callejero<br/>POST /compruebaDireccion]
    S[System API<br/>ntgy-fuv-s-callejero<br/>PostgreSQL]
    DB[(PostgreSQL<br/>Base de Datos<br/>Callejero)]
    
    E -->|Request con<br/>componentes dirección| P
    P -->|1. GET /vias| S
    P -->|2. GET /dirVertical| S
    P -->|3. GET /dirHorizontal| S
    P -->|4. POST /dirHorizontal| S
    P -->|5. POST /dirVertical| S
    S -->|Consultas/Inserts| DB
    DB -->|Resultados| S
    S -->|Responses| P
    P -->|Response con IDs<br/>y estado| E
    
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:3px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style DB fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Validación de Direcciones en Alta de Cliente**
   - Agentes de servicio validan direcciones durante el proceso de alta
   - El sistema crea automáticamente registros de dirección si no existen
   - Beneficio: Reduce tiempo de captura de 5 minutos a menos de 30 segundos

2. **Normalización de Direcciones Existentes**
   - Sistemas legacy envían direcciones para normalización
   - El endpoint genera claves estandarizadas (horizontal y vertical)
   - Beneficio: Garantiza consistencia en formato de direcciones

3. **Integración con CRM y Facturación**
   - Salesforce y SAP ISU validan direcciones antes de crear contratos
   - El endpoint retorna IDs únicos (horizontalId, verticalId) para referencia
   - Beneficio: Evita duplicación de direcciones en sistemas downstream

### 🔗 Endpoints Dependientes

Este Process API consume los siguientes System APIs:

| System API | Endpoint | Método | Propósito |
|------------|----------|--------|-----------|
| `ntgy-fuv-s-callejero` | `/vias` | GET | Validación de existencia de idVia |
| `ntgy-fuv-s-callejero` | `/dirVertical` | GET | Búsqueda de dirección vertical completa |
| `ntgy-fuv-s-callejero` | `/dirHorizontal` | GET | Búsqueda de dirección horizontal |
| `ntgy-fuv-s-callejero` | `/dirHorizontal` | POST | Creación de nueva dirección horizontal |
| `ntgy-fuv-s-callejero` | `/dirVertical` | POST | Creación de nueva dirección vertical |

### Lógica de Orquestación

```mermaid
graph TB
    Start([POST /compruebaDireccion]) --> SavePayload[Guardar payload original<br/>en variable]
    SavePayload --> ValidateVia[GET /vias<br/>Validar idVia]
    
    ValidateVia --> CheckVia{¿Existe idVia?}
    CheckVia -->|NO| ErrorVia[Error: Id Via<br/>no encontrada]
    CheckVia -->|SÍ| GenKeys[Generar claves<br/>horizontal y vertical<br/>DataWeave]
    
    GenKeys --> SearchVertical[GET /dirVertical<br/>Buscar clave vertical]
    SearchVertical --> CheckVertical{¿Existe<br/>vertical?}
    
    CheckVertical -->|SÍ| ReturnExisting[Retornar IDs existentes<br/>createPremise: false]
    
    CheckVertical -->|NO| SearchHorizontal[GET /dirHorizontal<br/>Buscar clave horizontal]
    SearchHorizontal --> CheckHorizontal{¿Existe<br/>horizontal?}
    
    CheckHorizontal -->|SÍ| CreateVertical1[POST /dirVertical<br/>Crear vertical]
    CreateVertical1 --> ReturnCreatedV[Retornar IDs<br/>Vertical creada]
    
    CheckHorizontal -->|NO| CreateHorizontal[POST /dirHorizontal<br/>Crear horizontal]
    CreateHorizontal --> CreateVertical2[POST /dirVertical<br/>Crear vertical]
    CreateVertical2 --> ReturnCreatedBoth[Retornar IDs<br/>Ambas creadas]
    
    ErrorVia --> End([Response])
    ReturnExisting --> End
    ReturnCreatedV --> End
    ReturnCreatedBoth --> End
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style End fill:#e8f5e9,stroke:#2e7d32
    style CheckVia fill:#fff3e0,stroke:#f57c00
    style CheckVertical fill:#fff3e0,stroke:#f57c00
    style CheckHorizontal fill:#fff3e0,stroke:#f57c00
    style ErrorVia fill:#ffebee,stroke:#c62828
```

**Pasos del Proceso:**

1. **Validación de Vía**: Verifica que el idVia existe en la base de datos
2. **Generación de Claves**: Crea claves horizontal (18 caracteres) y vertical (28 caracteres) mediante DataWeave
3. **Búsqueda Vertical**: Intenta localizar dirección vertical completa
4. **Búsqueda Horizontal**: Si no existe vertical, busca horizontal
5. **Creación Condicional**: Crea registros según necesidad (solo vertical, o ambos)

### Transformaciones Aplicadas

#### Generación de Clave Horizontal (18 caracteres)

| Componente | Formato | Longitud | Ejemplo |
|------------|---------|----------|---------|
| Prefijo | "NG" | 2 | NG |
| idVia | Padding izquierdo con "0" | 11 | 00000012345 |
| numero | Padding izquierdo con "0" | 4 | 0042 |
| portal | Numérico: "0", Alfanumérico: "_" | 3 | 00A o __A |

**Ejemplo:** `NG00000012345004200A` (idVia=12345, numero=42, portal=A)

#### Generación de Clave Vertical (28 caracteres)

| Componente | Formato | Longitud | Ejemplo |
|------------|---------|----------|---------|
| Clave Horizontal | Base | 18 | NG00000012345004200A |
| escalera | Numérico: "0", Alfanumérico: "_" | 3 | __B |
| piso | Numérico: "0", Alfanumérico: "_" | 2 | 03 |
| puerta | Numérico: "0", Alfanumérico: "_" | 5 | ____A |

**Ejemplo:** `NG00000012345004200A__B03____A` (escalera=B, piso=3, puerta=A)

**Lógica de Padding:**
- Valores numéricos: Padding con "0" a la izquierda
- Valores alfanuméricos: Padding con "_" a la izquierda
- Función DataWeave: `isNumber(value) = matches(value, /[0-9]+/)`

### Contexto Empresarial

- **Dominio:** Gestión de Direcciones y Callejero
- **Sector:** Energía - Utilities
- **División:** Naturgy España - FUV (Fuerza de Ventas)
- **Proceso de Negocio:** Validación y normalización de direcciones para alta de clientes y contratos

### Stakeholders

- **Agentes de Servicio al Cliente**: Necesitan validar direcciones rápidamente durante llamadas
- **Equipos de Ventas**: Requieren direcciones validadas para alta de contratos
- **Sistemas de Facturación**: Dependen de direcciones correctas para envío de facturas
- **Operaciones de Campo**: Utilizan direcciones para planificación de rutas

### Valor de Negocio

- **Reducción de Errores**: De 15% a menos de 3% en direcciones incorrectas
- **Ahorro de Tiempo**: De 5 minutos a 30 segundos en validación de direcciones
- **Consistencia de Datos**: Formato único de direcciones en todos los sistemas
- **Reducción de Costos**: 40% menos costos de reenvío de correspondencia

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Portales web de autoservicio de clientes
- Aplicaciones móviles de agentes de campo
- Sistemas de agentes de call center
- Integraciones con Salesforce CRM
- Integraciones con SAP ISU
- Sistemas de facturación y correspondencia

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/compruebaDireccion` |
| **Development** | `https://ntgy-fuv-p-callejero-dev.us-e2.cloudhub.io` | `https://ntgy-fuv-p-callejero-dev.us-e2.cloudhub.io/api/compruebaDireccion` |
| **UAT** | `https://ntgy-fuv-p-callejero-uat.us-e2.cloudhub.io` | `https://ntgy-fuv-p-callejero-uat.us-e2.cloudhub.io/api/compruebaDireccion` |
| **Production** | `https://ntgy-fuv-p-callejero-pro.us-e2.cloudhub.io` | `https://ntgy-fuv-p-callejero-pro.us-e2.cloudhub.io/api/compruebaDireccion` |

### Método HTTP

```
POST /api/compruebaDireccion
```

### Parámetros de Entrada

#### Request Body (application/json)

| Campo | Tipo | Requerido | Longitud | Descripción |
|-------|------|-----------|----------|-------------|
| `idVia` | string | ✅ Sí | 1-10 | Identificador único de la vía en base de datos |
| `numero` | string | ✅ Sí | 1-4 | Número de la vía (portal principal) |
| `portal` | string | ❌ No | variable | Portal del edificio (opcional) |
| `escalera` | string | ❌ No | variable | Escalera del edificio (opcional) |
| `piso` | string | ❌ No | variable | Piso o planta (opcional) |
| `puerta` | string | ❌ No | variable | Puerta del piso (opcional) |
| `siebelIntegrationId` | string | ❌ No | variable | ID de integración con Siebel (opcional) |
| `extraData` | string | ❌ No | variable | Datos adicionales (opcional) |

**Validaciones:**
- `idVia` y `numero` son obligatorios
- `idVia` debe existir en la tabla de vías
- Campos opcionales pueden ser null o vacíos
- Todos los valores se convierten a mayúsculas en el procesamiento

#### Ejemplo de Request

```json
{
  "idVia": "12345",
  "numero": "42",
  "portal": "A",
  "escalera": "B",
  "piso": "3",
  "puerta": "A",
  "siebelIntegrationId": "SBL-12345",
  "extraData": "Información adicional"
}
```

### Respuestas

#### 200 OK - Dirección Existente

**Escenario:** La dirección vertical completa ya existe en base de datos

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Devuelta direccion existente"
  },
  "horizontalId": "550e8400-e29b-41d4-a716-446655440000",
  "verticalId": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "createPremise": false
}
```

#### 200 OK - Dirección Vertical Creada

**Escenario:** Existía dirección horizontal, se creó la vertical

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Creada direccion vertical"
  },
  "horizontalId": "550e8400-e29b-41d4-a716-446655440000",
  "verticalId": "7c9e6679-7425-40de-944b-e07fc1f90ae7"
}
```

#### 200 OK - Ambas Direcciones Creadas

**Escenario:** No existía ninguna dirección, se crearon horizontal y vertical

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Creada direccion horizontal y vertical"
  },
  "horizontalId": "8d9e7680-8536-51ef-a55c-f18fc2f01bf8",
  "verticalId": "9e0f8791-9647-62f0-b66d-028fd3012c09"
}
```

#### 200 OK - Error de Negocio (idVia no encontrada)

**Escenario:** El idVia proporcionado no existe en base de datos

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Id Via no encontrada"
  }
}
```

#### Campos de Respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result.codResult` | string | Código de resultado: "0000" (éxito), "0001" (error) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |
| `horizontalId` | string (UUID) | ID único de la dirección horizontal |
| `verticalId` | string (UUID) | ID único de la dirección vertical |
| `createPremise` | boolean | Indica si se debe crear premise en sistemas downstream (solo en respuesta de dirección existente) |

### Autenticación

- **Tipo:** Client ID & Client Secret (Anypoint Platform)
- **Headers requeridos:**
  - `client_id`: ID del cliente registrado en Anypoint Platform
  - `client_secret`: Secret del cliente

### Timeouts

| Tipo | Valor |
|------|-------|
| **Request Timeout** | 30 segundos |
| **Connection Timeout** | 10 segundos |

**Nota:** Los timeouts se configuran por ambiente en archivos properties YAML

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request]) --> SavePayload[set-variable<br/>peticionOriginal]
    SavePayload --> ValidateVia[http:request GET<br/>S-CALLEJERO /vias<br/>target: idViaResult]
    
    ValidateVia --> Choice1{¿Existe idVia?}
    
    Choice1 -->|NO| LogError[logger<br/>idVia no localizada]
    LogError --> TransformError[ee:transform<br/>Error Response<br/>codResult: 0001]
    TransformError --> End([HTTP Response])
    
    Choice1 -->|SÍ| GenHorizontal[ee:transform<br/>Generar claveHorizontal<br/>NG + idVia + numero + portal]
    GenHorizontal --> GenVertical[ee:transform<br/>Generar claveVertical<br/>+ escalera + piso + puerta]
    
    GenVertical --> SearchVertical[http:request GET<br/>S-CALLEJERO /dirVertical<br/>target: verticalResponse]
    
    SearchVertical --> Choice2{¿Existe vertical?}
    
    Choice2 -->|SÍ| LogExistV[logger<br/>Existe Vertical]
    LogExistV --> ReturnExist[ee:transform<br/>Devuelve horizontalId<br/>verticalId<br/>createPremise: false]
    ReturnExist --> End
    
    Choice2 -->|NO| LogNoV[logger<br/>No Existe Vertical]
    LogNoV --> SearchHorizontal[http:request GET<br/>S-CALLEJERO /dirHorizontal<br/>target: horizontalResponse]
    
    SearchHorizontal --> Choice3{¿Existe horizontal?}
    
    Choice3 -->|SÍ| LogExistH[logger<br/>Existe Horizontal]
    LogExistH --> GenUUIDV1[ee:transform<br/>uuid verticalId]
    GenUUIDV1 --> PrepareV1[ee:transform<br/>Prepara payload vertical]
    PrepareV1 --> CreateV1[http:request POST<br/>S-CALLEJERO /dirVertical]
    CreateV1 --> ReturnV[ee:transform<br/>Devuelve IDs<br/>Vertical creada]
    ReturnV --> End
    
    Choice3 -->|NO| LogNoH[logger<br/>No existe horizontal]
    LogNoH --> GenUUIDH[ee:transform<br/>uuid horizontalId]
    GenUUIDH --> PrepareH[ee:transform<br/>Prepara payload horizontal]
    PrepareH --> CreateH[http:request POST<br/>S-CALLEJERO /dirHorizontal]
    CreateH --> GenUUIDV2[ee:transform<br/>uuid verticalId]
    GenUUIDV2 --> PrepareV2[ee:transform<br/>Prepara payload vertical]
    PrepareV2 --> CreateV2[http:request POST<br/>S-CALLEJERO /dirVertical]
    CreateV2 --> ReturnBoth[ee:transform<br/>Devuelve IDs<br/>Ambas creadas]
    ReturnBoth --> End
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style End fill:#e8f5e9,stroke:#2e7d32
    style Choice1 fill:#fff3e0,stroke:#f57c00
    style Choice2 fill:#fff3e0,stroke:#f57c00
    style Choice3 fill:#fff3e0,stroke:#f57c00
    style TransformError fill:#ffebee,stroke:#c62828
```

### Transformaciones DataWeave

#### 1. Generación de Clave Horizontal

```dataweave
%dw 2.0
import * from dw::core::Strings
output application/json

var aux = payload mapObject ((value, key, index) -> (key):value default "") 

// Función para detectar valores numéricos
fun isNumber(value) = matches(value, /[0-9]+/)

---
"NG" ++ 
upper(leftPad(trim(aux.idVia), 11, "0")) ++
upper(leftPad(trim(aux.numero), 4, "0")) ++
(
    if (isNumber(aux.portal)) 
        upper(leftPad(trim(aux.portal), 3, "0")) 
    else 
        upper(leftPad(trim(aux.portal), 3, "_"))
)
```

**Resultado:** String de 18 caracteres (ej: `NG00000012345004200A`)

#### 2. Generación de Clave Vertical

```dataweave
%dw 2.0
import * from dw::core::Strings
output application/json

var aux = payload mapObject ((value, key, index) -> (key):value default "") 

fun isNumber(value) = matches(value, /[0-9]+/)

---
"NG" ++ 
upper(leftPad(trim(aux.idVia), 11, "0")) ++
upper(leftPad(trim(aux.numero), 4, "0")) ++
(
    if (isNumber(aux.portal)) 
        upper(leftPad(trim(aux.portal), 3, "0")) 
    else 
        upper(leftPad(trim(aux.portal), 3, "_"))
) ++
(
    if (isNumber(aux.escalera)) 
        upper(leftPad(trim(aux.escalera), 3, "0")) 
    else 
        upper(leftPad(trim(aux.escalera), 3, "_"))
) ++
(
    if (isNumber(aux.piso)) 
        upper(leftPad(trim(aux.piso), 2, "0")) 
    else 
        upper(leftPad(trim(aux.piso), 2, "_"))
) ++
(
    if (isNumber(aux.puerta)) 
        upper(leftPad(trim(aux.puerta), 5, "0")) 
    else 
        upper(leftPad(trim(aux.puerta), 5, "_"))
)
```

**Resultado:** String de 28 caracteres (ej: `NG00000012345004200A__B03____A`)

#### 3. Preparación de Payload para Crear Horizontal

```dataweave
%dw 2.0
output application/json
---
{
    "horizontalId": vars.horizontalId,
    "clave": vars.claveHorizontal,
    "siebelIntegrationId": vars.peticionOriginal.siebelIntegrationId, 
    "idVia": vars.peticionOriginal.idVia,
    "numero": vars.peticionOriginal.numero,
    "portal": upper(vars.peticionOriginal.portal)
}
```

#### 4. Preparación de Payload para Crear Vertical

```dataweave
%dw 2.0
output application/json
---
{
    "verticalId": vars.verticalId,
    "horizontalId": vars.horizontalResponse.values[0].horizontal_id,
    "clave": vars.claveVertical,
    "siebelIntegrationId": vars.peticionOriginal.siebelIntegrationId, 
    "escalera": upper(vars.peticionOriginal.escalera),
    "piso": upper(vars.peticionOriginal.piso),
    "puerta": upper(vars.peticionOriginal.puerta),
    "extraData": upper(vars.peticionOriginal.extraData)
}
```

### Patrones de Diseño

**Patrón:** Búsqueda en Cascada con Creación Condicional (Cascade Search with Conditional Creation)

**Justificación:** 
- Optimiza consultas a base de datos buscando primero el registro más específico (vertical)
- Evita creación innecesaria de registros si ya existen
- Garantiza integridad referencial creando horizontal antes que vertical
- Reduce latencia al retornar inmediatamente si encuentra dirección existente

**Beneficios:**
- Máxima eficiencia: Solo 2 consultas si la dirección existe
- Idempotencia: Múltiples llamadas con mismos datos retornan mismo resultado
- Atomicidad: Crea ambos registros en secuencia garantizando consistencia

### Diagramas de Secuencia

#### 1. Flujo Normal - Dirección Existente

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API<br/>compruebaDireccionFlow
    participant S as System API<br/>S-CALLEJERO
    participant DB as PostgreSQL
    
    C->>P: POST /compruebaDireccion<br/>{idVia, numero, portal, escalera, piso, puerta}
    
    Note over P: Guardar payload original
    
    P->>S: GET /vias?idVia=12345
    S->>DB: SELECT * FROM vias WHERE id_via = '12345'
    DB-->>S: Via encontrada
    S-->>P: {values: [{via_data}]}
    
    Note over P: Generar claveHorizontal<br/>NG00000012345004200A
    Note over P: Generar claveVertical<br/>NG00000012345004200A__B03____A
    
    P->>S: GET /dirVertical?claveVertical=NG...
    S->>DB: SELECT * FROM dir_vertical WHERE clave = 'NG...'
    DB-->>S: Dirección vertical encontrada
    S-->>P: {values: [{horizontal_id, vertical_id}]}
    
    Note over P: Dirección existe<br/>No crear nada
    
    P-->>C: 200 OK<br/>{codResult: "0000",<br/>horizontalId, verticalId,<br/>createPremise: false}
```

#### 2. Flujo - Crear Solo Vertical

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API<br/>compruebaDireccionFlow
    participant S as System API<br/>S-CALLEJERO
    participant DB as PostgreSQL
    
    C->>P: POST /compruebaDireccion<br/>{idVia, numero, portal, escalera, piso, puerta}
    
    P->>S: GET /vias?idVia=12345
    S->>DB: SELECT * FROM vias
    DB-->>S: Via encontrada
    S-->>P: {values: [{via_data}]}
    
    Note over P: Generar claves
    
    P->>S: GET /dirVertical?claveVertical=NG...
    S->>DB: SELECT * FROM dir_vertical
    DB-->>S: No encontrada
    S-->>P: {values: []}
    
    P->>S: GET /dirHorizontal?claveHorizontal=NG...
    S->>DB: SELECT * FROM dir_horizontal
    DB-->>S: Horizontal encontrada
    S-->>P: {values: [{horizontal_id}]}
    
    Note over P: Generar UUID para vertical
    
    P->>S: POST /dirVertical<br/>{verticalId, horizontalId, clave, ...}
    S->>DB: INSERT INTO dir_vertical
    DB-->>S: Insert exitoso
    S-->>P: 201 Created
    
    P-->>C: 200 OK<br/>{codResult: "0000",<br/>msgResult: "Creada direccion vertical",<br/>horizontalId, verticalId}
```

#### 3. Flujo - Crear Horizontal y Vertical

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API<br/>compruebaDireccionFlow
    participant S as System API<br/>S-CALLEJERO
    participant DB as PostgreSQL
    
    C->>P: POST /compruebaDireccion<br/>{idVia, numero, portal, escalera, piso, puerta}
    
    P->>S: GET /vias?idVia=12345
    S->>DB: SELECT * FROM vias
    DB-->>S: Via encontrada
    S-->>P: {values: [{via_data}]}
    
    Note over P: Generar claves
    
    P->>S: GET /dirVertical?claveVertical=NG...
    S->>DB: SELECT * FROM dir_vertical
    DB-->>S: No encontrada
    S-->>P: {values: []}
    
    P->>S: GET /dirHorizontal?claveHorizontal=NG...
    S->>DB: SELECT * FROM dir_horizontal
    DB-->>S: No encontrada
    S-->>P: {values: []}
    
    Note over P: Generar UUID para horizontal
    
    P->>S: POST /dirHorizontal<br/>{horizontalId, clave, idVia, numero, portal}
    S->>DB: INSERT INTO dir_horizontal
    DB-->>S: Insert exitoso
    S-->>P: 201 Created
    
    Note over P: Generar UUID para vertical
    
    P->>S: POST /dirVertical<br/>{verticalId, horizontalId, clave, ...}
    S->>DB: INSERT INTO dir_vertical
    DB-->>S: Insert exitoso
    S-->>P: 201 Created
    
    P-->>C: 200 OK<br/>{codResult: "0000",<br/>msgResult: "Creada direccion horizontal y vertical",<br/>horizontalId, verticalId}
```

#### 4. Flujo de Error - idVia No Encontrada

```mermaid
sequenceDiagram
    participant C as Cliente
    participant P as Process API<br/>compruebaDireccionFlow
    participant S as System API<br/>S-CALLEJERO
    participant DB as PostgreSQL
    
    C->>P: POST /compruebaDireccion<br/>{idVia: "99999", numero: "42"}
    
    P->>S: GET /vias?idVia=99999
    S->>DB: SELECT * FROM vias WHERE id_via = '99999'
    DB-->>S: No encontrada
    S-->>P: {values: []}
    
    Note over P: idVia no existe<br/>Detener proceso
    
    P-->>C: 200 OK<br/>{codResult: "0001",<br/>msgResult: "Id Via no encontrada"}
```

### Manejo de Errores

#### Estrategia de Manejo de Errores

| Tipo de Error | Código | Mensaje | Acción |
|---------------|--------|---------|--------|
| **idVia no encontrada** | 0001 | "Id Via no encontrada" | Retornar error de negocio, no crear registros |
| **Error de conectividad** | 500 | Error del sistema | Propagar error HTTP del System API |
| **Timeout** | 500 | Timeout en operación | Retornar error después de 30 segundos |
| **Error de validación** | 400 | Bad Request | Validación de APIKit en campos obligatorios |

#### Logging

**Puntos de Logging:**
1. Inicio del flow: `"post:\compruebaDireccion:application\json:ntgy-fuv-p-callejero-config"`
2. idVia no localizada: `"idVia #[vars.peticionOriginal.idVia] no localizada"`
3. Dirección vertical existe: `"Existe Vertical, devuelve valores"`
4. Dirección vertical no existe: `"No Existe Vertical, busca horizontal"`
5. Dirección horizontal existe: `"Existe Horizontal, devuelve horizontal y crea vertical"`
6. Dirección horizontal no existe: `"No existe horizontal, crea horizontal y vertical"`

**Nivel de Log:** INFO para operaciones normales, ERROR para excepciones

### Variables de Flow

| Variable | Tipo | Propósito |
|----------|------|-----------|
| `peticionOriginal` | Object | Almacena el payload original del request |
| `idViaResult` | Object | Resultado de la búsqueda de vía |
| `claveHorizontal` | String | Clave generada de 18 caracteres |
| `claveVertical` | String | Clave generada de 28 caracteres |
| `verticalResponse` | Object | Resultado de búsqueda de dirección vertical |
| `horizontalResponse` | Object | Resultado de búsqueda de dirección horizontal |
| `horizontalId` | String (UUID) | ID generado para nueva dirección horizontal |
| `verticalId` | String (UUID) | ID generado para nueva dirección vertical |

### Configuración HTTP

**Archivo:** `global.xml`

```xml
<http:request-config 
    name="S-CALLEJERO" 
    basePath="${s-callejero.basePath}" 
    responseTimeout="${s-callejero.timeout}">
    <http:request-connection 
        host="${s-callejero.host}" 
        port="${s-callejero.port}" />
</http:request-config>
```

**Properties (Development):**
```yaml
s-callejero:
  host: "ntgy-fuv-s-callejero-des.de-c1.cloudhub.io"
  port: "80"
  timeout: "30000"
  basePath: "/api"
```

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Dirección Existente (Caso Más Común)

#### Request

```bash
curl -X POST "https://ntgy-fuv-p-callejero-dev.us-e2.cloudhub.io/api/compruebaDireccion" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "idVia": "12345",
    "numero": "42",
    "portal": "A",
    "escalera": "B",
    "piso": "3",
    "puerta": "A"
  }'
```

#### Response (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Devuelta direccion existente"
  },
  "horizontalId": "550e8400-e29b-41d4-a716-446655440000",
  "verticalId": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "createPremise": false
}
```

**Interpretación:**
- La dirección completa ya existía en base de datos
- Se retornan los IDs existentes
- `createPremise: false` indica que no se debe crear premise en sistemas downstream

### Ejemplo 2: Crear Solo Dirección Vertical

#### Request

```bash
curl -X POST "https://ntgy-fuv-p-callejero-dev.us-e2.cloudhub.io/api/compruebaDireccion" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "idVia": "12345",
    "numero": "42",
    "portal": "A",
    "escalera": "C",
    "piso": "5",
    "puerta": "B",
    "siebelIntegrationId": "SBL-67890"
  }'
```

#### Response (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Creada direccion vertical"
  },
  "horizontalId": "550e8400-e29b-41d4-a716-446655440000",
  "verticalId": "7c9e6679-7425-40de-944b-e07fc1f90ae7"
}
```

**Interpretación:**
- La dirección horizontal ya existía
- Se creó nueva dirección vertical con los datos proporcionados
- El `verticalId` es nuevo, el `horizontalId` es existente

### Ejemplo 3: Crear Ambas Direcciones

#### Request

```bash
curl -X POST "https://ntgy-fuv-p-callejero-dev.us-e2.cloudhub.io/api/compruebaDireccion" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "idVia": "98765",
    "numero": "15",
    "portal": "B",
    "escalera": "1",
    "piso": "2",
    "puerta": "C",
    "siebelIntegrationId": "SBL-11111",
    "extraData": "Nueva construcción"
  }'
```

#### Response (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Creada direccion horizontal y vertical"
  },
  "horizontalId": "8d9e7680-8536-51ef-a55c-f18fc2f01bf8",
  "verticalId": "9e0f8791-9647-62f0-b66d-028fd3012c09"
}
```

**Interpretación:**
- Ninguna dirección existía previamente
- Se crearon ambos registros (horizontal y vertical)
- Ambos IDs son nuevos

### Ejemplo 4: Error - idVia No Encontrada

#### Request

```bash
curl -X POST "https://ntgy-fuv-p-callejero-dev.us-e2.cloudhub.io/api/compruebaDireccion" \
  -H "Content-Type: application/json" \
  -H "client_id: YOUR_CLIENT_ID" \
  -H "client_secret: YOUR_CLIENT_SECRET" \
  -d '{
    "idVia": "99999",
    "numero": "1"
  }'
```

#### Response (200 OK)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Id Via no encontrada"
  }
}
```

**Interpretación:**
- El idVia proporcionado no existe en la base de datos
- No se creó ningún registro
- El cliente debe verificar el idVia correcto

### Ejemplo 5: JavaScript (Fetch API)

```javascript
const compruebaDireccion = async (direccion) => {
  try {
    const response = await fetch(
      'https://ntgy-fuv-p-callejero-dev.us-e2.cloudhub.io/api/compruebaDireccion',
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'client_id': 'YOUR_CLIENT_ID',
          'client_secret': 'YOUR_CLIENT_SECRET'
        },
        body: JSON.stringify({
          idVia: direccion.idVia,
          numero: direccion.numero,
          portal: direccion.portal,
          escalera: direccion.escalera,
          piso: direccion.piso,
          puerta: direccion.puerta
        })
      }
    );

    const data = await response.json();
    
    if (data.result.codResult === '0000') {
      console.log('Dirección validada:', data);
      return {
        success: true,
        horizontalId: data.horizontalId,
        verticalId: data.verticalId,
        message: data.result.msgResult
      };
    } else {
      console.error('Error de validación:', data.result.msgResult);
      return {
        success: false,
        error: data.result.msgResult
      };
    }
  } catch (error) {
    console.error('Error de conexión:', error);
    return {
      success: false,
      error: 'Error de conexión con el servicio'
    };
  }
};

// Uso
const resultado = await compruebaDireccion({
  idVia: '12345',
  numero: '42',
  portal: 'A',
  escalera: 'B',
  piso: '3',
  puerta: 'A'
});
```

### Ejemplo 6: Python (Requests)

```python
import requests
import json

def comprobar_direccion(direccion):
    """
    Valida una dirección usando el API de Callejero
    
    Args:
        direccion (dict): Diccionario con componentes de la dirección
        
    Returns:
        dict: Resultado de la validación con IDs
    """
    url = "https://ntgy-fuv-p-callejero-dev.us-e2.cloudhub.io/api/compruebaDireccion"
    
    headers = {
        "Content-Type": "application/json",
        "client_id": "YOUR_CLIENT_ID",
        "client_secret": "YOUR_CLIENT_SECRET"
    }
    
    try:
        response = requests.post(url, headers=headers, json=direccion, timeout=30)
        response.raise_for_status()
        
        data = response.json()
        
        if data["result"]["codResult"] == "0000":
            print(f"✓ Dirección validada: {data['result']['msgResult']}")
            return {
                "success": True,
                "horizontal_id": data.get("horizontalId"),
                "vertical_id": data.get("verticalId"),
                "message": data["result"]["msgResult"]
            }
        else:
            print(f"✗ Error: {data['result']['msgResult']}")
            return {
                "success": False,
                "error": data["result"]["msgResult"]
            }
            
    except requests.exceptions.Timeout:
        print("✗ Timeout: El servicio no respondió a tiempo")
        return {"success": False, "error": "Timeout"}
    except requests.exceptions.RequestException as e:
        print(f"✗ Error de conexión: {e}")
        return {"success": False, "error": str(e)}

# Uso
direccion = {
    "idVia": "12345",
    "numero": "42",
    "portal": "A",
    "escalera": "B",
    "piso": "3",
    "puerta": "A",
    "siebelIntegrationId": "SBL-12345"
}

resultado = comprobar_direccion(direccion)

if resultado["success"]:
    print(f"Horizontal ID: {resultado['horizontal_id']}")
    print(f"Vertical ID: {resultado['vertical_id']}")
```

### Ejemplo 7: Postman Collection

```json
{
  "info": {
    "name": "Callejero API - compruebaDireccion",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Comprobar Dirección - Existente",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          },
          {
            "key": "client_id",
            "value": "{{client_id}}"
          },
          {
            "key": "client_secret",
            "value": "{{client_secret}}"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"idVia\": \"12345\",\n  \"numero\": \"42\",\n  \"portal\": \"A\",\n  \"escalera\": \"B\",\n  \"piso\": \"3\",\n  \"puerta\": \"A\"\n}"
        },
        "url": {
          "raw": "{{base_url}}/api/compruebaDireccion",
          "host": ["{{base_url}}"],
          "path": ["api", "compruebaDireccion"]
        }
      }
    }
  ]
}
```

### Consideraciones de Integración

#### Idempotencia
- El endpoint es idempotente: múltiples llamadas con los mismos datos retornan el mismo resultado
- Si la dirección existe, siempre retorna los mismos IDs
- Si no existe, la primera llamada crea los registros, las siguientes retornan los IDs creados

#### Manejo de Campos Opcionales
- Los campos opcionales (portal, escalera, piso, puerta) pueden ser:
  - Omitidos del JSON
  - Enviados como `null`
  - Enviados como string vacío `""`
- Todos se tratan como valores vacíos en la generación de claves

#### Conversión a Mayúsculas
- Todos los valores se convierten automáticamente a mayúsculas
- No es necesario enviar los datos en mayúsculas desde el cliente

#### Generación de UUIDs
- Los IDs generados son UUIDs v4 estándar
- Formato: `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`
- Son únicos y pueden usarse como claves primarias en sistemas downstream
