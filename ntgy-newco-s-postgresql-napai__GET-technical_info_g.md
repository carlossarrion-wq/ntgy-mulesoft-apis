---
api: "ntgy-newco-s-postgresql-napai"
sidebar_position: 4
sidebar_label: "GET /technical_info_g"
sql_metadata:
  data_sources:
    - database: "PostgreSQL"
      schema: "Configurado dinámicamente (sips.esquema.gas)"
      type: "PostgreSQL"
      tables:
        - name: "tnpi_dim_suministros_gas"
          operations: ["READ"]
        - name: "tnpi_dim_t_consumos_gas"
          operations: ["READ"]
        - name: "tnpi_dim_tarifa"
          operations: ["READ"]
---

# Endpoint: GET /technical_info_g

## 📋 Descripción General

Recupera los datos técnicos de suministros de gas desde la base de datos PostgreSQL.

**Tipo de API:** System API (Internal)  
**Capa:** System Layer  
**Versión:** 1.12.0-SNAPSHOT  
**Última actualización:** 2026-06-17

---

## 🎯 Propósito

Este endpoint proporciona acceso a información técnica completa de puntos de suministro de gas:

- Consulta por código CUPS con validación de formato
- Información de tarifa normalizada y presión de suministro
- Datos del contador, calibre y tipo de corrector
- Consumos históricos (12 meses, año cerrado, año móvil)
- Información de distribuidora y datos de inspección
- Datos de telemedida y conexión a punto satélite
- Información de ubicación completa del punto de suministro
- Más de 40 campos con información detallada

**Casos de uso:**
- Consulta de características técnicas del suministro
- Validación de tarifa y presión
- Información para cambios de contrato
- Datos para análisis de consumo
- Auditoría de datos técnicos
- Verificación de consumos históricos
- Planificación de inspecciones

---

## 🔗 URL del Endpoint

```
GET /api/technical_info_g?idCUPS=<CUPS>
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-napai-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-napai-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `idCUPS` | string | Sí | Código CUPS del punto de suministro de gas |

**Formato esperado del CUPS:**
- Patrón regex: `^ES\d{4}\d{12}[A-Z]{2}([A-Z0-9]{0,2})?$`
- Ejemplo válido: `ES0021000000000001JN`
- Estructura: ES + 4 dígitos + 12 dígitos + 2 letras + (opcional) hasta 2 caracteres alfanuméricos

**Validación:**
- ✅ Si el CUPS cumple el formato, se ejecuta la consulta
- ❌ Si el CUPS NO cumple el formato, retorna array vacío en `cups` (sin error)

**Ejemplo:**
```
?idCUPS=ES0021000000000001JN
```

### Headers

| Header | Valor | Requerido | Descripción |
|--------|-------|-----------|-------------|
| `Accept` | `application/json` | No | Tipo de contenido esperado en la respuesta |

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

**Estructura completa de la respuesta:**

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": ""
  },
  "cups": [
    {
      "fec_actualiz_reg": "2026-06-15T10:30:00",
      "idCUPS": "ES0021000000000001JN",
      "idProvinciaPto": "28",
      "idCodigoPostalPto": "28001",
      "idPresionPto": "BP",
      "idRangoPresion": "1",
      "idTarifaNormalizada": "RL.1",
      "idCalibreContador": "G4",
      "idEscaleraPto": "A",
      "idAnyoMesConsumo": "202606",
      "desMunicipioPto": "Madrid",
      "idNumeroFincaPto": "1",
      "desPCTD": "PCTD001",
      "idPisoPto": "3",
      "idPortalFincaPto": "B",
      "desPrimerApellido": "García",
      "idPuertaPto": "2",
      "idUnidadOrganizativa": "SEG01",
      "desSegundoApellido": "López",
      "desTipoContador": "Electrónico",
      "idTipoEnergia": "Gas Natural",
      "idNomViaPto": "Calle Mayor",
      "idFechaUltCambioComerc": "01/03/2025",
      "idFechaUltInsp": "15/01/2024",
      "idAccessContador": "1",
      "idCNAE": "9820",
      "idContador": "GAS12345678",
      "idEmpresaDistribuidora": "0123",
      "idConectadoPtaSatelite": "N",
      "idIndDerechoTUR": "S",
      "idIndTelemedida": "N",
      "idMunicipioPto": "28079",
      "idPropiedadEquipoMedida": "Distribuidora",
      "idResultadoUltInsp": "Favorable",
      "idTipoCorrector": "Electrónico",
      "idPerfilConsumo": "1",
      "idTipoViaPto": "CL",
      "fCaudalHorario": "2.5",
      "fCaudalMaxContratado": "10.0",
      "fConsumoAnyoCerr": "8500",
      "idConsumo12Meses": "8750",
      "idConsumo12MesesAnt": "8200",
      "idConsumoAnyoNatural": "8500",
      "fConsumo12mes2Ant": "7900",
      "fConsumoAnyoCerrAnterior": "8200"
    }
  ]
}
```

