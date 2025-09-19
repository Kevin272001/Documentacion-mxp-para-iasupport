---
id: dipay-generate-payment
sidebar_label: Pago Digital (generatePayDigital)
---

# Módulo de Pagos Digitales - Generación y Anulación de Pagos

## 1. Propósito y alcance
El módulo **Pagos Digitales** permite la generación y anulación de pagos electrónicos en el sistema MAXPOINT a través del microservicio MXPi-DiPay.  
Gestiona el proceso de cobro, registro de información del punto de venta, pagador, productos y configuración de callbacks para notificaciones.

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- Integración de Pagos  
- Gestión de Ventas  
- Seguridad de API  

## 2. Descripción general del sistema
- **Microservicio:** mxpi-dipay  
- **Rol principal:** gestión de pagos digitales y anulaciones  
- **Función crítica:** asegurar la correcta integración y registro de pagos electrónicos  

## 3. Arquitectura y componentes

### 3.1 Capa de fachada
| Componente                | Código         | Descripción                                                                 |
|---------------------------|---------------|-----------------------------------------------------------------------------|
| Interfaz REST Pago Digital| DIPAY-001     | Punto de entrada para solicitudes de generación y anulación de pagos digitales. Valida la estructura y procesa la transacción. |

### 3.2 Validaciones
- **Método de pago válido:** verifica que el método de pago exista en el catálogo.
- **Datos de punto de venta:** valida la información de caja, cajero y canal.
- **Detalle de precios:** verifica moneda, total, impuestos y descuentos.
- **Datos del pagador:** valida documento, nombre, dirección y contacto.
- **Productos:** cada producto debe tener nombre, precio, SKU y cantidad.
- **Callbacks:** URLs válidas para notificaciones de éxito o fallo (opcional).
- **Formato correcto:** todos los campos requeridos deben estar presentes y correctamente estructurados.

## 4. Gestión de procesos
### Flujo de trabajo estándar
1. **Recepción** → Se recibe la solicitud POST en `/api/v1/generatePayment`.
2. **Validación** → Se verifica la estructura y los campos obligatorios.
3. **Procesamiento** → Se registra el pago y se integra con los sistemas de cobro.
4. **Respuesta** → Se retorna el resultado de la operación, incluyendo mensajes y datos de referencia.

## 5. Puntos de integración
- **Entrada:** datos de venta, pagador y productos desde sistemas externos.
- **Salida:** confirmación de pago, mensajes y datos de referencia.
- **Rol central:** garantizar la correcta integración y registro de pagos digitales.

## 6. Patrones de operación
- **Validación estricta:** todos los campos requeridos deben estar presentes.
- **Seguridad:** la información se procesa de forma segura y auditada.
- **Respuestas claras:** mensajes descriptivos y códigos de estado.

## 7. Escenarios de uso
✅ **Caso exitoso:**  
Solicitud POST `/api/v1/generatePayment` con datos completos y válidos → validación → procesamiento → respuesta de éxito.

⚠️ **Posibles fallos:**  
- Método de pago inválido → error de validación.
- Datos faltantes o incorrectos → error de validación.
- Problemas de integración o procesamiento → respuesta de error.

## 8. Endpoints

### Generar Pago Digital
- **Método:** POST  
- **URL:** `{{serverDipay}}/api/v1/generatePayment`  
- **Headers:**
  ```http
  Content-Type: application/json
  ```
