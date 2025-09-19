# Módulo de Publicación SIR (MXPI-PUB SIR QA)

## 1. Propósito y alcance

El módulo **MXPI-PUB SIR DEV** es el sistema de orquestación y publicación de datos procesados hacia el sistema SIR dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo de datos entre módulos, garantiza la coherencia y coordina los procesos ETL (Extract, Transform, Load). Recibe datos procesados de las tuberías de transformación y los distribuye a la interfaz SIR.

ℹ️ Para más detalles de los módulos relacionados, consulta:
- MXP Init
- MXP FE Build
- Impresora MXP

## 2. Descripción general del sistema

- **Microservicio:** mxpv2.integration.pub.sir
- **Rol principal:** Gestionar procesos de integración y publicación hacia SIR.
- **Función crítica:** Asegurar que los datos transformados lleguen correctamente al sistema SIR dentro del flujo ETL.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente           | Código | Descripción                                              |
|----------------------|--------|----------------------------------------------------------|
| SendInterfaceSIR     | F100   | Publica datos procesados hacia la interfaz SIR.          |

🔑 El endpoint actúa como gateway unificado para datos procesados, validando estructura e integridad.

### 3.2. Capa de transformación

| Componente | Código | Descripción                                      |
|------------|--------|--------------------------------------------------|
| Transformaciones varias | T083, T084, T085, T086, T087, T101 | Procesan y preparan los datos para la carga en SIR. |

## 4. Gestión de procesos

**Flujo de trabajo estándar:**
- Ingestión de datos → Recepción de paquetes ETL.
- Validación → Verificación de estructura e integridad.
- Enrutamiento → Envío al proceso correcto según el contenido.
- Transformación → Procesamiento según el tipo de publicación.
- Distribución → Entrega a la interfaz SIR.

## 5. Puntos de integración

El publicador conecta el ecosistema MAXPOINT V2.0 con SIR:
- **Entrada:** Recibe datos procesados desde MongoDB.
- **Salida:** Publica datos validados y transformados en el endpoint REST de SIR.
- **Rol central:** Mantiene coherencia en procesos ETL inter-módulos.

## 6. Patrones de sincronización

- Recepción centralizada → Todos los ETL llegan vía el endpoint de fachada.
- Procesamiento de metadatos → Cada transformación gestiona la información específica del proceso.
- Distribución coordinada → Los datos se envían a SIR según el tipo de publicación.
- Garantía de coherencia → Evita inconsistencias en la plataforma.

## 7. Escenarios de uso

### ✅ Caso exitoso
ETL válido → Recepción → Validación → Transformación → Distribución correcta a SIR.

### ⚠️ Posibles fallos
- **Error en validación:** Datos rechazados, log de error, revisar fuente.
- **Error en transformación:** No se distribuyen, rollback y alerta.
- **Error en distribución:** SIR no disponible, reintentos automáticos.

## 8. Endpoint documentado

### POST SendInterfaceSIR

**URL:** `{{server_pub_sir}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body (ejemplo):**
```json
{
    "code": "F100",
    "withResponse": true,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "ECU",
    "processes": {
        "EXTRACTS": [
            {
                "code": "E001",
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "withResponse": true,
                "withCache": false,
                "configuration": {
                    "connection": {
                        "server": "{{server_mongo}}",
                        "port": "",
                        "user": "{{user_mongo}}",
                        "password": "{{pass_mongo}}",
                        "repository": "{{repository_mongo_hml}}",
                        "adapter": "Mongo"
                    },
                    "entities": [
                        {
                            "name": "uuids",
                            "properties": [],
                            "filters": [
                                {
                                    "name": "id_v2",
                                    "operator": "equals",
                                    "value": "18ab3836-3202-f6d4-1eef-a5b5de5a8e40"
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
                "withCache": false,
                "configuration": {
                    "connection": {
                        "server": "{{server_mongo}}",
                        "port": "",
                        "user": "{{user_mongo}}",
                        "password": "{{pass_mongo}}",
                        "repository": "QA_KFC_MXPi_PubSale",
                        "adapter": "Mongo"
                    },
                    "entities": [
                        {
                            "name": "Pub_Sale_Cash_Closure",
                            "properties": [],
                            "filters": [
                                {
                                    "name": "period_id",
                                    "operator": "equals",
                                    "datatype": "uuid",
                                    "value": "8bda8c52-e2fb-44ff-9a2c-97c630f3f14c"
                                },
                                {
                                    "name": "restaurant_id",
                                    "operator": "equals",
                                    "datatype": "uuid",
                                    "value": "18ab3836-3202-f6d4-1eef-a5b5de5a8e40"
                                },
                                {
                                    "name": "start_date",
                                    "operator": "equals",
                                    "value": "29/08/2025"
                                }
                            ]
                        }
                    ]
                }
            },
            // ...otros extractos E003-E009...
        ],
        "TRANSFORMS": [
            {
                "code": "T083",
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "withResponse": true,
                "withCache": false,
                "withData": "true"
            },
            // ...otros transformadores T084-T101...
        ],
        "LOADS": [
            {
                "code": "L001",
                "withCache": false,
                "withResponse": true,
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "configuration": {
                    "connection": {
                        "server": "https://sirqa.grupokfc.com",
                        "port": "",
                        "user": "",
                        "password": "",
                        "repository": "/serviciosweb/interface/servicio.php",
                        "adapter": "Rest"
                    },
                    "entities": []
                },
                "dataInput": {}
            }
        ]
    }
}
```

**Ejemplo de request:**
```bash
curl --location -g '{{server_pub_sir}}/api/v1/facade' \
--data '{ ... }'
```

**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "status": "success",
  "process": "I001",
  "message": "Integración completada exitosamente",
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "data": { ... }
}
```

## 9. Preguntas frecuentes (para IA Support)

- **¿Qué pasa si falla la validación?**  
  El paquete se descarta y se registra en logs. Revisar la fuente de datos.

- **¿Cómo sé que la transformación fue exitosa?**  
  Revisar el log del proceso de transformación (ej. T083, T084, etc.).

- **¿El publicador almacena datos?**  
  No, solo orquesta flujos. La persistencia depende de los módulos de carga.

- **¿Qué hacer si la carga falla?**  
  Se activa la política de reintentos. Si persiste, revisar la interfaz SIR y los logs.

## 10. Referencias

- mxp-sync/process/readme.md (1-28)
- mxp-feBuild/process/readme.md (9-10)
- Microservicio: mxpv2.integration.pub.sir

## 11. Contacto

- **Equipo responsable:** Integraciones MAXPOINT
- **Canal de soporte:** #mxp-integraciones (Slack interno)
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico