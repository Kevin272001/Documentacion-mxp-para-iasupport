# Documentación Técnica - Módulo MXPi-Init

## 🔄 Endpoint: Propagación de Menú PLU (Price Look-Up)

### 📌 Descripción
Este endpoint se encarga de la propagación de información de productos (PLU) entre sistemas, incluyendo datos completos del producto, imágenes, categorías, impuestos y configuraciones relacionadas.

### 🔗 Endpoint
```http
POST {{server_etlp_extract}}/api/v1/facade
```

### 🔑 Autenticación
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

### 📥 Cuerpo de la Solicitud

#### Estructura Principal
```json
{
    "code": "F101",
    "withResponse": true,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": "{{legacy_cdn_id}}",
    "processes": {
        "EXTRACTS": [
            // Configuración de extracción de datos
        ],
        "TRANSFORMS": [
            // Configuración de transformación (opcional)
        ],
        "LOADS": [
            // Configuración de carga (opcional)
        ]
    }
}
```

#### Parámetros Principales
| Parámetro      | Tipo    | Requerido | Descripción |
|----------------|---------|-----------|-------------|
| code           | string  | Sí        | Código de operación: "F101" |
| withResponse   | boolean | Sí        | Indica si se requiere respuesta detallada |
| withCache      | boolean | Sí        | Habilita/deshabilita el uso de caché |
| syncId         | string  | Sí        | Identificador único de sincronización (UUID) |
| country        | string  | Sí        | Código de país (ej: "EC" para Ecuador) |
| franchise_id   | string  | Sí        | ID de la franquicia |

### 🔍 Proceso de Extracción (EXTRACTS)

#### Configuración de Conexión
```json
{
    "code": "E001",
    "withResponse": true,
    "withCache": true,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "configuration": {
        "connection": {
            "server": "dbmaxpoint-ec-mi.132fe021d3ac.database.windows.net",
            "user": "sis_mxpv2",
            "password": "******",
            "repository": "MAXPOINT_AZURE_102924",
            "adapter": "SqlServerSP"
        },
        "entities": [
            // Configuración de entidades a extraer
        ]
    }
}
```

#### Entidades de Datos
El endpoint extrae múltiples conjuntos de datos relacionados con el PLU:

1. **Datos Básicos del PLU**
   ```json
   {
       "name": "mxp_v2.USP_Propagate_Menu_Plus",
       "filters": [
           {"name": "country", "operator": "equals", "value": "{{country}}"},
           {"name": "cdn_id", "operator": "equals", "value": "{{legacy_cdn_id}}"},
           {"name": "menu_name", "operator": "equals", "value": "{{menu_name}}"},
           {"name": "plu_id", "operator": "equals", "value": "{{plu_id}}"},
           {"name": "option", "operator": "equals", "value": "plus_for_franchise"}
       ]
   }
   ```

2. **Imágenes del PLU**
   ```json
   {
       "name": "mxp_v2.USP_Propagate_Menu_Plus",
       "filters": [
           // Filtros similares al anterior
           {"name": "option", "operator": "equals", "value": "plus_for_franchise_images"}
       ]
   }
   ```

3. **Categorías del PLU**
   ```json
   {
       "name": "mxp_v2.USP_Propagate_Menu_Plus",
       "filters": [
           // Filtros similares
           {"name": "option", "operator": "equals", "value": "plus_for_franchise_categories"}
       ]
   }
   ```

4. **Preguntas y Respuestas**
   ```json
   {
       "name": "mxp_v2.USP_Propagate_Menu_Plus",
       "filters": [
           // Filtros similares
           {"name": "option", "operator": "equals", "value": "plus_for_franchise_questions"}
       ]
   }
   ```

5. **Impuestos**
   ```json
   {
       "name": "mxp_v2.USP_Propagate_Menu_Plus",
       "filters": [
           // Filtros similares
           {"name": "option", "operator": "equals", "value": "plus_for_franchise_taxes"}
       ]
   }
   ```

### 🔄 Procesamiento de Datos

#### Transformación (TRANSFORMS)
```json
{
    "code": "T001",
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "withResponse": true,
    "withCache": false
}
```

#### Carga (LOADS)
```json
{
    "code": "L001",
    "withCache": false,
    "withResponse": true,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "configuration": {
        "connection": {
            "server": "https://apim-dev-bo.azure-api.net",
            "repository": "/propagations/api/v1/propagationlegacy/startplupropagation",
            "adapter": "Rest"
        },
        "entities": [
            // Mapeo de campos para la carga
        ]
    }
}
```

### 📤 Respuesta Exitosa (200)
```json
{
    "code": 200,
    "message": "Proceso de propagación iniciado correctamente",
    "data": {
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "status": "IN_PROGRESS",
        "startedAt": "2024-09-15T16:15:00-05:00",
        "processes": [
            {
                "code": "E001",
                "status": "PENDING",
                "recordsProcessed": 0
            }
        ]
    }
}
```

### ⚠️ Códigos de Error Comunes
| Código | Descripción |
|--------|-------------|
| 400 | Solicitud incorrecta (parámetros inválidos) |
| 401 | No autorizado (token inválido o expirado) |
| 403 | Prohibido (permisos insuficientes) |
| 500 | Error interno del servidor |

### 🔄 Estados del Proceso
| Estado | Descripción |
|--------|-------------|
| PENDING | El proceso está en cola de espera |
| IN_PROGRESS | El proceso está en ejecución |
| COMPLETED | El proceso finalizó exitosamente |
| FAILED | El proceso falló |
| CANCELLED | El proceso fue cancelado |

### 📝 Notas de Implementación
1. **Seguridad**:
   - Todas las conexiones utilizan autenticación segura
   - Las credenciales sensibles nunca se devuelven en las respuestas
   - Se recomienda utilizar variables de entorno para almacenar credenciales

2. **Rendimiento**:
   - Se recomienda utilizar caché para consultas frecuentes
   - Las operaciones por lotes están optimizadas para grandes volúmenes de datos

3. **Manejo de Errores**:
   - Todos los errores son registrados con información detallada
   - Se implementan reintentos automáticos para fallos transitorios

### 📊 Métricas de Rendimiento
- **Tiempo de Respuesta Promedio**: < 1s (para confirmación de inicio)
- **Tiempo de Procesamiento**: Depende del volumen de datos
- **Disponibilidad**: 99.9% (SLA)
- **Tasa de Éxito**: > 99.5%
