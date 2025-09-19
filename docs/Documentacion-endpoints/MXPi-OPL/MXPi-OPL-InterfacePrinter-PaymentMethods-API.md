# Documentación Técnica - InterfacePrinter-segregationOfPaymentMethods (TX008)

## 1. Propósito y Alcance
### 1.1 Objetivo
Este documento proporciona las especificaciones detalladas para el endpoint de impresión de segregación de métodos de pago dentro de MAXPOINT V2.0. El endpoint maneja la generación de informes de segregación de métodos de pago para transacciones de punto de venta.

### 1.2 Alcance
- Generación de informes de segregación de métodos de pago
- Soporte para diferentes tipos de métodos de pago
- Integración con sistemas de punto de venta
- Conciliación financiera detallada

## 2. Descripción del Sistema
### 2.1 Visión General
Parte del microservicio `mxpv2.integration.opl`, esta API permite la impresión de informes detallados de segregación de métodos de pago, incluyendo desgloses por diferentes tipos de pagos para propósitos de conciliación financiera.

### 2.2 Características Clave
- Impresión de informes de segregación de pagos
- Integración con sistemas de gestión de caja
- Soporte para múltiples métodos de pago
- Generación de informes detallados para conciliación

## 3. Especificación Técnica
### 3.1 Endpoint
```
POST {{serverPrinter}}/api/v1/transaction/TX008
```

### 3.2 Headers
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

### 3.3 Cuerpo de la Petición (Ejemplo)
```json
{
    "type_printer": "RETIROS_FORMASPAGO",
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
            "half_integration_name": "App",
            "user_admin": "Carlos Zambrano",
            "restaurant_name": "G008 - EL INCA",
            "franchise_name": "GUS"
        },
        "total_segregation_payment_methods": {
            "value": 351.00
        },
        "segregation_payment_methods": [
            {
                "payment_method": "EFECTIVO",
                "value": 140.00
            },
            {
                "description": "TARJEAS",
                "value": 210.00
            },
            {
                "description": "CUPONES",
                "value": 1.00
            }
        ]
    }
}
```

## 4. Descripción de Campos

### 4.1 Campos de la Solicitud
- **type_printer**: Tipo de trabajo de impresión (ej. "RETIROS_FORMASPAGO")
- **transaction**: Contenedor para los detalles de la transacción
  - **point_of_sale**: Información del punto de venta
    - **cdn_id**: Identificador único del punto de venta
    - **restaurant_id**: ID del restaurante
    - **cash_register_id/name**: Detalles de la caja registradora
    - **user_seller_id/name**: Usuario que inició la transacción
    - **device_os_name**: Sistema operativo del dispositivo (ej. "ios")
    - **source_app_name**: Nombre de la aplicación fuente
    - **half_integration_id/name**: Detalles de integración
    - **user_admin**: Nombre del administrador
    - **restaurant_name**: Nombre del restaurante (formato: "Código - Nombre")
    - **franchise_name**: Nombre de la franquicia
  - **total_segregation_payment_methods**: Resumen de la segregación de métodos de pago
    - **value**: Valor monetario total en todos los métodos de pago
  - **segregation_payment_methods**: Arreglo de detalles de métodos de pago
    - **payment_method/description**: Tipo de método de pago (ej. "EFECTIVO", "TARJEAS", "CUPONES")
    - **value**: Valor total para este método de pago

## 5. Respuestas

### 5.1 Respuesta Exitosa (200)
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

### 5.2 Respuesta con Error - Solicitud Incorrecta (400)
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

### 5.3 Respuesta con Error - Error del Servidor (500)
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

## 6. Patrones de Gestión
- **Autenticación**: Se requiere token Bearer
- **Manejo de Errores**: Códigos de estado HTTP estándar con mensajes descriptivos
- **Validación**: Validación del cuerpo de la solicitud para campos requeridos
- **Idempotencia**: Se recomienda implementar clave de idempotencia para reintentos

## 7. Preguntas Frecuentes

### ¿Cuál es el propósito del campo `type_printer`?
El campo `type_printer` especifica el tipo de informe de segregación de métodos de pago que se generará. Para este endpoint, debe establecerse como "RETIROS_FORMASPAGO".

### ¿Cómo deben estructurarse los métodos de pago en la solicitud?
Los métodos de pago deben proporcionarse en el arreglo `segregation_payment_methods`, donde cada elemento contiene el tipo de método de pago y su valor correspondiente. Puede usar `payment_method` o `description` para especificar el tipo de método de pago.

### ¿Qué debo hacer si recibo un error 400?
Verifique el mensaje de error en la respuesta. Los problemas comunes incluyen campos obligatorios faltantes, formatos de datos inválidos o problemas de configuración de plantilla en el servidor.

## 8. Referencias
- Documentación de la API MAXPOINT V2.0
- Estándares de Procesamiento de Pagos
- Directrices de Conciliación Financiera

## 9. Información de Contacto
Para soporte o preguntas, por favor contacte a:
- **Correo Electrónico**: soporte@maxpoint.com
- **Teléfono**: +1-800-123-4567
- **Portal de Soporte**: https://support.maxpoint.com
