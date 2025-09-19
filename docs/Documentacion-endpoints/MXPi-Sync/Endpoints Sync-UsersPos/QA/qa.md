# Módulo de Sincronización (MXP-Sync)

## 1. Propósito y alcance

MXP-Sync es el sistema de orquestación de sincronización de datos dentro del ecosistema **MAXPOINT V2.0**.

* Gestiona el flujo de datos entre diferentes módulos.
* Garantiza la coherencia del sistema.
* Coordina los procesos ETL (Extract, Transform, Load).
* Recibe datos procesados de las tuberías de transformación y los distribuye a las interfaces de fachada (facades).

ℹ️ Para más detalles de los módulos relacionados, consulta:
MXP Init, MXP FE Build, Impresora MXP

## 2. Descripción general del sistema

* **Microservicio:** mxpv2.integration.sync
* **Rol principal:** gestionar procesos de integración y sincronización.
* **Función crítica:** asegurar que los datos transformados lleguen a las fachadas correctas dentro del flujo ETL.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código | Descripción                                                                                                                           |
| ------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| Interfaz de recepción ETL | F008   | Punto de entrada principal para datos ETL (extract, transform, load). Actúa como gateway unificado validando estructura e integridad. |

### 3.2. Capa de transformación

| Componente                 | Código      | Descripción                                                                                            |
| -------------------------- | ----------- | ------------------------------------------------------------------------------------------------------ |
| Transformación de usuarios | T080 / T121 | Procesa metadatos de compilación, asegurando estructura y disponibilidad según país y tipo de usuario. |

## 4. Gestión de procesos

**Flujo de trabajo estándar:**

1. Ingestión de datos → El F008 recibe los paquetes ETL.
2. Validación → Se verifica estructura e integridad.
3. Enrutamiento → Los datos se envían al proceso correcto según el contenido.
4. Procesamiento de transformación → Info de usuarios procesada por T080/T121.
5. Distribución → Datos transformados entregados a las fachadas de destino (Load).

## 5. Puntos de integración

* **Entrada:** recibe datos desde MXP Init, QA y otros orígenes.
* **Salida:** distribuye datos a MongoLocal, fachadas de ARES y otros sistemas.
* **Rol central:** mantiene coherencia en procesos ETL inter-módulos.

## 6. Patrones de sincronización

* Recepción centralizada → Todos los ETL llegan vía F008.
* Procesamiento de metadatos → T080/T121 gestiona info de usuarios y perfiles.
* Distribución coordinada → Datos enviados a fachadas correctas.
* Garantía de coherencia → Evita inconsistencias en la plataforma.

## 7. Escenarios de uso

### 7.1. POST /api/v1/facade - Sync-Generic

* **Descripción:** Sincroniza datos genéricos de usuarios y settings.
* **Autorización:** Bearer Token (`{{token}}`)
* **Body JSON:**

```json
{
    "code": "F008",
    "syncId": "...",
    "chunkLoad": 10,
    "processes": {
        "EXTRACTS": [...],
        "TRANSFORMS": [...],
        "LOADS": [...],
        "PUBLISHERS": []
    }
}
```

* **Ejemplo Request:**

```bash
curl --location 'https://{{server_sync}}/api/v1/facade' \
--header 'Authorization: Bearer {{token}}' \
--data '{...JSON Body...}'
```

* **Ejemplo Response:**

```json
{
  "message": "Error processing request: 'name'"
}
```

### 7.2. POST /api/v1/facade - Sync-Users-Ares

* **Descripción:** Sincroniza usuarios específicos para ARES.

* **Chunk Load:** 100

* **Transformación:** T121

* **Load destino:** MXPi\_ARES\_Validate\_Tenant

* **Body JSON:** Similar al ejemplo anterior, filtrando por franquicia y restaurante.

### 7.3. GET /api/users\_pos - Sync-Expose-Users-Pos

* **Descripción:** Expone usuarios POS filtrando por `cdn_id` y `restaurant_id`.

* **Autorización:** Bearer Token (`{{token}}`)

* **Params:**

  * `cdn_id`: bc2d8ada-e25e-1bb7-55fe-32d1205ac4af
  * `restaurant_id`: e17d03da-b8b6-f424-febc-3a767b6401bb

* **Ejemplo Request:**

```bash
curl --location -g '{{server_pub}}/api/users_pos?cdn_id=...&restaurant_id=...' \
--header 'Authorization: Bearer {{token}}'
```

* **Ejemplo Response:**

```json
{
  "code": 200,
  "status": "success",
  "message": "Data retrieved successfully",
  "details": [
    {
      "user": { ... },
      "roles_profiles_franchise": [ ... ]
    }
  ]
}
```

## 8. Preguntas frecuentes

**Q:** ¿Qué pasa si falla la validación en F008?
**A:** El paquete se descarta y se registra en logs. Revisar la fuente.

**Q:** ¿Cómo sé que la info de usuarios se procesó correctamente?
**A:** Revisa el log de T080/T121. Éxito indica `build_metadata_ok`.

**Q:** ¿El sincronizador almacena datos?
**A:** No, solo orquesta flujos. La persistencia depende de otros módulos.

## 9. Referencias

* mxp-sync/process/readme.md
* mxp-ares/process/readme.md

## 10. Contacto

* **Equipo responsable:** Integraciones MAXPOINT
* **Canal de soporte:** #mxp-integraciones (Slack interno)
* **Escalamiento:** Arquitectura de datos → Liderazgo técnico
