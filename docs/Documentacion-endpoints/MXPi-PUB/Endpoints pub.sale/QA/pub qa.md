# Módulo de Publicación de Ventas (MXPI-PUB pub.sale DEV)

## 1. Propósito y alcance

El módulo **MXPI-PUB pub.sale DEV** es responsable de la orquestación y publicación de datos de ventas procesados dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo de datos entre diferentes módulos, garantiza la coherencia del sistema y coordina los procesos ETL (Extract, Transform, Load). Recibe datos procesados de las tuberías de transformación y los distribuye a las interfaces de fachada (facades).

ℹ️ Para más detalles de los módulos relacionados, consulta:
- MXP Init
- MXP FE Build
- Impresora MXP

## 2. Descripción general del sistema

- **Microservicio:** mxpv2.integration.pub.sale
- **Rol principal:** Gestionar procesos de integración y publicación de ventas.
- **Función crítica:** Asegurar que los datos transformados lleguen a las fachadas correctas dentro del flujo ETL.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente | Código | Descripción |
|------------|--------|-------------|
| Facade-Cash-Closure | F100 | Publica cierres de caja procesados. |
| Facade-Hourly-Sale | F100 | Publica ventas por hora procesadas. |
| Facade-Sale-Payment-Method | F100 | Publica métodos de pago utilizados en ventas. |
| Facade-Sale-Payment-Method-For-Channels | F100 | Publica métodos de pago por canal de venta. |
| Facade-Sales-Product | F100 | Publica ventas por producto procesadas. |
| Transaction-SendSale | TX012 | Envía transacciones de venta para impresión. |

### 3.2. Capa de transformación

| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación de cierre de caja | T083 | Procesa datos para cierre de caja. |
| Transformación de ventas por hora | T086 | Procesa datos para ventas por hora. |
| Transformación de métodos de pago | T084 | Procesa datos para métodos de pago. |
| Transformación de métodos de pago por canal | T087 | Procesa datos para métodos de pago por canal. |
| Transformación de ventas por producto | T085 | Procesa datos para ventas por producto. |

## 4. Gestión de procesos

**Flujo de trabajo estándar:**
- Ingestión de datos → Recepción de paquetes ETL.
- Validación → Verificación de estructura e integridad.
- Enrutamiento → Envío al proceso correcto según el contenido.
- Transformación → Procesamiento según el tipo de publicación.
- Distribución → Entrega a las fachadas de destino.

## 5. Puntos de integración

El publicador conecta todo el ecosistema MAXPOINT V2.0:
- **Entrada:** Recibe datos procesados desde MXP Init, MXP FE Build y MXP Printer.
- **Salida:** Distribuye datos validados y transformados a fachadas de destino.
- **Rol central:** Mantiene coherencia en procesos ETL inter-módulos.

## 6. Patrones de sincronización

- Recepción centralizada → Todos los ETL llegan vía los endpoints de fachada.
- Procesamiento de metadatos → Cada transformación gestiona la información específica del proceso.
- Distribución coordinada → Los datos se envían a las colecciones de destino según el tipo de publicación.
- Garantía de coherencia → Evita inconsistencias en la plataforma.

## 7. Escenarios de uso

### ✅ Caso exitoso
ETL válido → Recepción → Validación → Transformación → Distribución correcta.

### ⚠️ Posibles fallos
- **Error en validación:** Datos rechazados, log de error, revisar fuente.
- **Error en transformación:** No se distribuyen, rollback y alerta.
- **Error en distribución:** Fachada de destino no disponible, reintentos automáticos.

## 8. Endpoints documentados

### POST Facade-Cash-Closure

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body:**
```json
{
  "code": "F100",
  "withResponse": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "processes": {
    "EXTRACTS": [ ... ],
    "TRANSFORMS": [ { "code": "T083", ... } ],
    "LOADS": [ ... ],
    "PUBLISHERS": []
  }
}
```
**Descripción:** Publica el cierre de caja procesado en la colección `Pub_Sale_Cash_Closure`.

**Ejemplo de respuesta:**
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