### 📊 Estructura de Respuesta

**Wrapper principal:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result` | object | Objeto con información del resultado de la operación |
| `result.codResult` | string | Código de resultado ("0000" = éxito) |
| `result.msgResult` | string | Mensaje de resultado (vacío si éxito) |
| `cups` | array | Array con los datos técnicos del suministro (puede estar vacío si CUPS inválido) |

**Campos principales del objeto en `cups[]` (más de 40 campos):**

**Identificación:**
- `idCUPS`: Código CUPS del punto de suministro
- `idProvinciaPto`: Código de provincia del punto de suministro
- `idCodigoPostalPto`: Código postal del punto de suministro
- `idMunicipioPto`: Código de municipio

**Tarifa y Presión:**
- `idTarifaNormalizada`: Tarifa normalizada (ej: "RL.1", "RL.2")
- `idPresionPto`: Presión del punto (BP, MP, AP)
- `idRangoPresion`: Rango de presión numérico
- `idTipoEnergia`: Tipo de energía (Gas Natural)

**Contador y Medida:**
- `idContador`: Número del contador de gas
- `idCalibreContador`: Calibre del contador (G4, G6, G10, etc.)
- `desTipoContador`: Tipo de contador (Electrónico, Mecánico)
- `idTipoCorrector`: Tipo de corrector
- `idPropiedadEquipoMedida`: Propiedad del equipo de medida
- `idAccessContador`: Accesibilidad del contador

**Consumos Históricos:**
- `fConsumoAnyoCerr`: Consumo año cerrado
- `idConsumo12Meses`: Consumo 12 meses (año móvil)
- `idConsumo12MesesAnt`: Consumo 12 meses anterior
- `idConsumoAnyoNatural`: Consumo año natural
- `fConsumo12mes2Ant`: Consumo 12 meses 2 años anteriores
- `fConsumoAnyoCerrAnterior`: Consumo año cerrado anterior

**Caudal:**
- `fCaudalHorario`: Caudal horario (m³/h)
- `fCaudalMaxContratado`: Caudal máximo contratado (m³/h)

**Empresa y Servicios:**
- `idEmpresaDistribuidora`: Código de la distribuidora
- `idIndTelemedida`: Indicador de telemedida
- `idConectadoPtaSatelite`: Conectado a punto satélite
- `idIndDerechoTUR`: Indicador de derecho a TUR

**Ubicación:**
- `idNomViaPto`: Nombre de la vía
- `idTipoViaPto`: Tipo de vía (CL, AV, PZ, etc.)
- `idNumeroFincaPto`: Número de finca
- `idPortalFincaPto`: Portal
- `idEscaleraPto`: Escalera
- `idPisoPto`: Piso
- `idPuertaPto`: Puerta
- `desMunicipioPto`: Municipio

**Fechas (formato dd/mm/yyyy):**
- `idFechaUltCambioComerc`: Fecha último cambio comercial
- `idFechaUltInsp`: Fecha última inspección

**Inspección:**
- `idResultadoUltInsp`: Resultado última inspección
- `fec_actualiz_reg`: Fecha de actualización del registro

**Otros:**
- `idCNAE`: Código CNAE
- `idPerfilConsumo`: Perfil de consumo
- `idAnyoMesConsumo`: Año y mes de consumo máximo
- `desPCTD`: Código PCTD
- `idUnidadOrganizativa`: Segmento/Unidad organizativa
- `desPrimerApellido`: Primer apellido
- `desSegundoApellido`: Segundo apellido

**Nota:** La respuesta incluye más de 40 campos específicos de gas. Consultar el ejemplo completo para ver todos los campos disponibles.

---

### ❌ Errores

Los errores siguen la misma estructura que `technical_info_e`. Ver documentación de errores en ese endpoint.

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consultar datos técnicos de un CUPS de gas válido

```bash
curl -X GET "http://localhost:8081/api/technical_info_g?idCUPS=ES0021000000000001JN" \
  -H "Accept: application/json"
```

**Respuesta (estructura completa con más de 40 campos):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": ""
  },
  "cups": [
    {
      "idCUPS": "ES0021000000000001JN",
      "idTarifaNormalizada": "RL.1",
      "idPresionPto": "BP",
      "idCalibreContador": "G4",
      "idContador": "GAS12345678",
      "fCaudalMaxContratado": "10.0",
      "idConsumo12Meses": "8750"
      // ... más de 30 campos adicionales
    }
  ]
}
```

---

### Ejemplo 2: CUPS con formato inválido

```bash
curl -X GET "http://localhost:8081/api/technical_info_g?idCUPS=INVALID_CUPS" \
  -H "Accept: application/json"
```

