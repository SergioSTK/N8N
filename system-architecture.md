# Arquitectura de Sistema de Gestión de Turnos

## 1. Visión General de Microservicios

### 1.1. Microservicio de Gestión de Agendas
**Responsabilidad:** Gestionar la configuración y mantenimiento de agendas.

**Endpoints REST:**
```
POST /api/agendas
GET /api/agendas
GET /api/agendas/{id}
PUT /api/agendas/{id}
DELETE /api/agendas/{id}
```

**Ejemplo Payload Creación:**
```json
{
  "nombre": "planes especiales",
  "configuracion": {
    "hora_inicio": "8:00",
    "hora_fin": "13:00",
    "duracion_turno": 30,
    "dias_atencion": ["lun", "mar", "mie", "jue", "vie"],
    "turnos_simultaneos": 1
  }
}
```

### 1.2. Microservicio de Gestión de Turnos
**Responsabilidad:** Manejar la creación, asignación y gestión de turnos.

**Endpoints REST:**
```
POST /api/turnos/generar
GET /api/turnos/disponibles
POST /api/turnos/asignar
PUT /api/turnos/{id}/liberar
GET /api/turnos/afiliado/{nro_afiliado}
```

**Ejemplo Payload Generación:**
```json
{
  "agenda_id": "123",
  "dias_vista": 7,
  "hora_inicio": "8:00",
  "hora_fin": "13:00",
  "duracion": 30,
  "atencion": ["lun", "mar", "mie", "jue", "vie"],
  "cant_simultaneo": 1
}
```

### 1.3. Microservicio de Calendario
**Responsabilidad:** Gestionar feriados y licencias.

**Endpoints REST:**
```
POST /api/calendario/feriados
POST /api/calendario/licencias
GET /api/calendario/excepciones
DELETE /api/calendario/feriados/{id}
DELETE /api/calendario/licencias/{id}
```

**Ejemplo Payload Feriado:**
```json
{
  "fecha": "2025-01-01",
  "descripcion": "Año Nuevo"
}
```

## 2. Modelo de Datos (MongoDB)

### 2.1. Colección Agendas
```json
{
  "_id": ObjectId,
  "nombre": String,
  "configuracion": {
    "hora_inicio": String,
    "hora_fin": String,
    "duracion_turno": Number,
    "dias_atencion": Array<String>,
    "turnos_simultaneos": Number
  },
  "activa": Boolean
}
```

### 2.2. Colección Turnos
```json
{
  "_id": ObjectId,
  "agenda_id": ObjectId,
  "fecha": Date,
  "hora": String,
  "estado": String,
  "afiliado": {
    "nombre": String,
    "numero": String,
    "correo": String,
    "telefono": String
  }
}
```

### 2.3. Colección Excepciones
```json
{
  "_id": ObjectId,
  "tipo": String, // "feriado" o "licencia"
  "agenda_id": ObjectId, // null para feriados
  "fecha_inicio": Date,
  "fecha_fin": Date,
  "descripcion": String
}
```

## 3. Observabilidad

### 3.1. Métricas Prometheus
- Contador de turnos creados
- Contador de turnos asignados
- Latencia de endpoints
- Tasa de errores
- Uso de memoria y CPU

### 3.2. Dashboards Grafana
- Panel de estado de servicios
- Métricas de performance
- Tiempos de respuesta
- Monitoreo de recursos

### 3.3. Alertas
- Servicio no responde (5xx)
- Latencia superior a 2 segundos
- Uso de CPU > 80%

## 4. Instrucciones para Frontend

### 4.1. Gestión de Errores
- 400: Error de validación
- 401: No autorizado
- 403: Acceso prohibido
- 404: Recurso no encontrado
- 409: Conflicto (turno ya asignado)
- 500: Error interno

### 4.2. Validaciones Frontend
- Verificar formato de fechas
- Validar horarios dentro del rango permitido
- Prevenir múltiples turnos por afiliado en la misma agenda
- Verificar campos requeridos antes de enviar

### 4.3. Consideraciones de Seguridad
- Implementar autenticación JWT
- Validar roles y permisos
- Sanitizar inputs
- Implementar rate limiting

## 5. Dependencias Técnicas

### 5.1. Infraestructura
- MongoDB
- Node.js/Express
- Docker
- Kubernetes
- Prometheus
- Grafana

### 5.2. Seguridad
- JWT para autenticación
- CORS configurado
- Rate limiting
- Validación de inputs

## 6. Consideraciones de Performance
- Indexar campos frecuentemente consultados en MongoDB
- Implementar caché para consultas frecuentes
- Paginación en endpoints de listado
- Compresión de respuestas
