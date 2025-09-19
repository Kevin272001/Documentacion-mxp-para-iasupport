# Módulo MXP-POS (MAXPOINT V2.0)

## 1. Propósito y alcance

El módulo MXP-POS gestiona operaciones clave relacionadas con el cálculo de facturas, menús, configuración y conciliaciones en el ecosistema MAXPOINT V2.0.

**Funciones principales:**

- Calcular facturas con impuestos, descuentos y redondeos aplicables.  
- Gestionar y exponer menús con preguntas sugeridas.  
- Permitir registro de ventas (contratos POS).  
- Manejar configuraciones de franquicias, restaurantes y países.  
- Obtener tipos de conciliación para operaciones financieras.  

## 2. Descripción general del sistema

- **Microservicio:** `mxpv2.integration.pos`  
- **Rol principal:** Servir como punto de integración del POS con cálculos, menús y configuraciones.  
- **Función crítica:** Asegurar coherencia entre cálculos de facturación, menús y reglas de configuración.  

## 3. Arquitectura y componentes

| Componente                        | Código | Descripción                                                       |
|-----------------------------------|--------|-------------------------------------------------------------------|
| API de cálculo de factura         | P010   | Permite calcular facturas con descuentos, impuestos y redondeos.  |
| API de venta POS - contrato       | P011   | Punto para registrar contratos de venta POS.                      |
| API de configuración combinada    | P012   | Gestiona relación entre franquicias, restaurantes y configuraciones. |
| API de menús                      | P013   | Provee menús junto con preguntas sugeridas.                       |
| API de conciliación               | P014   | Devuelve tipos de conciliación financiera.                        |
| API de configuraciones por país   | P015   | Retorna configuraciones aplicables a un país específico.          |

## 4. Gestión de procesos

**Flujo estándar:**  

1. Solicitud POS → al endpoint correspondiente.  
2. Validación → parámetros o body de entrada.  
3. Procesamiento → el microservicio ejecuta reglas de cálculo, configuración o consulta.  
4. Respuesta → JSON estructurado con resultados para el POS.  

## 5. Puntos de integración

- **Entrada:** solicitudes desde terminales POS, restaurantes y módulos internos de franquicia.  
- **Salida:** cálculos de facturación, menús estructurados, configuraciones aplicables y conciliaciones.  

## 6. Endpoints documentados

> **Nota:** Reemplace las variables `{{server_pos}}`, `{{server_pos_back}}`, `{{micro_service_menu}}` y similares por las URLs/valores concretos de su entorno (UAT, QA, Prod).

### 6.1 CalculateInvoice

- **Método:** GET  
- **URL:** `{{server_pos}}/invoicecalculator/api/v1/calculator/calculate_invoice`  

**Body (raw):**

```json
{
  "default_franchise_tax": {
    "tax_name": "IVA 15 %",
    "value": 15
  },
  "discounts": [
    { "discount": 10, "is_percentage": true }
  ],
  "plus": [
    {
      "pvp": 300,
      "quantity": 1,
      "taxes": [
        { "tax_name": "IVA 15 %", "value": 15 },
        { "tax_name": "IVA 10 %", "value": 10 }
      ],
      "discounts": [
        { "discount": 10, "is_percentage": true }
      ]
    },
    {
      "pvp": 200,
      "quantity": 3,
      "taxes": [
        { "tax_name": "IVA 17 %", "value": 17 }
      ],
      "discounts": []
    }
  ],
  "decimal_quantity_presentation": 2,
  "decimal_quantity": 6
}
```

**Función:** Calcula totales de una factura aplicando impuestos, descuentos y políticas de redondeo.  

**Notas:**  
- Este endpoint no persiste la factura; devuelve únicamente el cálculo.  
- Revisar `decimal_quantity_presentation` y `decimal_quantity` para control de formatos.  

---

### 6.2 Venta de POS - Contrato

- **Método:** POST  
- **URL:** (definir URL final según entorno)  

**Función:** Permite registrar una venta de POS en formato contrato.  

**Estado de la documentación:** Body aún no definido en la documentación recibida.  
Recomendación: incluir modelo JSON con campos mínimos requeridos (`contract_id`, `items[]`, `payment_method`, `customer`, `timestamps`, `totals`) y ejemplos de errores esperados.  

---

### 6.3 postCombinedFranchiseAndConfig

- **Método:** POST  
- **URL:** `https://apim-uat-ec.azure-api.net/internalClients/api/v1/franchise/post_combined_franchise_and_config`  

**Body (raw):**

```json
{
  "franchises": [
    {
      "franchise_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
      "restaurants": [
        { "restaurant_id": "e17d03da-b8b6-f424-febc-3a767b6401bb" },
        { "restaurant_id": "35b71d9e-7e9c-8f4e-25c0-472cfbbff75c" }
      ]
    }
  ]
}
```

**Función:** Registrar franquicias con configuraciones asociadas y restaurantes vinculados.  

---

