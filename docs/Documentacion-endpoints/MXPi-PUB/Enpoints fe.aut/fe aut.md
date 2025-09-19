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

## 2. Descripción general del sistema

* **Microservicio:** mxpv2.integration.sync
* **Rol principal:** gestionar procesos de integración y sincronización.
* **Función crítica:** asegurar que los datos transformados lleguen a las fachadas correctas dentro del flujo ETL.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código | Descripción                                                                                                                                                     |
| ------------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Interfaz de recepción ETL | F008   | Punto de entrada principal para datos ETL (extract, transform, load). 🔑 Actúa como gateway unificado para datos procesados, validando estructura e integridad. |

### 3.2. Capa de transformación

| Componente                        | Código | Descripción                                                               |
| --------------------------------- | ------ | ------------------------------------------------------------------------- |
| Construir información Transformar | T080   | Procesa metadatos de compilación, asegurando estructura y disponibilidad. |

## 4. Gestión de procesos

**Flujo de trabajo estándar:**
Ingestión de datos → El F008 recibe los paquetes ETL → Validación → Enrutamiento → Procesamiento de compilación → Distribución.

## 5. Puntos de integración

* **Entrada:** recibe datos procesados desde MXP Init, MXP FE Build y MXP Printer.
* **Salida:** distribuye datos validados y transformados a fachadas de destino.
* **Rol central:** mantiene coherencia en procesos ETL inter-módulos.

## 6. Patrones de sincronización

* Recepción centralizada → Todos los ETL llegan vía F008.
* Procesamiento de metadatos → El módulo T080 gestiona info de compilación.
* Distribución coordinada → Los datos son enviados a fachadas correctas.
* Garantía de coherencia → Evita inconsistencias en la plataforma.

## 7. Escenarios de uso

✅ Caso exitoso: ETL válido → se recibe en F008 → validación → transformación en T080 → distribución correcta.

⚠️ Posibles fallos:

* **Error en validación:** Datos rechazados. Log de error con detalle del paquete fallido. Acción recomendada: verificar fuente.
* **Error en transformación (T080):** Datos no pasan a distribución. Se activa rollback y alerta al sistema de monitoreo.
* **Error en distribución:** Fachada de destino no disponible. El sincronizador reintenta el envío según política de reintentos (retry policy).

## 8. Preguntas frecuentes (para IA Support)

**Q:** ¿Qué pasa si falla la validación en F008?
**A:** El paquete se descarta, se registra en logs y se debe revisar la fuente.

**Q:** ¿Cómo sé que la info de compilación se procesó bien?
**A:** Revisa el log de T080. Si hay éxito, se marca como `build_metadata_ok`.

**Q:** ¿El sincronizador almacena datos?
**A:** No, solo orquesta flujos. La persistencia depende de otros módulos.

**Q:** ¿Qué debo hacer si la distribución falla?
**A:** Se activa la política de reintentos, pero si el error persiste, revisa la fachada de destino y los logs de distribución.

## 9. Referencias

* Fuentes internas: `mxp-sync/process/readme.md (1-28)`
* Microservicio: `mxpv2.integration.sync`

## 10. Contacto

* Equipo responsable: Integraciones MAXPOINT
* Canal de soporte: `#mxp-integraciones` (Slack interno)
* Escalamiento: Arquitectura de datos → Liderazgo técnico

---

# Endpoints

## 1. fe.aut

**Método:** POST
**Endpoint:** `{{server_pub}}/api/v1/facade`
**Autorización:** Bearer Token (`{{token_legacy}}`)

### Body (JSON)

```json
{
  "code": "F008",
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "chunkLoad": 1,
  "processes": { ... }
}
```

### Example Request

```bash
curl --location 'https://{{server_pub}}/api/v1/facade' \
--data '{ "code": "H000", "withResponse": false, "withCache": false, "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6", "country": "{{country}}", "franchise_id": 0, "processes": { ... } }'
```

### Example Response

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

---

## 2. GetAutorizationDocument

**Método:** POST
**Endpoint:** `{{server_pub}}/api/v1/facade`
**Autorización:** Bearer Token (`{{token_legacy}}`)

### Body (JSON)

```json
{
  "code": "F009",
  "withResponse": true,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "chunkLoad": 1,
  "processes": { ... }
}
```

### Example Request

```bash
curl --location 'https://{{server_pub}}/api/v1/facade' \
--data '{ "code": "H000", "withResponse": false, "withCache": false, "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6", "country": "{{country}}", "franchise_id": 0, "processes": { ... } }'
```

### Example Response

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

---

## 3. CheckDocumentStatus

**Método:** GET
**Endpoint:** `{{server_pub}}/P002`
**Parametros:**

* `access_key`: `1505202501179141513200110980880000001604126153310`

### Example Request

```bash
curl --location -g '{{server_pub}}/P002?access_key=1505202501179141513200110980880000001604126153310'
```

### Example Response

```json
{
  "code": 200,
  "status": "success",
  "message": "Data retrieved successfully",
  "data": { ... }
}
```
