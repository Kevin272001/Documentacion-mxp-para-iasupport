id: property
title: Módulo de Propiedades (Property)
sidebar_label: Property
---

# Módulo de Propiedades (Property)

## 1. Propósito y alcance
El módulo **Property** gestiona las propiedades asociadas a entidades dentro del ecosistema MAXPOINT V2.0.  
Permite **crear**, **listar**, **actualizar** y **consultar** propiedades de manera centralizada.  
Garantiza la coherencia y disponibilidad de la información de propiedades para otros módulos.  

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- Entity  
- Status  
- Type  

## 2. Descripción general del sistema
- **Microservicio:** mxpv2.integration.property  
- **Rol principal:** gestionar propiedades de entidades.  
- **Función crítica:** asegurar que la información de propiedades sea correcta, actualizada y accesible para otros módulos.  

## 3. Arquitectura y componentes

### 3.1 Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz REST Property | P001 | Punto de entrada principal para solicitudes de propiedades (CRUD). 🔑 Valida estructura e integridad de los datos. |

### 3.2 Capa de persistencia
| Componente | Código | Descripción |
|------------|--------|-------------|
| Base de datos MongoDB | DB-PROP | Almacena propiedades con sus atributos: id, code, name, typeId, entityId, statusId. |

## 4. Gestión de procesos
### Flujo de trabajo estándar
1. **Creación** → Se recibe la solicitud POST en `/properties/create`.  
2. **Listar propiedades** → POST `/properties/getAllPaginated`.  
3. **Consulta por entidad** → GET `/properties/getByEntityId`.  
4. **Consulta por ID o código** → GET `/properties/getById` o `/properties/getByCode`.  
5. **Actualización** → PUT `/properties/update`.  

### Ejemplo de flujo
- Creación → Validación → Persistencia en DB-PROP → Confirmación de éxito.  
- Listado → Filtro → Ordenamiento → Entrega de resultados.  

## 5. Puntos de integración
- **Entrada:** solicitudes de frontend, otros módulos de MAXPOINT V2.0.  
- **Salida:** respuesta con datos de propiedades (JSON) para consumo interno o externo.  
- **Rol central:** asegura integridad de propiedades y disponibilidad de la información.  

## 6. Patrones de operación
- **Centralización:** todas las propiedades se gestionan a través de la API de Property.  
- **Validación de datos:** todos los endpoints validan estructura y campos requeridos.  
- **Coherencia:** evita inconsistencias de estado o tipos asociados a propiedades.  

## 7. Escenarios de uso
✅ **Caso exitoso:**  
Solicitud POST `/properties/create` con datos correctos → validación → persistencia → respuesta `Creado Correctamente`.  

⚠️ **Posibles fallos:**  
- Datos incompletos o inválidos → error de validación.  
- ID de propiedad no encontrado en actualización → error 404.  
- Error en base de datos → respuesta de fallo y log de detalle.  

## 8. Endpoints
### Create
- **Method:** POST  
- **URL:** `/api/v1/properties/create`  
- **Body:**
```json
{
    "name": "sdfsfdfdsf",
    "typeId": "4bbd7a39-31a9-4eb8-abd9-f8aea46df7ea",
    "entityId": "059bb531-5b34-4435-bdad-21c81b41cac1",
    "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```
- **Response example:**
```json
{
    "code": 10,
    "messages": ["Creado Correctamente"],
    "data": {
        "id":"06b8588c-4cef-4f9e-b3ae-2283508be599",
        "code":"P001",
        "name": "description",
        "typeId": "4bbd7a39-31a9-4eb8-abd9-f8aea46df7ea",
        "entityId": "059bb531-5b34-4435-bdad-21c81b41cac1",
        "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
    }
}
```

### getAll
- **Method:** POST  
- **URL:** `/api/v1/properties/getAllPaginated`  
- **Body:**
```json
{
  "search": "",
  "sort_order": 0,
  "sort_field": "",
  "rows": 20,
  "first": 1,
  "activeOnly":true
}
```

### getByEntityId
- **Method:** GET  
- **URL:** `/api/v1/properties/getByEntityId?entityId=059bb531-5b34-4435-bdad-21c81b41cac1`  
- **Query Param:** `entityId`  

### Update
- **Method:** PUT  
- **URL:** `/api/v1/properties/update?id=06b8588c-4cef-4f9e-b3ae-2283508be599`  
- **Body:**
```json
{
  "name": "description",
  "typeId": "4bbd7a39-31a9-4eb8-abd9-f8aea46df7ea",
  "entityId": "059bb531-5b34-4435-bdad-21c81b41cac1",
  "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

### getById
- **Method:** GET  
- **URL:** `/api/v1/properties/getById?id=059bb531-5b34-4435-bdad-21c81b41cac1`  
- **Query Param:** `id`  

### getByCode
- **Method:** GET  
- **URL:** `/api/v1/entities/getByCode?code=P001`  
- **Query Param:** `code`  

## 9. Preguntas frecuentes (IA Support)
Q: ¿Qué hago si no se encuentra la propiedad por ID?  
A: Verifica que el ID sea correcto y exista en la base de datos.  

Q: ¿Cómo filtrar solo propiedades activas?  
A: Usa `activeOnly: true` en el endpoint `/getAllPaginated`.  

Q: ¿Se puede actualizar el código de una propiedad?  
A: No, el código (`code`) se genera automáticamente y es único.  

## 10. Referencias
- Fuentes internas: `property/process/readme.md`  
- Microservicio: `mxpv2.integration.property`  

## 11. Contacto
- **Equipo responsable:** Integraciones MAXPOINT  
- **Canal de soporte:** `#mxp-integraciones` (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico
