---
id: auth-register
sidebar_label: Registro de Usuario
---

# Módulo de Autenticación - Registro de Usuario

## 1. Propósito y alcance
El módulo **Registro de Usuario** permite la creación de nuevas cuentas de usuario en el sistema MAXPOINT V2.0.  
Gestiona el proceso de registro, validación de datos y creación de credenciales de acceso.  
Garantiza la seguridad en la creación de cuentas mediante validación de contraseñas y correos electrónicos únicos.

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- Autenticación  
- Gestión de Usuarios  
- Recuperación de Contraseña  

## 2. Descripción general del sistema
- **Microservicio:** mxpv2.authentication  
- **Rol principal:** gestión de registro de nuevos usuarios  
- **Función crítica:** asegurar la creación segura de cuentas de usuario  

## 3. Arquitectura y componentes

### 3.1 Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz REST Register | AUTH-REG-001 | Punto de entrada para solicitudes de registro de nuevos usuarios. Valida estructura e integridad de los datos. |

### 3.2 Validaciones
- **Usuario único:** verifica que no exista otro usuario con el mismo nombre
- **Email único:** valida que el correo electrónico no esté registrado
- **Fortaleza de contraseña:** verifica complejidad mínima
- **Coincidencia de contraseñas:** confirma que ambas contraseñas sean idénticas

## 4. Gestión de procesos
### Flujo de trabajo estándar
1. **Recepción** → Se recibe la solicitud POST en `/register`.  
2. **Validación** → Se verifica la estructura y los campos obligatorios.  
3. **Verificación de unicidad** → Se comprueba que no exista un usuario con el mismo email.  
4. **Creación de usuario** → Se almacena la información del nuevo usuario.  
5. **Respuesta** → Se confirma la creación exitosa del usuario.  

## 5. Puntos de integración
- **Entrada:** formulario de registro desde frontend.  
- **Salida:** confirmación de creación de usuario.  
- **Rol central:** garantizar la creación segura de cuentas de usuario.  

## 6. Patrones de operación
- **Validación estricta:** todos los campos son obligatorios.  
- **Seguridad:** las contraseñas se almacenan de forma encriptada.  
- **Respuestas claras:** mensajes de error descriptivos.  

## 7. Escenarios de uso
✅ **Caso exitoso:**  
Solicitud POST `/register` con datos correctos → validación → creación de usuario → respuesta de éxito.  

⚠️ **Posibles fallos:**  
- Email ya registrado → error de validación.  
- Contraseñas no coinciden → error de validación.  
- Datos faltantes → error de validación.  

## 8. Endpoints
### Registrar Nuevo Usuario
- **Método:** POST  
- **URL:** `/register`  
- **Headers:**
  ```http
  Content-Type: application/json
  ```
- **Body:**
```json
{
    "username": "MXPv2 Fe.Builder",
    "email": "mxpv2.fe.builder@kfc.com.ec",
    "password": "Maker*4033",
    "confirm_password": "Maker*4033"
}
```

#### Parámetros del Body
- `username`: Nombre completo del usuario (obligatorio)
- `email`: Correo electrónico del usuario (obligatorio, debe ser único)
- `password`: Contraseña del usuario (obligatorio, mínimo 8 caracteres)
- `confirm_password`: Confirmación de contraseña (debe coincidir con password)

### Respuesta Exitosa (200)
```json
{
    "data": {
        "username": "MXPv2 Fe.Builder",
        "email": "mxpv2.fe.builder@kfc.com.ec"
    },
    "msg": "Usuario registrado exitosamente"
}
```

### Respuesta de Error (400) - Email ya registrado
```json
{
    "code": 400,
    "status": "failed",
    "alert": "Error al registrar usuario",
    "messages": [
        "El correo electrónico ya está registrado"
    ]
}
```

### Respuesta de Error (400) - Contraseñas no coinciden
```json
{
    "code": 400,
    "status": "failed",
    "alert": "Error de validación",
    "messages": [
        "Las contraseñas no coinciden"
    ]
}
```

## 9. Preguntas frecuentes (Soporte IA)
Q: ¿Qué requisitos debe cumplir la contraseña?  
A: La contraseña debe tener al menos 8 caracteres, incluyendo mayúsculas, minúsculas, números y caracteres especiales.  

Q: ¿Puedo registrar un correo que ya existe?  
A: No, cada correo electrónico debe ser único en el sistema.  

Q: ¿Qué hago si olvido mi contraseña?  
A: Utiliza el módulo de recuperación de contraseña con tu correo electrónico registrado.  

## 10. Referencias
- Políticas de seguridad de contraseñas  
- Estándares de validación de correos electrónicos  
- Documentación técnica de autenticación  

## 11. Contacto
- **Equipo responsable:** Seguridad y Autenticación  
- **Canal de soporte:** `#mxp-soporte` (Slack interno)  
- **Escalamiento:** Equipo de Seguridad → Administradores de Sistema
