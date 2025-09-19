# Endpoints de OPL (MXPi-OPL) - Impresión de Documentos

## 1. Propósito y alcance

El endpoint InterfacePrinter-generic permite la generación e impresión de documentos fiscales y no fiscales dentro del ecosistema MAXPOINT V2.0.

* Generación de facturas electrónicas
* Impresión de órdenes de pedido
* Emisión de notas de crédito
* Soporte para diferentes tipos de impresoras fiscales

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Auth  
* MXP Server  
* MXP Orchestrator  

---

## 2. Descripción general del sistema

**Microservicio:** mxpv2.integration.opl  
**Rol principal:** gestión de impresión de documentos fiscales y no fiscales.  
**Función crítica:** generación segura y confiable de documentos legales y operativos.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente               | Código | Descripción                                                                 |
|--------------------------|--------|-----------------------------------------------------------------------------|
| Interfaz de impresión    | OPL-01 | Punto de entrada para solicitudes de impresión de documentos.               |

### 3.2. Capa de procesamiento

| Componente              | Código | Descripción                                                                 |
|-------------------------|--------|-----------------------------------------------------------------------------|
| Servicio de impresión   | OPL-S1 | Gestiona la lógica de generación y formato de documentos para impresión.    |

---

## 4. Gestión de procesos

Flujo de trabajo estándar:

1. **Solicitud de impresión:** Sistema envía POST con los datos del documento a imprimir.  
2. **Validación:** El sistema valida los campos obligatorios y formatos.  
3. **Procesamiento:** Se genera el documento en el formato requerido.  
4. **Impresión:** Se envía el documento a la impresora configurada.  
5. **Respuesta:** Se confirma la impresión o se devuelve mensaje de error.  

---

## 5. Puntos de integración

El módulo de impresión interactúa con:

* **Entrada:** datos de transacciones, clientes, productos y pagos.  
* **Salida:** documentos impresos y códigos de respuesta.  
* **Sistemas relacionados:** sistema de facturación electrónica, POS, impresoras fiscales.  

---

## 6. Patrones de gestión

* **Validación de datos:** todos los campos son validados antes del procesamiento.  
* **Manejo de plantillas:** soporte para diferentes formatos de documentos.  
* **Respuesta estandarizada:** formato consistente para éxito y errores.  

---

## 7. Escenarios de uso

✅ Caso exitoso:  
* Se envía una transacción válida → se valida → se genera el documento → se imprime → respuesta 201.  

⚠️ Posibles fallos:  
* **Datos incompletos o inválidos:** la solicitud será rechazada con código 400.  
* **Error en la impresora:** se devolverá un error indicando el fallo.  
* **Error en el servidor:** problemas al procesar la solicitud (código 500).  

---

## 8. Endpoints

### 8.1. InterfacePrinter-generic

**Method:** POST  

```
http://192.168.101.13:5002/api/v1/transaction/TX004
```

**Headers:**
```
Content-Type: application/json
Authorization: Bearer {{token}}
```

