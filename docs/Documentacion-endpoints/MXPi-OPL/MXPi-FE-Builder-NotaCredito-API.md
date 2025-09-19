# Documentación Técnica - Endpoint de Nota de Crédito

## 1. Propósito y Alcance
### 1.1 Objetivo
Este documento proporciona la especificación técnica para el endpoint `Interface-fe-build_NOTA_DE_CREDITO`, encargado de gestionar la generación de notas de crédito electrónicas en cumplimiento con las regulaciones fiscales ecuatorianas.

### 1.2 Alcance
- Generación de notas de crédito electrónicas
- Integración con el SRI (Servicio de Rentas Internas de Ecuador)
- Manejo de anulaciones y devoluciones
- Procesamiento de información fiscal y contable

## 2. Descripción del Sistema
### 2.1 Visión General
El endpoint permite la generación de notas de crédito electrónicas como documentos tributarios que respaldan la anulación total o parcial de una factura electrónica previamente emitida.

### 2.2 Características Clave
- Generación de claves de acceso únicas
- Cálculo automático de impuestos
- Validación de datos de entrada
- Integración con sistemas contables
- Generación de archivos XML y PDF

## 3. Arquitectura
### 3.1 Componentes Principales
- **API REST**: Interfaz de comunicación
- **Servicio de Facturación Electrónica**: Procesamiento de documentos
- **Base de Datos**: Almacenamiento de transacciones
- **Integración SRI**: Comunicación con el servicio de rentas internas

### 3.3 Tecnologías Utilizadas
- Lenguaje: Java/Spring Boot
- Base de Datos: SQL Server
- Comunicación: REST/JSON
- Seguridad: JWT

## 4. Flujo del Proceso
### 4.1 Diagrama de Secuencia
1. Recepción de la solicitud de nota de crédito
2. Validación de datos de entrada
3. Procesamiento de la transacción
4. Generación de la nota de crédito
5. Envío al SRI
6. Notificación de respuesta
7. Generación de PDF
8. Envío de respuesta al cliente

## 5. Puntos de Integración
### 5.1 Entradas
- Datos de la factura original
- Información del cliente
- Detalles de los productos/servicios
- Motivo de la anulación
- Información de pagos

### 5.2 Salidas
- Nota de crédito electrónica
- Clave de acceso
- Número de autorización
- Enlace de descarga
- Estado de la transacción

## 6. Patrones de Operación
### 6.1 Validaciones
- Existencia de la factura original
- Coherencia de montos
- Validez de datos tributarios
- Estado de la factura original

### 6.2 Manejo de Errores
- Códigos de error estandarizados
- Mensajes descriptivos
- Registro de eventos
- Reintentos automáticos

## 7. Casos de Uso
### 7.1 Flujo Exitoso
1. Cliente solicita anulación de factura
2. Sistema valida datos
3. Se genera nota de crédito
4. Se notifica al cliente

### 7.2 Flujo con Errores
1. Cliente solicita anulación
2. Sistema detecta error en datos
3. Se notifica el error
4. Se registra el intento

## 8. Especificación de Endpoints

### 8.1 Generar Nota de Crédito
```
POST {{serverFeBuilder}}/api/v1/transaction/TX003
```

#### Headers
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

#### Cuerpo de la Petición
```json
{
    "type_document": "NOTA_CREDITO-ECU",
    "transaction": {
        "transaction_date": "2025-07-17T15:33:20.123",
        "point_of_sale": {
            "cdn_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
            "restaurant_id": "e17d03da-b8b6-f424-febc-3a767b6401bb",
            "cash_register_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "cash_register_name": "EC-CAJA001",
            "user_seller_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "user_seller_name": "Daniel Llerena",
            "device_os_name": "ios",
            "source_app_name": "Maxpoint 2.0",
            "half_integration_id": 3,
            "half_integration_name": "App"
        },
        "client": {
            "client_id": "030B9503-85CF-E511-80C6-000D3A3261F3",
            "name": "María José",
            "last_name": "Hernández Zurita",
            "phone": "+593986610329",
            "email": "mari_jhz@hotmail.com",
            "gov_id_type": "CI",
            "gov_id_number": "0929691038"
        },
        "order": {
            "order_id": "0001292432-010103",
            "createdAt": "2025-05-15 13:53:40",
            "invoice_id": "D20A9503-85CF-E511-80C6-000D3A3261F3",
            "invoice_key_aut": "4323432534654765686789798679789",
            "products": [
                {
                    "plu_id": "401",
                    "plu_name": "COMBO IDEAL",
                    "plu_quantity": 1,
                    "plu_description": "",
                    "price": {
                        "total_per_plu": {
                            "currency_code": "USD",
                            "subtotal_without_taxes": 5.65,
                            "discount_percentage": 0,
                            "discounts_value": 0,
                            "subtotal_include_discounts": 5.65,
                            "tax_value": 0.85,
                            "total": 6.5
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
                    "taxes_percentage": 15,
                    "tax_value": 0.85,
                    "total_before_sale": 6.5,
                    "total": 6.5
                }
            ]
        },
        "additional_data": [
            {
                "key": "MARCA",
                "value": "TOYOTA"
            }
        ],
        "annulation_comment": "me equivoque!!"
    }
}
```

#### Respuesta Exitosa (200)
```json
{
    "code": 200,
    "status": "success",
    "alert": "Integración completada exitosamente",
    "data": {
        "companyTaxData": {
            "name": "INT FOOD SERVICES CORP SA",
            "identification_number": "1234567890001",
            "special_taxpayer": "GRAN CONTRIBUYENTE"
        },
        "dataCredit": {
            "sequential": "050-050-000470031",
            "transaction": "K024N001529034",
            "accessKey": "091020240117914151320011024050000470020412615331",
            "url_document": "https://facturacion.kfc.ec"
        }
    }
}
```

#### Respuesta con Error (400)
```json
{
    "code": 400,
    "status": "failed",
    "alert": "Integración incompleta",
    "messages": [
        "Faltan campos requeridos",
        "La factura original no existe"
    ]
}
```

## 9. Preguntas Frecuentes (FAQ)

### 9.1 ¿Qué es una nota de crédito?
Es un documento tributario que anula total o parcialmente una factura electrónica previamente emitida.

### 9.2 ¿Cuánto tiempo tengo para generar una nota de crédito?
Según el SRI, las notas de crédito deben emitirse dentro de los 30 días posteriores a la emisión de la factura original.

### 9.3 ¿Qué sucede si la factura ya fue reportada al SRI?
La nota de crédito se reportará al SRI para mantener la trazabilidad fiscal, anulando los efectos de la factura original.

## 10. Referencias
- Reglamento para la Aplicación de la Ley de Régimen Tributario Interno
- Normativa Técnica del SRI para Comprobantes Electrónicos
- Manual de Usuario de Facturación Electrónica

## 11. Contacto y Soporte
### 11.1 Soporte Técnico
- Email: soporte@empresa.com
- Teléfono: +593 2 123 4567
- Horario: Lunes a Viernes 08:00 - 17:00

### 11.2 Canales de Atención
- Portal de Soporte: https://soporte.empresa.com
- Chat en Línea
- Sistema de Tickets

### 11.3 Escalación
Para asuntos urgentes fuera de horario laboral, contactar al +593 99 123 4567.
