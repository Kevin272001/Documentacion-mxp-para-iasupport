# Módulo de Homologación de IDs (MXP-Homologation)

## Propósito y alcance
MXP Homologation es el sistema de gestión de homologación de identificadores dentro del ecosistema MAXPOINT V2.0. Gestiona el proceso de conversión de IDs legacy (v1) a IDs unificados (v2) para todas las entidades del sistema. Garantiza la consistencia en la identificación de recursos a través de diferentes módulos y sistemas.

## Descripción general del sistema
**Microservicio**: mxpv2.integration.homologation  
**Rol principal**: Convertir identificadores legacy a formato UUID v2 unificado  
**Función crítica**: Mantener el mapeo consistente entre sistemas legacy y nueva plataforma

## Arquitectura y componentes

### Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de homologación | H000 | Punto de entrada principal para procesos de homologación de IDs |
| Transformación de homologación | H001 | Procesa y convierte IDs legacy a formato UUID v2 |

### Capa de extracción
| Componente | Código | Descripción |
|------------|--------|-------------|
| Extractor SQL Server | E001 | Extrae datos de bases de datos legacy con diferentes entidades |

## Endpoints de Homologación

### 1. Homologation-v1-Ids-For-Franchise

**Método**: POST  
**URL**: `{{server_init}}/api/v1/facade`  
**Autenticación**: Bearer Token

