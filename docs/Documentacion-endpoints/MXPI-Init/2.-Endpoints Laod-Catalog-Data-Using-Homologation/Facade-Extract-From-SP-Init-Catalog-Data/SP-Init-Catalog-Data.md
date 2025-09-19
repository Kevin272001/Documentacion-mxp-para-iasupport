# Módulo de Inicialización de Catálogos (MXPi-init)

## Propósito y alcance

MXPi-init es el sistema de inicialización y carga de datos maestros dentro del ecosistema MAXPOINT V2.0. Gestiona la extracción, transformación y carga de datos catalogados desde sistemas legacy hacia la nueva plataforma. Garantiza la consistencia de datos maestros en toda la plataforma y coordina los procesos ETL (Extract, Transform, Load) para información catalogada.

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Sync
* MXP FE Build
* MXP Printer

## Descripción general del sistema

* **Microservicio:** mxpv2.integration.init
* **Rol principal:** gestionar procesos de inicialización de datos catalogados
* **Función crítica:** asegurar que los datos maestros sean correctamente migrados y homologados desde sistemas legacy

## Arquitectura y componentes

### Capa de fachada

| Componente                | Código | Descripción                                                                                                                                                                                        |
| ------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Interfaz de recepción ETL | F100   | Punto de entrada principal para procesos de inicialización de catálogos. 🔑 El F100 actúa como gateway unificado para datos catalogados, validando estructura e integridad antes del procesamiento |

### Capa de extracción

| Componente                 | Código           | Descripción                                                    |
| -------------------------- | ---------------- | -------------------------------------------------------------- |
| Extracción desde SP Legacy | E001             | Extrae datos desde stored procedures de SQL Server             |
| Extracción de UUIDs        | E002, E003, E004 | Obtiene identificadores únicos desde MongoDB para homologación |

### Capa de transformación

| Componente                   | Código | Descripción                            |
| ---------------------------- | ------ | -------------------------------------- |
| Transformación de usuarios   | T010   | Procesa y homologa datos de usuarios   |
| Transformación de impuestos  | T011   | Procesa y homologa datos de impuestos  |
| Transformación de monedas    | T012   | Procesa y homologa datos de monedas    |
| Transformación de países     | T013   | Procesa y homologa datos de países     |
| Transformación de tipos PLU  | T014   | Procesa y homologa tipos de productos  |
| Transformación de provincias | T015   | Procesa y homologa datos de provincias |
| Transformación de ciudades   | T016   | Procesa y homologa datos de ciudades   |

### Capa de carga

| Componente      | Código | Descripción                                                       |
| --------------- | ------ | ----------------------------------------------------------------- |
| Carga a MongoDB | L001   | Inserta datos transformados en las colecciones destino de MongoDB |

## Endpoints disponibles con JSON

### 1. Facade-Extract-From-SP-Init-Catalog-Users

* **Método:** POST
* **Endpoint:** `{{server_init}}/api/v1/facade`
* **Autenticación:** Bearer Token
* **Descripción:** Extrae, transforma y carga datos de usuarios desde el sistema legacy
* **Body Request:**

```json
{
    "code": "F100",
    "withResponse": true,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "chunkLoad": 100,
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
                    "entities": [ ... ]
                }
            },
            {
                "code": "E002",
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "withResponse": true,
                "withCache": false,
                "configuration": { ... }
            }
        ],
        "TRANSFORMS": [ { "code": "T010", "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6" } ],
        "LOADS": [ { "code": "L001", "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6", "configuration": { ... } } ]
    }
}
```

### 2. Facade-Extract-From-SP-Init-Catalog-Taxes

* **Método:** POST
* **Endpoint:** `{{server_init}}/api/v1/facade`
* **Autenticación:** Bearer Token
* **Descripción:** Extrae, transforma y carga datos de impuestos desde el sistema legacy
* **Body Request:** Estructura JSON similar al anterior, con transformaciones T011 y carga L001

### 3. Facade-Extract-From-SP-Init-Catalog-Currency

* **Método:** POST
* **Endpoint:** `{{server_init}}/api/v1/facade`
* **Autenticación:** Bearer Token
* **Descripción:** Extrae, transforma y carga datos de monedas desde el sistema legacy
* **Body Request:** Estructura JSON similar, transformador T012, carga L001

