---
title: Proceso de Carga
description: Manual técnico y de usuario para la creación de procesos de carga en MaxPoint.
updated: 2025-03-14
tags: [proceso, carga, gestión, usuario, MaxPoint, IA-support]
sidebar_position: 3
---

# Módulo de Procesos de Carga (MXP-LoadProcess)

## 1. Propósito y alcance

El módulo de **Procesos de Carga** en MaxPoint permite crear y configurar procesos para la transferencia de datos entre sistemas.
- Facilita la administración centralizada de procesos de carga.
- Permite configuraciones recomendadas y personalizadas según la necesidad.

---

## 2. Descripción técnica

- **Microservicio:** mxpv2.integration.processes
- **Rol principal:** Administrar la creación y configuración de procesos de carga.
- **Función crítica:** Definir la estructura, método y propiedades para la carga de datos.

---

## 3. Componentes y arquitectura

| Componente              | Código | Descripción                                                                 |
|-------------------------|--------|-----------------------------------------------------------------------------|
| Proceso de carga lógica | P001   | Configuración de entidades, propiedades y métodos de consumo.               |
| Control de validación   | P002   | Validación de campos obligatorios y estructura de scripts.                  |
| UI de Procesos          | P003   | Interfaz para crear, editar y visualizar procesos de carga.                 |

---

## 4. Gestión de procesos

### 4.1. Creación de un Proceso de Carga (Configuración Recomendada)

1. Ingresar al Configurador de Procesos (`Procesos > Crear Nuevo Proceso`)
2. Seleccionar **Tipo de Proceso:** Carga
3. Ingresar **Nombre del Proceso** (ejemplo: "Carga de datos de clientes")
4. Seleccionar **Configuración Recomendada:** Sí/No
5. Elegir **Método de Consumo:** POST, UPDATE, DELETEGE, GET
6. Seleccionar una **Conexión Existente**
7. Configurar el **Caché:** Sí/No
8. Definir si se **Retorna una Respuesta:** Sí/No
9. Revisar la **Entidad Seleccionada**
10. Completar la **Configuración de la Propiedad**:
    - Llenar el campo "ToName"
    - Agregar una nueva Propiedad
    - Completar: fromName, fromType, toName, toType, hasUuids, defaultValue (opcional), isPrimaryKey (obligatorio)
11. Guardar el Proceso

:::tip Nota
Todos los campos son obligatorios excepto `defaultValue`.  
El campo "isPrimaryKey" debe estar marcado.  
Revisar la entidad antes de guardar para evitar errores.
:::

---

### 4.2. Creación de un Proceso de Carga (Configuración No Recomendada)

1. Ingresar al Configurador de Procesos
2. Seleccionar **Tipo de Proceso:** Carga
3. Ingresar **Nombre del Proceso**
4. Seleccionar **Configuración Recomendada:** No
5. Ingresar el **Script de Carga** (código o consulta personalizada)
6. Agregar una **Descripción** (opcional)
7. Guardar el Proceso

:::tip Nota
Todos los campos son obligatorios excepto la descripción.  
El script debe estar bien estructurado para evitar errores.
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

- Cada proceso de carga tiene un identificador único.
- El campo "isPrimaryKey" debe estar marcado para crear el proceso.
- El script debe estar correctamente estructurado en configuración no recomendada.

---

## 7. Escenarios de uso

✅ **Caso exitoso:**  
Creación de proceso de carga con configuración recomendada o personalizada, validación y confirmación.

 **Posibles fallos:**  
- Error por campos incompletos o mal configurados.
- Script mal estructurado en configuración no recomendada.
- No marcar "isPrimaryKey" impide la creación.

---

## 8. Preguntas frecuentes (FAQ)

**¿Cómo creo un proceso de carga con configuración recomendada?**  
Accede al módulo de procesos, selecciona "Crear Nuevo Proceso", elige "Carga", ingresa el nombre, selecciona "Sí" en configuración recomendada y completa todos los campos obligatorios. Marca "isPrimaryKey" y guarda la configuración.

**¿Qué pasa si no marco "isPrimaryKey"?**  
El sistema no permitirá la creación del proceso. Este campo es obligatorio para identificar la clave primaria en la entidad.

**¿Cómo configuro el método de consumo?**  
Selecciona entre POST, UPDATE, DELETEGE o GET según el tipo de operación que deseas realizar en el proceso de carga.

**¿Qué diferencia hay entre configuración recomendada y no recomendada?**  
La configuración recomendada guía al usuario en la selección de campos y propiedades, mientras que la no recomendada permite ingresar un script personalizado para la carga de datos.

**¿Qué hago si recibo un mensaje de error al guardar el proceso?**  
Revisa que todos los campos obligatorios estén completos y correctamente configurados. Si usas un script, verifica su estructura.

**¿Puedo agregar una descripción al proceso?**  
Sí, la descripción es opcional y puede ayudar a identificar el propósito del proceso.

**¿Cómo sé que el proceso se creó correctamente?**  
El sistema mostrará el mensaje "Creado correctamente" tras guardar la configuración.

**¿Qué hacer si tengo dudas o problemas con el módulo?**  
Contacta al equipo responsable de Integraciones MaxPoint a través del canal de soporte interno #mxp-integraciones (Slack).

---

## 9. Referencias y contacto

- Documentación relacionada: Integraciones, Sincronizaciones, Conexiones
- Soporte: #mxp-integraciones (Slack interno)
- Equipo responsable: Integraciones MaxPoint

---

## 10. Última Actualización

Este manual fue actualizado por última vez el **14 de marzo de 2025**.