### POST Facade-Hourly-Sale

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body:**
```json
{
  "code": "F100",
  "withResponse": false,
  "syncId": "...",
  "processes": {
    "EXTRACTS": [ ... ],
    "TRANSFORMS": [ { "code": "T086", ... } ],
    "LOADS": [ ... ],
    "PUBLISHERS": []
  }
}
```
**Descripción:** Publica ventas por hora en la colección `Pub_Sale_Hourly_Sale`.

**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "status": "success",
  "process": "I001",
  "message": "Integración completada exitosamente",
  "syncId": "...",
  "data": { ... }
}
```

---

### POST Facade-Sale-Payment-Method

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body:**
```json
{
  "code": "F100",
  "withResponse": false,
  "syncId": "...",
  "processes": {
    "EXTRACTS": [ ... ],
    "TRANSFORMS": [ { "code": "T084", ... } ],
    "LOADS": [ ... ],
    "PUBLISHERS": []
  }
}
```
**Descripción:** Publica métodos de pago utilizados en ventas en la colección `Pub_Sale_Sale_Payment_Method`.

**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "status": "success",
  "process": "I001",
  "message": "Integración completada exitosamente",
  "syncId": "...",
  "data": { ... }
}
```

---

### POST Facade-Sale-Payment-Method-For-Channels

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body:**
```json
{
  "code": "F100",
  "withResponse": false,
  "syncId": "...",
  "processes": {
    "EXTRACTS": [ ... ],
    "TRANSFORMS": [ { "code": "T087", ... } ],
    "LOADS": [ ... ],
    "PUBLISHERS": []
  }
}
```
**Descripción:** Publica métodos de pago por canal en la colección `Pub_Sale_Sale_Payment_Method_For_Channels`.

**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "status": "success",
  "process": "I001",
  "message": "Integración completada exitosamente",
  "syncId": "...",
  "data": { ... }
}
```

---

### POST Facade-Sales-Product

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body:**
```json
{
  "code": "F100",
  "withResponse": false,
  "syncId": "...",
  "processes": {
    "EXTRACTS": [ ... ],
    "TRANSFORMS": [ { "code": "T085", ... } ],
    "LOADS": [ ... ],
    "PUBLISHERS": []
  }
}
```
**Descripción:** Publica ventas por producto en la colección `Pub_Sale_Sale_Product`.

**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "status": "success",
  "process": "I001",
  "message": "Integración completada exitosamente",
  "syncId": "...",
  "data": { ... }
}
```

---

### POST Transaction-SendSale

**URL:** `{{server_pub_sale}}/api/v1/transaction/TX012`  
**Autenticación:** Bearer Token  
**Body:**
```json
{
  "cdn_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
  "restaurant_id": "24260579-faf8-763f-aac7-5e16faff96d1",
  "period_id": "0ac60718-148e-4453-a23c-c3642d2e38de"
}
```
**Descripción:** Envía la transacción de venta para impresión y genera alerta de éxito.

**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "status": "success",
  "alert": "Impresión generada correctamente",
  "messages": [
    "Impresión de factura completada"
  ]
}
```

---

## 9. Preguntas frecuentes (para IA Support)

- **¿Qué pasa si falla la validación?**  
  El paquete se descarta y se registra en logs. Revisar la fuente de datos.

- **¿Cómo sé que la transformación fue exitosa?**  
  Revisar el log del proceso de transformación (ej. T083, T086, etc.).

- **¿El publicador almacena datos?**  
  No, solo orquesta flujos. La persistencia depende de los módulos de carga.

- **¿Qué hacer si la carga falla?**  
  Se activa la política de reintentos. Si persiste, revisar la fachada de destino y los logs.

## 10. Referencias

- mxp-sync/process/readme.md (1-28)
- mxp-feBuild/process/readme.md (9-10)
- Microservicio: mxpv2.integration.pub.sale

## 11. Contacto

- **Equipo responsable:** Integraciones MAXPOINT
- **Canal de soporte:** #mxp-integraciones (Slack interno)
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico