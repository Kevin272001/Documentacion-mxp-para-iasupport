# Módulo de Sincronización (MXP-Sync)

## 1. Propósito y alcance

MXP Sincronizador es el sistema de orquestación de sincronización de datos dentro del ecosistema MAXPOINT V2.0.

* Gestiona el flujo de datos procesados entre diferentes módulos.
* Garantiza la coherencia del sistema en toda la plataforma.
* Coordina los procesos ETL (Extract, Transform, Load).
* Recibe datos procesados de las tuberías de transformación y los distribuye a las interfaces de fachada (facades).

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Init
* MXP FE Build
* Impresora MXP

---

## 2. Descripción general del sistema

* **Microservicio:** mxpv2.integration.sync
* **Rol principal:** gestionar procesos de integración y sincronización.
* **Función crítica:** asegurar que los datos transformados lleguen a las fachadas correctas dentro del flujo ETL.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código | Descripción                                                           |
| ------------------------- | ------ | --------------------------------------------------------------------- |
| Interfaz de recepción ETL | F008   | Punto de entrada principal para datos ETL (extract, transform, load). |

🔑 El F008 actúa como gateway unificado para datos procesados, validando estructura e integridad.

### 3.2. Capa de transformación

| Componente                        | Código | Descripción                                                               |
| --------------------------------- | ------ | ------------------------------------------------------------------------- |
| Construir información Transformar | T080   | Procesa metadatos de compilación, asegurando estructura y disponibilidad. |

---

## 4. Gestión de procesos

**Flujo de trabajo estándar:**

1. **Ingestión de datos:** El F008 recibe los paquetes ETL.
2. **Validación:** Se verifica estructura e integridad.
3. **Enrutamiento:** Los datos son enviados al proceso correcto según el contenido.
4. **Procesamiento de compilación:** La info de construcción se transforma con T080.
5. **Distribución:** Los datos procesados se entregan a las fachadas de destino.

---

## 5. Puntos de integración

* **Entrada:** recibe datos procesados desde MXP Init, MXP FE Build y MXP Printer.
* **Salida:** distribuye datos validados y transformados a fachadas de destino.
* **Rol central:** mantiene coherencia en procesos ETL inter-módulos.

---

## 6. Patrones de sincronización

* **Recepción centralizada:** Todos los ETL llegan vía F008.
* **Procesamiento de metadatos:** El módulo T080 gestiona info de compilación.
* **Distribución coordinada:** Los datos son enviados a fachadas correctas.
* **Garantía de coherencia:** Evita inconsistencias en la plataforma.

---

## 7. Escenarios de uso

### ✅ Caso exitoso

ETL válido → se recibe en F008 → validación → transformación en T080 → distribución correcta.

### ⚠️ Posibles fallos

**Error en validación:**

* Datos rechazados.
* Se genera log de error con detalle del paquete fallido.
* **Acción recomendada:** verificar fuente.

**Error en transformación (T080):**

* Datos no pasan a distribución.
* Se activa rollback y alerta al sistema de monitoreo.

**Error en distribución:**

* Fachada de destino no disponible.
* El sincronizador reintenta el envío según política de reintentos (retry policy).

---

## 8. Endpoint: Sync-Printing-Settings (QA)

* **Entorno:** QA
* **Método:** POST
* **URL:** `{{server_sync}}/api/v1/facade`
* **Autenticación:** Bearer Token (`{{token}}`)

### Request Body (raw JSON)

```json
{
  "code": "F008",
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "chunkLoad": 100,
  "processes": {
    "EXTRACTS": [
      {
        "code": "E001",
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "withResponse": true,
        "withCache": false,
        "configuration": {
          "connection": {
            "server": "{{server_mongo_qa}}",
            "port": "",
            "user": "{{user_mongo_qa}}",
            "password": "{{pass_mongo_qa}}",
            "repository": "QA_KFC_MXPi_Settings",
            "adapter": "Mongo"
          },
          "entities": [
            {
              "name": "Settings_BO_Printing_Settings",
              "properties": [],
              "filters": [
                {"name": "country_id", "operator": "equals", "datatype": "uuid", "value": "d6cac83d-6ce5-f62e-2628-1e2e9ae7b51f"},
                {"name": "cdn_id", "operator": "equals", "datatype": "uuid", "value": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af"},
                {"name": "restaurant_id", "operator": "equals", "datatype": "uuid", "value": "24260579-faf8-763f-aac7-5e16faff96d1"}
              ]
            }
          ]
        }
      }
    ],
    "TRANSFORMS": [
      {"code": "T080", "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6", "withResponse": true, "withCache": true, "country": "ECU"}
    ],
    "LOADS": [
      {
        "code": "L001",
        "withCache": false,
        "withResponse": true,
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "configuration": {
          "connection": {
            "server": "192.168.101.13",
            "port": "27020",
            "user": "admin",
            "password": "admin123",
            "repository": "MXPi_Settings",
            "adapter": "MongoLocal"
          },
          "entities": [
            {
              "name": "Settings_BO_Printing_Settings",
              "toName": "Settings_BO_Printing_Settings",
              "properties": [
                {"fromName": "_id", "toName": "_id", "toType": "uuid", "isPrimaryKey": "true"},
                {"fromName": "country_id", "toName": "country_id", "toType": "uuid"},
                {"fromName": "cdn_id", "toName": "cdn_id", "toType": "uuid"},
                {"fromName": "restaurant_id", "toName": "restaurant_id", "toType": "uuid"},
                {"fromName": "cash_registers", "toName": "cash_registers", "toType": "array"}
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

### Example Request (curl)

```bash
curl --location '{{server_sync}}/api/v1/facade' \
--header 'Authorization: Bearer {{token}}' \
--data-raw '<JSON body del request anterior>'
```

### Example Response

```json
{
  "message": "Error processing request: 'name'"
}
```

---

## 9. Preguntas frecuentes (para IA Support)

* **Q:** ¿Qué pasa si falla la validación en F008?
  **A:** El paquete se descarta, se registra en logs y se debe revisar la fuente.

* **Q:** ¿Cómo sé que la info de compilación se procesó bien?
  **A:** Revisa el log de T080. Si hay éxito, se marca como `build_metadata_ok`.

* **Q:** ¿El sincronizador almacena datos?
  **A:** No, solo orquesta flujos. La persistencia depende de otros módulos.

* **Q:** ¿Qué debo hacer si la distribución falla?
  **A:** Se activa la política de reintentos, pero si el error persiste, revisa la fachada de destino y los logs de distribución.

---

## 10. Referencias

* **Fuentes internas:**

  * `mxp-sync/process/readme.md (1-28)`
  * `mxp-feBuild/process/readme.md (9-10)`
* **Microservicio:** `mxpv2.integration.sync`

---

## 11. Contacto

* **Equipo responsable:** Integraciones MAXPOINT
* **Canal de soporte:** `#mxp-integraciones` (Slack interno)
* **Escalamiento:** Arquitectura de datos → Liderazgo técnico
