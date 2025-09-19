# Endpoints de Clientes (MXPi-Clients)

## 1. Propósito y alcance

MXPi-Clients es el módulo encargado de la **gestión de información de clientes** dentro del ecosistema MAXPOINT V2.0.

* Permite la creación de nuevos clientes en el sistema.
* Facilita la integración de microservicios que requieran registrar nuevos clientes.
* Garantiza que la información del cliente sea almacenada de manera segura.

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Auth  
* MXP Server  
* MXP Orchestrator  

---

## 2. Descripción general del sistema

**Microservicio:** mxpv2.integration.clients  
**Rol principal:** gestión de creación de clientes.  
**Función crítica:** registrar nuevos clientes de manera segura mediante autenticación.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente               | Código | Descripción                                                                 |
|--------------------------|--------|-----------------------------------------------------------------------------|
| Interfaz de creación     | C-FH01 | Punto de entrada para solicitudes POST de creación de cliente.              |

### 3.2. Capa de procesamiento

| Componente              | Código | Descripción                                                                 |
|-------------------------|--------|-----------------------------------------------------------------------------|
| Servicio de clientes    | C-CL02 | Recibe los datos del cliente, valida la información y la almacena.         |

---

## 4. Gestión de procesos

Flujo de trabajo estándar:

1. **Solicitud de creación:** Cliente envía POST con los datos del cliente.  
2. **Validación:** El sistema valida los campos obligatorios y formatos.  
3. **Procesamiento:** Se crea el nuevo registro de cliente en la base de datos.  
4. **Respuesta:** Se devuelve la información del cliente creado o mensaje de error.  

---

## 5. Puntos de integración

El módulo de clientes interactúa con todo el ecosistema MAXPOINT V2.0:

* **Entrada:** datos del cliente (documento, nombres, apellidos, correo, teléfono, etc.).  
* **Salida:** confirmación de creación con datos del cliente.  
* **Rol central:** registro seguro de nuevos clientes en el sistema.  

---

## 6. Patrones de gestión

* **Validación de datos:** todos los campos obligatorios son validados.  
* **Procesamiento seguro:** los datos sensibles son manejados de forma segura.  
* **Respuesta estandarizada:** formato consistente para éxito y errores.  

---

## 7. Escenarios de uso

✅ Caso exitoso:  
* Cliente envía datos completos y válidos → se valida → se crea el cliente → respuesta con código 201.  

⚠️ Posibles fallos:  
* **Datos incompletos o inválidos:** la solicitud será rechazada con código 200 y mensaje de error.  
* **Error en el servidor:** problemas al crear el registro en la base de datos.  

---

## 8. Endpoints

### 8.1. CreateClient

**Method:** POST  

```
{{server_client}}/api/v1/client/create
```

**Headers:**
```
Content-Type: application/json
```

**Body:**
```json
{
    "cdn_id": 10,
    "pais": "ECU",
    "sistemaOrigen": "LandingPage",
    "fechaCreacion": "2023-10-24T13:49:44Z",
    "autenticacion": true,
    "documento": "0950040279",
    "tipoDocumento": "CEDULA",
    "email": "lcoronel@gmail.com",
    "telefono": "0969458949",
    "primerNombre": "CEDULA LUIS",
    "segundoNombre": "P2",
    "apellidos": "CORONEL BENEFICIO VACIO 2",
    "fechaNacimiento": "1990-02-01T05:00:00Z",
    "mayor16anios": true,
    "aceptacionPoliticas": true,
    "envioComunicacionesComerciales": true,
    "envioComunicacionesComercialesPush": true,
    "analisisDeDatosPerfiles": true,
    "cesionDatosATercerosNacionales": true,
    "cesionDatosATercerosInternacionales": true,
    "fechaAceptoPrivacidad": "2023-06-05T19:02:10Z",
    "fechaActualizacion": "2023-10-24T00:00:00Z"
}
```

**Response examples:**  

<details>
<summary>201 Created</summary>

```json
{
    "code": 21,
    "messages": [
        "Creado Correctamente"
    ],
    "data": {
        "id": "23452343-234324-2344jsdhfurjfdh",
        "cdn_id": 10,
        "pais": "ECU",
        "sistemaOrigen": "LandingPage",
        "fechaCreacion": "2023-10-24T13:49:44Z",
        "autenticacion": true,
        "documento": "0950040279",
        "tipoDocumento": "CEDULA",
        "email": "lcoronel@gmail.com",
        "telefono": "0969458949",
        "primerNombre": "CEDULA LUIS",
        "segundoNombre": "P2",
        "apellidos": "CORONEL BENEFICIO VACIO 2",
        "fechaNacimiento": "1990-02-01T05:00:00Z",
        "mayor16anios": true,
        "aceptacionPoliticas": true,
        "envioComunicacionesComerciales": true,
        "envioComunicacionesComercialesPush": true,
        "analisisDeDatosPerfiles": true,
        "cesionDatosATercerosNacionales": true,
        "cesionDatosATercerosInternacionales": true,
        "fechaAceptoPrivacidad": "2023-06-05T19:02:10Z",
        "fechaActualizacion": "2023-10-24T00:00:00Z"
    }
}
```
</details>

<details>
<summary>200 OK (Error en validación)</summary>

```json
{
    "code": 20,
    "messages": [
        "no Creado Correctamente",
        "falta el campo fgfgf"
    ],
    "data": {
        "cdn_id": 10,
        "pais": "ECU",
        "sistemaOrigen": "LandingPage",
        "fechaCreacion": "2023-10-24T13:49:44Z",
        "autenticacion": true,
        "documento": "0950040279",
        "tipoDocumento": "CEDULA",
        "email": "lcoronel@gmail.com",
        "telefono": "0969458949",
        "primerNombre": "CEDULA LUIS",
        "segundoNombre": "P2",
        "apellidos": "CORONEL BENEFICIO VACIO 2",
        "fechaNacimiento": "1990-02-01T05:00:00Z",
        "mayor16anios": true,
        "aceptacionPoliticas": true,
        "envioComunicacionesComerciales": true,
        "envioComunicacionesComercialesPush": true,
        "analisisDeDatosPerfiles": true,
        "cesionDatosATercerosNacionales": true,
        "cesionDatosATercerosInternacionales": true,
        "fechaAceptoPrivacidad": "2023-06-05T19:02:10Z",
        "fechaActualizacion": "2023-10-24T00:00:00Z"
    }
}
```
</details>

---

## 9. Preguntas frecuentes (para IA Support)

**Q: ¿Qué campos son obligatorios para crear un cliente?**  
A: Todos los campos mostrados en el ejemplo son requeridos para la creación exitosa de un cliente.

**Q: ¿Qué formato debo usar para las fechas?**  
A: Las fechas deben estar en formato ISO 8601 (ej: "2023-10-24T13:49:44Z").

**Q: ¿Qué significa el código de respuesta 21?**  
A: El código 21 indica que el cliente fue creado exitosamente.

**Q: ¿Qué hago si recibo un código 20?**  
A: El código 20 indica un error en la validación. Revisa los mensajes para identificar los campos con problemas.

**Q: ¿Se puede crear un cliente sin aceptar las políticas?**  
A: No, el campo "aceptacionPoliticas" debe ser true para crear un cliente.

---

## 10. Referencias

Fuentes internas:  

* mxp-clients/process/readme.md  
* Microservicio: mxpv2.integration.clients  

---

## 11. Contacto

**Equipo responsable:** Integraciones MAXPOINT  
**Canal de soporte:** #mxp-integraciones (Slack interno)  
**Escalamiento:** Arquitectura de datos → Liderazgo técnico
