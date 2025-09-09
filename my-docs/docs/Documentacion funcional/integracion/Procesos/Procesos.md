---
title: Procesos
description: Manual técnico y de usuario para la gestión de procesos en MaxPoint.
updated: 2025-03-14
tags: [procesos, gestión, usuario, MaxPoint, IA-support]
sidebar_position: 3.3
---

# Módulo de Administración de Procesos (MXP-ProcessAdmin)

## 1. Propósito y alcance

El módulo de **Administración de Procesos** en MaxPoint permite gestionar, crear, editar, visualizar y administrar el estado de los procesos operativos del sistema.
- Facilita la administración centralizada de procesos.
- Permite aplicar filtros, visualizar detalles y modificar configuraciones.

:::info Permisos requeridos
El usuario debe contar con los permisos necesarios para realizar acciones dentro de este módulo.
:::

---

## 2. Descripción técnica

- **Microservicio:** mxpv2.integration.processes
- **Rol principal:** Administrar la configuración y estado de los procesos.
- **Función crítica:** Permitir la gestión eficiente de procesos de carga, extracción y transformación.

---

## 3. Componentes y arquitectura

| Componente              | Código | Descripción                                                                 |
|-------------------------|--------|-----------------------------------------------------------------------------|
| Proceso lógico          | PR001  | Configuración y administración de procesos operativos.                      |
| Control de estado       | PR002  | Gestión de estados: Activo/Inactivo con validación de dependencias.         |
| UI de Procesos          | PR003  | Interfaz para crear, editar, visualizar y administrar procesos.             |

---

## 4. Gestión de procesos

### 4.1. Filtros de Búsqueda

El sistema permite aplicar filtros dinámicos para localizar procesos por nombre o código, facilitando la gestión eficiente.

### 4.2. Lista de Procesos

Cada proceso incluye los siguientes campos:
- **Código:** Identificador único, generado automáticamente.
- **Nombre:** Definido por el usuario.
- **Tipo de Proceso:** Extracción, Carga, Transformación.
- **Conexión:** Base de datos o API asociada.
- **Estado:** Activo/Inactivo.

:::tip Filtrado Rápido
Utiliza la barra de búsqueda para localizar procesos específicos por su nombre o código.
:::

---

### 4.3. Acciones disponibles

#### Editar un Proceso

1. Selecciona el proceso en la lista.
2. Haz clic en el botón de Editar 📝.
3. Modifica los campos necesarios según el tipo de proceso.
4. Guarda los cambios y confirma la edición.

#### Visualizar un Proceso

1. Ubica el proceso en la lista.
2. Haz clic en el botón de Visualizar 👁️.
3. Revisa la configuración detallada del proceso.

#### Cambiar el Estado de un Proceso

1. Haz clic en el botón de Activar/Inactivar 👁️‍🗨️.
2. Confirma la acción en el mensaje emergente.
3. El estado cambiará y se mostrará un mensaje de actualización exitosa.

:::caution Advertencia
No es posible inactivar un registro vinculado a una integración.
:::

---

## 5. Comportamiento del Sistema

| **Funcionalidad**       | **Descripción**                                                                                          |
|-------------------------|----------------------------------------------------------------------------------------------------------|
| Validación de campos    | El sistema verifica que todos los campos obligatorios estén completos.                                   |
| Confirmación de cambios | Mensaje de actualización exitosa tras editar o cambiar estado.                                           |
| Control de dependencias | No se permite inactivar procesos vinculados a integraciones.                                             |

---

## 6. Patrones y restricciones

- Cada proceso tiene un identificador único.
- No se puede inactivar procesos vinculados a integraciones.
- La edición solo está disponible para procesos previamente creados.

---

## 7. Escenarios de uso

✅ **Caso exitoso:**  
Creación, edición y visualización de procesos con confirmación de cambios.

⚠️ **Posibles fallos:**  
- Error por campos incompletos o mal configurados.
- Intento de inactivar procesos vinculados a integraciones.

---

## 8. Preguntas frecuentes (FAQ)

**¿Cómo creo un proceso en MaxPoint?**  
Ingresa al módulo desde `Configurador > Procesos`. Haz clic en **"Crear Nuevo Proceso"**, completa todos los campos obligatorios (nombre, tipo, conexión, estado) y guarda la configuración. El sistema mostrará un mensaje de confirmación si todo está correcto.

**¿Qué campos son obligatorios al crear un proceso?**  
Debes completar:  
- Nombre  
- Tipo de Proceso  
- Conexión  
- Estado  
El código se genera automáticamente.

**¿Puedo editar cualquier proceso?**  
Sí, puedes editar los procesos creados desde la interfaz, siempre y cuando no estén bloqueados por dependencias o reglas del sistema.

**¿Cómo visualizo la configuración de un proceso?**  
Haz clic en el botón de Visualizar 👁️ en la lista de procesos para ver todos los detalles sin modificar la información.

**¿Por qué no puedo inactivar un proceso?**  
No es posible inactivar procesos que estén vinculados a una integración activa. Debes liberar la dependencia antes de cambiar el estado.

**¿Qué sucede si el proceso falla al guardar o editar?**  
El sistema mostrará un mensaje de error y no permitirá guardar los cambios hasta que todos los campos estén correctamente configurados.

**¿Cómo puedo buscar o filtrar procesos?**  
Utiliza la barra de búsqueda para localizar procesos por nombre o código. Puedes ajustar la paginación para mostrar más o menos registros según tus necesidades.

**¿Qué información puedo ver en la lista de procesos?**  
La lista muestra:  
- Código  
- Nombre  
- Tipo de Proceso  
- Conexión  
- Estado  
- Acciones disponibles (editar, visualizar, cambiar estado)

**¿Qué hacer si tengo dudas o problemas con el módulo?**  
Contacta al equipo responsable de Integraciones MaxPoint a través del canal de soporte interno #mxp-integraciones (Slack).

---

## 9. Referencias y contacto

- Documentación relacionada: Integraciones, Sincronizaciones, Conexiones
- Soporte: #mxp-integraciones (Slack interno)
- Equipo responsable: Integraciones MaxPoint

---

## 10. Última Actualización

Este manual fue actualizado por última vez el **14 de marzo de