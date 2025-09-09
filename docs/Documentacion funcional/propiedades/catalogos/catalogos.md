---


sidebar_label: Catálogo
---

# Módulo de Catálogos (MXP-Catalogs)

## 1. Propósito y alcance
El módulo **Catálogos** en MAXPOINT V2.0 permite administrar listas jerárquicas de información que sirven como referencia para otros procesos dentro del sistema.  

- Proporciona una estructura **padre-hijo** para organizar datos.  
- Permite crear catálogos principales y registros dependientes.  
- Facilita la activación e inactivación de catálogos para un control ordenado.  

ℹ️ Módulos relacionados:
- MXP Integraciones  
- MXP Conexión  
- MXP Entidades  

---

## 2. Descripción general del sistema
- **Microservicio**: `mxpv2.integration.catalogs`  
- **Rol principal**: gestionar listas jerárquicas de valores de referencia.  
- **Función crítica**: mantener catálogos actualizados y organizados para otros módulos.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de gestión
| Componente              | Código | Descripción                                                                    |
|-------------------------|--------|--------------------------------------------------------------------------------|
| Catálogo Padre          | C001   | Categoría principal, independiente de otros catálogos.                         |
| Catálogo Hijo           | C002   | Registro dependiente que se asocia a un catálogo padre.                        |
| Control de estado       | C003   | Permite activar o inactivar catálogos, verificando dependencias jerárquicas.   |

### 3.2. Capa de operación
| Componente                | Código | Descripción                                                          |
|---------------------------|--------|----------------------------------------------------------------------|
| Crear catálogo padre      | C010   | Formulario para registrar un catálogo principal.                     |
| Crear catálogo hijo       | C020   | Formulario para asociar un catálogo a su padre.                      |
| Activar catálogo          | C030   | Cambia el estado de inactivo a activo.                               |
| Inactivar catálogo        | C040   | Deshabilita un catálogo, validando que no tenga hijos activos.        |

---

## 4. Gestión de procesos
**Flujo de trabajo estándar**  
1. **Crear catálogo padre (C010)** → se ingresa código, nombre, valor y se define como padre.  
2. **Crear catálogo hijo (C020)** → se ingresa código, nombre, valor, se define como hijo y se selecciona el padre.  
3. **Listar catálogos** → se consulta la jerarquía padre-hijo en la vista general.  
4. **Activar/Inactivar** → mediante C030 o C040 se controla el estado de los catálogos.  
5. **Confirmación** → el sistema notifica si la acción fue exitosa o si existen dependencias que lo impiden.  

---

## 5. Puntos de integración
- **Entrada**: recibe configuraciones y registros definidos por el usuario.  
- **Salida**: provee valores de referencia para otros módulos (ej. Integraciones, Entidades).  
- **Rol central**: actuar como catálogo maestro de valores jerárquicos.  

---

## 6. Patrones de operación
- **Jerarquía padre-hijo** → los catálogos se organizan en categorías principales y sus dependientes.  
- **Validación de unicidad** → códigos y nombres no pueden repetirse.  
- **Control de dependencias** → un catálogo padre no puede inactivarse si tiene hijos activos.  
- **Reutilización flexible** → los catálogos activos están disponibles para múltiples procesos.  

---

## 7. Escenarios de uso

✅ **Caso exitoso**  
- Crear catálogo padre → ingresar código único, nombre, valor y marcar como padre → guardar.  
- Crear catálogo hijo → ingresar código único, nombre, valor, asociarlo a un padre → guardar.  
- Activar catálogo → seleccionar catálogo inactivo, confirmar → estado cambia a activo.  

⚠️ **Posibles fallos**  
- **Código duplicado**:  
  - El sistema rechaza la creación.  
  - Acción recomendada: usar un código único.  

- **Inactivación con hijos activos**:  
  - El catálogo padre no puede ser inactivado.  
  - Acción recomendada: inactivar o desvincular primero los hijos.  

---

## 8. Preguntas frecuentes (para IA Support)

**Q: ¿Cómo creo un catálogo hijo?**  
A: Ingresa al módulo Catálogo, crea un nuevo registro, asigna el código, nombre, valor y descripción, selecciona **“No”** en Catálogo Padre, elige el padre correspondiente y guarda.  

**Q: ¿Qué pasa si intento inactivar un catálogo padre con hijos activos?**  
A: El sistema no lo permite, primero debes inactivar los catálogos hijos.  

**Q: ¿Puedo reactivar un catálogo inactivo?**  
A: Sí, selecciona el catálogo en la lista, haz clic en **Activar** y confirmará el cambio de estado.  

**Q: ¿Cómo diferencio un catálogo padre de uno hijo?**  
A: El padre no depende de ningún otro catálogo, mientras que el hijo siempre está asociado a un padre.  

---

## 9. Referencias
- `mxp-catalogs/process/readme.md`  
- Documentación interna de MXP Integraciones  
- Microservicio: `mxpv2.integration.catalogs`  

---

## 10. Contacto
- **Equipo responsable**: Integraciones MAXPOINT  
- **Canal de soporte**: `#mxp-integraciones` (Slack interno)  
- **Escalamiento**: Arquitectura de datos → Liderazgo técnico  
