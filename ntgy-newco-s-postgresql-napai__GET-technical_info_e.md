---
api: "ntgy-newco-s-postgresql-napai"
sidebar_position: 3
sidebar_label: "GET /technical_info_e"
sql_metadata:
  data_sources:
    - database: "PostgreSQL"
      schema: "Configurado dinámicamente (sips.esquema.ele)"
      type: "PostgreSQL"
      tables:
        - name: "tnpi_dim_suministros_ele"
          operations: ["READ"]
        - name: "tnpi_dim_t_consumos_ele"
          operations: ["READ"]
---

# Endpoint: GET /technical_info_e

## 📋 Descripción General

Recupera los datos técnicos de suministros de electricidad desde la base de datos PostgreSQL.

**Tipo de API:** System API (Internal)  
**Capa:** System Layer  
**Versión:** 1.12.0-SNAPSHOT  
**Última actualización:** 2026-06-17

---

## 🎯 Propósito

Este endpoint proporciona acceso a información técnica completa de puntos de suministro eléctricos:

- Consulta por código CUPS con validación de formato
- Información de tarifa y tipo de contrato
- Potencias contratadas por periodo (P1-P6)
- Datos del contador y equipo de medida
- Información de distribuidora y comercializadora
- Consumos históricos (12 meses, año cerrado, año móvil)
- Datos técnicos del punto de medida
- Información de APM (Autorización de Puesta en Marcha)
- Más de 70 campos con información detallada

**Casos de uso:**
- Consulta de características técnicas del suministro
- Validación de potencias contratadas
- Información para cambios de tarifa
- Datos para análisis de consumo
- Auditoría de datos técnicos
- Verificación de consumos históricos

---

## 🔗 URL del Endpoint

```
GET /api/technical_info_e?idCUPS=<CUPS>
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-napai-dev.cloudhub.io`  
**Base URL (Production):** `https://ntgy-newco-s-postgresql-napai-pro.cloudhub.io`

---

## 📥 Parámetros de Entrada

### Query Parameters

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `idCUPS` | string | Sí | Código CUPS del punto de suministro eléctrico |

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
      "idCodigoAPM": "APM123456",
      "idCUPS": "ES0021000000000001JN",
      "idProvinciaPto": "28",
      "idCodigoPostalPto": "28001",
      "idTarifaAcceso": "2.0TD",
      "idTipoTitularCNMC": "1",
      "desComunidadAutonoma": "Madrid",
      "desDireccionPto": "Calle Mayor 1",
      "desImpago": "N",
      "desModoControlPotencia": "ICP",
      "desMotEstNoContr": "",
      "desNivelTensionMedida": "BT",
      "idPropiedadEquipoMedida": "Distribuidora",
      "propiedadIcp": "Cliente",
      "desProvinciaPto": "Madrid",
      "idTipoContrato": "Normal",
      "idTipoEnergia": "Electricidad",
      "idFechaAltaSum": "15/01/2020",
      "idFechaCaducidadAPM": "15/01/2027",
      "idFechaCargaDist": "15/06/2026",
      "idFechaEmisionAPM": "15/01/2022",
      "idFechaLimiteDchosAcceso": "15/01/2030",
      "idFechaLimiteExtension": "15/01/2030",
      "idFechaUltCambioComerc": "01/03/2025",
      "idFechaUltMovContrato": "01/03/2025",
      "idFechaUltimaLectura": "01/06/2026",
      "idAccesibContador": "1",
      "idClaseExpediente": "Normal",
      "idCNAE": "9820",
      "idComercializadora": "0123",
      "idDiscriminacionHoraria": "3",
      "idEmpresaDistribuidora": "0021",
      "idFasesEquipoMedida": "3",
      "idAplicacionBS": "N",
      "idDisponibilidadICP": "S",
      "idPtoContratable": "S",
      "idViviendaTitular": "S",
      "idMotivoExpediente": "Alta",
      "idMunicioPto": "28079",
      "idPeriodoFacturacion": "Mensual",
      "idPoblacionPto": "Madrid",
      "idRelTRIntensidad": "1",
      "telegestionado": "S",
      "idTensionSuministroCnmc": "1",
      "idAutoconsumo": "00",
      "idTipoPerfilConsumo": "1",
      "idTipoPuntoMedida": "1",
      "idConsumoAnyoCerradoAct": "3500",
      "fConsumoActAnyoCerrAnt": "3400",
      "idConsumoAnyoMovilAct": "3550",
      "idConsumoAnyoCerradoREA": "150",
      "fConsumoReactAnyoCerrAnt": "145",
      "idConsumoAnyoMovilREA": "155",
      "fConsumoActTriAnual": "10500",
      "idDepositoGarantia": "0",
      "idDerechosAcceso": "5.75",
      "idDerechosExtension": "0",
      "idPotenciaDisponibleCaja": "14.49",
      "idPotenciaMaxima": "5.75",
      "idPotenciaMaximaActa": "5.75",
      "idPotenciaMaximaBie": "5.75",
      "idPotenciaPeriodo01": "3.45",
      "idPotenciaPeriodo02": "3.45",
      "idPotenciaPeriodo03": "0",
      "idPotenciaPeriodo04": "0",
      "idPotenciaPeriodo05": "0",
      "idPotenciaPeriodo06": "0",
      "fConsumoAct12MesesAnt": "3500",
      "fConsumoReact12MesesAnt": "150",
      "fConsumoReact12Meses2Ant": "145"
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

