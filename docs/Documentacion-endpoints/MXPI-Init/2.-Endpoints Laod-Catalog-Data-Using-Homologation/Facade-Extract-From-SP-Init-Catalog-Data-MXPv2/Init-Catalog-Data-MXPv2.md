Aquí tienes la documentación completa en formato MD:

# Módulo de Extracción de Datos de Catálogo (Facade-Extract-From-SP-Init-Catalog-Data-MXPv2)

## Propósito y alcance

El módulo Facade-Extract-From-SP-Init-Catalog-Data-MXPv2 es el sistema especializado en la extracción y transformación de datos de catálogos dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo de datos catalogados desde sistemas legacy hacia las colecciones MongoDB del nuevo sistema. Garantiza la coherencia de los datos maestros en toda la plataforma. Coordina procesos ETL (Extract, Transform, Load) para múltiples tipos de catálogos del sistema.

ℹ️ **Para más detalles de los módulos relacionados, consulta**: 
- MXP Init 
- MXP FE Build 
- MXP Printer

## Descripción general del sistema

**Microservicio**: mxpv2.integration.catalog  
**Rol principal**: Gestionar procesos de extracción y transformación de datos catalogados desde SQL Server Legacy.  
**Función crítica**: Asegurar que los datos de configuración y catálogos del sistema legacy sean correctamente migrados y transformados al nuevo esquema de MAXPOINT V2.0.

## Arquitectura y componentes

### 3.1. Capa de fachada

| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de recepción ETL | F100 | Punto de entrada principal para solicitudes de procesamiento ETL de catálogos. 🔑 El F100 actúa como gateway unificado para datos catalogados, validando estructura e integridad. |

### 3.2. Capa de extracción

| Componente | Código | Descripción |
|------------|--------|-------------|
| Extracción desde SQL Server | E001 | Extrae datos de catálogos desde stored procedures de SQL Server Legacy. |
| Extracción de UUIDs desde MongoDB | E002 | Obtiene UUIDs y referencias desde colecciones de MongoDB para homologación. |

### 3.3. Capa de transformación

| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación Status Suggested Question | T030 | Procesa datos de estados de preguntas sugeridas. |
| Transformación Status Menu | T029 | Procesa datos de estados de menú. |
| Transformación Device Images Menu | T028 | Procesa datos de dispositivos de imágenes de menú. |
| Transformación Type Question | T027 | Procesa datos de tipos de preguntas. |
| Transformación Status PLU | T026 | Procesa datos de estados de PLU. |
| Transformación Status Category Price PLU | T025 | Procesa datos de estados de categorías de precio PLU. |
| Transformación Device Images PLU | T024 | Procesa datos de dispositivos de imágenes PLU. |

### 3.4. Capa de carga

| Componente | Código | Descripción |
|------------|--------|-------------|
| Carga a MongoDB | L001 | Inserta los datos transformados en las colecciones destino de MongoDB BO_Menus. |

## Gestión de procesos

### Flujo de trabajo estándar

1. **Ingestión de datos** → El F100 recibe las solicitudes de procesamiento de catálogos
2. **Extracción** → Se ejecutan E001 y E002 para obtener datos desde SQL Server y MongoDB
3. **Validación** → Se verifica estructura e integridad de los datos catalogados
4. **Transformación** → Los procesos T024-T030 transforman los datos según el tipo de catálogo
5. **Carga** → Los datos procesados se insertan en las colecciones MongoDB correspondientes

## Puntos de integración

El módulo de catálogos conecta múltiples sistemas dentro del ecosistema:

- **Entrada**: 
  - SQL Server Legacy mediante stored procedures USP_Catalog_Data_MXPLegacy
  - MongoDB para UUIDs y referencias de homologación
- **Salida**: 
  - Colecciones Menus_BO en MongoDB (Status_Suggested_Question, Status_Menu, Device_Images_Menu, etc.)
- **Rol central**: Mantiene coherencia en datos maestros y catalogados entre sistemas legacy y nuevo ecosistema

## Endpoints

### POST /api/v1/facade

**Descripción**: Endpoint principal para procesamiento de datos de catálogos MXPv2

**Autenticación**: Bearer Token

**Headers**:
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

#### Catálogos disponibles:

