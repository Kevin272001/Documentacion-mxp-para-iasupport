---


sidebar_label: Entidades 
---

# Módulo de Administración de Entidades (MXP-Entities)

## 1. Propósito y alcance
El módulo **Administración de Entidades** en MAXPOINT V2.0 permite gestionar las entidades de repositorios de manera centralizada.  
Su función principal es facilitar la creación, edición, activación, inactivación y consulta de entidades registradas en el sistema.  

- Garantiza que las entidades estén organizadas y disponibles para los procesos de integración.  
- Permite mantener la coherencia de repositorios y sus componentes.  
- Asegura que las entidades activas estén listas para ser utilizadas en flujos de conexión e integración.  

ℹ️ Módulos relacionados:
- MXP Init  
- MXP Conexión  
- MXP Integraciones  

---

## 2. Descripción general del sistema
- **Microservicio**: `mxpv2.integration.entities`  
- **Rol principal**: administrar repositorios y sus entidades.  
- **Función crítica**: permitir el ciclo de vida completo de una entidad (alta, modificación, activación e inactivación).  

---

## 3. Arquitectura y componentes

### 3.1. Capa de gestión
| Componente           | Código | Descripción                                                                 |
|----------------------|--------|-----------------------------------------------------------------------------|
| Listado de entidades | E001   | Tabla centralizada que muestra repositorios, entidades, tipos y estados.   |
| Buscador             | E002   | Permite localizar entidades por nombre.                                    |
| Paginación           | E003   | Controla la cantidad de registros visibles (10, 20, 50, 100).              |

### 3.2. Capa de operación
| Componente        | Código | Descripción                                                            |
|-------------------|--------|------------------------------------------------------------------------|
| Crear entidad     | E010   | Permite registrar una nueva entidad en un repositorio.                 |
| Editar entidad    | E020   | Modifica datos existentes de la entidad seleccionada.                  |
| Activar entidad   | E030   | Cambia el estado de una entidad inactiva a activa.                     |
| Inactivar entidad | E040   | Deshabilita la entidad para procesos, manteniéndola visible en consulta.|

---

## 4. Gestión de procesos
**Flujo de trabajo estándar**  
1. **Listar entidades** → E001 muestra todas las entidades registradas.  
2. **Filtrar/Buscar** → E002 localiza la entidad específica.  
3. **Crear/Editar** → mediante E010 o E020 se registran o modifican entidades.  
4. **Activar/Inactivar** → E030 y E040 actualizan el estado de la entidad.  
5. **Confirmación** → el sistema muestra notificación del resultado de la acción.  

---

## 5. Puntos de integración
- **Entrada**: recibe datos de repositorios configurados en MXP Conexión.  
- **Salida**: expone entidades listas para integraciones y procesos ETL en MAXPOINT.  
- **Rol central**: organiza y asegura la consistencia de entidades entre repositorios.  

---

## 6. Patrones de operación
- **Gestión centralizada** → todas las entidades se administran en un único módulo.  
- **Validación de duplicados** → evita que dos entidades con mismo nombre/repositorio convivan.  
- **Control de estado** → entidades activas son utilizables, inactivas quedan disponibles solo para consulta.  
- **Seguridad de cambios** → toda acción requiere confirmación del usuario.  

---

## 7. Escenarios de uso

✅ **Caso exitoso**  
- Crear una nueva entidad → se completa formulario → se valida unicidad → se guarda → aparece en la lista.  
- Editar entidad → se modifica nombre o tipo → se guarda → sistema actualiza la información.  
- Activar entidad → usuario selecciona el ícono → confirma → estado cambia a Activo.  

⚠️ **Posibles fallos**  
- **Entidad duplicada**:  
  - Se rechaza la creación.  
  - Acción recomendada: usar un nombre único.  

- **Conflicto al activar**:  
  - La entidad no puede activarse por conflicto con repositorio.  
  - Acción recomendada: verificar integridad del repositorio.  

- **Inactivación bloqueada**:  
  - No se permite inactivar si la entidad está vinculada a un proceso activo.  
  - Acción recomendada: desvincular primero la entidad.  

---

## 8. Preguntas frecuentes (para IA Support)

**Q: ¿Cómo veo las entidades registradas?**  
A: En **Configurador > Conexión > Entidades**, verás una tabla con repositorio, entidad, tipo y estado.  

**Q: ¿Cómo creo una entidad nueva?**  
A: Haz clic en **Crear entidad**, completa los campos (repositorio, nombre, tipo) y guarda los cambios.  

**Q: ¿Cómo edito una entidad existente?**  
A: Selecciona el ícono de lápiz (✏️), modifica los datos y guarda.  

**Q: ¿Qué pasa si intento crear una entidad duplicada?**  
A: El sistema no lo permite, debes usar un nombre único.  

**Q: ¿Cómo inactivo una entidad?**  
A: Haz clic en el ícono de inactivar, confirma la acción y quedará en estado Inactivo.  

---

## 9. Referencias
- `mxp-entities/process/readme.md`  
- Documentación interna de MXP Conexión  
- Microservicio: `mxpv2.integration.entities`  

---

## 10. Contacto
- **Equipo responsable**: Integraciones MAXPOINT  
- **Canal de soporte**: `#mxp-integraciones` (Slack interno)  
- **Escalamiento**: Arquitectura de datos → Liderazgo técnico  
