# Módulo de Servicios (MXPI-Services)

## 1. Propósito y Alcance
El módulo **MXPI-Services** centraliza los procesos de integración de transacciones, impresión de documentos fiscales, vouchers, apertura de caja y soporte dentro del ecosistema **MAXPOINT V2.0**.  

Su objetivo principal es:
- Gestionar operaciones críticas de punto de venta.
- Asegurar la emisión de facturas electrónicas y vouchers.
- Coordinar impresiones locales y procesos de caja.
- Recibir incidencias y soportes técnicos.

ℹ️ Para más detalles de los módulos relacionados, consulta:
- MXP Init  
- MXP FE Build  
- MXP Printer  

---

## 2. Descripción General del Sistema
**Microservicio:** `mxpv2.integration.services`  
**Rol principal:** orquestar la integración de servicios en el punto de venta.  
**Función crítica:** garantizar que todas las transacciones (venta, impresión, soporte, apertura de caja) se ejecuten de forma consistente y validada.

---

## 3. Arquitectura y Componentes

### 3.1. Transacciones Principales
| Código | Endpoint | Descripción |
|--------|----------|-------------|
| TX003  | `/api/v1/transaction/TX003` | Generación de **Factura Electrónica** |
| TX004  | `/api/v1/transaction/TX004` | Impresión de **Facturas / Órdenes de Pedido** |
| TX005  | `/api/v1/transaction/TX005` | Impresión de **Vouchers (aprobado, no aprobado, anulación)** |
| TX012  | `/api/v1/transaction/TX012` | **Apertura de Caja** |
| LOGIN  | `/login` | Autenticación de usuarios y generación de JWT |
| SUPPORT | `/api/support/receiver` | Recepción de incidencias y soporte |

---

## 4. Gestión de Procesos

### Flujo Estándar
1. **Autenticación** → `/login` retorna un JWT para consumir el resto de servicios.  
2. **Emisión de factura** → `/transaction/TX003` recibe orden, cliente, productos, pagos y retorna acceso a documento electrónico.  
3. **Impresión de documentos** → `/transaction/TX004` y `/transaction/TX005` generan comprobantes locales.  
4. **Apertura de caja** → `/transaction/TX012` inicializa sesión de caja registradora.  
5. **Soporte técnico** → `/support/receiver` registra incidencias de impresión o facturación.  

---

## 5. Puntos de Integración

- **Entrada:**  
  - Aplicaciones móviles (Maxpoint 2.0).  
  - Dispositivos de punto de venta (cajas, impresoras).  

- **Salida:**  
  - Facturación electrónica (SRI Ecuador).  
  - Impresoras locales.  
  - Sistemas de soporte.  

---

## 6. Endpoints Documentados

### 6.1. Facturación Electrónica – TX003
**POST** `/api/v1/transaction/TX003`  
**Función:** genera facturas electrónicas validadas por SRI.  
**Request principal:** contiene cliente, productos, orden, pagos.  
**Response esperado:**  
```json
{
  "code": 200,
  "status": "success",
  "alert": "Integración completada exitosamente",
  "data": {
    "companyTaxData": { ... },
    "dataInvoice": { ... },
    "document_code": "01"
  }
}
```

---

### 6.2. Impresión de Documentos – TX004
**POST** `/api/v1/transaction/TX004`  
**Función:** imprime comprobantes locales (FACTURA, ORDEN_PEDIDO, CUPONES).  
**Validaciones clave:**
- `point_of_sale` completo.  
- Productos con precios en USD.  
- Métodos de pago aprobados.  

**Response esperado:**
```json
{
  "code": 200,
  "status": "success",
  "alert": "Impresión generada correctamente",
  "messages": ["Impresión de factura completada"]
}
```

---

### 6.3. Impresión de Vouchers – TX005
**POST** `/api/v1/transaction/TX005`  
**Función:** genera vouchers de aprobación, no aprobación o anulación.  
**Request:** incluye detalle de pago, productos, pagador y resultado de transacción.  
**Response esperado:**
```json
{
  "code": 200,
  "status": "success",
  "alert": "Impresión generada correctamente",
  "messages": ["Impresión de voucher completada"]
}
```

---

### 6.4. Apertura de Caja – TX012
**POST** `/api/v1/transaction/TX012`  
**Función:** inicializa la sesión de caja.  
**Validación clave:** `type_publisher` debe ser `"OPEN_CASH"`.  
**Response esperado:**
```json
{
  "code": 200,
  "status": "success",
  "alert": "Apertura de Caja generada correctamente",
  "message": ["Apertura de Cajón exitosa", "Proceso completado"]
}
```

---

### 6.5. Login
**POST** `/login`  
**Función:** genera **JWT** para autenticar servicios.  
**Request:**
```json
{
  "email": "user@mail.com",
  "password": "password"
}
```
**Response esperado:**
```json
{
  "data": { "token": "jwt-token..." },
  "msg": "Ok"
}
```

---

### 6.6. Soporte Técnico – Receiver
**POST** `/api/support/receiver`  
**Función:** registrar incidentes de facturación e impresión.  
**Request ejemplo:**
```json
{
  "clientApp": "opl.printer",
  "process": "FACTURA_PRINT",
  "referenceID": "212121-878787-455445-454",
  "priority": 1,
  "messages": [
    "es obligatorio la facturación",
    "la impresora no responde",
    "falta papel"
  ]
}
```
**Response esperado:**
```json
{
  "code": 202,
  "status": "success",
  "alert": "Integración recibida exitosamente",
  "messages": [
    "Integración completada exitosamente"
  ]
}
```

---

## 7. Escenarios de Uso

✅ **Caso exitoso:**  
- Autenticación → JWT válido.  
- Transacción → validación completa.  
- Impresión → generada correctamente.  

⚠️ **Posibles fallos:**  
- Error en validación de request.  
- Rechazo por facturación electrónica.  
- Impresora no disponible.  

---

## 8. Preguntas Frecuentes

**Q:** ¿Qué pasa si falla la validación en TX003?  
**A:** Se rechaza el request y se genera log.  

**Q:** ¿Cómo sé si la apertura de caja fue exitosa?  
**A:** El response debe incluir `"Apertura de Cajón exitosa"`.  

**Q:** ¿Qué debo hacer si no imprime un voucher?  
**A:** Revisar estado de la impresora y repetir envío.  

---

## 9. Referencias
- `mxpi-services/process/readme.md`  
- `mxp-printer/process/readme.md`  
- Microservicio: `mxpv2.integration.services`  
- Equipo responsable: **Integraciones MAXPOINT**  
- Canal de soporte: `#mxp-integraciones` (Slack interno)  
