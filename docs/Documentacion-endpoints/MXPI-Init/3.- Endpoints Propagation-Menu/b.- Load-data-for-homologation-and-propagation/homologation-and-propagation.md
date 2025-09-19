# Módulo de Homologación y Propagación de Menú (MXPi-init Propagation-Menu)

## Propósito y alcance
El módulo **MXPi-init Propagation-Menu** es el sistema especializado en la homologación y propagación de datos de menú dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo completo ETL para la conversión de identificadores legacy a UUIDs V2, asegurando la consistencia de datos de menú en toda la plataforma.

**Funciones principales:**
- Extracción de datos de menú desde sistemas legacy
- Transformación y homologación de identificadores
- Carga de datos homologados a MongoDB
- Propagación de datos para consumo por otros módulos

## Descripción general del sistema
**Microservicio:** mxpv2.integration.init.propagation-menu  
**Rol principal:** Gestionar procesos de homologación de datos de menú  
**Función crítica:** Asegurar la correcta conversión de IDs legacy a UUIDs V2 para menús, plus, preguntas sugeridas y respuestas

## Arquitectura y componentes

### Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de homologación | H000 | Punto de entrada principal para procesos de homologación de menú |
| 🔑 El H000 actúa como gateway unificado para solicitudes de homologación, validando estructura y parámetros requeridos |

### Capa de extracción (EXTRACTS)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Extractor SQL Server | E001 | Extrae datos desde bases de datos legacy SQL Server |
| Extractor MongoDB | E001 | Extrae datos desde colecciones MongoDB transformadas |

### Capa de transformación (TRANSFORMS)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Homologador de IDs | H001 | Transforma IDs legacy a UUIDs V2 y genera mapeo |

### Capa de carga (LOADS)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Cargador MongoDB | L001 | Inserta datos homologados en colección de UUIDs |

## Endpoints disponibles

### 1. Homologation-for-propagation-menu-plus
**POST** `{{server_init}}/api/v1/facade`

Homologa identificadores de productos Plus (PLUs) desde SQL Server a UUIDs V2.

**Entidades procesadas:**
- Plus (productos)

**Filtros aplicados:**
- cdn_id igual a franchise específico
- IDStatus específico para productos activos

### 2. Homologation-for-propagation-menu-suggested-questions
**POST** `{{server_init}}/api/v1/facade`

Homologa identificadores de preguntas sugeridas desde SQL Server a UUIDs V2.

**Entidades procesadas:**
- Pregunta_Sugerida (preguntas sugeridas)

**Filtros aplicados:**
- cdn_id igual a franchise específico
- IDStatus específico para preguntas activas

### 3. Homologation-for-propagation-menu-answer
**POST** `{{server_init}}/api/v1/facade`

Homologa identificadores de respuestas desde vista SQL Server a UUIDs V2.

**Entidades procesadas:**
- mxp_v2.View_Answer_For_Franchise (respuestas)

**Filtros aplicados:**
- cdn_id igual a franchise específico

### 4. Homologation-for-propagation-menu-menu
**POST** `{{server_init}}/api/v1/facade`

Homologa identificadores de menús desde SQL Server a UUIDs V2.

**Entidades procesadas:**
- Menu (menús)

**Filtros aplicados:**
- cdn_id igual a franchise específico
- IDStatus específico para menús activos

### 5. Homologation-for-propagation-Up-Selling
**POST** `{{server_init}}/api/v1/facade`

Homologa identificadores de up-selling desde colecciones MongoDB transformadas.

**Entidades procesadas:**
- transformation_collection_plu (datos de up-selling transformados)

**Filtros aplicados:**
- entity igual a "PlusColeccionDeDatos_UpSelling"

### 6. Homologation-for-propagation-restrictions
**POST** `{{server_init}}/api/v1/facade`

Homologa identificadores de restricciones desde colecciones MongoDB transformadas.

**Entidades procesadas:**
- transformation_restrictions (restricciones transformadas)

