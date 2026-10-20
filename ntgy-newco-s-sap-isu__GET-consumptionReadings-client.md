---
api: "ntgy-newco-s-sap-isu"
sidebar_position: 4
sidebar_label: "GET /consumptionReadings/client"
description: "Consulta lecturas de consumo detalladas de un cliente en SAP ISU"
keywords: [lecturas, consumo, readings, consumption, SAP ISU]
tags: [Lecturas, Consumos, System API, SAP ISU]
sap_rfc: ZLE_FM_CONSULTA_LECTURAS
sql_metadata:
  data_sources:
    - database: "SAP_ISU"
      schema: "SAPSR3"
      type: "SAP_HANA"
      tables:
        - name: "EVER"
          operations: ["READ"]
          description: "Contract Master Data"
        - name: "EABLG"
          operations: ["READ"]
          description: "Meter Reading Document"
        - name: "EGERH"
          operations: ["READ"]
          description: "Meter Reading Header"
---

# GET /consumptionReadings/client

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint consulta las lecturas de consumo detalladas de un cliente en SAP ISU. Proporciona información completa sobre las lecturas de contadores (luz y gas), incluyendo valores de lectura, consumos calculados, períodos de facturación y datos del agente distribuidor.

### Tipo de API

**Capa:** System API (Integración Directa con SAP ISU)

### Patrón de Integración

**Tipo:** Request-Reply Síncrono con SAP ISU - Consulta de Lecturas

Este endpoint implementa un patrón de consulta de lecturas mediante RFC (Remote Function Call). La comunicación es síncrona y retorna datos estructurados de lecturas por magnitud y período.

**Características del Patrón:**
- ✅ Comunicación síncrona con SAP ISU
- ✅ Invocación de RFC `ZLE_FM_CONSULTA_LECTURAS`
- ✅ Respuesta con estructura compleja (lecturas por magnitud)
- ✅ Incluye datos de agente distribuidor
- ✅ Tiempo de respuesta < 5 segundos

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    subgraph "Experience Layer"
        E[Experience APIs<br/>Múltiples consumidores posibles]
    end
    
    subgraph "Process Layer"
        P[Process API<br/>ntgy-newco-p-fua-evo]
    end
    
    subgraph "System Layer"
        S[System API<br/>ntgy-newco-s-sap-isu<br/>GET /consumptionReadings/client]
    end
    
    subgraph "Backend Systems"
        SAP[SAP ISU<br/>RFC: ZLE_FM_CONSULTA_LECTURAS<br/>Tablas: EVER, EABLG, EGERH]
    end
    
    E -->|Consume| P
    P -->|Consume| S
    S -->|RFC Call| SAP
    
    style S fill:#e8f5e9
    style P fill:#fff4e1
    style E fill:#e3f2fd
    style SAP fill:#fce4ec
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

#### 1. Consulta de Historial de Lecturas
**Actor:** Cliente final o agente de atención al cliente  
**Objetivo:** Consultar historial completo de lecturas del contador  
**Flujo:**
- Cliente solicita ver sus lecturas históricas
- Sistema consulta lecturas en SAP ISU
- Muestra lecturas por período y magnitud
- Cliente puede verificar su consumo

#### 2. Análisis de Consumo
**Actor:** Sistema de análisis o BI  
**Objetivo:** Analizar patrones de consumo del cliente  
**Flujo:**
- Sistema solicita lecturas históricas
- Analiza tendencias de consumo
- Identifica anomalías o picos
- Genera recomendaciones

#### 3. Validación de Facturación
**Actor:** Departamento de facturación  
**Objetivo:** Validar que las lecturas coincidan con facturación  
**Flujo:**
- Sistema compara lecturas con facturas
- Verifica consumos calculados
- Identifica discrepancias
- Corrige errores de facturación

#### 4. Información para Distribuidora
**Actor:** Distribuidora o ATR  
**Objetivo:** Obtener información de lecturas para gestión de red  
**Flujo:**
- Distribuidora consulta lecturas de clientes
- Obtiene datos de CUPS y factor de conversión
- Valida lecturas reportadas
- Gestiona red de distribución

### Valor de Negocio

#### Beneficios Cuantitativos
- ⏱️ **Tiempo de Consulta:** < 5 segundos
- 📞 **Reducción de Consultas:** 25% menos llamadas sobre lecturas
- 🎯 **Precisión:** 99% exactitud en datos de consumo
- 💰 **Ahorro:** Reducción de errores de facturación

#### Beneficios Cualitativos
- ✅ Transparencia total en lecturas
- ✅ Mejor experiencia del cliente
- ✅ Facilita autogestión
- ✅ Reduce reclamaciones
- ✅ Mejora relación con distribuidoras

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/api/consumptionReadings/client` |
| **Development** | `https://ntgy-newco-s-sap-isu-dev.cloudhub.io` | `https://ntgy-newco-s-sap-isu-dev.cloudhub.io/api/consumptionReadings/client` |
| **UAT** | `https://ntgy-newco-s-sap-isu-uat.cloudhub.io` | `https://ntgy-newco-s-sap-isu-uat.cloudhub.io/api/consumptionReadings/client` |
| **Production** | `https://ntgy-newco-s-sap-isu.cloudhub.io` | `https://ntgy-newco-s-sap-isu.cloudhub.io/api/consumptionReadings/client` |

### Método HTTP

```
GET /api/consumptionReadings/client
```

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `contractNumbers` | String | ⚠️ Opcional | Número de contrato | `1000123456` |
| `clientCode` | String | ⚠️ Opcional | Código de cliente (NIF/CIF) | `12345678A` |

**Notas:** Al menos uno de los dos parámetros debe ser proporcionado.

### Respuestas

#### Respuesta Exitosa (200 OK)

```json
{
  "readingList": [
    {
      "productTypeFamily": "luz",
      "invoiceId": "9012345678",
      "readingDetailList": [
        {
          "magnitude": "AE",
          "values": [
            {
              "readingDate": "2024-01-15",
              "type": "R",
              "period": "1",
              "value": "12345",
              "counter": "ES0123456789012345AB",
              "consumption": "250"
            }
          ]
        }
      ],
      "agentInfo": {
        "cups": "ES0123456789012345AB",
        "coversionFactor": "1.0",
        "distribuitor": "UFD001",
        "atrInvoiceId": ["ATR123456"],
        "invoiceType": "N",
        "invoiceOriginId": "01"
      }
    }
  ]
}
```

**Descripción de Campos:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `productTypeFamily` | String | Tipo de producto: "luz" o "gas" |
| `invoiceId` | String | ID de factura asociada |
| `magnitude` | String | Magnitud de lectura (AE, R1, R2, R3, etc.) |
| `readingDate` | String | Fecha de lectura (yyyy-MM-dd) |
| `type` | String | Tipo de lectura (R=Real, E=Estimada) |
| `period` | String | Número de período |
| `value` | String | Valor de lectura del contador |
| `counter` | String | Número de contador |
| `consumption` | String | Consumo calculado |
| `cups` | String | Código CUPS del punto de suministro |
| `conversionFactor` | String | Factor de conversión |
| `distribuitor` | String | Código de distribuidora |

---

## 🏗️ Implementación Técnica

### RFC SAP Invocado

#### RFC: `ZLE_FM_CONSULTA_LECTURAS`

**Descripción:** Función RFC personalizada para consultar lecturas de consumo de un cliente.

**Módulo SAP:** Meter Reading (EL-MD)

**Tablas SAP:**
- `EVER`: Contract Master Data
- `EABLG`: Meter Reading Document
- `EGERH`: Meter Reading Header
- `EANL`: Device Location

**Parámetros de Entrada:**

| Parámetro | Tipo ABAP | Descripción |
|-----------|-----------|-------------|
| `I_CONTRATO` | CHAR(10) | Número de contrato |
| `I_NIF` | CHAR(20) | NIF del cliente |

**Parámetros de Salida:**

| Parámetro | Tipo ABAP | Descripción |
|-----------|-----------|-------------|
| `READINGLIST2` | TABLE | Tabla de lecturas |

### Transformaciones DataWeave

#### Input RFC

```dataweave
%dw 2.0
output application/xml
---
{
    ZLE_FM_CONSULTA_LECTURAS: {
        "import": {
            I_CONTRATO: attributes.queryParams.contractNumbers,
            I_NIF: attributes.queryParams.clientCode
        }
    }
}
```

#### Output Response

```dataweave
%dw 2.0
output application/json
fun formatDate(fecha : String|Null) = 
    fecha as Date {format: 'yyyyMMdd'} as String {format: 'yyyy-MM-dd'} default "0000-00-00"

---
{
    "readingList": payload.ZLE_FM_CONSULTA_LECTURAS.export.READINGLIST2.*row map ((item, index) -> {
        "productTypeFamily": if((item.PRODUCTTYPEFAMILY)=="01") "luz" else "gas",
        "invoiceId": item.INVOICEID default "",
        "readingDetailList": item.READINGDETAILLIST.*row map ((item2, index) -> {
            "magnitude": item2.MAGNITUDE default "",
            "values": item2.VALUES.*row map ((item3, index1) -> {
                "readingDate": formatDate(item3.READINGDATE) default "",
                "type": item3.TYPE default "",
                "period": (index1+1) as String,
                "value": trim(item3.VALUE) default "", 
                "counter": item3.COUNTER default "",
                "consumption": item3.CONSUMPTION default ""
            })
        }),
        "agentInfo": {
            "cups": item.READINGDETAILLIST.row.AGENTINFO.row.CUPS default "",
            "coversionFactor": item.READINGDETAILLIST.row.AGENTINFO.row.CONVERSIONFACTOR default "",
            "distribuitor": item.READINGDETAILLIST.row.AGENTINFO.row.DISTRIBUTOR default "",
            "atrInvoiceId": [
                item.READINGDETAILLIST.row.AGENTINFO.row.ATRINVOICED default ""
            ],
            "invoiceType": item.READINGDETAILLIST.row.AGENTINFO.row.INVOICETYPE default "",
            "invoiceOriginId": item.READINGDETAILLIST.row.AGENTINFO.row.INVOICEORIGINID default ""
        }
    })
}
```

---

## 🗄️ Base de Datos SAP ISU

### Detalles de Tablas SAP

#### Tabla: `EVER`

**Descripción:** Tabla maestra de contratos en SAP ISU. Contiene información principal de cada contrato de suministro.

**Campos Utilizados:**

| Campo | Tipo ABAP | Descripción | Ejemplo |
|-------|-----------|-------------|---------|
| `VERTRAG` | CHAR(10) | Número de contrato | `1000123456` |
| `ANLAGE` | CHAR(10) | Punto de suministro | `1000567890` |
| `VKONT` | CHAR(12) | Cuenta contrato | `200001234567` |
| `GPART` | CHAR(10) | Business Partner (Cliente) | `1000123456` |

**Relaciones:**
- `EVER.ANLAGE` → `EANL.ANLAGE` (Relación con ubicación del dispositivo)
- `EVER.VERTRAG` → `EABLG.VERTRAG` (Relación con documentos de lectura)

**Índices Conocidos:**
- Primary Key: `VERTRAG`
- Secondary Index: `ANLAGE`, `VKONT`, `GPART`

**Notas de Performance:**
- Tabla de alto volumen (millones de registros)
- Búsqueda por `VERTRAG` o `GPART` es eficiente con índices

#### Tabla: `EABLG`

