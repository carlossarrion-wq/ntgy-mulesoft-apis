---
api: "ntgy-fua-s-ac-fua"
sidebar_position: 7
description: Consulta histórica de consumos de gas y electricidad por contrato desde Oracle Database con agregación por tarifas y periodos
keywords:
  - consumptions
  - consumos
  - gas
  - electricidad
  - facturas
  - tarifas
  - histórico
  - Oracle Database
  - System API
  - MUL-COM-FUA-S-AC-FUA
tags:
  - System API
  - Oracle Database
  - Consulta
  - Agregación
sql_metadata:
  data_sources:
    - database: "Oracle"
      schema: "OVPRO_DELTA"
      type: "Oracle Database 19c"
      tables:
        - name: "v_fua_invoices"
          operations: ["READ"]
          description: "Vista de facturas con datos de consumo por tarifa y periodo"
          columns:
            - SAP_CONS_BELZART_CHZ1 (código de tarifa)
            - SAP_CONS_SUM_ABRMENGE_CHZ1 (cantidad consumida)
            - INV_INVOICE_DATE (fecha de factura)
            - INV_SAP_VKONT (número de contrato)
---

# GET /consumptions

## 📌 Descripción General

Endpoint de System API que consulta el histórico de consumos de gas o electricidad para uno o múltiples contratos. Realiza una consulta dinámica a la vista `v_fua_invoices` en Oracle Database, procesa los datos según el tipo de producto (gas/electricidad) y agrega los consumos por mes y tarifa.

**Tipo de API:** System API  
**Protocolo:** REST/HTTP  
**Método:** GET  
**Backend:** Oracle Database 19c  
**Autenticación:** Requerida (según configuración del API)

### Características Principales

- ✅ Consulta SQL dinámica construida en tiempo de ejecución
- ✅ Soporte para múltiples contratos simultáneos
- ✅ Procesamiento diferenciado por tipo de producto (gas vs electricidad)
- ✅ Agregación de consumos por 16+ tipos de tarifas
- ✅ Cálculo automático de periodos mensuales entre fechas
- ✅ Respuesta estructurada con desglose por periodos tarifarios
- ✅ Validación de contratos activos
- ✅ Manejo de errores personalizado

## 💼 Propósito del Negocio

### Objetivo

Proporcionar información histórica detallada de consumos energéticos (gas y electricidad) para análisis, facturación y visualización de patrones de consumo. Este endpoint es fundamental para:

- **Análisis de Consumo:** Visualizar patrones históricos de consumo por mes
- **Facturación:** Obtener datos de consumo para cálculos de facturación
- **Comparativas:** Comparar consumos entre periodos
- **Auditoría:** Verificar consumos facturados vs reales
- **Optimización Energética:** Identificar oportunidades de ahorro

### Casos de Uso

1. **Dashboard de Consumos**
   - Usuario accede a su área personal
   - Sistema muestra gráfico de consumos históricos
   - Datos agregados por mes y tipo de energía

2. **Análisis Comparativo**
   - Cliente solicita comparativa anual
   - Sistema obtiene consumos de 12 meses
   - Presenta evolución y tendencias

3. **Validación de Factura**
   - Cliente cuestiona factura
   - Agente consulta consumos del periodo
   - Verifica datos contra factura emitida

4. **Múltiples Contratos**
   - Cliente con varios puntos de suministro
   - Sistema consulta todos los contratos
   - Agrega consumos totales

### Valor de Negocio

- **Transparencia:** Cliente visualiza su consumo real
- **Confianza:** Datos verificables contra facturas
- **Eficiencia:** Automatización de consultas de consumo
- **Análisis:** Base para recomendaciones de ahorro
- **Soporte:** Herramienta para atención al cliente

## 🔧 Especificación Técnica

### Endpoint

```
GET /consumptions?contractAccountId={ids}&startDate={fecha}&endDate={fecha}&productType={tipo}&contractId={id}
```

### Parámetros de Consulta

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `contractAccountId` | string | Sí | ID(s) de cuenta de contrato. Múltiples valores separados por coma | `123456,789012` |
| `startDate` | string | Sí | Fecha inicio del periodo (formato: YYYY-MM-DD) | `2024-01-01` |
| `endDate` | string | Sí | Fecha fin del periodo (formato: YYYY-MM-DD) | `2024-12-31` |
| `productType` | string | Sí | Tipo de producto: `gas` o `luz` (electricidad) | `gas` |
| `contractId` | string | Sí | ID del contrato principal | `CON123456` |

### Headers Requeridos

```http
Content-Type: application/json
Authorization: Bearer {token}
```

### Respuesta Exitosa (200 OK)

#### Para Gas:

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "consumptions": {
    "contractId": "CON123456",
    "productTypeFamily": "gas",
    "consumptionsValues": [
      {
        "date": "01-01-2024",
        "valueTotal": 245.50
      },
      {
        "date": "01-02-2024",
        "valueTotal": 312.75
      }
    ]
  }
}
```

#### Para Electricidad:

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "consumptions": {
    "contractId": "CON123456",
    "productTypeFamily": "luz",
    "consumptionsValues": [
      {
        "date": "01-01-2024",
        "valuePoint": 45.30,
        "valuePlain": 78.20,
        "valueValley": 102.50,
        "valueP4": 12.30,
        "valueP5": 8.40,
        "valueP6": 5.20,
        "valueTotal": 251.90
      },
      {
        "date": "01-02-2024",
        "valuePoint": 52.10,
        "valuePlain": 85.40,
        "valueValley": 115.30,
        "valueP4": 14.20,
        "valueP5": 9.80,
        "valueP6": 6.10,
        "valueTotal": 282.90
      }
    ]
  }
}
```

### Estructura de Respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result` | object | Objeto contenedor del resultado de la operación |
| `result.codResult` | string | Código de resultado (`0000` = éxito, `-0001` = error) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |
| `consumptions` | object | Objeto contenedor de los datos de consumo |
| `consumptions.contractId` | string | ID del contrato consultado |
| `consumptions.productTypeFamily` | string | Tipo de producto (`gas` o `luz`) |
| `consumptions.consumptionsValues` | array | Array de consumos por mes |

#### Estructura de consumptionsValues (Gas):

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `date` | string | Fecha del periodo (formato: DD-MM-YYYY) |
| `valueTotal` | number | Consumo total del mes en kWh |

#### Estructura de consumptionsValues (Electricidad):

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `date` | string | Fecha del periodo (formato: DD-MM-YYYY) |
| `valuePoint` | number | Consumo en periodo punta (P1) en kWh |
| `valuePlain` | number | Consumo en periodo llano (P2) en kWh |
| `valueValley` | number | Consumo en periodo valle (P3) en kWh |
| `valueP4` | number | Consumo en periodo P4 en kWh |
| `valueP5` | number | Consumo en periodo P5 en kWh |
| `valueP6` | number | Consumo en periodo P6 en kWh |
| `valueTotal` | number | Consumo total del mes en kWh |

### ⚠️ Nota sobre Discrepancia con RAML

**Importante:** Existe una discrepancia entre la especificación RAML y la implementación real del código.

#### Campos Especificados en RAML (NO Implementados):

El RAML especifica los siguientes campos adicionales que **NO están presentes** en la respuesta real generada por el código:

| Campo | Tipo (RAML) | Estado | Descripción |
|-------|-------------|--------|-------------|
| `consumed` | number | ❌ No implementado | Cantidad consumida (no generado por el código) |
| `aported` | number | ❌ No implementado | Cantidad aportada (no generado por el código) |

#### Diferencias de Formato:

| Aspecto | RAML | Implementación Real |
|---------|------|---------------------|
| **Tipo de valores** | string (`"123"`) | number (`123.45`) |
| **Formato de fecha** | `"2021-01-01"` (YYYY-MM-DD) | `"01-01-2024"` (DD-MM-YYYY) |

#### Estructura RAML (Referencia):

```json
{
  "date": "2021-01-01",
  "consumed": 1,
  "valuePoint": "123",
  "valuePlain": "123",
  "valueValley": "123",
  "valueP4": "123",
  "valueP5": "123",
  "valueP6": "123",
  "aported": 1,
  "valueTotal": "2345"
}
```

#### Estructura Real (Implementación):

**Gas:**
```json
{
  "date": "01-01-2024",
  "valueTotal": 245.50
}
```

**Electricidad:**
```json
{
  "date": "01-01-2024",
  "valuePoint": 45.30,
  "valuePlain": 78.20,
  "valueValley": 102.50,
  "valueP4": 12.30,
  "valueP5": 8.40,
  "valueP6": 5.20,
  "valueTotal": 251.90
}
```

**Conclusión:** La documentación refleja la **implementación real** del código (fuente de verdad), no la especificación RAML que está desactualizada.

### Códigos de Respuesta HTTP

| Código | Descripción | Escenario |
|--------|-------------|-----------|
| 200 | OK | Consulta ejecutada exitosamente |
| 400 | Bad Request | Parámetros faltantes o inválidos |
| 401 | Unauthorized | Token de autenticación inválido o ausente |
| 500 | Internal Server Error | Error en la consulta a base de datos |
| 504 | Gateway Timeout | Timeout en la consulta a Oracle |

### Códigos de Resultado de Negocio

| Código | Mensaje | Descripción |
|--------|---------|-------------|
| `0000` | No hay error | Operación exitosa |
| `-0001` | Contrato no encontrado | El contractAccountId no existe o está inactivo |

### Tarifas Procesadas

#### Gas (2 tarifas):
- **ZGFCON** - Consumo de gas fijo
- **ZGFCO1** - Consumo de gas variable

#### Electricidad (16 tarifas en 6 periodos):

**Periodo 1 (Punta):**
- ZEFCON, ZEC20D, ZEFCPT, ZEIEP1, ZEFACT, ZEFAP1

**Periodo 2 (Llano):**
- ZEC20N, ZEFCLL, ZEIEP2, ZEFAP2

**Periodo 3 (Valle):**
- ZEFCVA, ZEIEP3, ZEFAP3

**Periodo 4:**
- ZEFCSV, ZEIEP4, ZEFAP4

**Periodo 5:**
- ZEIEP5, ZEFAP5

**Periodo 6:**
- ZEIEP6, ZEFAP6

## 🏗️ Implementación Técnica

### Arquitectura del Flujo

```mermaid
sequenceDiagram
    participant Client as Cliente
    participant API as MuleSoft API
    participant Logger as Logger
    participant Validator as Validador
    participant DW as DataWeave
    participant DB as Oracle Database
    participant Aggregator as Agregador

    Client->>API: GET /consumptions?params
    API->>Logger: Log entrada
    Logger-->>API: Logged
    
    API->>Validator: Validar contractAccountId
    alt Contrato válido
        Validator-->>API: ✓ Válido
        API->>DW: Calcular array de meses
        DW-->>API: Array de meses
        API->>DW: Construir query SQL dinámica
        DW-->>API: Query construida
        API->>DB: SELECT consumos por tarifa
        DB-->>API: Datos de facturas
        API->>DW: Transformar a lista de consumos
        DW-->>API: Lista transformada
        
        loop Para cada mes
            API->>Aggregator: Procesar mes
            alt productType = gas
                Aggregator->>Aggregator: Filtrar tarifas gas
                Aggregator->>Aggregator: Sumar consumos
                Aggregator-->>API: {date, valueTotal}
            else productType = luz
                Aggregator->>Aggregator: Filtrar por 6 periodos
                Aggregator->>Aggregator: Sumar por periodo
                Aggregator-->>API: {date, valueP1-P6, valueTotal}
            end
        end
        
        API->>DW: Construir respuesta final
        DW-->>API: Response estructurado
        API->>Logger: Log salida
        Logger-->>API: Logged
        API-->>Client: 200 OK + JSON Response
    else Contrato inválido
        Validator-->>API: ✗ Inválido
        API->>API: Raise error APP:ERRORCONTRATO
        API->>DW: Transform error response
        DW-->>API: Error response
        API-->>Client: 200 OK + Error JSON
    end
```

