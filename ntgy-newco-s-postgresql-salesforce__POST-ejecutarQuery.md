---
api: "ntgy-newco-s-postgresql-salesforce"
sidebar_position: 8
sidebar_label: "POST /ejecutarQuery"
sql_metadata:
  data_sources:
    - database: "newcosalesforce"
      schema: "newcosalesforce"
      type: "PostgreSQL"
      tables:
        - name: "ALL"
          operations: ["READ"]
---

# Endpoint: POST /ejecutarQuery

## ⚠️ ADVERTENCIA DE SEGURIDAD

**ESTE ES UN ENDPOINT DE DESARROLLO/DEBUG - NO DEBE ESTAR ACTIVO EN PRODUCCIÓN**

---

## 📋 Descripción General

Ejecuta consultas SQL SELECT directas contra la base de datos PostgreSQL sin validación ni sanitización.

**Tipo de API:** System API (Internal/Debug)  
**Capa:** System Layer  
**Versión:** 1.42.0  
**Última actualización:** 2026-05-19

---

## 🎯 Propósito

Este endpoint permite ejecutar queries SQL arbitrarias para propósitos de:
- Desarrollo y debugging
- Consultas ad-hoc de datos
- Troubleshooting de base de datos
- Análisis de datos en entornos no productivos

---

## 🔗 URL del Endpoint

```
POST /api/ejecutarQuery
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-salesforce-dev.cloudhub.io`  
**⛔ Production:** **DEBE ESTAR DESHABILITADO**

---

## 📥 Parámetros de Entrada

### Request Body (JSON)

```json
{
  "query": "SELECT * FROM newcosalesforce.account LIMIT 10"
}
```

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `query` | string | Sí | Consulta SQL SELECT a ejecutar |

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

```json
[
  {
    "sfid": "001aA000001234567",
    "name": "Cliente Ejemplo",
    "acn_fld_documentnumber__c": "12345678A"
  },
  {
    "sfid": "001aA000001234568",
    "name": "Otro Cliente",
    "acn_fld_documentnumber__c": "87654321B"
  }
]
```

### ❌ Errores

#### 400 Bad Request
```json
{
  "errorCode": 400,
  "message": "Bad request",
  "description": "Missing or invalid query parameter"
}
```

#### 500 Internal Server Error
```json
{
  "errorCode": 500,
  "message": "DB:QUERY_EXECUTION",
  "description": "SQL syntax error or query execution failed"
}
```

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Consultar clientes

```bash
curl -X POST "http://localhost:8081/api/ejecutarQuery" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "SELECT sfid, name FROM newcosalesforce.account LIMIT 5"
  }'
```

### Ejemplo 2: Consultar con filtros

```bash
curl -X POST "http://localhost:8081/api/ejecutarQuery" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "SELECT * FROM newcosalesforce.case WHERE accountid = '\''001aA000001234567'\''"
  }'
```

---

## ⚠️ Riesgos de Seguridad

### 🔴 Vulnerabilidades

1. **SQL Injection:** Sin validación de entrada
2. **Exposición de Datos:** Acceso completo a todas las tablas
3. **Performance:** Queries no optimizadas pueden saturar la BD
4. **Auditoría:** Sin registro de quién ejecuta qué queries

### 🛡️ Mitigaciones Requeridas

- ✅ **Deshabilitar en producción** (configuración por ambiente)
- ✅ **Autenticación fuerte** (API Key, OAuth)
- ✅ **Whitelist de IPs** (solo IPs internas)
- ✅ **Rate limiting** (máximo X queries por minuto)
- ✅ **Logging completo** (auditoría de todas las queries)
- ✅ **Timeout corto** (evitar queries largas)
- ✅ **Read-only user** (usuario de BD sin permisos de escritura)

---

## 🚫 Restricciones Recomendadas

### Configuración por Ambiente

```yaml
# dev.yaml
ejecutarQuery:
  enabled: true
  timeout: 30s
  maxRows: 1000

# prod.yaml
ejecutarQuery:
  enabled: false  # ⛔ DESHABILITADO
```

### Validaciones Mínimas

```javascript
// Validar que solo sean SELECT
if (!query.trim().toUpperCase().startsWith('SELECT')) {
  throw new Error('Only SELECT queries allowed');
}

// Validar longitud
if (query.length > 5000) {
  throw new Error('Query too long');
}

// Blacklist de comandos peligrosos
const forbidden = ['DROP', 'DELETE', 'UPDATE', 'INSERT', 'TRUNCATE'];
if (forbidden.some(cmd => query.toUpperCase().includes(cmd))) {
  throw new Error('Forbidden SQL command');
}
```

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-salesforce.raml`
- **Main Flow:** `src/main/mule/s-postgresql-salesforce.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`

---

## ⚠️ RECORDATORIO FINAL

**ESTE ENDPOINT ES EXTREMADAMENTE PELIGROSO EN PRODUCCIÓN**

- Permite acceso completo a la base de datos
- Sin validación ni sanitización
- Riesgo de SQL injection
- Riesgo de exposición de datos sensibles
- Debe estar DESHABILITADO en producción

---

**Última actualización:** 2026-05-19  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy