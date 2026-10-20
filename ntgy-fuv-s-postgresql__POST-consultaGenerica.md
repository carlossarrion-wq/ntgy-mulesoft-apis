---
api: "ntgy-fuv-s-postgresql"
sidebar_position: 28
sidebar_label: "POST /consultaGenerica"
description: "Ejecuta consultas SQL genéricas de lectura en PostgreSQL para análisis y reportes ad-hoc del sistema FUV"
keywords: [consulta generica, postgresql, sql query, system api, fuv, dynamic query, ad-hoc reporting]
tags: [Consultas, PostgreSQL, System API, FUV, SQL]
sql_metadata:
  data_sources:
    - database: "FUV_DB"
      schema: "public"
      type: "PostgreSQL"
      tables:
        - name: "DYNAMIC"
          operations: ["READ"]
          description: "Tablas dinámicas según query proporcionada por el usuario"
---

# POST /consultaGenerica

## 📌 Descripción General

### Propósito del Endpoint

El endpoint `POST /consultaGenerica` permite ejecutar consultas SQL genéricas de lectura (SELECT) en la base de datos PostgreSQL del sistema FUV (Facturación Única Virtual). Este endpoint proporciona flexibilidad para realizar consultas ad-hoc sin necesidad de crear endpoints específicos para cada caso de uso.

El endpoint soporta:
- **Consultas Parametrizadas**: Ejecución de queries SQL con parámetros dinámicos para prevenir SQL injection
- **Múltiples Tablas**: Capacidad de consultar cualquier tabla accesible en el schema público
- **Resultados Dinámicos**: Estructura de respuesta adaptable según los campos seleccionados en la query
- **Análisis Ad-Hoc**: Soporte para reportes y análisis de datos personalizados

### Tipo de API

**Capa:** System API (API-Led Connectivity)

**Función:** Integración directa con PostgreSQL para consultas genéricas de lectura en el sistema FUV.

### Patrón de Integración

**Tipo:** Request-Reply Síncrono con SQL Query Dinámica

**Descripción:** El endpoint recibe una query SQL y parámetros opcionales, valida que sea una operación de lectura, construye una query parametrizada segura, la ejecuta en PostgreSQL y retorna los resultados en formato JSON con estructura dinámica.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    Client[Cliente/Consumidor<br/>Process API o Sistema]
    S[System API<br/>ntgy-fuv-s-postgresql<br/>POST /consultaGenerica]
    PG[(PostgreSQL<br/>FUV Database<br/>Schema: public<br/>Tablas: Dinámicas)]
    
    Client -->|Request JSON<br/>query + parameters| S
    S -->|SELECT Query<br/>Parametrizada| PG
    PG -->|Result Set| S
    S -->|Response JSON<br/>results array| Client
    
    style S fill:#e1f5ff,stroke:#01579b,stroke-width:3px
    style PG fill:#fff3e0,stroke:#e65100,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Análisis y Reportes Ad-Hoc**
   - Equipos de análisis necesitan consultar datos sin esperar desarrollo de endpoints específicos
   - Permite exploración rápida de datos para toma de decisiones
   - Facilita generación de reportes personalizados

2. **Validación de Datos**
   - Verificación de integridad de datos entre sistemas
   - Validación de migraciones de datos
   - Auditoría de información almacenada

3. **Extracción de Datos para Integraciones**
   - Obtención de datos específicos para integraciones temporales
   - Soporte para procesos de ETL personalizados
   - Exportación de datos para análisis externos

4. **Troubleshooting y Soporte**
   - Equipos de soporte pueden consultar datos para resolver incidencias
   - Verificación de estados de procesos
   - Diagnóstico de problemas de datos

### Contexto Empresarial

- **Dominio:** Gestión de Datos - Facturación Única Virtual
- **Sector:** Energía - Utilities
- **División:** Naturgy España - FUV Platform
- **Proceso de Negocio:** Acceso flexible a datos para análisis, reportes y troubleshooting
- **Base de Datos:** PostgreSQL (FUV Database)

### Stakeholders

