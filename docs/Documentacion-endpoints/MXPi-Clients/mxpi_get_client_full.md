# Endpoints de Clientes (MXPi-Clients)

## 1. Propósito y alcance

MXPi-Clients es el módulo encargado de la **gestión de información de clientes** dentro del ecosistema MAXPOINT V2.0.

* Permite la consulta de datos de clientes a través de parámetros como DNI, tipo y país.
* Facilita la integración de microservicios que requieren validar o recuperar información de clientes registrados.
* Garantiza que solo usuarios autenticados puedan acceder a la información.

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Auth  
* MXP Server  
* MXP Orchestrator  

---

## 2. Descripción general del sistema

**Microservicio:** mxpv2.integration.clients  
**Rol principal:** recuperar información de clientes.  
**Función crítica:** exponer datos de clientes de manera segura mediante autenticación con token.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente               | Código | Descripción                                                                 |
|---------------------------|--------|-----------------------------------------------------------------------------|
| Interfaz de consulta      | C-FH02 | Punto de entrada para solicitudes GET de información de cliente.            |

### 3.2. Capa de procesamiento

| Componente              | Código | Descripción                                                                  |
|--------------------------|--------|------------------------------------------------------------------------------|
| Servicio de clientes     | C-CL01 | Recibe parámetros (`dni`, `type`, `country`), valida token y obtiene datos. |

---

## 4. Gestión de procesos

Flujo de trabajo estándar:

1. **Solicitud de datos:** Cliente envía GET con parámetros (`dni`, `type`, `country`) y token de autenticación.  
2. **Validación:** El sistema valida el token de autorización.  
3. **Procesamiento:** Se buscan los datos del cliente en la base correspondiente.  
4. **Respuesta:** Se devuelve la información del cliente o un mensaje indicando que no fue encontrado.  

---

## 5. Puntos de integración

El módulo de clientes interactúa con todo el ecosistema MAXPOINT V2.0:

* **Entrada:** parámetros de búsqueda (`dni`, `type`, `country`) y token de autenticación.  
* **Salida:** datos del cliente (documento, nombres, apellidos, correo, teléfono).  
* **Rol central:** proveer información confiable de clientes a otros microservicios.  

---

## 6. Patrones de gestión

* **Recepción centralizada:** todas las solicitudes pasan por C-FH02.  
* **Procesamiento seguro:** los parámetros se validan y se protege la información con token.  
* **Distribución controlada:** solo clientes autenticados obtienen resultados.  

---

## 7. Escenarios de uso

✅ Caso exitoso:  
* Cliente envía token válido y parámetros correctos → se valida → se recuperan los datos del cliente → respuesta con información completa.  

⚠️ Posibles fallos:  
* **Token inválido o ausente:** la solicitud será rechazada.  
* **Cliente no encontrado:** se devuelve un `code: 24` indicando que no fue encontrado.  
* **Error interno:** problemas en la búsqueda o acceso a la base de datos.  

---

## 8. Endpoints

### 8.1. GetClient

**Method:** GET  

```
{{server_client}}/api/v1/client/getData?dni=0804724672&type=1&country=ECU
```

**Query Params:**

| Param   | Value       |
|---------|-------------|
| dni     | 0804724672  |
| type    | 1           |
| country | ECU         |

**🔑 Authentication bearer:**

| Param | Value                   | Type   |
|-------|-------------------------|--------|
| token | {{server_client_token}} | string |

**Response examples:**  

<details>
<summary>200 OK</summary>

```json
{
    "code": 20,
    "messages": [
        "Encontrado Correctamente"
    ],
    "data": {
        "documento": "0950040279",
        "tipoDocumento": "CEDULA",
        "email": "lcoronel@gmail.com",
        "telefono": "0969458949",
        "primerNombre": "CEDULA LUIS",
        "segundoNombre": "P2",
        "apellidos": "CORONEL BENEFICIO VACIO 2"
    }
}
```
</details>

<details>
<summary>204 No Content</summary>

```json
{
    "code": 24,
    "messages": [
        "No Encontrado Correctamente"
    ],
    "data": {}
}
```
</details>

---

## 9. Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si envío un token inválido?**  
A: La solicitud será rechazada y no se retornará información.  

**Q: ¿Qué ocurre si el cliente no existe en el sistema?**  
A: El servicio devolverá un `code: 24` con mensaje "No Encontrado Correctamente".  

**Q: ¿Se pueden realizar múltiples consultas con el mismo token?**  
A: Sí, siempre y cuando el token no haya expirado.  

**Q: ¿El servicio permite consultar con diferentes tipos de documento?**  
A: Sí, el parámetro `type` define el tipo de documento utilizado en la búsqueda.  

**Q: ¿Qué debo hacer si recibo un error interno?**  
A: Contactar al equipo de integraciones MAXPOINT para escalar el problema.  

---

## 10. Referencias

Fuentes internas:  

* mxp-clients/process/readme.md (21-40)  
* Microservicio: mxpv2.integration.clients  

---

## 11. Contacto

**Equipo responsable:** Integraciones MAXPOINT  
**Canal de soporte:** #mxp-integraciones (Slack interno)  
**Escalamiento:** Arquitectura de datos → Liderazgo técnico  
