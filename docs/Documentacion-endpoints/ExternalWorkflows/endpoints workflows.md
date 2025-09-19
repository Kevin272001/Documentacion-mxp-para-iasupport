# Módulo de Workflower (MXPI-workflower)

## 1. Propósito y alcance
**MXPI Workflower** es el sistema encargado de orquestar flujos internos relacionados con impresión, control de dispositivos y orquestación de pagos dentro del ecosistema **MAXPOINT V2.0**.

- Gestiona la integración entre dispositivos físicos (impresoras, cajones de dinero, terminales de pago) y los procesos digitales.
- Asegura la validación estructural de datos antes de ejecutar procesos críticos.
- Provee respuestas estandarizadas que garantizan trazabilidad y consistencia en las operaciones.

ℹ️ Para más detalles de los módulos relacionados, consulta:
- MXPI Internal Workflows
- MXPI Summary Sale
- MXP-Sync

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.workflower`
- **Rol principal:** Orquestar procesos de impresión, apertura de cajones y ejecución de pagos electrónicos.
- **Función crítica:** Garantizar que las solicitudes a dispositivos físicos sean validadas, procesadas y respondidas correctamente.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de impresión | WF001 | Recepción de solicitudes de impresión de órdenes y facturas. |
| Interfaz de dispositivos | WF002 | Orquesta la apertura de cajones y comunicación con periféricos. |
| Interfaz de pagos | WF003 | Gestiona la comunicación con terminales de pago y autorizadores. |

### 3.2. Capa de validación
- Validación de campos requeridos en `extra_properties`.
- Validación estructural de datos de clientes, productos y métodos de pago.
- Validación de punto de venta y metadatos de transacción.

---

## 4. Gestión de procesos

### Flujo de trabajo estándar
1. **Recepción de datos** → Se recibe el request con datos de impresión, dispositivo o pago.  
2. **Validación** → Se verifica la estructura y obligatoriedad de los campos.  
3. **Procesamiento** → Se ejecuta la acción correspondiente (impresión, apertura de cajón, solicitud de pago).  
4. **Respuesta** → Se retorna un objeto estandarizado con código, descripción y mensajes de confirmación.  

---

## 5. Endpoints documentados

### 5.1. Endpoint: `PUT /generic_printer_external`
**Descripción:** Procesa la impresión de órdenes y facturas.  
**Request Body:** Debe incluir `authorization`, `extra_properties`, `structured_data`.  

**Validaciones clave:**
- `extra_properties` → contiene `cdn_id`, `restaurant_id`, `cash_register_suscription_id`, `event_code = print_order_and_invoice`.
- Cliente debe incluir: `client_id`, `name`, `last_name`, `phone`, `email`, `gov_id_type`, `gov_id_number`, `external_id`.
- Productos deben incluir: `plu_id`, `plu_name`, `plu_quantity`, `price`.
- Métodos de pago deben incluir: `processor`, `transaction_status = APPROVED`, `total_bill > 0`.

**Response esperado:**
```json
{
  "code": 200,
  "description": "Impresión generada correctamente",
  "data": {
    "code_transaction": 200,
    "messages": ["Impresión de factura completada"],
    "data_transaction": {
      "dataInvoice": {
        "sequential": "12345",
        "accessKey": "ABCDEF123456",
        "transaction": "...",
        "createdAt": "2025-09-12T00:00:00Z"
      }
    }
  }
}
```

---

### 5.2. Endpoint: `PUT /pub_device_publisher`
**Descripción:** Gestiona la apertura de cajones de dinero en el punto de venta.  
**Request Body:** Debe incluir `authorization`, `extra_properties`, `structured_data.transaction.point_of_sale`.

**Validaciones clave:**
- `extra_properties.event_code = drawer_opening`.
- `point_of_sale` debe contener: `cdn_id`, `restaurant_id`, `cash_register_id`, `cash_register_name`, `user_seller_id`, `device_os_name`, `source_app_name`, `half_integration_id`.

**Response esperado:**
```json
{
  "code": 200,
  "description": "Apertura de Caja generada correctamente",
  "data": {
    "messages": [
      "Apertura de Cajón exitosa",
      "Proceso completado"
    ]
  }
}
```

---

### 5.3. Endpoint: `POST /payments_orchestrator`
**Descripción:** Orquesta el proceso de pagos con tarjeta.  
**Request Body:** Debe incluir `authorization`, `extra_properties`, `structured_data.transaction_data`.  

**Validaciones clave:**
- `extra_properties.event_code = payment_request_card`.
- `extra_properties.payment_method` debe ser un UUID válido.
- `metadata.transaction_execute = 1`, `metadata.difered_code = normal`.
- `point_of_sale` debe contener datos clave (`cdn_id`, `restaurant_id`, `cash_register_id`, `cash_register_name`, `user_seller_id`).
- Productos en `order_detail.products` deben incluir: `plu_id`, `cdn_plu_id`, `plu_quantity`, `price` con `currency_code = USD`.

**Response esperado:**
```json
{
  "code": 200,
  "description": "Transacción realizada exitosamente",
  "data": {
    "data_transaction": {
      "tipoMensaje": "0200",
      "tipoRespuesta": "00",
      "codigoAdquiriente": "123",
      "codigorespuestaAutorizador": "00",
      "mensajeRespuesta": "authorization OK",
      "numeroAutorizacion": "999888777",
      "merchantID": "MID12345",
      "terminalID": "TID67890"
    }
  }
}
```

---

## 6. Escenarios de uso

✅ **Caso exitoso**  
- Se recibe request válido → validación exitosa → acción ejecutada (impresión, apertura, pago) → respuesta confirmada.

⚠️ **Posibles fallos**  
- Error en validación: request rechazado, se genera log de error.  
- Error en ejecución: acción no completada, rollback y alerta.  
- Error en respuesta externa: reintento según política definida.  

---

## 7. Preguntas frecuentes (para IA Support)
- **Q:** ¿Qué pasa si falla la validación de `extra_properties`?  
  **A:** El request se rechaza y se loggea el error.  

- **Q:** ¿Qué sucede si el cajón no responde?  
  **A:** Se marca error en dispositivo, se notifica al sistema de monitoreo.  

- **Q:** ¿El módulo almacena datos?  
  **A:** No, solo orquesta flujos. La persistencia depende de otros microservicios.  

- **Q:** ¿Cómo sé que el pago fue exitoso?  
  **A:** Revisar `codigorespuestaAutorizador = 00` y `mensajeRespuesta = authorization OK`.  

---

## 8. Referencias
- Fuentes internas: `mxpi-workflower/docs/readme.md`
- Microservicio: `mxpv2.integration.workflower`
- Equipo responsable: **Integraciones MAXPOINT**
- Canal de soporte: **#mxp-integraciones** (Slack interno)
- Escalamiento: **Arquitectura de datos → Liderazgo técnico**