- **Analistas de Datos**: Necesitan consultar datos para análisis y reportes sin depender de desarrollo
- **Equipos de Soporte**: Requieren acceso a datos para resolver incidencias y verificar estados
- **Desarrolladores de Integraciones**: Necesitan extraer datos específicos para integraciones temporales
- **Equipos de QA**: Requieren validar datos durante pruebas y migraciones

### Valor de Negocio

- **Agilidad**: Reduce tiempo de respuesta para consultas de datos de días a minutos
- **Flexibilidad**: Permite análisis personalizados sin necesidad de desarrollo adicional
- **Eficiencia**: Elimina necesidad de crear endpoints específicos para cada consulta
- **Autonomía**: Empodera a equipos técnicos para obtener datos sin dependencias

### Consumidores Potenciales

Este endpoint puede ser consumido por:
- Process APIs que requieren consultas específicas no cubiertas por otros endpoints
- Herramientas de análisis y BI (Business Intelligence)
- Scripts de automatización y ETL
- Sistemas de monitoreo y auditoría
- Equipos de soporte técnico con acceso controlado

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/consultaGenerica` |
| **Development** | `https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io` | `https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/consultaGenerica` |
| **UAT** | `https://ntgy-fuv-s-postgresql-uat.us-e2.cloudhub.io` | `https://ntgy-fuv-s-postgresql-uat.us-e2.cloudhub.io/api/consultaGenerica` |
| **UAT01** | `https://ntgy-fuv-s-postgresql-uat01.us-e2.cloudhub.io` | `https://ntgy-fuv-s-postgresql-uat01.us-e2.cloudhub.io/api/consultaGenerica` |
| **Production** | `https://ntgy-fuv-s-postgresql-pro.us-e2.cloudhub.io` | `https://ntgy-fuv-s-postgresql-pro.us-e2.cloudhub.io/api/consultaGenerica` |

### Método HTTP

```
POST /api/consultaGenerica
```

### Parámetros de Entrada

#### Headers HTTP

| Header | Tipo | Requerido | Descripción | Ejemplo |
|--------|------|-----------|-------------|---------|
| `Content-Type` | string | ✅ Sí | Tipo de contenido del request | `application/json` |
| `Accept` | string | ❌ No | Tipo de contenido esperado en response | `application/json` |
| `client_id` | string | ✅ Sí | Client ID de Anypoint Platform | `abc123...` |
| `client_secret` | string | ✅ Sí | Client Secret de Anypoint Platform | `xyz789...` |

#### Body Parameters

**Formato:** `text/plain`

El body del request debe contener la query SQL completa como texto plano.

**IMPORTANTE:** 
- No se soportan parámetros nombrados (`:param`)
- Los valores deben incluirse directamente en la query
- La query se ejecuta directamente sin validaciones previas

**Ejemplo de Request Body:**

```sql
SELECT id, name, email, created_at 
FROM customers 
WHERE status = 'active' 
ORDER BY created_at DESC 
LIMIT 100
```

### Respuestas

#### Respuesta Exitosa (200 OK)

```json
{
  "results": [
    {
      "id": 1,
      "name": "Cliente Ejemplo 1",
      "email": "cliente1@example.com",
      "created_at": "2026-01-15T10:30:00Z"
    },
    {
      "id": 2,
      "name": "Cliente Ejemplo 2",
      "email": "cliente2@example.com",
      "created_at": "2026-01-14T15:45:00Z"
    }
  ],
  "rowCount": 2,
  "executionTime": 45
}
```

**Campos de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result` | object | Información del resultado de la operación |
| `result.codResult` | string | Código de resultado ("0000" = éxito) |
| `result.msgResult` | string | Mensaje descriptivo con número de registros encontrados |
| `contratos` | array | Array de objetos con los resultados. Estructura dinámica según campos seleccionados. Solo presente cuando hay resultados |

#### Respuesta de Error de Validación (400 Bad Request)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Query inválida: solo se permiten operaciones SELECT"
  }
}
```

#### Respuesta de Error de SQL (400 Bad Request)

```json
{
  "result": {
    "codResult": "0002",
    "msgResult": "Error de sintaxis SQL: tabla 'invalid_table' no existe"
  }
}
```

#### Respuesta de Error Técnico (500 Internal Server Error)

