# Módulo de Flujos Internos (MXPI-InternalWorkflows)

## 1. Propósito y alcance
El módulo **MXPI-InternalWorkflows** orquesta los flujos internos de facturación, pagos con tarjeta y generación de vouchers dentro del ecosistema **MAXPOINT V2.0**.  
Su objetivo principal es centralizar y validar las transacciones críticas entre los módulos de facturación, pagos y printer service.  

- Asegura que la información de facturación sea enviada y validada correctamente.
- Gestiona la integración con el orquestador de pagos.
- Controla la generación de vouchers de impresión.  

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- MXP Init  
- MXP FE Build  
- MXP Printer  

---

## 2. Descripción general del sistema
**Microservicio:** `mxpv2.integration.internalworkflows`  
**Rol principal:** orquestar los procesos internos de facturación, pagos y vouchers.  
**Función crítica:** garantizar la coherencia y validez de las transacciones internas.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Facturación FBuilder | F101 | Endpoint para recepción y validación de datos de facturación. |
| Pago Tarjeta | F102 | Endpoint para validar credenciales y datos de transacción de pagos con tarjeta. |
| Printer Voucher | F103 | Endpoint que gestiona la validación de datos para la generación de vouchers. |

### 3.2. Capa de validación
- Validación de **tokens y credenciales** de acceso.  
- Validación de **datos estructurados** (POS, cliente, productos, pagos, etc.).  
- Validación de **endpoints externos** (`feBuilder_endpoint`, `url_domain`, `ares_endpoint`).  

---

## 4. Gestión de procesos

### Flujo estándar
1. Recepción de la solicitud en el endpoint correspondiente.  
2. Validación de credenciales/autenticación.  
3. Validación de datos estructurados.  
4. Enrutamiento y procesamiento de la transacción.  
5. Respuesta con logs de validación o error.  

---

## 5. Endpoints documentados

### 5.1. **POST** `/facturacion_fbuilder`
**Descripción:**  
Orquesta la validación de facturación, POS, cliente, productos y métodos de pago.  

**Request Body esperado:**  
```json
{
   "authorization":{
      "access_token":"<jwt>"
   },
   "extra_properties":{
      "feBuilder_endpoint":"<endpoint>"
   },
   "structured_data":{
      "print_data":{
         "transaction":{ ... }
      }
   }
}
```

**Validaciones principales:**  
- `authorization.access_token` → válido, no vacío, formato JWT.  
- `extra_properties.feBuilder_endpoint` → URL válida.  
- `transaction.point_of_sale` → contiene `cdn_id`, `restaurant_id`, `cash_register_id`, etc.  
- `transaction.client` → datos completos y correo válido.  
- `transaction.order.products` → lista no vacía con precios correctos.  
- `transaction.payments.payment_methods` → array válido, `transaction_status = APPROVED`.  

---

### 5.2. **POST** `/pago_tarjeta`
**Descripción:**  
Orquesta el flujo de pagos con tarjeta validando credenciales, propiedades extra y datos de transacción.  

**Request Body esperado:**  
```json
{
   "authorization":{
      "login_credentials":{
         "usuario":"usr_orquestator",
         "clave":"usr_orquestator"
      }
   },
   "extra_properties":{
      "url_domain":"<domain>"
   },
   "structured_data":{
      "orquestador_data":{ ... }
   }
}
```

**Validaciones principales:**  
- `authorization.login_credentials.usuario` y `clave` → obligatorios.  
- `extra_properties.url_domain` → URL válida.  
- `orquestador_data` contiene:  
  - Campos obligatorios: `tipo`, `tipoMensaje`, `tipoTransaccion`, `codigoAdquiriente`, `montoTotal`, `hora`, `fecha`, `mid`, `tid`, `cid`.  
  - Valores numéricos válidos en `montoTotal`, `montoIva`, `propinaServicio`, etc.  
  - `fecha` → formato ISO con milisegundos.  
  - `hora` → formato `HH:mm:ss`.  

---

### 5.3. **POST** `/printer_voucher`
**Descripción:**  
Gestiona la validación y estructura de los vouchers impresos.  

**Request Body esperado:**  
```json
{
   "authorization":{
      "login_credentials":{
         "user":"<email>",
         "password":"<password>"
      }
   },
   "extra_properties":{
      "ares_endpoint":"<url>",
      "url_domain":"<url>",
      "opl_endpoint":"<url>"
   },
   "structured_data":{
      "data_transaction":{
         "transaction":{ ... }
      }
   }
}
```

**Validaciones principales:**  
- `authorization.login_credentials.user` y `password` → obligatorios.  
- `extra_properties` debe contener `ares_endpoint`, `url_domain`, `opl_endpoint`.  
- `transaction.point_of_sale` → POS con datos completos.  
- `transaction.price` → campos `currency`, `total`, `discount`, `tax`.  
- `transaction.payer` → datos completos (`gov_id_type`, `gov_id_number`, `email` válido).  
- `transaction.result_payment` → estructura con `datos`, `codigo`, `mensaje`, `datosRespuesta`.  

---

## 6. Patrones de orquestación
- **Recepción centralizada** → endpoints F101, F102, F103.  
- **Validación estructural** de credenciales, POS, cliente y transacción.  
- **Procesamiento transaccional** → garantiza integridad de pagos y facturación.  
- **Distribución coordinada** → los datos se enrutan a módulos de impresión, pagos o facturación.  

---

## 7. Escenarios de uso

✅ **Caso exitoso**  
1. Cliente envía datos a `/facturacion_fbuilder`.  
2. Se validan credenciales, datos de POS y métodos de pago.  
3. Flujo continúa hacia facturación y emisión de voucher.  

⚠️ **Posibles fallos**  
- Error en credenciales → rechazo inmediato.  
- Error en validación de POS/cliente → transacción detenida.  
- Error en distribución (endpoints externos) → se activa retry policy.  

---

## 8. Preguntas frecuentes (para IA Support)

**Q:** ¿Qué pasa si el access_token es inválido?  
**A:** El endpoint rechaza la transacción y genera log de error.  

**Q:** ¿Cómo sé si la transacción fue aprobada?  
**A:** Revisa `transaction.payments.payment_methods[].transaction_status`.  

**Q:** ¿El microservicio almacena datos?  
**A:** No, solo valida y enruta información.  

**Q:** ¿Qué ocurre si falla la conexión al printer?  
**A:** Se activa la política de reintentos y se genera alerta de error.  

---

## 9. Referencias
Fuentes internas:  
- mxpi-internalworkflows/process/readme.md  
- mxp-feBuild/process/readme.md  
- mxp-printer/process/readme.md  

**Microservicio:** `mxpv2.integration.internalworkflows`  
**Contacto:** Equipo de Integraciones MAXPOINT  
**Canal de soporte:** `#mxpi-internalworkflows` (Slack interno)  
**Escalamiento:** Arquitectura de datos → Liderazgo técnico  
