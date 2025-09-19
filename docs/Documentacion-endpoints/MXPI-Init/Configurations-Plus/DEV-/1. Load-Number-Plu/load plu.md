# Módulo de Configuración Init (MXPi-init – 1. Load-Number-Plu)

## 1. Propósito y alcance

**MXPi-init – 1. Load-Number-Plu** es el microservicio encargado de la **extracción, transformación y carga de identificadores PLU y medios de venta** dentro del ecosistema **MAXPOINT V2.0**.

Funciones principales:

* Extraer información de PLUs y medios de venta desde sistemas legacy (SQL Server).
* Integrar y transformar identificadores para unificarlos en MongoDB.
* Asegurar coherencia y unicidad de los registros PLU y medios de venta.
* Coordinar procesos ETL (Extract, Transform, Load).

ℹ️ Módulos relacionados:

* **MXP Init**
* **MXP FE Build**
* **MXP Printer**

---

## 2. Descripción general del sistema

* **Microservicio:** `mxpv2.integration.init.loadnumberplu`
* **Rol principal:** homologación y carga de identificadores PLU y medios de venta.
* **Función crítica:** transformar datos legacy y garantizar consistencia en MongoDB.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                     | Código | Descripción                                      |
| ------------------------------ | ------ | ------------------------------------------------ |
| Interfaz de homologación PLU   | F100   | Punto de entrada principal para carga de PLUs.   |
| Interfaz de medios de venta    | F100   | Punto de entrada principal para carga de medios. |

### 3.2. Capa de transformación

| Componente | Código | Descripción                                        |
| ---------- | ------ | -------------------------------------------------- |
| Transform  | T128   | Homologación de números PLU.                       |
| Transform  | T129   | Homologación de medios de venta.                   |

### 3.3. Capa de carga

| Componente  | Código | Descripción                                         |
| ----------- | ------ | --------------------------------------------------- |
| Carga Mongo | L001   | Inserta PLUs y medios de venta homologados en DB.   |

---

## 4. Gestión de procesos

**Flujo de trabajo estándar:**

1. **Extracción** → desde SQL Server (`sp_GetNumPluForPlu`, `mxp_v2.Medio_Venta`).
2. **Validación** → se verifican filtros (`cdn_id`, `entity_name`, etc.).
3. **Transformación** → reglas T128 (PLUs) y T129 (Medios).
4. **Carga** → inserción en MongoDB (`plu_num_plu`, `plu_means_sale`).
5. **Confirmación** → logs de éxito o error.

---

## 5. Puntos de integración

* **Entrada:**
  * SQL Server (procedimientos almacenados y tablas legacy).
* **Salida:**
  * MongoDB (`plu_num_plu`, `plu_means_sale`).
* **Rol central:**
  * Garantizar que identificadores de productos y medios se homologuen correctamente.

---

## 6. Patrones de sincronización

* Extracción controlada desde SQL Server.
* Transformación con T128 y T129.
* Persistencia centralizada en MongoDB.
* Validación estricta antes de la carga.

---

## 7. Escenarios de uso

### ✅ Caso exitoso

1. Se extraen datos desde SQL Server.
2. Transformación con reglas T128/T129.
3. Carga en MongoDB.
4. Respuesta JSON de éxito.

### ⚠️ Posibles fallos

* **Extracción:** credenciales inválidas o SP no disponible.
* **Transformación:** reglas inválidas → rollback.
* **Carga:** indisponibilidad de MongoDB → reintentos.

---

## 8. Endpoints documentados

### 🔹 1. Facade-Init-Plu-Num-Plu

**Método:** `POST`  
**URL:**

```http
{{init}}/api/v1/facade
```

**Autorización:**

```
Bearer {{token}}
```

**Body ejemplo:**

```json
{
  "code": "F100",
  "withResponse": true,
  "withCache": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "processes": {
    "EXTRACTS": [...],
    "TRANSFORMS": [...],
    "LOADS": [...]
  }
}
```

**Ejemplo curl:**

```bash
curl --location -g '{{init}}/api/v1/facade' --data '{
  "code": "F100",
  "withResponse": true,
  "withCache": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "processes": {
    "EXTRACTS": [...],
    "TRANSFORMS": [...],
    "LOADS": [...]
  }
}'
```

---

### 🔹 2. Facade-Init-Plu-Menu-Medio

**Método:** `POST`  
**URL:**

```http
{{init}}/api/v1/facade
```

**Autorización:**

```
Bearer {{token}}
```

**Body ejemplo:**

```json
{
  "code": "F100",
  "withResponse": true,
  "withCache": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "processes": {
    "EXTRACTS": [...],
    "TRANSFORMS": [...],
    "LOADS": [...]
  }
}
```

**Ejemplo curl:**

```bash
curl --location -g '{{init}}/api/v1/facade' --data '{
  "code": "F100",
  "withResponse": true,
  "withCache": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "processes": {
    "EXTRACTS": [...],
    "TRANSFORMS": [...],
    "LOADS": [...]
  }
}'
```

---

## 9. Preguntas frecuentes (IA Support)

**Q:** ¿Qué pasa si falla la extracción?  
**A:** Se genera log y se detiene el flujo.

**Q:** ¿Dónde se guardan los datos homologados?  
**A:** En MongoDB (`plu_num_plu`, `plu_means_sale`).

**Q:** ¿Qué hago si falla la carga?  
**A:** Verificar conexión con MongoDB y revisar logs de `L001`.

---

## 10. Referencias

* `mxpi-init/process/readme.md`
* `mxp-sync/process/readme.md`
* `mxp-feBuild/process/readme.md`

---

## 11. Contacto

* **Equipo responsable:** Integraciones MAXPOINT
* **Canal de soporte:** `#mxp-integraciones` (Slack interno)
* **Escalamiento:** Arquitectura de datos → Liderazgo técnico
