# Módulo de Homologación de Cajas (MXPI-INIT SERVER LEGACY - HOMOLOGATION CASH REGISTRY QA)

## 1. Propósito y alcance

El módulo **MXPI-INIT SERVER LEGACY - HOMOLOGATION CASH REGISTRY QA** se encarga de gestionar los procesos de homologación de cajas registradoras provenientes del sistema **Legacy** hacia la arquitectura **MAXPOINT V2.0**.
Su objetivo es garantizar la coherencia de los identificadores de cajas entre sistemas antiguos (v1) y el nuevo ecosistema (v2), manteniendo consistencia y evitando duplicidad.

🔑 Este módulo coordina procesos **ETL (Extract, Transform, Load)**, asegurando que la información de las cajas sea extraída desde Legacy, transformada bajo los lineamientos de homologación y cargada en MongoDB.

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Init
* MXP FE Build
* MXP Sync
* MXP Legacy

---

## 2. Descripción general del sistema

* **Microservicio:** `mxpv2.init.homologation`
* **Rol principal:** homologar información de cajas registradoras desde Legacy.
* **Función crítica:** asegurar la correspondencia correcta de identificadores de cajas y persistir la homologación en la base de datos QA.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código   | Descripción                                                                                                    |
| ------------------------- | -------- | -------------------------------------------------------------------------------------------------------------- |
| Interfaz de recepción ETL | **H000** | Endpoint principal para iniciar la homologación de cajas. Recibe los parámetros de país, cadena y restaurante. |

### 3.2. Capa de extracción

| Componente       | Código   | Descripción                                                                                   |
| ---------------- | -------- | --------------------------------------------------------------------------------------------- |
| Extractor Legacy | **E001** | Extrae información de cajas desde la vista `mxp_v2.View_Cash_Register` en el servidor Legacy. |

### 3.3. Capa de transformación

| Componente                    | Código   | Descripción                                                                |
| ----------------------------- | -------- | -------------------------------------------------------------------------- |
| Transformador de homologación | **H001** | Procesa los datos de cajas extraídos, homologando identificadores v1 ↔ v2. |

### 3.4. Capa de carga

| Componente | Código   | Descripción                                                                                                      |
| ---------- | -------- | ---------------------------------------------------------------------------------------------------------------- |
| Loader QA  | **L001** | Carga los datos homologados en la colección `uuids` del repositorio `QA_KFC_MXPi-Homologation_v1_v2` en MongoDB. |

---

## 4. Gestión de procesos

**Flujo de trabajo estándar:**

1. **Ingestión de datos** → `H000` recibe la solicitud de homologación.
2. **Extracción** → `E001` consulta las cajas registradoras en `mxp_v2.View_Cash_Register`.
3. **Validación y transformación** → `H001` aplica las reglas de homologación.
4. **Carga** → `L001` guarda los datos homologados en la base de datos QA.

---

## 5. Endpoint documentado

### POST Homologación de Cajas Legacy → QA

```
POST {{server_init}}/api/v1/facade
```

#### Body de ejemplo

```json
{
    "code": "H000",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "country": "{{country}}",
    "franchise_id": {{legacy_cdn_id}},
    "processes": {
        "EXTRACTS": [ { ... } ],
        "TRANSFORMS": [ { ... } ],
        "LOADS": [ { ... } ],
        "PUBLISHERS": []
    }
}
```

#### Respuesta esperada

```json
{
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "data": null,
  "cacheId": null
}
```

---

## 6. Patrones de sincronización

* **Recepción centralizada** → Todos los procesos inician vía `H000`.
* **Extracción SQL** → Se obtiene la información desde `mxp_v2.View_Cash_Register` en Legacy.
* **Transformación de homologación** → `H001` normaliza y valida identificadores.
* **Carga en MongoDB** → `L001` persiste la homologación en QA.

---

## 7. Escenarios de uso

✅ **Caso exitoso**:
Se recibe solicitud en `H000` → extracción `E001` → transformación `H001` → carga `L001` → respuesta con `syncId`.

⚠️ **Posibles fallos**:

* Error en extracción (E001): servidor Legacy no responde → log de error.
* Error en transformación (H001): datos inconsistentes → rollback.
* Error en carga (L001): MongoDB no disponible → política de reintentos.

---

## 8. Validaciones automatizadas (Postman)

```javascript
// Código de estado
pm.test("El código de estado es 200", function () {
    pm.response.to.have.status(200);
});

// Tiempo de respuesta < 1000ms
pm.test("El tiempo de respuesta es menor a 1000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});

// Tipo de contenido JSON
pm.test("El tipo de contenido es application/json", function () {
    pm.expect(pm.response.headers.get('Content-Type')).to.include('application/json');
});

// Validación de estructura de respuesta
const response = pm.response.json();
pm.test("La respuesta contiene las propiedades esperadas", function () {
    pm.expect(response).to.have.all.keys('syncId', 'data', 'cacheId');
});

// syncId es string
pm.test("syncId es una cadena", function () {
    pm.expect(response.syncId).to.be.a('string');
});

// data es nulo
pm.test("data es nulo", function () {
    pm.expect(response.data).to.be.null;
});

// cacheId es nulo
pm.test("cacheId es nulo", function () {
    pm.expect(response.cacheId).to.be.null;
});
```

---

## 9. Preguntas frecuentes (para IA Support)

* **Q:** ¿Qué pasa si falla la extracción en `E001`?
  **A:** El sistema registra el error en logs y no avanza en el flujo. Revisar la conexión a Legacy.

* **Q:** ¿Cómo sé que la homologación se procesó bien?
  **A:** Revisa los logs de `H001`. Si la homologación fue exitosa, los identificadores quedan registrados en Mongo.

* **Q:** ¿Dónde se almacenan los datos homologados?
  **A:** En la colección `uuids` del repositorio QA `QA_KFC_MXPi-Homologation_v1_v2`.

* **Q:** ¿Qué hacer si falla la carga en QA?
  **A:** Se activa la política de reintentos. Si persiste, verificar conexión a MongoDB QA.

---

## 10. Referencias

* **Fuente interna:** `mxpi-init/server-legacy/homologation/readme.md`
* **Microservicio:** `mxpv2.init.homologation`
* **Equipo responsable:** Integraciones MAXPOINT
* **Canal de soporte:** `#mxp-integraciones` (Slack interno)
* **Escalamiento:** Arquitectura de datos → Liderazgo técnico