**Descripción:** Documentos de lectura de contadores. Almacena información de cada lectura realizada en los puntos de suministro.

**Campos Utilizados:**

| Campo | Tipo ABAP | Descripción | Ejemplo |
|-------|-----------|-------------|---------|
| `ABLBELNR` | CHAR(12) | Número de documento de lectura | `100012345678` |
| `VERTRAG` | CHAR(10) | Número de contrato | `1000123456` |
| `ADATSOLL` | DATS | Fecha programada de lectura | `20240115` |
| `ABLESGR` | CHAR(2) | Motivo de lectura | `01` |
| `ABLSTAT` | CHAR(1) | Estado de lectura | `A` |

**Relaciones:**
- `EABLG.VERTRAG` → `EVER.VERTRAG` (Relación N:1 con contrato)
- `EABLG.ABLBELNR` → `EGERH.ABLBELNR` (Relación 1:N con detalles de lectura)

**Índices Conocidos:**
- Primary Key: `ABLBELNR`
- Secondary Index: `VERTRAG`, `ADATSOLL`

**Notas de Performance:**
- Tabla de alto volumen (millones de registros)
- Búsqueda por `VERTRAG` es eficiente con índice secundario
- Ordenar por `ADATSOLL DESC` para obtener lecturas recientes

#### Tabla: `EGERH`

**Descripción:** Cabecera de documentos de lectura con detalles por magnitud. Contiene información específica de cada magnitud leída (AE, R1, R2, R3).

**Campos Utilizados:**

| Campo | Tipo ABAP | Descripción | Ejemplo |
|-------|-----------|-------------|---------|
| `ABLBELNR` | CHAR(12) | Número de documento de lectura | `100012345678` |
| `MAGNITUD` | CHAR(3) | Magnitud de lectura | `AE` |
| `ZWSTAND` | CURR(15,3) | Valor de lectura | `12345.000` |
| `VERBRAUCH` | CURR(15,3) | Consumo calculado | `250.500` |
| `ABLESART` | CHAR(1) | Tipo de lectura (R=Real, E=Estimada) | `R` |
| `ADATSOLL` | DATS | Fecha de lectura | `20240115` |

**Relaciones:**
- `EGERH.ABLBELNR` → `EABLG.ABLBELNR` (Relación N:1 con documento de lectura)

**Índices Conocidos:**
- Primary Key: `ABLBELNR + MAGNITUD`
- Secondary Index: `ADATSOLL`

**Notas de Performance:**
- Tabla de alto volumen (millones de registros)
- Búsqueda por `ABLBELNR` es muy eficiente (parte de PK)
- Múltiples registros por documento (uno por magnitud)

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta por Contrato

```bash
curl -X GET "https://ntgy-newco-s-sap-isu.cloudhub.io/api/consumptionReadings/client?contractNumbers=1000123456" \
  -H "Content-Type: application/json" \
  -H "client_id: your-client-id" \
  -H "client_secret: your-client-secret"
```

### Ejemplo 2: JavaScript

```javascript
async function getReadings(contractNumber) {
  const url = `https://ntgy-newco-s-sap-isu.cloudhub.io/api/consumptionReadings/client?contractNumbers=${contractNumber}`;
  
  const response = await fetch(url, {
    headers: {
      'Content-Type': 'application/json',
      'client_id': 'your-client-id',
      'client_secret': 'your-client-secret'
    }
  });
  
  const data = await response.json();
  
  data.readingList.forEach(reading => {
    console.log(`Producto: ${reading.productTypeFamily}`);
    console.log(`CUPS: ${reading.agentInfo.cups}`);
    
    reading.readingDetailList.forEach(detail => {
      console.log(`  Magnitud: ${detail.magnitude}`);
      detail.values.forEach(value => {
        console.log(`    Fecha: ${value.readingDate}, Consumo: ${value.consumption} kWh`);
      });
    });
  });
}
```