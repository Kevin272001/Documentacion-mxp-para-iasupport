# Módulo de Clientes Legacy (MXPI-legacy-clients)

## 1. Propósito y alcance
El módulo **MXPI-legacy-clients** gestiona la autenticación y el acceso de clientes legacy dentro del ecosistema **MAXPOINT V2.0**.  
Permite validar credenciales de clientes antiguos y generar tokens que les permitan interactuar con otros microservicios de manera segura y controlada.  

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- MXP Init  
- MXP FE Build  
- Impresora MXP 2  

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.legacy-clients`  
- **Rol principal:** Gestionar autenticación y validación de clientes legacy.  
- **Función crítica:** Garantizar que solo clientes válidos puedan acceder a los servicios protegidos.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente                     | Código | Descripción                                                                 |
|--------------------------------|--------|-----------------------------------------------------------------------------|
| Endpoint de autenticación       | T001   | Recibe `clientID` y `clientSecret` y devuelve datos validados.             |

### 3.2. Capa de validación
| Componente                     | Código | Descripción                                                                 |
|--------------------------------|--------|-----------------------------------------------------------------------------|
| Validación de credenciales      | V030   | Verifica la estructura y el formato de `clientID` y `clientSecret`.        |
| Validación UUID                 | V031   | Asegura que `clientID` y `clientSecret` cumplen formato UUID.               |

---

## 4. Gestión de procesos

### Flujo de trabajo estándar
1. **Solicitud de token** → El endpoint `/api/auth/token` recibe `clientID` y `clientSecret`.  
2. **Validación** → Se verifica que los campos existan y tengan formato UUID.  
3. **Autenticación** → Si la validación es correcta, el sistema retorna los mismos valores como confirmación.  
4. **Distribución** → Los datos validados se devuelven al cliente para uso en integración con otros servicios.  

---

## 5. Endpoints disponibles

### 📌 Generar token para cliente legacy
**Método:** `POST`  
**URL:** `{{server_clients_legacy}}/api/auth/token`

#### Request Body
```json
{
  "clientID": "f2ffebd1-77eb-49d2-a623-99824b1f485d",
  "clientSecret": "c0f2afed-edc3-4cec-8af0-ef58fcb50062"
}
```

#### Validaciones principales
- `clientID` y `clientSecret` deben estar presentes y no ser vacíos.  
- Ambos deben cumplir el formato UUID.  
- Código HTTP esperado: 200 o 201.  

#### Response esperado
```json
{
  "clientID": "f2ffebd1-77eb-49d2-a623-99824b1f485d",
  "clientSecret": "c0f2afed-edc3-4cec-8af0-ef58fcb50062"
}
```

#### Validaciones post-response (Postman)
- Request contiene `clientID` y `clientSecret` válidos.  
- Response devuelve los mismos valores que el request.  
- Código HTTP dentro de los permitidos (200 o 201).  

---

## 6. Puntos de integración
El microservicio **MXPI-legacy-clients** asegura que los clientes legacy tengan acceso controlado a los servicios:  
- **Entrada:** `clientID` y `clientSecret`.  
- **Salida:** Confirmación de credenciales válidas y token de acceso (si aplica).  
- **Rol central:** Mantener seguridad y autenticación de clientes antiguos.  

---

## 7. Escenarios de uso

### ✅ Caso exitoso
- Cliente envía `clientID` y `clientSecret` válidos → respuesta con mismos valores → acceso permitido.  

### ⚠️ Posibles fallos
- **Campos faltantes o vacíos:** Se rechaza la solicitud.  
- **Formato UUID inválido:** No se valida el cliente.  
- **Código HTTP distinto de 200/201:** Indica error en autenticación.  

---

## 8. Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si el UUID de `clientID` es incorrecto?**  
A: La autenticación falla y no se genera token.  

**Q: ¿Puedo reutilizar el token generado por este endpoint?**  
A: Este endpoint solo valida credenciales y devuelve confirmación; el token real dependerá de otros servicios de integración.  

**Q: ¿Qué valores se devuelven en el response?**  
A: Se devuelven los mismos `clientID` y `clientSecret` enviados en el request como confirmación.  

---

## 9. Referencias
- **Fuentes internas:**  
  - `mxpi-legacy-clients/endpoints/api/auth/token`  

- **Microservicio:** `mxpv2.integration.legacy-clients`  

- **Contacto:**  
  - **Equipo responsable:** Integraciones MAXPOINT  
  - **Canal de soporte:** #mxp-integraciones (Slack interno)  
  - **Escalamiento:** Arquitectura de datos → Liderazgo técnico
