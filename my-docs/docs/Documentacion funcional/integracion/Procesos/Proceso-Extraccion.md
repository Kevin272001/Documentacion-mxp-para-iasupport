---
title: Proceso de Extracción
description: Manual técnico y de usuario para la creación de procesos de extracción en MaxPoint.
updated: 2025-03-14
tags: [proceso, extracción, gestión, usuario, MaxPoint, IA-support]
sidebar_position: 3.1
---

# Módulo de Procesos de Extracción (MXP-ExtractProcess)

## 1. Propósito y alcance

El módulo de **Procesos de Extracción** en MaxPoint permite crear y configurar procesos para la obtención de datos desde sistemas externos o internos.
- Facilita la administración centralizada de procesos de extracción.
- Permite configuraciones recomendadas y personalizadas según la necesidad.

---

## 2. Descripción técnica

- **Microservicio:** mxpv2.integration.processes
- **Rol principal:** Administrar la creación y configuración de procesos de extracción.
- **Función crítica:** Definir la estructura, método y propiedades para la extracción de datos.

---

## 3. Componentes y arquitectura

| Componente                | Código | Descripción                                                                 |
|---------------------------|--------|-----------------------------------------------------------------------------|
| Proceso de extracción     | P101   | Configuración de entidades, propiedades y métodos de consumo.               |
| Control de validación     | P102   | Validación de campos obligatorios y estructura de scripts.                  |
| UI de Procesos            | P103   | Interfaz para crear, editar y visualizar procesos de extracción.            |

---

## 4. Gestión de procesos

### 4.1. Creación de un Proceso de Extracción (Configuración Recomendada)

1. Ingresar al Configurador de Procesos (`Procesos > Crear Nuevo Proceso`)
2. Seleccionar **Tipo de Proceso:** Extracción
3. Ingresar **Nombre del Proceso** (ejemplo: "Extracción de datos de ventas")
4. Seleccionar **Configuración Recomendada:** Sí
5. Elegir **Método de Consumo:** POST, UPDATE, DELETEGE, GET (ejemplo: GET para obtener datos)
6. Seleccionar una **Conexión Existente**
7. Configurar el **Caché:** Sí/No
8. Definir si se **Retorna una Respuesta:** Sí/No
9. Agregar una **Descripción** (opcional)
10. Revisar la **Entidad Elegida**
11. Guardar el Proceso

:::tip Nota
- Todos los campos son obligatorios excepto la descripción.  
- Revisa la entidad antes de guardar para evitar errores.  
- Si algún campo no está configurado correctamente, el sistema podría mostrar un mensaje de error.
:::

---

### 4.2. Creación de un Proceso de Extracción (Configuración Personalizada)

1. Ingresar al Configurador de Procesos
2. Seleccionar **Tipo de Proceso:** Extracción
3. Ingresar **Nombre del Proceso**
4. Seleccionar **Configuración Recomendada:** No
5. Ingresar el **Script de Extracción** (código o consulta personalizada)
6. Agregar una **Descripción** (opcional)
7. Guardar el Proceso

:::tip Nota
- Todos los campos son obligatorios, excepto la descripción.
- El script debe estar bien estructurado para evitar errores en la extracción.
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

- Cada proceso de extracción tiene un identificador único.
- El script debe estar correctamente estructurado en configuración personalizada.
- La entidad debe ser revisada antes de guardar.

---

## 7. Escenarios de uso

✅ **Caso exitoso:**  
Creación de proceso de extracción con configuración recomendada o personalizada, validación y confirmación.

⚠️ **Posibles fallos:**  
- Error por campos incompletos o mal configurados.
- Script mal estructurado en configuración personalizada.
- No revisar la entidad puede causar errores.

---

## 8. Preguntas frecuentes (FAQ)

**¿Cómo creo un proceso de extracción en MaxPoint?**  
Para crear un proceso de extracción, ingresa al módulo desde `Configurador > Procesos`. Haz clic en **"Crear Nuevo Proceso"**, selecciona "Extracción" como tipo de proceso, completa todos los campos obligatorios (nombre, método de consumo, conexión, caché, respuesta, entidad) y, si lo deseas, agrega una descripción. Finalmente, haz clic en **"Guardar Configuración"**. El sistema mostrará un mensaje de confirmación si todo está correcto.

**¿Qué campos son obligatorios al crear un proceso de extracción?**  
Debes completar:  
- Nombre del Proceso  
- Tipo de Proceso  
- Método de Consumo  
- Conexión  
- Caché  
- Retornar respuesta  
- Entidad  
Si usas configuración personalizada, también debes ingresar el script de extracción.

**¿Puedo editar cualquier proceso de extracción?**  
Sí, puedes editar los procesos creados desde la interfaz, siempre y cuando no estén bloqueados por dependencias o reglas del sistema.

**¿Cómo ejecuto un proceso de extracción manualmente?**  
Una vez creado el proceso, puedes ejecutarlo desde el módulo de procesos seleccionando el proceso y utilizando la opción de ejecución disponible.

**¿Por qué no puedo eliminar un proceso de extracción?**  
La eliminación de procesos puede estar restringida por dependencias o reglas de seguridad del sistema. Consulta con el equipo de soporte si tienes dudas.

**¿Qué sucede si el proceso de extracción falla?**  
El sistema mostrará un mensaje de error y no permitirá guardar el proceso hasta que todos los campos estén correctamente configurados. Revisa la entidad, el script y los parámetros ingresados.

**¿Cómo puedo buscar o filtrar procesos de extracción?**  
Utiliza la barra de búsqueda en el módulo de procesos para localizar procesos por nombre o tipo. Puedes ajustar la paginación para mostrar más o menos registros según tus necesidades.

**¿Qué información puedo ver en la tabla de procesos de extracción?**  
La tabla muestra:  
- Nombre  
- Tipo de Proceso  
- Método de Consumo  
- Conexión  
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

Este manual fue actualizado por última vez el **14 de marzo de