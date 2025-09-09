---
title: Proceso de Transformación
description: Manual técnico y de usuario para la creación de procesos de transformación en MaxPoint.
updated: 2025-03-14
tags: [proceso, transformación, gestión, usuario, MaxPoint, IA-support]
sidebar_position: 3.2
---

# Módulo de Procesos de Transformación (MXP-TransformProcess)

## 1. Propósito y alcance

El módulo de **Procesos de Transformación** en MaxPoint permite crear y configurar procesos para modificar, enriquecer o adaptar datos entre sistemas.
- Facilita la administración centralizada de procesos de transformación.
- Permite definir reglas y operaciones sobre los datos antes de su carga o extracción.

---

## 2. Descripción técnica

- **Microservicio:** mxpv2.integration.processes
- **Rol principal:** Administrar la creación y configuración de procesos de transformación.
- **Función crítica:** Definir las reglas, operaciones y propiedades para transformar datos.

---

## 3. Componentes y arquitectura

| Componente                  | Código | Descripción                                                                 |
|-----------------------------|--------|-----------------------------------------------------------------------------|
| Proceso de transformación   | P201   | Configuración de reglas, transformaciones y propiedades.                    |
| Control de validación       | P202   | Validación de campos obligatorios y estructura de transformaciones.         |
| UI de Procesos              | P203   | Interfaz para crear, editar y visualizar procesos de transformación.        |

---

## 4. Gestión de procesos

### 4.1. Creación de un Proceso de Transformación

1. Ingresar al Configurador de Procesos (`Procesos > Crear Nuevo Proceso`)
2. Seleccionar **Tipo de Proceso:** Transformación
3. Ingresar las **Transformaciones** necesarias para el proceso
4. Ingresar **Nombre del Proceso** (ejemplo: "Transformación de datos de ventas")
5. Configurar el **Caché:** Sí/No
6. Definir si se **Retorna una Respuesta:** Sí/No
7. Agregar una **Descripción** (opcional)
8. Guardar el Proceso

:::tip Nota
- Todos los campos son obligatorios excepto la descripción.  
- Si algún campo no está configurado correctamente, el sistema podría mostrar un mensaje de error.
:::

---

## 5. Comportamiento del Sistema

| **Funcionalidad**       | **Descripción**                                                                                          |
|-------------------------|----------------------------------------------------------------------------------------------------------|
| Validación de campos    | El sistema verifica que todos los campos obligatorios estén completos.                                   |
| Confirmación de creación| Mensaje "Creado correctamente" al guardar exitosamente.                                                  |
| Control de errores      | Si algún campo no está configurado correctamente, se muestra un mensaje de error.                        |

---

## 6. Patrones y restricciones

- Cada proceso de transformación tiene un identificador único.
- Las transformaciones deben estar correctamente definidas.
- La descripción es opcional pero recomendable para identificar el proceso.

---

## 7. Escenarios de uso

✅ **Caso exitoso:**  
Creación de proceso de transformación con reglas y operaciones correctamente configuradas, validación y confirmación.

⚠️ **Posibles fallos:**  
- Error por campos incompletos o mal configurados.
- Transformaciones mal definidas.
- No agregar descripción puede dificultar la identificación del proceso.

---

## 8. Preguntas frecuentes (FAQ)

**¿Cómo creo un proceso de transformación en MaxPoint?**  
Para crear un proceso de transformación, ingresa al módulo desde `Configurador > Procesos`. Haz clic en **"Crear Nuevo Proceso"**, selecciona "Transformación" como tipo de proceso, agrega las transformaciones necesarias, completa todos los campos obligatorios (nombre, caché, respuesta) y, si lo deseas, agrega una descripción. Finalmente, haz clic en **"Guardar Configuración"**. El sistema mostrará un mensaje de confirmación si todo está correcto.

**¿Qué campos son obligatorios al crear un proceso de transformación?**  
Debes completar:  
- Tipo de Proceso  
- Transformaciones  
- Nombre del Proceso  
- Caché  
- Retornar respuesta  
La descripción es opcional.

**¿Puedo editar cualquier proceso de transformación?**  
Sí, puedes editar los procesos creados desde la interfaz, siempre y cuando no estén bloqueados por dependencias o reglas del sistema.

**¿Cómo ejecuto un proceso de transformación manualmente?**  
Una vez creado el proceso, puedes ejecutarlo desde el módulo de procesos seleccionando el proceso y utilizando la opción de ejecución disponible.

**¿Por qué no puedo eliminar un proceso de transformación?**  
La eliminación de procesos puede estar restringida por dependencias o reglas de seguridad del sistema. Consulta con el equipo de soporte si tienes dudas.

**¿Qué sucede si el proceso de transformación falla?**  
El sistema mostrará un mensaje de error y no permitirá guardar el proceso hasta que todos los campos estén correctamente configurados. Revisa las transformaciones y los parámetros ingresados.

**¿Cómo puedo buscar o filtrar procesos de transformación?**  
Utiliza la barra de búsqueda en el módulo de procesos para localizar procesos por nombre o tipo. Puedes ajustar la paginación para mostrar más o menos registros según tus necesidades.

**¿Qué información puedo ver en la tabla de procesos de transformación?**  
La tabla muestra:  
- Nombre  
- Tipo de Proceso  
- Transformaciones  
- Estado actual  
- Acciones disponibles (editar, ejecutar, visualizar)

**¿Qué hacer si tengo dudas o problemas con el módulo?**  
Contacta al equipo responsable de Integraciones MaxPoint a través del canal de soporte interno #mxp-integraciones (Slack).

---

## 9. Referencias y contacto

- Documentación relacionada: Integraciones, Sincronizaciones, Conexiones
- Soporte: #mxp-integraciones (Slack interno)
- Equipo responsable: Integraciones MaxPoint

---

## 10. Última Actualización

Este manual fue actualizado por última vez el **14 de