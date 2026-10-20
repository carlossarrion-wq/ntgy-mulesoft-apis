---
api: "ntgy-newco-s-postgresql-salesforce"
sidebar_position: 9
sidebar_label: "POST /ejecutarUpdate"
sql_metadata:
  data_sources:
    - database: "newcosalesforce"
      schema: "newcosalesforce"
      type: "PostgreSQL"
      tables:
        - name: "ALL"
          operations: ["WRITE"]
---

# Endpoint: POST /ejecutarUpdate

## 🔴 ADVERTENCIA CRÍTICA DE SEGURIDAD

**ESTE ES UN ENDPOINT EXTREMADAMENTE PELIGROSO - DEBE ESTAR DESHABILITADO EN PRODUCCIÓN**

---

## 📋 Descripción General

Ejecuta sentencias SQL de modificación (UPDATE/INSERT/DELETE) directas contra la base de datos PostgreSQL sin validación ni sanitización.

**Tipo de API:** System API (Internal/Debug)  
**Capa:** System Layer  
**Versión:** 1.42.0  
**Última actualización:** 2026-05-19

---

## 🎯 Propósito

Este endpoint permite ejecutar operaciones de escritura SQL para:
- Corrección de datos en desarrollo
- Scripts de mantenimiento en entornos no productivos
- Debugging de problemas de datos
- **NUNCA debe usarse en producción**

---

## 🔗 URL del Endpoint

```
POST /api/ejecutarUpdate
```

**Base URL (Local):** `http://localhost:8081`  
**Base URL (Dev):** `https://ntgy-newco-s-postgresql-salesforce-dev.cloudhub.io` (con restricciones)  
**⛔ Production:** **DEBE ESTAR COMPLETAMENTE DESHABILITADO**

---

## 📥 Parámetros de Entrada

### Request Body (JSON)

```json
{
  "update": "UPDATE newcosalesforce.account SET name = 'Updated Name' WHERE sfid = '001aA000001234567'"
}
```

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `update` | string | Sí | Sentencia SQL UPDATE/INSERT/DELETE a ejecutar |

---

## 📤 Respuestas

### ✅ Respuesta Exitosa (HTTP 200)

```json
{
  "affectedRows": 1,
  "message": "Update executed successfully"
}
```

### ❌ Errores

#### 400 Bad Request
```json
{
  "errorCode": 400,
  "message": "Bad request",
  "description": "Missing or invalid update parameter"
}
```

#### 500 Internal Server Error
```json
{
  "errorCode": 500,
  "message": "DB:UPDATE_EXECUTION",
  "description": "SQL syntax error or update execution failed"
}
```

---

## 🧪 Ejemplos de Uso

### Ejemplo 1: Actualizar un registro

```bash
curl -X POST "http://localhost:8081/api/ejecutarUpdate" \
  -H "Content-Type: application/json" \
  -d '{
    "update": "UPDATE newcosalesforce.account SET name = '\''Nuevo Nombre'\'' WHERE sfid = '\''001aA000001234567'\''"
  }'
```

### Ejemplo 2: Insertar un registro

```bash
curl -X POST "http://localhost:8081/api/ejecutarUpdate" \
  -H "Content-Type: application/json" \
  -d '{
    "update": "INSERT INTO newcosalesforce.test_table (id, value) VALUES (1, '\''test'\'')"
  }'
```

---

## 🔴 Riesgos Críticos de Seguridad

### ⚠️ Vulnerabilidades Extremas

1. **SQL Injection:** Sin ninguna validación de entrada
2. **Pérdida de Datos:** Posibilidad de DELETE/TRUNCATE sin restricciones
3. **Corrupción de Datos:** UPDATE sin validación de integridad
4. **Sin Rollback:** Cambios permanentes sin posibilidad de deshacer
5. **Sin Auditoría:** No hay registro de quién modificó qué
6. **Bypass de Lógica de Negocio:** Modifica datos sin pasar por validaciones

### 🛡️ Mitigaciones OBLIGATORIAS

#### Nivel 1: Deshabilitar en Producción
```yaml
# prod.yaml
ejecutarUpdate:
  enabled: false  # ⛔ COMPLETAMENTE DESHABILITADO
```

