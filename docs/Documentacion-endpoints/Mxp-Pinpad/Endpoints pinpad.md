
# Módulo de Autenticación Pinpad (MXP-Pinpad)

## 1. Propósito y alcance
El módulo **MXP-Pinpad** gestiona la autenticación de dispositivos de pago (pinpads) dentro del ecosistema **MAXPOINT V2.0**.  

- Permite iniciar sesión en el pinpad para habilitar transacciones seguras.  
- Garantiza que solo dispositivos autorizados puedan integrarse al flujo de ventas POS.  
- Establece la sesión inicial para operaciones de débito/crédito.  

ℹ️ Para más detalles de módulos relacionados, consulta:  

- MXP POS  
- MXP Payments  
- MXP Configuration  

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.pinpad`  
- **Rol principal:** gestionar autenticación y sesión de pinpads.  
- **Función crítica:** habilitar dispositivos de pago en el ecosistema de ventas.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de autenticación
| Componente                    | Código | Descripción |
|-------------------------------|--------|-------------|
| API de login de pinpad        | P301   | Permite autenticar el pinpad y generar sesión activa. |

🔑 El **P301** asegura que cada dispositivo esté autorizado antes de iniciar operaciones financieras.  

---

## 4. Gestión de procesos

**Flujo de trabajo estándar:**
1. Solicitud de login → El pinpad envía credenciales o petición de sesión.  
2. Validación → El sistema valida el dispositivo y permisos.  
3. Autenticación → Se genera una sesión activa para operaciones financieras.  
4. Respuesta → Se retorna confirmación de login o error de autenticación.  

---

## 5. Puntos de integración
El pinpad conecta el POS con los servicios de pago:  

- **Entrada:** solicitudes directas del pinpad.  
- **Salida:** confirmación de sesión válida para operar transacciones.  
- **Rol central:** habilitar la comunicación segura entre POS y medios de pago.  

---

## 6. Endpoints documentados

### 6.1 Login – Pinpad  

- **Método:** `POST`  
- **URL:**  
  ```
  http://192.168.101.96:7165/login
  ```  

**Body (raw – JSON):**  
Actualmente no definido en la documentación recibida.  
👉 Recomendación: incluir campos mínimos como `device_id`, `user`, `password` o `token`.  

**Ejemplo de Request (cURL):**
```bash
curl --location --request POST 'http://192.168.101.96:7165/login'
```

**Notas:**  
- Se espera que el servicio retorne un token de sesión o confirmación de login.  
- Es recomendable manejar errores 401/403 en caso de credenciales inválidas.  

---

## 7. Escenarios de uso

✅ **Caso exitoso**  
- Login correcto → validación de dispositivo → sesión activa habilitada.  

⚠️ **Posibles fallos**  
- Credenciales inválidas → `401 Unauthorized`.  
- Dispositivo no autorizado → `403 Forbidden`.  
- Servicio no disponible → `500 Internal Server Error`.  

---

## 8. Preguntas frecuentes (para IA Support)

- **Q:** ¿El login persiste la sesión?  
  **A:** Sí, hasta que expire o se cierre manualmente.  

- **Q:** ¿Se puede reutilizar el mismo login en múltiples dispositivos?  
  **A:** No, cada pinpad debe autenticarse individualmente.  

- **Q:** ¿Qué pasa si el pinpad pierde la conexión durante la sesión?  
  **A:** El login expira y se debe volver a autenticar.  

---

## 9. Referencias
Fuentes internas:  
- `mxp-pinpad/login/readme.md`  
- `mxp-payments/authentication/readme.md`  
- API Local: `192.168.101.96:7165`  

---

## 10. Contacto
- **Equipo responsable:** Integraciones MAXPOINT – Pinpad  
- **Canal de soporte:** #mxp-pinpad (Slack interno)  
- **Escalamiento:** Arquitectura de pagos → Liderazgo técnico  
