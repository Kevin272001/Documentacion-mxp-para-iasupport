# Módulo y Microservicio: Homologation-v1-Ids-Catalogs-For-MXPv2

**Propósito y alcance**

Homologation-v1-Ids-Catalogs-For-MXPv2 es el microservicio responsable de gestionar la homologación de identificadores y catálogos dentro del ecosistema MAXPOINT V2.0. Se encarga de orquestar los procesos ETL (Extract, Transform, Load) necesarios para mapear y persistir relaciones entre identificadores v2 → v1 y mantener catálogos de referencia.

- Garantiza la consistencia entre identificadores de diferentes versiones.
- Centraliza procesos de extracción desde fuentes legadas, transformaciones y cargas en repositorios destino (por ejemplo, MongoDB).
- Expuesto como fachada (facade) para iniciar sincronizaciones.

ℹ️ Módulos relacionados: MXP Init, MXP Sync, MXP FE Build.

---

## 1. Descripción general del sistema

- **Microservicio:** Homologation-v1-Ids-Catalogs-For-MXPv2
- **Endpoint principal:** `POST {{server_init}}/api/v1/facade`
- **Rol principal:** orquestar procesos de extracción, transformación y carga para la homologación de identificadores y catálogos.
- **Función crítica:** asegurar que los `uuids` y metadatos asociados queden persistidos y normalizados en el repositorio destino.

---

## 2. Endpoint: Homologation-v1-Ids-Catalogs-For-MXPv2

### POST /api/v1/facade

**Authorization**

```
Authorization: Bearer {{token}}
```

**Body (raw JSON)**

```json
{
  "code": "H000",
  "withResponse": true,
  "withCache": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "country": "{{country}}",
  "franchise_id": 0,
  "processes": {
    "EXTRACTS": [ ... ],
    "TRANSFORMS": [ ... ],
    "LOADS": [ ... ],
    "PUBLISHERS": []
  }
}
```

**Ejemplo de solicitud (curl)**

```bash
curl --location -g '{{server_init}}/api/v1/facade' \
--header 'Authorization: Bearer {{token}}' \
--header 'Content-Type: application/json' \
--data '{ ... }'
```

> Sustituir `...` por el JSON completo mostrado en la sección *Body (raw JSON)*.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

- **Facada (FACADE)** — Punto de entrada `POST /api/v1/facade` que recibe la especificación del proceso a ejecutar. Valida headers, token y estructura del payload.

### 3.2. Capa de extracción (EXTRACTS)

- **E001** — Orquestador de extracción desde la base legada (ej. SqlServer). Configura conexiones, ejecuta consultas/lecturas sobre `views` especificadas y mapea propiedades.

### 3.3. Capa de transformación (TRANSFORMS)

- **H001** — Transformaciones específicas de homologación (ej. normalización de identificadores, decodificación base64 a hex/string, reglas de negocio por país).

### 3.4. Capa de carga (LOADS)

- **01** — Carga en MongoDB (u otro repositorio configurado). La entidad `uuids` define mapeos `fromName` → `toName`, tipos de destino y claves primarias.

### 3.5. Publicadores (PUBLISHERS)

- Actualmente `PUBLISHERS` está vacío en el payload de ejemplo, pero el sistema soporta publicar resultados a colas, webhooks o sistemas de notificación.

---

## 4. Gestión de procesos

**Flujo de trabajo estándar**

1. Cliente invoca `POST /api/v1/facade` con el payload de proceso.
2. Fasada valida token, permiso y esquema.
3. Se ejecutan las etapas en orden: `EXTRACTS` → `TRANSFORMS` → `LOADS` → `PUBLISHERS`.
4. Cada etapa reporta estado (ok / error) y el sistema puede devolver respuesta inmediata o asíncrona según `withResponse`.

**Notas sobre `withResponse` y `withCache`**

- `withResponse: true` — Indica que se espera respuesta directa con el resultado o estado del proceso.
- `withCache: true|false` — Controla si el resultado debe almacenarse en cache para reutilización.

---

## 5. Validaciones y reglas

- `syncId` debe ser un UUID válido por proceso (ej. `3fa85f64-5717-4562-b3fc-2c963f66afa6`).
- Las conexiones deben incluir `adapter` válido (`SqlServer`, `Mongo`, etc.).
- En `entities.properties`, cuando `type` = `base64`, se espera que el sistema decodifique el valor según la lógica de transformación.
- En cargas, `isPrimaryKey: "true"` marca la propiedad que determina unicidad en el repositorio destino.

---

## 6. Escenarios de uso y manejo de errores

### Caso exitoso

- ETL válido → `EXTRACTS` lee registros → `TRANSFORMS` normaliza ids → `LOADS` inserta/actualiza `uuids` en MongoDB → respuesta con estado `200` y resumen.

### Errores comunes

- **Fallo en autenticación** — Código `401`. Verificar `Authorization` header.
- **Error de conexión a origen** — Código `502` o `500`. Revisar credenciales y disponibilidad del servidor legado.
- **Error de transformación** — Registro saltado y log detallado. Posible rollback parcial según política.
- **Error en carga** — Reintentos según política; si persiste, se genera alerta y se marca el sync como fallido.

---

## 7. Preguntas frecuentes (FAQ)

**Q: Qué hace `Homologation-v1-Ids-Catalogs-For-MXPv2`?**
A: Homologa identificadores entre versiones y persiste mappings en repositorios de referencia.

**Q: Qué significan `EXTRACTS`, `TRANSFORMS` y `LOADS`?**
A: Etapas clásicas del patrón ETL: extracción desde origen, transformación de datos y carga al destino.

**Q: Dónde se configuran las conexiones?**
A: En el objeto `configuration.connection` de cada proceso (`EXTRACTS`, `LOADS`, etc.).

---

## 8. Referencias internas

- `mxp-init/process/readme.md`
- `mxp-sync/process/readme.md`
- `mxp-feBuild/process/readme.md`

---

## 9. Contacto

- **Equipo responsable:** Integraciones MAXPOINT
- **Canal de soporte:** #mxp-integraciones (Slack interno)
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico

---

*Documento generado automáticamente: Homologation-v1-Ids-Catalogs-For-MXPv2*
