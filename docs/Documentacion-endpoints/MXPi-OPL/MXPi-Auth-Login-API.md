---
id: auth-login
sidebar_label: Inicio de Sesión
---

# Módulo de Autenticación - Inicio de Sesión

## 1. Propósito y alcance
El módulo **Inicio de Sesión** permite a los usuarios autenticarse en el sistema MAXPOINT V2.0.  
Gestiona la validación de credenciales y la generación de tokens de acceso.  
Proporciona seguridad mediante la generación de tokens JWT para autorización en solicitudes posteriores.

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- Registro de Usuario  
- Gestión de Tokens  
- Recuperación de Contraseña  

## 2. Descripción general del sistema
- **Microservicio:** mxpv2.authentication  
- **Rol principal:** autenticación de usuarios  
- **Función crítica:** garantizar el acceso seguro al sistema  

## 3. Arquitectura y componentes

### 3.1 Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz REST Login | AUTH-LOGIN-001 | Punto de entrada para solicitudes de autenticación. Valida credenciales y genera tokens. |

### 3.2 Proceso de autenticación
- **Validación de credenciales:** verifica usuario y contraseña
- **Generación de token JWT:** crea un token de acceso seguro
- **Respuesta segura:** devuelve el token para autorización

## 4. Gestión de procesos
### Flujo de trabajo estándar
1. **Recepción** → Se recibe la solicitud POST en `/login`.  
2. **Validación** → Se verifica la estructura del JSON y campos obligatorios.  
3. **Autenticación** → Se validan las credenciales contra la base de datos.  
4. **Generación de token** → Se crea un token JWT para el usuario autenticado.  
5. **Respuesta** → Se devuelve el token de acceso.  

## 5. Puntos de integración
- **Entrada:** credenciales de usuario (email y contraseña).  
- **Salida:** token JWT para autorización.  
- **Rol central:** garantizar el acceso seguro al sistema.  

## 6. Patrones de operación
- **Validación estricta:** credenciales obligatorias.  
- **Seguridad:** las contraseñas se verifican de forma segura.  
- **Tokens JWT:** con tiempo de expiración para mayor seguridad.  

## 7. Escenarios de uso
✅ **Caso exitoso:**  
Solicitud POST `/login` con credenciales correctas → validación → generación de token → respuesta con token.  

⚠️ **Posibles fallos:**  
- Credenciales incorrectas → error de autenticación.  
- Usuario no encontrado → error de autenticación.  
- Datos faltantes → error de validación.  

## 8. Endpoints
### Iniciar Sesión
- **Método:** POST  
- **URL:** `/login`  
- **Headers:**
  ```http
  Content-Type: application/json
  ```
- **Body:**
```json
{
    "email": "mxpv2.fe.builder@kfc.com.ec",
    "password": "Maker*4033"
}
```

#### Parámetros del Body
- `email`: Correo electrónico del usuario (obligatorio)
- `password`: Contraseña del usuario (obligatorio)

### Respuesta Exitosa (200)
```json
{
    "data": {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2Nzg2OWRiOWI4YTIxMjQ1ODNhZGYzYjAiLCJ1c2VyX2lkIjoiNjc4NjlkYjliOGEyMTI0NTgzYWRmM2IwIiwiZW1haWwiOiJkeWxhbkBnbWFpbC5jb20iLCJleHAiOjE3MzY5NjE4NjV9.bkweXdeQ4JRoBAamkbDEBgrdEadvvJIupXI_cO8ktKo"
    },
    "msg": "Ok"
}
```

### Respuesta de Error (401) - Credenciales inválidas
```json
{
    "code": 401,
    "status": "failed",
    "alert": "Error de autenticación",
    "messages": [
        "Credenciales inválidas"
    ]
}
```

### Respuesta de Error (400) - Datos faltantes
```json
{
    "code": 400,
    "status": "failed",
    "alert": "Error de validación",
    "messages": [
        "El campo 'email' es requerido"
    ]
}
```

## 9. Preguntas frecuentes (Soporte IA)
Q: ¿Por cuánto tiempo es válido el token JWT?  
A: El token JWT tiene un tiempo de expiración configurado, típicamente 24 horas.  

Q: ¿Qué hago si olvidé mi contraseña?  
A: Utiliza el módulo de recuperación de contraseña con tu correo electrónico registrado.  

Q: ¿Puedo usar el token en múltiples dispositivos?  
A: Sí, el token puede usarse en múltiples dispositivos hasta su expiración.  

## 10. Referencias
- Documentación de JWT (JSON Web Tokens)  
- Políticas de seguridad de autenticación  
- Guía de implementación de OAuth 2.0  

## 11. Contacto
- **Equipo responsable:** Seguridad y Autenticación  
- **Canal de soporte:** `#mxp-soporte` (Slack interno)  
- **Escalamiento:** Equipo de Seguridad → Administradores de Sistema