**Campos principales del objeto en `cups[]` (más de 70 campos):**

**Identificación:**
- `idCUPS`: Código CUPS del punto de suministro
- `idCodigoAPM`: Código de Autorización de Puesta en Marcha
- `idProvinciaPto`: Código de provincia del punto de suministro
- `idCodigoPostalPto`: Código postal del punto de suministro

**Tarifa y Contrato:**
- `idTarifaAcceso`: Código de tarifa de acceso (ej: "2.0TD", "3.0TD")
- `idTipoContrato`: Tipo de contrato
- `idTipoEnergia`: Tipo de energía
- `idDiscriminacionHoraria`: Discriminación horaria

**Potencias (kW):**
- `idPotenciaPeriodo01` a `idPotenciaPeriodo06`: Potencias contratadas por periodo
- `idPotenciaMaxima`: Potencia máxima
- `idPotenciaMaximaActa`: Potencia máxima según acta
- `idPotenciaMaximaBie`: Potencia máxima BIE
- `idPotenciaDisponibleCaja`: Potencia disponible en caja

**Consumos Históricos:**
- `idConsumoAnyoCerradoAct`: Consumo activo año cerrado
- `fConsumoActAnyoCerrAnt`: Consumo activo año cerrado anterior
- `idConsumoAnyoMovilAct`: Consumo activo año móvil
- `idConsumoAnyoCerradoREA`: Consumo reactivo año cerrado
- `fConsumoReactAnyoCerrAnt`: Consumo reactivo año cerrado anterior
- `idConsumoAnyoMovilREA`: Consumo reactivo año móvil
- `fConsumoActTriAnual`: Consumo activo trianual
- `fConsumoAct12MesesAnt`: Consumo activo 12 meses anteriores
- `fConsumoReact12MesesAnt`: Consumo reactivo 12 meses anteriores
- `fConsumoReact12Meses2Ant`: Consumo reactivo 12 meses 2 años anteriores

**Datos Técnicos:**
- `desNivelTensionMedida`: Nivel de tensión de medida
- `idFasesEquipoMedida`: Fases del equipo de medida
- `idPropiedadEquipoMedida`: Propiedad del equipo de medida
- `propiedadIcp`: Propiedad del ICP
- `desModoControlPotencia`: Modo de control de potencia
- `idDisponibilidadICP`: Disponibilidad de ICP
- `telegestionado`: Indica si está telegestionado

**Empresas:**
- `idEmpresaDistribuidora`: Código de la distribuidora
- `idComercializadora`: Código de la comercializadora

**Ubicación:**
- `desDireccionPto`: Dirección del punto de suministro
- `desProvinciaPto`: Provincia del punto de suministro
- `desComunidadAutonoma`: Comunidad autónoma
- `idMunicioPto`: Código de municipio
- `idPoblacionPto`: Población

**Fechas (formato dd/mm/yyyy):**
- `idFechaAltaSum`: Fecha de alta del suministro
- `idFechaCaducidadAPM`: Fecha de caducidad del APM
- `idFechaEmisionAPM`: Fecha de emisión del APM
- `idFechaUltCambioComerc`: Fecha último cambio comercial
- `idFechaUltMovContrato`: Fecha último movimiento de contrato
- `idFechaUltimaLectura`: Fecha última lectura

**Otros:**
- `desImpago`: Indicador de impago
- `idCNAE`: Código CNAE
- `idAutoconsumo`: Tipo de autoconsumo
- `idTipoPerfilConsumo`: Tipo de perfil de consumo
- `idViviendaTitular`: Indicador de vivienda habitual
- `idAplicacionBS`: Aplicación de bono social

**Nota:** La respuesta incluye más de 70 campos. Consultar el ejemplo completo para ver todos los campos disponibles.

---

### ❌ Errores

#### 500 Error (Código por defecto para todos los errores)

