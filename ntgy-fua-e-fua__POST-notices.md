---
api: "ntgy-fua-e-fua"
sidebar_position: 69
sidebar_label: "POST /notices"
description: "Actualiza el estado de lectura o eliminación de avisos del cliente en la aplicación FUA"
keywords: [notices, avisos, actualizar, lectura, eliminación, notificaciones, FUA, cliente, DynamoDB]
tags: [Notices, Avisos, Experience API, Mobile, Web]
channel: mobile
endpoint_dependencies:
  - method: "POST"
    api_name: "ntgy-newco-p-fua-evo"
    path: "/notices"
    purpose: "Actualización del estado de avisos en DynamoDB"
---

# POST /notices

## 📌 Descripción General

### Propósito del Endpoint

Este endpoint permite a los clientes actualizar el estado de sus avisos en la aplicación FUA, marcándolos como leídos o eliminados. Es fundamental para la gestión de notificaciones y la experiencia de usuario en la aplicación móvil y web.

### Tipo de API

**Capa:** Experience API (API-Led Connectivity)

**Función:** Adaptación de experiencia para canales web y móvil

### Patrón de Integración

**Tipo:** Pass-Through con enriquecimiento de headers

**Descripción:** El Experience API actúa como pass-through hacia el Process API, enriqueciendo la petición con headers de contexto FUA (x-client-code, x-interaction-id, etc.) y proporcionando una respuesta simplificada al cliente.

### Arquitectura API-Led Connectivity

```mermaid
graph TB
    C[Cliente<br/>App Móvil/Web]
    E[Experience API<br/>ntgy-fua-e-fua<br/>POST /notices]
    P[Process API<br/>ntgy-newco-p-fua-evo<br/>POST /notices]
    S[System API<br/>ntgy-fua-s-dynamodb<br/>POST /query + POST /item]
    D[(DynamoDB<br/>ntgy-dydb-fua-notice)]
    
    C -->|Request| E
    E -->|Enrich Headers| P
    P -->|Query Aviso| S
    S -->|Consulta| D
    D -->|Datos Aviso| S
    S -->|Response| P
    P -->|Update Aviso| S
    S -->|Actualiza| D
    D -->|Confirmación| S
    S -->|Response| P
    P -->|Response| E
    E -->|Simplified Response| C
    
    style C fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    style E fill:#fff4e1,stroke:#f57c00,stroke-width:3px
    style P fill:#ffe1f5,stroke:#c2185b,stroke-width:2px
    style S fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style D fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
```

---

## 💼 Propósito del Negocio

### Casos de Uso Principales

1. **Marcar Aviso como Leído**
   - El cliente visualiza un aviso en la app y lo marca como leído
   - Mejora la experiencia al mantener un registro de avisos no leídos

2. **Eliminar Aviso**
   - El cliente elimina un aviso que ya no desea ver
   - Permite al usuario gestionar sus notificaciones activamente

3. **Actualización Combinada**
   - Marcar como leído y eliminar en una sola operación
   - Optimiza las llamadas al backend

### 📱 Canal y Audiencia

#### Canal de Consumo

**Canal:** Mobile y Web

**Características del Canal:**
- **Plataforma**: Aplicación móvil FUA (iOS/Android) y portal web
- **Dispositivos**: Smartphones, tablets, navegadores web
- **Conectividad**: Requiere conexión a internet
- **Experiencia de Usuario**: Actualización inmediata del estado de avisos

#### Audiencia Objetivo

**Usuarios Finales:**
- **Clientes Residenciales**: Gestión de avisos personales sobre facturas, servicios, promociones
- **Clientes Empresariales**: Gestión de avisos relacionados con contratos y servicios empresariales

**Contexto de Uso:**
- **Gestión de Notificaciones**: Cliente revisa y gestiona sus avisos pendientes
- **Limpieza de Bandeja**: Cliente elimina avisos antiguos o no relevantes

#### Adaptaciones del Canal

**Optimizaciones Aplicadas:**
- **Simplificación de Respuesta**: Solo devuelve confirmación de éxito/error
- **Headers de Contexto**: Enriquecimiento automático con información del cliente
- **Timeout Optimizado**: 60 segundos para garantizar la actualización en DynamoDB

