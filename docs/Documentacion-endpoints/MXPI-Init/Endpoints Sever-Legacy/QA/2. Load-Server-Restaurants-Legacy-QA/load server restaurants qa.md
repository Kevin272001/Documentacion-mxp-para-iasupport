# 📖 Documentación del Módulo MXPI-INIT SERVER LEGACY - FACADE INIT SERVER LEGACY QA

---

## 1. Propósito y alcance

El módulo **MXPI-INIT SERVER LEGACY - FACADE INIT SERVER LEGACY QA** actúa como un **orquestador ETL** encargado de extraer, transformar y cargar información de servidores legacy dentro del ecosistema **MAXPOINT V2.0**. Su principal objetivo es garantizar la correcta homologación de configuraciones y datos relacionados con **servidores y cajas registradoras**.

Coordina los procesos de:

* **Extracción** desde fuentes SQL Server y MongoDB.
* **Transformación** de datos en el proceso T122.
* **Carga** hacia los repositorios de configuración (MongoDB QA).

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Init
* MXP FE Build
* Impresora MXP

---

## 2. Descripción general del sistema

* **Microservicio:** `mxpv2.integration.init.facade`
* **Rol principal:** Gestionar procesos de integración entre servidores legacy y repositorios de configuración.
* **Función crítica:** Asegurar que los datos de servidores y cajas registradoras se sincronicen correctamente en entornos QA.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

* **Componente:** F101
* **Descripción:** Punto de entrada principal que recibe los procesos ETL para inicializar servidores legacy.

### 3.2. Capa de extracción

* **E001:** Obtiene servidores y restaurantes desde `mxp_v2.USP_Server_Restaurants` (SQL Server SP).
* **E002:** Extrae identificadores (UUIDs) de país desde MongoDB QA.
* **E003:** Extrae datos de cadenas y restaurantes asociados a un país y franquicia.
* **E004:** Extrae configuraciones de cajas registradoras desde MongoDB QA.

### 3.3. Capa de transformación

* **T122:** Proceso que normaliza y transforma la información de servidores legacy para su homologación.

### 3.4. Capa de carga

* **L001:** Inserta los datos procesados en la colección `Settings_BO_MXPLegacy_Server_Settings` en MongoDB QA.

---

## 4. Gestión de procesos

**Flujo estándar de trabajo:**

1. **Ingestión de datos** → F101 recibe el paquete ETL.
2. **Extracción** → Se ejecutan procesos E001 a E004 desde SQL Server y MongoDB QA.
3. **Transformación** → T122 asegura la consistencia de la data extraída.
4. **Carga** → L001 inserta los datos finales en la colección de configuración.
5. **Distribución** → La configuración procesada queda disponible para otros módulos.

---

## 5. Endpoint documentado

### POST `/api/v1/facade`

**Request Body:**

```json
{
  "code": "F101",
  "withResponse": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "country": "{{country}}",
  "franchise_id": {{legacy_cdn_id}},
  "chunkLoad": 100,
  "processes": {
    "EXTRACTS": [...],
    "TRANSFORMS": [...],
    "LOADS": [...],
    "PUBLISHERS": []
  }
}
```

**Explicación:**

* `code: F101` → Código principal de la fachada Init Server Legacy.
* `syncId` → UUID de control de sincronización.
* `country` → Código de país.
* `franchise_id` → Identificador de franquicia Legacy.
* `chunkLoad` → Define el tamaño de los bloques de datos procesados.
* `processes` → Define el flujo ETL:

  * **EXTRACTS (E001-E004):** Extracciones de SQL Server y Mongo.
  * **TRANSFORMS (T122):** Transformación de datos.
  * **LOADS (L001):** Carga en MongoDB QA.

---

## 6. Patrones de sincronización

* **Recepción centralizada** en F101.
* **Procesamiento jerárquico** de país, cadena y restaurante.
* **Transformación T122** para estandarizar configuraciones.
* **Carga en Mongo QA** para homologación de servidores legacy.

---

## 7. Escenarios de uso

✅ **Caso exitoso:**

* Se recibe F101 → validación → ejecución de E001-E004 → transformación T122 → carga en L001 → datos listos en Mongo QA.

⚠️ **Posibles fallos:**

* **Error en E001:** No se encuentran servidores legacy. → Revisar conexión SQL Server.
* **Error en T122:** Fallo en normalización. → Se activa rollback.
* **Error en L001:** Inserción fallida en Mongo QA. → Revisar credenciales y esquema de colección.

---

## 8. Preguntas frecuentes (IA Support)

**Q:** ¿Qué ocurre si no se encuentra información en `mxp_v2.USP_Server_Restaurants`?

* **A:** Se genera log de error. Verificar conexión y existencia de datos en SQL Server.

**Q:** ¿Dónde se almacenan las configuraciones procesadas?

* **A:** En la colección `Settings_BO_MXPLegacy_Server_Settings` en MongoDB QA.

**Q:** ¿El módulo permite reintentos automáticos?

* **A:** Sí, aplica política de reintentos en caso de fallos temporales.

---

## 9. Referencias

* Fuentes internas: `mxp-init/process/readme.md`
* Microservicio: `mxpv2.integration.init.facade`
* Contacto: Equipo de Integraciones MAXPOINT
* Canal de soporte: `#mxp-integraciones` (Slack interno)
* Escalamiento: Arquitectura de datos → Liderazgo técnico
