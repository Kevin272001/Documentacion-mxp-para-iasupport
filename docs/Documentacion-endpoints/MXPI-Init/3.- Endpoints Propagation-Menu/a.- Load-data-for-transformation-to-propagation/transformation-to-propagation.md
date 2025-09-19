# Módulo de Propagación de Menú (MXPi-init Propagation-Menu)

## Propósito y alcance
MXPi-init Propagation-Menu es el sistema especializado en la carga y transformación de datos para la propagación de menús dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo de datos relacionados con:
- Upselling para franquicias
- Restricciones de categorías de menú
- Restricciones de Plus
- Restricciones de Menú

Garantiza la correcta transformación y carga de datos de menú desde sistemas legacy hacia la base de datos MongoDB de destino.

## Descripción general del sistema
**Microservicio**: mxpv2.integration.init.propagation-menu  
**Rol principal**: Gestionar procesos ETL para datos de propagación de menús  
**Función crítica**: Asegurar que los datos transformados de menús lleguen correctamente a la base de datos de destino

## Arquitectura y componentes

### Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de transformación de Upselling | F100 | Punto de entrada para transformación de datos de upselling |
| Interfaz de transformación de Restricciones | F100 | Punto de entrada para transformación de datos de restricciones |

### Capa de extracción
| Componente | Código | Descripción |
|------------|--------|-------------|
| Extractora SQL Server | E001 | Extrae datos desde bases de datos SQL Server legacy |

### Capa de transformación
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformadora de Upselling | T036 | Procesa datos de upselling para franquicias |
| Transformadora de Restricciones-Categoría | T037 | Procesa restricciones de categoría de menú |
| Transformadora de Restricciones-Menú | T038 | Procesa restricciones de menú |
| Transformadora de Restricciones-Plus | T039 | Procesa restricciones de plus |

### Capa de carga
| Componente | Código | Descripción |
|------------|--------|-------------|
| Cargadora MongoDB | L001 | Inserta datos transformados en MongoDB |

## Gestión de procesos

### Flujo de trabajo estándar
1. **Ingestión de datos** → El F100 recibe las solicitudes de transformación
2. **Extracción** → E001 extrae datos desde SQL Server legacy
3. **Validación** → Se verifica estructura e integridad de los datos
4. **Transformación** → Los transformadores específicos (T036-T039) procesan los datos
5. **Carga** → L001 inserta los datos transformados en MongoDB

## Puntos de integración
El módulo de propagación de menú se conecta con:

**Entradas**:
- Bases de datos SQL Server legacy
- Sistema de autenticación mediante Bearer Token

**Salidas**:
- Base de datos MongoDB para almacenamiento de datos transformados

## Endpoints disponibles

### 1. Transformación de Upselling para Franquicia
**Método**: POST  
**Endpoint**: `{{server_init}}/api/v1/facade`  
**Autenticación**: Bearer Token  
**Transformador**: T036  
**Descripción**: Transforma datos de upselling desde SQL Server a MongoDB

**Estructura de datos de origen**:
```json
{
    "entities": [
        {
            "name": "mxp_v2.Coleccion_Plu_UpSelling",
            "properties": [
                {"name": "cdn_id", "type": "base64"},
                {"name": "legacy_collection_id", "type": "base64"},
                {"name": "legacy_collection_data_id", "type": "base64"},
                {"name": "plu_father", "type": "base64"}
            ]
        }
    ]
}
```

**Estructura de datos de destino**:
```json
{
    "toName": "transformation_collection_plu",
    "properties": [
        {"fromName": "combined_id", "toName": "value", "isPrimaryKey": "true"},
        {"fromName": "entity", "toName": "entity"},
        {"fromName": "cdn_id", "toName": "legacy_cdn_id"},
        {"fromName": "legacy_collection_data_id", "toName": "legacy_collection_data_id", "toType": "array"},
        {"fromName": "legacy_collection_id", "toName": "legacy_collection_id"},
        {"fromName": "plu_father", "toName": "plu_id"},
        {"toName": "created_at", "toType": "timestamp", "defaultValue": "true"},
        {"toName": "updated_at", "toType": "timestamp", "defaultValue": "true"}
    ]
}
```

