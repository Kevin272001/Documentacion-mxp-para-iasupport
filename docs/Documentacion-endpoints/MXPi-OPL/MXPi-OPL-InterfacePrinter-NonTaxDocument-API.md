# Documentación Técnica - InterfacePrinter-nonTaxDocument (TX010)

## 1. Propósito y Alcance
### 1.1 Objetivo
Este documento proporciona las especificaciones detalladas para el endpoint de impresión de documentos no tributarios dentro de MAXPOINT V2.0. El endpoint maneja la generación de documentos no tributarios como comprobantes de caja chica y documentos de cuentas por cobrar para operaciones de punto de venta.

### 1.2 Alcance
- Generación de documentos no tributarios
- Soporte para diferentes tipos de documentos internos
- Integración con sistemas de punto de venta
- Gestión de documentos financieros internos

## 2. Descripción del Sistema
### 2.1 Visión General
Parte del microservicio `mxpv2.integration.opl`, esta API permite la impresión de varios documentos no tributarios que son esenciales para el seguimiento financiero interno y el mantenimiento de registros en entornos minoristas y de restaurantes.

### 2.2 Características Clave
- Impresión de diferentes tipos de documentos no tributarios
- Integración con sistemas de gestión de caja
- Soporte para múltiples tipos de comprobantes internos
- Generación de documentos detallados

## 3. Especificación Técnica
### 3.1 Endpoint
```
POST {{serverPrinter}}/api/v1/transaction/TX010
```

### 3.2 Headers
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

### 3.3 Cuerpo de la Petición (Ejemplo)
```json
{
    "type_printer": "CAJA_CHICA",
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
        "non_tax_document": {
            "total": 51.3500
        }
    }
}
```

## 4. Descripción de Campos

### 4.1 Campos de la Solicitud
- **type_printer**: Tipo de documento no tributario (ej. "CAJA_CHICA" para caja chica, "FALTANTE_CXC" para cuentas por cobrar)
- **transaction**: Contenedor para los detalles de la transacción
  - **point_of_sale**: Información del punto de venta
    - **cdn_id**: Identificador único de la franquicia
    - **restaurant_id**: Identificador único del restaurante
    - **cash_register_id/name**: Identificación de la caja registradora
    - **user_seller_id/name**: Usuario que procesó la transacción
    - **device_os_name**: Sistema operativo del dispositivo (ej. "ios")
    - **source_app_name**: Aplicación fuente (ej. "Maxpoint 2.0")
    - **half_integration_id/name**: Detalles del canal de integración
    - **user_admin**: Nombre del administrador
    - **restaurant_name**: Nombre del restaurante (formato: "Código - Nombre")
    - **franchise_name**: Nombre de la franquicia
  - **non_tax_document**: Detalles del documento no tributario
    - **total**: Monto total del documento

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

### ¿Qué tipos de documentos no tributarios se pueden generar?
El endpoint admite varios tipos de documentos incluyendo "CAJA_CHICA" para caja chica y "FALTANTE_CXC" para cuentas por cobrar. Los tipos disponibles pueden extenderse según los requisitos del negocio.

### ¿Cómo se debe formatear el campo `total`?
El campo `total` debe ser un número decimal con hasta 4 decimales, que representa el monto total para el documento no tributario.

### ¿Qué debo hacer si recibo un error 400?
Verifique el mensaje de error en la respuesta. Los problemas comunes incluyen campos obligatorios faltantes, formatos de datos inválidos o problemas de configuración de plantilla en el servidor.

## 8. Referencias
- Documentación de la API MAXPOINT V2.0
- Estándares de Documentos Financieros
- Directrices de Documentos No Tributarios

## 9. Información de Contacto
Para soporte o preguntas, por favor contacte a:
- **Correo Electrónico**: soporte@maxpoint.com
- **Teléfono**: +1-800-123-4567
- **Portal de Soporte**: https://support.maxpoint.com