**Filtros aplicados:**
- cdn_id igual a franchise específico

## Estructura de solicitud común
Todos los endpoints comparten una estructura base con variaciones específicas por entidad:

```json
{
    "code": "H000",
    "withResponse": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": "{{legacy_cdn_id}}",
    "processes": {
        "EXTRACTS": [...],
        "TRANSFORMS": [...],
        "LOADS": [...],
        "PUBLISHERS": []
    }
}
```

## Gestión de procesos

### Flujo de trabajo estándar
1. **Extracción** → El E001 recupera datos desde la fuente (SQL Server o MongoDB)
2. **Validación** → Se verifica estructura e integridad de datos
3. **Transformación** → El H001 convierte IDs legacy a UUIDs V2
4. **Carga** → El L001 inserta los mapeos en la colección de UUIDs de MongoDB

### Configuración de conexiones
**Conexión SQL Server:**
```json
"connection": {
    "server": "{{server_legacy}}",
    "port": "",
    "user": "{{user_legacy}}",
    "password": "{{pass_legacy}}",
    "repository": "{{repository_legacy}}",
    "adapter": "SqlServer"
}
```

**Conexión MongoDB:**
```json
"connection": {
    "server": "{{server_mongo}}",
    "port": "",
    "user": "{{user_mongo}}",
    "password": "{{pass_mongo}}",
    "repository": "{{repository_mongo_hml}}",
    "adapter": "{{adapter_mongo}}"
}
```

## Estructura de datos de salida
Los datos homologados se almacenan en la colección "uuids" con la siguiente estructura:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id_v2 | string (PK) | UUID versión 2 generado |
| country_code | string | Código de país |
| franchise_id | string | Identificador del franchise |
| entity_name | string | Nombre de la entidad homologada |
| id_v1 | string | Identificador legacy original |
| created_at | timestamp | Fecha de creación del registro |

## Escenarios de uso

### ✅ Caso exitoso
1. Solicitud válida recibida en H000
2. Extracción exitosa de datos legacy
3. Transformación correcta a UUIDs V2
4. Carga exitosa en colección de UUIDs
5. Confirmación de proceso completado

### ⚠️ Posibles fallos
**Error en extracción:**
- Fuente de datos no disponible
- Credenciales incorrectas
- Datos no encontrados con los filtros aplicados

**Error en transformación:**
- Formato de ID legacy incompatible
- Error en generación de UUID

**Error en carga:**
- Conexión MongoDB no disponible
- Violación de constraints de unique key

## Políticas de reintento
- Reintentos automáticos en fallos de conexión
- Límite de 3 reintentos por etapa
- Backoff exponencial entre reintentos

## Monitorización
- Logs detallados por cada etapa del proceso
- Métricas de tiempo de ejecución
- Contadores de registros procesados/errores

## Preguntas frecuentes (para IA Support)

**Q: ¿Qué debo hacer si falla la homologación de un menú completo?**
A: Verificar que el franchise_id sea correcto y que existan menús activos con el IDStatus especificado.

**Q: ¿Cómo verificar que la homologación fue exitosa?**
A: Consultar la colección "uuids" en MongoDB filtrando por franchise_id y entity_name.

**Q: ¿Qué pasa si un ID legacy ya fue homologado previamente?**
A: El sistema detecta duplicados por la combinación franchise_id + entity_name + id_v1 y no crea registros duplicados.

**Q: ¿Cómo obtener los UUIDs V2 generados?**
A: Ejecutar consulta en MongoDB: 
```javascript
db.uuids.find({
    franchise_id: "franchise_id",
    entity_name: "entity_name"
})
```

## Referencias
- mxp-init/process/homologation-readme.md
- mxp-sync/process/readme.md (1-28)
- Microservicio: mxpv2.integration.init

## Contacto
**Equipo responsable:** Homologación y Propagación MAXPOINT  
**Canal de soporte:** #mxp-init-propagation (Slack interno)  
**Escalamiento:** Arquitectura de datos → Liderazgo técnico