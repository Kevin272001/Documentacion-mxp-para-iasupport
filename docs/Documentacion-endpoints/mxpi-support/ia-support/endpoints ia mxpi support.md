# 📌 Módulo de Soporte (MXPI-ia support)

---

## 1. Propósito y alcance
El módulo **MXPI-ia support** es responsable de la gestión y recepción de incidencias dentro del ecosistema **MAXPOINT V2.0**.  
Su objetivo es centralizar errores reportados por aplicaciones cliente, registrar solicitudes de soporte y asegurar que los problemas detectados puedan ser diagnosticados y solucionados.  

### Funciones principales:
- Recepción de errores generados por procesos internos o externos.  
- Registro de incidencias y trazabilidad en los sistemas de soporte.  
- Validación de estructura de mensajes enviados por las aplicaciones cliente.  
- Priorización de solicitudes de soporte según criticidad.  

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- MXP Init  
- MXP FE Build  
- MXP Printer  

---

## 2. Descripción general del sistema
**Microservicio:** `mxpv2.integration.iasupport`  
- **Rol principal:** gestionar y centralizar incidencias.  
- **Función crítica:** asegurar que los mensajes de error y soporte se reciban, validen y registren para ser gestionados correctamente.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de recepción
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de soporte | R001 | Punto de entrada principal para incidencias y mensajes de error. |

🔑 **R001** actúa como *gateway* para reportes de error y solicitudes de soporte.

---

## 4. Gestión de procesos

### Flujo de trabajo estándar:
1. **Recepción de error** → Se recibe el mensaje de error o incidencia vía endpoint.  
2. **Validación** → Se verifica que el contenido incluya campos obligatorios.  
3. **Registro** → Se almacena y clasifica según prioridad.  
4. **Respuesta** → El sistema confirma la recepción con estado **202 Accepted**.  

---

## 5. Endpoints documentados

### 🛠️ Endpoint: Reportar error matemático
#### ➡️ Método: POST
```
/api/support/error-math
```

#### 📥 Request Body
```json
{
  "error_message": "Division by zero error",
  "solution": "Check for zero before division"
}
```

#### 📌 Descripción de campos
- **error_message** (string) → Mensaje de error reportado.  
- **solution** (string) → Posible solución o recomendación al error.  

#### 📤 Response esperado
```json
{
  "code": 202,
  "status": "success",
  "alert": "Error registrado correctamente"
}
```

---

### 🛠️ Endpoint: Reportar error de conexión
#### ➡️ Método: POST
```
/api/support/error-connection
```

#### 📥 Request Body
```json
{
  "message": "mongo-db:27017: [Errno -3] Try again"
}
```

#### 📌 Descripción de campos
- **message** (string) → Error de conexión detectado.  

#### 📤 Response esperado
```json
{
  "code": 202,
  "status": "success",
  "alert": "Error de conexión recibido y registrado"
}
```

---

### 🛠️ Endpoint: Receiver (incidencias de cliente)
#### ➡️ Método: POST
```
https://apim-uat-bo.azure-api.net/api/support/receiver
```

#### 📥 Request Body
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
- **priority** (number) → Nivel de prioridad (1=alta, 4=baja).  
- **messages** (array) → Lista de mensajes asociados al incidente.  
- **date** (string, ISO8601) → Fecha y hora del reporte.  

#### 📤 Response esperado
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

---

## 6. Escenarios de uso

✅ **Caso exitoso:**  
- Se envía un error bien estructurado → validación correcta → registro exitoso → respuesta con código 202.  

⚠️ **Posibles fallos:**  
- Campos obligatorios faltantes → error en validación.  
- Formato de fecha incorrecto → rechazo del mensaje.  
- Prioridad fuera de rango → incidencia no registrada.  

---

## 7. Preguntas frecuentes (IA Support)

**Q:** ¿Qué pasa si falta un campo obligatorio?  
**A:** El sistema rechaza el request con error de validación.  

**Q:** ¿Cómo sé que se registró la incidencia?  
**A:** El response devuelve código `202` y un mensaje de confirmación.  

**Q:** ¿Qué prioridad se debe usar?  
**A:** Valores entre **1 y 4**, donde 1 es crítica y 4 es baja.  

---

## 8. Referencias
- **Fuentes internas:**  
  - mxpi-support/receptor/readme.md  
  - mxpi-support/error-handling.md  

- **Microservicio:** `mxpv2.integration.iasupport`  

- **Contacto:**  
  - Equipo responsable: Soporte MAXPOINT  
  - Canal de soporte: #mxp-support (Slack interno)  
  - Escalamiento: Arquitectura de datos → Liderazgo técnico  