### Diagrama de Flujo de Decisión

```mermaid
flowchart TD
    A[Request con parámetros] --> B{Validar<br/>contractAccountId}
    B -->|Vacío o null| C[Raise Error<br/>APP:ERRORCONTRATO]
    B -->|Válido| D[Calcular array<br/>de meses]
    
    D --> E[Construir query<br/>SQL dinámica]
    E --> F[Ejecutar SELECT<br/>en Oracle]
    F --> G[Transformar<br/>resultados]
    
    G --> H[Iniciar loop<br/>por cada mes]
    H --> I{productType?}
    
    I -->|gas| J[Filtrar tarifas<br/>ZGFCON, ZGFCO1]
    I -->|luz| K[Filtrar tarifas<br/>por 6 periodos]
    
    J --> L[Sumar consumos<br/>del mes]
    K --> M[Sumar por periodo<br/>P1-P6]
    
    L --> N{¿Más meses?}
    M --> N
    
    N -->|Sí| H
    N -->|No| O[Construir<br/>respuesta final]
    
    O --> P[Log salida]
    P --> Q[Response 200]
    
    C --> R[Transform<br/>error message]
    R --> S[Response 200<br/>con error]
    
    style F fill:#ffcccc
    style G fill:#ccffcc
    style I fill:#ffffcc
    style O fill:#ccccff
```

### Diagrama de Componentes

```mermaid
graph TB
    subgraph "MuleSoft Runtime"
        A[APIKit Router] --> B[get:\consumptions Flow]
        B --> C[consumptionsFlow]
        
        subgraph "Validación"
            C --> D[Logger Entrada]
            D --> E[Set queryParams]
            E --> F[Split contractAccountId]
            F --> G{Choice: Validar}
            G -->|Válido| H[Continue]
            G -->|Inválido| I[Raise Error]
        end
        
        subgraph "Preparación"
            H --> J[DW: Calcular meses]
            J --> K[DW: Construir query]
        end
        
        subgraph "Consulta"
            K --> L[DB Select]
            L --> M[DW: Transform lista]
        end
        
        subgraph "Agregación"
            M --> N[For Each mes]
            N --> O{Choice: Tipo}
            O -->|Gas| P[DW: Agregar gas]
            O -->|Luz| Q[DW: Agregar luz]
            P --> R[Acumular resultado]
            Q --> R
        end
        
        subgraph "Respuesta"
            R --> S[DW: Response final]
            S --> T[Logger Salida]
        end
        
        subgraph "Error Handler"
            I --> U[DW: Error response]
        end
    end
    
    subgraph "Oracle Database"
        V[(v_fua_invoices)]
    end
    
    L -.->|Query SQL| V
    V -.->|Resultados| L
    
    style L fill:#ffcccc
    style J fill:#ccffcc
    style K fill:#ccffcc
    style M fill:#ccffcc
    style P fill:#ffffcc
    style Q fill:#ffffcc
    style S fill:#ccccff
```

### Diagrama de Transformación de Datos

```mermaid
flowchart LR
    subgraph "Input"
        A[Query Params]
        A1[contractAccountId]
        A2[startDate]
        A3[endDate]
        A4[productType]
        A5[contractId]
    end
    
    subgraph "Procesamiento"
        B[Calcular Meses]
        C[Construir Query]
        D[Ejecutar Query]
        E[Datos Oracle]
    end
    
    subgraph "Transformación 1"
        F[Lista Consumos]
        F1[tarifa]
        F2[valor]
        F3[fecha MM-YYYY]
    end
    
    subgraph "Agregación"
        G{Por Tipo}
        G1[Gas: 2 tarifas]
        G2[Luz: 16 tarifas]
    end
    
    subgraph "Output Gas"
        H[Array Mensual]
        H1[date]
        H2[valueTotal]
    end
    
    subgraph "Output Luz"
        I[Array Mensual]
        I1[date]
        I2[valuePoint P1]
        I3[valuePlain P2]
        I4[valueValley P3]
        I5[valueP4]
        I6[valueP5]
        I7[valueP6]
        I8[valueTotal]
    end
    
    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G -->|gas| G1
    G -->|luz| G2
    G1 --> H
    G2 --> I
    
    style B fill:#e1f5ff
    style C fill:#e1f5ff
    style F fill:#ffe1e1
    style G fill:#ffffcc
    style H fill:#e1ffe1
    style I fill:#e1ffe1
```

### Configuración de Base de Datos

**Archivo:** `src/main/resources/properties/ntgy-fua-s-ac-fua-global-config.yaml`

```yaml
dbAC:
  queryTimeout: 15000  # Timeout en milisegundos (15 segundos)
  user: "OVPRO_DELTA"
  host1: "oc-pro-exa-04-clu-02-hdfdi-scan.prodb03.ocpro.oraclevcn.com"
  host2: "oc-pro-exa-03-clu-02-dfhl9-scan.prodb03.ocpro.oraclevcn.com"
  port: "1521"
  serviceName: "s_areapriv_online_pro.prodb03.ocpro.oraclevcn.com"
```

### Implementación del Flow

**Archivo:** `src/main/mule/implementation/consumptionsFlow.xml`