- **Body:**
```json
{
    "paymentMethod": "de-una",
    "pointOfSale": {
        "id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
        "code": "EC-CAJA001",
        "cashierId": "7421A2EE-B13B-EA11-80EA-000D3A019254",
        "cashierName": "pepito",
        "platform": "ios",
        "account": "KFC",
        "accountId": 1,
        "source": "App",
        "channelId": 3,
        "channelName": "App"
    },
    "price": {
        "currency": "USD",
        "total": 100,
        "subsidy": 0,
        "discount": 0,
        "tax": 20
    },
    "payer": {
        "documentType": "CC",
        "documentNumber": "1009283738",
        "firstName": "Joy",
        "lastName": "Lino",
        "phone": "+593988734644",
        "address": "Eloy Alfaro 139 y Catalina Aldaz",
        "city": "Quito",
        "state": "Pichincha",
        "country": "Ecuador",
        "zipCode": "170402"
    },
    "orderDetail": {
        "products": [
            {
                "name": "ALITAS",
                "price": {
                    "currency": "USD",
                    "unitaryPrice": 24,
                    "tax": 0
                },
                "sku": "10101042",
                "quantity": 100
            },
            {
                "name": "HAMBURGUESA CON QUESO",
                "price": {
                    "currency": "USD",
                    "unitaryPrice": 24,
                    "tax": 0
                },
                "sku": "004834GQ",
                "quantity": 100
            }
        ]
    },
    "settings": {
        "callbacks": {
            "paymentTimeout": 234,
            "approved": "https://callback.maxpointv2.com",
            "failed": "https://callback.activereport.maxpointv2.com"
        },
        "theme": {
            "primaryColor": "#000001",
            "secundaryColor": "#000002",
            "backgroundColor": "#000003",
            "textColor": "black"
        }
    },
    "metadata": {
        "deviceIpAddress": "192.168.2.38"
    }
}
```

#### Parámetros del Body
- `paymentMethod`: Método de pago (obligatorio, catálogo).
- `pointOfSale`: Información de caja, cajero, canal, etc. (obligatorio).
- `price`: Detalle de precios, impuestos, descuentos (obligatorio).
- `payer`: Datos del pagador (obligatorio).
- `orderDetail`: Productos vendidos (obligatorio).
- `settings`: Callbacks y tema visual (opcional).
- `metadata`: Información adicional, como IP (opcional).

### Respuesta Exitosa (200)
```json
{
    "code": 200,
    "status": "success",
    "alert": "Pagado Correctamente",
    "messages": [
        "Integración completada exitosamente",
        "Integración completada exitosamente"
    ],
    "data": {
        "total": 12.00,
        "paymentMethod": "de-una",
        "idReference": "dsfsdfsdfsdfsd"
    }
}
```

#### Campos de la Respuesta Exitosa
| Campo         | Tipo     | Descripción                                 |
|---------------|----------|---------------------------------------------|
| code          | integer  | Código de estado HTTP                       |
| status        | string   | Estado de la operación                      |
| alert         | string   | Mensaje de alerta general                   |
| messages      | array    | Mensajes descriptivos                       |
| data.total    | number   | Total pagado                                |
| data.paymentMethod | string | Método de pago utilizado                |
| data.idReference   | string | Identificador de la transacción         |

### Respuesta con Error (420)
```json
{
    "code": 420,
    "status": "success",
    "alert": "Pagado NO Exitoso",
    "messages": [
        "Integración completada exitosamente",
        "Integración completada exitosamente"
    ],
    "data": {
        "total": 12.00,
        "paymentMethod": "de-una",
        "idReference": "dsfsdfsdfsdfsd"
    }
}
```

## 9. Preguntas frecuentes (Soporte IA)
Q: ¿Qué métodos de pago están disponibles?  
A: Los métodos de pago válidos se encuentran en el catálogo configurado en el sistema.

Q: ¿Puedo personalizar los callbacks de notificación?  
A: Sí, puedes definir URLs para notificaciones de éxito o fallo en el objeto `settings.callbacks`.

Q: ¿Qué hago si recibo un código 420?  
A: Revisa los mensajes y valida los datos enviados. El pago no fue exitoso, pero la integración sí se procesó.

## 10. Referencias
- Políticas de integración de pagos digitales  
- Estándares de seguridad para pagos electrónicos  
- Documentación técnica de MXPi-DiPay

## 11. Contacto
- **Equipo responsable:** Integraciones y Pagos Digitales  
- **Canal de soporte:** `#mxp-soporte` (Slack interno)  
- **Escalamiento:** Equipo de Pagos → Administradores de Sistema