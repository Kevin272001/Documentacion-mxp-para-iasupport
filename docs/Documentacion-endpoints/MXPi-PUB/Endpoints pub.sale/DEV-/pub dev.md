# Módulo de Publicación de Ventas (MXPI-PUB pub.sale DEV)

## 1. Propósito y alcance

El módulo **MXPI-PUB pub.sale DEV** permite la orquestación y publicación de datos de ventas procesados en el ecosistema MAXPOINT V2.0. Gestiona la integración, transformación y carga de información relevante para la operación de ventas, pagos y productos, asegurando la coherencia y disponibilidad de datos en las fachadas de destino.

## 2. Descripción general del sistema

- **Microservicio:** mxpv2.integration.pub.sale
- **Rol principal:** Publicar y distribuir datos de ventas procesados.
- **Función crítica:** Integrar procesos ETL (Extract, Transform, Load) para ventas, pagos y productos, garantizando la correcta entrega a las interfaces de fachada.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código | Descripción                                                                 |
|--------------------------|--------|-----------------------------------------------------------------------------|
| Facade-Cash-Closure      | F100   | Publica cierres de caja procesados.                                         |
| Facade-Hourly-Sale       | F100   | Publica ventas por hora procesadas.                                         |
| Facade-Sale-Payment-Method | F100 | Publica métodos de pago utilizados en ventas.                               |
| Facade-Sale-Payment-Method-For-Channels | F100 | Publica métodos de pago por canal de venta.                        |
| Facade-Sales-Product     | F100   | Publica ventas por producto procesadas.                                     |
| Transaction-SendSale     | TX012  | Envía transacciones de venta para impresión.                                |

### 3.2. Capa de transformación

| Componente | Código | Descripción                                                                 |
|------------|--------|-----------------------------------------------------------------------------|
| Transformación de cierre de caja | T083 | Procesa datos para cierre de caja.                        |
| Transformación de ventas por hora | T086 | Procesa datos para ventas por hora.                       |
| Transformación de métodos de pago | T084 | Procesa datos para métodos de pago.                       |
| Transformación de métodos de pago por canal | T087 | Procesa datos para métodos de pago por canal.      |
| Transformación de ventas por producto | T085 | Procesa datos para ventas por producto.                   |

## 4. Gestión de procesos

**Flujo estándar:**
1. **Ingesta:** Se reciben los datos ETL desde las fuentes configuradas.
2. **Validación:** Se verifica la estructura e integridad de los datos.
3. **Enrutamiento:** Los datos se envían al proceso de transformación correspondiente.
4. **Transformación:** Se procesan los datos según el tipo de publicación.
5. **Carga:** Los datos transformados se almacenan en la fachada de destino.

## 5. Puntos de integración

- **Entrada:** Datos extraídos de repositorios MongoDB y SQL Server.
- **Salida:** Publicación en colecciones MongoDB específicas para cada tipo de venta.
- **Rol central:** Mantener la coherencia y trazabilidad de los procesos ETL.

## 6. Patrones de sincronización

- **Recepción centralizada:** Todos los datos ETL llegan vía los endpoints de fachada.
- **Procesamiento de metadatos:** Cada transformación gestiona la información específica del proceso.
- **Distribución coordinada:** Los datos se envían a las colecciones de destino según el tipo de publicación.
- **Garantía de coherencia:** Se evita la duplicidad y se valida la integridad de los registros.

## 7. Escenarios de uso

### ✅ Caso exitoso
- Datos válidos → Recepción → Validación → Transformación → Carga en fachada → Respuesta exitosa.

### ⚠️ Posibles fallos
- **Error en validación:** Datos rechazados, log de error, revisar fuente.
- **Error en transformación:** No se carga, rollback y alerta.
- **Error en distribución:** Fachada no disponible, reintentos automáticos.

## 8. Endpoints documentados

### POST Facade-Cash-Closure

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body (ejemplo):**
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

---

### POST Facade-Hourly-Sale

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body (ejemplo):**
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

---

### POST Facade-Sale-Payment-Method

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body (ejemplo):**
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

---

### POST Facade-Sale-Payment-Method-For-Channels

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body (ejemplo):**
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

---

### POST Facade-Sales-Product

**URL:** `{{server_pub_sale}}/api/v1/facade`  
**Autenticación:** Bearer Token  
**Body (ejemplo):**
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

---

### POST Transaction-SendSale

**URL:** `{{server_pub_sale}}/api/v1/transaction/TX012`  
**Autenticación:** Bearer Token  
**Body (ejemplo):**
```json
{
  "cdn_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
  "restaurant_id": "24260579-faf8-763f-aac7-5e16faff96d1",
  "period_id": "0ac60718-148e-4453-a23c-c3642d2e38de"
}
```
**Descripción:** Envía la transacción de venta para impresión y genera alerta de éxito.

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