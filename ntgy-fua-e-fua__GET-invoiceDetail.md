---
api: "ntgy-fua-e-fua"
sidebar_position: 23
sidebar_label: "GET /invoiceDetail"
description: "Obtiene el detalle completo de una factura para canales web y móvil"
keywords: [factura, invoice, detalle, experience api, web, mobile, billing]
tags: [Billing, Experience API, Web, Mobile]
channel: web
endpoint_dependencies:
  - method: "GET"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/invoiceDetail"
    purpose: "Orquestación de consulta de detalle de factura"
---

# GET /invoiceDetail

## 📌 Descripción General

### Propósito del Endpoint

Obtiene el detalle completo de una factura específica adaptado para canales web y móvil. Este Experience API proporciona acceso optimizado a la información de facturación para aplicaciones de autoservicio de clientes.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para canales web y móvil

### Patrón de Integración

**Tipo:** Pass-Through (sin transformación)

**Descripción:** Este Experience API actúa como pass-through puro hacia el Process API, delegando la consulta sin aplicar transformaciones específicas del canal.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente<br/>Web/Mobile]
    E[Experience API<br/>ntgy-fua-e-fua<br/>GET /invoiceDetail]
    P[Process API<br/>ntgy-newco-p-fua-evo]
    S[System API<br/>ntgy-newco-s-sap-isu]
    SAP[SAP ISU<br/>RFC: ZFA_FM_FAC_INTERACTIVA]
    
    C -->|Request| E
    E -->|Pass-Through| P
    P -->|Pass-Through| S
    S -->|RFC Call| SAP
    SAP -->|Response| S
    S -->|JSON| P
    P -->|JSON| E
    E -->|Adapted Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style SAP fill:#fff3e0,stroke:#e65100,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Consulta de Factura en Portal Web**
   - Cliente accede a su área personal
   - Visualiza el detalle completo de una factura específica
   - Descarga o imprime la información de facturación

2. **Consulta de Factura en App Móvil**
   - Cliente consulta facturas desde dispositivo móvil
   - Accede a desglose de consumos y cargos
   - Verifica fechas de emisión y cargo

3. **Soporte al Cliente**
   - Agentes consultan detalles de factura para atención al cliente
   - Verifican información de facturación en tiempo real
   - Resuelven consultas sobre cargos y conceptos

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Web | Mobile

**Características del Canal:**
- **Plataforma**: Navegadores web modernos, aplicaciones móviles iOS/Android
- **Dispositivos**: Desktop, tablets, smartphones
- **Conectividad**: Requiere conexión a internet estable
- **Experiencia de Usuario**: Interfaz responsive y optimizada para autoservicio

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Consulta de facturas de energía (electricidad/gas)
- **Clientes Empresariales**: Acceso a información de facturación corporativa
- **Agentes de Atención**: Soporte y resolución de consultas de facturación

**Contexto de Uso:**
- **Autoservicio**: Cliente consulta sus facturas sin intervención de agente
- **Atención al Cliente**: Agente verifica información durante llamada o chat
- **Gestión Financiera**: Cliente revisa y planifica pagos de facturas

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Formato de Respuesta**: JSON optimizado para consumo web/móvil
- **Headers Personalizados**: Incluye `x-client-code` para identificación del cliente
- **Timeout Optimizado**: 60 segundos para garantizar respuesta en conexiones móviles

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/invoiceDetail` | GET | Orquestación de consulta de detalle de factura desde SAP ISU |

### Contexto Empresarial

- **Dominio:** Autoservicio de Clientes - Facturación
- **Sector:** Energía - Utilities
- **División:** Naturgy España - FUA (Frontal Único de Atención)
- **Canal:** Web/Móvil

### Stakeholders

- **Clientes Finales**: Usuarios del portal web y app móvil
- **Equipo de Producto Digital**: Responsables de la experiencia de usuario
- **Desarrollo Frontend**: Consumidores del API para interfaces web/móvil
- **Atención al Cliente**: Usuarios de herramientas de soporte

### Valor de Negocio

- **Autoservicio**: Reduce carga en call center mediante consulta autónoma
- **Transparencia**: Información detallada y accesible para clientes
- **Satisfacción**: Mejora experiencia del cliente con acceso 24/7
- **Eficiencia**: Reduce tiempo de resolución de consultas de facturación

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/invoiceDetail` |
| **Development** | `https://ntgy-fua-e-fua-dev.us-e2.cloudhub.io` | `https://ntgy-fua-e-fua-dev.us-e2.cloudhub.io/fua/api/invoiceDetail` |
| **UAT** | `https://ntgy-fua-e-fua-uat.us-e2.cloudhub.io` | `https://ntgy-fua-e-fua-uat.us-e2.cloudhub.io/fua/api/invoiceDetail` |
| **Production** | `https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io` | `https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/invoiceDetail` |

