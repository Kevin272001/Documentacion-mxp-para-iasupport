---
id: administracion-adaptadores
title: Administración de Adaptadores
sidebar_label: Adaptadores
---

# Módulo de Adaptadores (MXP-Adapters)

## 1. Propósito y alcance
El módulo **Adaptadores** en MaxPoint permite gestionar los diferentes **conectores** usados para la integración de sistemas.  

- Habilita la comunicación entre fuentes de datos externas (Bases de datos, APIs).  
- Asegura la disponibilidad y consistencia en los procesos ETL.  
- Permite crear, editar, activar o inactivar adaptadores según las necesidades de integración.  

ℹ️ Para un correcto funcionamiento, este módulo se relaciona con:  
- **MXP-Servers** (servidores registrados).  
- **MXP-Repositories** (repositorios disponibles).  
- **MXP-Connections** (conexiones que enlazan adaptadores con otros componentes).  

---

## 2. Descripción general del sistema
- **Microservicio**: `mxpv2.integration.adapters`  
- **Rol principal**: administrar adaptadores disponibles en la plataforma.  
- **Función crítica**: garantizar que existan conectores válidos y actualizados para las integraciones de datos.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de configuración  
| Componente           | Código | Descripción |
|----------------------|--------|-------------|
| Registro de adaptador| A001   | Permite crear un nuevo adaptador en la plataforma. |
| Control de estado    | A002   | Maneja activación/inactivación de adaptadores. |

### 3.2. Capa de gestión  
| Componente          | Código | Descripción |
|---------------------|--------|-------------|
| UI Adaptadores      | A003   | Interfaz en MaxPoint para crear, editar, listar y visualizar adaptadores. |

---

## 4. Gestión de procesos  

### Flujo estándar de adaptador  
1. **Creación** → el usuario define Código, Tipo, Nombre y Versión.  
2. **Validación** → el sistema verifica duplicados por nombre y versión.  
3. **Activación** → el adaptador queda disponible para conexiones y procesos ETL.  
4. **Edición** → permite actualizar información (tipo, nombre, versión).  
5. **Inactivación** → deshabilita el adaptador sin eliminarlo físicamente.  

---

## 5. Puntos de integración
El módulo conecta con:  
- **Entrada** → servidores y configuraciones definidas en la plataforma.  
- **Salida** → conexiones (MXP-Connections) que consumen los adaptadores.  
- **Rol central** → proveer conectores válidos para garantizar comunicación entre sistemas externos y MaxPoint.  

---

## 6. Patrones de configuración
- **Identificador único** → cada adaptador se registra con un código único (ej. A0001).  
- **Dependencia controlada** → un adaptador no se puede eliminar si está vinculado a una conexión activa.  
- **Gestión centralizada** → todos los adaptadores se administran desde un único módulo en MaxPoint.  

---

## 7. Escenarios de uso  

✅ **Caso exitoso**  
El usuario crea un adaptador con datos válidos → validación correcta → queda en estado *Activo* → disponible para conexiones.  

⚠️ **Posibles fallos**  
- **Error al crear** → duplicado de nombre y versión.  
  - Acción recomendada: cambiar nombre o revisar adaptadores existentes.  
- **Error al inactivar** → el adaptador está en uso dentro de una conexión activa.  
  - Acción recomendada: liberar dependencias antes de inactivar.  
- **Error en edición** → campos obligatorios no completados.  
  - Acción recomendada: validar información ingresada.  

---

## 8. Preguntas frecuentes (para IA Support)  

**Q:** ¿Cómo creo un nuevo adaptador en MaxPoint?  
**A:** Ingresa a *Configurador > Integraciones > Adaptadores*, haz clic en *Crear adaptador*, completa Código, Tipo, Nombre y Versión, y guarda.  

**Q:** ¿Puedo inactivar cualquier adaptador?  
**A:** Solo si no está vinculado a una conexión activa.  

**Q:** ¿Qué diferencia hay entre *Inactivar* y *Eliminar*?  
**A:** Inactivar deshabilita el adaptador, pero no lo borra; Eliminar lo remueve permanentemente (solo posible si no tiene dependencias).  

**Q:** ¿Cómo busco un adaptador específico?  
**A:** Usa la barra de búsqueda por código, nombre o tipo en la lista de adaptadores.  

---

## 9. Referencias  
- `mxpv2.integration.adapters/readme.md`  
- Documentación relacionada:  
  - **MXP-Servers**  
  - **MXP-Repositories**  
  - **MXP-Connections**  

---

## 10. Contacto  
- **Equipo responsable**: Integraciones MaxPoint  
- **Canal de soporte**: #mxp-integraciones (Slack interno)  
- **Escalamiento**: Arquitectura de datos → Liderazgo técnico  

