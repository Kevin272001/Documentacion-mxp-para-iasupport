# Documentación Técnica - Módulo de Integración (Mxp-Integration)

## ❓ Propósito del Módulo

El módulo **Mxp-Integration** es el núcleo de integración del ecosistema MAXPOINT, encargado de gestionar la comunicación entre diferentes servicios y sistemas internos. Este módulo actúa como orquestador de procesos y facilitador de datos entre los distintos componentes de la plataforma.

## ✅ Funcionalidades Principales

* 🔄 Sincronización de menús y productos
* 💳 Gestión de métodos de pago
* 🏷️ Configuración de impuestos
* 🌍 Configuración por país
* 📊 Monitoreo de estados
* 🔄 Procesos de propagación

---

## 🧩 Descripción Técnica

* **Microservicio Principal:** `mxpv2.integration.core`
* **Tecnologías Clave:** 
  - .NET Core 6.0
  - Entity Framework Core
  - SQL Server
  - Redis (caché)
* **Patrones de Diseño:**
  - Patrón Fachada (Facade)
  - Patrón Repositorio
  - Patrón Unit of Work

---

## 🔌 Endpoints Disponibles

### 1. Sincronización de Menú (Facade)

**Endpoint:**
```http
POST {{server_etlp_extract}}/api/v1/facade
```

**Código de Operación:** `F101`

**Cuerpo de la Solicitud:**
```json
{
    "code": "F101",
    "withResponse": true,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "EC",
    "franchise_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
    "processes": {
        "EXTRACTS": [
            {
                "processName": "MenuExtract",
                "parameters": {
                    "franchiseId": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af"
                }
            }
        ]
    }
}
```

**Respuesta Exitosa (200):**
```json
{
    "code": 200,
    "message": "Proceso de sincronización iniciado correctamente",
    "data": {
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "status": "IN_PROGRESS",
        "startTime": "2024-09-15T11:00:00-05:00"
    }
}
```

**Códigos de Error Comunes:**
- `400`: Solicitud incorrecta (parámetros inválidos)
- `401`: No autorizado
- `500`: Error interno del servidor

---

### 2. Configuración de Impuestos

**Endpoint:**
```http
POST {{server_bo}}/propagations/api/v1/propagationlegacy/startrestaurantenablingtaxespropagation
```

**Parámetros de Consulta:**
- `user`: ID del usuario que realiza la acción (UUID)

**Cuerpo de la Solicitud:**
```json
{
    "restaurantId": "24260579-faf8-763f-aac7-5e16faff96d1",
    "taxes": [
        {
            "taxId": "IVA-12",
            "name": "IVA 12%",
            "rate": 12.0,
            "isActive": true,
            "isIncludedInPrice": true
        }
    ]
}
```

**Respuesta Exitosa (200):**
```json
{
    "code": 200,
    "message": "Configuración de impuestos aplicada correctamente",
    "data": {
        "propagationId": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
        "status": "PENDING",
        "timestamp": "2024-09-15T11:05:00-05:00"
    }
}
```

---

### 3. Gestión de Estados

#### 3.1 Crear Estado

**Endpoint:**
```http
POST {{server_orchestrator}}/api/v1/status/create
```

**Cuerpo de la Solicitud:**
```json
{
    "code": "ORDER_PENDING",
    "name": "Orden Pendiente",
    "description": "La orden ha sido creada y está pendiente de procesamiento",
    "isActive": true,
    "module": "ORDERS"
}
```

#### 3.2 Obtener Estado por ID

**Endpoint:**
```http
GET {{server_orchestrator}}/api/v1/status/getById?id=3fa85f64-5717-4562-b3fc-2c963f66afa6
```

#### 3.3 Listar Estados (Paginado)

**Endpoint:**
```http
POST {{server_orchestrator}}/api/v1/states/getAllPaginated
```

**Cuerpo de la Solicitud:**
```json
{
    "pageNumber": 1,
    "pageSize": 10,
    "filters": {
        "module": "ORDERS",
        "isActive": true
    },
    "sortField": "name",
    "sortOrder": "ASC"
}
```

---

### 4. Configuración por País

**Endpoint:**
```http
GET {{server_config}}/api/v1/configurations/getallbycountry?country_id=d6cac83d-6ce5-f62e-2628-1e2e9ae7b51f
```

**Respuesta Exitosa (200):**
```json
{
    "code": 200,
    "message": "Configuración obtenida correctamente",
    "data": {
        "countryId": "d6cac83d-6ce5-f62e-2628-1e2e9ae7b51f",
        "countryCode": "EC",
        "currency": "USD",
        "timezone": "America/Guayaquil",
        "dateFormat": "dd/MM/yyyy",
        "decimalSeparator": ",",
        "thousandsSeparator": ".",
        "taxConfiguration": {
            "defaultTaxRate": 12.0,
            "taxIncludedInPrices": true
        },
        "features": {
            "digitalInvoicing": true,
            "onlinePayments": true,
            "loyaltyProgram": true
        },
        "createdAt": "2024-01-01T00:00:00Z",
        "updatedAt": "2024-09-15T10:30:00Z"
    }
}
```

---

## 🔄 Flujos de Trabajo Principales

### 1. Sincronización de Menú

1. El sistema inicia el proceso de sincronización llamando al endpoint de Facade
2. El servicio de ETL procesa la solicitud y genera tareas de extracción
3. Los datos se transforman y cargan en la base de datos
4. Se notifica el resultado del proceso

### 2. Configuración de Impuestos

1. El usuario configura los impuestos para un restaurante
2. El sistema valida la configuración
3. Se inicia el proceso de propagación a los servicios afectados
4. Se actualiza el estado de la propagación

## ⚠️ Consideraciones de Implementación

1. **Idempotencia**: Todos los endpoints que modifican datos son idempotentes
2. **Seguridad**: Se requiere autenticación mediante JWT
3. **Rendimiento**: Se recomienda implementar caché para consultas frecuentes
4. **Manejo de Errores**: Todos los errores incluyen códigos y mensajes descriptivos

## 📊 Métricas y Monitoreo

- **Latencia Promedio**: < 500ms
- **Disponibilidad**: 99.9%
- **Tasa de Error**: < 0.1%
- **Tiempo de Respuesta P95**: < 1s

## 📞 Soporte Técnico

Para reportar problemas o solicitar asistencia:

- **Correo Electrónico**: soporte.integracion@maxpoint.com
- **Horario de Atención**: 24/7
- **Sistema de Tickets**: [soporte.maxpoint.com/integration](https://soporte.maxpoint.com/integration)

---

*Última actualización: 15 de Septiembre de 2024*
*Versión del Documento: 1.0.0*
