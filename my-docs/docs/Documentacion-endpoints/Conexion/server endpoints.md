# Endpoints de Servidores (MXP-Server)

## 1. Propósito y alcance

MXP Server es el módulo encargado de la gestión de servidores dentro del ecosistema MAXPOINT V2.0.

* Permite registrar, actualizar, consultar y eliminar servidores.
* Centraliza la información de los servidores para su uso en otros módulos.
* Garantiza que los servidores estén correctamente configurados y activos.

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Init
* MXP FE Build
* Impresora MXP

---

## 2. Descripción general del sistema

**Microservicio:** mxpv2.integration.server
**Rol principal:** administrar información de servidores.
**Función crítica:** asegurar que los servidores estén correctamente registrados y disponibles para otros módulos del ecosistema.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                        | Código | Descripción                                                                                                     |
| --------------------------------- | ------ | --------------------------------------------------------------------------------------------------------------- |
| Interfaz de gestión de servidores | S-FH01 | Punto de entrada para operaciones CRUD sobre servidores. 🔑 Valida los datos de entrada y asegura consistencia. |

### 3.2. Capa de procesamiento

| Componente         | Código | Descripción                                                                                    |
| ------------------ | ------ | ---------------------------------------------------------------------------------------------- |
| Procesamiento CRUD | S-CR01 | Gestiona la creación, actualización, eliminación y consulta de servidores en la base de datos. |

---

## 4. Gestión de procesos

Flujo de trabajo estándar:

1. **Creación:** Se envía un POST a `/servers/create` con los datos del servidor.
2. **Consulta general:** Se utiliza `/servers/getAllPaginated` para listar servidores activos o filtrados.
3. **Consulta individual:** Se utiliza `/servers/getById` o `/servers/getByCode` para obtener información de un servidor específico.
4. **Actualización:** Se envía un PUT a `/servers/update` con los cambios requeridos.
5. **Eliminación:** Se envía un DELETE a `/servers/delete` con el ID del servidor.

---

## 5. Puntos de integración

El módulo de servidores interactúa con todo el ecosistema MAXPOINT V2.0:

* **Entrada:** solicitudes de registro y actualización desde módulos internos o interfaces administrativas.
* **Salida:** datos de servidores listos para ser consumidos por otros microservicios o interfaces de usuario.
* **Rol central:** mantiene la consistencia de los servidores activos y su información asociada.

---

## 6. Patrones de gestión

* **Recepción centralizada:** Todos los CRUD llegan vía el componente S-FH01.
* **Procesamiento coordinado:** Las operaciones CRUD se gestionan mediante S-CR01, asegurando integridad y consistencia.
* **Distribución controlada:** Los datos validados se entregan a otros módulos según necesidad.

---

## 7. Escenarios de uso

✅ Caso exitoso:

* Se crea un servidor → se valida → se almacena correctamente → disponible para consumo por otros módulos.

⚠️ Posibles fallos:

* **Error en validación:** Datos incompletos o inválidos. Se registra en logs y la operación falla.
* **Error en almacenamiento:** La base de datos no responde o hay conflicto de ID. La operación se rechaza y se notifica al usuario.
* **Error en consulta:** Servidor no encontrado. Se retorna código de error y mensaje descriptivo.

---

## 8. Endpoints

### 8.1. Create

**Method:** POST

```
{{server_orchestrator}}/api/v1/servers/create
```

**Body (raw):**

```json
{
    "name": "server_centos",
    "typeServerId": "1b689793-a9d8-47a3-bb6d-92d652befbfd",
    "statusId": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3",
    "url": "https://server.com"
}
```

**Response example:**

<details>
<summary>200 OK</summary>
```json
{
    "code": 10,
    "messages": ["created succesfully"],
    "data": {
        "id": "a4d9ba2b-7b84-47bf-b298-033ffd446729",
        "code": "S001",
        "name": "server_centos",
        "typeServerId": "1b689793-a9d8-47a3-bb6d-92d652befbfd",
        "url": "https://server.com",
        "statusId": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3"
    }
}
```
</details>

---

### 8.2. getAll

**Method:** POST

```
{{server_orchestrator}}/api/v1/servers/getAllPaginated
```

**Body (raw):**

```json
{
  "search": "",
  "sort_order": 0,
  "sort_field": "",
  "rows": 20,
  "first": 1,
  "activeOnly": true
}
```

**Response example:**

