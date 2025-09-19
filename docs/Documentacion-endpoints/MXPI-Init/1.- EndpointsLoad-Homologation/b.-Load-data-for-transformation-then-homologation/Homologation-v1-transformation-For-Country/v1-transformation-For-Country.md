# Módulo de transformation-For-Country (MXPi-Init)

## Propósito y alcance
MXPi-Init es el sistema de inicialización y homologación de datos dentro del ecosistema MAXPOINT V2.0.  
Gestiona la preparación de datos para su transformación y sincronización.  
Garantiza la consistencia del sistema al inicializar procesos ETL (Extract, Transform, Load).  
Coordina la ejecución de procesos de extracción, transformación y carga hacia las fachadas de destino.

ℹ️ Para más detalles de los módulos relacionados, consulta: MXP Sync, MXP FE Build, Impresora MXP

## Descripción general del sistema
- **Microservicio:** `mxpv2.integration.init`
- **Rol principal:** Inicializar procesos de integración y homologación de datos.
- **Función crítica:** Asegurar que los datos legacy estén correctamente preparados para su transformación y carga.

## Arquitectura y componentes

### 3.1. Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de recepción ETL | F008 | Punto de entrada principal para datos ETL (extract, transform, load) |

🔑 El F008 actúa como gateway unificado para datos inicializados, validando estructura e integridad.

### 3.2. Capa de transformación
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación Colección PLU | H001 | Procesa datos de colección PLU, aplicando lógica de inicialización y transformación |

### 3.3. Capa de extracción y carga
| Componente | Código | Descripción |
|------------|--------|-------------|
| Extractor MongoDB | E001 | Extrae datos desde MongoDB según configuración de entidad y filtros | 
| Loader MongoDB | L001 | Carga datos inicializados y transformados hacia MongoDB |

## Ejemplo de Request - Homologation Collection Type PLU

**Método:** POST  
**URL:** `{{server_init}}/api/v1/facade`  
**Autenticación:** Bearer Token  

**Body (JSON):**
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": 0,
    "processes": {
        "EXTRACTS": [
            {
                "code": "E001",
                "withResponse": true,
                "withCache": false,
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "configuration": {
                    "connection": {
                        "server": "{{server_mongo}}",
                        "port": "",
                        "user": "{{user_mongo}}",
                        "password": "{{pass_mongo}}",
                        "repository": "{{repository_mongo_hml}}",
                        "adapter": "{{adapter_mongo}}"
                    },
                    "entities": [
                        {
                            "name": "transformation_collection_plu",
                            "properties": [
                                {"name": "value", "type": "base64"}
                            ],
                            "filters": [
                                {
                                    "name": "entity",
                                    "operator": "equals",
                                    "value": "ColeccionDeDatosPlus_TipoProducto"
                                }
                            ]
                        }
                    ]
                }
            }
        ],
        "TRANSFORMS": [
            {
                "code": "H001",
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "withResponse": true,
                "withCache": false,
                "country": "{{country}}"
            }
        ],
        "LOADS": [
            {
                "code": "L001",
                "withCache": false,
                "withResponse": true,
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "configuration": {
                    "connection": {
                        "server": "{{server_mongo}}",
                        "port": "",
                        "user": "{{user_mongo}}",
                        "password": "{{pass_mongo}}",
                        "repository": "{{repository_mongo_hml}}",
                        "adapter": "{{adapter_mongo}}"
                    },
                    "entities": [
                        {
                            "name": "uuids",
                            "toName": "uuids",
                            "properties": [
                                {"fromName": "id_v2","toName": "id_v2","toType": "string","isPrimaryKey":"true"},
                                {"fromName": "country_code","toName": "country_code"},
                                {"fromName": "franchise_id","toName": "franchise_id"},
                                {"fromName": "entity_name","toName": "entity_name"},
                                {"fromName": "id_v1","toName": "id_v1"},
                                {"toName": "created_at","toType": "timestamp","defaultValue":"true"}
                            ]
                        }
                    ]
                },
                "dataInput": {}
            }
        ],
        "PUBLISHERS": []
    }
}
```

**Ejemplo con curl:**
```bash
curl --location -g '{{server_init}}/api/v1/facade' --data '{ ... }'
```

## Gestión de procesos

### Flujo de trabajo estándar
1. **Ingestión de datos** → El F008 recibe los paquetes de inicialización.  
2. **Validación** → Se verifica estructura e integridad.  
3. **Extracción** → Componente E001 extrae datos desde MongoDB.  
4. **Transformación** → Componente H001 aplica lógica de inicialización de PLU.  
5. **Carga** → Componente L001 almacena datos inicializados en MongoDB.  
6. **Confirmación** → Retorno de resultado según parámetro `withResponse`.  

### Puntos de integración
- **Entrada:** Datos legacy desde sistemas previos.  
- **Salida:** Datos transformados hacia fachadas y microservicios destino.  
- **Rol central:** Garantiza coherencia en la inicialización de datos para procesos ETL.

### Escenarios de uso
✅ **Caso exitoso:** Datos legacy válidos → extracción correcta → transformación H001 → carga L001 → confirmación.  
⚠️ **Posibles fallos:**  
- Error en extracción (E001): problemas de conexión o datos corruptos.  
- Error en transformación (H001): datos incompatibles o falla de lógica.  
- Error en carga (L001): falla de conexión o constraints de MongoDB.  

### Preguntas frecuentes (para IA Support)
**Q:** ¿Qué hacer si falla la extracción?  
**A:** Verificar conectividad con MongoDB y validar configuración del extractor E001.  

**Q:** ¿Cómo saber si la transformación H001 fue exitosa?  
**A:** Revisar logs del componente H001 y confirmar que los datos se cargaron en L001.  

**Q:** ¿El módulo almacena datos permanentemente?  
**A:** No, solo inicializa y transforma; la persistencia depende de L001 y otras cargas.

## Referencias
- Documentación interna: `mxpi-init/process/readme.md`  
- Microservicio: `mxpv2.integration.init`  

## Contacto
- **Equipo responsable:** Integraciones MAXPOINT  
- **Canal de soporte:** #mxp-integraciones (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico  
