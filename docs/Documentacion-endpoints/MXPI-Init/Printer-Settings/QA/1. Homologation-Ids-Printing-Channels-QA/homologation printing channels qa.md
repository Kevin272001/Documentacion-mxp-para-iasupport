# Documentación Módulo MXPI-INIT Printer Settings QA

## Propósito y alcance
MXPI-INIT Printer Settings QA es el módulo de homologación de identificadores para configuraciones de impresión dentro del ecosistema MAXPOINT V2.0 en entorno de calidad. Gestiona la generación y sincronización de UUIDs para entidades relacionadas con impresión, garantizando la consistencia de identificadores entre sistemas legacy y la nueva plataforma.

## Descripción general del sistema
**Microservicio**: mxpv2.integration.init.printer.qa  
**Rol principal**: gestionar procesos de homologación de UUIDs para impresión  
**Función crítica**: asegurar que los identificadores de entidades de impresión estén correctamente mapeados entre sistemas legacy y V2.0

## Arquitectura y componentes

### Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de homologación | H000 | Punto de entrada principal para procesos de homologación de UUIDs |

### Capa de extracción (Extracts)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Extracción SQL Server | E001 | Extrae datos de tablas de SQL Server para homologación |

### Capa de transformación (Transforms)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación homologación | H001 | Genera UUIDs V2 y prepara datos para homologación |

### Capa de carga (Loads)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Carga MongoDB UUIDs | L001 | Inserta UUIDs homologados en colección `uuids` de MongoDB |

## Endpoints documentados

### 1. Homologación de Canales de Impresión
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Homologar identificadores de canales de impresión desde SQL Server a MongoDB

**Flujo**:
- Extrae canales de impresión de SQL Server (E001)
- Filtra por cadena (`cdn_id`) y estado activo (`IDStatus`)
- Transforma datos con H001 (generación de UUIDs V2)
- Carga en colección `uuids` de MongoDB

**Estructura de datos de salida**:
```json
{
  "id_v2": "string (UUID)",
  "country_code": "string",
  "franchise_id": "string",
  "entity_name": "string (Canal_Impresion)",
  "id_v1": "string (IDCanalImpresion)",
  "created_at": "timestamp"
}
```

**Filtros aplicados**:
- `cdn_id`: igual al ID de cadena legacy
- `IDStatus`: igual a "8c039503-85cf-e511-80c6-000d3a3261f3" (estado activo)

### 2. Homologación de Documentos de Impresión
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Homologar identificadores de documentos de impresión desde SQL Server a MongoDB

**Flujo**:
- Extrae documentos de impresión de SQL Server (E001)
- Filtra por cadena (`cdn_id`)
- Transforma datos con H001 (generación de UUIDs V2)
- Carga en colección `uuids` de MongoDB

**Estructura de datos de salida**:
```json
{
  "id_v2": "string (UUID)",
  "country_code": "string",
  "franchise_id": "string",
  "entity_name": "string (mxp_v2.View_Printing_Documents)",
  "id_v1": "string (documento)",
  "created_at": "timestamp"
}
```

**Filtros aplicados**:
- `cdn_id`: igual al ID de cadena legacy

## Gestión de procesos

### Flujo de trabajo estándar
1. **Iniciación** → El H000 recibe la solicitud de homologación
2. **Extracción** → E001 obtiene datos de SQL Server con filtros específicos
3. **Transformación** → H001 genera UUIDs V2 y estructura datos de homologación
4. **Carga** → L001 inserta los UUIDs homologados en MongoDB
5. **Confirmación** → Se retorna estado de la operación (syncId, data: null, cacheId: null)

### Configuración de conexiones
- **SQL Server**: Conexión legacy con autenticación por usuario/contraseña
- **MongoDB QA**: Conexión al entorno de calidad con credenciales específicas
- **Adapter**: Sistema de adaptación para diferentes tipos de bases de datos

## Puntos de integración
El módulo conecta con:
- **SQL Server Legacy**: Datos originales de canales y documentos de impresión
- **MongoDB QA**: Almacenamiento de UUIDs homologados en entorno de calidad
- **Sistema de Homologación**: Generación consistente de identificadores V2

## Patrones de homologación
- **Extracción filtrada**: Datos obtenidos con criterios específicos por entidad
- **Generación consistente**: UUIDs V2 generados determinísticamente
- **Estructura uniforme**: Schema consistente para todos los UUIDs homologados
- **Metadatos completos**: Inclusión de información de país, cadena y timestamp

## Escenarios de uso

### ✅ Caso exitoso
Solicitud H000 válida → extracción exitosa → transformación H001 → carga exitosa en MongoDB → respuesta 200 con syncId

### ⚠️ Posibles fallos
- **Error en extracción**: Datos no disponibles en SQL Server
- **Error en conexión**: Base de datos no disponible
- **Error en transformación**: Fallo en generación de UUIDs
- **Error en carga**: MongoDB QA no disponible

## Validaciones de respuesta
El sistema implementa tests automatizados para verificar:
- Código de estado HTTP 200
- Tiempo de respuesta menor a 1000ms
- Content-Type: application/json
- Estructura correcta de respuesta: {syncId, data, cacheId}
- Valores nulos en data y cacheId
- syncId como string válido

## Preguntas frecuentes (para IA Support)

**Q: ¿Qué hacer si falla la homologación?**  
A: Verificar que los datos existan en SQL Server con los filtros aplicados y que las conexiones estén configuradas correctamente.

**Q: ¿Cómo verificar que una homologación fue exitosa?**  
A: Revisar la colección `uuids` en MongoDB QA y confirmar que existan los registros con los entity_name correspondientes.

**Q: ¿Los procesos son idempotentes?**  
A: Sí, la carga utiliza `isPrimaryKey: true` que evita duplicados basado en `id_v2`.

**Q: ¿Qué significa el IDStatus "8c039503-85cf-e511-80c6-000d3a3261f3"?**  
A: Es el identificador único que representa el estado "Activo" en el sistema legacy.

## Referencias
- **Fuentes internas**: mxpi-init/printer-settings-qa/readme.md
- **Microservicio**: mxpv2.integration.init.printer.qa
- **Colección MongoDB**: uuids (en repository_mongo_hml)

## Contacto
**Equipo responsable**: Homologación y Calidad MAXPOINT  
**Canal de soporte**: #mxp-homologacion-qa (Slack interno)  
**Escalamiento**: Arquitectura de Datos → Liderazgo técnico