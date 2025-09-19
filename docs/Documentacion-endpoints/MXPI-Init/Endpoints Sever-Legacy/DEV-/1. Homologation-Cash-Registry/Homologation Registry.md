### Módulo de Homologación (MXPI-SERVER LEGACY DEV Homologation Cash Registry)

---

## 1. Propósito y alcance

El módulo **MXPI-SERVER LEGACY Homologation Cash Registry** gestiona la homologación y registro de cajas dentro del ecosistema **MAXPOINT V2.0**. Su objetivo principal es:

* Asegurar la coherencia de los registros de caja en el sistema.
* Estandarizar la información proveniente de sistemas legacy.
* Procesar, transformar y almacenar datos homologados en repositorios modernos.
* Orquestar el flujo **ETL (Extract, Transform, Load)** enfocado en el registro de cajas.

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Init
* MXP FE Build
* MXP Printer

---

## 2. Descripción general del sistema

* **Microservicio:** `mxpv2.integration.sever legacy`
* **Rol principal:** Gestionar la homologación de cajas.
* **Función crítica:** Estandarizar, validar y almacenar información de registros de caja en el flujo ETL.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código   | Descripción                                                                                                                    |
| ------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Interfaz de recepción ETL | **H000** | Punto de entrada para homologación de cajas. Valida la estructura de datos, maneja parámetros de cache y control de respuesta. |

### 3.2. Capa de transformación

| Componente                     | Código   | Descripción                                                                        |
| ------------------------------ | -------- | ---------------------------------------------------------------------------------- |
| Transformación de homologación | **H001** | Se encarga de transformar los datos de caja según parámetros de país y franquicia. |

### 3.3. Capa de carga

| Componente          | Código   | Descripción                                                                                         |
| ------------------- | -------- | --------------------------------------------------------------------------------------------------- |
| Registro en MongoDB | **L001** | Carga los datos homologados en el repositorio Mongo, validando duplicidad de registros mediante PK. |

---

## 4. Gestión de procesos

### Flujo de trabajo estándar

1. **Ingestión de datos:** El endpoint H000 recibe la solicitud con configuraciones de conexión.
2. **Extracción (E001):** Los datos de `mxp_v2.View_Cash_Register` se extraen desde el sistema legacy.
3. **Transformación (H001):** Los datos se adaptan a los estándares internos, considerando país y franquicia.
4. **Carga (L001):** Los registros se almacenan en MongoDB con validación de duplicados.
5. **Distribución:** Los datos homologados quedan disponibles para consumo por otros módulos.

---

## 5. Puntos de integración

El módulo conecta con:

* **Entrada:** Servidores legacy SQL Server.
* **Transformación:** Procesos de homologación con códigos de país y franquicia.
* **Salida:** Repositorios MongoDB homologados.

Rol central: Mantener coherencia de registros de caja entre sistemas legacy y la plataforma moderna.

---

## 6. Patrones de homologación

* **Recepción centralizada:** Todos los procesos llegan vía H000.
* **Extracción validada:** Se extraen datos con filtros (ejemplo: `cdn_id`).
* **Transformación estándar:** Adaptación según país/franquicia.
* **Carga confiable:** Los registros en Mongo se validan contra duplicados.

---

## 7. Endpoint principal

### POST `{{server_init}}/api/v1/facade`

#### Request Body

```json
{
  "code": "H000",
  "withResponse": false,
  "withCache": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "country": "{{country}}",
  "franchise_id": {{legacy_cdn_id}},
  "processes": {
    "EXTRACTS": [...],
    "TRANSFORMS": [...],
    "LOADS": [...],
    "PUBLISHERS": []
  }
}
```

#### Validaciones incluidas

* **Status 200** obligatorio.
* **Tiempo de respuesta** < 1000ms.
* **Tipo de contenido:** `application/json`.
* **Propiedades esperadas en respuesta:** `syncId`, `data`, `cacheId`.

#### Ejemplo de Respuesta

```json
{
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "data": null,
  "cacheId": null
}
```

---

## 8. Escenarios de uso

✅ **Caso exitoso:**

* Se envía un request H000 → Extracción E001 → Transformación H001 → Carga L001 → Respuesta OK.

⚠️ **Posibles fallos:**

* **Error en validación:** Datos rechazados en H000. Acción: revisar fuente legacy.
* **Error en transformación H001:** Datos no homologados. Se activa rollback.
* **Error en carga L001:** Registro duplicado o Mongo no disponible. Acción: revisar logs y política de reintentos.

---

## 9. Preguntas frecuentes (para IA Support)

**Q:** ¿Qué pasa si falla la validación en H000?
**A:** El paquete se descarta y se registra en logs.

**Q:** ¿Cómo sé si la homologación se procesó correctamente?
**A:** Revisa los logs de H001. Si hay éxito, se marca como `homologation_ok`.

**Q:** ¿El módulo almacena datos?
**A:** Sí, en MongoDB mediante L001.

**Q:** ¿Qué debo hacer si la carga falla en MongoDB?
**A:** Revisar duplicidad de registros y conectividad. La política de reintentos se activa automáticamente.

---

## 10. Referencias

* **Fuentes internas:**

  * `mxpi-init/process/readme.md` 
  * `mxp-sync/process/readme.md` 

* **Microservicio:** `mxpv2.integration.server legacy`

* **Contacto:**

  * Equipo responsable: Integraciones MAXPOINT
  * Canal de soporte: `#mxp-integraciones` (Slack interno)
  * Escalamiento: Arquitectura de datos → Liderazgo técnico
