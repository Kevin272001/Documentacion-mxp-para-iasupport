# Endpoints de Repositorios (MXP-Repository)

## 1. Propósito y alcance

El módulo **MXP-Repository** gestiona la configuración y administración de repositorios dentro del ecosistema **MAXPOINT V2.0**.

* Permite crear, actualizar y consultar repositorios de bases de datos.
* Garantiza la integración segura con servidores y adaptadores.
* Facilita la gestión de autenticación y estados de repositorios.

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Init
* MXP Connection

## 2. Descripción general del sistema

* Microservicio: `mxpv2.integration.repository`
* Rol principal: gestionar repositorios y su información asociada.
* Función crítica: asegurar la conectividad y consistencia de datos hacia los módulos que consumen estos repositorios.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente               | Código | Descripción                                                                                                      |
| ------------------------ | ------ | ---------------------------------------------------------------------------------------------------------------- |
| Interfaz de repositorios | F001   | Punto de entrada principal para la gestión de repositorios. Valida la estructura y seguridad de las solicitudes. |

### 3.2. Capa de datos

| Componente              | Código | Descripción                                                                                 |
| ----------------------- | ------ | ------------------------------------------------------------------------------------------- |
| Gestión de repositorios | D001   | Almacena la información de los repositorios, incluyendo credenciales, estado y descripción. |

## 4. Gestión de procesos

### Flujo de trabajo estándar

1. **Creación de repositorio** → Solicitud POST a `create`.
2. **Actualización de repositorio** → Solicitud PUT a `update`.
3. **Consulta de repositorios** → Solicitudes GET/POST a `getById`, `getByCode`, `getAllPaginated`.
4. **Validación y registro** → Se asegura que los datos sean correctos y persistentes.

## 5. Endpoints

### 📌 Create Repository

**Método:** `POST`  
**Endpoint:** `{{server_orchestrator}}/api/v1/repositories/create`

**Body (raw JSON):**

```json
{
    "port": 9562,
    "username": "admin",
    "password": "admin123",
    "databaseName": "mxp-dev",
    "authTypeId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "url": "https://apim-hera.azure-api.net/mxpi-init-prepro1",
    "repositoryDescription": "Repositorio para mxp legacy"
}
```

**Response:**

<details open>
<summary>Ejemplo de respuesta</summary>

```json
{
    "code": "21",
    "messages": [],
    "data": {
        "id": "6bd36e91-c0de-4b6d-87b8-5dea7f2c2274",
        "code": "R001",
        "port": 9562,
        "username": "admin",
        "password": "admin123",
        "databaseName": "mxp-dev",
        "authTypeId": "Bearer",
        "statusId": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3"
    }
}
```

</details>

---

### 📌 Update Repository

**Método:** `PUT`  
**Endpoint:** `{{server_orchestrator}}/api/v1/repositories/update?id=3fa85f64-5717-4562-b3fc-2c963f66afa6`

**Body (raw JSON):**

```json
{
    "id": "3fa85f64-5717-4562-bfc-2c963f66afa6",
    "code": "R001",
    "port": 9562,
    "username": "admin",
    "password": "admin123",
    "databaseName": "mxp-dev",
    "authTypeId": "3fa85f64-5717-4562-bfc-2c963f66afa6",
    "statusId": "3fa85f64-5717-4562-bfc-2c963f66afa6",
    "url": "https://apim-hera.azure-api.net/mxpi-init-prepro1",
    "repositoryDescription": "Repositorio para mxp legacy"
}
```

**Query Params:**

| Param | Value                               |
| ----- | ----------------------------------- |
| id    | 3fa85f64-5717-4562-bfc-2c963f66afa6 |

**Response:**

<details open>
<summary>Ejemplo de respuesta</summary>

```json
{
    "code": "21",
    "messages": [],
    "data": {
        "id": "6bd36e91-c0de-4b6d-87b8-5dea7f2c2274",
        "code": "R001",
        "port": 9562,
        "username": "admin",
        "password": "admin123",
        "databaseName": "mxp-dev",
        "authTypeId": "3fa85f64-5717-4562-bfc-2c963f66afa6",
        "statusId": "3fa85f64-5717-4562-bfc-2c963f66afa6"
    }
}
```

