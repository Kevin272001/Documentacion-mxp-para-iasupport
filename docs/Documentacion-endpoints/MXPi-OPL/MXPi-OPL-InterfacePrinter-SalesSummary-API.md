# Documentación Técnica - InterfacePrinter-salesSummary (TX011)

## 1. Propósito y Alcance
### 1.1 Objetivo
Este documento proporciona las especificaciones detalladas para el endpoint de impresión de resúmenes de ventas dentro de MAXPOINT V2.0. El endpoint maneja la generación de informes resumidos de ventas, incluyendo cierres de día, asignaciones de cajero e informes X/Z para operaciones de punto de venta.

### 1.2 Alcance
- Generación de informes de ventas detallados
- Cierres de caja diarios
- Conciliación financiera
- Análisis operativo

## 2. Descripción del Sistema
### 2.1 Visión General
Parte del microservicio `mxpv2.integration.opl`, esta API permite la generación de resúmenes de ventas detallados que son esenciales para informes financieros, conciliación y análisis operativo en entornos minoristas y de restaurantes.

### 2.2 Características Clave
- Múltiples tipos de informes (cierre de día, asignación de cajero, informe X)
- Cálculo automático de totales e impuestos
- Conciliación de valores declarados vs calculados
- Gestión de fondos de caja

## 3. Especificación Técnica
### 3.1 Endpoint
```
POST {{serverPrinter}}/api/v1/transaction/TX011
```

### 3.2 Headers
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

### 3.3 Cuerpo de la Petición (Ejemplo)
```json
{
    "type_printer": "FIN_DE_DIA",
    "transaction": {
        "point_of_sale": {
            "cdn_id": "f2c13a02-3cb5-4781-9e10-d3f1519c51e2",
            "restaurant_id": "f2c13a02-3cb5-4781-9e10-d3f1519c51e2",
            "cash_register_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "cash_register_name": "EC-CAJA001",
            "user_seller_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "user_seller_name": "pepito",
            "device_os_name": "ios",
            "source_app_name": "Maxpoint 2.0",
            "half_integration_id": 3,
            "half_integration_name": "App",
            "user_admin": "Carlos Zambrano",
            "restaurant_name": "G008 - EL INCA",
            "franchise_name": "GUS"
        },
        "summaries": {
            "transacction_number": 23,
            "gross_sales": 234.53,
            "iva_base": 34.32,
            "not_iva_base": 3.43,
            "iva": 17.9,
            "discount": 45,
            "average_ticket": 47,
            "gratuities": 46.9,
            "exchange": {
                "quantity": 2,
                "total_value": 23.56
            },
            "credits": {
                "quantity": 2,
                "total_value": 23.56
            },
            "courtesies": {
                "quantity": 2,
                "total_value": 23.56
            },
            "totalCCC": {
                "quantity": 2,
                "total_value": 23.56
            },
            "correctionOfErrors": {
                "quantity": 2,
                "total_value": 23.56
            },
            "creditNotes": {
                "quantity": 2,
                "total_value": 23.56
            },
            "declaredValues": {
                "total_withdrawals_declared": 255,
                "total_withdrawals": 130,
                "total_declared": 125,
                "detail": [
                    {
                        "paymentMethod": "EFECTIVO",
                        "withdrawals": 50,
                        "declared": 100,
                        "pos_calculated": 150
                    },
                    {
                        "paymentMethod": "RETENCIÓN IVA",
                        "withdrawals": 0,
                        "declared": 10,
                        "pos_calculated": 10
                    },
                    {
                        "paymentMethod": "FIDELIZACIÓN",
                        "withdrawals": 55,
                        "declared": 0,
                        "pos_calculated": 55
                    },
                    {
                        "paymentMethod": "tarjetas credito",
                        "withdrawals": 80,
                        "declared": 15,
                        "pos_calculated": 95,
                        "detail": [
                            {
                                "paymentMethod": "AMEX",
                                "withdrawals": 0,
                                "declared": 15,
                                "pos_calculated": 15
                            },
                            {
                                "paymentMethod": "DINERS",
                                "withdrawals": 80,
                                "declared": 0,
                                "pos_calculated": 80
                            }
                        ]
                    }
                ]
            },
            "egress": [
                {
                    "description": "EFECTIVO",
                    "value": 100
                }
            ],
            "income": null,
            "surplus_or_shortage": -0.75,
            "cash_fund": {
                "assigned_value": 45.39,
                "confirmed_value": 34.2,
                "withdrawn_value": 21.3,
                "value_for_withdrawal": 32
            }
        }
    }
}
```

## 4. Descripción de Campos

### 4.1 Campos de la Solicitud
- **type_printer**: Tipo de informe de resumen (ej. "FIN_DE_DIA" para cierre de día, "Corte en X" para informe X)
- **transaction**: Contenedor para los detalles de la transacción
  - **point_of_sale**: Información del punto de venta
  - **summaries**: Información detallada del resumen de ventas
    - **transacction_number**: Número total de transacciones
    - **gross_sales**: Monto total de ventas brutas
    - **iva_base**: Base imponible
    - **not_iva_base**: Base no imponible
    - **iva**: Monto total de impuestos
    - **discount**: Descuentos totales aplicados
    - **average_ticket**: Valor promedio por transacción
    - **gratuities**: Propinas totales
    - **exchange/credits/courtesies/totalCCC/correctionOfErrors/creditNotes**: Varios tipos de transacciones con cantidad y valor
    - **declaredValues**: Detalles de conciliación
    - **egress/income**: Información de flujo de efectivo
    - **surplus_or_shortage**: Diferencia entre montos declarados y calculados
    - **cash_fund**: Detalles de gestión de efectivo

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

### ¿Qué tipos de resúmenes de ventas se pueden generar?
El endpoint admite varios tipos de informes incluyendo "FIN_DE_DIA" (cierre de día), "Arquero" (asignación de cajero), "Desasignar cajero" (desasignación de cajero) y "Corte en X" (informe X).

### ¿Cómo se calcula el campo `surplus_or_shortage`?
Este campo representa la diferencia entre los montos declarados y los calculados por el POS. Un valor positivo indica un excedente, mientras que un valor negativo indica un faltante.

### ¿Qué debo hacer si recibo un error 400?
Verifique el mensaje de error en la respuesta. Los problemas comunes incluyen campos obligatorios faltantes, formatos de datos inválidos o problemas de configuración de plantilla en el servidor.

## 8. Referencias
- Documentación de la API MAXPOINT V2.0
- Estándares de Informes Financieros
- Directrices de Conciliación de Ventas

## 9. Información de Contacto
Para soporte o preguntas, por favor contacte a:
- **Correo Electrónico**: soporte@maxpoint.com
- **Teléfono**: +1-800-123-4567
- **Portal de Soporte**: https://support.maxpoint.com