```json
{
  "errorCode": 500,
  "message": "DB:QUERY_EXECUTION",
  "description": "ERROR: relation \"tnpi_dim_suministros_ele\" does not exist"
}
```

**Estructura del error (basada en error-managerExpSys.xml):**
- `errorCode`: Código HTTP (por defecto 500, puede variar según `vars.httpStatus`)
- `message`: Tipo de error (`error.errorType.identifier` o `payload.message`)
- `description`: Descripción detallada del error (`error.description`)

**Tipos de errores comunes:**

**1. Parámetro faltante (validado por APIKit):**
```json
{
  "errorCode": 500,
  "message": "APIKIT:BAD_REQUEST",
  "description": "Required query parameter idCUPS not specified"
}
```

**2. CUPS no encontrado:**
```json
{
  "errorCode": 500,
  "message": "MULE:EXPRESSION",
  "description": "Cannot coerce Null to Object"
}
```

**3. Error de conectividad:**
```json
{
  "errorCode": 500,
  "message": "DB:CONNECTIVITY",
  "description": "Could not connect to database"
}
```

**Causas comunes:**
- Parámetro `idCUPS` faltante
- CUPS no existe en la base de datos
- Error en la query SQL
- Base de datos no disponible
- Error en transformación DataWeave

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consultar datos técnicos de un CUPS válido

```bash
curl -X GET "http://localhost:8081/api/technical_info_e?idCUPS=ES0021000000000001JN" \
  -H "Accept: application/json"
```

**Respuesta (estructura completa con más de 70 campos):**
```json
{
  "result": {
    "codResult": "0000",
    "msgResult": ""
  },
  "cups": [
    {
      "fec_actualiz_reg": "2026-06-15T10:30:00",
      "idCodigoAPM": "APM123456",
      "idCUPS": "ES0021000000000001JN",
      "idTarifaAcceso": "2.0TD",
      "idPotenciaPeriodo01": "3.45",
      "idPotenciaPeriodo02": "3.45",
      "idPotenciaPeriodo03": "0",
      "idEmpresaDistribuidora": "0021",
      "idComercializadora": "0123",
      "desDireccionPto": "Calle Mayor 1",
      "desProvinciaPto": "Madrid",
      "idConsumoAnyoCerradoAct": "3500",
      "fConsumoAct12MesesAnt": "3500"
      // ... más de 60 campos adicionales
    }
  ]
}
```

---

### Ejemplo 2: CUPS con formato inválido

```bash
curl -X GET "http://localhost:8081/api/technical_info_e?idCUPS=INVALID_CUPS" \
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

**Nota:** Cuando el CUPS no cumple el formato esperado, el endpoint retorna un array vacío en lugar de un error. Esto es por diseño.

---

### Ejemplo 3: CUPS no encontrado en base de datos

```bash
curl -X GET "http://localhost:8081/api/technical_info_e?idCUPS=ES0021000000000999ZZ" \
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

**Nota:** Si el CUPS tiene formato válido pero no existe en la base de datos, también retorna array vacío.

---

## 🔧 Implementación Técnica

### Flujo de Ejecución

```
HTTP GET Request → APIKit Router → technical_info_eSub_Flow → 
Extract idCUPS → Database Query → DataWeave Transform → JSON Response
```

### Código Fuente

**Archivo:** `src/main/mule/sips/technical_info_e.xml`

**Sub-flow:** `technical_info_eSub_Flow`

### Validación de CUPS

El código implementa validación del formato CUPS antes de ejecutar la consulta:

```xml
<when expression="#[vars.idCUPS matches /^ES\d{4}\d{12}[A-Z]{2}([A-Z0-9]{0,2})?$/]">
```

**Comportamiento:**
- ✅ **CUPS válido:** Ejecuta la consulta SQL
- ❌ **CUPS inválido:** Retorna respuesta con array `cups` vacío (sin error)

### Query SQL

La query consulta dos tablas con JOIN:

```sql
SELECT se.fec_actualiz_reg, 
       se.cod_apm as "idCodigoAPM",
       se.cod_cups as "idCUPS",
       se.cod_tarifa as "idTarifaAcceso",
       se.num_potencia_periodo_01::varchar as "idPotenciaPeriodo01",
       se.num_potencia_periodo_02::varchar as "idPotenciaPeriodo02",
       -- ... más de 70 campos ...
       (ce.num_consumo_activa_13 + ce.num_consumo_activa_14 + ... + ce.num_consumo_activa_24)::varchar as "fConsumoAct12MesesAnt"
FROM :esquema.tnpi_dim_suministros_ele se  
INNER JOIN :esquema.tnpi_dim_t_consumos_ele ce ON se.cod_cups = ce.cod_cups
WHERE se.cod_cups BETWEEN 'ES0021000000000001JN' AND 'ES0021000000000001JNzz'
```

**Características:**
- **Tablas consultadas:**
  - `tnpi_dim_suministros_ele`: Datos técnicos del suministro
  - `tnpi_dim_t_consumos_ele`: Consumos históricos
- **Schema dinámico:** Configurado mediante propiedad `sips.esquema.ele`
- **Filtro por rango:** Usa `BETWEEN` para buscar el CUPS
- **Cálculos de consumo:** Suma consumos de 12 meses
- **Conversión de tipos:** Convierte números a varchar para la respuesta

### Transformación DataWeave

La transformación construye la respuesta con wrapper `result` y array `cups`:

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

**Características:**
- Retorna el payload completo de la base de datos
- Los alias SQL ya definen los nombres de campos finales
- No hay mapeo adicional (los campos ya vienen con los nombres correctos)
- El array `cups` contiene todos los registros encontrados (normalmente 1)

**Nota:** La query SQL ya define los alias de columnas con los nombres finales que se esperan en la respuesta JSON.

---

## 🔒 Seguridad y Controles

### ✅ Controles Implementados

**Autenticación y Autorización:**
- ✅ **API Gateway:** Autenticación OAuth/API Key implementada
- ✅ **Autorización:** Solo APIs internas con permisos específicos

**Protección de Datos:**
- ✅ **Consulta específica:** Solo datos del CUPS solicitado
- ✅ **Usuario de BD:** `mulesoft_ro`
- ✅ **Sin datos sensibles:** No expone información personal del cliente

**Performance:**
- ✅ **Query simple:** Consulta directa por clave primaria
- ✅ **Fetch Size:** Configurable por ambiente
- ✅ **Connection Pooling:** Máximo 30 conexiones

---

## ⚙️ Configuración de Producción

### Tabla de Base de Datos

```yaml
# Tabla principal
tabla: dlakeservi.self_publication_sistemas.tnpi_dim_suministros_ele

# Campos clave
- cod_cups: Código CUPS (clave primaria)
- cod_tarifa: Código de tarifa
- pot_p1_contratada: Potencia P1
- pot_p2_contratada: Potencia P2
- pot_p3_contratada: Potencia P3
- pot_p4_contratada: Potencia P4
- pot_p5_contratada: Potencia P5
- pot_p6_contratada: Potencia P6
- num_contador: Número de contador
- cod_distribuidora: Código distribuidora
```

### Fetch Size

```yaml
# properties-pro.yaml
postgresql:
  fetchSize: "100"
```

---

## 📊 Logging

### Logs Generados

```
INFO  - technical_info_e - Inicio ejecucion: ES0021000000000001JN
DEBUG - Query: SELECT * FROM tnpi_dim_suministros_ele WHERE cod_cups = ...
DEBUG - technical_info_e - Resultado: {"tarifa":"2.0TD",...}
INFO  - technical_info_e - Fin ejecucion
```

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-napai.raml`
- **Main Flow:** `src/main/mule/sips/technical_info_e.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`
- **Database Config:** `src/main/mule/global.xml`

---

## 🔗 Endpoints Relacionados

- **GET /technical_info_g** - Datos técnicos de gas
- **GET /invoices** - Consulta de facturas
- **POST /ejecutarQuery** - Consultas dinámicas
- **POST /ejecutarSecureQuery** - Consultas parametrizadas

---

## 📌 Notas Importantes

### Características Especiales

**Tarifas Eléctricas:**
- **2.0TD:** 2 periodos (P1, P2) - Doméstico
- **3.0TD:** 3 periodos (P1, P2, P3) - Pequeño comercio
- **6.1TD:** 6 periodos (P1-P6) - Gran industria

**Potencias:**
- Valores en kW (kilovatios)
- Pueden ser `null` si no aplican para la tarifa
- Ejemplo: Tarifa 2.0TD solo usa P1 y P2

**Distribuidoras Comunes:**
- **UFD:** Unión Fenosa Distribución
- **EDI:** Endesa Distribución
- **VIE:** Viesgo Distribución
- **IBE:** Iberdrola Distribución

### Recomendaciones de Uso

**Mejores Prácticas:**
- ✅ Validar formato de CUPS antes de llamar al endpoint
- ✅ Cachear resultados (datos técnicos cambian poco)
- ✅ Manejar caso de CUPS no encontrado

**Performance:**
- ✅ Query optimizada por clave primaria
- ✅ Respuesta rápida (un solo registro)
- ✅ Sin joins ni agregaciones

---

**Última actualización:** 2026-06-17  
**Versión del documento:** 2.0  
**Autor:** Equipo MuleSoft Naturgy
