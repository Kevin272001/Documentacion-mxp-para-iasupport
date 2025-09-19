# Módulo de Clientes Legacy (MXPI-Legacy-Clients)

## 1. Propósito y alcance
MXPI-Legacy-Clients es el sistema de gestión de clientes legacy dentro del ecosistema MAXPOINT V2.0. Gestiona el registro, consulta y actualización de información de clientes provenientes de sistemas legacy. Garantiza la integridad y consistencia de los datos de clientes en toda la plataforma.

## 2. Descripción general del sistema
- **Microservicio:** mxpi-legacy-clients  
- **Rol principal:** gestionar operaciones CRUD de clientes legacy  
- **Función crítica:** asegurar la sincronización y validación de datos de clientes entre sistemas legacy y la plataforma moderna  

## 3. Arquitectura y componentes

### 3.1. Endpoints principales

#### POST /api/client
**Descripción:** Crea un nuevo cliente en el sistema legacy

**Request Body:**
```json
{
    "cdn_id": 10,
    "pais": "ECU",
    "sistemaOrigen": "LandingPage",
    "fechaCreacion": "2023-10-24T13:49:44Z",
    "autenticacion": true,
    "documento": "1717171715",
    "tipoDocumento": "CEDULA",
    "email": "test@gmail.com",
    "telefono": "0969458949",
    "primerNombre": "string",
    "segundoNombre": "string",
    "apellidos": "string",
    "fechaNacimiento": "2023-10-24T13:49:44Z",
    "mayor16anios": true,
    "aceptacionPoliticas": true,
    "envioComunicacionesComerciales": true,
    "envioComunicacionesComercialesPush": true,
    "analisisDeDatosPerfiles": true,
    "cesionDatosATercerosNacionales": true,
    "cesionDatosATercerosInternacionales": true,
    "fechaAceptoPrivacidad": "2023-10-24T13:49:44Z",
    "fechaActualizacion": "2023-10-24T13:49:44Z"
}
```

**Validaciones:**
- ✅ Campos principales requeridos
- ✅ Formato de email válido
- ✅ Formato ISO en fechas
- ✅ Validación de duplicados

**Respuesta exitosa:**
```json
{
    "success": true,
    "statusCode": 201,
    "message": "Cliente creado exitosamente",
    "data": { /* datos del cliente creado */ }
}
```

**Respuesta de error (GKFC002):**
```json
{
    "success": false,
    "statusCode": 400,
    "codeError": "GKFC002",
    "message": "Cliente ya se encuentra registrado",
    "data": {}
}
```

---

#### GET /api/client/{cdn_id}/{documento}
**Descripción:** Consulta información detallada de un cliente específico

**Parámetros:**
- `cdn_id` (number): Identificador de la cadena
- `documento` (string): Número de documento del cliente

**Respuesta exitosa:**
```json
{
    "success": true,
    "statusCode": 200,
    "message": "informacion cargada",
    "data": {
        "cliente": {
            "_id": "string",
            "uid": "string",
            "cdn_id": 10,
            "pais": "ECU",
            "sistemaOrigen": "LandingPage",
            "documento": "1717171715",
            "tipoDocumento": "CEDULA",
            "email": "test@gmail.com",
            "telefono": "0969458949",
            "primerNombre": "string",
            "apellidos": "string",
            "fechaNacimiento": "2023-10-24T13:49:44Z",
            "mayor16anios": true,
            "fechaCreacion": "2023-10-24T13:49:44Z",
            "fechaActualizacion": "2023-10-24T13:49:44Z",
            "beneficiosPorCadena": [
                {
                    "aceptoBeneficio": true,
                    "infoBeneficio": {
                        "idBeneficio": "string",
                        "nombrePlu": "string",
                        "pluId": "string",
                        "fechaCanje": "2023-10-24T13:49:44Z",
                        "infoRestaurante": {
                            "nombre_cadena": "string",
                            "local": "string"
                        }
                    }
                }
            ]
        },
        "infoAdicional": {
            "infoBeneficio": {
                "remidioBeneficio": true,
                "message": "redimio su beneficio"
            },
            "infoSistemaOrigen": {},
            "aceptoPoliticas": true
        }
    }
}
```

**Validaciones:**
- ✅ Código HTTP 200
- ✅ Estructura principal válida
- ✅ Datos generales del cliente completos
- ✅ Beneficios por cadena estructurados correctamente
- ✅ Información adicional consistente

---

#### PUT /api/client
**Descripción:** Actualiza la información de un cliente existente

**Request Body:**
```json
{
    "_id": "67fd2eba197cb8a3aa47dd5e",
    "aceptacionPoliticas": true,
    "analisisDeDatosPerfiles": true,
    "apellidos": "TEST BRICEÑO",
    "autenticacion": true,
    "cdn_id": 10,
    "sistemaOrigen": "KIOSKO",
    "cesionDatosATercerosInternacionales": true,
    "cesionDatosATercerosNacionales": true,
    "email": "test@gmail.com",
    "envioComunicacionesComerciales": true,
    "envioComunicacionesComercialesPush": true,
    "fechaAceptoPrivacidad": "2023-10-24T13:49:44.123Z",
    "fechaActualizacion": "2023-10-24T13:49:44.123Z",
    "primerNombre": "string",
    "telefono": "0969458949"
}
```

**Validaciones:**
- ✅ Campos principales requeridos
- ✅ Formato de email válido
- ✅ Formato ISO estricto en fechas (con milisegundos)
- ✅ Longitud válida de teléfono (9-10 dígitos)

**Respuesta exitosa:**
```json
{
    "success": true,
    "statusCode": 200,
    "message": "Cliente actualizado exitosamente",
    "data": { /* datos del cliente actualizado */ }
}
```

**Respuesta de error (GKFC001):**
```json
{
    "success": false,
    "statusCode": 404,
    "codeError": "GKFC001",
    "message": "Cliente no encontrado",
    "data": {}
}
```

---

## 4. Gestión de procesos
**Flujo de trabajo estándar**
1. Validación de entrada → Se verifican campos requeridos y formatos  
2. Verificación de duplicados → Se comprueba si el cliente ya existe  
3. Procesamiento → Los datos se transforman y almacenan  
4. Respuesta → Se retorna el resultado de la operación  

## 5. Puntos de integración
El módulo se conecta con:
- Sistemas Legacy: Para sincronización de datos de clientes
- MXP Sync: Para coordinación de procesos ETL
- Sistemas de Autenticación: Para validación de identidad

## 6. Códigos de error
| Código   | Descripción              | HTTP Status |
|----------|--------------------------|-------------|
| GKFC001  | Cliente no encontrado    | 404         |
| GKFC002  | Cliente ya registrado    | 400         |
| GKFC003  | Datos de entrada inválidos | 400       |
| GKFC004  | Error de validación      | 422         |

## 7. Escenarios de uso
✅ Caso exitoso creación: Request válido → validación → creación → respuesta 201  
✅ Caso exitoso consulta: Parámetros válidos → búsqueda → datos encontrados → respuesta 200  
✅ Caso exitoso actualización: Request válido → validación → actualización → respuesta 200  

⚠️ Posibles fallos:
- Error de validación: Datos rechazados por formato incorrecto  
- Cliente duplicado: Se detecta cliente existente (GKFC002)  
- Cliente no encontrado: No existe el cliente para actualizar (GKFC001)  

## 8. Preguntas frecuentes (para IA Support)
**Q: ¿Qué hacer si recibo error GKFC002?**  
A: Verificar si el cliente ya existe antes de intentar crearlo nuevamente.

**Q: ¿Cómo validar el formato de fechas requerido?**  
A: Las fechas deben estar en formato ISO 8601: YYYY-MM-DDTHH:mm:ss.sssZ

**Q: ¿Qué campos son obligatorios para crear un cliente?**  
A: Todos los campos listados en la sección de validación del POST.

**Q: ¿Cómo manejar errores de teléfono inválido?**  
A: Asegurar que el teléfono tenga entre 9-10 dígitos numéricos.

## 9. Referencias
- Fuentes internas: `mxpi-legacy-clients/receptor/`  
- Microservicio: `mxpi-legacy-clients`  

## 10. Contacto
- **Equipo responsable:** Legacy Integration Team  
- **Canal de soporte:** #mxp-legacy-integration (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico  
