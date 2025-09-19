# Módulo MXPI-INIT - CONFIGURATION PAYMENTS-FOR-CARD

## 1. Propósito y alcance

MXPI-INIT (CONFIGURATION PAYMENTS-FOR-CARD) es el microservicio encargado de orquestar la carga, transformación y publicación de la configuración relacionada con los pagos con tarjeta dentro del ecosistema MAXPOINT V2.0.

* Gestiona la extracción de configuraciones desde repositorios (MongoDB / servicios legacy).
* Transforma y homologa identificadores entre versiones (V1 ↔ V2).
* Publica las configuraciones resultantes a las fachadas (facades) que consumen los POS y servicios relacionados.

ℹ️ Mantiene coherencia con el flujo ETL general de MXP Init y se integra con MXP Sync y otros microservicios.

## 2. Descripción general del sistema

**Microservicio:** mxpv2.integration.mxpi-init

**Rol principal:** Preparar y publicar la configuración de pagos con tarjeta (CONFIGURACION\_PAGOS\_CON\_TARJETA) hacia las fachadas que lo requieran.

**Función crítica:** Asegurar que la configuración publicada esté correctamente filtrada por país, CDN, restaurante y caja, y que las referencias entre versiones estén homogeneizadas.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código | Descripción                                                                                                                                                                                            |
| ------------------------- | -----: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Interfaz de recepción ETL |   F101 | Punto de entrada principal para la ejecución del flujo de configuración de pagos con tarjeta. Actúa como gateway unificado para solicitudes de configuración y coordina Extracts → Transforms → Loads. |

### 3.2. Capa de extracción (EXTRACTS)

Cada extract corresponde a una fuente de datos y trae entidades filtradas para el proceso.

* **E001**

  * Fuente: Mongo (DEV\_KFC\_MXPi\_Settings)
  * Entidad: `Settings_BO_Service_Integration_Settings`
  * Filtro principal: `service_integration_config.key == CONFIGURACION_PAGOS_CON_TARJETA`
  * Propósito: Obtener la configuración específica del servicio de integración que define parámetros de pagos con tarjeta.

* **E002**

  * Fuente: Mongo (DEV\_KFC\_MXPi\_Settings)
  * Entidad: `Settings_BO_MXPLegacy_Server_Settings`
  * Filtros: `country_id`, `cdn_id`, `restaurant_id`, `cash_registers.cash_register_ip`
  * Propósito: Obtener los detalles de servidores legacy y puntos de integración por país, CDN y restaurante.

* **E003**

  * Fuente: Mongo (homologation\_v1\_v2)
  * Entidad: `uuids`
  * Propósito: Resolver mapeos de IDs entre v1 y v2 para `mxp_v2.View_Cash_Register` y otras entidades relacionadas.

> Nota: Existe un ejemplo comentado (E004) para llamados RestNoSSL si se necesita integrar una fuente REST externa en el futuro.

### 3.3. Capa de transformación (TRANSFORMS)

* **T123**

  * Código: T123
  * Función: Transformación de datos de configuración, enriquecimiento con campos homologados y preparación del payload final para publicación.
  * Parámetros: `country`, `withData=true` (indica que debe incluir datos en la respuesta)

### 3.4. Capa de carga/ publicación (LOADS / PUBLISHERS)

* En esta definición no se incluyen LOADS activas (están comentadas como ejemplo). Los LOADS típicos pueden publicar a colecciones Mongo `MXP_integrations` o a endpoints REST según la necesidad.

## 4. Gestión de procesos

### Flujo de trabajo estándar

1. **Invocación** → Se recibe una llamada POST a `F101` con parámetros de sync y configuración.
2. **Ingestión (EXTRACTS)** → E001, E002, E003 ejecutan consultas con sus filtros.
3. **Validación** → Se verifica estructura de datos y presencia de mapeos de UUIDs necesarios.
4. **Transformación (T123)** → Homologación y armado del payload final.
5. **Publicación (LOADS / PUBLISHERS)** → Distribución a colecciones o endpoints destino.

### Políticas operativas

* **chunkLoad** controla el tamaño de procesamiento por lote (ej. 100).
* **withCache** habilitado en extracts/transforms para acelerar re-procesos.
* **withResponse** indica que el proceso debe devolver un resultado/estado de ejecución.

## 5. Puntos de integración

* **Entrada:** Peticiones F101 desde orquestadores o procesos internos.
* **Fuentes de datos:** MongoDB (DEV\_KFC\_MXPi\_Settings, homologation\_v1\_v2), potenciales endpoints REST.
* **Salida:** Publicación en colecciones Mongo o llamadas a fachadas que consumen la configuración.

## 6. Patrones de sincronización

* **Recepción centralizada:** Todas las solicitudes de configuración pasan por F101.
* **Procesamiento por lotes:** chunkLoad para particionar carga y evitar sobrecarga.
* **Caching:** withCache habilitado donde aplica para evitar reconsultas costosas.
* **Homologación:** Uso de la colección `uuids` para mapear identidades entre versiones.

## 7. Escenarios de uso

### ✅ Caso exitoso

* Solicitud POST a F101 con `syncId` válido → E001/E002/E003 extraen datos → T123 transforma → Publicación en destino → Respuesta con `status: success`.

### ⚠️ Posibles fallos

* **Fallo en extract (p. ej. credenciales o conexión Mongo):** Logs con detalle del error; acción: revisar credenciales y accesos de red.
* **Fallo en transformación (T123):** Datos incompletos o falta de mapeo UUID → rollback lógico y alerta a monitoreo.
* **Fallo en publicación:** Reintentos según política; si persiste, escalamiento al equipo de Integraciones.