```json
{
  "result": {
    "codResult": "0003",
    "msgResult": "Error interno del servidor al ejecutar query"
  }
}
```

### Validaciones

El endpoint NO implementa validaciones previas a la ejecución de la query:

1. **Ejecución Directa:**
   - La query SQL se ejecuta directamente tal como se recibe
   - No hay validación de tipo de operación (SELECT, INSERT, UPDATE, etc.)
   - No hay sanitización de la query
   - Los valores deben incluirse directamente en la query (no hay prepared statements)

2. **Validación de Recursos:**
   - Timeout de 30 segundos para ejecución de query
   - Fetch size: 5000 registros por batch
   - Límite de resultados debe especificarse en la query con LIMIT

3. **Seguridad:**
   - La seguridad se gestiona mediante políticas de API Manager y control de acceso
   - Es responsabilidad del cliente validar las queries antes de enviarlas

### Autenticación y Autorización

- **Tipo:** Client ID & Client Secret (Anypoint Platform)
- **Autorización PostgreSQL:** El usuario de base de datos debe tener permisos de:
  - SELECT en schema público
  - Acceso a tablas consultadas
  - Permisos de lectura en vistas (si aplica)

### Timeouts

| Tipo | Valor | Descripción |
|------|-------|-------------|
| **Query Timeout** | 30 segundos | Tiempo máximo de ejecución de la query SQL |
| **Connection Timeout** | 10 segundos | Tiempo máximo para establecer conexión con PostgreSQL |
| **Request Timeout** | 60 segundos | Tiempo máximo total del request HTTP |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request]) --> LogStart[Log: Inicio consultaGenerica]
    LogStart --> SetVar[Set Variable: query<br/>payload → vars.query]
    SetVar --> LogQuery[Log: Query recibida]
    LogQuery --> DBSelect[db:select<br/>Ejecutar query directamente<br/>fetchSize: 5000]
    DBSelect --> Choice{Resultados?<br/>sizeOf payload > 0}
    Choice -->|Sí| Transform1[Transform: DB to JSON<br/>Crear array contratos<br/>Agregar result.codResult]
    Choice -->|No| Transform2[Transform: Sin Datos<br/>Solo result object]
    Transform1 --> LogEnd[Log: Finished consultaGenerica]
    Transform2 --> LogEnd
    LogEnd --> End([HTTP 200 Response])
    
    DBSelect -->|SQL Error| Error1[Error 500<br/>SQL Exception]
    DBSelect -->|Timeout| Error2[Error 500<br/>Timeout]
    
    style Start fill:#e8f5e9,stroke:#2e7d32
    style DBSelect fill:#fff3e0,stroke:#e65100,stroke-width:3px
    style Choice fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style End fill:#e8f5e9,stroke:#2e7d32
    style Error1 fill:#ffebee,stroke:#c62828
    style Error2 fill:#ffebee,stroke:#c62828
