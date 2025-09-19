# Módulo de Autenticación (MXPI-auth)

## 1. Propósito y alcance
El módulo **MXPI-auth** es el sistema encargado de la autenticación y registro de usuarios dentro del ecosistema **MAXPOINT V2.0**.  
Garantiza la seguridad de acceso mediante la gestión de credenciales y emisión de tokens JWT.  
Facilita la validación de usuarios que interactúan con los microservicios del ecosistema.

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- MXP Init  
- MXP FE Build  
- Impresora MXP 2  

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.auth`  
- **Rol principal:** gestionar el registro y autenticación de usuarios.  
- **Función crítica:** permitir el acceso seguro a través de tokens JWT y validar credenciales de usuarios.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente              | Código | Descripción                                                                 |
|--------------------------|--------|-----------------------------------------------------------------------------|
| Interfaz de Autenticación | A001   | Expone endpoints públicos para registro y login de usuarios.                 |

### 3.2. Capa de validación
| Componente              | Código | Descripción                                                                 |
|--------------------------|--------|-----------------------------------------------------------------------------|
| Validación de credenciales | V010 | Verifica la estructura y contenido de los datos enviados en el request.     |
| Validación de JWT         | V011 | Comprueba la validez y estructura del token emitido.                        |

---

## 4. Gestión de procesos

### Flujo de trabajo estándar
1. **Registro de usuario** → El endpoint `/register` recibe los datos de usuario.  
2. **Validación** → Se verifican estructura, formato de email, contraseñas y confirmación.  
3. **Creación** → Se almacena el usuario en el sistema de autenticación.  
4. **Autenticación** → El endpoint `/login` recibe credenciales y genera un token JWT.  
5. **Distribución** → El token se devuelve al cliente para permitir acceso a otros módulos.  

---

## 5. Endpoints disponibles

### 📌 Registro de usuario
**Método:** `POST`  
**URL:** `{{server_auth}}/register`

#### Request Body
```json
{
  "username": "Admin System MXPv2",
  "email": "admin.system.mxpv2@kfc.com.ec",
  "password": "Maker*4033",
  "confirm_password": "Maker*4033"
}
```

#### Validaciones principales
- El body debe contener: `username`, `email`, `password`, `confirm_password`.  
- `password` y `confirm_password` deben coincidir.  
- `email` debe tener un formato válido.  

#### Response esperado
```json
{
  "data": {
    "username": "Admin System MXPv2",
    "email": "admin.system.mxpv2@kfc.com.ec"
  },
  "msg": "Ok"
}
```

---

### 📌 Login de usuario
**Método:** `POST`  
**URL:** `{{server_auth}}/login`

#### Request Body
```json
{
  "email": "admin.system.mxpv2@kfc.com.ec",
  "password": "Maker*4033"
}
```

#### Validaciones principales
- El body debe contener `email` y `password`.  
- Ambos deben ser strings no vacíos.  
- El `email` debe tener un formato válido.  

#### Response esperado
```json
{
  "data": {
    "token": "jwt_token_generado"
  },
  "msg": "Ok"
}
```

🔑 **El token JWT devuelto sigue el formato estándar:**  
`header.payload.signature`

---

## 6. Puntos de integración
El microservicio **MXPI-auth** conecta el ecosistema MAXPOINT V2.0:  
- **Entrada:** credenciales de usuarios enviados por aplicaciones cliente.  
- **Salida:** token JWT válido que permite el acceso a otros microservicios.  
- **Rol central:** control de autenticación y validación de credenciales.  

---

## 7. Escenarios de uso

### ✅ Caso exitoso
- Registro → Se envían datos válidos → usuario creado correctamente.  
- Login → Se envían credenciales correctas → token JWT generado.  

### ⚠️ Posibles fallos
- **Error en validación (registro/login):** Body incompleto o email inválido.  
- **Contraseñas no coinciden:** Registro rechazado.  
- **Credenciales incorrectas:** No se genera token.  
- **Token inválido:** El acceso a otros módulos es rechazado.  

---

## 8. Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si las contraseñas no coinciden en el registro?**  
A: El sistema rechaza la solicitud con un mensaje de error.  

**Q: ¿Cómo sé si el login fue exitoso?**  
A: Si el campo `msg` = "Ok" y `response.data.token` contiene un JWT válido.  

**Q: ¿Qué formato debe tener el token JWT?**  
A: Tres segmentos separados por puntos: `header.payload.signature`.  

**Q: ¿El módulo almacena usuarios?**  
A: Sí, solo los datos mínimos requeridos para autenticación.  

---

## 9. Referencias
- **Fuentes internas:**  
  - `mxpi-auth/endpoints/register`  
  - `mxpi-auth/endpoints/login`  

- **Microservicio:** `mxpv2.integration.auth`  

- **Contacto:**  
  - **Equipo responsable:** Integraciones MAXPOINT  
  - **Canal de soporte:** #mxp-integraciones (Slack interno)  
  - **Escalamiento:** Arquitectura de datos → Liderazgo técnico
