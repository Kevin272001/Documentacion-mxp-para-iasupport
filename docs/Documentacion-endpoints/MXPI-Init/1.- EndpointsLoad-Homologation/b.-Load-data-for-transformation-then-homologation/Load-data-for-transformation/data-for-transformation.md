# Módulo de Transformación de Datos (MXPi-Init)

## Propósito y alcance
MXP Transform es el sistema de procesamiento y transformación de datos dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo de datos entre sistemas legacy y la nueva plataforma, aplicando transformaciones específicas para diferentes entidades y garantizando la integridad de los datos durante el proceso ETL (Extract, Transform, Load).

## Descripción general del sistema
- **Microservicio:** `mxpv2.integration.transform`
- **Rol principal:** Transformar datos legacy a formatos y estructuras compatibles con MAXPOINT V2.0
- **Función crítica:** Asegurar que los datos transformados mantengan consistencia y sean almacenados correctamente en MongoDB

## Arquitectura y componentes

### 3.1. Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de transformación | F001, F100 | Puntos de entrada para procesos de transformación de datos |

🔑 Las fachadas actúan como gateway unificado para datos a transformar, validando estructura e integridad

### 3.2. Capa de extracción
| Componente | Código | Descripción |
|------------|--------|-------------|
| Extractor SQL Server | E001 | Extrae datos de bases de datos legacy |
| Extractor MongoDB | E002 | Extrae datos de MongoDB para procesos de homologación |

### 3.3. Capa de transformación
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación Clasificaciones | T033 | Procesa datos de clasificaciones |
| Transformación Medios de Venta | T034 | Procesa medios de venta y half-integration |
| Transformación Imágenes Categoría | T035 | Procesa imágenes de categorías de menú |
| Transformación Colecciones PLU | T040 | Procesa colecciones y tipos PLU |

### 3.4. Capa de carga
| Componente | Código | Descripción |
|------------|--------|-------------|
| Loader MongoDB | L001 | Carga datos transformados a MongoDB |

## Endpoints de Transformación

### 1. Facade-Transform-Classification
- **Método:** POST
- **URL:** `{{server_init}}/api/v1/facade`
- **Autenticación:** Bearer Token

**Descripción**  
Transforma datos de clasificaciones desde SQL Server a MongoDB. Extrae clasificaciones activas del sistema legacy, las combina con datos de UUIDs de cadenas y aplica transformaciones específicas.

**Características principales:**
- Extrae clasificaciones con estado activo específico (`5b039503-85cf-e511-80c6-000d3a3261f3`)
- Combina con datos de UUIDs de cadenas desde MongoDB
- Almacena en colección `transformation_classifications`