### 🔗 Endpoints Dependientes

Este Experience API consume el siguiente Process API:

| Process API | Endpoint | Método | Propósito |
|-------------|----------|--------|-----------|
| `ntgy-newco-p-fua-evo` | `/notices` | POST | Actualización del estado de avisos en DynamoDB |

### Contexto Empresarial

- **Dominio:** Gestión de Notificaciones y Comunicaciones
- **Sector:** Energía - Utilities
- **División:** Naturgy España - NewCo
- **Canal:** Mobile y Web

### Stakeholders

- **Clientes Finales**: Necesitan gestionar sus avisos de forma eficiente
- **Equipo de Producto FUA**: Requiere métricas de interacción con avisos
- **Equipo de Marketing**: Analiza efectividad de comunicaciones

### Valor de Negocio

- **Mejora de Experiencia**: Permite al cliente controlar sus notificaciones
- **Reducción de Ruido**: Elimina avisos no relevantes para el cliente
- **Métricas de Engagement**: Tracking de avisos leídos vs no leídos
- **Personalización**: Base para futuras mejoras en comunicaciones dirigidas

---

## 🔧 Especificación Técnica

### URLs por Ambiente

| Ambiente | URL Base | URL Completa |
|----------|----------|--------------|
| **Local** | `http://localhost:8081` | `http://localhost:8081/fua/api/notices` |
| **Development** | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-dev.de-c1.cloudhub.io/fua/api/notices` |
| **Production** | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io` | `https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices` |

### Método HTTP

```
POST /fua/api/notices
```

### 🎯 Consideraciones del Canal

#### Limitaciones del Canal

| Aspecto | Limitación | Razón |
|---------|------------|-------|
| **Timeout** | 60 segundos | Operaciones de actualización en DynamoDB |
| **Rate Limiting** | Según política API Gateway | Protección del backend |
| **Tamaño de Request** | < 10 KB | Payload simple con 3 campos |

#### Campos Específicos del Canal

**Headers Enriquecidos por el Experience API:**

| Header | Tipo | Descripción | Origen |
|--------|------|-------------|--------|
| `x-client-code` | string | Código del cliente | Contexto FUA (vars.fuaHeaders) |
| `x-interaction-id` | string | ID de interacción | Contexto FUA |
| `x-user-id` | string | ID del usuario | Contexto FUA |

### Parámetros de Entrada

#### Request Body

**Content-Type:** `application/json`

| Campo | Tipo | Requerido | Descripción | Ejemplo |
|-------|------|-----------|-------------|---------|
| `read` | boolean | No | Indica si el cliente ha leído el aviso | `true` |
| `delete` | boolean | No | Indica si el cliente ha eliminado el aviso | `false` |
| `idNotice` | string | Sí | ID único del aviso a actualizar | `"12345"` |

**Validaciones:**
- Al menos uno de los campos `read` o `delete` debe estar presente
- `idNotice` es obligatorio y debe corresponder a un aviso existente

#### Ejemplo de Request Body

```json
{
  "read": true,
  "delete": false,
  "idNotice": "12345"
}
```

### Respuestas

#### 200 OK - Actualización Exitosa

**Content-Type:** `application/json`

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "El aviso se ha actualizado"
  }
}
```

**Estructura de Respuesta:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `result` | object | Objeto con resultado de la operación |
| `result.codResult` | string | Código de resultado ("0000" = éxito) |
| `result.msgResult` | string | Mensaje descriptivo del resultado |

#### 400 Bad Request - Error de Validación

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "idNotice es un campo obligatorio"
  }
}
```

#### 500 Internal Server Error - Error del Sistema

```json
{
  "result": {
    "codResult": "0001",
    "msgResult": "Error al actualizar el aviso en DynamoDB"
  }
}
```

### Autenticación

- **Tipo:** OAuth 2.0 con JWT
- **Header:** `Authorization: Bearer {token}`
- **Validación:** Realizada por el flow `jwt-validation` antes del procesamiento

### Timeouts

| Tipo | Valor | Razón |
|------|-------|-------|
| **Request Timeout** | 60 segundos | Operaciones de consulta y actualización en DynamoDB |
| **Connection Timeout** | 30 segundos | Establecimiento de conexión con Process API |

---

## 🏗️ Implementación Técnica

### Arquitectura del Flow

```mermaid
flowchart TD
    Start([HTTP POST Request]) --> JWT[JWT Validation]
    JWT --> SetHeaders[Set fuaHeaders Variable]
    SetHeaders --> CallProcess[Call Process API<br/>POST /notices]
    CallProcess --> Transform[Transform Response<br/>Actualización OK]
    Transform --> End([HTTP Response 200])
    
    CallProcess -->|Error| ErrorHandler[Error Handler]
    ErrorHandler --> ErrorTransform[Transform Error Message]
    ErrorTransform --> ErrorLog[Log Error]
    ErrorLog --> ErrorEnd([HTTP Response 500])
    
    style Start fill:#e1f5ff,stroke:#0277bd
    style End fill:#c8e6c9,stroke:#2e7d32
    style ErrorEnd fill:#ffcdd2,stroke:#c62828
    style CallProcess fill:#fff9c4,stroke:#f57f17
```

### Transformaciones de Canal

**El Experience API NO transforma la estructura de datos del request.**

El Experience API actúa como pass-through, enriqueciendo el request con headers de contexto FUA:

| Header Original | Header Enriquecido | Transformación |
|-----------------|-------------------|----------------|
| N/A | `x-client-code` | Extraído de `vars.fuaHeaders` |
| N/A | `x-interaction-id` | Extraído de `vars.fuaHeaders` |
| N/A | `x-user-id` | Extraído de `vars.fuaHeaders` |

**Lógica de Enriquecimiento:**

```dataweave
%dw 2.2
output application/java
---
{
  ("x-interaction-id": attributes.headers['x-interaction-id']) if (attributes.headers['x-interaction-id'] != "null" and !isEmpty(attributes.headers['x-interaction-id'])),
  ("x-client-code": attributes.headers['x-client-code']) if (attributes.headers['x-client-code'] != "null" and !isEmpty(attributes.headers['x-client-code'])),
  ("x-user-id": attributes.headers['x-user-id']) if (attributes.headers['x-user-id'] != "null" and !isEmpty(attributes.headers['x-user-id']))
}
```

**Transformación de Respuesta:**

La respuesta del Process API se transforma a un mensaje simplificado:

```dataweave
%dw 2.0
output application/json
---
{
  "result": {
    "codResult": "0000",
    "msgResult": "El aviso se ha actualizado"
  }
}
```

### Patrones de Diseño

**Patrón:** Pass-Through con Enriquecimiento de Headers

**Justificación:** 
- Simplifica la integración para clientes móviles y web
- Centraliza la gestión de headers de contexto FUA
- Proporciona respuestas consistentes independientemente del backend

### Diagramas de Secuencia

#### 1. Flujo Normal de Éxito

```mermaid
sequenceDiagram
    participant C as Cliente Mobile/Web
    participant E as Experience API
    participant P as Process API
    participant S as System API DynamoDB
    participant D as DynamoDB
    
    C->>E: POST /notices<br/>{read: true, idNotice: "12345"}
    Note over E: JWT Validation
    Note over E: Set fuaHeaders
    E->>P: POST /notices<br/>+ fuaHeaders
    Note over P: Extract docNumber
    P->>S: POST /query<br/>Consulta aviso por CLIENT_CODE
    S->>D: Query ntgy-dydb-fua-notice
    D-->>S: Datos del aviso
    S-->>P: Aviso encontrado
    Note over P: Actualiza campos READ/DELETE
    P->>S: POST /item<br/>Actualiza aviso
    S->>D: Update ntgy-dydb-fua-notice
    D-->>S: Confirmación
    S-->>P: Actualización exitosa
    P-->>E: Response 200
    Note over E: Transform Response
    E-->>C: {codResult: "0000",<br/>msgResult: "El aviso se ha actualizado"}
```

#### 2. Flujo con Error de Negocio