### 4. Facade-Extract-From-SP-Init-Catalog-Country

* **Método:** POST
* **Endpoint:** `{{server_init}}/api/v1/facade`
* **Autenticación:** Bearer Token
* **Descripción:** Extrae, transforma y carga datos de países con impuestos y monedas asociadas
* **Body Request:** JSON con transformador T013, carga L001

### 5. Facade-Extract-From-SP-Init-Catalog-Type-Plu

* **Método:** POST
* **Endpoint:** `{{server_init}}/api/v1/facade`
* **Autenticación:** Bearer Token
* **Descripción:** Extrae, transforma y carga datos de tipos de productos (PLU)
* **Body Request:** JSON con transformador T014, carga L001

### 6. Facade-Extract-From-SP-Init-Catalog-Province-For-Country

* **Método:** POST
* **Endpoint:** `{{server_init}}/api/v1/facade`
* **Autenticación:** Bearer Token
* **Descripción:** Extrae, transforma y carga datos de provincias asociadas a países
* **Body Request:** JSON con transformador T015, carga L001

### 7. Facade-Extract-From-SP-Init-Catalog-City-For-Country

* **Método:** POST
* **Endpoint:** `{{server_init}}/api/v1/facade`
* **Autenticación:** Bearer Token
* **Descripción:** Extrae, transforma y carga datos de ciudades asociadas a provincias y países
* **Body Request:** JSON con transformador T016, carga L001

## Gestión de procesos

**Flujo de trabajo estándar:**

1. Ingestión de datos → El F100 recibe la solicitud de inicialización
2. Validación → Se verifica estructura e integridad del request
3. Extracción → Los datos son extraídos desde múltiples fuentes (SQL Server SP, MongoDB)
4. Transformación → Los datos son homologados y transformados según el código T0XX correspondiente
5. Carga → Los datos procesados se insertan en las colecciones destino de MongoDB

## Puntos de integración

* Entrada: Sistemas legacy SQL Server via stored procedures
* Entrada: MongoDB para obtención de UUIDs de homologación
* Salida: MongoDB para almacenamiento de datos catalogados procesados
* Coordinación: MXP Sync para orquestación de procesos

## Patrones de procesamiento

* Extracción múltiple → Múltiples fuentes de datos en un mismo proceso
* Homologación de UUIDs → Conversión de IDs legacy a UUIDs V2
* Transformación especializada → Diferentes transformadores para cada tipo de dato
* Carga estructurada → Mapeo preciso de campos hacia colecciones destino

## Escenarios de uso

**✅ Caso exitoso:** Request válido → se recibe en F100 → validación → extracción múltiple → transformación especializada → carga exitosa

**⚠️ Posibles fallos:**

* Error en validación: Request mal formado, se rechaza con detalle del error
* Error en conexión legacy: No se puede conectar al SQL Server, se reintenta según política
* Error en transformación: Datos no cumplen con estructura esperada, se genera log de error
* Error en carga: Collection de MongoDB no disponible, se reintenta según política de reintentos

## Preguntas frecuentes (para IA Support)

* **Q:** ¿Qué pasa si falla la validación en F100?
  **A:** El request se descarta, se registra en logs y se devuelve error con detalle de la validación fallida.
* **Q:** ¿Cómo sé que la transformación se procesó correctamente?
  **A:** Revisa el log del transformador específico (T010, T011, etc.). Si hay éxito, se marca con el estado correspondiente.
* **Q:** ¿El módulo almacena datos temporalmente?
  **A:** Solo durante el procesamiento ETL. No persiste datos beyond el scope del proceso.
* **Q:** ¿Qué debo hacer si la carga falla consistentemente?
  **A:** Verifica la conexión a MongoDB, los permisos de escritura y que la collection destino exista.

## Referencias

* Fuentes internas: mxp-init/process/readme.md
* Microservicio: mxpv2.integration.init
* Esquemas de base de datos: /database/schemas/init-catalogs.md

## Contacto

* **Equipo responsable:** Inicialización MAXPOINT
* **Canal de soporte:** #mxp-inicializacion (Slack interno)
* **Escalamiento:** Arquitectura de datos → Liderazgo técnico