```xml
<flow name="consumptionsFlow">
    <!-- Logger de entrada -->
    <logger level="DEBUG" 
            doc:name="Logger entrada"
            message="s-ac-consumptions.logEntrada ## payload: #[payload]"/>
    
    <!-- Guardar parámetros de query -->
    <set-variable 
        value="#[attributes.queryParams]" 
        doc:name="Set Variable"
        variableName="queryParams"/>
    
    <!-- Dividir contractAccountId por comas -->
    <set-variable 
        value='#[attributes.queryParams.contractAccountId splitBy(",")]'
        doc:name="listaActivos"
        variableName="listaActivos"/>
    
    <!-- Validar que existe contractAccountId -->
    <choice doc:name="Choice">
        <when expression='#[vars.listaActivos[0] != null and vars.listaActivos[0] != ""]'>
            <logger level="INFO" 
                    doc:name="vkon activo encontrado"
                    message="vkon activo encontrado"/>
        </when>
        <otherwise>
            <logger level="INFO" 
                    doc:name="vkon no encontrado"
                    message="vkon no encontrado"/>
            <raise-error doc:name="Raise error" 
                         type="APP:ERRORCONTRATO"/>
        </otherwise>
    </choice>
    
    <!-- Calcular array de meses entre fechas -->
    <ee:transform doc:name="Transform Message">
        <ee:variables>
            <ee:set-variable variableName="monthsArray"><![CDATA[%dw 2.0
import * from dw::core::Periods
import dw::core::Periods
output application/json

var in = vars.queryParams.startDate splitBy (/-/)
var out = vars.queryParams.endDate splitBy (/-/)
var auxIn = in[0]++"-"++ in[1] ++ "-01"
var auxOut = out[0] ++ "-"++ out[1] ++ "-01"
var meses = Periods::between((auxOut as String) as Date, (auxIn as String as Date))

fun prepareList(list:Array, maxSize: Number) = 
    if(sizeOf(list) >= maxSize)
        list
    else (
        prepareList(
            list ++ [(auxOut as String as Date - period({months:(sizeOf(list))})) 
                     as String {format: "MM-uuuu"}],
            maxSize
        )
    )
---
prepareList([], meses.months + (meses.years * 12) + 1)
]]></ee:set-variable>
            
            <ee:set-variable variableName="consumptions"><![CDATA[%dw 2.0
output application/java
var consumptions = []
---
consumptions]]></ee:set-variable>
        </ee:variables>
    </ee:transform>
    
    <!-- Construir query SQL dinámica -->
    <ee:transform doc:name="Transform Message">
        <ee:variables>
            <ee:set-variable variableName="query"><![CDATA[%dw 2.0
output application/json
var query = "SELECT SAP_CONS_BELZART_CHZ1, SAP_CONS_SUM_ABRMENGE_CHZ1, INV_INVOICE_DATE 
             FROM v_fua_invoices 
             WHERE INV_SAP_VKONT IN (" ++ vars.queryParams.contractAccountId ++ ")"
var dates = if (!isEmpty(vars.queryParams.startDate) and !isEmpty(vars.queryParams.endDate)) 
            " AND INV_INVOICE_DATE BETWEEN TO_DATE('"++ vars.queryParams.startDate ++
            "','YYYY-MM-DD') AND TO_DATE('"++ vars.queryParams.endDate ++"','YYYY-MM-DD')" 
            else ""
---
query ++ dates]]></ee:set-variable>
        </ee:variables>
    </ee:transform>
    
    <!-- Ejecutar consulta en Oracle -->
    <db:select doc:name="Select"
               config-ref="Database_Config"
               queryTimeout="${dbAC.queryTimeout}">
        <db:sql><![CDATA[#[vars.query]]]></db:sql>
    </db:select>
    
    <!-- Transformar resultados a lista de consumos -->
    <ee:transform doc:name="Transform Message">
        <ee:variables>
            <ee:set-variable variableName="listaConsumo"><![CDATA[%dw 2.0
output application/json

fun transform(listConsumoGas) = listConsumoGas map (consumoGas, index) -> {
    'tarifa': consumoGas.SAP_CONS_BELZART_CHZ1,
    'valor': consumoGas.SAP_CONS_SUM_ABRMENGE_CHZ1 as Number,
    'fecha': consumoGas.INV_INVOICE_DATE as Date {format: "MM-uuuu"}
}
---
transform(payload)]]></ee:set-variable>
        </ee:variables>
    </ee:transform>
    
    <!-- Iterar sobre cada mes -->
    <foreach doc:name="For Each" collection="#[vars.monthsArray]">
        <choice doc:name="Choice">
            <!-- Procesamiento para GAS -->
            <when expression='#[vars.queryParams.productType == "gas"]'>
                <logger level="DEBUG" doc:name="factura de gas"/>
                <ee:transform doc:name="Transform Message">
                    <ee:variables>
                        <ee:set-variable variableName="consumptions"><![CDATA[%dw 2.0
output application/json
var zG = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZGFCON" or value.tarifa == "ZGFCO1") and value.fecha == payload)
fun suma(a) =
    if (a.valor == null) 0
    else sum(a.valor)
---
if (suma(zG) != 0) 
    vars.consumptions ++ [{
        "date": "01-" ++ payload,
        "valueTotal": suma(zG)
    }]
else
    vars.consumptions]]></ee:set-variable>
                    </ee:variables>
                </ee:transform>
            </when>
            
            <!-- Procesamiento para ELECTRICIDAD -->
            <otherwise>
                <logger level="DEBUG" doc:name="factura de luz"/>
                <ee:transform doc:name="Transform Message">
                    <ee:variables>
                        <ee:set-variable variableName="consumptions"><![CDATA[%dw 2.0
output application/json
var zE = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZEFCON" or value.tarifa == "ZEC20D" or value.tarifa == "ZEFCPT" or 
     value.tarifa == "ZEIEP1" or value.tarifa == "ZEFACT" or value.tarifa == "ZEFAP1" or 
     value.tarifa == "ZEC20N" or value.tarifa == "ZEFCLL" or value.tarifa == "ZEIEP2" or 
     value.tarifa == "ZEFAP2" or value.tarifa == "ZEFCVA" or value.tarifa == "ZEIEP3" or 
     value.tarifa == "ZEFAP3" or value.tarifa == "ZEIEP5" or value.tarifa == "ZEFAP5" or 
     value.tarifa == "ZEIEP6" or value.tarifa == "ZEFAP6") and value.fecha == payload)
var zEAP1 = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZEFCON" or value.tarifa == "ZEC20D" or value.tarifa == "ZEFCPT" or 
     value.tarifa == "ZEIEP1" or value.tarifa == "ZEFACT" or value.tarifa == "ZEFAP1") 
     and value.fecha == payload)
var zEAP2 = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZEC20N" or value.tarifa == "ZEFCLL" or value.tarifa == "ZEIEP2" or 
     value.tarifa == "ZEFAP2") and value.fecha == payload)
var zEAP3 = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZEFCVA" or value.tarifa == "ZEIEP3" or value.tarifa == "ZEFAP3") 
     and value.fecha == payload)
var zEAP4 = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZEFCSV" or value.tarifa == "ZEIEP4" or value.tarifa == "ZEFAP4") 
     and value.fecha == payload)
var zEAP5 = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZEIEP5" or value.tarifa == "ZEFAP5") and value.fecha == payload)
var zEAP6 = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZEIEP6" or value.tarifa == "ZEFAP6") and value.fecha == payload)
fun suma(a) =
    if (a.valor == null) 0
    else sum(a.valor)
---
if (suma(zE) != 0) 
    vars.consumptions ++ [{
        "date": "01-" ++ payload,
        "valuePoint": suma(zEAP1),
        "valuePlain": suma(zEAP2),
        "valueValley": suma(zEAP3),
        "valueP4": suma(zEAP4),
        "valueP5": suma(zEAP5),
        "valueP6": suma(zEAP6),
        "valueTotal": suma(zE)
    }]
else 
    vars.consumptions]]></ee:set-variable>
                    </ee:variables>
                </ee:transform>
            </otherwise>
        </choice>
    </foreach>
    
    <!-- Construir respuesta final -->
    <ee:transform doc:name="Transform Message">
        <ee:message>
            <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    result: {
        codResult: "0000",
        msgResult: "No hay error"
    },
    consumptions: {
        contractId: vars.queryParams.contractId,
        productTypeFamily: vars.queryParams.productType,
        consumptionsValues: vars.consumptions
    }
}]]></ee:set-payload>
        </ee:message>
    </ee:transform>
    
    <!-- Logger de salida -->
    <logger level="INFO"
            doc:name="Logger salida"
            message="s-ac-consumptions.logSalida ## payload: #[payload]"/>
    
    <!-- Manejo de errores -->
    <error-handler>
        <on-error-propagate enableNotifications="true"
                           logException="true"
                           doc:name="On Error Propagate"
                           type="APP:ERRORCONTRATO">
            <ee:transform doc:name="Transform Message">
                <ee:message>
                    <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    result: {
        codResult: "-0001",
        msgResult: "Contrato no encontrado"
    }
}]]></ee:set-payload>
                </ee:message>
            </ee:transform>
        </on-error-propagate>
    </error-handler>
</flow>
```

### Query SQL Dinámica

**Construcción en DataWeave:**

```dataweave
%dw 2.0
output application/json
var query = "SELECT SAP_CONS_BELZART_CHZ1, 
             SAP_CONS_SUM_ABRMENGE_CHZ1, 
             INV_INVOICE_DATE 
             FROM v_fua_invoices 
             WHERE INV_SAP_VKONT IN (" ++ vars.queryParams.contractAccountId ++ ")"
var dates = if (!isEmpty(vars.queryParams.startDate) and !isEmpty(vars.queryParams.endDate)) 
            " AND INV_INVOICE_DATE BETWEEN TO_DATE('"++ vars.queryParams.startDate ++
            "','YYYY-MM-DD') AND TO_DATE('"++ vars.queryParams.endDate ++"','YYYY-MM-DD')" 
            else ""
---
query ++ dates
```

**Query SQL Resultante (ejemplo):**

```sql
SELECT SAP_CONS_BELZART_CHZ1, 
       SAP_CONS_SUM_ABRMENGE_CHZ1, 
       INV_INVOICE_DATE 
FROM v_fua_invoices 
WHERE INV_SAP_VKONT IN (123456,789012)
AND INV_INVOICE_DATE BETWEEN TO_DATE('2024-01-01','YYYY-MM-DD') 
                         AND TO_DATE('2024-12-31','YYYY-MM-DD')
```

**Columnas Consultadas:**
- `SAP_CONS_BELZART_CHZ1` - Código de tarifa (ZGFCON, ZEFCON, etc.)
- `SAP_CONS_SUM_ABRMENGE_CHZ1` - Cantidad consumida en kWh
- `INV_INVOICE_DATE` - Fecha de la factura
- `INV_SAP_VKONT` - Número de contrato (usado en WHERE)

### Transformaciones DataWeave

#### 1. Cálculo de Array de Meses

```dataweave
%dw 2.0
import * from dw::core::Periods
output application/json

var in = vars.queryParams.startDate splitBy (/-/)
var out = vars.queryParams.endDate splitBy (/-/)
var auxIn = in[0]++"-"++ in[1] ++ "-01"
var auxOut = out[0] ++ "-"++ out[1] ++ "-01"
var meses = Periods::between((auxOut as String) as Date, (auxIn as String as Date))

fun prepareList(list:Array, maxSize: Number) = 
    if(sizeOf(list) >= maxSize)
        list
    else (
        prepareList(
            list ++ [(auxOut as String as Date - period({months:(sizeOf(list))})) 
                     as String {format: "MM-uuuu"}],
            maxSize
        )
    )
---
prepareList([], meses.months + (meses.years * 12) + 1)
```

**Resultado (ejemplo para 2024-01-01 a 2024-03-31):**
```json
["03-2024", "02-2024", "01-2024"]
```

#### 2. Transformación de Resultados Oracle

