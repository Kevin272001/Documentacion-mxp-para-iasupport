---
id: catalog
title: Módulo de Catálogos (Catalog)
sidebar_label: Catalog
---

# Módulo de Catálogos (Catalog)

## 1. Propósito y alcance
El módulo **Catalog** gestiona catálogos y subcatálogos dentro del ecosistema MAXPOINT V2.0.  
Permite **crear**, **listar**, **actualizar** y **consultar** catálogos de manera centralizada.  
Garantiza la coherencia y disponibilidad de la información para otros módulos.  

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- Property  
- Status  
- Type  

## 2. Descripción general del sistema
- **Microservicio:** mxpv2.integration.catalog  
- **Rol principal:** gestionar catálogos y sus relaciones padre-hijo.  
- **Función crítica:** asegurar que la información de catálogos sea correcta, actualizada y accesible.  

## 3. Arquitectura y componentes

### 3.1 Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz REST Catalog | C001 | Punto de entrada principal para solicitudes de catálogos (CRUD). 🔑 Valida estructura e integridad de los datos. |

### 3.2 Capa de persistencia
| Componente | Código | Descripción |
|------------|--------|-------------|
| Base de datos MongoDB | DB-CAT | Almacena catálogos con sus atributos: id, code, name, value, fatherId, isFather, detail, statusId. |

## 4. Gestión de procesos
### Flujo de trabajo estándar
1. **Creación** → Se recibe la solicitud POST en `/catalogs/create`.  
2. **Consulta por ID** → GET `/catalogs/getById`.  
3. **Actualización** → PUT `/catalogs/update`.  
4. **Consulta por padre** → GET `/catalogs/getByFatherId`.  
5. **Listar catálogos** → POST `/catalogs/getAllPaginated`.  

### Ejemplo de flujo
- Creación → Validación → Persistencia en DB-CAT → Confirmación de éxito.  
- Listado → Filtro → Ordenamiento → Entrega de resultados.  

## 5. Puntos de integración
- **Entrada:** solicitudes de frontend, otros módulos de MAXPOINT V2.0.  
- **Salida:** respuesta con datos de catálogos (JSON) para consumo interno o externo.  
- **Rol central:** asegura integridad de catálogos y disponibilidad de la información.  

## 6. Patrones de operación
- **Centralización:** todos los catálogos se gestionan a través de la API de Catalog.  
- **Validación de datos:** todos los endpoints validan estructura y campos requeridos.  
- **Coherencia:** evita inconsistencias entre catálogos padre e hijo.  

## 7. Escenarios de uso
✅ **Caso exitoso:**  
Solicitud POST `/catalogs/create` con datos correctos → validación → persistencia → respuesta `Creado correctamente`.  

⚠️ **Posibles fallos:**  
- Datos incompletos o inválidos → error de validación.  
- ID de catálogo no encontrado en actualización → error 404.  
- Error en base de datos → respuesta de fallo y log de detalle.  

## 8. Endpoints
### Create
- **Method:** POST  
- **URL:** `/api/v1/catalogs/create`  
- **Body:**
```json
{
    "name": "prueba",
    "code":1,
    "value": "prueba",
    "fatherId": null,
    "isFather":true,
    "detail": "",
    "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```
- **Response example:**  
```json
{
    "code": "10",
    "messages": ["Creado correctamente"],
    "data": {
        "id": "351faf974-5c9e-4936-b234-91a173356dea",
        "code":1,
        "name": "prueba",
        "value": "prueba",
        "fatherId": null,
        "isFather":true,
        "detail": "",
        "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
    }
}
```

### getById
- **Method:** GET  
- **URL:** `/api/v1/catalogs/getById?id=51faf974-5c9e-4936-b234-91a173356dea`  

### Update
- **Method:** PUT  
- **URL:** `/api/v1/catalogs/update?id=51faf974-5c9e-4936-b234-91a173356dea`  
- **Body:**
```json
{
    "name": "Tipo proceso",
    "code":1,
    "value": "process_type",
    "fatherId": null,
    "isFather":true,
    "detail": "",
    "statusId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

### GetByFatherId
- **Method:** GET  
- **URL:** `/api/v1/catalogs/getByFatherId?code=1`  
- **Query Param:** `code`  

### getAll
- **Method:** POST  
- **URL:** `/api/v1/catalogs/getAllPaginated`  
- **Body:**
```json
{
  "search": "",
  "sort_order": 0,
  "sort_field": "",
  "rows": 20,
  "first": 1
}
```

## 9. Preguntas frecuentes (IA Support)
Q: ¿Cómo obtener solo catálogos hijos de un padre?  
A: Usa el endpoint `/getByFatherId` con el código del padre.  

Q: ¿Se puede crear un catálogo sin padre?  
A: Sí, se crea como `isFather: true` y `fatherId: null`.  

Q: ¿Qué pasa si no se encuentra el catálogo por ID?  
A: Verifica que el ID sea correcto y exista en la base de datos.  

## 10. Referencias
- Fuentes internas: `catalog/process/readme.md`  
- Microservicio: `mxpv2.integration.catalog`  

## 11. Contacto
- **Equipo responsable:** Integraciones MAXPOINT  
- **Canal de soporte:** `#mxp-integraciones` (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico
