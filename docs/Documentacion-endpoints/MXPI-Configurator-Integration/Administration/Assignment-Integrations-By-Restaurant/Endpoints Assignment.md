# Módulo de Sincronización de Integraciones por Restaurante (AssignmentIntegrationsByRestaurant)

## 1. Propósito y alcance

El módulo **AssignmentIntegrationsByRestaurant** es el sistema de orquestación de sincronización de asignaciones de integraciones por restaurante dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo de datos procesados entre diferentes módulos, garantizando la coherencia y coordinación de los procesos ETL (Extract, Transform, Load). Recibe datos procesados de las tuberías de transformación y los distribuye a las interfaces de fachada.

ℹ️ Para más detalles de los módulos relacionados, consulta:
- MXP Init
- MXP FE Build
- Impresora MXP

## 2. Descripción general del sistema

- **Microservicio:** mxpv2.integration.sync
- **Rol principal:** Gestionar procesos de integración y sincronización de asignaciones por restaurante.
- **Función crítica:** Asegurar que los datos transformados lleguen a las fachadas correctas dentro del flujo ETL.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                        | Código | Descripción                                                        |
|-----------------------------------|--------|--------------------------------------------------------------------|
| Interfaz de recepción ETL         | F008   | Punto de entrada principal para datos ETL (extract, transform, load). |

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

## 8. Endpoints documentados

### POST Facade-Cash-Closure

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body (ejemplo):**
```json
{
  "code": "F100",
  "withResponse": true,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "country": "ECU",
  "processes": { ... }
}
```

### POST getAllPaginated

**URL:** `https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/assignmentIntegrationsByRestaurant/getAllPaginated`  
**Body (ejemplo):**
```json
{
  "search": "string",
  "sort_order": 0,
  "sort_field": "string",
  "rows": 20,
  "first": 1,
  "activeOnly": true
}
```
**Ejemplo de request:**
```bash
curl --location 'https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/assignmentIntegrationsByRestaurant/getAllPaginated' \
--data '{ ... }'
```
**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "messages": "Data returned succesfully",
  "data": {
    "total_rows": 100,
    "rows": [
      {
        "id": "69503d8f-fa70-2196-f0a0-e3a85fa17aec",
        "name": "Asignación de documentos",
        "country": { ... },
        "company": { ... },
        "franchise": { ... },
        "restaurant": { ... },
        "integrator": [ ... ],
        "status": { ... }
      }
      // ...otros registros...
    ]
  }
}
```

### GET getById

**URL:** `https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/assignmentIntegrationsByRestaurant/getById?id={id}`  
**Ejemplo de request:**
```bash
curl --location 'https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/assignmentIntegrationsByRestaurant/getById?id=b5ad9754-0419-46e5-b556-7ab1ab8db3e1'
```
**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "messages": "Data returned succesfully",
  "data": {
    "id": "69503d8f-fa70-2196-f0a0-e3a85fa17aec",
    "name": "Asignación de documentos",
    "country": { ... },
    "company": { ... },
    "franchise": { ... },
    "restaurant": { ... },
    "integrator": [ ... ],
    "status": { ... }
  }
}
```

### POST Create

**URL:** `https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/assignmentIntegrationsByRestaurant`  
**Body (ejemplo):**
```json
{
  "name": "Asignación de documentos",
  "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa4",
  "country": { ... },
  "company": { ... },
  "franchise": { ... },
  "restaurant": { ... },
  "integrator": [ ... ],
  "status": "3fa85f64-5717-4562-b3fc-2c963f66afa4"
}
```
**Ejemplo de request:**
```bash
curl --location 'https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/integrations/create' \
--data '{ ... }'
```
**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "messages": "created succefully",
  "data": { ... }
}
```

### PUT Update

**URL:** `https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/assignmentIntegrationsByRestaurant?id={id}`  
**Body (ejemplo):**
```json
{
  "name": "Asignación de documentos",
  "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa4",
  "country": { ... },
  "company": { ... },
  "franchise": { ... },
  "restaurant": { ... },
  "integrator": [ ... ],
  "status": "3fa85f64-5717-4562-b3fc-2c963f66afa4"
}
```
**Ejemplo de request:**
```bash
curl --location --request PUT 'https://770f9b7c-4111-4a6b-9ace-447c99efdd8f.mock.pstmn.io/api/v1/assignmentIntegrationsByRestaurant?id=e47c3813-7daf-407f-bc02-544d57290f1b' \
--data '{ ... }'
```
**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "messages": "created succefully",
  "data": {
    "id": "1234569",
    "name": "Integracion de Forma de Pagos",
    "status": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "observations": "",
    "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa4",
    "process": [
      { "id": "b5ad9754-0419-46e5-b556-7ab1ab8db3e1" }
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