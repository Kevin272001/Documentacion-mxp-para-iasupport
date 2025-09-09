---
id: administracion-servidores
title: Administración de Servidores
sidebar_label: Servidores
---

# Módulo de Servidores (MXP-Servers)

## 1. Propósito y alcance
El módulo **Servidores** en MaxPoint administra los servidores que sirven como base para establecer conexiones con bases de datos o servicios externos (APIs REST).  

- Permite registrar, listar y editar servidores.  
- Controla atributos críticos como tipo, URL y estado.  
- Proporciona el punto inicial para la construcción de conexiones en el ecosistema MaxPoint.  

ℹ️ Este módulo trabaja en conjunto con:  
- **MXP-Repositories** (fuentes/destinos de datos).  
- **MXP-Adapters** (tipos de conectores).  
- **MXP-Connections** (enlaces que combinan servidores, repositorios y adaptadores).  

---

## 2. Descripción general del sistema
- **Microservicio**: `mxpv2.integration.servers`  
- **Rol principal**: centralizar la administración de servidores disponibles para integraciones.  
- **Función crítica**: asegurar que los parámetros de cada servidor estén correctamente configurados y actualizados.  

---

## 3. Arquitectura y componentes  

### 3.1. Capa de configuración  
| Componente             | Código | Descripción |
|-------------------------|--------|-------------|
| Registro de servidor    | S001   | Permite crear y almacenar un nuevo servidor. |
| Control de estado       | S002   | Gestiona la activación o inactivación de servidores. |

### 3.2. Capa de gestión  
| Componente             | Código | Descripción |
|-------------------------|--------|-------------|
| UI Servidores           | S003   | Interfaz de usuario para listar, crear, editar y gestionar servidores. |

---

## 4. Gestión de procesos  

### Flujo estándar de servidor  
1. **Creación** → se registran código, nombre, tipo, URL y estado.  
2. **Listado** → tabla con todos los servidores disponibles.  
3. **Edición** → se actualizan datos de configuración como tipo o URL.  
4. **Cambio de estado** → activación o inactivación según necesidades de integración.  

---

## 5. Puntos de integración
- **Entrada** → datos configurados por el usuario (nombre, tipo, URL, estado).  
- **Salida** → conexiones (MXP-Connections) que utilizan servidores como base.  
- **Rol central** → servir como base para establecer integraciones de datos dentro del ecosistema MaxPoint.  

---

## 6. Patrones de configuración
- **Identificador único** → cada servidor tiene un código (ejemplo: S0001).  
- **Validación de accesibilidad** → la URL debe corresponder al tipo de servidor (ej. SQL, MongoDB, API REST).  
- **Gestión de ciclo de vida** → en lugar de eliminar, los servidores se marcan como *Inactivos* para mantener el histórico.  

---

## 7. Escenarios de uso  

✅ **Caso exitoso**  
El usuario registra un servidor con datos válidos (ejemplo: nombre *Server SQL Azure*, tipo *SQL Server*) → validación correcta → servidor queda en estado *Activo* → disponible para integraciones.  

⚠️ **Posibles fallos**  
- **Error en creación** → código duplicado.  
  - Acción recomendada: usar un identificador único.  
- **Error en URL** → formato incorrecto o inaccesible.  
  - Acción recomendada: revisar dirección y tipo de servidor.  
- **Error al inactivar** → servidor vinculado a conexiones activas.  
  - Acción recomendada: liberar dependencias antes de inactivar.  

---

## 8. Preguntas frecuentes (para IA Support)  

**Q:** ¿Cómo agrego un nuevo servidor en MaxPoint?  
**A:** Ingresa a *Conexión > Servidores*, haz clic en *Crear servidor*, completa los campos requeridos (código, nombre, tipo, URL, estado) y guarda.  

**Q:** ¿Qué pasa si un servidor ya no se usa?  
**A:** Debe marcarse como *Inactivo* en lugar de eliminarse, para mantener registro histórico.  

**Q:** ¿Qué tipos de servidores se soportan?  
**A:** Entre otros: SQL Server, MongoDB, MariaDB, y APIs REST.  

**Q:** ¿Se puede editar la URL de un servidor?  
**A:** Sí, siempre y cuando se actualicen también las conexiones que dependan de él.  

---

## 9. Referencias  
- `mxpv2.integration.servers/readme.md`  
- Documentación relacionada:  
  - **MXP-Repositories**  
  - **MXP-Adapters**  
  - **MXP-Connections**  

---

## 10. Contacto  
- **Equipo responsable**: Integraciones MaxPoint  
- **Canal de soporte**: #mxp-integraciones (Slack interno)  
- **Escalamiento**: Arquitectura de datos → Liderazgo técnico  
