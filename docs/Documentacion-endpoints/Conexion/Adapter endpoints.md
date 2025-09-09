# Endpoints de Adaptadores (MXP-Adapter)

## 1. Propósito y alcance

MXP Adapter es el sistema encargado de la **gestión centralizada de adaptadores de integración** dentro del ecosistema **MAXPOINT V2.0**.

* Permite administrar conectores hacia **Bases de Datos, Archivos, APIs REST y APIs SOAP**.
* Facilita la creación, consulta, actualización y listado de adaptadores.
* Garantiza la interoperabilidad de los procesos de orquestación (Orchestrator, ETL, Sync).

ℹ️ Para más detalles de los módulos relacionados, consulta:

* MXP Orchestrator
* MXP Init
* MXP Sync

## 2. Descripción general del sistema

* **Microservicio**: `mxpv2.integration.adapter`
* **Rol principal**: gestionar adaptadores de integración con sistemas externos.
* **Función crítica**: asegurar la disponibilidad y configuración correcta de los adaptadores que alimentan el ecosistema.

## 3. Arquitectura y componentes

### 3.1. Capa de administración

| Componente      | Código | Descripción                                |
| --------------- | ------ | ------------------------------------------ |
| Crear adaptador | A001   | Registra un nuevo adaptador en el sistema. |
| Actualizar      | A002   | Permite actualizar un adaptador existente. |

### 3.2. Capa de consulta

| Componente         | Código | Descripción                                         |
| ------------------ | ------ | --------------------------------------------------- |
| Obtener por ID     | A003   | Recupera un adaptador mediante su **ID**.           |
| Obtener por Código | A004   | Recupera un adaptador por su **código único**.      |
| Obtener por Tipo   | A005   | Lista adaptadores por tipo (ej. `EXTRACT`).         |
| Obtener todos      | A006   | Devuelve todos los adaptadores en formato paginado. |

## 4. Gestión de procesos

**Flujo estándar de trabajo con adaptadores:**

1. **Creación** → Se registra el adaptador con nombre, tipo y versión.
2. **Validación** → El sistema valida que el tipo sea permitido (`FILE`, `BD`, `API_REST`, `API_SOAP`).
3. **Consulta** → Se pueden buscar adaptadores por ID, código o tipo.
4. **Actualización** → Se modifican parámetros como nombre, versión o estado.
5. **Listado** → Se obtiene un catálogo paginado y filtrado de adaptadores activos.

## 5. Puntos de integración

MXP Adapter conecta el ecosistema MAXPOINT V2.0:

* **Entrada**: recibe solicitudes de creación, consulta o actualización desde el Orchestrator.
* **Salida**: entrega configuraciones de adaptadores a módulos consumidores (ej. ETL, MXP Sync).
* **Rol central**: garantizar conectividad estándar hacia sistemas externos.

## 6. Patrones de gestión

* **Creación centralizada** → todos los adaptadores se registran mediante `/create`.
* **Consultas flexibles** → búsqueda por ID, código o tipo.
* **Control de estado** → los adaptadores se gestionan con `statusId` para habilitar o deshabilitar.

## 7. Escenarios de uso

✅ **Caso exitoso**
Se crea un adaptador `MongoDB` versión `8.0` → validación correcta → queda disponible para otros módulos.

⚠️ **Posibles fallos**

* **Error en validación**: tipo no permitido. → Acción: usar solo `FILE`, `BD`, `API_REST`, `API_SOAP`.
* **Error en consulta**: ID/código inexistente. → Acción: verificar parámetros enviados.
* **Error en actualización**: campos obligatorios faltantes. → Acción: corregir payload antes de reenviar.

## 8. Preguntas frecuentes (IA Support)

**Q: ¿Qué tipos de adaptadores se permiten?**
A: `FILE`, `BD`, `API_REST`, `API_SOAP`.

**Q: ¿Cómo obtener solo adaptadores activos?**
A: Usar `POST /getAllPaginated` con `"activeOnly": true`.

**Q: ¿Qué pasa si un adaptador no existe?**
A: Se devuelve mensaje de “No encontrado”, revisar `id` o `code`.

**Q: ¿Puedo cambiar el tipo de un adaptador ya creado?**
A: No, solo se actualizan nombre, versión y estado.

## 9. Referencias

Fuentes internas:

* `mxp-adapter/process/readme.md`
* `mxp-orchestrator/docs/endpoints`
  Microservicio: **mxpv2.integration.adapter**

## 10. Contacto

* **Equipo responsable**: Integraciones MAXPOINT
* **Canal de soporte**: `#mxp-integraciones` (Slack interno)
* **Escalamiento**: Arquitectura de datos → Liderazgo técnico

# 📌 Endpoints Adapter

### 🔹 Crear adaptador

```http
POST {{server_orchestrator}}/api/v1/adapters/create
```

**Body**

```json
{
  "name": "mongo",
  "typeAdapaterId": "69503d8f-fa70-2196-f0a0-e3a85fa10aec",
  "version": "8.0",
  "statusId": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3"
}
```

**Response**

```json
{
  "code": 10,
  "messages": ["created succefully"],
  "data": {
    "id": "1829da9e-fbbe-4a3b-d24a-3360b5bf6c08",
    "code": "A001",
    "name": "mongo",
    "typeAdapaterId": "69503d8f-fa70-2196-f0a0-e3a85fa10aec",
    "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
  }
}
```

### 🔹 Obtener por código

```http
GET {{server_orchestrator}}/api/v1/adapters/getByCode?code=A001
```

**Response**

```json
{
  "code": 20,
  "messages": ["Encontrado Correctamente"],
  "data": {
    "id": "6f5672e8-32fc-4195-b1f0-326f9e2ac82f",
    "code": "A001",
    "name": "mongodb",
    "typeAdapaterId": "69503d8f-fa70-2196-f0a0-e3a85fa10aec",
    "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "version": "10.0"
  }
}
```

### 🔹 Obtener por tipo

```http
GET {{server_orchestrator}}/api/v1/adapters/getByType?code=EXTRACT
```

**Response**

```json
{
  "code": 20,
  "description": "Data returned succesfully",
  "data": [
    {
      "id": "6f5672e8-32fc-4195-b1f0-326f9e2ac82f",
      "code": "A001",
      "name": "mongodb",
      "typeAdapaterId": "69503d8f-fa70-2196-f0a0-e3a85fa10aec",
      "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "version": "10.0"
    }
  ]
}
```

### 🔹 Obtener todos (paginado)

```http
POST {{server_orchestrator}}/api/v1/adapters/getAllPaginated
```

**Body**

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

**Response**

```json
{
  "code": 20,
  "description": "Data returned succesfully",
  "data": {
    "total_rows": 100,
    "rows": [
      {
        "id": "6f5672e8-32fc-4195-b1f0-326f9e2ac82f",
        "code": "A001",
        "name": "mongodb",
        "typeAdapaterId": "69503d8f-fa70-2196-f0a0-e3a85fa10aec",
        "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "version": "10.0"
      },
      {
        "id": "8d9c0ccb-27aa-4559-b533-ddf4b120c661",
        "code": "A002",
        "name": "SQLSERVER",
        "typeAdapaterId": "69503d8f-fa70-2196-f0a0-e3a85fa10aec",
        "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "version": "10.0"
      }
    ]
  }
}
```

### 🔹 Obtener por ID

```http
GET {{server_orchestrator}}/api/v1/adapters/getById?id=b5ad9754-0419-46e5-b556-7ab1ab8db3e1
```

**Response**

```json
{
  "code": 20,
  "messages": ["Encontrado Correctamente"],
  "data": {
    "id": "6f5672e8-32fc-4195-b1f0-326f9e2ac82f",
    "code": "A001",
    "name": "mongodb",
    "typeAdapaterId": "69503d8f-fa70-2196-f0a0-e3a85fa10aec",
    "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "version": "10.0"
  }
}
```

### 🔹 Actualizar adaptador

```http
PUT {{server_orchestrator}}/api/v1/adapters/update?id=7654cb16-e955-4605-8579-b891a67f94c7
```

**Body**

```json
{
  "name": "mongo",
  "typeAdapaterId": "69503d8f-fa70-2196-f0a0-e3a85fa10aec",
  "statusId": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3"
}
```

**Response**

```json
{
  "code": 10,
  "messages": ["updated succesfully"],
  "data": {
    "id": "7654cb16-e955-4605-8579-b891a67f94c7",
    "code": "A001",
    "name": "mongo",
    "typeAdapaterId": "69503d8f-fa70-2196-f0a0-e3a85fa10aec",
    "statusId": "676f8b52-09d0-4bb5-94ef-d854b4a26fd3"
  }
}

```