</details>

---

### 📌 Get All Repositories

**Método:** `POST`  
**Endpoint:** `{{server_orchestrator}}/api/v1/repositories/getAllPaginated`

**Body (raw JSON):**

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

**Response:**

<details open>
<summary>Ejemplo de respuesta</summary>

```json
{
    "code": 20,
    "description": "Data returned successfully",
    "data": {
        "total_rows": 1,
        "rows": [
            {
                "id": "6bd36e91-c0de-4b6d-87b8-5dea7f2c2274",
                "code": "R001",
                "port": 9562,
                "username": "admin",
                "password": "admin123",
                "databaseName": "mxp-dev",
                "authTypeId": "3fa85f64-5717-4562-bfc-2c963f66afa6",
                "statusId": "3fa85f64-5717-4562-bfc-2c963f66afa6",
                "url": "https://apim-hera.azure-api.net/mxpi-init-prepro1",
                "repositoryDescription": "Repositorio para mxp legacy"
            }
        ]
    }
}
```

</details>

---

### 📌 Get Repository by Code

**Método:** `GET`  
**Endpoint:** `{{server_orchestrator}}/api/v1/repositories/getByCode?code=R001`

**Query Params:**

| Param | Value |
| ----- | ----- |
| code  | R001  |

---

### 📌 Get Repository by Id

**Método:** `GET`  
**Endpoint:** `{{server_orchestrator}}/api/v1/repositories/getById?id=6bd36e91-c0de-4b6d-87b8-5dea7f2c2274`

**Query Params:**

| Param | Value                                |
| ----- | ------------------------------------ |
| id    | 6bd36e91-c0de-4b6d-87b8-5dea7f2c2274 |

**Response:**

<details open>
<summary>Ejemplo de respuesta</summary>

```json
{
    "code": 20,
    "messages": ["Encontrado Correctamente"],
    "data": {
        "id": "6bd36e91-c0de-4b6d-87b8-5dea7f2c2274",
        "code": "R001",
        "port": 9562,
        "username": "admin",
        "password": "admin123",
        "databaseName": "mxp-dev",
        "authTypeId": "3fa85f64-5717-4562-bfc-2c963f66afa6",
        "statusId": "3fa85f64-5717-4562-bfc-2c963f66afa6",
        "url": "https://apim-hera.azure-api.net/mxpi-init-prepro1",
        "repositoryDescription": "Repositorio para mxp legacy"
    }
}
```

</details>

## 6. Patrones de integración

* **Creación y actualización centralizada** → Todos los repositorios se gestionan vía API.
* **Consulta paginada** → Permite optimizar la carga de datos y filtrar activos.
* **Validación de credenciales y estado** → Asegura la coherencia de la información.

## 7. Escenarios de uso

✅ Caso exitoso:

* Se crea un repositorio → se valida → se almacena correctamente → se consulta por código o ID → se obtiene información correcta.

⚠️ Posibles fallos:

* Error de validación → datos incorrectos → se rechaza la solicitud y se registra log.
* Error de actualización → repositorio no existente → se retorna error de no encontrado.

## 8. Preguntas frecuentes (para IA Support)

Q: ¿Qué pasa si el repositorio no se puede crear?  
A: Se retorna un código de error y un mensaje, revisa que los parámetros sean correctos.

Q: ¿Cómo puedo verificar que la actualización fue exitosa?  
A: Consulta el repositorio por `getById` o `getByCode` y verifica los datos modificados.

Q: ¿Puedo consultar solo repositorios activos?  
A: Sí, usando `getAllPaginated` con `activeOnly=true`.

## 9. Referencias

* Fuentes internas: `mxp-repository/process/readme.md`
* Microservicio: `mxpv2.integration.repository`

## 10. Contacto

* Equipo responsable: Integraciones MAXPOINT  
* Canal de soporte: #mxp-integraciones (Slack interno)  
* Escalamiento: Arquitectura de datos → Liderazgo técnico
