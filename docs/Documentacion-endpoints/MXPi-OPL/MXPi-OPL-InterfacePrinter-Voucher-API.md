# Documentación Técnica - InterfacePrinter-voucher (TX005)

## 1. Propósito y Alcance
### 1.1 Objetivo
Este documento proporciona las especificaciones detalladas para el endpoint de impresión de vouchers dentro de MAXPOINT V2.0. El endpoint maneja la generación de comprobantes de pago aprobados para transacciones.

### 1.2 Alcance
- Generación de comprobantes de pago
- Impresión de transacciones aprobadas
- Soporte para múltiples métodos de pago

## 2. Descripción del Sistema
### 2.1 Visión General
Parte del microservicio `mxpv2.integration.opl`, esta API permite la impresión de comprobantes de transacciones con información detallada de pago, incluyendo productos desglosados, impuestos y detalles de autorización de pago.

### 2.2 Características Clave
- Impresión de comprobantes detallados
- Soporte para transacciones con tarjeta
- Integración con sistemas de pago EMV
- Cálculo automático de impuestos

## 3. Especificación Técnica
### 3.1 Endpoint
```
POST {{serverPrinter}}/api/v1/transaction/TX005
```

### 3.2 Headers
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

### 3.3 Cuerpo de la Petición (Ejemplo)
```json
{
    "type_printer": "VOUCHER_APROBADO",
    "transaction": {
        "point_of_sale": {
            "cdn_id": "533d60dc-c672-d14c-b9a9-29801c8ae01a",
            "restaurant_id": "84226b3d-e13b-cc41-b262-d55f324239d0",
            "cash_register_id": "d7a253d7-d95f-0a45-9b99-f823219a79b2",
            "cash_register_name": "Caja 1",
            "user_seller_id": "668d3b43-ab48-4fe0-ab01-ea4f21107be8",
            "user_seller_name": "ANA GOMEZ",
            "device_os_name": "web",
            "source_app_name": "Maxpoint 2.0",
            "half_integration_id": 3,
            "half_integration_name": "App",
            "franchise_name": "KENTUCKY FRIED CHICKEN"
        },
        "price": {
            "currency": "USD",
            "total": 141.99,
            "subsidy": 0,
            "discount": 0,
            "tax": 11.99,
            "base_amount": 130,
            "base_amountWithOutTax": 0,
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
        "payer": {
            "gov_id_type": "Cedula",
            "gov_id_number": "1234567891",
            "name": "EDISON FIGUEROA",
            "phone": "992087877"
        },
        "order_detail": {
            "products": [
                {
                    "plu_id": "a3a5830e-0385-424a-9e8c-785b305679e7",
                    "cdn_plu_id": "Cajun - COMBO CHOP SUEY",
                    "plu_quantity": 1,
                    "price": {
                        "total_per_plu": {
                            "currency_code": "USD",
                            "subtotal_without_taxes": 5.65,
                            "discount_percentage": 0,
                            "discounts_value": 0,
                            "subtotal_include_discounts": 5.65,
                            "tax_value": 0.85,
                            "total": 6.5,
                            "taxes": [
                                {
                                    "name": "IVA 15%",
                                    "code": "iva_12",
                                    "subtotal_without_taxes": 5.65,
                                    "percentage": 15,
                                    "total": 0.85
                                }
                            ]
                        },
                        "total_price": {
                            "currency_code": "USD",
                            "subtotal_without_taxes": 5.65,
                            "discount_percentage": 0,
                            "discounts_value": 0,
                            "subtotal_include_discounts": 5.65,
                            "tax_value": 0.85,
                            "total": 6.5,
                            "taxes": [
                                {
                                    "name": "IVA 15%",
                                    "code": "iva_12",
                                    "subtotal_without_taxes": 5.65,
                                    "percentage": 15,
                                    "total": 0.85
                                }
                            ]
                        }
                    }
                }
            ]
        },
        "result_payment": {
            "error": false,
            "mensaje": "",
            "datosRespuesta": [
                {
                    "tipoMensaje": 0,
                    "tipoRespuesta": 0,
                    "codigoAdquiriente": 2,
                    "codigorespuestaAutorizador": "00",
                    "mensajeRespuesta": "00 AUTORIZACION OK. ",
                    "secuenciaTransaccion": "003635",
                    "numeroLote": "000046",
                    "horaTransaccion": "143441",
                    "fechaTransaccion": "20250611",
                    "numeroAutorizacion": "769974",
                    "terminalID": "KFC07001",
                    "merchantID": "000000832412   ",
                    "nombreGrupoTarjeta": "VISA ELECT/DEB           ",
                    "modoLectura": 7,
                    "nombreTarjetaHabiente": "PAYWAVE/VISA                         ",
                    "idemv": "VISA DEBITO         ",
                    "aidemv": "A0000000031010      ",
                    "tipoemv": "80                  ",
                    "arqc": "6C84F48EC0C9082A",
                    "tvr": "0000000000",
                    "tsi": "0000",
                    "numTarjTrunca": "41668300XXXX1426         ",
                    "fechVencTrj": "3002",
                    "numTarjEncri": "A1920759CE395D83A5AB2A5484ECAF325CB753F8EA687E32D3AC7109C968176D",
                    "proveedorTarjeta": "MEDIANET"
                }
            ]
        }
    }
}
```

## 4. Respuestas

### 4.1 Respuesta Exitosa (200)
```json
{
    "code": 200,
    "status": "success",
    "alert": "Impresión generada correctamente",
    "messages": [
        "Impresión de voucher completada"
    ]
}
```

### 4.2 Respuesta con Error - Solicitud Incorrecta (400)
```json
{
    "code": 400,
    "status": "failed",
    "alert": "Ocurrió un error al imprimir",
    "messages": [
        "Plantilla no cumple formato"
    ]
}
```

### 4.3 Respuesta con Error - Error del Servidor (500)
```json
{
    "code": 500,
    "status": "failed",
    "alert": "Ocurrió un error al imprimir",
    "messages": [
        "Se requiere un cuerpo de solicitud no vacío"
    ]
}
```

## 5. Patrones de Gestión
- **Autenticación**: Se requiere token Bearer
- **Manejo de Errores**: Códigos de estado HTTP estándar con mensajes descriptivos
- **Validación**: Validación del cuerpo de la solicitud para campos requeridos
- **Idempotencia**: Se recomienda implementar clave de idempotencia para reintentos

## 6. Preguntas Frecuentes

### ¿Cuál es el propósito del campo `type_printer`?
El campo `type_printer` especifica el tipo de comprobante a generar. Para este endpoint, siempre debe establecerse en `VOUCHER_APROBADO`.

### ¿Cómo se calculan y muestran los impuestos?
Los impuestos se calculan según el arreglo `taxes` en la solicitud, que incluye el nombre del impuesto, código, porcentaje y monto calculado. El sistema los mostrará de acuerdo con la plantilla configurada.

### ¿Qué métodos de pago son compatibles?
El endpoint admite varios métodos de pago como se indica en el objeto `result_payment.datosRespuesta`, incluyendo pagos con tarjeta de crédito/débito con datos de chip EMV.

## 7. Referencias
- Documentación de la API MAXPOINT V2.0
- Estándares de Procesamiento de Pagos
- Requisitos de Cumplimiento Fiscal

## 8. Información de Contacto
Para soporte o preguntas, por favor contacte a:
- **Correo Electrónico**: soporte@maxpoint.com
- **Teléfono**: +1-800-123-4567
- **Portal de Soporte**: https://support.maxpoint.com
