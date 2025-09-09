---
title: Sincronizaciones
description: Manual técnico y de usuario para la gestión de sincronizaciones en MaxPoint.
updated: 2025-03-14
tags: [sincronización, gestión, usuario, MaxPoint, integraciones, IA-support]
sidebar_position: 1
---

# Módulo de Sincronizaciones (MXP-Synchronizations)

## 1. Propósito y alcance

El módulo de **Sincronizaciones** en MaxPoint permite gestionar la ejecución y programación de procesos de integración entre sistemas y fuentes de datos.  
- Permite listar, crear, editar, ejecutar y eliminar sincronizaciones.
- Garantiza la correcta transferencia y trazabilidad de datos.
- Asegura control y consistencia en los procesos de integración.

:::info Permisos requeridos
El usuario debe contar con los permisos adecuados para realizar acciones dentro del módulo de sincronizaciones.
:::

---

## 2. Descripción técnica

- **Microservicio:** mxpv2.integration.synchronizations
- **Rol principal:** Administrar la ejecución y programación de sincronizaciones.
- **Función crítica:** Coordinar el flujo de datos y mantener la consistencia entre orígenes y destinos.

---

## 3. Componentes y arquitectura

| Componente              | Código | Descripción                                                                 |
|-------------------------|--------|-----------------------------------------------------------------------------|
| Sincronización lógica   | S001   | Relación entre integraciones, empresa, franquicia y restaurante configurados.|
| Control de estado       | S002   | Gestión de estados: Programado, En ejecución, Exitoso, Error, Cancelado.    |
| UI de Sincronizaciones  | S003   | Interfaz para crear, editar, ejecutar y visualizar sincronizaciones.         |

---

## 4. Gestión de procesos

### 4.1. Listar Sincronizaciones

Al acceder al módulo, se muestra una tabla con las sincronizaciones registradas:

| **Campo**               | **Descripción**                                                                                          |
|------------------------|----------------------------------------------------------------------------------------------------------|
| Nombre                 | Nombre de la sincronización.                                                                             |
| Hora de Sincronización | Fecha y hora de ejecución o programación (`dd/mm/yyyy hh:mm:ss`).                                        |
| Estados                | Estado actual: Exitoso, En ejecución, Error, Cancelado, Programado.                                      |
| Observaciones          | Resultado o comentarios de la sincronización.                                                            |
| Acciones               | Opciones para editar, ejecutar o eliminar la sincronización.                                             |

:::tip Filtrado y Navegación
Usa la barra de búsqueda para localizar sincronizaciones específicas por nombre. Ajusta la paginación entre 10, 20, 50 o 100 registros.
:::



### 4.2. Crear una Sincronización

#### Pasos para Crear

1. Navega a `Configurador` > `Sincronizaciones`.
2. Haz clic en **"Nueva Sincronización"**.
3. Completa los siguientes campos:
   - Nombre de Sincronización (alfanumérico, máx. 100 caracteres)
   - Integraciones (una o varias)
   - Estado
   - País
   - Empresa
   - Franquicia
   - Restaurante
   - Programar Sincronización (switch para fecha/hora específica)
4. Haz clic en **"Guardar"**.
5. Si programaste la sincronización, se mostrará un resumen con los detalles registrados.

:::caution
No se pueden editar sincronizaciones que ya hayan sido ejecutadas.
:::



---

### 4.3. Sincronización Programada

#### Pasos para Programar

1. Accede al módulo.
2. Haz clic en **"Crear sincronización"**.
3. Completa los campos requeridos.
4. Activa el switch de programación y selecciona fecha/hora.
5. Haz clic en **"Guardar"**.

:::important
Completa todos los campos obligatorios para evitar errores al guardar.
:::

Al guardar, confirma el mensaje y finaliza la programación.


---

### 4.4. Acciones Disponibles

#### Editar

- Solo sincronizaciones en estado *Programado*.
- Modifica campos y guarda.



#### Ejecutar

- Disponible para cualquier estado.
- Confirma ejecución en el cuadro de diálogo.

:::tip
Antes de ejecutar, revisa las observaciones en caso de sincronizaciones fallidas.
:::

#### Eliminar

- Solo sincronizaciones no ejecutadas.
- Confirma en ventana emergente.

:::warning
No se pueden eliminar sincronizaciones ya ejecutadas.
:::

---

## 5. Comportamiento del Sistema

| **Funcionalidad**       | **Descripción**                                                                                          |
|-------------------------|----------------------------------------------------------------------------------------------------------|
| Errores                 | Estado cambia a *Error* con mensaje específico.                                                          |
| Límite de Ejecuciones   | Máximo 4 ejecuciones manuales.                                                                          |
| Intentos Automáticos    | Hasta 3 intentos automáticos en caso de error.                                                          |
| Fechas Automáticas      | Fecha/hora asignadas automáticamente si no se programa.                                                 |

---

## 6. Patrones y restricciones

- Identificador único por sincronización.
- No se puede editar/eliminar sincronizaciones ejecutadas.
- Gestión centralizada desde el módulo.

---

## 7. Escenarios de uso

**Caso exitoso:**  
Creación y ejecución correcta de una sincronización.

 **Posibles fallos:**  
- Campos incompletos o integraciones no seleccionadas.
- Error en ejecución por datos incorrectos.
- Restricciones de edición/eliminación.

---

## 8. Preguntas frecuentes (FAQ)

**¿Cómo creo una nueva sincronización en MaxPoint?**  
Para crear una sincronización, ingresa al módulo desde `Configurador > Sincronizaciones`. Haz clic en **"Nueva Sincronización"**, completa todos los campos obligatorios (nombre, país, empresa, franquicia, restaurante, integraciones) y, si lo deseas, activa el switch para programar la sincronización en una fecha y hora específica. Finalmente, haz clic en **"Guardar"**. El sistema mostrará un resumen y confirmará el registro exitoso.

**¿Qué campos son obligatorios al crear una sincronización?**  
Debes completar:  
- Nombre de Sincronización  
- País  
- Empresa  
- Franquicia  
- Restaurante  
- Integraciones  
Si programas la sincronización, también debes definir la fecha y hora.

**¿Puedo editar cualquier sincronización?**  
No. Solo puedes editar sincronizaciones que estén en estado *Programado*. Para editar, localiza la sincronización en la tabla, haz clic en el ícono de editar, modifica los campos necesarios y guarda los cambios.

**¿Cómo ejecuto una sincronización manualmente?**  
Ubica la sincronización en la tabla y haz clic en el botón **"Ejecutar"** (ícono de flecha circular). Confirma la acción en el cuadro de diálogo. El sistema procesará la sincronización inmediatamente y actualizará el estado.

**¿Por qué no puedo eliminar una sincronización?**  
Solo puedes eliminar sincronizaciones que no hayan sido ejecutadas. Si la sincronización ya fue ejecutada, el sistema bloqueará la opción de eliminar.

**¿Qué sucede si la sincronización falla?**  
El estado cambiará a *Error* y se mostrará un mensaje en las observaciones. El sistema intentará sincronizar automáticamente hasta 3 veces. Si el error persiste, revisa la configuración y los datos ingresados antes de volver a ejecutar.

**¿Cuántas veces puedo ejecutar una sincronización manualmente?**  
Puedes ejecutar una sincronización manualmente hasta 4 veces. Si se supera este límite, el sistema bloqueará nuevas ejecuciones manuales.

**¿Cómo puedo buscar o filtrar sincronizaciones?**  
Utiliza la barra de búsqueda para localizar sincronizaciones por nombre. Puedes ajustar la paginación para mostrar 10, 20, 50 o 100 registros según tus necesidades.

**¿Qué información puedo ver en la tabla de sincronizaciones?**  
La tabla muestra:  
- Nombre  
- Hora de Sincronización  
- Estado actual  
- Observaciones  
- Acciones disponibles (editar, ejecutar, eliminar)

**¿Qué hacer si tengo dudas o problemas con el módulo?**  
Contacta al equipo responsable de Integraciones MaxPoint a través del canal de soporte interno #mxp-integraciones (Slack).

---

## 9. Referencias y contacto

- Documentación relacionada: Integraciones, Empresas, Franquicias, Restaurantes
- Soporte: #mxp-integraciones (Slack interno)
- Equipo responsable: Integraciones MaxPoint

---

## 10. Última Actualización

Este manual fue actualizado por última vez el **14 de marzo de 2025**.