```mermaid
sequenceDiagram
    participant C as Cliente Mobile/Web
    participant E as Experience API
    participant P as Process API
    participant S as System API DynamoDB
    
    C->>E: POST /notices<br/>{idNotice: "99999"}
    Note over E: JWT Validation
    E->>P: POST /notices<br/>+ fuaHeaders
    P->>S: POST /query<br/>Consulta aviso
    S-->>P: Aviso no encontrado
    P-->>E: Error 404
    Note over E: Error Handler
    E-->>C: {codResult: "0001",<br/>msgResult: "Aviso no encontrado"}
```

#### 3. Flujo con Error Técnico

```mermaid
sequenceDiagram
    participant C as Cliente Mobile/Web
    participant E as Experience API
    participant P as Process API
    
    C->>E: POST /notices<br/>{read: true, idNotice: "12345"}
    Note over E: JWT Validation
    E->>P: POST /notices<br/>+ fuaHeaders
    Note over P: Timeout o Error DynamoDB
    P-->>E: Error 500
    Note over E: Error Handler
    Note over E: Log Error
    E-->>C: {codResult: "0001",<br/>msgResult: "Error al actualizar el aviso"}
```

### Flujo Detallado del Process API

El Process API realiza las siguientes operaciones:

1. **Extracción de Datos:**
   - Extrae `docNumber` de `x-client-code` o `payload.nif`
   - Crea variable `dynEntry` con los datos a actualizar

2. **Consulta del Aviso:**
   - Query a DynamoDB por `CLIENT_CODE`
   - Filtra el aviso específico por `NOTICE_ID`

3. **Actualización del Aviso:**
   - Combina datos existentes con nuevos estados (`READ`, `DELETE`)
   - POST a DynamoDB `/item` para actualizar el registro

### Manejo de Errores

**Error Handler del Experience API:**

```xml
<on-error-propagate>
  <ee:transform>
    <ee:set-payload>
      {
        "result": {
          "codResult": "0001",
          "msgResult": error.description as String replace /\"/ with("")
        }
      }
    </ee:set-payload>
  </ee:transform>
  <logger level="ERROR" message="Error actualizando aviso"/>
</on-error-propagate>
```

**Tipos de Errores:**

| Código | Descripción | Causa |
|--------|-------------|-------|
| `0000` | Éxito | Aviso actualizado correctamente |
| `0001` | Error | Aviso no encontrado, error de DynamoDB, timeout |

---

## 💻 Ejemplos de Uso

### Ejemplo 1: Marcar Aviso como Leído

#### cURL

```bash
curl -X POST "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-client-code: 12345678A" \
  -H "x-interaction-id: 550e8400-e29b-41d4-a716-446655440000" \
  -d '{
    "read": true,
    "delete": false,
    "idNotice": "abc123-def456-ghi789"
  }'
```

#### Respuesta (200 OK)

```json
{
  "result": {
    "codResult": "0000",
    "msgResult": "El aviso se ha actualizado"
  }
}
```

### Ejemplo 2: Eliminar Aviso

#### JavaScript (React Native / Web)

```javascript
const updateNotice = async (noticeId, isRead, isDeleted) => {
  try {
    const response = await fetch('https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${authToken}`,
        'x-client-code': clientCode,
        'x-interaction-id': generateInteractionId()
      },
      body: JSON.stringify({
        read: isRead,
        delete: isDeleted,
        idNotice: noticeId
      })
    });

    const data = await response.json();
    
    if (data.result.codResult === '0000') {
      console.log('Aviso actualizado correctamente');
      return true;
    } else {
      console.error('Error:', data.result.msgResult);
      return false;
    }
  } catch (error) {
    console.error('Error de red:', error);
    return false;
  }
};

// Uso: Eliminar aviso
await updateNotice('abc123-def456-ghi789', false, true);
```

### Ejemplo 3: Marcar como Leído y Eliminar

#### Python

```python
import requests
import json

