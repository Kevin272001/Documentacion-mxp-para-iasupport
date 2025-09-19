---
id: auth-token
sidebar_label: Token de API (OtherClient)
---

# Módulo de Autenticación - Obtención de Token API

## 1. Propósito y alcance
El módulo **Token de API** permite la obtención de un token de autenticación para clientes externos en el sistema MAXPOINT.  
Gestiona el proceso de validación de credenciales de cliente y la generación de un token seguro para el acceso a otros servicios protegidos.

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- Autenticación  
- Gestión de Clientes  
- Seguridad de API  

## 2. Descripción general del sistema
- **Microservicio:** mxp.otherclient  
- **Rol principal:** gestión de autenticación de clientes externos  
- **Función crítica:** asegurar el acceso seguro mediante tokens  

## 3. Arquitectura y componentes

### 3.1 Capa de fachada
| Componente                | Código         | Descripción                                                                 |
|---------------------------|---------------|-----------------------------------------------------------------------------|
| Interfaz REST Token       | OTHERCLIENT-001| Punto de entrada para solicitudes de token de clientes externos. Valida las credenciales y genera el token. |

### 3.2 Validaciones
- **ClientId válido:** verifica que el clientId exista y sea válido
- **ClientSecret válido:** valida que el clientSecret corresponda al clientId
- **Formato correcto:** ambos campos deben estar presentes y ser cadenas no vacías

## 4. Gestión de procesos
### Flujo de trabajo estándar
1. **Recepción** → Se recibe la solicitud POST en `/api/auth/token`.  
2. **Validación** → Se verifica la estructura y los campos obligatorios.  
3. **Verificación de credenciales** → Se comprueba que el clientId y clientSecret sean correctos.  
4. **Generación de token** → Se crea y retorna un token de acceso seguro.  
5. **Respuesta** → Se confirma la autenticación exitosa y se entrega el token.

## 5. Puntos de integración
- **Entrada:** credenciales de cliente desde sistemas externos.  
- **Salida:** token de autenticación para uso en otros endpoints protegidos.  
- **Rol central:** garantizar el acceso seguro de clientes externos.

## 6. Patrones de operación
- **Validación estricta:** ambos campos son obligatorios.  
- **Seguridad:** los tokens se generan de forma segura y tienen tiempo de expiración.  
- **Respuestas claras:** mensajes de error descriptivos.

## 7. Escenarios de uso
✅ **Caso exitoso:**  
Solicitud POST `/api/auth/token` con clientId y clientSecret válidos → validación → generación de token → respuesta de éxito.

⚠️ **Posibles fallos:**  
- clientId o clientSecret incorrectos → error de autenticación.  
- Datos faltantes → error de validación.

## 8. Endpoints
### Obtener Token de API
- **Método:** POST  
- **URL:** `http://135.232.49.215/api/auth/token`  
- **Headers:**
  ```http
  Content-Type: application/json
  ```
- **Body:**
```json
{
    "clientId": "f2ffebd1-77eb-49d2-a623-99824b1f485d",
    "clientSecret": "c0f2afed-edc3-4cec-8af0-ef58fcb50062"
}
```

#### Parámetros del Body
- `clientId`: Identificador único del cliente externo (obligatorio)
- `clientSecret`: Clave secreta del cliente externo (obligatorio)

### Respuesta Exitosa (200)
```json
{
    "code": 200,
    "status": "success",
    "message": "Token generado exitosamente",
    "data": {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
        "expire_in": "2025-09-19 23:59:59"
    }
}
```

#### Campos de la Respuesta Exitosa
| Campo         | Tipo     | Descripción                                 |
|---------------|----------|---------------------------------------------|
| code          | integer  | Código de estado HTTP                       |
| status        | string   | Estado de la operación                      |
| message       | string   | Mensaje descriptivo                         |
| data.token    | string   | Token JWT generado                          |
| data.expire_in| string   | Fecha y hora de expiración del token (UTC)  |

#### Respuesta con Error (400)
```json
{
    "code": 400,
    "status": "error",
    "message": "Error en la validación de datos",
    "errors": [
        "El campo 'clientId' es requerido",
        "El campo 'clientSecret' es requerido"
    ]
}
```

#### Respuesta con Error (401)
```json
{
    "code": 401,
    "status": "error",
    "message": "Credenciales inválidas"
}
```

## 9. Preguntas frecuentes (Soporte IA)
Q: ¿Por cuánto tiempo es válido el token?  
A: El token tiene una fecha de expiración incluida en el campo `expire_in`.  

Q: ¿Puedo usar el mismo token varias veces?  
A: Sí, mientras el token no haya expirado.  

Q: ¿Qué hago si pierdo mi clientSecret?  
A: Solicita la regeneración de credenciales al administrador del sistema.

## 10. Referencias
- Políticas de seguridad de autenticación  
- Estándares de generación de tokens JWT  
- Documentación técnica de integración de clientes externos

## 11. Contacto
- **Equipo responsable:** Seguridad y Autenticación  
- **Canal de soporte:** `#mxp-soporte` (Slack interno)  
- **Escalamiento:** Equipo de Seguridad → Administradores de Sistema