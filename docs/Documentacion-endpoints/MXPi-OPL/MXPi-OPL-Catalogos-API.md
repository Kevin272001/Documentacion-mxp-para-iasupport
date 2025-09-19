# Documentación Técnica - Endpoint de Catálogos

## 1. Propósito y Alcance
### 1.1 Objetivo
Este documento proporciona la especificación técnica para el endpoint `Catálogos`, encargado de gestionar la obtención de información de configuración del sistema, incluyendo la apertura de caja (OPEN_CASH) y otros parámetros de configuración.

### 1.2 Alcance
- Obtención de configuraciones del sistema
- Soporte para múltiples tipos de catálogos
- Gestión de parámetros de configuración
- Integración con otros módulos del sistema

## 2. Descripción del Sistema
### 2.1 Visión General
El endpoint permite consultar diferentes catálogos del sistema necesarios para el funcionamiento de la aplicación, incluyendo la configuración de apertura de caja.

### 2.2 Características Clave
- Consulta de múltiples tipos de catálogos
- Respuesta estructurada en formato JSON
- Autenticación mediante token
- Validación de parámetros de entrada

## 3. Especificación de Endpoints

### 3.1 Obtener Catálogo
```
GET {{server_opl_catalogos}}/getCatalogos
```

#### Headers
```
Content-Type: application/json
```

#### Cuerpo de la Petición
```json
{
    "key": "OPEN_CASH"
}
```

#### Respuesta Exitosa (200)
```json
{
    "code": 200,
    "status": "success",
    "message": "Catálogo obtenido exitosamente",
    "data": {
        "catalog_type": "OPEN_CASH",
        "parameters": {
            "allowed_currencies": ["USD", "EUR"],
            "min_opening_amount": 50.00,
            "max_opening_amount": 10000.00,
            "required_fields": ["cash_register_id", "user_id", "opening_amount"],
            "default_currency": "USD"
        },
        "last_updated": "2025-09-15T08:30:45.123Z"
    }
}
```

#### Campos de la Respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| code | integer | Código de estado HTTP |
| status | string | Estado de la operación |
| message | string | Mensaje descriptivo |
| data.catalog_type | string | Tipo de catálogo solicitado |
| data.parameters | object | Parámetros de configuración específicos del catálogo |
| data.last_updated | string | Fecha y hora de la última actualización |

#### Respuesta con Error (400)
```json
{
    "code": 400,
    "status": "error",
    "message": "Parámetro 'key' es requerido"
}
```

#### Respuesta con Error (404)
```json
{
    "code": 404,
    "status": "error",
    "message": "Catálogo no encontrado"
}
```

## 4. Preguntas Frecuentes (FAQ)

### 4.1 ¿Qué tipos de catálogos están disponibles?
El sistema soporta múltiples tipos de catálogos, incluyendo pero no limitado a: configuración de apertura de caja (OPEN_CASH), métodos de pago, tipos de documento, entre otros.

### 4.2 ¿Cómo sé qué campos son obligatorios para cada tipo de catálogo?
Cada tipo de catálogo puede tener diferentes campos obligatorios. Consulte la documentación específica del catálogo o utilice este endpoint para obtener los campos requeridos.

### 4.3 ¿Qué debo hacer si recibo un error 404?
Verifique que el valor del parámetro 'key' sea correcto y que tenga los permisos necesarios para acceder al catálogo solicitado.

## 5. Referencias
- Documentación de la API de Catálogos
- Guía de Integración de MAXPOINT V2.0
- Estándares de Integración

## 6. Contacto y Soporte
### 6.1 Soporte Técnico
- Email: soporte@empresa.com
- Teléfono: +593 2 123 4567
- Horario: Lunes a Viernes 08:00 - 17:00

### 6.2 Canales de Atención
- Portal de Soporte: https://soporte.empresa.com
- Sistema de Tickets
- Chat en Línea

### 6.3 Escalación
Para asuntos urgentes fuera de horario laboral, contactar al +593 99 123 4567.

---
*Última actualización: Septiembre 2025*
