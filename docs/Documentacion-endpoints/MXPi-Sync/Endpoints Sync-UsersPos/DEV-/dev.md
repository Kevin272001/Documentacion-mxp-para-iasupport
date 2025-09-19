---

title: Módulo de Sincronización MXP-Sync - Users POS
sidebar\_label: MXP-Sync Users POS
----------------------------------

# Módulo de Sincronización (MXP-Sync)

## 1. Propósito y alcance

MXP-Sync es el sistema de orquestación de sincronización de datos dentro del ecosistema **MAXPOINT V2.0**.

* Gestiona el flujo de datos procesados entre diferentes módulos.
* Garantiza la coherencia del sistema en toda la plataforma.
* Coordina los procesos ETL (Extract, Transform, Load).
* Recibe datos procesados de las tuberías de transformación y los distribuye a las interfaces de fachada (facades).

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Init
* MXP FE Build
* Impresora MXP

## 2. Descripción general del sistema

* **Microservicio:** `mxpv2.integration.sync`
* **Rol principal:** gestionar procesos de integración y sincronización.
* **Función crítica:** asegurar que los datos transformados lleguen a las fachadas correctas dentro del flujo ETL.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código | Descripción                                                                                                                                                          |
| ------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Interfaz de recepción ETL | F008   | Punto de entrada principal para datos ETL (extract, transform, load). El F008 actúa como gateway unificado para datos procesados, validando estructura e integridad. |

### 3.2. Capa de transformación

| Componente                        | Código      | Descripción                                                                                |
| --------------------------------- | ----------- | ------------------------------------------------------------------------------------------ |
| Construir información Transformar | T080 / T121 | Procesa metadatos de compilación y transformación, asegurando estructura y disponibilidad. |

## 4. Gestión de procesos

### Flujo de trabajo estándar

1. **Ingestión de datos:** El F008 recibe los paquetes ETL desde las fuentes configuradas.
2. **Validación:** Se verifica la estructura e integridad del paquete.
3. **Enrutamiento:** Los datos son enviados al proceso correcto según su contenido.
4. **Procesamiento de compilación:** La info de construcción se transforma con T080/T121.
5. **Distribución:** Los datos procesados se entregan a las fachadas de destino.

## 5. Puntos de integración

El sincronizador conecta todo el ecosistema **MAXPOINT V2.0**:

* **Entrada:** recibe datos procesados desde MXP Init, MXP FE Build y MXP Printer.
* **Salida:** distribuye datos validados y transformados a fachadas de destino.
* **Rol central:** mantiene coherencia en procesos ETL inter-módulos.

## 6. Patrones de sincronización

* **Recepción centralizada:** todos los ETL llegan vía F008.
* **Procesamiento de metadatos:** los módulos T080 y T121 gestionan info de compilación y transformación.
* **Distribución coordinada:** los datos son enviados a fachadas correctas.
* **Garantía de coherencia:** evita inconsistencias en la plataforma.

## 7. Escenarios de uso

### ✅ Caso exitoso

* ETL válido → se recibe en F008 → validación → transformación en T080/T121 → distribución correcta.

### ⚠️ Posibles fallos

* **Error en validación:**

  * Datos rechazados.
  * Se genera log de error con detalle del paquete fallido.
  * Acción recomendada: verificar fuente.

* **Error en transformación (T080/T121):**

  * Datos no pasan a distribución.
  * Se activa rollback y alerta al sistema de monitoreo.

* **Error en distribución:**

  * Fachada de destino no disponible.
  * El sincronizador reintenta el envío según política de reintentos (retry policy).

## 8. Endpoints

### 8.1. Sync-Generic

**POST** `{{server_sync}}/api/v1/facade`
**Authorization:** Bearer Token `{{token}}`

**Body (JSON):**

```json
{
  "code": "F008",
  "syncId": "...",
  "chunkLoad": 10,
  "processes": { ... }
}
```

### 8.2. Sync-Users-Ares

**POST** `{{server_sync}}/api/v1/facade`
**Authorization:** Bearer Token `{{token}}`

**Body (JSON):**

```json
{
  "code": "F008",
  "syncId": "...",
  "chunkLoad": 100,
  "processes": { ... }
}
```

### 8.3. Sync-Expose-Users-Pos

**GET** `{{server_sync}}/api/users_pos?cdn_id=<cdn_id>&restaurant_id=<restaurant_id>`
**Authorization:** Bearer Token `{{token}}`

**Params:**

| Param          | Valor                                |
| -------------- | ------------------------------------ |
| cdn\_id        | bc2d8ada-e25e-1bb7-55fe-32d1205ac4af |
| restaurant\_id | e17d03da-b8b6-f424-febc-3a767b6401bb |

**Example Response:**

```json
{
  "code": 200,
  "status": "success",
  "message": "Data retrieved successfully",
  "details": [ ... ]
}
```

## 9. Preguntas frecuentes (para IA Support)

**Q:** ¿Qué pasa si falla la validación en F008?
**A:** El paquete se descarta, se registra en logs y se debe revisar la fuente.

**Q:** ¿Cómo sé que la info de compilación se procesó bien?
**A:** Revisa el log de T080/T121. Si hay éxito, se marca como `build_metadata_ok`.

**Q:** ¿El sincronizador almacena datos?
**A:** No, solo orquesta flujos. La persistencia depende de otros módulos.

**Q:** ¿Qué debo hacer si la distribución falla?
**A:** Se activa la política de reintentos; si el error persiste, revisa la fachada de destino y los logs de distribución.

## 10. Referencias

* Fuentes internas:

  * `mxp-sync/process/readme.md`
  * `mxp-feBuild/process/readme.md`
* Microservicio: `mxpv2.integration.sync`

## 11. Contacto

* **Equipo responsable:** Integraciones MAXPOINT
* **Canal de soporte:** `#mxp-integraciones` (Slack interno)
* **Escalamiento:** Arquitectura de datos → Liderazgo técnico