**Estructura de salida en MongoDB:**
```json
{
  "cdn_id": "id_v1 de cadena",
  "entity": "tipo de entidad",
  "value": "valor transformado",
  "legacy_classification_id": "IDClasificacion original",
  "combined_id": "PK string",
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

### 2. Facade-Transform-Half-Integration
- **Método:** POST
- **URL:** `{{server_init}}/api/v1/facade`
- **Autenticación:** Bearer Token

**Descripción**  
Transforma datos de medios de venta para procesos de half-integration. Extrae información de la vista `mxp_v2.Medio_Venta` y la combina con UUIDs de cadenas.

**Características principales:**
- Extrae medios de venta, canales de menú y nombres de half-integration
- Combina con datos de UUIDs de cadenas desde MongoDB
- Almacena en colección `transformation_half_integration`

**Estructura de salida en MongoDB:**
```json
{
  "cdn_id": "id_v1 de cadena",
  "entity": "tipo de entidad",
  "value": "valor transformado",
  "legacy_medio_canal_menu_id": "ID de canal de menú",
  "combined_id": "PK string",
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

### 3. Facade-Transform-Collection-Type-Plu
- **Método:** POST
- **URL:** `{{server_init}}/api/v1/facade`
- **Autenticación:** Bearer Token

**Descripción**  
Transforma datos de colecciones y tipos PLU para gestión de productos. Extrae información de la vista `mxp_v2.Coleccion_Tipo_Plu`.

**Características principales:**
- Extrae tipos PLU, IDs de cadena legacy y IDs de colección
- Transforma con componente `T040` específico para colecciones PLU
- Almacena en colección `transformation_collection_plu`
- Campos de array con soporte para UUIDs

**Estructura de salida en MongoDB:**
```json
{
  "value": "PK string",
  "entity": "tipo de entidad",
  "legacy_cdn_id": "ID de cadena legacy",
  "legacy_collection_data_id": "array con UUIDs",
  "legacy_collection_id": "array con UUIDs",
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

### 4. Facade-Transform-Category-Image-For-Franchise
- **Método:** POST
- **URL:** `{{server_init}}/api/v1/facade`
- **Autenticación:** Bearer Token

**Descripción**  
Transforma datos de imágenes de categorías de menú para una franquicia específica. Extrae agrupaciones de menú activas filtradas por franquicia.

**Características principales:**
- Extrae agrupaciones de menú con estado activo específico (`78039503-85cf-e511-80c6-000d3a3261f3`)
- Filtra por ID de cadena específico (franquicia)
- Transforma con componente `T035` para imágenes de categoría
- Almacena en colección `transformation_category_image`

**Estructura de salida en MongoDB:**
```json
{
  "cdn_id": "ID de cadena",
  "entity": "tipo de entidad",
  "value": "valor transformado",
  "IDMenuAgrupacion": "ID de agrupación de menú",
  "combined_id": "PK string",
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

## Gestión de procesos

### Flujo de trabajo estándar
1. **Ingestión de datos** → La fachada (F001/F100) recibe la solicitud de transformación  
2. **Extracción** → Componentes E001/E002 extraen datos de SQL Server y MongoDB  
3. **Validación** → Se verifica estructura e integridad de los datos extraídos  
4. **Transformación** → Componentes T033-T040 aplican transformaciones específicas  
5. **Carga** → Componente L001 almacena datos transformados en MongoDB  
6. **Confirmación** → Retorno de resultado según parámetro `withResponse`  

### Parámetros comunes de configuración
- **withResponse:** Controla si el proceso es síncrono o asíncrono  
- **withCache:** Habilita/deshabilita uso de caché  
- **syncId:** Identificador único de sincronización  
- **country:** Código de país para filtrado  

## Puntos de integración
- **Sistemas Legacy:** SQL Server con datos originales  
- **Almacenamiento MongoDB:** Para datos de UUIDs y resultados transformados  
- **Sistema de Autenticación:** Token Bearer para seguridad  

## Patrones de transformación
- **Extracción dual:** Combinación de datos de SQL Server y MongoDB  
- **Transformación especializada:** Diferentes componentes T0xx para distintos tipos de datos  
- **Almacenamiento estructurado:** Esquemas consistentes en MongoDB  
- **Filtrado por estado:** Uso de IDStatus para datos activos  

## Escenarios de uso
✅ **Caso exitoso**  
Datos legacy válidos → extracción exitosa → transformación específica → almacenamiento en MongoDB → confirmación  

⚠️ **Posibles fallos**
- **Error en extracción:**
  - Conexión fallida con SQL Server o MongoDB
  - Datos corruptos o en formato incorrecto
  - Filtros que no retornan resultados
- **Error en transformación:**
  - Datos incompatibles con la transformación especificada
  - Error en lógica de transformación del componente T0xx
- **Error en carga:**
  - Conexión fallida con MongoDB de destino
  - Violación de constraints o esquema

## Preguntas frecuentes (para IA Support)
**Q:** ¿Qué diferencia hay entre los diferentes componentes de transformación (T033, T034, T035, T040)?  
**A:** Cada componente está especializado en un tipo de dato específico: clasificaciones, medios de venta, imágenes de categoría y colecciones PLU respectivamente.

**Q:** ¿Cómo verificar que la transformación fue exitosa?  
**A:** Revisar las colecciones correspondientes en MongoDB (`transformation_classifications`, `transformation_half_integration`, etc.) y validar que los datos estén presentes y correctos.

**Q:** ¿Los procesos de transformación son síncronos o asíncronos?  
**A:** Depende del parámetro `withResponse`. Si es false, el proceso es asíncrono.

**Q:** ¿Qué hacer si falla la transformación?  
**A:** Verificar logs del componente E001/E002 (extracción) y del componente T0xx (transformación), validar conectividad con las bases de datos y verificar que los datos de entrada cumplan con los formatos esperados.

## Referencias
- Documentación interna: `mxp-transform/process/readme.md`
- Esquema de base de datos: `legacy-sql/schema.md`
- Estructura MongoDB: `mxp-storage/transformation-collections.md`

## Contacto
- **Equipo responsable:** Transformación de Datos MAXPOINT  
- **Canal de soporte:** #mxp-data-transform (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico  
