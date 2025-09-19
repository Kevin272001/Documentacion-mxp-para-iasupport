# Endpoints de Clientes (MXPi-Clients) - Aceptación de Beneficios

## 1. Propósito y alcance

El endpoint AcceptBenefits permite a los clientes aceptar y canjear beneficios dentro del ecosistema MAXPOINT V2.0.

* Registra la aceptación de un beneficio específico por parte de un cliente.
* Registra el canje de premios o beneficios en establecimientos asociados.
* Facilita el seguimiento de beneficios utilizados por los clientes.

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Auth  
* MXP Server  
* MXP Orchestrator  

---

## 2. Descripción general del sistema

**Microservicio:** mxpv2.integration.clients  
**Rol principal:** gestión de aceptación de beneficios.  
**Función crítica:** registrar la aceptación y canje de beneficios de manera segura.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente               | Código | Descripción                                                                 |
|--------------------------|--------|-----------------------------------------------------------------------------|
| Interfaz de beneficios  | C-FH03 | Punto de entrada para solicitudes POST de aceptación de beneficios.         |

### 3.2. Capa de procesamiento

| Componente              | Código | Descripción                                                                 |
|-------------------------|--------|-----------------------------------------------------------------------------|
| Servicio de beneficios  | C-BF01 | Gestiona la lógica de aceptación y registro de beneficios.                 |

---

## 4. Gestión de procesos

Flujo de trabajo estándar:

1. **Solicitud de canje:** Cliente o sistema envía POST con los datos del beneficio a canjear.  
2. **Validación:** El sistema valida los campos obligatorios y la existencia del cliente.  
3. **Procesamiento:** Se registra la aceptación del beneficio en el sistema.  
4. **Respuesta:** Se devuelve confirmación del registro o mensaje de error.  

---

## 5. Puntos de integración

El módulo de aceptación de beneficios interactúa con:

* **Entrada:** datos del cliente, establecimiento y beneficio a canjear.  
* **Salida:** confirmación del registro con datos del canje.  
* **Sistemas relacionados:** sistema de puntos, catálogo de beneficios, sistema de clientes.  

---

## 6. Patrones de gestión

* **Validación de datos:** todos los campos son validados antes del procesamiento.  
* **Registro seguro:** se registra la transacción con marca de tiempo y datos del establecimiento.  
* **Respuesta estandarizada:** formato consistente para éxito y errores.  

---

## 7. Escenarios de uso

✅ Caso exitoso:  
* Cliente canjea un beneficio → se valida → se registra → respuesta con código 201.  

⚠️ Posibles fallos:  
* **Datos incompletos o inválidos:** la solicitud será rechazada con código 200 y mensaje de error.  
* **Cliente no encontrado:** se devolverá un error indicando que el cliente no existe.  
* **Error en el servidor:** problemas al registrar el canje en la base de datos.  

---

## 8. Endpoints

### 8.1. AcceptBenefits

**Method:** POST  

```
{{server_client}}/api/v1/client/acceptBenefits
```

**Headers:**
```
Content-Type: application/json
```

**Body:**
```json
{
    "uid_cliente": "6527d97980537036f7d3e807",
    "cdn_id": 10,
    "rst_id": 40,
    "nombreCadena": "KFC",
    "local": "K004",
    "direccionRestaurante": "Av las Americas",
    "pluId": 1454,
    "nombrePlus": "1 presa KFC Crispy",
    "fechaCanje": "2023-10-12T18:05:05Z"
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
        "uid_cliente": "6527d97980537036f7d3e807",
        "cdn_id": 10,
        "rst_id": 40,
        "nombreCadena": "KFC",
        "local": "K004",
        "direccionRestaurante": "Av las Americas",
        "pluId": 1454,
        "nombrePlus": "1 presa KFC Crispy",
        "fechaCanje": "2023-10-12T18:05:05Z"
    }
}
```
</details>

---

## 9. Preguntas frecuentes (para IA Support)

**Q: ¿Qué campos son obligatorios en la solicitud?**  
A: Todos los campos mostrados en el ejemplo son requeridos para registrar correctamente el canje de un beneficio.

**Q: ¿Qué formato debo usar para la fecha de canje?**  
A: La fecha debe estar en formato ISO 8601 (ej: "2023-10-12T18:05:05Z").

**Q: ¿Qué significa el código de respuesta 21?**  
A: El código 21 indica que el canje de beneficio fue registrado exitosamente.

**Q: ¿Qué hago si el cliente no existe en el sistema?**  
A: Se debe registrar primero al cliente en el sistema antes de intentar registrar un canje de beneficio.

**Q: ¿Se puede canjear un beneficio sin especificar el local?**  
A: No, el campo 'local' es obligatorio para identificar dónde se realiza el canje.

---

## 10. Referencias

Fuentes internas:  

* mxp-clients/process/benefits.md  
* Microservicio: mxpv2.integration.clients  

---

## 11. Contacto

**Equipo responsable:** Integraciones MAXPOINT  
**Canal de soporte:** #mxp-integraciones (Slack interno)  
**Escalamiento:** Equipo de Beneficios → Liderazgo técnico
