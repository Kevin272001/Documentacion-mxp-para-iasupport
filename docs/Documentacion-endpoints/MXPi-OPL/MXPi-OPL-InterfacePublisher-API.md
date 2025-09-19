# Documentación Técnica - Endpoint de Publicación de Eventos

## 1. Propósito y Alcance
### 1.1 Objetivo
Este documento proporciona la especificación técnica para el endpoint `Interface Publisher`, encargado de gestionar la publicación de eventos y notificaciones en el sistema, como la apertura de caja (OPEN_CASH) y el manejo de cupones (CUPONES).

### 1.2 Alcance
- Publicación de eventos del sistema
- Manejo de diferentes tipos de publicaciones (apertura de caja, cupones, etc.)
- Gestión de notificaciones en tiempo real
- Integración con sistemas externos

## 2. Descripción del Sistema
### 2.1 Visión General
El endpoint permite la publicación de eventos en el sistema, facilitando la comunicación entre diferentes módulos y la generación de notificaciones en tiempo real.

### 2.2 Características Clave
- Soporte para múltiples tipos de publicaciones
- Notificaciones en tiempo real
- Gestión de eventos del sistema
- Registro de transacciones

## 3. Arquitectura
### 3.1 Componentes Principales
- **API REST**: Interfaz de publicación de eventos
- **Servicio de Mensajería**: Gestión de colas de mensajes
- **Base de Datos**: Almacenamiento de eventos
- **Sistema de Notificaciones**: Distribución de eventos

### 3.2 Tecnologías Utilizadas
- Lenguaje: Java/Spring Boot
- Base de Datos: SQL Server
- Comunicación: REST/JSON
- Seguridad: JWT
- Mensajería: RabbitMQ/Kafka

## 4. Flujo del Proceso
### 4.1 Diagrama de Secuencia
1. Cliente envía solicitud de publicación
2. Servicio valida autenticación y autorización
3. Sistema procesa el evento
4. Se notifica a los suscriptores
5. Se registra la transacción
6. Se envía confirmación al cliente

## 5. Puntos de Integración
### 5.1 Entradas
- Token de autenticación
- Tipo de publicación
- Datos específicos del evento
- Metadatos de la transacción

### 5.2 Salidas
- Confirmación de publicación
- Estado del evento
- Código de seguimiento
- Timestamp de la operación

## 6. Patrones de Operación
### 6.1 Validaciones
- Autenticación del usuario
- Permisos de publicación
- Formato de los datos
- Disponibilidad del servicio

### 6.2 Manejo de Errores
- Códigos de error estandarizados
- Reintentos automáticos
- Registro de fallos
- Notificaciones de error

## 7. Casos de Uso
### 7.1 Flujo Exitoso
1. Aplicación cliente publica evento
2. Sistema valida y procesa
3. Se notifica a los suscriptores
4. Se confirma la publicación

### 7.2 Flujo con Errores
1. Publicación sin autenticación
2. Datos inválidos
3. Servicio no disponible
4. Error en el procesamiento

## 8. Especificación de Endpoints

### 8.1 Publicar Evento
```
POST {{server_pub_device}}/api/v1/transaction/TX012
```

#### Headers
```
Authorization: Bearer {{token}}
Content-Type: application/json
```

#### Cuerpo de la Petición (Ejemplo: Apertura de Caja)
```json
{
    "type_publisher": "OPEN_CASH",
    "transaction": {
        "point_of_sale": {
            "cash_register_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "user_id": "D20A9503-85CF-E511-80C6-000D3A3261F3",
            "opening_amount": 100.00,
            "opening_currency": "USD",
            "opening_notes": "Apertura de caja turno mañana"
        }
    }
}
```

#### Cuerpo de la Petición (Ejemplo: Cupón)
```json
{
    "type_publisher": "CUPONES",
    "transaction": {
        "coupon": {
            "code": "DESC20",
            "type": "DISCOUNT",
            "value": 20.00,
            "currency": "USD",
            "valid_until": "2025-12-31T23:59:59",
            "max_uses": 100,
            "min_purchase": 50.00
        }
    }
}
```

#### Respuesta Exitosa (200)
```json
{
    "code": 200,
    "status": "success",
    "message": "Evento publicado exitosamente",
    "data": {
        "event_id": "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8",
        "type": "OPEN_CASH",
        "status": "PROCESSED",
        "timestamp": "2025-09-15T08:30:45.123Z",
        "details": {
            "cash_register_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "opening_amount": 100.00,
            "opening_currency": "USD"
        }
    }
}
```

#### Campos de la Respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| code | integer | Código de estado HTTP |
| status | string | Estado de la operación |
| message | string | Mensaje descriptivo |
| data.event_id | string | Identificador único del evento |
| data.type | string | Tipo de evento publicado |
| data.status | string | Estado del procesamiento |
| data.timestamp | string | Fecha y hora del evento |
| data.details | object | Detalles específicos del evento |

#### Respuesta con Error (400)
```json
{
    "code": 400,
    "status": "error",
    "message": "Datos de entrada inválidos",
    "errors": [
        "El campo 'opening_amount' es requerido",
        "El valor de 'opening_currency' no es válido"
    ]
}
```

#### Respuesta con Error (401)
```json
{
    "code": 401,
    "status": "error",
    "message": "No autorizado",
    "details": "Token de autenticación inválido o expirado"
}
```

#### Respuesta con Error (500)
```json
{
    "code": 500,
    "status": "error",
    "message": "Error interno del servidor",
    "reference": "ERR-500-9876"
}
```

## 9. Preguntas Frecuentes (FAQ)

### 9.1 ¿Qué tipos de publicaciones soporta este endpoint?
El endpoint soporta múltiples tipos de publicaciones, incluyendo pero no limitado a: apertura de caja (OPEN_CASH), cierre de caja (CLOSE_CASH), cupones (CUPONES), y otros eventos del sistema.

### 9.2 ¿Cómo maneja el sistema la concurrencia de publicaciones?
El sistema utiliza bloqueos optimistas y colas de mensajes para manejar la concurrencia, asegurando que no se pierdan mensajes y manteniendo la integridad de los datos.

### 9.3 ¿Qué debo hacer si recibo un error de timeout?
Se recomienda implementar un mecanismo de reintento con retroceso exponencial. Si el problema persiste, contactar al equipo de soporte con el código de referencia del error.

### 9.4 ¿Cómo puedo verificar el estado de una publicación?
Cada publicación exitosa devuelve un `event_id` único que puede ser utilizado para consultar el estado del evento a través del endpoint correspondiente.

## 10. Referencias
- Documentación de la API de Publicación
- Guía de Integración de Eventos
- Estándares de Mensajería Empresarial

## 11. Contacto y Soporte
### 11.1 Soporte Técnico
- Email: soporte@empresa.com
- Teléfono: +593 2 123 4567
- Horario: Lunes a Viernes 08:00 - 17:00

### 11.2 Canales de Atención
- Portal de Soporte: https://soporte.empresa.com
- Sistema de Tickets
- Chat en Línea

### 11.3 Escalación
Para asuntos urgentes fuera de horario laboral, contactar al +593 99 123 4567.

---
*Última actualización: Septiembre 2025*