1. **Status Suggested Question** (`catalog_mxpv2_status_suggested_question`)
2. **Status Menu** (`catalog_mxpv2_status_menu`) 
3. **Device Images Menu** (`catalog_mxpv2_device_images_menu`)
4. **Type Question** (`catalog_mxpv2_type_question`)
5. **Status PLU** (`catalog_mxpv2_status_plu`)
6. **Status Category Price PLU** (`catalog_mxpv2_status_category_price_plu`)
7. **Device Images PLU** (`catalog_mxpv2_device_images_plu`)

#### Estructura base de la solicitud:

```json
{
    "code": "F100",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "processes": {
        "EXTRACTS": [
            {
                "code": "E001",
                "withResponse": true,
                "withCache": false,
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "configuration": {
                    "connection": {
                        "server": "{{server_legacy}}",
                        "port": "",
                        "user": "{{user_legacy}}",
                        "password": "{{pass_legacy}}",
                        "repository": "{{repository_legacy}}",
                        "adapter": "SqlServerSP"
                    },
                    "entities": [
                        {
                            "name": "mxp_v2.USP_Catalog_Data_MXPLegacy",
                            "properties": [],
                            "filters": [
                                {
                                    "name": "country",
                                    "operator": "equals",
                                    "value": "{{country}}"
                                },
                                {
                                    "name": "cdn_id",
                                    "operator": "equals",
                                    "value": "0"
                                },
                                {
                                    "name": "option",
                                    "operator": "equals",
                                    "value": "nombre_del_catalogo"
                                }
                            ]
                        }
                    ]
                }
            }
        ],
        "TRANSFORMS": [
            {
                "code": "T0XX",
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "withResponse": true,
                "withCache": false,
                "country": "{{country}}"
            }
        ],
        "LOADS": [
            {
                "code": "L001",
                "withResponse": true,
                "withCache": false,
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "configuration": {
                    "connection": {
                        "server": "{{server_mongo}}",
                        "port": "",
                        "user": "{{user_mongo}}",
                        "password": "{{pass_mongo}}",
                        "repository": "UAT_KFC_BO_Menus",
                        "adapter": "{{adapter_mongo}}"
                    },
                    "entities": [
                        {
                            "name": "nombre_catalogo_origen",
                            "toName": "Nombre_Coleccion_Destino",
                            "properties": [
                                // Mapeo de propiedades específicas
                            ]
                        }
                    ]
                }
            }
        ]
    }
}
```

#### Ejemplo usando cURL para Status Suggested Question:

```bash
curl --location -g '{{server_init}}/api/v1/facade' \
--header 'Authorization: Bearer {{token}}' \
--header 'Content-Type: application/json' \
--data '{
    "code": "F100",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "processes": {
        "EXTRACTS": [
            {
                "code": "E001",
                "withResponse": true,
                "withCache": false,
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "configuration": {
                    "connection": {
                        "server": "{{server_legacy}}",
                        "port": "",
                        "user": "{{user_legacy}}",
                        "password": "{{pass_legacy}}",
                        "repository": "{{repository_legacy}}",
                        "adapter": "SqlServerSP"
                    },
                    "entities": [
                        {
                            "name": "mxp_v2.USP_Catalog_Data_MXPLegacy",
                            "properties": [],
                            "filters": [
                                {
                                    "name": "country",
                                    "operator": "equals",
                                    "value": "{{country}}"
                                },
                                {
                                    "name": "cdn_id",
                                    "operator": "equals",
                                    "value": "0"
                                },
                                {
                                    "name": "option",
                                    "operator": "equals",
                                    "value": "catalog_mxpv2_status_suggested_question"
                                }
                            ]
                        }
                    ]
                }
            }
        ],
        "TRANSFORMS": [
            {
                "code": "T030",
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "withResponse": true,
                "withCache": false,
                "country": "{{country}}"
            }
        ],
        "LOADS": [
            {
                "code": "L001",
                "withResponse": true,
                "withCache": false,
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "configuration": {
                    "connection": {
                        "server": "{{server_mongo}}",
                        "port": "",
                        "user": "{{user_mongo}}",
                        "password": "{{pass_mongo}}",
                        "repository": "UAT_KFC_BO_Menus",
                        "adapter": "{{adapter_mongo}}"
                    },
                    "entities": [
                        {
                            "name": "catalog_mxpv2_status_suggested_question",
                            "toName": "Menus_BO_Status_Suggested_Question",
                            "properties": [
                                {
                                    "fromName": "homologation_id",
                                    "fromType": "",
                                    "toName": "_id",
                                    "toType": "uuid",
                                    "defaultValue": "",
                                    "isPrimaryKey": "true"
                                },
                                {
                                    "fromName": "status_name",
                                    "fromType": "",
                                    "toName": "status_name",
                                    "toType": "",
                                    "defaultValue": ""
                                }
                                // ... más propiedades
                            ]
                        }
                    ]
                }
            }
        ]
    }
}'
```