### 6.4 GetStartAllMenusPlusSuggestedQuestions

- **Método:** GET  
- **URL:** `{{server_pos_back}}{{micro_service_menu}}/api/v1/menu/get_start_all_menus_plus_suggested_questions`  

**Query Params:**  

- `franchise` (UUID) — ID de la franquicia.  
- `restaurant` (UUID) — ID del restaurante.  
- `category_id_plu` (UUID) — ID de categoría PLU.  
- `device` (UUID) — Dispositivo POS.  
- `menu_deep_level_nested` (Number) — Nivel de anidación del menú.  

**Función:** Obtener menús iniciales con preguntas sugeridas.  

**Ejemplo (curl):**

```bash
curl --location 'https://apim-uat-ec.azure-api.net/menus/api/v1/menu/get_start_all_menus_plus_suggested_questions?franchise=bc2d8ada-e25e-1bb7-55fe-32d1205ac4af&restaurant=e17d03da-b8b6-f424-febc-3a767b6401bb&category_id_plu=9bdeeb7f-136c-8f6a-489b-d9fc02db904a&device=8028a2ff-c639-6df4-68e8-8982c0256be3&menu_deep_level_nested=3'
```

**Ejemplo de Response (200 OK):**

```json
{
  "menus": [ { "id": "12345", "name": "Combo Familiar", "items": [...], "suggested_questions": [...] } ],
  "device": "8028a2ff-c639-...",
  "franchise": "bc2d8ada-...",
  "restaurant": "e17d03da-..."
}
```

---

### 6.5 getAllMenusPlusSuggestedQuestions

- **Método:** GET  
- **URL:** `{{server_pos_back}}{{micro_service_menu}}/api/v1/menu/get_all_menus_plus_suggested_questions`  

**Parámetros:** iguales al endpoint anterior (aplicados a todos los menús).  

**Función:** Obtener todos los menús disponibles más preguntas sugeridas.  

---

### 6.6 / 6.7 Versiones específicas por franquicia (CAJUN, ILCAPPO)

- **Método:** GET  
- **URL:** Variante de `getAllMenusPlusSuggestedQuestions` con parámetros predefinidos por franquicia.  

**Función:** Endpoints con filtros/valores predeterminados para franquicias específicas (ej. Cajun, Il Capo).  

---

### 6.8 get_all_conciliation_types

- **Método:** GET  
- **URL:** `https://apim-uat-ec.azure-api.net/purchaseorders/api/v1/conciliationtypes/get_all_conciliation_types`  

**Función:** Retorna todos los tipos de conciliación disponibles en el sistema de órdenes de compra.  

---

### 6.9 Configuration - getallbycountry / Get Country Configuration by ID

- **Método:** GET  
- **URL:** `https://apim-uat-ec.azure-api.net/configuration/api/v1/configurations/getallbycountry?country_id=<country_id>`  

**Parámetros:**  

- `country_id` (UUID)  

**Función:** Retorna configuraciones aplicables a un país específico.  

**Ejemplo alternativo:**

- **URL:** `https://apim-uat-ec.azure-api.net/internalClients/api/v1/country/get_country_configuration_by_id?country=<country_uuid>`  

**Ejemplo de Response (200 OK):**

```json
{
  "country_id": "d6cac83d-6ce5-f62e-2628-1e2e9ae7b51f",
  "name": "Ecuador",
  "currency": "USD",
  "tax_rate": 12,
  "time_zone": "GMT-5",
  "language": "es",
  "pos_configuration": {
    "max_discount": 15,
    "enable_tips": true,
    "rounding_policy": "nearest_cent"
  }
}
```

---

## 7. Escenarios de uso

- **Caso exitoso:** Solicitud correcta con parámetros válidos → Respuesta JSON con resultados esperados.  
- **Posibles fallos:**  
  - Parámetros inválidos → `400 Bad Request`.  
  - Body incompleto → `422 Unprocessable Entity` o `400` (según validación).  
  - Servicio no disponible → `5xx` (reintentos o circuit breaker recomendable).  

## 8. Preguntas frecuentes (IA Support)

- **Q:** ¿El cálculo de factura persiste los datos?  
  **A:** No, el endpoint de cálculo solo devuelve el cálculo; la persistencia depende de módulos de facturación.  

- **Q:** ¿Las configuraciones por país son dinámicas?  
  **A:** Sí, pueden cambiar según la administración central.  

- **Q:** ¿Qué pasa si un menú no tiene preguntas sugeridas?  
  **A:** El endpoint devuelve el menú sin preguntas adicionales.  

## 9. Referencias

- `mxp-pos/invoice/readme.md`  
- `mxp-pos/menu/readme.md`  
- API Gateway: `apim-uat-ec.azure-api.net`  

## 10. Contacto

- **Equipo responsable:** Integraciones MAXPOINT – POS  
- **Canal de soporte:** #mxp-pos (Slack interno)  
- **Escalamiento:** Arquitectura de POS → Liderazgo técnico  
