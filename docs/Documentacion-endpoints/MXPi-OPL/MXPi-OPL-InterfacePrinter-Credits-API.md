# Documentación Técnica - InterfacePrinter-credits (TX013)

## 1. Propósito y Alcance
### 1.1 Objetivo
Este documento proporciona las especificaciones detalladas para el endpoint de impresión de créditos dentro de MAXPOINT V2.0. El endpoint maneja la generación de documentos relacionados con créditos, incluyendo créditos a empleados, créditos externos, créditos de empresa y créditos de productos para operaciones de punto de venta.

### 1.2 Alcance
- Generación de documentos de crédito
- Soporte para diferentes tipos de créditos
- Integración con sistemas de punto de venta
- Gestión de documentos financieros

## 2. Descripción del Sistema
### 2.1 Visión General
Parte del microservicio `mxpv2.integration.opl`, esta API permite la impresión de varios documentos relacionados con créditos que son esenciales para el seguimiento financiero y el mantenimiento de registros en entornos minoristas y de restaurantes.

### 2.2 Características Clave
- Impresión de diferentes tipos de documentos de crédito
- Integración con sistemas de gestión de clientes
- Soporte para múltiples métodos de pago
- Generación de comprobantes detallados

## 3. Especificación Técnica
### 3.1 Endpoint
```
POST {{serverPrinter}}/api/v1/transaction/TX013
```

### 3.2 Headers
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

### 3.3 Cuerpo de la Petición (Ejemplo)
```json
{
    "type_printer": "CREDITO_EMPLEADO",
    "transaction": {
        "point_of_sale": {
            "cdn_id": "f2c13a02-3cb5-4781-9e10-d3f1519c51e2",
            "restaurant_id": "f2c13a02-3cb5-4781-9e10-d3f1519c51e2",
            "cash_register_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "cash_register_name": "EC-CAJA001",
            "user_seller_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "user_seller_name": "Akiles Baesta",
            "device_os_name": "ios",
            "source_app_name": "Maxpoint 2.0",
            "half_integration_id": 3,
            "half_integration_name": "Local",
            "user_admin": "Carlos Zambrano"
        },
        "company": {
            "name": "EMBUTSER S.A.",
            "identificationNumber": "1792351057001",
            "companyAddress": "COREA 126 Y AV AMAZONAS",
            "restaurantAddress": "PICHINCHA / QUITO / AV. AMAZONAS 6121 Y EL INCA"
        },
        "client": {
            "client_id": "030B9503-85CF-E511-80C6-000D3A3261F3",
            "name": "ESTIVEN",
            "last_name": "ARRIAGA",
            "phone": "+593986610329",
            "email": "mari_jhz@hotmail.com",
            "gov_id_type": "CI",
            "gov_id_number": "1206302802",
            "additional_info": {
                "birthdate": ""
            }
        },
        "detail": {
            "creation_date": "10/01/2025 10:41AM",
            "transaction": "BD02F000177034",
            "method_of_payment": "EMPLEADO",
            "total": 54.21
        }
    }
}
```

## 4. Descripción de Campos

### 4.1 Campos de la Solicitud
- **type_printer**: Tipo de documento de crédito (ej. "CREDITO_EMPLEADO", "CREDITO_EXTERNO", "CREDITO_EMPRESA", "CREDITO_PRODUCTO")
- **transaction**: Contenedor para los detalles de la transacción
  - **point_of_sale**: Información del punto de venta
    - **cdn_id**: Identificador de la franquicia
    - **restaurant_id**: Identificador del restaurante
    - **cash_register_id/name**: Identificación de la caja registradora
    - **user_seller_id/name**: Usuario que procesó la transacción
    - **device_os_name**: Sistema operativo del dispositivo
    - **source_app_name**: Aplicación fuente
    - **half_integration_id/name**: Detalles del canal de integración
    - **user_admin**: Nombre del administrador
  - **company**: Información de la empresa
    - **name**: Nombre de la empresa
    - **identificationNumber**: RUC de la empresa
    - **companyAddress**: Dirección de la empresa
    - **restaurantAddress**: Dirección del restaurante
  - **client**: Información del cliente
    - **client_id**: Identificador único del cliente
    - **name/last_name**: Nombre completo del cliente
    - **phone/email**: Información de contacto
    - **gov_id_type/number**: Detalles de identificación gubernamental
    - **additional_info**: Datos adicionales del cliente (ej. fecha de nacimiento)
  - **detail**: Detalles de la transacción de crédito
    - **creation_date**: Fecha y hora de creación
    - **transaction**: Número de referencia de la transacción
    - **method_of_payment**: Método de pago (ej. "EMPLEADO")
    - **total**: Monto total del crédito

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

### ¿Qué tipos de documentos de crédito se pueden generar?
El endpoint admite varios tipos de créditos incluyendo:
- CREDITO_EMPLEADO (Crédito a empleado)
- CREDITO_EXTERNO (Crédito externo)
- CREDITO_EMPRESA (Crédito de empresa)
- CREDITO_PRODUCTO (Crédito de producto)

### ¿Cuáles son los campos obligatorios para un documento de crédito?
Los campos obligatorios incluyen:
- type_printer
- transaction.point_of_sale (todos los subcampos)
- transaction.client (client_id, name, gov_id_type, gov_id_number)
- transaction.detail (creation_date, transaction, method_of_payment, total)

### ¿Qué debo hacer si recibo un error 400?
Verifique el mensaje de error en la respuesta. Los problemas comunes incluyen campos obligatorios faltantes, formatos de datos inválidos o problemas de configuración de plantilla en el servidor.

## 8. Referencias
- Documentación de la API MAXPOINT V2.0
- Estándares de Procesamiento de Créditos
- Directrices de Documentos Financieros

## 9. Información de Contacto
Para soporte o preguntas, por favor contacte a:
- **Correo Electrónico**: soporte@maxpoint.com
- **Teléfono**: +1-800-123-4567
- **Portal de Soporte**: https://support.maxpoint.com
