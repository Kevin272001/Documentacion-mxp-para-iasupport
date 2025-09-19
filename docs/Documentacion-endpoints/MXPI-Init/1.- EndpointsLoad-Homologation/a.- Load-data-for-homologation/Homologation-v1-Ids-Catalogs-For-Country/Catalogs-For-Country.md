# Módulo de Sincronización (MXP INIT - Load-Homologation / Load-data-for-homologation)

## 1. Propósito y alcance

El módulo **MXP INIT - Load-Homologation / Load-data-for-homologation** es el sistema de orquestación de sincronización de datos de homologación de catálogos por país dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo de datos procesados entre diferentes módulos, garantiza la coherencia del sistema en toda la plataforma y coordina los procesos ETL (Extract, Transform, Load). Recibe datos procesados de las tuberías de transformación y los distribuye a las interfaces de fachada (facades).

ℹ️ Para más detalles de los módulos relacionados, consulta:
- MXP Init
- MXP FE Build
- Impresora MXP

## 2. Descripción general del sistema

- **Microservicio:** mxpv2.init.load-homologation
- **Rol principal:** Gestionar procesos de integración y sincronización de homologación de catálogos por país.
- **Función crítica:** Asegurar que los datos transformados lleguen a las fachadas correctas dentro del flujo ETL.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                  | Código | Descripción                                                        |
|-----------------------------|--------|--------------------------------------------------------------------|
| Interfaz de recepción ETL   | H000   | Punto de entrada principal para datos ETL de homologación.          |

🔑 El H000 actúa como gateway unificado para datos procesados, validando estructura e integridad.

### 3.2. Capa de transformación

| Componente                | Código | Descripción                                               |
|---------------------------|--------|-----------------------------------------------------------|
| Transformación de datos   | H001   | Procesa metadatos de homologación, asegurando estructura y disponibilidad. |

## 4. Gestión de procesos

**Flujo de trabajo estándar:**
- Ingestión de datos → El H000 recibe los paquetes ETL.
- Validación → Se verifica estructura e integridad.
- Enrutamiento → Los datos son enviados al proceso correcto según el contenido.
- Procesamiento de homologación → La info se transforma con H001.
- Distribución → Los datos procesados se entregan a las fachadas de destino.

## 5. Puntos de integración

El sincronizador conecta todo el ecosistema MAXPOINT V2.0:
- **Entrada:** Recibe datos procesados desde MXP Init, MXP FE Build y MXP Printer.
- **Salida:** Distribuye datos validados y transformados a fachadas de destino.
- **Rol central:** Mantiene coherencia en procesos ETL inter-módulos.

## 6. Patrones de sincronización

- Recepción centralizada → Todos los ETL llegan vía H000.
- Procesamiento de metadatos → El módulo H001 gestiona info de homologación.
- Distribución coordinada → Los datos son enviados a fachadas correctas.
- Garantía de coherencia → Evita inconsistencias en la plataforma.

## 7. Escenarios de uso

### ✅ Caso exitoso
ETL válido → se recibe en H000 → validación → transformación en H001 → distribución correcta.

### ⚠️ Posibles fallos
- **Error en validación:**  
  Datos rechazados. Se genera log de error con detalle del paquete fallido.  
  Acción recomendada: verificar fuente.
- **Error en transformación (H001):**  
  Datos no pasan a distribución. Se activa rollback y alerta al sistema de monitoreo.
- **Error en distribución:**  
  Fachada de destino no disponible. El sincronizador reintenta el envío según política de reintentos (retry policy).

## 8. Endpoints documentados

### POST Homologation-v1-Ids-Catalogs-User-Admin

**URL:** `{{server_init}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body (ejemplo):**
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
                        "server": "{{server_legacy}}",
                        "port": "",
                        "user": "{{user_legacy}}",
                        "password": "{{pass_legacy}}",
                        "repository": "{{repository_legacy}}",
                        "adapter": "SqlServer"
                    },
                    "entities": [
                        {
                            "name": "Users_Pos",
                            "properties": [
                                {
                                    "name": "IDUsersPos",
                                    "type": "base64"
                                }
                            ],
                            "filters": [
                                {
                                    "name": "IDPerfilPos",
                                    "operator": "equals",
                                    "value": "55059503-85cf-e511-80c6-000d3a3261f3"
                                },
                                {
                                    "name": "usr_usuario",
                                    "operator": "equals",
                                    "value": "dllerena"
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
                                {
                                    "fromName": "id_v2",
                                    "fromType": "",
                                    "toName": "id_v2",
                                    "toType": "string",
                                    "defaultValue": "",
                                    "isPrimaryKey": "true"
                                },
                                {
                                    "fromName": "country_code",
                                    "fromType": "",
                                    "toName": "country_code",
                                    "toType": "",
                                    "defaultValue": ""
                                },
                                {
                                    "fromName": "franchise_id",
                                    "fromType": "",
                                    "toName": "franchise_id",
                                    "toType": "",
                                    "defaultValue": ""
                                },
                                {
                                    "fromName": "entity_name",
                                    "fromType": "",
                                    "toName": "entity_name",
                                    "toType": "",
                                    "defaultValue": ""
                                },
                                {
                                    "fromName": "id_v1",
                                    "fromType": "",
                                    "toName": "id_v1",
                                    "toType": "",
                                    "defaultValue": ""
                                },
                                {
                                    "fromName": "",
                                    "fromType": "",
                                    "toName": "created_at",
                                    "toType": "timestamp",
                                    "defaultValue": "true"
                                }
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
**Ejemplo de request:**
```bash
curl --location '{{server_init}}/api/v1/facade' \
--header 'Authorization: Bearer {{token}}' \
--data '{ ... }'
```

### POST Homologation-v1-Ids-Catalogs-Country

**URL:** `{{server_init}}/api/v1/facade`  
**Body (ejemplo):**
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
                        "server": "{{server_legacy}}",
                        "port": "",
                        "user": "{{user_legacy}}",
                        "password": "{{pass_legacy}}",
                        "repository": "{{repository_legacy}}",
                        "adapter": "SqlServer"
                    },
                    "entities": [
                        {
                            "name": "Pais",
                            "properties": [
                                {
                                    "name": "pais_id",
                                    "type": "base64"
                                }
                            ],
                            "filters": [
                                {
                                    "name": "pais_id",
                                    "operator": "equals",
                                    "value": "{{legacy_country_id}}"
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
                "code": "01",
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
                                {
                                    "fromName": "id_v2",
                                    "fromType": "",
                                    "toName": "id_v2",
                                    "toType": "string",
                                    "defaultValue": "",
                                    "isPrimaryKey": "true"
                                },
                                {
                                    "fromName": "country_code",
                                    "fromType": "",
                                    "toName": "country_code",
                                    "toType": "",
                                    "defaultValue": ""
                                },
                                {
                                    "fromName": "franchise_id",
                                    "fromType": "",
                                    "toName": "franchise_id",
                                    "toType": "",
                                    "defaultValue": ""
                                },
                                {
                                    "fromName": "entity_name",
                                    "fromType": "",
                                    "toName": "entity_name",
                                    "toType": "",
                                    "defaultValue": ""
                                },
                                {
                                    "fromName": "id_v1",
                                    "fromType": "",
                                    "toName": "id_v1",
                                    "toType": "",
                                    "defaultValue": ""
                                },
                                {
                                    "fromName": "",
                                    "fromType": "",
                                    "toName": "created_at",
                                    "toType": "timestamp",
                                    "defaultValue": "true"
                                }
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
**Ejemplo de request:**
```bash
curl --location -g '{{server_init}}/api/v1/facade' \
--data '{ ... }'
```

### POST Homologation-v1-Ids-Catalogs-Currency

**URL:** `{{server_init}}/api/v1/facade`  
**Body (ejemplo):**
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": true,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": 0,
    "processes": { ... }
}
```
**Ejemplo de request:**
```bash
curl --location -g '{{server_init}}/api/v1/facade' \
--data '{ ... }'
```

### POST Homologation-v1-Ids-Catalogs-Taxes

**URL:** `{{server_init}}/api/v1/facade`  
**Body (ejemplo):**
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": 0,
    "processes": { ... }
}
```
**Ejemplo de request:**
```bash
curl --location -g '{{server_init}}/api/v1/facade' \
--data '{ ... }'
```

### POST Homologation-v1-Ids-Catalogs-Province

**URL:** `{{server_init}}/api/v1/facade`  
**Body (ejemplo):**
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": 0,
    "processes": { ... }
}
```
**Ejemplo de request:**
```bash
curl --location -g '{{server_init}}/api/v1/facade' \
--data '{ ... }'
```

### POST Homologation-v1-Ids-Catalogs-City

**URL:** `{{server_init}}/api/v1/facade`  
**Body (ejemplo):**
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": true,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": 0,
    "processes": { ... }
}
```
**Ejemplo de request:**
```bash
curl --location -g '{{server_init}}/api/v1/facade' \
--data '{ ... }'
```

## 9. Preguntas frecuentes (para IA Support)

- **¿Qué pasa si falla la validación en H000?**  
  El paquete se descarta, se registra en logs y se debe revisar la fuente.

- **¿Cómo sé que la info de homologación se procesó bien?**  
  Revisa el log de H001. Si hay éxito, se marca como homologation_metadata_ok.

- **¿El sincronizador almacena datos?**  
  No, solo orquesta flujos. La persistencia depende de otros módulos.

- **¿Qué debo hacer si la distribución falla?**  
  Se activa la política de reintentos, pero si el error persiste, revisa la fachada de destino y los logs de distribución.

## 10. Referencias

- mxp-sync/process/readme.md 
- mxp-feBuild/process/readme.md 
- Microservicio: mxpv2.init.load-homologation

## 11. Contacto

- **Equipo responsable:** Integraciones MAXPOINT
- **Canal de soporte:** #mxp-integraciones (Slack interno)
- **Escalamiento:** Arquitectura de