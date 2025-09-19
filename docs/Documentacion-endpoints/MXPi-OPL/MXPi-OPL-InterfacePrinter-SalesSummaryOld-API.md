---
id: interface-printer-salessummaryold
sidebar_label: InterfacePrinter-salesSummaryOld
---

# Módulo de Resumen de Ventas (InterfacePrinter-salesSummaryOld)

## 1. Propósito y alcance
El módulo **InterfacePrinter-salesSummaryOld** se encarga de generar resúmenes de ventas para diferentes tipos de informes financieros, incluyendo cierres de día, desasignaciones de cajero, reportes X y arqueos de caja.  
Permite la generación de reportes detallados con información de transacciones, impuestos, descuentos y métodos de pago.  
Garantiza la trazabilidad de las operaciones realizadas en los puntos de venta.

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- InterfacePrinter  
- Facade-Printer-Voucher  
- Transaction  

## 2. Descripción general del sistema
- **Microservicio:** mxpv2.integration.opl  
- **Rol principal:** generar reportes de resumen de ventas  
- **Función crítica:** asegurar la exactitud de la información financiera reportada  

## 3. Arquitectura y componentes

### 3.1 Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz REST SalesSummary | SS001 | Punto de entrada para solicitudes de generación de resúmenes de ventas. Valida estructura e integridad de los datos. |

### 3.2 Tipos de reportes soportados
- `FIN_DE_DIA`: Resumen de cierre de día
- `DESASIGNAR_CAJERO`: Reporte de desasignación de cajero
- `REPORTE_X`: Reporte X de caja
- `ARQUEO_CAJA`: Arqueo de caja

## 4. Gestión de procesos
### Flujo de trabajo estándar
1. **Recepción** → Se recibe la solicitud POST en `/api/v1/transaction/TX011`.  
2. **Validación** → Se verifica la estructura y los campos obligatorios.  
3. **Procesamiento** → Se genera el resumen de ventas según el tipo de reporte solicitado.  
4. **Respuesta** → Se devuelve el resultado de la operación.  

## 5. Puntos de integración
- **Entrada:** solicitudes de frontend, otros módulos de MAXPOINT V2.0.  
- **Salida:** documento con el resumen de ventas.  
- **Rol central:** garantizar la exactitud de la información financiera reportada.  

## 6. Patrones de operación
- **Validación estricta:** todos los campos obligatorios deben estar presentes.  
- **Manejo de errores:** respuestas detalladas en caso de fallos.  
- **Registro de auditoría:** todas las operaciones quedan registradas.  

## 7. Escenarios de uso
✅ **Caso exitoso:**  
Solicitud POST `/api/v1/transaction/TX011` con datos correctos → validación → generación de reporte → respuesta de éxito.  

⚠️ **Posibles fallos:**  
- Datos incompletos o inválidos → error de validación.  
- Error en el formato de fechas → error de procesamiento.  
- Problemas de conexión con la base de datos → error del servidor.  

## 8. Endpoints
### Generar Resumen de Ventas
- **Método:** POST  
- **URL:** `/api/v1/transaction/TX011`  
- **Headers:**
  ```http
  Authorization: Bearer {{token}}
  Content-Type: application/json
  ```
- **Body:**
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
            "user_admin": "Carlos Zambrano"
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
            "swaps": {
                "quantity": 2,
                "total_talue": 23.56
            },
            "credits": {
                "quantity": 2,
                "total_talue": 23.56
            },
            "courtesies": {
                "quantity": 2,
                "total_talue": 23.56
            },
            "totalCCC": {
                "quantity": 2,
                "total_talue": 23.56
            },
            "correctionOfErrors": {
                "quantity": 2,
                "total_talue": 23.56
            },
            "creditNotes": {
                "quantity": 2,
                "total_talue": 23.56
            },
            "declaredValues": [
                {
                    "paymentMethod": "DATAFONO",
                    "quantity": 2,
                    "total_talue": 23.56
                },
                {
                    "paymentMethod": "FIDELIZACION",
                    "quantity": 2,
                    "total_talue": 23.56
                }
            ],
            "expenses": 0.00,
            "income": 45.90,
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

### Respuesta Exitosa (200)
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
    "code": 500,
    "status": "failed",
    "alert": "Ocurrió un error al imprimir",
    "messages": [
        "A non-empty request body is required."
    ]
}
```

## 9. Preguntas frecuentes (Soporte IA)
Q: ¿Qué tipos de reportes soporta este endpoint?  
A: Soporta FIN_DE_DIA, DESASIGNAR_CAJERO, REPORTE_X y ARQUEO_CAJA.  

Q: ¿Qué hago si recibo un error de autenticación?  
A: Verifica que el token de autenticación sea válido y tenga los permisos necesarios.  

Q: ¿Cómo interpreto el campo 'declaredValues'?  
A: Contiene información de los valores declarados por método de pago, incluyendo cantidad y monto total.  

## 10. Referencias
- Documentación técnica de MAXPOINT V2.0  
- Especificaciones de formato de reportes financieros  

## 11. Contacto
- **Equipo responsable:** Integraciones MAXPOINT  
- **Canal de soporte:** `#mxp-integraciones` (Slack interno)  
- **Escalamiento:** Equipo de Operaciones → Soporte Técnico