def update_notice(notice_id, read=False, delete=False):
    url = "https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices"
    
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {auth_token}",
        "x-client-code": client_code,
        "x-interaction-id": interaction_id
    }
    
    payload = {
        "read": read,
        "delete": delete,
        "idNotice": notice_id
    }
    
    response = requests.post(url, headers=headers, json=payload)
    
    if response.status_code == 200:
        result = response.json()
        if result['result']['codResult'] == '0000':
            print("Aviso actualizado correctamente")
            return True
        else:
            print(f"Error: {result['result']['msgResult']}")
            return False
    else:
        print(f"Error HTTP: {response.status_code}")
        return False

# Uso: Marcar como leído y eliminar
update_notice("abc123-def456-ghi789", read=True, delete=True)
```

### Ejemplo 4: Manejo de Errores

#### TypeScript (Angular)

```typescript
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, map } from 'rxjs/operators';

interface NoticeUpdateRequest {
  read?: boolean;
  delete?: boolean;
  idNotice: string;
}

interface NoticeUpdateResponse {
  result: {
    codResult: string;
    msgResult: string;
  };
}

@Injectable({
  providedIn: 'root'
})
export class NoticeService {
  private apiUrl = 'https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices';

  constructor(private http: HttpClient) {}

  updateNotice(request: NoticeUpdateRequest): Observable<boolean> {
    const headers = new HttpHeaders({
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${this.authToken}`,
      'x-client-code': this.clientCode,
      'x-interaction-id': this.generateInteractionId()
    });

    return this.http.post<NoticeUpdateResponse>(this.apiUrl, request, { headers })
      .pipe(
        map(response => {
          if (response.result.codResult === '0000') {
            console.log('Aviso actualizado:', response.result.msgResult);
            return true;
          } else {
            throw new Error(response.result.msgResult);
          }
        }),
        catchError(error => {
          console.error('Error actualizando aviso:', error);
          return throwError(() => new Error('No se pudo actualizar el aviso'));
        })
      );
  }
}

// Uso en componente
this.noticeService.updateNotice({
  read: true,
  delete: false,
  idNotice: 'abc123-def456-ghi789'
}).subscribe({
  next: (success) => {
    if (success) {
      this.showSuccessMessage('Aviso marcado como leído');
      this.refreshNotices();
    }
  },
  error: (error) => {
    this.showErrorMessage(error.message);
  }
});
```

### Ejemplo 5: Integración con Estado de Aplicación

#### React con Redux

```javascript
// actions/noticeActions.js
export const updateNoticeStatus = (noticeId, isRead, isDeleted) => async (dispatch, getState) => {
  dispatch({ type: 'UPDATE_NOTICE_REQUEST' });
  
  const { auth, user } = getState();
  
  try {
    const response = await fetch('https://ntgy-fua-e-fua-pro.de-c1.cloudhub.io/fua/api/notices', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${auth.token}`,
        'x-client-code': user.clientCode,
        'x-interaction-id': generateInteractionId()
      },
      body: JSON.stringify({
        read: isRead,
        delete: isDeleted,
        idNotice: noticeId
      })
    });

    const data = await response.json();

    if (data.result.codResult === '0000') {
      dispatch({
        type: 'UPDATE_NOTICE_SUCCESS',
        payload: { noticeId, isRead, isDeleted }
      });
      
      // Actualizar lista de avisos
      dispatch(fetchNotices());
    } else {
      dispatch({
        type: 'UPDATE_NOTICE_FAILURE',
        error: data.result.msgResult
      });
    }
  } catch (error) {
    dispatch({
      type: 'UPDATE_NOTICE_FAILURE',
      error: 'Error de conexión'
    });
  }
};

// reducer/noticeReducer.js
const noticeReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'UPDATE_NOTICE_REQUEST':
      return { ...state, loading: true };
      
    case 'UPDATE_NOTICE_SUCCESS':
      return {
        ...state,
        loading: false,
        notices: state.notices.map(notice =>
          notice.noticeId === action.payload.noticeId
            ? { ...notice, read: action.payload.isRead, delete: action.payload.isDeleted }
            : notice
        )
      };
      
    case 'UPDATE_NOTICE_FAILURE':
      return { ...state, loading: false, error: action.error };
      
    default:
      return state;
  }
};