**Respuesta (array vacío, sin error):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": ""
  },
  "cups": []
}
```

---

### Ejemplo 3: CUPS no encontrado en base de datos

```bash
curl -X GET "http://localhost:8081/api/technical_info_g?idCUPS=ES0021000000000999ZZ" \
  -H "Accept: application/json"
```

**Respuesta (array vacío):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": ""
  },
  "cups": []
}
```

---

## 🔧 Implementación Técnica

### Flujo de Ejecución

```
HTTP GET Request → APIKit Router → technical_info_gSub_Flow → 
Extract idCUPS → Validate CUPS → Database Query → DataWeave Transform → JSON Response
```

### Código Fuente

**Archivo:** `src/main/mule/sips/technical_info_g.xml`

**Sub-flow:** `technical_info_gSub_Flow`

### Validación de CUPS

El código implementa validación del formato CUPS antes de ejecutar la consulta:

```xml
<when expression="#[vars.idCUPS matches /^ES\d{4}\d{12}[A-Z]{2}([A-Z0-9]{0,2})?$/]">
```

**Comportamiento:**
- ✅ **CUPS válido:** Ejecuta la consulta SQL
- ❌ **CUPS inválido:** Retorna respuesta con array `cups` vacío (sin error)

### Query SQL

La query consulta tres tablas con JOINs:

```sql
SELECT sg.fec_actualiz_reg,
       sg.cod_cups as "idCUPS",
       sg.cod_presion_pto as "idPresionPto",
       CASE 
         WHEN t.id_tarifa_sbl IS NOT NULL THEN t.id_tarifa_sbl
         ELSE sg.cod_tarifa
       END as "idTarifaNormalizada",
       sg.des_calibre_contador as "idCalibreContador",
       -- ... más de 40 campos ...
FROM :esquema.tnpi_dim_suministros_gas sg  
INNER JOIN :esquema.tnpi_dim_t_consumos_gas cg ON sg.cod_cups = cg.cod_cups
LEFT JOIN publication_napai.tnpi_dim_tarifa t ON t.cod_tarifa = sg.cod_tarifa 
  AND t.id_sector = '2' AND t.id_tarifa_sbl NOT LIKE '*%'
WHERE sg.cod_cups = 'ES0021000000000001JN'
```

**Características:**
- **Tablas consultadas:**
  - `tnpi_dim_suministros_gas`: Datos técnicos del suministro de gas
  - `tnpi_dim_t_consumos_gas`: Consumos históricos de gas
  - `tnpi_dim_tarifa`: Normalización de tarifas
- **Schema dinámico:** Configurado mediante propiedad `sips.esquema.gas`
- **Lógica de tarifa:** Usa tarifa normalizada si existe, sino usa la original
- **Cálculos de consumo:** Suma consumos de 12 meses con lógica condicional

### Transformación DataWeave

```dataweave
%dw 2.0
output application/json
---
{
  "result": {
    "codResult": "0000",
    "msgResult": ""
  },
  "cups": payload
}
```

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-napai.raml`
- **Main Flow:** `src/main/mule/sips/technical_info_g.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`
- **Database Config:** `src/main/mule/global.xml`

---

## 🔗 Endpoints Relacionados

- **GET /technical_info_e** - Datos técnicos de electricidad
- **GET /invoices** - Consulta de facturas
- **POST /ejecutarQuery** - Consultas dinámicas
- **POST /ejecutarSecureQuery** - Consultas parametrizadas

---

## 📌 Notas Importantes

### Características Especiales

**Tarifas de Gas:**
- **RL.1:** Tarifa de último recurso 1 (consumo < 5.000 kWh/año)
- **RL.2:** Tarifa de último recurso 2 (consumo entre 5.000 y 50.000 kWh/año)
- **RL.3:** Tarifa de último recurso 3 (consumo entre 50.000 y 100.000 kWh/año)
- **RL.4:** Tarifa de último recurso 4 (consumo > 100.000 kWh/año)

**Presiones de Suministro:**
- **BP (Baja Presión):** < 0,05 bar - Uso doméstico y pequeño comercio
- **MP (Media Presión):** 0,05 - 4 bar - Comercios e industria ligera
- **AP (Alta Presión):** > 4 bar - Gran industria

**Distribuidoras de Gas Comunes:**
- **NGD:** Naturgy Distribución (antes Gas Natural Distribución)
- **RED:** Redexis Gas
- **MAD:** Madrileña Red de Gas
- **NOR:** Nortegas

### Recomendaciones de Uso

**Mejores Prácticas:**
- ✅ Validar formato de CUPS antes de llamar al endpoint
- ✅ Cachear resultados (datos técnicos cambian poco)
- ✅ Manejar caso de CUPS no encontrado

**Performance:**
- ✅ Query optimizada con JOINs eficientes
- ✅ Respuesta rápida (un solo registro)
- ✅ Schema dinámico configurable

---

**Última actualización:** 2026-06-17  
**Versión del documento:** 2.0  
**Autor:** Equipo MuleSoft Naturgy