```

### Componentes del Flow

#### 1. Recepción y Almacenamiento de Query

**Ubicación:** Flow principal `s-postgresql-consultaGenerica`

**Lógica:**
```xml
<set-variable value="#[payload]" variableName="query" />
<logger level="INFO" message="query: #[vars.query]" />
```

El payload (texto plano) se guarda directamente en la variable `query`.

#### 2. Ejecución de Query

**Componente:** Database Connector - Select Operation

**Configuración:**
```xml
<db:select config-ref="Database_Config" fetchSize="5000">
    <db:sql><![CDATA[#[vars.query]]]></db:sql>
</db:select>
```

**IMPORTANTE:** La query se ejecuta directamente sin prepared statements.

#### 3. Transformación de Resultados

**DataWeave Transformation (cuando hay resultados):**
```dataweave
%dw 2.0
output application/json
---
{
  result: {
    codResult: "0000",
    msgResult: "No hay error. Registros encontrados: " ++ (sizeOf(payload)) as String
  },
  contratos: payload map (record, index) -> 
    record mapObject using (key = $$, value = $) {
      ($$): $ default ""
    }
}
```

**DataWeave Transformation (sin resultados):**
```dataweave
%dw 2.0
output application/json
---
{
  result: {
    codResult: "0000",
    msgResult: "No se han encontrado datos"
  }
}
```

### Diagrama de Secuencia

```mermaid
sequenceDiagram
    participant Client as Cliente
    participant API as System API<br/>consultaGenerica
    participant DB as PostgreSQL<br/>FUV Database
    
    Client->>API: POST /consultaGenerica<br/>text/plain: SQL query
    API->>API: Log: STARTING consultaGenerica
    API->>API: Set Variable: query = payload
    API->>API: Log: query recibida
    API->>DB: db:select<br/>Ejecutar vars.query directamente<br/>(fetchSize: 5000)
    
    alt Query exitosa con resultados (sizeOf > 0)
        DB-->>API: Result Set (registros)
        API->>API: Transform: DB to JSON<br/>mapObject cada registro<br/>Crear array contratos
        API->>API: result.msgResult = "Registros encontrados: N"
        API->>API: Log: FINISHED consultaGenerica
        API-->>Client: 200 OK<br/>{result, contratos}
    else Query exitosa sin resultados (sizeOf = 0)
        DB-->>API: Result Set vacío
        API->>API: Transform: Sin Datos
        API->>API: result.msgResult = "No se han encontrado datos"
        API->>API: Log: FINISHED consultaGenerica
        API-->>Client: 200 OK<br/>{result}
    else Error SQL
        DB-->>API: SQL Exception
        API->>API: Log error
        API-->>Client: 500 Internal Error<br/>{result: {codResult: "0002", msgResult: "ERROR: ..."}}
    end
    
    Note over API,DB: No hay validaciones previas<br/>La query se ejecuta directamente
```

### Transformación de Datos

```mermaid
graph LR
    subgraph Input
        I1[query: string]
        I2[parameters: object]
    end
    
    subgraph Processing
        P1[Validación]
        P2[Prepared Statement]
        P3[DB Execution]
    end
    
    subgraph Output
        O1[results: array]
        O2[rowCount: int]
        O3[executionTime: int]
    end
    
    I1 --> P1
    I2 --> P1
    P1 --> P2
    P2 --> P3
    P3 --> O1
    P3 --> O2
    P3 --> O3
    
    style P3 fill:#fff3e0,stroke:#e65100
```

### Configuración de Base de Datos

**Database Connector Config:**
```xml
<db:config name="Database_Config">
    <db:generic-connection 
        url="${dbFUV.url}" 
        driverClassName="org.postgresql.Driver"
        user="${dbFUV.user}"
        password="${secure::dbFUV.password}">
        <db:pooling-profile 
            maxPoolSize="20" 
            minPoolSize="5" 
            acquireIncrement="1" 
            maxWait="10000" 
            maxWaitUnit="SECONDS"/>
    </db:generic-connection>
</db:config>
```

**Propiedades por Ambiente:**
- `dbFUV.url`: URL de conexión PostgreSQL
- `dbFUV.user`: Usuario de base de datos
- `dbFUV.password`: Contraseña encriptada

### Manejo de Errores

El endpoint solo maneja errores generados por la base de datos durante la ejecución:

| Tipo de Error | Código | Mensaje | HTTP Status |
|---------------|--------|---------|-------------|
| Error SQL | 0002 | "ERROR: [detalle del error SQL]" | 500 |
| Tabla no existe | 0002 | "ERROR: relation \"[nombre]\" does not exist" | 500 |
| Sintaxis SQL | 0002 | "ERROR: syntax error at or near \"[token]\"" | 500 |
| Timeout query | 0003 | "Timeout al ejecutar query" | 500 |
| Error conexión DB | 0003 | "Error de conexión a base de datos" | 500 |

**Nota:** No hay validaciones previas, por lo que cualquier error de query se detecta durante la ejecución en PostgreSQL.

### Logging

**Puntos de Log:**
1. Inicio de ejecución con query recibida (sanitizada)
2. Resultado de validaciones
3. Query a ejecutar (con parámetros sanitizados)
4. Resultado de ejecución (rowCount, executionTime)
5. Errores con stack trace completo

**Nivel de Log:**
- INFO: Inicio, fin exitoso, métricas
- DEBUG: Detalles de query y parámetros
- ERROR: Excepciones y errores

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta Simple de Clientes

**Request:**
```bash
curl -X POST "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/consultaGenerica" \
  -H "Content-Type: text/plain" \
  -H "client_id: your-client-id" \
  -H "client_secret: your-client-secret" \
  -d "SELECT * FROM contract_control WHERE estado = 'ACTIVO' LIMIT 10"
```

**Response (200 OK):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error. Registros encontrados: 2"
  },
  "contratos": [
    {
      "id": 1,
      "id_pedido": "ORD-001",
      "estado": "ACTIVO",
      "fecha_creacion": "2026-01-15"
    },
    {
      "id": 2,
      "id_pedido": "ORD-002",
      "estado": "ACTIVO",
      "fecha_creacion": "2026-01-14"
    }
  ]
}
```

---

### Ejemplo 2: Consulta con Múltiples Tablas

**Request:**
```bash
curl -X POST "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/consultaGenerica" \
  -H "Content-Type: text/plain" \
  -H "client_id: your-client-id" \
  -H "client_secret: your-client-secret" \
  -d "SELECT cc.id_pedido, cc.estado, dc.document_name, dc.status 
      FROM contract_control cc 
      LEFT JOIN documentation_control dc ON cc.id_pedido = dc.order_id 
      WHERE cc.estado = 'PROCESADO' 
      LIMIT 5"
```

**Response (200 OK):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error. Registros encontrados: 3"
  },
  "contratos": [
    {
      "id_pedido": "ORD-001",
      "estado": "PROCESADO",
      "document_name": "contrato_001.pdf",
      "status": "ENVIADO"
    },
    {
      "id_pedido": "ORD-002",
      "estado": "PROCESADO",
      "document_name": "contrato_002.pdf",
      "status": "ENVIADO"
    },
    {
      "id_pedido": "ORD-003",
      "estado": "PROCESADO",
      "document_name": null,
      "status": null
    }
  ]
}
```

---

### Ejemplo 3: Consulta de Agregación

**Request:**
```bash
curl -X POST "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/consultaGenerica" \
  -H "Content-Type: text/plain" \
  -H "client_id: your-client-id" \
  -H "client_secret: your-client-secret" \
  -d "SELECT estado, COUNT(*) as total, 
      MIN(fecha_creacion) as primera_fecha, 
      MAX(fecha_creacion) as ultima_fecha 
      FROM contract_control 
      GROUP BY estado 
      ORDER BY total DESC"