### Método HTTP

```
GET /fua/api/invoiceDetail
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Optimizado para conexiones móviles y consultas complejas |
| **Rate Limiting** | 100 requests/minuto por cliente | Prevención de abuso y garantía de disponibilidad |
| **Autenticación** | OAuth 2.0 requerido | Seguridad y trazabilidad de accesos |

### Parámetros de Entrada

#### Query Parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `invoiceNum` | String | ✅ Sí | Número de factura a consultar | `"1234567890"` |

#### Headers Requeridos

| Header | Tipo | Requerido | Descripción | Ejemplo |
|--------|------|-----------|-------------|---------|
| `x-client-code` | String | ✅ Sí | Código de identificación del cliente (NIF/NIE) | `"12345678A"` |
| `Authorization` | String | ✅ Sí | Token de autenticación OAuth 2.0 | `"Bearer eyJhbGc..."` |

**Ejemplo de URL:**
```
GET /fua/api/invoiceDetail?invoiceNum=1234567890
Headers:
  x-client-code: 12345678A
  Authorization: Bearer eyJhbGc...
```

### Respuesta Exitosa (200 OK)

⚠️ **IMPORTANTE - Pass-Through API**

Este Experience API actúa como **pass-through puro** hacia el Process API `ntgy-newco-p-fua-evo` sin aplicar transformaciones.

**Schema completo documentado aquí como referencia para desarrolladores del canal web/móvil.**

📖 **Cadena de Llamadas:**
- Experience API (este) → Process API → System API (SOURCE OF TRUTH)

#### Estructura de Respuesta

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Operación exitosa"
  },
  "detalleFactura": {
    "cuentaContrato": "200012345678",
    "fecEmision": "15.01.2024",
    "fecCargo": "20.01.2024",
    "contactos": {
      "contacAtc": "912345678",
      "contacEle": "info@example.com",
      "contacGas": "912345679",
      "contacSer": "servicio@example.com"
    },
    "financiaciones": {
      "desgloseFin": [
        {
          "codProd": "ELEC",
          "impFin": "50,00"
        }
      ],
      "impTotFin": "50,00"
    },
    "impTotFactura": "150,50",
    "impTotImpuestos": "31,61",
    "invoice_amount": "150,50",
    "invoice_discharge": "0,00",
    "invoice_discharge02": "0,00",
    "current_total": "150,50",
    "notificaciones": {
      "desgloseNot": [
        {
          "codMensaje": "INFO001",
          "info": "Factura emitida correctamente"
        }
      ]
    },
    "numFactura": "1234567890",
    "producto": {
      "codProd": "ELEC",
      "consAcumulado": "1500",
      "consLimite": "2000",
      "consPeriodo": "500",
      "fechaFin": "31.01.2024",
      "fechaFinTP": "31.01.2024",
      "fechaInicio": "01.01.2024",
      "fechaInicioTP": "01.01.2024",
      "impBrutoEne": "100,00",
      "impBrutoEneSin": "95,00",
      "impImpuestoEsp": "5,00",
      "impImpuestoInd": "21,00",
      "impReguladoAlq": "10,00",
      "impReguladoPea": "5,00",
      "impTotEne": "100,00",
      "impTotImp": "26,00",
      "impTotProd": "126,00",
      "impTotReg": "15,00",
      "tipoSubSec": "DOM",
      "indECO": "X",
      "indTP": "X",
      "numDias": "31",
      "sector": "ELEC"
    },
    "promociones": {
      "desgloseProm": [
        {
          "impProm": "10,00",
          "sector": "ELEC"
        }
      ],
      "impTotProm": "10,00"
    },
    "servicios": {
      "desgloseSer": [
        {
          "codProd": "MANT",
          "impBruto": "20,00",
          "impDescuento": "2,00"
        }
      ],
      "impImpuestoInd": "3,78",
      "impTotBrutoSer": "20,00",
      "impTotBrutoSerSin": "18,00",
      "impTotSer": "21,78"
    },
    "tipoImpuesto": "IVA",
    "ALIANZ_EXISTE": true
  }
}
```