**Body:**
```json
{
    "type_printer": "FACTURA",
    "transaction": {
        "point_of_sale": {
            "cdn_id": "f2c13a02-3cb5-4781-9e10-d3f1519c51e2",
            "restaurant_id": "f2c13a02-3cb5-4781-9e10-d3f1519c51e2",
            "cash_register_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "cash_register_name": "EC-CAJA001",
            "user_seller_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "user_seller_name": "Daniel Llerena",
            "device_os_name": "ios",
            "source_app_name": "Maxpoint 2.0",
            "half_integration_id": 3,
            "half_integration_name": "Local"
        },
        "transaction_date": "YYYY-MM-DDTHH:MM:SS",
        "client": {
            "client_id": "030B9503-85CF-E511-80C6-000D3A3261F3",
            "name": "María José",
            "last_name": "Hernández Zurita",
            "phone": "+593986610329",
            "email": "mari_jhz@hotmail.com",
            "gov_id_type": "CI",
            "gov_id_number": "0929691038",
            "external_id": "030B9503-85CF-E511-80C6-000D3A3261F3",
            "additional_info": {
                "birthdate": "",
                "disclaimer":"Las facturas electrónicas ó físicas (una vez trasmitidas al SRI) emitidas a consumidor final no podrán ser anuladas ni modificadas mediante notas de crédito"
            }
        },
        "order": {
            "order_id": "0001292432-010103",
            "createdAt": "2024-09-11 12:14:40",
            "selected_shipping_method": "pickup",
            "accumulate_points": false,
            "redeem_points": false,
            "stock": false,
            "discount": false,
            "order_comment": "",
            "annulation_comment": "",
            "products": [
                {
                    "plu_id": "401",
                    "plu_name": "COMBO IDEAL",
                    "plu_quantity": 1,
                    "plu_comment": "",
                    "price": {
                        "total_per_plu": {
                            "currency_code": "USD",
                            "subtotal_without_taxes": 5.65,
                            "discount_percentage": 0,
                            "discounts_value": 0,
                            "subtotal_include_discounts": 5.65,
                            "tax_value": 0.85,
                            "total": 6.5,
                            "total_before_sale": 6.5,
                            "suggested_price": 0,
                            "taxes": [
                                {
                                    "tax_name": "IVA 10 %",
                                    "tax_code": "iva23313",
                                    "tax_percentage": 10,
                                    "total_tax_vulue": 4.99,
                                    "base_amount": 130
                                }
                            ]
                        },
                        "total_price": {
                            "currency_code": "USD",
                            "subtotal_without_taxes": 5.65,
                            "discount_percentage": null,
                            "discounts_value": 0,
                            "subtotal_include_discounts": 5.65,
                            "tax_value": 0.85,
                            "total": 6.5,
                            "total_before_sale": 6.5,
                            "suggested_price": 0,
                            "taxes": [
                                {
                                    "tax_name": "IVA 10 %",
                                    "tax_code": "iva23313",
                                    "tax_percentage": 10,
                                    "total_tax_vulue": 4.99,
                                    "base_amount": 130
                                }
                            ]
                        }
                    }
                }
            ]
        },
        "payments": {
            "totals": [
                {
                    "currency_code": "USD",
                    "subtotal_without_taxes": 5.65,
                    "discount_percentage": 0,
                    "discounts_value": 0,
                    "subtotal_include_discounts": 5.65,
                    "tax_value": 0.85,
                    "total_before_sale": 6.5,
                    "total": 6.5,
                    "taxes": [
                        {
                            "tax_name": "IVA 10 %",
                            "tax_code": "iva23313",
                            "tax_percentage": 10,
                            "total_tax_vulue": 4.99,
                            "base_amount": 130
                        }
                    ]
                }
            ],
            "payment_methods": [
                {
                    "payment_methods_id": "D20A9503-85CF-E511-80C6-000D3A3261F3",
                    "processor": "DEUNA",
                    "currency_code": "USD",
                    "payment_method_code": "93",
                    "transaction_type": "credit",
                    "transaction_id": "",
                    "transaction_status": "APPROVED"
                }
            ]
        }
    },
    "feBuilder": {
        "companyTaxData": {
            "name": "INT FOOD SERVICES CORP SA",
            "identificationNumber": "1234567890001",
            "specialTaxpayer": "GRAN CONTRIBUYENTE",
            "isSpecialTaxpayer": "Sí",
            "specialTaxpayerCode": "RESOL. N°: NAC-GCFOIOC21-00000900-E",
            "typeTax": "3243fdw23",
            "obligatoryToKeepAccounts": "Sí",
            "resolutionCode": "155",
            "companyAddress": "Av. Amazonas y Naciones Unidas, Quito, Ecuador",
            "restaurantAddress": "PICHINCHA / QUITO / AV. AMAZONAS 6121 Y EL INCA"
        },
        "dataInvoice": {
            "sequential": "050-070-000470031",
            "accessKey": "091020240117914151320011024050000470020412615331",
            "legend": "Estimado cliente: Por favor verifique los datos de su factura, únicamente se aceptarán cambios el mismo día de emisión.",
            "linkLabel": "Para obtener su factura electrónica ingrese a:",
            "urlDocument": "https://facturacion.kfc.ec",
            "accessKeyLabel": "(Usuario: CI/RUC, Clave: CI/RUC) o a la página web del SRI con la Clave de Acceso:",
            "createdAt": "2024-09-11 12:14:40"
        }
    }
}
```

**Response examples:**  

<details>
<summary>201 Created</summary>

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
</details>

<details>
<summary>400 Bad Request</summary>

```json
{
    "code": 400,
    "status": "failed",
    "alert": "Ocurrio un error al imprimir",
    "messages": [
        "Plantilla no cumple formato"
    ]
}
```
</details>

<details>
<summary>500 Internal Server Error</summary>

```json
{
    "code": 400,
    "status": "failed",
    "alert": "Ocurrio un error al imprimir",
    "messages": [
        "A non-empty request body is required."
    ]
}
```
</details>

---

## 9. Preguntas frecuentes (para IA Support)

**Q: ¿Qué tipos de documentos se pueden imprimir?**  
A: Se pueden imprimir facturas, órdenes de pedido, notas de crédito y otros documentos configurados en el sistema.

**Q: ¿Qué formato debe tener la fecha de transacción?**  
A: La fecha debe estar en formato ISO 8601 (ej: "2024-09-11T12:14:40").

**Q: ¿Cómo manejar los impuestos en los productos?**  
A: Cada producto puede tener múltiples impuestos configurados en el arreglo "taxes" con su respectivo porcentaje y valor.

**Q: ¿Qué hacer si la impresora no responde?**  
A: Verificar la conexión de la impresora, reiniciar el servicio de impresión y validar los logs del sistema.

**Q: ¿Cómo configurar una nueva plantilla de impresión?**  
A: Las plantillas se configuran en el sistema administrativo y deben seguir el formato JSON especificado.

---

## 10. Referencias

Fuentes internas:  

* mxp-opl/process/printing.md  
* Microservicio: mxpv2.integration.opl  

---

## 11. Contacto

**Equipo responsable:** Integraciones MAXPOINT  
**Canal de soporte:** #mxp-integraciones (Slack interno)  
**Escalamiento:** Equipo de Operaciones → Liderazgo técnico
