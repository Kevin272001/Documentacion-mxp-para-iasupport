# Módulo de Sincronización (MXPI-Sync)

## 1. Propósito y alcance

El módulo **MXPI-Sync** gestiona los procesos de sincronización de información crítica dentro del ecosistema **MAXPOINT V2.0**.

El endpoint **Sync-Billing-Information** se encarga de extraer, transformar y cargar información de facturación y secuencias de numeración hacia los repositorios locales.

**Alcance:** sincronización programada o bajo demanda de entidades relacionadas con facturación (billing_information, number_sequence) desde fuentes externas (MongoDB, SQL Server) hacia repositorios locales (Mongo local / KFC_MXPi_FeBuild).

---

## 2. Descripción general del sistema

* **Microservicio:** `mxpv2.integration.sync`
* **Rol principal:** Coordinar la sincronización de información de facturación.
* **Función crítica:** Asegurar la coherencia de datos fiscales entre sistemas externos y repositorios locales aplicando transformaciones (estándar ECU) y reglas de negocio.

---

## 3. Arquitectura y componentes

### 3.1. Flujo de procesos ETL

1. **EXTRACTS (E001)**: obtiene la información desde repositorios en MongoDB y/o SQL Server.
2. **TRANSFORMS (T080)**: aplica adaptaciones/normalizaciones al estándar de Ecuador (ECU) y reglas de validación.
3. **LOADS (01)**: inserta o actualiza los datos en el repositorio local `KFC_MXPi_FeBuild`.
4. **PUBLISHERS**: (vacío en este flujo) — reservado para notificaciones o publicaciones posteriores.

---

## 4. Gestión de procesos

### Flujo de trabajo estándar

* **Paso 1 - Extracción de entidades**: `billing_information`, `number_sequence` (código: `E001`).
* **Paso 2 - Transformación**: proceso `T080` — validaciones, normalizaciones y reglas fiscales (ECU).
* **Paso 3 - Carga**: proceso `01` hacia el repositorio local Mongo (`MongoLocal`).

### Reglas operativas

* Si `EXTRACTS` falla, no se ejecutan `TRANSFORMS` ni `LOADS`.
* Las transformaciones deben dejar los datos conformes al esquema destino; cualquier inconsistencia se registra y detiene el flujo para revisión.
* La carga puede ser por lotes (chunkLoad) para evitar saturación.

---

## 5. Endpoint: Sync-Billing-Information

* **Método:** `POST`
* **URL:** `{{server_sync}}/api/v1/facade`

### Headers

```
Authorization: Bearer {{token}}
Content-Type: application/json
```

### Body (raw JSON) — ejemplo completo

```json
{
  "code": "F008",
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "chunkLoad": 10,
  "processes": {
    "EXTRACTS": [
      {
        "code": "E001",
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "withResponse": true,
        "withCache": false,
        "configuration": {
          "connection": {
            "server": "ec-mxpv2-dev.3hro7.mongodb.net",
            "user": "dev_integration",
            "password": "*****",
            "repository": "MXP_integrations",
            "adapter": "Mongo"
          },
          "entities": [
            { "name": "billing_information" },
            { "name": "number_sequence" }
          ]
        }
      }
    ],
    "TRANSFORMS": [
      {
        "code": "T080",
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "withResponse": true,
        "withCache": false,
        "country": "ECU"
      }
    ],
    "LOADS": [
      {
        "code": "01",
        "withCache": true,
        "withResponse": true,
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "configuration": {
          "connection": {
            "server": "192.168.101.13",
            "port": "27020",
            "user": "admin",
            "password": "*****",
            "repository": "KFC_MXPi_FeBuild",
            "adapter": "MongoLocal"
          },
          "entities": [
            { "name": "billing_information" },
            { "name": "number_sequence" }
          ]
        }
      }
    ],
    "PUBLISHERS": []
  }
}
```

### Ejemplo cURL

```bash
curl --location '{{server_sync}}/api/v1/facade' --header 'Authorization: Bearer {{token}}' --header 'Content-Type: application/json' --data '{ ... }'
```

### Ejemplo de respuesta de error

```json
{
  "message": "Error processing request: 'name'"
}
```

---

## 6. Escenarios de uso

### ✅ Caso exitoso

* Los datos se extraen desde Mongo/SQL, se transforman con `T080` y se cargan en el repositorio local.

### ⚠️ Posibles fallos

* **Extracción:** credenciales inválidas, host inaccesible, timeouts.
* **Transformación (T080):** campos faltantes, formato incompatible con estándar ECU.
* **Carga:** conflicto de índices/claves, espacio insuficiente, replica set no disponible.

---

## 7. Preguntas frecuentes (FAQ)

**Q:** ¿Qué pasa si el proceso falla en EXTRACTS?

**A:** Se registra el fallo en logs y el flujo se detiene. No se ejecutarán `TRANSFORMS` ni `LOADS`.

**Q:** ¿Dónde se valida la coherencia de la información de facturación?

**A:** Durante el proceso `T080` (transformación).

**Q:** ¿Este proceso almacena datos?

**A:** No almacena de forma permanente por sí mismo; sincroniza hacia la base destino donde la persistencia depende de ésta.

---

## 8. Operación, monitoreo y troubleshooting

### Logs y trazabilidad

* Cada ejecución debe generar un registro con `syncId`, `code` del flujo (F008), tiempos de inicio/fin por proceso y conteo de registros procesados.
* Los errores se deben registrar con stacktrace y payload mínimo necesario para reproducir.

### Alertas

* Alertas sobre fallos de conexión o tasa de error elevada (por ejemplo >5% de registros con validación fallida).

### Reintentos

* Para errores transitorios (timeouts, errores de red) se implementan reintentos exponenciales limitados (configurable).
* Para errores de negocio (datos no conformes) no se reintenta automáticamente; se envía a cola de revisión manual.

### Seguridad

* No incluir credenciales en logs. Enmascarar (`****`) valores sensibles.
* Uso obligatorio de TLS en conexiones externas.
* Validar tokens y scopes del `Authorization: Bearer`.

---

## 9. Referencias

Fuentes internas y documentación de soporte:

* `mxp-sync/process/readme.md` (líneas 1-28)
* `mxp-feBuild/process/readme.md` (líneas 9-10)
* Microservicio: `mxpv2.integration.sync` (código fuente y README interno)

> Nota: consultar los readme locales para obtener detalles del mapping de campos y ejemplos de payloads por entidad.

---

## 10. Contacto

* **Equipo responsable:** Integraciones MAXPOINT
* **Canal de soporte (Slack):** `#mxp-integraciones` (canal interno)
* **Escalamiento:** Arquitectura de Datos → Liderazgo Técnico
* **Horario de soporte:** horario laboral interno (ver SLA de la organización)