#### Campos de la Respuesta

📖 **Para descripción detallada de cada campo, consultar:**
- **Process API:** `ntgy-newco-p-fua-evo` endpoint `/invoiceDetail`
- **System API (SOURCE OF TRUTH):** `ntgy-newco-s-sap-isu` endpoint `/invoiceDetail`

**Resumen de Estructura:**

| Sección | Descripción | Uso en UI |
|---------|-------------|-----------|
| `result` | Resultado de la operación | Mostrar mensaje de éxito/error |
| `detalleFactura.cuentaContrato` | Número de cuenta contrato | Identificación del contrato |
| `detalleFactura.fecEmision` | Fecha de emisión | Fecha de generación de factura |
| `detalleFactura.fecCargo` | Fecha de cargo | Fecha de cobro |
| `detalleFactura.impTotFactura` | Importe total | Monto principal a pagar |
| `detalleFactura.numFactura` | Número de factura | Identificador único |
| `detalleFactura.producto` | Detalle de producto energético | Desglose de consumos |
| `detalleFactura.servicios` | Servicios contratados | Servicios adicionales |
| `detalleFactura.promociones` | Promociones aplicadas | Descuentos y ofertas |

### Respuestas de Error

#### Error 400 - Bad Request

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Parámetro requerido faltante: invoiceNum"
  }
}
```

#### Error 401 - Unauthorized

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Token de autenticación inválido o expirado"
  }
}
```

#### Error 500 - Internal Server Error

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al consultar Process API: Connection timeout"
  }
}
```

### Autenticación

- **Tipo:** OAuth 2.0 Bearer Token
- **Header:** `Authorization: Bearer {token}`
- **Scope:** `read:invoices`

### Timeouts

| Tipo | Valor | Justificación |
|------|-------|---------------|
| **Request Timeout** | 60 segundos | Optimizado para consultas complejas en móvil |
| **Connection Timeout** | 30 segundos | Tiempo de establecimiento de conexión |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP GET Request<br/>Canal Web/Mobile]) --> SetVar[Set Variable: invoiceNumber]
    SetVar --> CallProcess[HTTP Request to Process API<br/>ntgy-newco-p-fua-evo]
    CallProcess --> End([HTTP Response<br/>Pass-Through])
    
    CallProcess -->|Error| ErrorHandler[Error Handler]
    ErrorHandler --> ErrorTransform[Transform Error Message]
    ErrorTransform --> ErrorLog[Log Error]
    ErrorLog --> ErrorResponse([Error Response])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#e1f5ff,stroke:#0277bd
    style CallProcess fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style ErrorHandler fill:#ffebee,stroke:#c62828
```

### Transformación de Datos

⚠️ **Este Experience API NO transforma la respuesta del Process API.**

**Patrón:** Pass-Through Puro

**Flujo de Datos:**
1. Recibe request del cliente web/móvil
2. Extrae `invoiceNum` del query parameter
3. Extrae `clientId` del header `x-client-code`
4. Llama al Process API con los parámetros mapeados
5. Retorna la respuesta del Process API sin modificaciones

**Código del Flow:**

```xml
<flow name="get:\invoiceDetail:e-fua-config">
    <set-variable value="#[attributes.queryParams.invoiceNum]" 
                  variableName="invoiceNumber" />
    <flow-ref name="get-invoice-detail" />
</flow>

<flow name="get-invoice-detail">
    <http:request method="GET" 
                  config-ref="HTTP_FUA_PROCESS_NEWCO_EVO" 
                  path="/invoiceDetail">
        <http:query-params>
            {
                "invoiceNumber": vars.invoiceNumber default "",
                "clientId": attributes.headers['x-client-code']
            } ++ vars.fuaHeaders
        </http:query-params>
    </http:request>
    <error-handler>
        <on-error-propagate>
            <ee:transform>
                <ee:set-payload>
                    {
                        "result": {
                            "codResult": "0001",
                            "msgResult": error.description
                        }
                    }
                </ee:set-payload>
            </ee:transform>
            <logger level="ERROR" message="Error in get-invoice-detail"/>
        </on-error-propagate>
    </error-handler>
</flow>
```