```dataweave
%dw 2.0
output application/json

fun transform(listConsumoGas) = listConsumoGas map (consumoGas, index) -> {
    'tarifa': consumoGas.SAP_CONS_BELZART_CHZ1,
    'valor': consumoGas.SAP_CONS_SUM_ABRMENGE_CHZ1 as Number,
    'fecha': consumoGas.INV_INVOICE_DATE as Date {format: "MM-uuuu"}
}
---
transform(payload)
```

#### 3. Agregación para Gas

```dataweave
%dw 2.0
output application/json
var zG = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZGFCON" or value.tarifa == "ZGFCO1") and value.fecha == payload)
fun suma(a) =
    if (a.valor == null) 0
    else sum(a.valor)
---
if (suma(zG) != 0) 
    vars.consumptions ++ [{
        "date": "01-" ++ payload,
        "valueTotal": suma(zG)
    }]
else
    vars.consumptions
```

#### 4. Agregación para Electricidad

```dataweave
%dw 2.0
output application/json
// Filtrar todas las tarifas de electricidad
var zE = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZEFCON" or value.tarifa == "ZEC20D" or ... ) 
    and value.fecha == payload)

// Filtrar por periodo P1 (Punta)
var zEAP1 = vars.listaConsumo filter ((value, index) -> 
    (value.tarifa == "ZEFCON" or value.tarifa == "ZEC20D" or value.tarifa == "ZEFCPT" or 
     value.tarifa == "ZEIEP1" or value.tarifa == "ZEFACT" or value.tarifa == "ZEFAP1") 
     and value.fecha == payload)

// ... (similar para P2-P6)

fun suma(a) =
    if (a.valor == null) 0
    else sum(a.valor)
---
if (suma(zE) != 0) 
    vars.consumptions ++ [{
        "date": "01-" ++ payload,
        "valuePoint": suma(zEAP1),
        "valuePlain": suma(zEAP2),
        "valueValley": suma(zEAP3),
        "valueP4": suma(zEAP4),
        "valueP5": suma(zEAP5),
        "valueP6": suma(zEAP6),
        "valueTotal": suma(zE)
    }]
else 
    vars.consumptions
```

### Manejo de Errores

**Error Handler Personalizado:**

```xml
<error-handler>
    <on-error-propagate enableNotifications="true"
                       logException="true"
                       type="APP:ERRORCONTRATO">
        <ee:transform doc:name="Transform Message">
            <ee:message>
                <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
    result: {
        codResult: "-0001",
        msgResult: "Contrato no encontrado"
    }
}]]></ee:set-payload>
            </ee:message>
        </ee:transform>
    </on-error-propagate>
</error-handler>
```

### Consideraciones de Rendimiento

- **Timeout de Query:** 15 segundos configurables
- **Índices Recomendados:** 
  - `INV_SAP_VKONT` (usado en WHERE)
  - `INV_INVOICE_DATE` (usado en BETWEEN)
- **Optimización:** La query dinámica permite filtrar por fechas solo cuando es necesario
- **Connection Pool:** Configurado en `Database_Config` con MaxPoolSize: 15
- **Múltiples Contratos:** Soporta consulta de varios contratos en una sola llamada

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta de Consumo de Gas (1 Contrato)

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/consumptions?contractAccountId=123456&startDate=2024-01-01&endDate=2024-03-31&productType=gas&contractId=CON123456" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (200 OK):**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "consumptions": {
    "contractId": "CON123456",
    "productTypeFamily": "gas",
    "consumptionsValues": [
      {
        "date": "01-01-2024",
        "valueTotal": 245.50
      },
      {
        "date": "01-02-2024",
        "valueTotal": 312.75
      },
      {
        "date": "01-03-2024",
        "valueTotal": 198.30
      }
    ]
  }
}
```

### Ejemplo 2: Consulta de Consumo de Electricidad (1 Contrato)

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/consumptions?contractAccountId=789012&startDate=2024-01-01&endDate=2024-02-29&productType=luz&contractId=CON789012" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (200 OK):**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "consumptions": {
    "contractId": "CON789012",
    "productTypeFamily": "luz",
    "consumptionsValues": [
      {
        "date": "01-01-2024",
        "valuePoint": 45.30,
        "valuePlain": 78.20,
        "valueValley": 102.50,
        "valueP4": 12.30,
        "valueP5": 8.40,
        "valueP6": 5.20,
        "valueTotal": 251.90
      },
      {
        "date": "01-02-2024",
        "valuePoint": 52.10,
        "valuePlain": 85.40,
        "valueValley": 115.30,
        "valueP4": 14.20,
        "valueP5": 9.80,
        "valueP6": 6.10,
        "valueTotal": 282.90
      }
    ]
  }
}
```

### Ejemplo 3: Múltiples Contratos

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/consumptions?contractAccountId=123456,789012,345678&startDate=2024-01-01&endDate=2024-01-31&productType=gas&contractId=CON123456" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (200 OK):**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "consumptions": {
    "contractId": "CON123456",
    "productTypeFamily": "gas",
    "consumptionsValues": [
      {
        "date": "01-01-2024",
        "valueTotal": 687.90
      }
    ]
  }
}
```

### Ejemplo 4: Periodo Anual Completo

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/consumptions?contractAccountId=123456&startDate=2023-01-01&endDate=2023-12-31&productType=luz&contractId=CON123456" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (200 OK):**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "No hay error"
  },
  "consumptions": {
    "contractId": "CON123456",
    "productTypeFamily": "luz",
    "consumptionsValues": [
      {
        "date": "01-01-2023",
        "valuePoint": 45.30,
        "valuePlain": 78.20,
        "valueValley": 102.50,
        "valueP4": 12.30,
        "valueP5": 8.40,
        "valueP6": 5.20,
        "valueTotal": 251.90
      },
      {
        "date": "01-02-2023",
        "valuePoint": 42.10,
        "valuePlain": 75.30,
        "valueValley": 98.20,
        "valueP4": 11.50,
        "valueP5": 7.80,
        "valueP6": 4.90,
        "valueTotal": 239.80
      }
      // ... 10 meses más
    ]
  }
}
```

