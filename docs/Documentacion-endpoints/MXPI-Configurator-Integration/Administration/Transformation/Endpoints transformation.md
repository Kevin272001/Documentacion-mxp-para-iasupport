# Módulo de Sincronización de Transformaciones (MXP-Sync Transformation)

## 1. Propósito y alcance

El módulo **MXP-Sync Transformation** es el sistema de orquestación de sincronización de transformaciones de datos dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo de datos procesados entre diferentes módulos, garantiza la coherencia del sistema y coordina los procesos ETL (Extract, Transform, Load). Recibe datos procesados de las tuberías de transformación y los distribuye a las interfaces de fachada.

ℹ️ Para más detalles de los módulos relacionados, consulta:
- MXP Init
- MXP FE Build
- Impresora MXP

## 2. Descripción general del sistema

- **Microservicio:** mxpv2.integration.sync
- **Rol principal:** Gestionar procesos de integración y sincronización de transformaciones.
- **Función crítica:** Asegurar que los datos transformados lleguen a las fachadas correctas dentro del flujo ETL.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                  | Código | Descripción                                                        |
|-----------------------------|--------|--------------------------------------------------------------------|
| Interfaz de recepción ETL   | F008   | Punto de entrada principal para datos ETL (extract, transform, load). |

🔑 El F008 actúa como gateway unificado para datos procesados, validando estructura e integridad.

### 3.2. Capa de transformación

| Componente                | Código | Descripción                                               |
|---------------------------|--------|-----------------------------------------------------------|
| Construir información     | T080   | Procesa metadatos de compilación, asegurando estructura y disponibilidad. |

## 4. Gestión de procesos

**Flujo de trabajo estándar:**
- Ingestión de datos → El F008 recibe los paquetes ETL.
- Validación → Se verifica estructura e integridad.
- Enrutamiento → Los datos son enviados al proceso correcto según el contenido.
- Procesamiento de compilación → La info de construcción se transforma con T080.
- Distribución → Los datos procesados se entregan a las fachadas de destino.

## 5. Puntos de integración

El sincronizador conecta todo el ecosistema MAXPOINT V2.0:
- **Entrada:** Recibe datos procesados desde MXP Init, MXP FE Build y MXP Printer.
- **Salida:** Distribuye datos validados y transformados a fachadas de destino.
- **Rol central:** Mantiene coherencia en procesos ETL inter-módulos.

## 6. Patrones de sincronización

- Recepción centralizada → Todos los ETL llegan vía F008.
- Procesamiento de metadatos → El módulo T080 gestiona info de compilación.
- Distribución coordinada → Los datos son enviados a fachadas correctas.
- Garantía de coherencia → Evita inconsistencias en la plataforma.

## 7. Escenarios de uso

### ✅ Caso exitoso
ETL válido → se recibe en F008 → validación → transformación en T080 → distribución correcta.

### ⚠️ Posibles fallos
- **Error en validación:**  
  Datos rechazados. Se genera log de error con detalle del paquete fallido.  
  Acción recomendada: verificar fuente.
- **Error en transformación (T080):**  
  Datos no pasan a distribución. Se activa rollback y alerta al sistema de monitoreo.
- **Error en distribución:**  
  Fachada de destino no disponible. El sincronizador reintenta el envío según política de reintentos (retry policy).

## 8. Endpoint documentado

### POST getAllPaginated

**URL:** `https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/transformation/getAllPaginated`  
**Body (ejemplo):**
```json
{
  "search": "string",
  "sort_order": 0,
  "sort_field": "string",
  "rows": 20,
  "first": 1
}
```
**Ejemplo de request:**
```bash
curl --location 'https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/transformation/getAllPaginated' \
--data '{ "search": "string", "sort_order": 0, "sort_field": "string", "rows": 20, "first": 1 }'
```
**Ejemplo de respuesta:**
```json
{
  "code": 20,
  "description": "Encontrado Correctamente",
  "data": {
    "total_rows": 100,
    "rows": [
      {
        "id": "98f556df-6edc-4e7c-8fad-b127ad2f3541",
        "code": "T001",
        "name": "Propagation Menu Plu",
        "description": "Transformation for the propagation of the products"
      },
      {
        "id": "98f556df-6edc-4e7c-8fad-b127ad2f3541",
        "code": "T002",
        "name": "Propagation Menu Menu",
        "description": "Transformation for the propagation of the menus"
      },
      {
        "id": "e28a77e1-c550-4efa-94e6-24566be52824",
        "code": "T003",
        "name": "Propagation Menu Suggested Questions",
        "description": "Transformation for the propagation of the suggested questions"
      },
      {
        "id": "0c139b10-9519-4d62-a28a-c4840339f28b",
        "code": "T004",
        "name": "Propagation Menu Up Selling",
        "description": "Transformation for the propagation of the up sellings"
      }
    ]
  }
}
```

## 9. Preguntas frecuentes (para IA Support)

- **¿Qué pasa si falla la validación en F008?**  
  El paquete se descarta, se registra en logs y se debe revisar la fuente.

- **¿Cómo sé que la info de compilación se procesó bien?**  
  Revisa el log de T080. Si hay éxito, se marca como build_metadata_ok.

- **¿El sincronizador almacena datos?**  
  No, solo orquesta flujos. La persistencia depende de otros módulos.

- **¿Qué debo hacer si la distribución falla?**  
  Se activa la política de reintentos, pero si el error persiste, revisa la fachada de destino y los logs de distribución.

## 10. Referencias

- mxp-sync/process/readme.md 
- mxp-feBuild/process/readme.md 
- Microservicio: mxpv2.integration.sync

## 11. Contacto

- **Equipo responsable:** Integraciones MAXPOINT
- **Canal de soporte:** #mxp-integraciones (Slack interno)
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico