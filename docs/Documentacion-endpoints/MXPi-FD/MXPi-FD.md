---
id: mxpi-fd-loyalty
sidebar_label: Fidelización (MXPi-FD)
---

# Módulo de Fidelización - Validación, Transacciones y Reversos

## 1. Propósito y alcance
El módulo **Fidelización** permite validar códigos de cliente, registrar transacciones de puntos, reversar y cancelar operaciones en el sistema de lealtad de MAXPOINT.  
Gestiona la acumulación, redención y reverso de puntos, así como la consulta y validación de clientes.

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- Gestión de Clientes  
- Gestión de Puntos  
- Seguridad de API  

## 2. Descripción general del sistema
- **Microservicio:** mxpi-fd  
- **Rol principal:** gestión de validaciones y transacciones de fidelización  
- **Función crítica:** asegurar la correcta administración de puntos y validaciones de clientes  

## 3. Arquitectura y componentes

### 3.1 Capa de fachada
| Componente                | Código         | Descripción                                                                 |
|---------------------------|---------------|-----------------------------------------------------------------------------|
| Interfaz REST Fidelización| FD-001         | Punto de entrada para validación de códigos, registro de transacciones, reversos y cancelaciones de puntos. |

### 3.2 Validaciones
- **Código de cliente válido:** verifica que el código de cliente exista y sea válido.
- **Transacción válida:** valida la estructura y los campos obligatorios de la transacción.
- **Reverso/Cancela válido:** verifica que el transactionId exista y la razón sea válida.
- **Formato correcto:** todos los campos requeridos deben estar presentes y correctamente estructurados.

## 4. Gestión de procesos
### Flujo de trabajo estándar
1. **Recepción** → Se recibe la solicitud en el endpoint correspondiente (validate, transaction, reverse, cancel).
2. **Validación** → Se verifica la estructura y los campos obligatorios.
3. **Procesamiento** → Se realiza la validación, registro, reverso o cancelación según corresponda.
4. **Respuesta** → Se retorna el resultado de la operación, incluyendo mensajes y datos relevantes.

## 5. Puntos de integración
- **Entrada:** datos de cliente, transacción o reverso desde sistemas externos.
- **Salida:** confirmación de validación, transacción, reverso o cancelación.
- **Rol central:** garantizar la correcta administración de puntos y validaciones de clientes.

## 6. Patrones de operación
- **Validación estricta:** todos los campos requeridos deben estar presentes.
- **Seguridad:** la información se procesa de forma segura y auditada.
- **Respuestas claras:** mensajes descriptivos y códigos de estado.

## 7. Escenarios de uso
✅ **Caso exitoso:**  
Solicitud GET `/api/loyalty/validate/{codigo}` → validación → respuesta con datos del cliente y puntos.  
Solicitud POST `/api/loyalty/transaction` → registro de transacción → respuesta con puntos acumulados/redimidos.  
Solicitud PUT `/api/loyalty/reverse` → reverso de puntos → respuesta con puntos revertidos.  
Solicitud PUT `/api/loyalty/cancel/{transactionId}` → cancelación de transacción → respuesta de éxito.

⚠️ **Posibles fallos:**  
- Código de cliente inválido → error de validación.
- Datos faltantes o incorrectos → error de validación.
- Usuario o contraseña incorrectos → error de autenticación.

## 8. Endpoints

### Validar Código de Cliente
- **Método:** GET  
- **URL:** `http://localhost:8080/api/loyalty/validate/{codigo}`  
- **Response exitosa:**
```json
{
    "status": "OK",
    "code": "200",
    "msg": "Se valido su código con éxito",
    "data": {
        "transactionId": "jV0QmFakiYbfWDJUaqRX8DdojrG2",
        "points": 300,
        "customer": {
            "id": "1311573586",
            "name": "Mychael Castro",
            "email": "devsystem16@gmail.com"
        }
    },
    "error": null
}
```
- **Response 401:**
```json
{
    "status": "UNAUTHORIZED",
    "code": "401",
    "msg": "Usuario o contraseña incorrectos",
    "data": null,
    "error": {
        "msg": "password required",
        "time": "2025-02-07T18:58:34Z",
        "route": "/api/loyalty/auth"
    }
}
```

### Registrar Transacción de Puntos
- **Método:** POST  
- **URL:** `http://localhost:8080/api/loyalty/transaction`  
- **Body:**
```json
{
    "transactionId":"jV0QmFakiYbfWDJUaqRX8DdojrG2",
    "commerce": "JUAN VALDEZ",
    "storeCode": 168,
    "redemtion": true,
    "acumulate": false,
    "invoice": {
        "code": "V008F000154584",
        "tax": 0,
        "taxBase": 10,
        "subtotal": 0,
        "total": 0,
        "payment_method": "CASH",
        "products": [
            {
                "code": "101",
                "amount": 2,
                "unitValue": 10.5,
                "tax": 0,
                "taxBase": 10,
                "total": 0,
                "points": 250
            }
        ]
    }
}
```
- **Response exitosa:**
```json
{
    "status": "OK",
    "code": "200",
    "msg": "Su transacción fue enviada con éxito",
    "data": {
        "acumPoints": 0,
        "redemPoints": 250,
        "currentPoints" : 100,
        "billingInformation": {
            "businessName": "PROMOTORA ECUATORIANA DE CAFE",
            "govIdType": "1",
            "govIdNumber": "1301115554001",
            "phone": "0960445403",
            "email": "promotora@trade.com",
            "address": "quito"
        },
        "provider": {
            "providerId": "1120120",
            "name": "Masivo",
            "app": "Amigos Juan Valdez"
        }
    },
    "error": null
}
```
- **Response 401:** igual a la anterior.

### Reversar Puntos de una Transacción
- **Método:** PUT  
- **URL:** `http://localhost:8080/api/loyalty/reverse`  
- **Body:**
```json
{
    "transactionId":"jV0QmFakiYbfWDJUaqRX8DdojrG2",
    "reazon": "{{reverseStore}}",
    "commerce": "JUAN VALDEZ",
    "storeCode": 168,
    "redemtion": true,
    "acumulate": false,
    "invoice": {
        "code": "V008F000154584",
        "tax": 0,
        "taxBase": 10,
        "subtotal": 0,
        "total": 0,
        "payment_method": "CASH",
        "products": [
            {
                "code": "101",
                "amount": 2,
                "unitValue": 10.5,
                "tax": 0,
                "taxBase": 10,
                "total": 0,
                "points": 250
            }
        ]
    }
}
```
- **Response exitosa:**
```json
{
    "status": "OK",
    "code": "200",
    "msg": "Se Reverso con éxito",
    "data": {
        "revertPoints": 30,
        "currentPoints" : 100
    },
    "error": null
}
```
- **Response 401:** igual a la anterior.

### Cancelar una Transacción
- **Método:** PUT  
- **URL:** `http://localhost:8080/api/loyalty/cancel/{transactionId}`  
- **Body:**
```json
{
    "transactionId": "KFC00101212121",
    "reazon": "Petición del cliente"
}
```
- **Response exitosa:**
```json
{
    "status": "OK",
    "code": "200",
    "msg": "Se Canceló con éxito",
    "data": null,
    "error": null
}
```
- **Response 401:** igual a la anterior.

### Otros Endpoints
- **Get Client:** Consulta de cliente (GET)
- **Canjeo de Cupones:** Redención de cupones (POST)
- **Reportar Ventas Total:** Reporte de ventas (POST)
- **Anular Venta:** Anulación de venta (POST)

## 9. Preguntas frecuentes (Soporte IA)
Q: ¿Cómo valido un código de cliente?  
A: Utiliza el endpoint `/api/loyalty/validate/{codigo}` y revisa la respuesta.

Q: ¿Puedo reversar una transacción?  
A: Sí, usando el endpoint `/api/loyalty/reverse` con el transactionId y los datos requeridos.

Q: ¿Qué hago si recibo un error 401?  
A: Verifica las credenciales de autenticación enviadas en la solicitud.

## 10. Referencias
- Políticas de acumulación y redención de puntos  
- Estándares de integración de fidelización  
- Documentación técnica de MXPi-FD

## 11. Contacto
- **Equipo responsable:** Fidelización y Puntos  
- **Canal de soporte:** `#mxp-soporte` (Slack interno)  
- **Escalamiento:** Equipo de Fidelización → Administradores de Sistema