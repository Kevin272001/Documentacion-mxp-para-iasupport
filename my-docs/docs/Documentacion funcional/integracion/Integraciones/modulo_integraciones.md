---
title: Integraciones
description: Manual técnico y de usuario para la gestión de integraciones en MaxPoint.
updated: 2025-03-14
tags: [integraciones, gestión, usuario, MaxPoint, IA-support]
sidebar_position: 2
---

# Módulo de Integraciones (MXP-Integrations)

## 1. Propósito y alcance

El módulo de **Integraciones** en MaxPoint permite gestionar la creación, edición, activación/inactivación y visualización de integraciones entre sistemas y procesos de datos.
- Facilita la administración centralizada de integraciones.
- Garantiza la trazabilidad y consistencia de los procesos asociados.

---

## 2. Descripción técnica

- **Microservicio:** mxpv2.integration.integrations
- **Rol principal:** Administrar la configuración y estado de las integraciones.
- **Función crítica:** Permitir la vinculación de procesos, lotes y respuestas entre sistemas.

---

## 3. Componentes y arquitectura

| Componente              | Código | Descripción                                                                 |
|-------------------------|--------|-----------------------------------------------------------------------------|
| Integración lógica      | I001   | Relación entre integrador, procesos, país, franquicia y lotes configurados. |
| Control de estado       | I002   | Gestión de estados: Activo/Inactivo con validación de dependencias.          |
| UI de Integraciones     | I003   | Interfaz para crear, editar, activar/inactivar y visualizar integraciones.   |

---

## 4. Gestión de procesos

### 4.1. Listar Integraciones

Al acceder al módulo, se muestra una tabla con las integraciones registradas:

| **Campo**           | **Descripción**                                                                                  |
|---------------------|-------------------------------------------------------------------------------------------------|
| Código              | Código único de la integración.                                                                 |
| Nombre              | Nombre identificador de la integración.                                                         |
| Integrador          | Integrador seleccionado del catálogo.                                                           |
| Procesos            | Lista de procesos asociados, separados por comas.                                               |
| País                | País relacionado (si aplica).                                                                   |
| Franquicia          | Franquicia asociada (si aplica).                                                                |
| Respuestas          | Indica si el sistema retorna respuesta tras procesar datos: Sí o No.                            |
| Estado              | Estado actual: Activo 🟢 o Inactivo 🔴.                                                          |
| Acciones            | Opciones para editar, activar/inactivar, o visualizar contenido generado.                       |

:::tip Filtrado Rápido
Utiliza la barra de búsqueda para localizar integraciones específicas por nombre.
:::

---

### 4.2. Crear una Integración

1. Haz clic en **"Crear integración"**.
2. Completa los campos:
   - Integrador (del catálogo)
   - Nombre de la integración (único y significativo)
   - Lotes de carga (define tamaño y cantidad según datos)
   - País y franquicia (opcional)
   - Retornar una respuesta (Sí/No)
3. Asigna procesos:
   - Selecciona tipo de proceso (Carga, Extracción, Transformación)
   - Elige el proceso específico y haz clic en **"Agregar proceso"**
4. Haz clic en **"Guardar"** para registrar la integración.
5. El sistema mostrará confirmación si la operación es exitosa.

:::caution
Una vez creada, la integración no se puede eliminar desde la interfaz de usuario.
:::

---

### 4.3. Acciones Disponibles

#### Visualizar contenido generado

- Haz clic en "Visualizar" en la columna de acciones.
- Se abre una ventana con el contenido generado.
- Puedes copiar el contenido para uso externo.

#### Activar/Inactivar integración

- Haz clic en "Activar/Inactivar".
- Si la integración está vinculada a una sincronización bajo el mismo código, se mostrará una alerta y no se permitirá inactivar.
- Si no hay vínculo, confirma la acción en la ventana emergente.

#### Editar integración

- Haz clic en "Editar" en la tabla.
- Modifica los campos necesarios y guarda los cambios.

---

## 5. Comportamiento del Sistema

| **Funcionalidad**       | **Descripción**                                                                                          |
|-------------------------|----------------------------------------------------------------------------------------------------------|
| Validación de dependencias | No se permite inactivar integraciones vinculadas a sincronizaciones activas.                        |
| Visualización de contenido | Permite revisar y copiar el contenido generado por la integración.                                  |
| Edición controlada         | Solo se pueden modificar campos permitidos; no se puede eliminar la integración desde la UI.        |

---

## 6. Patrones y restricciones

- Cada integración tiene un identificador único.
- No se puede eliminar integraciones desde la interfaz.
- No se puede inactivar integraciones vinculadas a sincronizaciones activas.

---

## 7. Escenarios de uso

✅ **Caso exitoso:**  
Creación de integración con procesos y lotes correctamente configurados, activación y visualización de contenido.

⚠️ **Posibles fallos:**  
- Error por nombre duplicado o campos incompletos.
- Intento de inactivar integración vinculada a sincronización activa.
- Edición de campos no permitidos.

---

## 8. Preguntas frecuentes (FAQ)

**¿Cómo creo una nueva integración en MaxPoint?**  
Haz clic en **"Crear integración"**, completa todos los campos requeridos (integrador, nombre, lotes, país, franquicia, respuesta) y asigna los procesos necesarios. Haz clic en **"Guardar"** y verifica el mensaje de confirmación.

**¿Qué campos son obligatorios al crear una integración?**  
Debes completar:  
- Integrador  
- Nombre de la integración  
- Lotes de carga  
- Procesos asociados  
Opcionalmente puedes agregar país y franquicia.

**¿Puedo eliminar una integración desde la interfaz?**  
No. Por seguridad y trazabilidad, las integraciones no pueden eliminarse desde la interfaz de usuario.

**¿Cómo visualizo el contenido generado por una integración?**  
En la tabla de integraciones, haz clic en **"Visualizar"** en la columna de acciones. Se abrirá una ventana con el contenido generado, que puedes revisar y copiar.

**¿Por qué no puedo inactivar una integración?**  
Si la integración está vinculada a una sincronización activa bajo el mismo código, el sistema mostrará una alerta y bloqueará la acción. Debes liberar la dependencia antes de inactivar.

**¿Cómo edito una integración existente?**  
Haz clic en **"Editar"** en la tabla, modifica los campos necesarios y guarda los cambios.

**¿Qué información puedo ver en la tabla de integraciones?**  
La tabla muestra:  
- Código  
- Nombre  
- Integrador  
- Procesos asociados  
- País  
- Franquicia  
- Respuestas  
- Estado  
- Acciones disponibles (editar, activar/inactivar, visualizar)

**¿Qué hacer si tengo dudas o problemas con el módulo?**  
Contacta al equipo responsable de Integraciones MaxPoint a través del canal de soporte interno #mxp-integraciones (Slack).

---

## 9. Referencias y contacto

- Documentación relacionada: Sincronizaciones, Procesos, Franquicias, Empresas
- Soporte: #mxp-integraciones (Slack interno)
- Equipo responsable: Integraciones MaxPoint

---