## Parámetros de configuración

### Conexiones configuradas:

1. **SQL Server Legacy**:
   - server: {{server_legacy}}
   - repository: {{repository_legacy}}
   - adapter: SqlServerSP

2. **MongoDB**:
   - server: {{server_mongo}}
   - repository: UAT_KFC_BO_Menus
   - adapter: {{adapter_mongo}}

### Variables de entorno requeridas:
- {{server_init}} - URL del servidor de inicialización
- {{token}} - Token de autenticación Bearer
- {{country}} - Código de país para filtrar datos
- {{server_legacy}}, {{user_legacy}}, {{pass_legacy}} - Credenciales SQL Server Legacy
- {{server_mongo}}, {{user_mongo}}, {{pass_mongo}} - Credenciales MongoDB

## Patrones de procesamiento

**Recepción centralizada** → Todos los ETL de catálogos llegan vía F100  
**Procesamiento especializado** → Cada tipo de catálogo tiene su transformador específico (T024-T030)  
**Carga estructurada** → Los datos son mapeados a esquemas MongoDB específicos  
**Garantía de coherencia** → Evita inconsistencias en datos maestros de la plataforma

## Escenarios de uso

### ✅ Caso exitoso
ETL válido → se recibe en F100 → validación → extracción exitosa → transformación especializada → carga correcta en MongoDB.

### ⚠️ Posibles fallos

**Error en validación**: Datos rechazados por estructura incorrecta. Se genera log de error con detalle del paquete fallido.  
**Acción recomendada**: Verificar la fuente de datos y la estructura del request.

**Error en extracción (E001)**: Conexión a SQL Server fallida. Se activa alerta al sistema de monitoreo.  
**Acción recomendada**: Verificar conectividad con SQL Server Legacy.

**Error en transformación (T024-T030)**: Datos no pasan a carga. Se activa rollback y alerta.  
**Acción recomendada**: Revisar logs del transformador específico.

**Error en carga MongoDB**: Colección de destino no disponible. El sistema reintenta según política de reintentos.  
**Acción recomendada**: Verificar que la colección MongoDB exista y tenga permisos de escritura.

## Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si falla la validación en F100?**  
A: El paquete se descarta, se registra en logs y se debe revisar la estructura del request enviado.

**Q: ¿Cómo sé que los datos de catálogo se procesaron correctamente?**  
A: Revisa el log del transformador específico (T024-T030). Si hay éxito, se marca con el código correspondiente.

**Q: ¿El módulo almacena datos temporalmente?**  
A: Solo durante el procesamiento en memoria. No persiste datos beyond el scope de la solicitud.

**Q: ¿Qué debo hacer si la carga a MongoDB falla persistentemente?**  
A: Verificar la conectividad con MongoDB, revisar los logs de carga y validar los permisos de escritura en las colecciones BO_Menus.

**Q: ¿Por qué el cdn_id siempre es "0" en los filters?**  
A: Porque estos catálogos son maestros globales que no dependen de una cadena específica.

**Q: ¿Cómo obtener el syncId correcto?**  
A: El syncId debe ser un UUID v4 generado para cada ejecución, puede usar herramientas de generación de UUID.

## Referencias

**Fuentes internas**: 
- mxp-catalog/process/readme.md 
- mxp-init/process/readme.md

**Microservicio**: mxpv2.integration.catalog  
**Esquemas MongoDB**: UAT_KFC_BO_Menus ( diversas colecciones BO)  
**Stored Procedures**: mxp_v2.USP_Catalog_Data_MXPLegacy

## Contacto

**Equipo responsable**: Catálogos MAXPOINT  
**Canal de soporte**: #mxp-catalogos (Slack interno)  
**Escalamiento**: Arquitectura de datos → Liderazgo técnico