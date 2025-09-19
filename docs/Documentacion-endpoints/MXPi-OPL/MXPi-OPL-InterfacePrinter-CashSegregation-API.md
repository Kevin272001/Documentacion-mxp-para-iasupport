# Documentación Técnica - InterfacePrinter-cashSegregation (TX007)

## 1. Propósito y Alcance
### 1.1 Objetivo
Este documento proporciona las especificaciones detalladas para el endpoint de impresión de segregación de efectivo dentro de MAXPOINT V2.0. El endpoint maneja la generación de informes de segregación de efectivo para transacciones de punto de venta.

### 1.2 Alcance
- Generación de informes detallados de segregación de efectivo
- Desglose por denominaciones
- Totales para gestión de efectivo

## 2. Descripción del Sistema
### 2.1 Visión General
Parte del microservicio `mxpv2.integration.opl`, esta API permite la impresión de informes detallados de segregación de efectivo, incluyendo desgloses por denominación y totales para propósitos de gestión de efectivo.

### 2.2 Características Clave
- Impresión de informes de segregación de efectivo
- Soporte para múltiples denominaciones
- Integración con sistemas de punto de venta
- Gestión de caja detallada

## 3. Especificación Técnica
### 3.1 Endpoint
```
POST {{serverPrinter}}/api/v1/transaction/TX007
```

### 3.2 Headers
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

### 3.3 Cuerpo de la Petición (Ejemplo)
```json
{
    "type_printer": "RETIRO_EFECTIVO",
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
        "total_cash_segregation": {
            "value": 351.00,
            "quantity": 18
        },
        "cash_segregation": [
            {
                "description": "Billete",
                "denomination": 20.00,
                "quantity": 7,
                "value": 140.00
            },
            {
                "description": "Billete",
                "denomination": 30.00,
                "quantity": 7,
                "value": 210.00
            },
            {
                "description": "Moneda",
                "denomination": 0.25,
                "quantity": 4,
                "value": 1.00
            }
        ]
    }
}
```

## 4. Descripción de Campos

### 4.1 Campos de la Solicitud
- **type_printer**: Tipo de trabajo de impresión (ej. "RETIRO_EFECTIVO" o "DESASIGNADO_CAJERO_DEN")
- **point_of_sale**: Información del punto de venta
  - **cdn_id**: Identificador único del punto de venta
  - **restaurant_id**: ID del restaurante
  - **cash_register_id/name**: Detalles de la caja registradora
  - **user_seller_id/name**: Usuario que inició la transacción
  - **device_os_name**: SO del dispositivo (ej. "ios")
  - **source_app_name**: Nombre de la aplicación fuente
  - **half_integration_id/name**: Detalles de integración
  - **user_admin**: Nombre del administrador
  - **restaurant_name**: Nombre del restaurante (formato: "Código - Nombre")
  - **franchise_name**: Nombre de la franquicia
- **total_cash_segregation**: Resumen de la segregación de efectivo
  - **value**: Valor monetario total
  - **quantity**: Número total de ítems/denominaciones
- **cash_segregation**: Arreglo de denominaciones de efectivo
  - **description**: Tipo de denominación (ej. "Billete", "Moneda")
  - **denomination**: Valor monetario de la denominación
  - **quantity**: Cantidad de ítems para esta denominación
  - **value**: Valor total para esta denominación

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
El campo `type_printer` especifica el tipo de informe de segregación de efectivo a generar. Los valores comunes incluyen "RETIRO_EFECTIVO" para retiros de efectivo y "DESASIGNADO_CAJERO_DEN" para la desasignación de cajero.

### ¿Cómo se deben estructurar las denominaciones en la solicitud?
Las denominaciones deben proporcionarse en el arreglo `cash_segregation`, donde cada elemento contiene los detalles de la denominación incluyendo descripción, valor, cantidad y valor total.

### ¿Qué debo hacer si recibo un error 400?
Verifique el mensaje de error en la respuesta. Los problemas comunes incluyen campos obligatorios faltantes, formatos de datos inválidos o problemas de configuración de plantilla en el servidor.

## 8. Referencias
- Documentación de la API MAXPOINT V2.0
- Estándares de Gestión de Efectivo
- Requisitos de Cumplimiento Fiscal

## 9. Información de Contacto
Para soporte o preguntas, por favor contacte a:
- **Correo Electrónico**: soporte@maxpoint.com
- **Teléfono**: +1-800-123-4567
- **Portal de Soporte**: https://support.maxpoint.com
