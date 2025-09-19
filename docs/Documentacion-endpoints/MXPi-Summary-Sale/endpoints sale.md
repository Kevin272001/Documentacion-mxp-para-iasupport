# Módulo de Resumen de Ventas (MXPI-summary-sale)

## 1. Propósito y alcance
El microservicio **MXPI-summary-sale** forma parte del ecosistema **MAXPOINT V2.0** y se encarga de centralizar, validar y exponer información de **ventas resumidas** en distintos niveles de granularidad:

- Cierres de caja
- Ventas por hora
- Ventas por método de pago
- Ventas por producto
- Consolidación de todos los resúmenes de ventas

### Objetivos principales:
- Proporcionar reportes estructurados de ventas.
- Asegurar consistencia en formatos de fechas, montos y transacciones.
- Validar la información recibida antes de exponerla.
- Integrar resultados de diferentes orígenes de ventas en un único punto.

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- **MXP Init**  
- **MXP FE Build**  
- **Impresora MXP**  

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.summarysale`  
- **Rol principal:** Gestión de consultas y resúmenes de ventas.  
- **Función crítica:** Consolidar ventas desde distintos enfoques (tiempo, método de pago, producto, canal).  

---

## 3. Arquitectura y componentes

### 3.1 Capa de fachada
| Componente | Endpoint | Descripción |
|------------|----------|-------------|
| API Cash Closure | `/GetAllPaginatedCashClosure` | Retorna ventas agrupadas por cierre de caja. |
| API Hourly Sale | `/GetAllPaginatedHourlySale` | Entrega ventas distribuidas por intervalos horarios. |
| API Payment Method | `/GetAllPaginatedSalePaymentMethod` | Devuelve ventas agrupadas por métodos de pago. |
| API Sale Product | `/GetAllPaginatedSaleProduct` | Reporta ventas por producto con detalle de cantidades y montos. |
| API Summary Sale | `/GetAllPaginatedSummarySale` | Consolida todos los reportes en un único bloque estructurado. |

---

## 4. Gestión de procesos

### Flujo estándar
1. **Recepción** → El endpoint recibe el body con filtros (cdn_id, restaurante, fechas, paginación).  
2. **Validación** → Se verifica estructura, formato de fechas y valores.  
3. **Procesamiento** → Se estructuran y calculan los datos de ventas según el tipo de endpoint.  
4. **Respuesta** → El sistema retorna código HTTP 200 con objeto JSON validado.  

---

## 5. Endpoints documentados

### 5.1. GetAllPaginatedCashClosure
**Método:** `PUT`  
**URL:** `{{server_summary_sales}}/api/v1/SummarySale/GetAllPaginatedCashClosure`  

📥 **Request Body**
```json
{
  "cdn_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
  "restaurant_id": "",
  "start_date": "01/06/2025",
  "final_date": "01/06/2025",
  "sort_field": "string",
  "sort_order": 0,
  "rows": 0
}
```

📤 **Validaciones principales**
- `cdn_id`, `restaurant_id`, `start_date`, `final_date`, `sort_field`, `sort_order`, `rows`, `first` obligatorios.
- Fechas en formato **DD/MM/YYYY**.
- Respuesta debe incluir: `code`, `status`, `message`, `data.rows` con ventas (`gross_sales`, `net_sales`, etc).

---

### 5.2. GetAllPaginatedHourlySale
**Método:** `POST`  
**URL:** `{{server_summary_sales}}/api/v1/SummarySale/GetAllPaginatedHourlySale`  

📥 **Request Body**
```json
{
  "cdn_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
  "restaurant_id": "f5a6f1d1-0c76-41cb-9a5a-88d2ca7d4f8f",
  "start_date": "2025-02-25",
  "final_date": "2025-02-25",
  "sort_field": "string",
  "sort_order": 0,
  "rows": 0
}
```

📤 **Validaciones principales**
- Fechas en formato **YYYY-MM-DD**.
- Estructura de respuesta: `rows` con bloques horarios (`time`, `transactions`, `value`).

---

### 5.3. GetAllPaginatedSalePaymentMethod
**Método:** `POST`  
**URL:** `{{server_summary_sales}}/api/v1/SummarySale/GetAllPaginatedSalePaymentMethod`  

📥 **Request Body**
```json
{
  "cdn_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
  "restaurant_id": "",
  "start_date": "2025-02-24",
  "final_date": "2025-05-25",
  "sort_field": "string",
  "sort_order": 0,
  "rows": 0
}
```

📤 **Validaciones principales**
- Cada registro en `sales` debe tener: `cashier`, `payment_method`, `value`, `transactions`.

---

### 5.4. GetAllPaginatedSaleProduct
**Método:** `POST`  
**URL:** `{{server_summary_sales}}/api/v1/SummarySale/GetAllPaginatedSaleProduct`  

📥 **Request Body**
```json
{
  "cdn_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
  "restaurant_id": "f5a6f1d1-0c76-41cb-9a5a-88d2ca7d4f8f",
  "start_date": "2025-02-24",
  "final_date": "2025-02-25",
  "sort_field": "string",
  "sort_order": 0,
  "rows": 0
}
```

📤 **Validaciones principales**
- `sales_by_product` debe contener:  
  - Identificador del producto (`cdn_plu_id`).  
  - Medio de venta (`means_of_sale`).  
  - Bloque de ventas con cantidades, precios brutos/netos, IVA, descuentos y servicios.  

---

### 5.5. GetAllPaginatedSummarySale
**Método:** `POST`  
**URL:** `{{server_summary_sales}}/api/v1/SummarySale/GetAllPaginatedSummarySale`  

📥 **Request Body**
```json
{
  "cdn_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
  "restaurant_id": "e17d03da-b8b6-f424-febc-3a767b6401bb",
  "period_date": "01/06/2025",
  "sort_field": "string",
  "sort_order": 1
}
```

📤 **Validaciones principales**
Incluye validación de sub-bloques:
- **cash_closure** → Ventas consolidadas.  
- **hourly_sale** → Intervalos de tiempo.  
- **sale_payment_method** → Cajero + método de pago.  
- **sale_product** → Ventas por producto.  
- **sale_payment_method_for_channels** → Métodos de pago agrupados por canal.  

---

## 6. Puntos de integración
- **Entrada:** Recepción de filtros (cdn_id, restaurante, fechas).  
- **Salida:** Reportes de ventas estructurados en bloques.  
- **Rol central:** Proveer reportes consistentes y validados para el ecosistema.  

---

## 7. Escenarios de uso

✅ **Caso exitoso:** Request válido → Validación → Respuesta con datos de ventas → 200 OK.  
⚠️ **Fallos posibles:**  
- **Error de validación:** Campos faltantes → request rechazado.  
- **Formato de fecha inválido:** Se genera log de error.  
- **Datos vacíos:** Respuesta con `rows: []`.  

---

## 8. Preguntas frecuentes (IA Support)

**Q: ¿Qué pasa si falta un campo en el request?**  
A: El request es rechazado y se genera log de validación.  

**Q: ¿Qué formatos de fecha soporta el servicio?**  
A: Depende del endpoint: algunos usan **DD/MM/YYYY** y otros **YYYY-MM-DD**.  

**Q: ¿Qué hago si no hay registros en la respuesta?**  
A: Se valida que `rows` esté vacío. No es error, solo indica ausencia de ventas.  

**Q: ¿El microservicio almacena datos?**  
A: No, solo expone y valida. La persistencia está en otros módulos.  

---

## Referencias
- Fuentes internas:  
  - `mxp-summary-sale/process/readme.md`  
- **Microservicio:** `mxpv2.integration.summarysale`  
- **Contacto:** Equipo de Integraciones MAXPOINT  
- **Soporte:** #mxp-integraciones (Slack interno)  
