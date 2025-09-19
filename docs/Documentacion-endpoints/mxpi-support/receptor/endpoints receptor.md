# 📌 Módulo: MXPI-support  
## 📂 Carpeta: receptor  

---

## 🛠️ End-point: Receiver

### ➡️ Método: POST
```
{{server.support}}/api/support/receiver
```

### 📥 Request Body
```json
{
  "clientApp": "opl.printer",
  "process": "FACTURA_PRINT",
  "referenceID": "212121-878787-455445-454",
  "priority": 1, 
  "messages": [
    "es obligatiorio la facturacion",
    "la impresora no responde",
    "falta papel"
  ],
  "date": "2013-02-27T13:57:21+0000"
}
```

#### 📌 Descripción de campos
- **clientApp** (string) → Identificador de la aplicación cliente.  
- **process** (string) → Proceso asociado al soporte.  
- **referenceID** (string) → Identificador único de referencia.  
- **priority** (number) → Nivel de prioridad del requerimiento (1, 2, 3, 4).  
- **messages** (array) → Lista de mensajes asociados al incidente.  
- **date** (string - formato ISO8601) → Fecha y hora de la solicitud.  

### 📤 Response (ejemplo)
```json
{
  "code": 202,
  "status": "success",
  "alert": "Solicitud recibida correctamente",
  "messages": [
    "Integración completada exitosamente"
  ]
}
```

### ✅ Validaciones (tests automatizados)
- El body contiene todos los campos obligatorios.  
- `priority` debe estar entre **1 y 4**.  
- El formato de la fecha debe ser válido (**YYYY-MM-DDTHH:mm:ss+0000**).  
- El código HTTP esperado es **202**.  
- La respuesta debe incluir `code`, `status`, `alert`, `messages`.  
- Los mensajes en la respuesta deben contener `"Integración completada exitosamente"`.  

---

## 🛠️ End-point: Get All Paginated

### ➡️ Método: POST
```
{{server.support_api}}/api/v1/support/getAllPaginated
```

### 📥 Request Body
```json
{
  "search": "",
  "sort_order": 0,
  "sort_field": "",
  "rows": 20,
  "first": 1,
  "filters": []
}
```

#### 📌 Descripción de campos
- **search** (string) → Texto de búsqueda.  
- **sort_order** (number) → Orden de los resultados (**1 = ascendente, -1 = descendente**).  
- **sort_field** (string) → Campo por el cual se ordena.  
- **rows** (number) → Cantidad de registros por página.  
- **first** (number) → Índice inicial de la paginación.  
- **filters** (array) → Lista de filtros aplicados a la búsqueda.  

### 📤 Response (ejemplo)
```json
{
  "code": 200,
  "status": "success",
  "data": {
    "totalRecords": 100,
    "records": [
      {
        "id": "123456",
        "clientApp": "opl.printer",
        "process": "FACTURA_PRINT",
        "priority": 1,
        "messages": ["es obligatiorio la facturacion"],
        "date": "2013-02-27T13:57:21+0000"
      }
    ]
  }
}
```

### ✅ Validaciones esperadas
- El request debe incluir parámetros de paginación (`rows`, `first`).  
- El response debe contener `totalRecords` y `records`.  
- Cada registro debe contener la estructura esperada (`id`, `clientApp`, `process`, `priority`, `messages`, `date`).  

---