### 2. Transformación de Restricciones-Categoría para Franquicia
**Método**: POST  
**Endpoint**: `{{server_init}}/api/v1/facade`  
**Autenticación**: Bearer Token  
**Transformador**: T037  
**Descripción**: Transforma restricciones de categoría de menú desde SQL Server a MongoDB

**Estructura de datos de origen**:
```json
{
    "entities": [
        {
            "name": "Menu_Agrupacion",
            "properties": [
                {"name": "IDMenuAgrupacion", "type": "base64"},
                {"name": "cdn_id", "type": "base64"},
                {"name": "mag_descripcion", "type": "base64"}
            ]
        }
    ]
}
```

### 3. Transformación de Restricciones-Plus para Franquicia
**Método**: POST  
**Endpoint**: `{{server_init}}/api/v1/facade`  
**Autenticación**: Bearer Token  
**Transformador**: T039  
**Descripción**: Transforma restricciones de plus desde SQL Server a MongoDB

### 4. Transformación de Restricciones-Menú para Franquicia
**Método**: POST  
**Endpoint**: `{{server_init}}/api/v1/facade`  
**Autenticación**: Bearer Token  
**Transformador**: T038  
**Descripción**: Transforma restricciones de menú desde SQL Server a MongoDB

## Configuraciones de conexión

### Conexión SQL Server (Origen)
```json
{
    "connection": {
        "server": "{{server_legacy}}",
        "user": "{{user_legacy}}",
        "password": "{{pass_legacy}}",
        "repository": "{{repository_legacy}}",
        "adapter": "SqlServer"
    }
}
```

### Conexión MongoDB (Destino)
```json
{
    "connection": {
        "server": "{{server_mongo}}",
        "user": "{{user_mongo}}",
        "password": "{{pass_mongo}}",
        "repository": "{{repository_mongo_hml}}",
        "adapter": "{{adapter_mongo}}"
    }
}
```

## Ejemplo de solicitud CURL

```bash
curl --location -g '{{server_init}}/api/v1/facade' \
--header 'Authorization: Bearer {{token}}' \
--header 'Content-Type: application/json' \
--data '{
    "code": "F100",
    "withResponse": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "processes": {
        "EXTRACTS": [...],
        "TRANSFORMS": [...],
        "LOADS": [...]
    }
}'
```

## Escenarios de uso

### ✅ Caso exitoso
1. Solicitud válida recibida en F100
2. Extracción exitosa con E001
3. Transformación exitosa con el transformador correspondiente (T036-T039)
4. Carga exitosa en MongoDB con L001
5. Confirmación de proceso completado

### ⚠️ Posibles fallos
**Error en extracción (E001)**:
- Datos no disponibles en SQL Server
- Error de conexión a la base de datos
- **Acción**: Verificar conectividad y disponibilidad de datos

**Error en transformación (T036-T039)**:
- Datos incompletos o en formato incorrecto
- **Acción**: Verificar estructura de datos de origen

**Error en carga (L001)**:
- MongoDB no disponible
- Error de validación de datos
- **Acción**: Verificar conectividad a MongoDB y esquema de datos

## Políticas de reintentos
- Reintentos automáticos en fallos de conexión
- Límite de 3 reintentos con retroceso exponencial
- Registro de errores en sistema de logging centralizado

## Preguntas frecuentes (para IA Support)

**Q: ¿Qué hacer si falla la extracción de datos?**  
A: Verificar la conectividad con SQL Server y que los datos existan con los filtros aplicados.

**Q: ¿Cómo verificar que la transformación fue exitosa?**  
A: Revisar los logs del transformador correspondiente (T036-T039).

**Q: ¿Dónde se almacenan los datos transformados?**  
A: En la colección de MongoDB especificada en la configuración de L001.

**Q: ¿Qué significa el campo "withResponse"?**  
A: Indica si el proceso debe esperar y retornar una respuesta sincrónica (true) o es asincrónico (false).

## Referencias
- **Documentación de MXP Init**: mxp-init/process/readme.md
- **Documentación de MXP Sync**: mxp-sync/process/readme.md
- **Microservicio**: mxpv2.integration.init.propagation-menu

## Contacto
**Equipo responsable**: Propagación de Datos MAXPOINT  
**Canal de soporte**: #mxp-data-propagation (Slack interno)  
**Escalamiento**: Líder de Arquitectura de Datos → Liderazgo técnico