## 8. Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si falla la conexión a DEV\_KFC\_MXPi\_Settings?**
A: El proceso registra el error y devuelve un `withResponse` indicando fallo. Revisar cadena de conexión, usuario y password, y reglas de firewall.

**Q: ¿Cómo comprobar que la homologación se realizó correctamente?**
A: Revisar la colección `homologation_v1_v2.uuids` y el log generado por T123; buscar `build_metadata_ok` o entradas de mapeo exitosas.

**Q: ¿El microservicio almacena datos sensibles (contraseñas)?**
A: El código de ejemplo contiene cadenas de conexión. En producción **no** deben dejarse credenciales en texto plano; usar vault/secret manager.

**Q: ¿Cómo forzar reintentos?**
A: La política de reintentos está definida en la capa de publicación; si es necesario, re-ejecutar la petición F101 con el mismo `syncId`.

## 9. Referencias

* Documentos internos: `mxp-init/process/readme.md` (recomendado)
* Colecciones Mongo: `DEV_KFC_MXPi_Settings`, `homologation_v1_v2`

## 10. Contacto

* Equipo responsable: Integraciones MAXPOINT
* Canal de soporte: `#mxp-integraciones` (Slack interno)
* Escalamiento: Arquitectura de datos → Liderazgo técnico

---

## Anexo: Definición del endpoint (ejemplo de petición)

**POST** `{{server_init}}/api/v1/facade`

```json
{
    "code": "F101",
    "withResponse": true,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "chunkLoad": 100,
    "processes": {
        "EXTRACTS": [
            {
                "code": "E001",
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "withResponse": true,
                "withCache": true,
                "configuration": {
                    "connection": {
                        "server": "ec-mxpv2-dev.3hro7.mongodb.net",
                        "port": "",
                        "user": "dev_integration",
                        "password": "rIJEUAk8WyynPhC4",
                        "repository": "DEV_KFC_MXPi_Settings",
                        "adapter": "Mongo"
                    },
                    "entities": [
                        {
                            "name": "Settings_BO_Service_Integration_Settings",
                            "properties": [],
                            "filters": [
                                {
                                    "name": "service_integration_config.key",
                                    "operator": "equals",
                                    "value": "CONFIGURACION_PAGOS_CON_TARJETA"
                                }
                            ]
                        }
                    ]
                }
            },
            {
                "code": "E002",
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "withResponse": true,
                "withCache": true,
                "configuration": {
                    "connection": {
                        "server": "ec-mxpv2-dev.3hro7.mongodb.net",
                        "port": "",
                        "user": "dev_integration",
                        "password": "rIJEUAk8WyynPhC4",
                        "repository": "DEV_KFC_MXPi_Settings",
                        "adapter": "Mongo"
                    },
                    "entities": [
                        {
                            "name": "Settings_BO_MXPLegacy_Server_Settings",
                            "properties": [],
                            "filters": [
                                {
                                    "name": "country_id",
                                    "operator": "equals",
                                    "datatype": "uuid",
                                    "value": "d6cac83d-6ce5-f62e-2628-1e2e9ae7b51f"
                                },
                                {
                                    "name": "cdn_id",
                                    "operator": "equals",
                                    "datatype": "uuid",
                                    "value": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af"
                                },
                                {
                                    "name": "restaurant_id",
                                    "operator": "equals",
                                    "datatype": "uuid",
                                    "value": "e17d03da-b8b6-f424-febc-3a767b6401bb"
                                },
                                {
                                    "name": "cash_registers.cash_register_ip",
                                    "operator": "equals",
                                    "value": "10.104.3.21"
                                }
                            ]
                        }
                    ]
                }
            },
            {
                "code": "E003",
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "withResponse": true,
                "withCache": true,
                "configuration": {
                    "connection": {
                        "server": "{{server_mongo}}",
                        "port": "",
                        "user": "{{user_mongo}}",
                        "password": "{{pass_mongo}}",
                        "repository": "homologation_v1_v2",
                        "adapter": "Mongo"
                    },
                    "entities": [
                        {
                            "name": "uuids",
                            "properties": [
                                {
                                    "name": "id_v2",
                                    "type": "base64"
                                },
                                {
                                    "name": "country_code",
                                    "type": "base64"
                                },
                                {
                                    "name": "entity_name",
                                    "type": "base64"
                                },
                                {
                                    "name": "id_v1",
                                    "type": "base64"
                                },
                                {
                                    "name": "franchise_id",
                                    "type": "base64"
                                }
                            ],
                            "filters": [
                                {
                                    "name": "country_code",
                                    "operator": "equals",
                                    "value": "{{country}}"
                                },
                                {
                                    "name": "entity_name",
                                    "operator": "equals",
                                    "value": "mxp_v2.View_Cash_Register"
                                }
                            ]
                        }
                    ]
                }
            }
        ],
        "TRANSFORMS": [
            {
                "code": "T123",
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "withResponse": true,
                "withCache": true,
                "country": "{{country}}",
                "withData": "true"
            }
        ],
        "LOADS": [],
        "PUBLISHERS": []
    }
}
```

---

*Generado automáticamente con la plantilla solicitada. Si deseas que exporte este MD como archivo descargable (.md) o que incluya más secciones (ej. ejemplos de respuestas, esquema de logs, diagramas de arquitectura), indícalo y lo agrego.*