#### Nivel 2: Restricciones en Desarrollo
```yaml
# dev.yaml
ejecutarUpdate:
  enabled: true
  requiresApproval: true
  maxAffectedRows: 10
  timeout: 10s
  allowedOperations: ['UPDATE']  # Solo UPDATE, no DELETE/TRUNCATE
```

#### Nivel 3: Seguridad Adicional
- ✅ **Autenticación MFA** (Multi-Factor Authentication)
- ✅ **Whitelist de IPs** (solo IPs de desarrolladores autorizados)
- ✅ **Logging completo** con alertas en tiempo real
- ✅ **Backup automático** antes de cada operación
- ✅ **Dry-run mode** (simular sin ejecutar)
- ✅ **Approval workflow** (requiere aprobación de otro desarrollador)

---

## 🚫 Restricciones Técnicas Recomendadas

### Validaciones Mínimas Obligatorias

```javascript
// 1. Validar que NO sea DROP/TRUNCATE
const forbidden = ['DROP', 'TRUNCATE', 'ALTER', 'CREATE'];
if (forbidden.some(cmd => update.toUpperCase().includes(cmd))) {
  throw new Error('Forbidden SQL command');
}

// 2. Validar longitud
if (update.length > 2000) {
  throw new Error('Update statement too long');
}

// 3. Validar que tenga WHERE clause (evitar updates masivos)
if (update.toUpperCase().includes('UPDATE') && 
    !update.toUpperCase().includes('WHERE')) {
  throw new Error('UPDATE without WHERE clause not allowed');
}

// 4. Limitar filas afectadas
const result = executeUpdate(update);
if (result.affectedRows > 100) {
  rollback();
  throw new Error('Too many rows affected. Operation rolled back.');
}
```

### Usuario de Base de Datos Restringido

```sql
-- Crear usuario con permisos limitados
CREATE USER mulesoft_update WITH PASSWORD 'secure_password';

-- Solo permisos de UPDATE en tablas específicas
GRANT UPDATE ON newcosalesforce.account TO mulesoft_update;
GRANT UPDATE ON newcosalesforce.contact TO mulesoft_update;

-- NO dar permisos de DELETE/TRUNCATE/DROP
REVOKE DELETE ON ALL TABLES IN SCHEMA newcosalesforce FROM mulesoft_update;
```

---

## 📊 Logging y Auditoría

### Información a Registrar

```json
{
  "timestamp": "2026-05-19T15:30:00Z",
  "endpoint": "/ejecutarUpdate",
  "user": "developer@naturgy.com",
  "ip": "192.168.1.100",
  "statement": "UPDATE newcosalesforce.account SET...",
  "affectedRows": 1,
  "executionTime": "45ms",
  "success": true
}
```

### Alertas Automáticas

- 🚨 Más de 10 filas afectadas
- 🚨 DELETE sin WHERE clause
- 🚨 Múltiples ejecuciones en corto tiempo
- 🚨 Ejecución fuera de horario laboral

---

## 🔄 Alternativas Más Seguras

En lugar de usar este endpoint, considerar:

1. **Crear endpoints específicos** para cada operación de actualización
2. **Usar APIs de Salesforce** para modificar datos
3. **Scripts de migración** con control de versiones
4. **Herramientas de administración** con interfaz gráfica
5. **Procesos batch** con validaciones y rollback

---

## 📚 Referencias

- **RAML Specification:** `src/main/resources/api/ntgy-newco-s-postgresql-salesforce.raml`
- **Main Flow:** `src/main/mule/s-postgresql-salesforce.xml`
- **Error Handler:** `src/main/mule/error-managerExpSys.xml`

---

## 🔴 RECORDATORIO FINAL

**ESTE ENDPOINT PUEDE DESTRUIR DATOS EN PRODUCCIÓN**

### Checklist de Seguridad:

- [ ] ¿Está deshabilitado en producción?
- [ ] ¿Requiere autenticación fuerte?
- [ ] ¿Tiene logging completo?
- [ ] ¿Hay backups automáticos?
- [ ] ¿Existe un proceso de rollback?
- [ ] ¿Hay límites de filas afectadas?
- [ ] ¿Se validan las sentencias SQL?
- [ ] ¿Hay alertas configuradas?

**Si alguna respuesta es NO, el endpoint NO debe estar activo.**

---

**Última actualización:** 2026-05-19  
**Versión del documento:** 1.0  
**Autor:** Equipo MuleSoft Naturgy