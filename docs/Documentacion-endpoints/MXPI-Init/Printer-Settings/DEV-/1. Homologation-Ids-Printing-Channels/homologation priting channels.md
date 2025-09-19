# Módulo MXPI-INIT SERVER LEGACY PRINTER SETTINGS DEV HOMOLOGATION V1 IDS PRINTING CHANNELS

## 1. Propósito y alcance
El microservicio **MXPI-INIT SERVER LEGACY PRINTER SETTINGS DEV HOMOLOGATION V1 IDS PRINTING CHANNELS** es responsable de la homologación y sincronización de configuraciones heredadas de impresión dentro del ecosistema **MAXPOINT V2.0**.  
Su propósito es garantizar que los **canales de impresión** y los **documentos de impresión** provenientes de sistemas legados se extraigan, transformen, validen y carguen adecuadamente en repositorios modernos (MongoDB).

Funciones principales:
- Homologar IDs de impresión de canales y documentos legados.
- Asegurar coherencia de datos en la migración y sincronización de sistemas de impresión.
- Integrar procesos **ETL** (Extract, Transform, Load) con pruebas de validación automatizadas.

ℹ️ Módulos relacionados:  
- MXP Init  
- MXP Printer  
- MXP FE Build  

---

## 2. Descripción general del sistema
- **Microservicio**: `mxpi-init.printer.homologation`
- **Rol principal**: Homologar configuraciones de impresión legadas (canales y documentos).  
- **Función crítica**: Transformar datos legados de SQL Server hacia estructuras modernas en MongoDB, asegurando consistencia y trazabilidad mediante `syncId`.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Facade de homologación | H000 | Punto de entrada principal para homologación de configuraciones de impresión. |

### 3.2. Capa de extracción (EXTRACTS)
| Componente | Código | Origen | Descripción |
|------------|--------|--------|-------------|
| Extractor de canales de impresión | E001 | SQL Server (tabla `Canal_Impresion`) | Extrae información de canales de impresión con filtros por `cdn_id` y `IDStatus`. |
| Extractor de documentos de impresión | E001 | SQL Server (vista `mxp_v2.View_Printing_Documents`) | Extrae documentos de impresión asociados a franquicias legadas. |

### 3.3. Capa de transformación (TRANSFORMS)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformador de homologación | H001 | Procesa los datos extraídos y los transforma en un formato estándar para MongoDB. |

### 3.4. Capa de carga (LOADS)
| Componente | Código | Destino | Descripción |
|------------|--------|---------|-------------|
| Loader de homologación | L001 | MongoDB (colección `uuids`) | Carga los datos transformados en la base de datos MongoDB validando claves primarias y duplicados. |

---

## 4. Gestión de procesos

### Flujo de trabajo estándar
1. **Ingestión de datos** → Se reciben las peticiones en la fachada `H000`.
2. **Extracción** → Conexión a SQL Server para obtener entidades (`Canal_Impresion`, `View_Printing_Documents`).
3. **Validación** → Se verifican filtros (`cdn_id`, `IDStatus`) y tipos de datos (ej. `base64`).
4. **Transformación** → El proceso `H001` homologa IDs legados a IDs modernos.
5. **Carga** → Los datos procesados se insertan en MongoDB en la colección `uuids`.
6. **Pruebas automatizadas** → Se validan estado HTTP, tiempo de respuesta y estructura JSON.

---

## 5. Puntos de integración

- **Entrada**: Datos de impresión desde **SQL Server Legacy** (`Canal_Impresion`, `View_Printing_Documents`).
- **Salida**: Entidades homologadas hacia **MongoDB** (`uuids`).
- **Rol central**: Garantizar interoperabilidad de sistemas de impresión legados con la infraestructura **MAXPOINT V2.0**.

---

## 6. Patrones de sincronización
- **Recepción centralizada**: Todas las peticiones se canalizan mediante `H000`.
- **Extracción segmentada**: Diferenciación entre entidades (`Canal_Impresion`, `Printing_Documents`).
- **Transformación de homologación**: `H001` asegura consistencia de IDs entre versiones.
- **Carga unificada**: `L001` persiste resultados en MongoDB.