#### Body Request (JSON)
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": "{{legacy_cdn_id}}",
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
                            "name": "Cadena",
                            "properties": [
                                {
                                    "name": "cdn_id",
                                    "type": "base64"
                                }
                            ],
                            "filters": [
                                {
                                    "name": "cdn_id",
                                    "operator": "equals",
                                    "value": "{{legacy_cdn_id}}"
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

#### Descripción
Homologa los IDs de franquicias (Cadena) del sistema legacy al formato UUID v2. Extrae el ID de la cadena, lo transforma a UUID y lo almacena en MongoDB para su posterior uso.

---

### 2. Homologation-v1-Ids-For-Franchise-Configurations

**Método**: POST  
**URL**: `{{server_init}}/api/v1/facade`  
**Autenticación**: Bearer Token

#### Body Request (JSON)
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": "{{legacy_cdn_id}}",
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
                            "name": "mxp_v2.View_Franchise_Configurations",
                            "properties": [
                                {
                                    "name": "id",
                                    "type": "base64"
                                }
                            ],
                            "filters": [
                                {
                                    "name": "cdn_id",
                                    "operator": "equals",
                                    "value": "{{legacy_cdn_id}}"
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

#### Descripción
Homologa los IDs de configuraciones de franquicias desde la vista `mxp_v2.View_Franchise_Configurations`. Utiliza el mismo proceso de transformación pero con una entidad diferente para configuraciones específicas.

---

### 3. Homologation-v1-Ids-For-Franchise-Restaurant

**Método**: POST  
**URL**: `{{server_init}}/api/v1/facade`  
**Autenticación**: Bearer Token

#### Body Request (JSON)
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": "{{legacy_cdn_id}}",
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
                            "name": "Restaurante",
                            "properties": [
                                {
                                    "name": "rst_id",
                                    "type": "base64"
                                }
                            ],
                            "filters": [
                                {
                                    "name": "IDStatus",
                                    "operator": "equals",
                                    "value": "71039503-85cf-e511-80c6-000d3a3261f3"
                                },
                                {
                                    "name": "cdn_id",
                                    "operator": "equals",
                                    "value": "{{legacy_cdn_id}}"
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

#### Descripción
Homologa los IDs de restaurantes activos (filtrados por IDStatus específico) del sistema legacy. Solo procesa restaurantes con estado activo y pertenecientes a la franquicia especificada.

---

### 4. Homologation-v1-Ids-For-Franchise-Category-Price

**Método**: POST  
**URL**: `{{server_init}}/api/v1/facade`  
**Autenticación**: Bearer Token

#### Body Request (JSON)
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": "{{legacy_cdn_id}}",
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
                            "name": "Categoria",
                            "properties": [
                                {
                                    "name": "IDCategoria",
                                    "type": "base64"
                                }
                            ],
                            "filters": [
                                {
                                    "name": "IDStatus",
                                    "operator": "equals",
                                    "value": "71039503-85cf-e511-80c6-000d3a3261f3"
                                },
                                {
                                    "name": "cdn_id",
                                    "operator": "equals",
                                    "value": "{{legacy_cdn_id}}"
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

#### Descripción
Homologa los IDs de categorías de precios activas del sistema legacy. Filtra por estado activo y franquicia específica, transformando los IDs de categoría a formato UUID v2.

---

### 5. Homologation-v1-Ids-For-Franchise-Departments

**Método**: POST  
**URL**: `{{server_init}}/api/v1/facade`  
**Autenticación**: Bearer Token

#### Body Request (JSON)
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": "{{legacy_cdn_id}}",
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
                            "name": "mxp_v2.Coleccion_Departamento_Plu",
                            "properties": [
                                {
                                    "name": "legacy_collection_data_id",
                                    "type": "base64"
                                }
                            ],
                            "filters": [
                                {
                                    "name": "legacy_cdn_id",
                                    "operator": "equals",
                                    "value": "{{legacy_cdn_id}}"
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

#### Descripción
Homologa los IDs de departamentos desde la vista especializada `mxp_v2.Coleccion_Departamento_Plu`. Este endpoint se enfoca en la estructura de departamentos específica para colecciones PLU.

---

### 6. Homologation-v1-Ids-For-Franchise-Menu-Category

**Método**: POST  
**URL**: `{{server_init}}/api/v1/facade`  
**Autenticación**: Bearer Token

#### Body Request (JSON)
```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": "{{legacy_cdn_id}}",
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
                            "name": "Menu_Agrupacion",
                            "properties": [
                                {
                                    "name": "IDMenuAgrupacion",
                                    "type": "base64"
                                }
                            ],
                            "filters": [
                                {
                                    "name": "IDStatus",
                                    "operator": "equals",
                                    "value": "78039503-85cf-e511-80c6-000d3a3261f3"
                                },
                                {
                                    "name": "cdn_id",
                                    "operator": "equals",
                                    "value": "{{legacy_cdn_id}}"
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

#### Descripción
Homologa los IDs de categorías de menú (Menu_Agrupacion) activas del sistema legacy. Utiliza un filtro de estado específico (78039503-85cf-e511-80c6-000d3a3261f3) para categorías de menú activas.

---

## Gestión de procesos

### Flujo de trabajo estándar
1. **Recepción**: El endpoint H000 recibe la solicitud de homologación
2. **Extracción**: El componente E001 extrae datos de la base legacy según la entidad especificada
3. **Transformación**: El componente H001 convierte los IDs legacy a formato UUID v2
4. **Carga**: Los datos transformados se almacenan en MongoDB en la colección "uuids"
5. **Respuesta**: Se retorna el resultado del proceso (dependiendo de withResponse)

### Estructura de datos de salida
Todos los endpoints almacenan los datos homologados en la misma estructura en MongoDB:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id_v2 | string (PK) | UUID v2 generado |
| country_code | string | Código de país |
| franchise_id | string | ID de la franquicia |
| entity_name | string | Nombre de la entidad homologada |
| id_v1 | string | ID legacy original |
| created_at | timestamp | Fecha de creación del registro |

## Puntos de integración
El módulo de homologación conecta con:

- **Sistemas Legacy**: SQL Server con datos originales
- **Almacenamiento**: MongoDB para guardar mapeos de IDs
- **Sistema de Autenticación**: Token Bearer para seguridad

## Patrones de homologación
- **Extracción unificada**: Todos usan el mismo extractor E001 con diferentes configuraciones
- **Transformación consistente**: Mismo proceso H001 para todas las entidades
- **Almacenamiento estructurado**: Misma estructura en MongoDB para todos los tipos de entidades

## Escenarios de uso
✅ **Caso exitoso**: 
Datos legacy válidos → extracción exitosa → transformación a UUID → almacenamiento en MongoDB

⚠️ **Posibles fallos**:
- Error de conexión con base legacy
- Datos legacy corruptos o en formato incorrecto
- Error de autenticación con MongoDB
- Timeout en el proceso de transformación

## Preguntas frecuentes (para IA Support)

**Q: ¿Qué diferencia hay entre los diferentes endpoints de homologación?**
A: Cada endpoint se especializa en un tipo de entidad diferente (franquicias, restaurantes, categorías, etc.) pero comparten el mismo proceso de transformación.

**Q: ¿Cómo verificar que la homologación fue exitosa?**
A: Revisar la colección "uuids" en MongoDB y buscar registros con el franchise_id y entity_name correspondientes.

**Q: ¿Los procesos de homologación son síncronos o asíncronos?**
A: Depende del parámetro "withResponse". Si es false, el proceso es asíncrono.

**Q: ¿Qué hacer si falla la homologación?**
A: Verificar logs del componente E001 (extracción) y H001 (transformación), y validar la conectividad con las bases de datos.

## Referencias
- Documentación interna: mxp-init/process/homologation.md
- Esquema de base de datos: legacy-sql/schema.md
- Estructura MongoDB: mxp-storage/uuids-collection.md

## Contacto
**Equipo responsable**: Homologación MAXPOINT  
**Canal de soporte**: #mxp-homologation (Slack interno)  
**Escalamiento**: Arquitectura de datos → Liderazgo técnico