### Ejemplo 5: Error - Contrato No Encontrado

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/consumptions?contractAccountId=&startDate=2024-01-01&endDate=2024-01-31&productType=gas&contractId=CON999999" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (200 OK con error):**

```json
{
  "result": {
    "codResult": "-0001",
    "msgResult": "Contrato no encontrado"
  }
}
```

### Ejemplo 6: Error - Parámetros Faltantes

**Request:**

```bash
curl -X GET "https://api.naturgy.com/fua/s-ac/v1/consumptions?contractAccountId=123456&productType=gas" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (400 Bad Request):**

```json
{
  "error": {
    "code": "400",
    "message": "Bad Request - Missing required parameters: startDate, endDate, contractId"
  }
}
```

### Ejemplo 7: Integración con JavaScript

```javascript
async function consultarConsumos(params) {
  const {
    contractAccountId,
    startDate,
    endDate,
    productType,
    contractId
  } = params;
  
  const queryParams = new URLSearchParams({
    contractAccountId,
    startDate,
    endDate,
    productType,
    contractId
  });
  
  const url = `https://api.naturgy.com/fua/s-ac/v1/consumptions?${queryParams}`;
  
  try {
    const response = await fetch(url, {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${getAuthToken()}`
      }
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    
    if (data.result.codResult === "0000") {
      console.log("Consumos obtenidos:", data.consumptions);
      return data.consumptions;
    } else {
      console.error("Error en la consulta:", data.result.msgResult);
      return null;
    }
  } catch (error) {
    console.error("Error al consultar consumos:", error);
    throw error;
  }
}

// Uso - Gas
consultarConsumos({
  contractAccountId: "123456",
  startDate: "2024-01-01",
  endDate: "2024-12-31",
  productType: "gas",
  contractId: "CON123456"
}).then(consumos => {
  // Procesar datos de gas
  consumos.consumptionsValues.forEach(mes => {
    console.log(`${mes.date}: ${mes.valueTotal} kWh`);
  });
});

// Uso - Electricidad
consultarConsumos({
  contractAccountId: "789012",
  startDate: "2024-01-01",
  endDate: "2024-12-31",
  productType: "luz",
  contractId: "CON789012"
}).then(consumos => {
  // Procesar datos de electricidad
  consumos.consumptionsValues.forEach(mes => {
    console.log(`${mes.date}:`);
    console.log(`  Punta (P1): ${mes.valuePoint} kWh`);
    console.log(`  Llano (P2): ${mes.valuePlain} kWh`);
    console.log(`  Valle (P3): ${mes.valueValley} kWh`);
    console.log(`  Total: ${mes.valueTotal} kWh`);
  });
});
```

### Ejemplo 8: Integración con Python

```python
import requests
import json
from datetime import datetime, timedelta

def consultar_consumos(contract_account_id, start_date, end_date, 
                       product_type, contract_id, token):
    """
    Consulta consumos históricos de gas o electricidad
    
    Args:
        contract_account_id (str): ID(s) de cuenta de contrato (separados por coma)
        start_date (str): Fecha inicio (YYYY-MM-DD)
        end_date (str): Fecha fin (YYYY-MM-DD)
        product_type (str): 'gas' o 'luz'
        contract_id (str): ID del contrato
        token (str): Token de autenticación
        
    Returns:
        dict: Datos de consumos o None si hay error
    """
    url = "https://api.naturgy.com/fua/s-ac/v1/consumptions"
    
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {token}"
    }
    
    params = {
        "contractAccountId": contract_account_id,
        "startDate": start_date,
        "endDate": end_date,
        "productType": product_type,
        "contractId": contract_id
    }
    
    try:
        response = requests.get(url, headers=headers, params=params, timeout=30)
        response.raise_for_status()
        
        data = response.json()
        
        if data.get("result", {}).get("codResult") == "0000":
            print(f"Consumos obtenidos para {product_type}")
            return data.get("consumptions")
        else:
            print(f"Error: {data.get('result', {}).get('msgResult')}")
            return None
            
    except requests.exceptions.RequestException as e:
        print(f"Error en la petición: {e}")
        return None

# Uso - Gas
token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
consumos_gas = consultar_consumos(
    contract_account_id="123456",
    start_date="2024-01-01",
    end_date="2024-12-31",
    product_type="gas",
    contract_id="CON123456",
    token=token
)

if consumos_gas:
    print(f"\nContrato: {consumos_gas['contractId']}")
    print(f"Tipo: {consumos_gas['productTypeFamily']}")
    print("\nConsumos mensuales:")
    for mes in consumos_gas['consumptionsValues']:
        print(f"  {mes['date']}: {mes['valueTotal']} kWh")

# Uso - Electricidad
consumos_luz = consultar_consumos(
    contract_account_id="789012",
    start_date="2024-01-01",
    end_date="2024-12-31",
    product_type="luz",
    contract_id="CON789012",
    token=token
)

if consumos_luz:
    print(f"\nContrato: {consumos_luz['contractId']}")
    print(f"Tipo: {consumos_luz['productTypeFamily']}")
    print("\nConsumos mensuales por periodo:")
    for mes in consumos_luz['consumptionsValues']:
        print(f"\n  {mes['date']}:")
        print(f"    Punta (P1):  {mes['valuePoint']:>8.2f} kWh")
        print(f"    Llano (P2):  {mes['valuePlain']:>8.2f} kWh")
        print(f"    Valle (P3):  {mes['valueValley']:>8.2f} kWh")
        print(f"    P4:          {mes['valueP4']:>8.2f} kWh")
        print(f"    P5:          {mes['valueP5']:>8.2f} kWh")
        print(f"    P6:          {mes['valueP6']:>8.2f} kWh")
        print(f"    Total:       {mes['valueTotal']:>8.2f} kWh")

# Función auxiliar para calcular totales
def calcular_total_periodo(consumos):
    """Calcula el total de consumo del periodo"""
    if not consumos or 'consumptionsValues' not in consumos:
        return 0
    
    total = sum(mes['valueTotal'] for mes in consumos['consumptionsValues'])
    return total

total_gas = calcular_total_periodo(consumos_gas)
total_luz = calcular_total_periodo(consumos_luz)

print(f"\n=== RESUMEN ===")
print(f"Total Gas:          {total_gas:>10.2f} kWh")
print(f"Total Electricidad: {total_luz:>10.2f} kWh")
print(f"Total General:      {total_gas + total_luz:>10.2f} kWh")
```

### Ejemplo 9: Testing con Postman

**Pre-request Script:**

```javascript
// Generar timestamp
pm.environment.set("timestamp", new Date().toISOString());

// Validar parámetros requeridos
const requiredParams = [
    'contractAccountId',
    'startDate',
    'endDate',
    'productType',
    'contractId'
];

requiredParams.forEach(param => {
    if (!pm.request.url.query.get(param)) {
        console.error(`Falta el parámetro requerido: ${param}`);
    }
});

// Validar formato de fechas
const startDate = pm.request.url.query.get('startDate');
const endDate = pm.request.url.query.get('endDate');
const dateRegex = /^\d{4}-\d{2}-\d{2}$/;

if (startDate && !dateRegex.test(startDate)) {
    console.error('startDate debe tener formato YYYY-MM-DD');
}
if (endDate && !dateRegex.test(endDate)) {
    console.error('endDate debe tener formato YYYY-MM-DD');
}

// Validar productType
const productType = pm.request.url.query.get('productType');
if (productType && !['gas', 'luz'].includes(productType)) {
    console.error('productType debe ser "gas" o "luz"');
}
```

**Tests:**

```javascript
// Verificar status code
pm.test("Status code es 200", function () {
    pm.response.to.have.status(200);
});

// Verificar estructura de respuesta
pm.test("Respuesta tiene estructura correcta", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property("result");
    pm.expect(jsonData.result).to.have.property("codResult");
    pm.expect(jsonData.result).to.have.property("msgResult");
});

// Verificar código de resultado exitoso
pm.test("Código de resultado es 0000", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.result.codResult).to.eql("0000");
});

// Verificar datos de consumos
pm.test("Respuesta contiene datos de consumos", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property("consumptions");
    pm.expect(jsonData.consumptions).to.have.property("contractId");
    pm.expect(jsonData.consumptions).to.have.property("productTypeFamily");
    pm.expect(jsonData.consumptions).to.have.property("consumptionsValues");
    pm.expect(jsonData.consumptions.consumptionsValues).to.be.an('array');
});

// Verificar estructura según tipo de producto
pm.test("Estructura de consumptionsValues es correcta", function () {
    var jsonData = pm.response.json();
    var productType = pm.request.url.query.get('productType');
    var values = jsonData.consumptions.consumptionsValues;
    
    if (values.length > 0) {
        var firstValue = values[0];
        pm.expect(firstValue).to.have.property('date');
        pm.expect(firstValue).to.have.property('valueTotal');
        
        if (productType === 'luz') {
            pm.expect(firstValue).to.have.property('valuePoint');
            pm.expect(firstValue).to.have.property('valuePlain');
            pm.expect(firstValue).to.have.property('valueValley');
            pm.expect(firstValue).to.have.property('valueP4');
            pm.expect(firstValue).to.have.property('valueP5');
            pm.expect(firstValue).to.have.property('valueP6');
        }
    }
});

// Verificar tiempo de respuesta
pm.test("Tiempo de respuesta menor a 5000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(5000);
});

// Guardar datos para siguiente request
if (pm.response.code === 200) {
    var jsonData = pm.response.json();
    if (jsonData.result.codResult === "0000") {
        pm.environment.set("lastConsumptionData", JSON.stringify(jsonData.consumptions));
    }
}
```

### Ejemplo 10: Visualización con Chart.js

```html
<!DOCTYPE html>
<html>
<head>
    <title>Consumos Energéticos</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <canvas id="consumptionChart" width="800" height="400"></canvas>
    
    <script>
        async function cargarConsumos() {
            const response = await fetch(
                'https://api.naturgy.com/fua/s-ac/v1/consumptions?' +
                'contractAccountId=123456&' +
                'startDate=2024-01-01&' +
                'endDate=2024-12-31&' +
                'productType=luz&' +
                'contractId=CON123456',
                {
                    headers: {
                        'Authorization': 'Bearer ' + getAuthToken()
                    }
                }
            );
            
            const data = await response.json();
            return data.consumptions.consumptionsValues;
        }
        
        cargarConsumos().then(consumos => {
            const ctx = document.getElementById('consumptionChart').getContext('2d');
            
            new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: consumos.map(c => c.date),
                    datasets: [
                        {
                            label: 'Punta (P1)',
                            data: consumos.map(c => c.valuePoint),
                            backgroundColor: 'rgba(255, 99, 132, 0.5)'
                        },
                        {
                            label: 'Llano (P2)',
                            data: consumos.map(c => c.valuePlain),
                            backgroundColor: 'rgba(54, 162, 235, 0.5)'
                        },
                        {
                            label: 'Valle (P3)',
                            data: consumos.map(c => c.valueValley),
                            backgroundColor: 'rgba(75, 192, 192, 0.5)'
                        }
                    ]
                },
                options: {
                    responsive: true,
                    scales: {
                        x: { stacked: true },
                        y: { 
                            stacked: true,
                            title: {
                                display: true,
                                text: 'Consumo (kWh)'
                            }
                        }
                    },
                    plugins: {
                        title: {
                            display: true,
                            text: 'Consumo Eléctrico por Periodo'
                        }
                    }
                }
            });
        });
    </script>
</body>
</html>
```
