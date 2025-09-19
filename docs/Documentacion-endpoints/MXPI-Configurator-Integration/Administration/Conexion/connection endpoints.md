# Endpoints de Conexiones (MXP-Connection)

## ❓ ¿Cuál es el propósito del módulo de Conexiones?

El módulo **MXP-Connection** gestiona la administración de conexiones dentro del ecosistema **MAXPOINT V2.0**.
Centraliza la creación, consulta, actualización y eliminación de conexiones que permiten la integración con servidores, adaptadores y repositorios.

## ✅ Respuesta clara y breve

Con este módulo puedes:

* 📡 Gestionar conexiones hacia servidores y repositorios.
* 🔒 Mantener credenciales seguras de acceso.
* ⚙️ Crear, actualizar y consultar conexiones de manera dinámica.
* 🗑️ Eliminar conexiones cuando ya no son necesarias.

---

## 🧩 Descripción general

* **Microservicio:** `mxpv2.integration.connection`
* **Rol principal:** administrar la configuración y ciclo de vida de las conexiones.
* **Función crítica:** garantizar que las conexiones necesarias para los procesos de integración estén disponibles y correctamente configuradas.

---

## 🔌 Endpoints disponibles

### 1. Crear conexión

**Endpoint:**

```http
POST {{server_orchestrator}}/api/v1/connections/create
```

### 📥 Body ejemplo

```json
{
  "serverId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e1",
  "adapterId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e2",
  "repositoryId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e3",
  "name": "Conexión database maxpoint legacy",
  "description": "alguna desc.",
  "statusId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e3"
}
```

<details>
  <summary>🟢 Respuesta 200</summary>

```json
{
  "code": 10,
  "messages": ["created successfully"],
  "data": {
    "id": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3",
    "code": "C001",
    "serverId": "a4d9ba2b-7b84-47bf-b298-033ffd446729",
    "adapterId": "1829da9e-fbbe-4a3b-d24a-3360b5bf6c08",
    "repositoryId": "6bd36e91-c0de-4b6d-87b8-5dea7f2c2274",
    "name": "Conexión database maxpoint legacy",
    "description": "alguna desc.",
    "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
  }
}
```

</details>

---

### 2. Obtener conexión por código

**Endpoint:**

```http
GET {{server_orchestrator}}/api/v1/connections/getByCode?code=C001
```

<details>
  <summary>🟢 Respuesta 200</summary>

```json
{
  "code": 20,
  "messages": ["Encontrado correctamente"],
  "data": {
    "id": "b5ad9754-0419-46e5-b556-7ab1ab8db3e1",
    "code": "C001",
    "serverId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e1",
    "repositoryId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e1",
    "adapterId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e1",
    "name": "Conexión database maxpoint legacy",
    "description": "alguna desc.",
    "statusId": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3"
  }
}
```

</details>

---

### 3. Actualizar conexión

**Endpoint:**

```http
PUT {{server_orchestrator}}/api/v1/connections/update?id=123456
```

### 📥 Body ejemplo

```json
{
  "serverId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e1",
  "adapterId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e2",
  "repositoryId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e3",
  "description": "alguna desc.",
  "statusId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e3"
}
```

<details>
  <summary>🟢 Respuesta 200</summary>

```json
{
  "code": 11,
  "messages": ["updated successfully"],
  "data": {
    "id": "712feaea-8228-4f03-9457-3ce36e618c56",
    "code": "C001",
    "serverId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e1",
    "repositoryId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e3",
    "adapterId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e2",
    "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "Conexión database maxpoint legacy",
    "description": "alguna desc."
  }
}
```

</details>

---

### 4. Listar todas las conexiones (paginado)

**Endpoint:**

```http
POST {{server_orchestrator}}/api/v1/connections/getAllPaginated
```

### 📥 Body ejemplo

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

<details>
  <summary>🟢 Respuesta 200</summary>

```json
{
  "code": 20,
  "description": "Data returned successfully",
  "data": {
    "total_rows": 100,
    "rows": [
      {
        "id": "712feaea-8228-4f03-9457-3ce36e618c56",
        "code": "C001",
        "serverId": "1b689793-a9d8-47a3-bb6d-92d652befbfd",
        "repositoryId": "6bd36e91-c0de-4b6d-87b8-5dea7f2c2274",
        "adapterId": "6f5672e8-32fc-4195-b1f0-326f9e2ac82f",
        "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "name": "Conexión database maxpoint legacy",
        "description": "alguna descripción"
      }
    ]
  }
}
```

</details>

---

### 5. Obtener conexión por ID

**Endpoint:**

```http
GET {{server_orchestrator}}/api/v1/connections/getById?id=b5ad9754-0419-46e5-b556-7ab1ab8db3e1
```

<details>
  <summary>🟢 Respuesta 200</summary>

```json
{
  "code": 20,
  "messages": ["Encontrado correctamente"],
  "data": {
    "id": "712feaea-8228-4f03-9457-3ce36e618c56",
    "code": "C001",
    "serverId": "1b689793-a9d8-47a3-bb6d-92d652befbfd",
    "repositoryId": "6bd36e91-c0de-4b6d-87b8-5dea7f2c2274",
    "adapterId": "6f5672e8-32fc-4195-b1f0-326f9e2ac82f",
    "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "Conexión database maxpoint legacy",
    "description": "alguna descripción"
  }
}
```

</details>

---

### 6. Eliminar conexión

**Endpoint:**

```http
DELETE {{server_orchestrator}}/api/v1/connections/delete?id=7654cb16-e955-4605-8579-b891a67f94c7
```

<details>
  <summary>🟢 Respuesta 200</summary>

```json
{
  "code": 10,
  "messages": ["deleted successfully"],
  "data": {
    "id": "7654cb16-e955-4605-8579-b891a67f94c7"
  }
}
```

</details>

---

## ⚠️ Escenarios de uso

✅ Caso exitoso: Crear → Consultar por ID o código → Actualizar → Eliminar.

❌ Posibles fallos:

* Conexión no encontrada → mensaje de error.
* ID inválido → petición rechazada.
* Dependencia activa → eliminación puede afectar otros módulos.

---

## 🙋 Preguntas frecuentes

**Q:** ¿Cómo obtener solo conexiones activas?
**A:** Usar `getAllPaginated` con `activeOnly=true`.

**Q:** ¿Cómo identifico una conexión única?
**A:** Con los campos `id` o `code`.

**Q:** ¿Qué pasa si elimino una conexión usada en otro módulo?
**A:** Los procesos dependientes fallarán. Se recomienda validar antes de eliminar.

---

## 📚 Referencias

* Documentación interna: `mxp-connection/readme.md`
* Microservicio: `mxpv2.integration.connection`

---

## 📞 Contacto

* Equipo responsable: **Integraciones MAXPOINT**
* Soporte: **#mxp-integraciones** (Slack interno)
* Escalamiento: **Arquitectura de datos → Liderazgo técnico**
