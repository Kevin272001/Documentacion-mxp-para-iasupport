---


sidebar_label:   Propiedades
---

# Módulo de Administración de Propiedades (MXP-Properties)

## 1. Propósito y alcance
El módulo **MXP Properties** gestiona las propiedades asociadas a entidades dentro del ecosistema **MAXPOINT V2.0**.

- Permite **listar, crear, editar, activar e inactivar** propiedades.
- Asegura la coherencia en el uso de tipos de datos y estados.
- Mantiene un catálogo único de propiedades asociadas a entidades.
- Facilita su uso en integraciones y sincronizaciones.

ℹ️ Para más detalles de módulos relacionados, consulta:
- **MXP Entidades**
- **MXP Conexión**
- **MXP Integraciones**

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.config.properties`  
- **Rol principal:** gestionar el ciclo de vida de propiedades asociadas a entidades.  
- **Función crítica:** asegurar que las propiedades estén disponibles, activas y correctamente configuradas.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de administración
| Componente            | Código | Descripción |
|------------------------|--------|-------------|
| Listar propiedades     | P001   | Muestra una tabla con propiedades registradas (código, entidad, nombre, tipo, estado). |
| Crear propiedad        | P002   | Permite registrar una nueva propiedad asociada a una entidad. |
| Editar propiedad       | P003   | Modifica los datos de una propiedad existente. |
| Activar propiedad      | P004   | Cambia el estado de una propiedad inactiva a activa. |
| Inactivar propiedad    | P005   | Inhabilita una propiedad sin eliminarla. |

---

## 4. Gestión de procesos

**Flujo estándar de trabajo:**
1. **Listado** → El sistema muestra propiedades registradas.  
2. **Creación** → Se registra una nueva propiedad asociada a una entidad.  
3. **Edición** → Se actualizan los valores de una propiedad existente.  
4. **Activación** → Una propiedad inactiva se habilita para uso en procesos.  
5. **Inactivación** → Una propiedad activa se deshabilita, pero permanece consultable.  

---

## 5. Puntos de integración
El módulo de propiedades conecta con otros sistemas de MAXPOINT:

- **Entrada:** configuración de entidades y datos de conexión.  
- **Salida:** propiedades disponibles para procesos de integración.  
- **Rol central:** mantener coherencia y disponibilidad en todo el ecosistema.  

---

## 6. Patrones de administración
- **Registro único** → No se permiten duplicados dentro de la misma entidad.  
- **Validación de tipos** → Cada propiedad debe tener un tipo de dato válido (`string`, `uuid`, `decimal`, etc.).  
- **Estados gestionados** → Activo / Inactivo para control de uso.  
- **Persistencia controlada** → Las propiedades inactivas no se eliminan, permanecen para consulta.  

---

## 7. Escenarios de uso

✅ **Caso exitoso**  
- Se crea una propiedad válida → guardado correcto → queda disponible para integraciones.  

⚠️ **Posibles fallos**  
- **Duplicado:** intento de registrar misma propiedad en una entidad.  
- **Inactivación fallida:** propiedad vinculada a proceso activo.  
- **Activación fallida:** conflicto con entidad o tipo incompatible.  

---

## 8. Preguntas frecuentes (IA Support)

**Q: ¿Dónde veo las propiedades registradas?**  
A: En **Configurador > Conexión > Propiedades**, donde se listan con código, entidad, nombre, tipo y estado.  

**Q: ¿Cómo agrego una propiedad?**  
A: Haz clic en **Crear propiedad**, completa los campos (entidad, nombre, tipo) y guarda.  

**Q: ¿Qué pasa si registro una propiedad duplicada?**  
A: El sistema bloquea el registro y muestra error.  

**Q: ¿Puedo eliminar una propiedad?**  
A: No. Solo puede **inactivarse** para consulta histórica.  

**Q: ¿Cómo sé si una propiedad está activa?**  
A: Aparecerá en estado **Activo** y podrá ser usada en procesos de integración.  

---

## 9. Referencias
Fuentes internas:
- `mxpv2/config/properties/readme.md`
- `mxpv2/entities/process/readme.md`
- Microservicio: `mxpv2.config.properties`

---

## 10. Contacto
- **Equipo responsable:** Configuración e Integraciones MAXPOINT  
- **Canal de soporte:** `#mxp-config` (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico
