# Módulo de Propagación (MXPi-init Propagation)

## Propósito y alcance

MXPi-init Propagation es el sistema de orquestación de propagación de datos dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo de datos procesados entre diferentes módulos y sistemas destino. Garantiza la correcta distribución de información de menús, preguntas sugeridas, plus, upselling y restricciones a los sistemas destino.

ℹ️ Para más detalles de los módulos relacionados, consulta:
- MXP Init
- MXP FE Build
- Impresora MXP

## 2. Descripción general del sistema

**Microservicio**: mxpv2.integration.propagation

**Rol principal**: gestionar procesos de propagación de datos transformados
**Función crítica**: asegurar que los datos transformados lleguen a los sistemas destino correctos dentro del flujo ETL

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de propagación ETL | F101 | Punto de entrada principal para solicitudes de propagación de datos |

🔑 **El F101 actúa como gateway unificado** para datos procesados, validando estructura e integridad antes de la propagación.

### 3.2. Capa de extracción

| Componente | Código | Descripción |
|------------|--------|-------------|
| Extracción SQL Server | E001 | Extrae datos desde bases de datos SQL Server mediante stored procedures |
| Extracción MongoDB | E002 | Extrae datos desde colecciones MongoDB, principalmente información de UUIDs |
| Extracción Configuraciones | E003 | Extrae configuraciones específicas desde MongoDB |

### 3.3. Capa de transformación

| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación Preguntas Sugeridas | T003 | Procesa y transforma datos de preguntas sugeridas |
| Transformación Plus | T001 | Procesa y transforma datos de productos (PLU) |
| Transformación Menús | T002 | Procesa y transforma datos de menús |
| Transformación Upselling | T004 | Procesa y transforma datos de upselling |
| Transformación Restricciones | T005 | Procesa y transforma datos de restricciones |

### 3.4. Capa de carga

| Componente | Código | Descripción |
|------------|--------|-------------|
| Carga REST API | L001 | Propaga datos a APIs REST externas |
| Carga MongoDB BO | L001 | Propaga datos a MongoDB Back Office |
| Carga MongoDB POS | L002 | Propaga datos a MongoDB Point of Sale |

## 4. Gestión de procesos

### Flujo de trabajo estándar

1. **Ingestión de solicitud** → El F101 recibe la solicitud de propagación
2. **Validación** → Se verifica estructura e integridad de la solicitud
3. **Extracción** → Los datos son extraídos desde las fuentes (SQL Server, MongoDB)
4. **Transformación** → Los datos son procesados y transformados según el tipo
5. **Propagación** → Los datos transformados se envían a los sistemas destino
6. **Confirmación** → Se confirma la correcta propagación de datos

## 5. Puntos de integración

El módulo de propagación conecta múltiples sistemas dentro del ecosistema MAXPOINT V2.0:

**Entradas**:
- Recibe datos procesados desde MXP Init
- Extrae datos desde SQL Server Legacy
- Extrae UUIDs desde MongoDB

**Salidas**:
- Propaga a sistemas REST externos (Propagation Service)
- Propaga a MongoDB Back Office
- Propaga a MongoDB Point of Sale

**Rol central**: mantiene coherencia en la distribución de datos entre sistemas

## 6. Endpoints de Propagación

### 6.1. Propagación de Preguntas Sugeridas del Menú

**POST** `{{server_init}}/api/v1/facade`

**Propósito**: Propagación de preguntas sugeridas y sus respuestas

**Autenticación**:
```http
Authorization: Bearer {{token}}
```

**Body**:
```json
{
    "code": "F101",
    "withResponse": true,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "chunkLoad": 100,
    "processes": {
        "EXTRACTS": [...],
        "TRANSFORMS": [...],
        "LOADS": [...]
    }
}
```

### 6.2. Propagación de PLU del Menú

**POST** `{{server_init}}/api/v1/facade`

**Propósito**: Propagación de productos (PLU) con todas sus relaciones

**Variantes**:
- Propagación a sistema REST externo
- Propagación a MongoDB (BO y POS)

### 6.3. Propagación de Menús

**POST** `{{server_init}}/api/v1/facade`

**Propósito**: Propagación de menús completos con categorías y productos

**Variantes**:
- Propagación a sistema REST externo
- Propagación a MongoDB (BO)

### 6.4. Propagación de Upselling

**POST** `{{server_init}}/api/v1/facade`

**Propósito**: Propagación de reglas de upselling entre productos

### 6.5. Propagación de Restricciones

**POST** `{{server_init}}/api/v1/facade`

**Propósito**: Propagación de restricciones de menús, categorías y productos

## 7. Patrones de propagación

- **Recepción centralizada** → Todas las solicitudes llegan vía F101
- **Extracción múltiple** → Datos provenientes de SQL Server y MongoDB
- **Transformación especializada** → Diferentes transformadores según el tipo de dato
- **Distribución flexible** → Múltiples destinos posibles (REST, MongoDB BO, MongoDB POS)
- **Garantía de coherencia** → Validación de UUIDs y relaciones antes de propagar

## 8. Escenarios de uso

### ✅ Caso exitoso
Solicitud válida → se recibe en F101 → validación → extracción → transformación → propagación correcta → confirmación.

### ⚠️ Posibles fallos

**Error en validación**:
- Datos rechazados en F101
- Se genera log de error con detalle del paquete fallido
- **Acción recomendada**: verificar estructura de la solicitud

**Error en extracción**:
- Fuente de datos no disponible
- **Acción recomendada**: verificar conectividad con bases de datos

**Error en transformación**:
- Datos no pasan validación de transformación
- **Acción recomendada**: revisar logs del transformador específico

**Error en propagación**:
- Sistema destino no disponible
- Se activa política de reintentos
- **Acción recomendada**: verificar conectividad con sistema destino

## 9. Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si falla la validación en F101?**
A: El paquete se descarta, se registra en logs y se debe revisar la estructura de la solicitud.

**Q: ¿Cómo sé que la propagación se realizó correctamente?**
A: Revisa el log de la carga correspondiente (L001, L002). Si hay éxito, se marca con el estado correspondiente.

**Q: ¿El propagador almacena datos?**
A: No, solo orquesta flujos. La persistencia depende de otros módulos.

**Q: ¿Qué debo hacer si la propagación falla consistentemente?**
A: Verificar la conectividad con el sistema destino, revisar logs de error y validar que los datos transformados sean correctos.

**Q: ¿Cómo maneja el sistema los reintentos?**
A: Se activa la política de reintentos configurada, pero si el error persiste, se debe escalar el problema.

## 10. Referencias

**Fuentes internas**:
- mxp-propagation/process/readme.md
- mxp-init/process/readme.md

**Microservicio**: mxpv2.integration.propagation

## 11. Contacto

**Equipo responsable**: Propagación MAXPOINT
**Canal de soporte**: #mxp-propagacion (Slack interno)
**Escalamiento**: Arquitectura de datos → Liderazgo técnico