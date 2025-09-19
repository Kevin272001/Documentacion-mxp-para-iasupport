---
id: administrar-conexiones
title: Administración de Conexiones
sidebar_label: Conexiones
---

# Módulo de Conexiones (MXP-Connections)

## 1. Propósito y alcance  
El módulo de **Conexiones** en MaxPoint gestiona los enlaces entre **servidores, adaptadores y repositorios** previamente configurados.  

- Permite crear nuevas conexiones de forma controlada.  
- Garantiza la correcta integración de datos en los procesos de carga.  
- Asegura la disponibilidad y consistencia de las conexiones durante la ejecución de procesos.  

ℹ️ Para un correcto funcionamiento, este módulo depende de:  
- **MXP-Servers** (servidores configurados).  
- **MXP-Adapters** (adaptadores disponibles).  
- **MXP-Repositories** (repositorios registrados).  

---

## 2. Descripción general del sistema  
- **Microservicio**: `mxpv2.integration.connections`  
- **Rol principal**: administrar relaciones entre componentes de integración.  
- **Función crítica**: habilitar la comunicación entre orígenes y destinos de datos a través de configuraciones seguras y gestionables.  

---

## 3. Arquitectura y componentes  

### 3.1. Capa de conexión  
| Componente | Código | Descripción |
|------------|--------|-------------|
| Conexión lógica | C001 | Relación entre servidor, adaptador y repositorio configurados. |
| Control de estado | C002 | Gestión de estados Activo/Inactivo con validación de dependencias. |

### 3.2. Interfaz de gestión  
| Componente | Código | Descripción |
|------------|--------|-------------|
| UI de Conexiones | C003 | Interfaz en MaxPoint que permite crear, editar, activar/inactivar y visualizar conexiones. |

---

## 4. Gestión de procesos  

### Flujo estándar de conexión  
1. **Creación** → el usuario define nombre, servidor, adaptador y repositorio.  
2. **Validación** → el sistema verifica duplicados y dependencias.  
3. **Activación** → la conexión queda disponible para procesos de integración.  
4. **Inactivación** → la conexión se deshabilita si no está vinculada a un proceso activo.  
5. **Edición** → el usuario puede actualizar parámetros (servidor, repositorio, adaptador, descripción).  

---

## 5. Puntos de integración  
El módulo conecta con:  
- **Entrada** → servidores y adaptadores previamente creados.  
- **Salida** → repositorios disponibles para los procesos ETL.  
- **Rol central** → coordinar la disponibilidad de conexiones para los procesos de integración en toda la plataforma.  

---

## 6. Patrones de conexión  
- **Configuración única** → cada conexión tiene un identificador único.  
- **Dependencia controlada** → no se puede inactivar una conexión si está vinculada a un proceso.  
- **Gestión centralizada** → todas las conexiones se administran desde un único módulo en MaxPoint.  

---

## 7. Escenarios de uso  

✅ **Caso exitoso**  
El usuario crea una conexión con servidor, adaptador y repositorio válidos → validación correcta → conexión activa → disponible para procesos.  

⚠️ **Posibles fallos**  
- **Error al crear** → nombre duplicado o parámetros incompletos.  
  - Acción recomendada: revisar que no exista otra conexión con el mismo nombre.  
- **Error al inactivar** → la conexión está vinculada a un proceso activo.  
  - Acción recomendada: liberar dependencias antes de inactivar.  
- **Error en edición** → servidor o repositorio no disponible.  
  - Acción recomendada: validar que los componentes estén activos.  

---

## 8. Preguntas frecuentes (para IA Support)  

**Q:** ¿Cómo creo una nueva conexión en MaxPoint?  
**A:** Ingresa a *Configurador > Integración > Conexión > Conexiones*, haz clic en *Crear conexión*, completa los campos requeridos y guarda. El sistema confirmará la creación.  

**Q:** ¿Puedo inactivar cualquier conexión?  
**A:** No. Solo puedes inactivar conexiones que no estén vinculadas a un proceso activo.  

**Q:** ¿Cómo sé si mi conexión está activa?  
**A:** Revisa la columna **Estado** en la lista de conexiones. Si dice *Activo*, está habilitada.  

**Q:** ¿Qué hacer si al guardar me sale error de duplicado?  
**A:** Cambia el nombre o valida que no exista otra conexión con la misma configuración.  

---

## 9. Referencias  
- `mxpv2.integration.connections/readme.md`  
- Documentación relacionada:  
  - **MXP-Servers**  
  - **MXP-Adapters**  
  - **MXP-Repositories**  

---

## 10. Contacto  
- **Equipo responsable**: Integraciones MaxPoint  
- **Canal de soporte**: #mxp-integraciones (Slack interno)  
- **Escalamiento**: Arquitectura de datos → Liderazgo técnico  