```

**Response (200 OK):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error. Registros encontrados: 4"
  },
  "contratos": [
    {
      "estado": "PROCESADO",
      "total": 150,
      "primera_fecha": "2026-01-01",
      "ultima_fecha": "2026-01-15"
    },
    {
      "estado": "ACTIVO",
      "total": 45,
      "primera_fecha": "2026-01-10",
      "ultima_fecha": "2026-01-15"
    },
    {
      "estado": "PENDIENTE",
      "total": 23,
      "primera_fecha": "2026-01-12",
      "ultima_fecha": "2026-01-15"
    },
    {
      "estado": "ERROR",
      "total": 5,
      "primera_fecha": "2026-01-14",
      "ultima_fecha": "2026-01-15"
    }
  ]
}
```

---

### Ejemplo 4: Sin Resultados

**Request:**
```bash
curl -X POST "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/consultaGenerica" \
  -H "Content-Type: text/plain" \
  -H "client_id: your-client-id" \
  -H "client_secret: your-client-secret" \
  -d "SELECT * FROM contract_control WHERE estado = 'NO_EXISTE'"
```

**Response (200 OK):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No se han encontrado datos"
  }
}
```

---

### Ejemplo 5: Error - Tabla No Existe

**Request:**
```bash
curl -X POST "https://ntgy-fuv-s-postgresql-dev.us-e2.cloudhub.io/api/consultaGenerica" \
  -H "Content-Type: text/plain" \
  -H "client_id: your-client-id" \
  -H "client_secret: your-client-secret" \
  -d "SELECT * FROM tabla_inexistente"
```

**Response (500 Internal Server Error):**
```json
{
  "result": {
    "codResult": "0002",
    "msgResult": "ERROR: relation \"tabla_inexistente\" does not exist"
  }
}
```