<details>
<summary>200 OK</summary>
```json
{
    "code": 20,
    "description": "Data returned succesfully",
    "data": {
        "total_rows": 1,
        "rows": [
            {
                "id": "1b689793-a9d8-47a3-bb6d-92d652befbfd",
                "code": "S001",
                "name": "server_centos",
                "typeServerId": "1b689793-a9d8-47a3-bb6d-92d652befbfd",
                "url": "https://server.com",
                "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
            }
        ]
    }
}
```
</details>

---

### 8.3. Update

**Method:** PUT

```
{{server_orchestrator}}/api/v1/servers/update?id=123456
```

**Body (raw):**

```json
{
    "name": "server_centos",
    "typeServerId": "1b689793-a9d8-47a3-bb6d-92d652befbfd",
    "url": "https://server.com",
    "statusId": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3"
}
```

**Query Params:**

| Param | Value  |
| ----- | ------ |
| id    | 123456 |

**Response example:**

<details>
<summary>200 OK</summary>
```json
{
    "code": 11,
    "messages": ["upated succesfully"],
    "data": {
        "name": "server_centos",
        "typeServerId": "1b689793-a9d8-47a3-bb6d-92d652befbfd",
        "url": "https://server.com",
        "statusId": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3"
    }
}
```
</details>

---

### 8.4. getById

**Method:** GET

```
{{server_orchestrator}}/api/v1/servers/getById?id=b5ad9754-0419-46e5-b556-7ab1ab8db3e1
```

**Query Params:**

| Param | Value                                |
| ----- | ------------------------------------ |
| id    | b5ad9754-0419-46e5-b556-7ab1ab8db3e1 |

**Response example:**

<details>
<summary>200 OK</summary>
```json
{
    "code": 20,
    "messages": ["Encontrado Correctamente"],
    "data": {
        "id": "123456",
        "code": "S001",
        "name": "mysql",
        "typeServerId": "1b689793-a9d8-47a3-bb6d-92d652befbfd",
        "url": "https://server.com",
        "statusId": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3"
    }
}
```
</details>

---

### 8.5. Delete

**Method:** DELETE

```
{{server_orchestrator}}/api/v1/servers/delete?id=7654cb16-e955-4605-8579-b891a67f94c7
```

**Query Params:**

| Param | Value                                |
| ----- | ------------------------------------ |
| id    | 7654cb16-e955-4605-8579-b891a67f94c7 |

**Response example:**

<details>
<summary>200 OK</summary>
```json
{
    "code": 10,
    "messages": ["deleted succefully"],
    "data": {
        "id": "7654cb16-e955-4605-8579-b891a67f94c7"
    }
}
```
</details>

---

### 8.6. getByCode

**Method:** GET

```
{{server_orchestrator}}/api/v1/servers/getByCode?code=S001
```

**Query Params:**

| Param | Value |
| ----- | ----- |
| code  | S001  |

---

## 9. Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si intento crear un servidor con datos incompletos o inválidos?**
A: La solicitud será rechazada y se retornará un mensaje de error en `messages`. Revisa que `name`, `typeServerId`, `statusId` y `url` estén completos y en el formato correcto.

**Q: ¿Cómo sé si la creación del servidor fue exitosa?**
A: Revisa el código de respuesta `code: 10` y el mensaje "created succesfully" en la respuesta. Además, se devolverán los datos completos del servidor creado.

**Q: ¿Qué ocurre si intento actualizar un servidor que no existe?**
A: La operación retornará un error indicando que el `id` no fue encontrado. Verifica el `id` enviado en los query params antes de realizar la actualización.

**Q: ¿Cómo puedo listar solo los servidores activos?**
A: Usa el endpoint `/servers/getAllPaginated` con el parámetro `activeOnly` en `true`.

**Q: ¿Qué sucede si solicito un servidor por ID o código que no existe?**
A: La respuesta contendrá un mensaje de error indicando que no se encontró el servidor. Comprueba que el `id` o `code` sean correctos.

**Q: ¿Qué pasa si intento eliminar un servidor que está en uso?**
A: La eliminación podría fallar según las reglas de negocio del sistema. La respuesta indicará "deleted succefully" solo si la operación se realizó correctamente. Si no, revisa los logs o dependencias del servidor.

**Q: ¿Cómo puedo verificar que los cambios se aplicaron correctamente tras una actualización?**
A: Consulta nuevamente el servidor usando `/servers/getById` o `/servers/getByCode` y verifica que los campos actualizados reflejen los nuevos valores.

---

## 10. Referencias

Fuentes internas:

* mxp-server/process/readme.md (1-30)
* Microservicio: mxpv2.integration.server

---

## 11. Contacto

**Equipo responsable:** Integraciones MAXPOINT
**Canal de soporte:** #mxp-integraciones (Slack interno)
**Escalamiento:** Arquitectura de datos → Liderazgo técnico
