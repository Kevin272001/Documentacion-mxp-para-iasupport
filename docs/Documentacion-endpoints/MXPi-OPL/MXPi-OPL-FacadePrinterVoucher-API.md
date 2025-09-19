# MAXPOINT V2.0 - API de Facade-Printer-Voucher

## 📌 Endpoint: Facade-Printer-Voucher

### 🔹 Descripción General
Este endpoint se encarga de la impresión de diversos tipos de documentos, con especial enfoque en notas de crédito ("NOTA_CREDITO") y otros documentos transaccionales. Soporta la generación de documentos complejos con información detallada de transacciones, detalles de procesamiento de pagos y configuraciones flexibles de impresión.

### 🔹 URL Base
```
{{serverPrinter}}/api/v1/facade
```

### 🔹 Método
`POST`

### 🔹 Autenticación
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

## 🔹 Cuerpo de la Solicitud

### Estructura de la Solicitud
```json
{
    "code": "8488484",
    "withResponse": true,
    "chunkLoad": 100,
    "syncId": "",
    "country": "ECU",
    "franchise_id": 0,
    "processes": {
        "TRANSACTIONAL": [
            {
                "code": "P0036",
                "withResponse": true,
                "withCache": false,
                "syncId": "00000000-0000-0000-0000-000000000000",
                "configuration": {
                    "connection": {
                        "server": "http://130.213.200.45:5002",
                        "port": "5002",
                        "user": "admin.system.mxpv2@kfc.com.ec",
                        "password": "********",
                        "repository": "/api/v1/transaction/TX004",
                        "adapter": "Rest"
                    },
                    "entities": []
                },
                "dataInput": {
                    "type_printer": "NOTA_CREDITO"
                    // ... resto de la configuración
                }
            }
        ]
        // ... resto de los procesos
    }
}
```

### Campos Principales de la Solicitud

#### Parámetros Principales
- `code`: Identificador único del trabajo de impresión
- `withResponse`: Booleano que indica si se requiere una respuesta
- `chunkLoad`: Número de ítems a procesar en cada lote
- `syncId`: ID de sincronización para seguimiento
- `country`: Código de país (ej: "ECU" para Ecuador)
- `franchise_id`: ID de la franquicia

#### Detalles de la Transacción
- `type_printer`: Tipo de documento a imprimir (ej: "NOTA_CREDITO")
- `transaction`: Contiene los detalles completos de la transacción

## 🔹 Ejemplos de Respuesta

### Respuesta Exitosa (201)
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

### Respuesta de Error (400)
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

### Respuesta de Error (500)
```json
{
    "code": 400,
    "status": "failed",
    "alert": "Ocurrió un error al imprimir",
    "messages": [
        "Se requiere un cuerpo de solicitud no vacío"
    ]
}
```

## 🔹 Descripción de Campos

### Objeto Transaction
- `point_of_sale`: Información del punto de venta
  - `cdn_id`: Identificador de la franquicia
  - `restaurant_id`: Identificador del restaurante
  - `cash_register_id`: ID de la caja registradora
  - `user_seller_id`: ID del cajero/vendedor
  - `device_os_name`: Sistema operativo del dispositivo
  - `source_app_name`: Nombre de la aplicación fuente

### Objeto Client
- `client_id`: Identificador único del cliente
- `name`: Nombre del cliente
- `last_name`: Apellido del cliente
- `gov_id_type`: Tipo de identificación gubernamental
- `gov_id_number`: Número de identificación gubernamental

### Objeto Order
- `order_id`: Identificador único del pedido
- `products`: Arreglo de productos pedidos
  - `plu_id`: Identificador del producto
  - `plu_name`: Nombre del producto
  - `price`: Información detallada de precios

## 🔹 Manejo de Errores

### Códigos de Error Comunes
- `400`: Solicitud incorrecta - Parámetros de entrada inválidos
- `401`: No autorizado - Autenticación inválida o faltante
- `500`: Error interno del servidor - Error en el procesamiento del servidor

## 🔹 Mejores Prácticas
1. Validar siempre el cuerpo de la solicitud antes de enviar
2. Implementar manejo adecuado de errores para todas las respuestas de la API
3. Utilizar el `syncId` para seguimiento y depuración
4. Asegurar la información sensible en la configuración
5. Implementar lógica de reintento para trabajos de impresión fallidos

## 🔹 Límites de Tasa
- Se aplican límites estándar (normalmente 100 solicitudes por minuto por IP)
- Implementar estrategias de retroceso apropiadas para solicitudes limitadas

## 🔹 Soporte
Para soporte o preguntas, por favor contacte al equipo de soporte de MAXPOINT en soporte@maxpoint.com.ec

## 🔹 Versión
Esta documentación es para la versión 2.0 de la API

## 🔹 Registro de Cambios
- 2024-09-14: Documentación inicial creada

---
*Documentación generada el: 2024-09-14*
*Última actualización: 2024-09-14*