---

## 7. Escenarios de uso

### ✅ Caso exitoso
1. Cliente envía petición a `H000` con `syncId` válido.  
2. `E001` extrae datos desde SQL Server.  
3. `H001` transforma IDs legados.  
4. `L001` carga datos homologados en MongoDB.  
5. Respuesta HTTP 200 con estructura `{syncId, data, cacheId}`.

### ⚠️ Posibles fallos
- **Error en validación**: Filtros inválidos → datos rechazados y logeados.  
- **Error en transformación (H001)**: No se generan IDs homologados → rollback activado.  
- **Error en carga (L001)**: Conflicto de claves en MongoDB → alerta al sistema de monitoreo.  
- **Error en distribución**: Tiempo de respuesta >1000ms → política de reintentos activa.

---

## 8. Endpoints documentados

### 8.1. Homologación de Canales de Impresión
**POST** `{{server_init}}/api/v1/facade`

- **Request Body (ejemplo)**:
```json
{
  "code": "H000",
  "withResponse": false,
  "withCache": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "country": "{{country}}",
  "franchise_id": {{legacy_cdn_id}},
  "processes": {
    "EXTRACTS": [
      {
        "code": "E001",
        "configuration": {
          "entities": [
            {
              "name": "Canal_Impresion",
              "properties": [{ "name": "IDCanalImpresion", "type": "base64" }],
              "filters": [
                { "name": "cdn_id", "operator": "equals", "value": "{{legacy_cdn_id}}" },
                { "name": "IDStatus", "operator": "equals", "value": "8c039503-85cf-e511-80c6-000d3a3261f3" }
              ]
            }
          ]
        }
      }
    ]
  }
}
```

- **Validaciones automatizadas**:
  - Código HTTP `200`
  - Tiempo de respuesta `<1000ms`
  - Content-Type: `application/json`
  - Respuesta esperada:
    ```json
    {
      "syncId": "string",
      "data": null,
      "cacheId": null
    }
    ```

---

### 8.2. Homologación de Documentos de Impresión
**POST** `{{server_init}}/api/v1/facade`

- **Request Body (ejemplo)**:
```json
{
  "code": "H000",
  "withResponse": false,
  "withCache": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "country": "{{country}}",
  "franchise_id": {{legacy_cdn_id}},
  "processes": {
    "EXTRACTS": [
      {
        "code": "E001",
        "configuration": {
          "entities": [
            {
              "name": "mxp_v2.View_Printing_Documents",
              "properties": [{ "name": "documento", "type": "base64" }],
              "filters": [
                { "name": "cdn_id", "operator": "equals", "value": "{{legacy_cdn_id}}" }
              ]
            }
          ]
        }
      }
    ]
  }
}
```

- **Validaciones automatizadas**:
  - Código HTTP `200`
  - Tiempo de respuesta `<1000ms`
  - Content-Type: `application/json`
  - Respuesta esperada:
    ```json
    {
      "syncId": "string",
      "data": null,
      "cacheId": null
    }
    ```

---

## 9. Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si falla la validación en `E001`?**  
A: Los datos se descartan, se registran en logs y se debe revisar la fuente.

**Q: ¿Cómo sé que los datos se homologaron correctamente en `H001`?**  
A: Revisa el log del proceso. Si el campo `build_metadata_ok` está presente, la homologación fue exitosa.

**Q: ¿Este microservicio almacena datos?**  
A: No, solo orquesta procesos. La persistencia se realiza en MongoDB vía `L001`.

**Q: ¿Qué hacer si falla la carga en MongoDB?**  
A: Revisar duplicados de `id_v2` o conflictos en claves primarias. El sistema ejecuta reintentos antes de escalar el error.

---

## 10. Referencias
- **Fuentes internas**:  
  - `mxpi-init/printer/homologation/readme.md`  
  - `mxp-sync/process/readme.md`  
- **Microservicio**: `mxpi-init.printer.homologation`  
- **Contacto**:  
  - Equipo responsable: Integraciones MAXPOINT  
  - Canal de soporte: `#mxp-integraciones` (Slack interno)  
  - Escalamiento: Arquitectura de datos → Liderazgo técnico  
