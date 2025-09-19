# Módulo de Configuración Plus (MXPi-init Configurations-Plus DEV)

**1.- Homologation-Number-Plu**

------------------------------------------------------------------------

## 1. Propósito y alcance

**MXPi-init Configurations-Plus** es el microservicio encargado de la
**homologación, configuración y carga de identificadores PLU** dentro
del ecosistema **MAXPOINT V2.0**. Su función es extraer datos desde
sistemas **legacy (SQL Server)**, aplicar procesos de homologación
(transformación de IDs) y persistirlos en **MongoDB**, asegurando
consistencia y unicidad.

Funciones principales:

-   Homologar números PLU desde canales de impresión y usuarios POS.
-   Integrar datos transformados en repositorios unificados (`uuids`).
-   Garantizar la coherencia de identificadores entre sistemas v1 y v2.
-   Coordinar procesos ETL (Extract, Transform, Load).

ℹ️ Módulos relacionados:

-   **MXP Init**
-   **MXP FE Build**
-   **MXP Printer**

------------------------------------------------------------------------

## 2. Descripción general del sistema

-   **Microservicio:** `mxpv2.integration.init.configurations.plus`
-   **Rol principal:** gestión de homologaciones PLU.
-   **Función crítica:** transformar y cargar identificadores legacy en
    el sistema v2 con consistencia.

------------------------------------------------------------------------

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

  ---------------------------------------------------------------------------
  Componente         Código   Descripción
  ------------------ -------- -----------------------------------------------
  Interfaz de        F009     Punto de entrada principal para homologar
  homologación                identificadores PLU.

  ---------------------------------------------------------------------------

### 3.2. Capa de transformación

  Componente                Código   Descripción
  ------------------------- -------- ---------------------------------
  Homologación de IDs PLU   H001     Transforma IDs legacy a IDs v2.

### 3.3. Capa de carga

  --------------------------------------------------------------------------
  Componente   Código   Descripción
  ------------ -------- ----------------------------------------------------
  Carga Mongo  L001     Inserta datos homologados en MongoDB (`uuids`).

  --------------------------------------------------------------------------

------------------------------------------------------------------------

## 4. Gestión de procesos

**Flujo de trabajo estándar:**

1.  **Extracción** → desde SQL Server (`Canal_Impresion`, `Users_Pos`).
2.  **Validación** → aplica filtros (`cdn_id`, `IDStatus`,
    `IDPerfilPos`).
3.  **Transformación** → reglas `H001` para homologación v1 → v2.
4.  **Carga** → inserción en MongoDB (`uuids`).
5.  **Confirmación** → logs de éxito o error.

------------------------------------------------------------------------

## 5. Puntos de integración

-   **Entrada:**

    -   `Canal_Impresion` (PLU desde canales de impresión).
    -   `Users_Pos` (usuarios POS filtrados).

-   **Salida:**

    -   MongoDB colección `uuids`.

-   **Rol central:** asegurar que todos los identificadores PLU se
    homologuen correctamente.

------------------------------------------------------------------------

## 6. Patrones de sincronización

-   Extracción controlada desde SQL Server.
-   Transformación de IDs con `H001`.
-   Persistencia centralizada en MongoDB.
-   Validación estricta antes de la carga.

------------------------------------------------------------------------

## 7. Escenarios de uso

### ✅ Caso exitoso

1.  Se extraen datos de SQL Server.
2.  Transformación en `H001`.
3.  Carga en MongoDB.
4.  Respuesta JSON con éxito.

### ⚠️ Posibles fallos

-   **Extracción:** credenciales inválidas o entidad no encontrada.
-   **Transformación:** reglas inválidas → rollback.
-   **Carga:** indisponibilidad de MongoDB → política de reintentos.

------------------------------------------------------------------------

## 8. Endpoints documentados

### 🔹 1. Homologation-Number-Plu

**Método:** `POST` **URL:**

``` http
{{server_init}}/api/v1/facade
```

**Autorización:**

    Bearer {{token}}

**Body ejemplo:**

``` json
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
    "LOADS": [...]
  }
}
```

------------------------------------------------------------------------

### 🔹 Ejemplo completo (Request)

``` bash
curl --location 'https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/facade' --data '{
  "code": "H000",
  "withResponse": false,
  "withCache": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "country": "{{country}}",
  "franchise_id": 0,
  "processes": {
    "EXTRACTS": [...],
    "TRANSFORMS": [...],
    "LOADS": [...]
  }
}'
```

### 🔹 Ejemplo de respuesta (Response)

``` json
{
  "code": 200,
  "status": "success",
  "process": "I001",
  "message": "Integración completada exitosamente",
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "data": {
    "EXTRACTS": [
      {
        "code": 200,
        "status": "success",
        "process": "E001",
        "message": "Extracción completada exitosamente",
        "dateTime": "2025-02-04T17:32:05.740+00:00"
      }
    ],
    "TRANSFORMS": [
      {
        "code": 200,
        "status": "success",
        "process": "T003",
        "message": "Transformación completada exitosamente",
        "dateTime": "2025-02-04T17:32:05.740+00:00"
      }
    ],
    "LOAD": [
      {
        "code": 200,
        "status": "success",
        "process": "L001",
        "message": "Carga completada exitosamente",
        "dateTime": "2025-02-04T17:32:05.740+00:00"
      }
    ]
  }
}
```

------------------------------------------------------------------------

## 9. Preguntas frecuentes (IA Support)

**Q:** ¿Qué pasa si falla la extracción?\
**A:** Se genera log y se detiene el flujo.

**Q:** ¿Dónde se guardan los datos homologados?\
**A:** En MongoDB (`uuids`).

**Q:** ¿Qué hago si falla la carga?\
**A:** Verificar conexión con MongoDB y revisar logs de `L001`.

------------------------------------------------------------------------

## 10. Referencias

-   `mxpi-init/process/readme.md`
-   `mxp-sync/process/readme.md`
-   `mxp-feBuild/process/readme.md`

------------------------------------------------------------------------

## 11. Contacto

-   **Equipo responsable:** Integraciones MAXPOINT
-   **Canal de soporte:** `#mxp-integraciones` (Slack interno)
-   **Escalamiento:** Arquitectura de datos → Liderazgo técnico
