# Endpoints de Entidades (MXP-Entity)

## 1. Propósito y alcance
El módulo de **Entidades** en MAXPOINT V2.0 gestiona la creación, consulta, actualización y listado de entidades asociadas a repositorios.  

- Permite administrar entidades a nivel de repositorio.  
- Facilita la consulta de entidades por distintos criterios (ID, código, repositorio).  
- Provee mecanismos de paginación y filtrado.  
- Garantiza coherencia en el manejo de datos relacionados con repositorios.  

ℹ️ Para más detalles de módulos relacionados, consulta:  
- MXP Repository  
- MXP Sync  

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.entity`  
- **Rol principal:** gestión y orquestación de entidades.  
- **Función crítica:** centralizar operaciones CRUD sobre entidades vinculadas a repositorios.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de exposición de endpoints
Componente | Endpoint | Método | Descripción
---|---|---|---
Consultar entidad por ID | `/api/v1/entities/getById` | GET | Obtiene una entidad por su identificador único.
Crear entidad | `/api/v1/entities/create` | POST | Registra una nueva entidad en el sistema.
Consultar por repositorio | `/entities/getByRepositoryId` | GET | Devuelve las entidades asociadas a un repositorio.
Consultar por código | `/api/v1/entities/getByCode` | GET | Busca entidades usando un código específico.
Listar entidades (paginado) | `/api/v1/entities/getAllPaginated` | POST | Devuelve entidades con filtros y paginación.
Actualizar entidad | `/api/v1/entities/update` | PUT | Modifica los datos de una entidad existente.

---

## 4. Gestión de procesos

### Flujo de trabajo estándar
1. **Consulta:** se reciben peticiones GET/POST en los endpoints correspondientes.  
2. **Validación:** se verifica que los parámetros de búsqueda o el body estén completos.  
3. **Procesamiento:** el microservicio aplica la lógica de negocio y consulta el repositorio de datos.  
4. **Respuesta:** se devuelve un objeto JSON con el resultado de la operación, mensajes y estado.  

---

## 5. Puntos de integración
El módulo de **Entidades** se conecta con otros componentes:  

- **Entrada:** solicitudes desde frontend u orquestadores (ejemplo: panel de administración).  
- **Salida:** acceso a repositorios, devolviendo entidades asociadas.  
- **Rol central:** mantener una capa de abstracción entre repositorios y consumidores de datos.  

---

## 6. Patrones de uso
- **Identificación directa:** búsqueda por `id` o `code`.  
- **Agrupación lógica:** búsqueda por `repositoryId`.  
- **Administración global:** paginación y listado de todas las entidades activas.  
- **Actualización segura:** modificación controlada de datos mediante PUT.  

---

## 7. Escenarios de uso

✅ **Caso exitoso:**  
- Se consulta `getById` con un UUID válido → se retorna la entidad asociada.  
- Se ejecuta `create` con parámetros completos → entidad creada correctamente.  
- Se consulta `getAllPaginated` con filtros → lista filtrada y paginada.  

⚠️ **Posibles fallos:**  
- **Entidad no encontrada:** respuesta con código de error y mensaje.  
- **Parámetros inválidos:** rechazo de la petición con descripción del error.  
- **Error en actualización:** si el `id` no existe, no se modifica nada.  

---

## 8. Endpoints detallados

### 📌 8.1. Obtener entidad por ID
**Method:** GET  
```
{{server_orchestrator}}/api/v1/entities/getById?id={uuid}
```

**Query Params:**  
| Param | Ejemplo |
|---|---|
| id | 059bb531-5b34-4435-bdad-21c81b41cac1 |

**Ejemplo de respuesta (200):**
```json
{
  "code": 20,
  "messages": ["Encontrado Correctamente"],
  "data": {
    "id": "059bb531-5b34-4435-bdad-21c81b41cac1",
    "code": "1",
    "name": "plu",
    "repositoryName": "mxp-dev"
  }
}
```

---

### 📌 8.2. Crear entidad
**Method:** POST  
```
{{server_orchestrator}}/api/v1/entities/create
```

**Body ejemplo:**
```json
{
  "name": "Nueva entidad",
  "typeId": "d528abf5-ec41-4c8f-9bc1-8211855c3749",
  "repositoryId": "b5ad9754-0419-46e5-b556-7ab1ab8db3e1",
  "repositoryName": "mxp-dev",
  "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

**Respuesta exitosa:**
```json
{
  "code": 10,
  "messages": ["Creado Correctamente"],
  "data": { "id": "059bb531-5b34-4435-bdad-21c81b41cac1" }
}
```

---

### 📌 8.3. Obtener entidades por repositorio
**Method:** GET  
```
{{server_orchestrator}}/entities/getByRepositoryId?repositoryId={uuid}
```

**Query Params:**  
| Param | Ejemplo |
|---|---|
| repositoryId | 6bd36e91-c0de-4b6d-87b8-5dea7f2c2274 |

---

### 📌 8.4. Obtener entidad por código
**Method:** GET  
```
{{server_orchestrator}}/api/v1/entities/getByCode?code=E001
```

**Query Params:**  
| Param | Ejemplo |
|---|---|
| code | E001 |

---

### 📌 8.5. Listar entidades (paginado)
**Method:** POST  
```
{{server_orchestrator}}/api/v1/entities/getAllPaginated
```

**Body ejemplo:**
```json
{
  "search": "",
  "sort_order": 1,
  "sort_field": "name",
  "rows": 20,
  "first": 1,
  "activeOnly": true
}
```

---

### 📌 8.6. Actualizar entidad
**Method:** PUT  
```
{{server_orchestrator}}/api/v1/entities/update?id={uuid}
```

**Query Params:**  
| Param | Ejemplo |
|---|---|
| id | 059bb531-5b34-4435-bdad-21c81b41cac1 |

**Body ejemplo:**
```json
{
  "name": "Entidad actualizada",
  "repositoryName": "mxp-dev",
  "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

---

## 9. Preguntas frecuentes (para IA Support)

**Q:** ¿Qué pasa si consulto un `id` inexistente?  
**A:** Se retorna un mensaje de error `"Entidad no encontrada"` con código distinto de 20.  

**Q:** ¿Puedo filtrar solo entidades activas?  
**A:** Sí, usando el campo `activeOnly: true` en el endpoint `getAllPaginated`.  

**Q:** ¿Qué diferencia hay entre `getByCode` y `getById`?  
**A:** `getById` busca por identificador único (UUID), mientras que `getByCode` lo hace por un código de negocio definido.  

---

## 10. Referencias
- Documentación interna: `mxp-entity/process/readme.md`  
- Repositorio: MXP Repository  
- Microservicio: `mxpv2.integration.entity`  

---

## 11. Contacto
- **Equipo responsable:** MAXPOINT - Gestión de Entidades  
- **Canal de soporte:** #mxp-entities (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico  