### Configuración de Integración

**HTTP Request Configuration:**

| Propiedad | Valor | Descripción |
|-----------|-------|-------------|
| **Config Ref** | `HTTP_FUA_PROCESS_NEWCO_EVO` | Configuración de conexión al Process API |
| **Method** | `GET` | Método HTTP |
| **Path** | `/invoiceDetail` | Path del endpoint |
| **Timeout** | 60000 ms | Timeout de request |

**Mapeo de Parámetros:**

| Parámetro Experience API | Parámetro Process API | Transformación |
|--------------------------|----------------------|----------------|
| `invoiceNum` (query) | `invoiceNumber` (query) | Renombrado |
| `x-client-code` (header) | `clientId` (query) | Header → Query param |

### Patrones de Diseño

**Patrón:** Pass-Through con Mapeo de Parámetros

**Justificación:**
- **Simplicidad**: Mínima lógica de transformación
- **Flexibilidad**: Permite cambios en Process API sin impactar clientes
- **Adaptación de Canal**: Mapeo de headers a query params para Process API
- **Monitoreo**: Logging específico del canal

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente Web/Mobile
    participant E as Experience API
    participant P as Process API
    participant S as System API
    participant SAP as SAP ISU
    
    C->>E: GET /invoiceDetail?invoiceNum=X<br/>Header: x-client-code=Y
    Note over E: Map Parameters<br/>invoiceNum → invoiceNumber<br/>x-client-code → clientId
    E->>P: GET /invoiceDetail?invoiceNumber=X&clientId=Y
    P->>S: GET /invoiceDetail?invoiceNumber=X&clientId=Y
    S->>SAP: RFC ZFA_FM_FAC_INTERACTIVA
    SAP-->>S: RFC Response
    S-->>P: 200 OK (JSON)
    P-->>E: 200 OK (Pass-Through)
    E-->>C: 200 OK (Pass-Through)
```

#### 2. Flujo con Error de Negocio

```mermaid
sequenceDiagram
    participant C as Cliente Web/Mobile
    participant E as Experience API
    participant P as Process API
    
    C->>E: GET /invoiceDetail?invoiceNum=INVALID
    E->>P: GET /invoiceDetail?invoiceNumber=INVALID
    P-->>E: 200 OK (codResult: "0001")
    Note over E: Log Error
    E-->>C: 200 OK (Pass-Through Error)
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant C as Cliente Web/Mobile
    participant E as Experience API
    participant P as Process API
    
    C->>E: GET /invoiceDetail?invoiceNum=X
    E->>P: GET /invoiceDetail
    P--xE: Connection Timeout
    Note over E: Error Handler<br/>Transform Error
    E->>E: Log Error
    E-->>C: 500 Internal Server Error
```

### Manejo de Errores

**Error Handler:**

```xml
<error-handler>
    <on-error-propagate enableNotifications="true" logException="true">
        <ee:transform doc:name="Set Error Message">
            <ee:set-payload>
                {
                    "result": {
                        "codResult": "0001",
                        "msgResult": error.description as String replace /\"/ with("")
                    }
                }
            </ee:set-payload>
        </ee:transform>
        <logger level="ERROR" message="Error in get-invoice-detail"/>
    </on-error-propagate>
</error-handler>
```

**Tipos de Errores Manejados:**

1. **Errores de Autenticación:** Token inválido o expirado
2. **Errores de Validación:** Parámetros faltantes o inválidos
3. **Errores de Conexión:** Timeout o fallo de conexión con Process API
4. **Errores del Process API:** Propagados sin modificación

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Consulta Exitosa desde Web

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/invoiceDetail?invoiceNum=1234567890" \
  -H "x-client-code: 12345678A" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "Consulta exitosa"
  },
  "detalleFactura": {
    "cuentaContrato": "200012345678",
    "fecEmision": "15.01.2024",
    "fecCargo": "20.01.2024",
    "impTotFactura": "150,50",
    "impTotImpuestos": "31,61",
    "numFactura": "1234567890",
    "producto": {
      "codProd": "ELEC",
      "consAcumulado": "1500",
      "consPeriodo": "500",
      "impTotProd": "126,00",
      "sector": "ELEC"
    },
    "tipoImpuesto": "IVA",
    "ALIANZ_EXISTE": true
  }
}
```

### Ejemplo 2: Factura No Encontrada

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/invoiceDetail?invoiceNum=9999999999" \
  -H "x-client-code: 12345678A" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### Respuesta (200 OK - Error de Negocio)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Factura no encontrada en SAP ISU"
  }
}
```

### Ejemplo 3: Error de Autenticación

#### cURL

```bash
curl -X GET "https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/invoiceDetail?invoiceNum=1234567890" \
  -H "x-client-code: 12345678A" \
  -H "Authorization: Bearer INVALID_TOKEN"
```

#### Respuesta (401 Unauthorized)

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Token de autenticación inválido o expirado"
  }
}
```

### Ejemplo 4: JavaScript/TypeScript (React/Angular)

```javascript
const getInvoiceDetail = async (invoiceNum, clientCode, authToken) => {
  try {
    const response = await fetch(
      `https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io/fua/api/invoiceDetail?invoiceNum=${invoiceNum}`,
      {
        method: 'GET',
        headers: {
          'x-client-code': clientCode,
          'Authorization': `Bearer ${authToken}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    const data = await response.json();
    
    if (data.result.codResult === '0000') {
      console.log('Detalle de factura:', data.detalleFactura);
      return {
        success: true,
        invoice: data.detalleFactura
      };
    } else {
      console.error('Error de negocio:', data.result.msgResult);
      return {
        success: false,
        error: data.result.msgResult
      };
    }
  } catch (error) {
    console.error('Error de conexión:', error);
    return {
      success: false,
      error: 'Error al conectar con el servidor'
    };
  }
};

// Uso en componente React
const InvoiceDetailComponent = () => {
  const [invoice, setInvoice] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const loadInvoiceDetail = async (invoiceNum) => {
    setLoading(true);
    const result = await getInvoiceDetail(
      invoiceNum,
      '12345678A',
      localStorage.getItem('authToken')
    );
    
    if (result.success) {
      setInvoice(result.invoice);
      setError(null);
    } else {
      setError(result.error);
      setInvoice(null);
    }
    setLoading(false);
  };
  
  return (
    <div>
      {loading && <p>Cargando...</p>}
      {error && <p>Error: {error}</p>}
      {invoice && (
        <div>
          <h2>Factura {invoice.numFactura}</h2>
          <p>Importe Total: {invoice.impTotFactura} €</p>
          <p>Fecha de Emisión: {invoice.fecEmision}</p>
        </div>
      )}
    </div>
  );
};
```

### Ejemplo 5: Python (para testing o scripts)

```python
import requests

class InvoiceAPIClient:
    def __init__(self, base_url, client_code, auth_token):
        self.base_url = base_url
        self.client_code = client_code
        self.auth_token = auth_token
    
    def get_invoice_detail(self, invoice_num):
        """
        Obtiene el detalle de una factura desde el Experience API
        """
        url = f"{self.base_url}/fua/api/invoiceDetail"
        
        params = {
            "invoiceNum": invoice_num
        }
        
        headers = {
            "x-client-code": self.client_code,
            "Authorization": f"Bearer {self.auth_token}",
            "Content-Type": "application/json"
        }
        
        try:
            response = requests.get(
                url, 
                params=params, 
                headers=headers, 
                timeout=60
            )
            response.raise_for_status()
            
            data = response.json()
            
            if data['result']['codResult'] == '0000':
                print(f"✓ Factura obtenida: {data['detalleFactura']['numFactura']}")
                return data['detalleFactura']
            else:
                print(f"✗ Error de negocio: {data['result']['msgResult']}")
                return None
                
        except requests.exceptions.Timeout:
            print("✗ Error: Timeout al consultar el API")
            return None
        except requests.exceptions.RequestException as e:
            print(f"✗ Error de conexión: {str(e)}")
            return None

# Uso
client = InvoiceAPIClient(
    base_url="https://ntgy-fua-e-fua-pro.us-e2.cloudhub.io",
    client_code="12345678A",
    auth_token="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
)

invoice = client.get_invoice_detail('1234567890')
if invoice:
    print(f"Importe total: {invoice['impTotFactura']}")
    print(f"Fecha de emisión: {invoice['fecEmision']}")
    print(f"Cuenta contrato: {invoice['cuentaContrato']}")