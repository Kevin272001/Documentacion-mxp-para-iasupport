# Endpoints de Clientes (MXPi-Clients)

## 1. Propósito y alcance

MXPi-Clients es el módulo encargado de la **gestión de autenticación y emisión de tokens** dentro del ecosistema MAXPOINT V2.0.

* Permite la validación de credenciales de clientes.
* Genera y entrega tokens seguros para el acceso a otros microservicios.
* Garantiza que solo clientes autorizados puedan interactuar con el sistema.

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Auth
* MXP Server
* MXP Orchestrator

---

## 2. Descripción general del sistema

**Microservicio:** mxpv2.integration.clients  
**Rol principal:** gestionar autenticación de clientes.  
**Función crítica:** validar credenciales (`clientId`, `clientSecret`) y entregar un token válido para la interacción con el ecosistema.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                   | Código | Descripción                                                                                   |
| ---------------------------- | ------ | --------------------------------------------------------------------------------------------- |
| Interfaz de autenticación    | C-FH01 | Punto de entrada para solicitudes de generación de token. 🔑 Valida credenciales de clientes. |

### 3.2. Capa de procesamiento

| Componente                 | Código | Descripción                                                                                     |
| -------------------------- | ------ | ----------------------------------------------------------------------------------------------- |
| Servicio de autenticación  | C-AU01 | Valida `clientId` y `clientSecret`, genera el token y define su tiempo de expiración (`expire`). |

---

## 4. Gestión de procesos

Flujo de trabajo estándar:

1. **Solicitud de token:** Cliente envía POST con sus credenciales.
2. **Validación:** El sistema valida `clientId` y `clientSecret`.
3. **Generación de token:** Se crea un JWT/token único con fecha de expiración.
4. **Respuesta:** El token se entrega al cliente junto con la fecha de expiración.

---

## 5. Puntos de integración

El módulo de clientes interactúa con todo el ecosistema MAXPOINT V2.0:

* **Entrada:** credenciales de cliente (`clientId`, `clientSecret`).
* **Salida:** token válido con tiempo de expiración.
* **Rol central:** asegurar que las interacciones estén restringidas a clientes autorizados.

---

## 6. Patrones de gestión

* **Recepción centralizada:** todas las solicitudes pasan por C-FH01.
* **Procesamiento seguro:** la validación de credenciales se maneja por C-AU01.
* **Distribución controlada:** los tokens generados se entregan solo a clientes válidos.

---

## 7. Escenarios de uso

✅ Caso exitoso:

* Cliente envía credenciales correctas → se valida → se genera token → token entregado con tiempo de expiración.

⚠️ Posibles fallos:

* **Error en validación:** `clientId` o `clientSecret` inválidos. El sistema rechaza la solicitud y no genera token.
* **Error interno:** falla en la generación del token o almacenamiento temporal. Se devuelve mensaje de error.

---

## 8. Endpoints

### 8.1. GetToken

**Method:** POST  

```
{{server_client}}/api/v1/auth/token
```

**Body (raw):**

```json
{
    "clientId": "f2ffebd1-77eb-49d2-a623-99824b1f485d",
    "clientSecret": "c0f2afed-edc3-4cec-8af0-ef58fcb50062"
}
```

**Response example:**

<details>
<summary>201 Created</summary>

```json
{
    "code": 21,
    "messages": [
        "Creado Correctamente"
    ],
    "data": {
        "token": "fcodifjdsfijsdiow4j5io435o43j534i50963490592049034...",
        "expire_in": "2017-07-23 13:10:11"
    }
}
```
</details>

---

## 9. Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si envío credenciales incorrectas?**  
A: La solicitud será rechazada con un mensaje de error en `messages`. El token no será generado.

**Q: ¿Cómo sé si el token se generó correctamente?**  
A: Revisa el código de respuesta `code: 21` y el mensaje "Creado Correctamente". Además, se devolverá el token y la fecha de expiración.

**Q: ¿El token tiene tiempo de expiración?**  
A: Sí, en la respuesta se incluye `expire_in` indicando hasta cuándo es válido.

**Q: ¿Qué ocurre si mi token expira?**  
A: El cliente debe volver a solicitar un nuevo token enviando `clientId` y `clientSecret`.

**Q: ¿Puedo reutilizar el token en diferentes módulos?**  
A: Sí, mientras el token esté vigente, puede usarse para interactuar con otros microservicios de MAXPOINT V2.0.

---

## 10. Referencias

Fuentes internas:

* mxp-clients/process/readme.md (1-20)  
* Microservicio: mxpv2.integration.clients  

---

## 11. Contacto

**Equipo responsable:** Integraciones MAXPOINT  
**Canal de soporte:** #mxp-integraciones (Slack interno)  
**Escalamiento:** Arquitectura de datos → Liderazgo técnico  
