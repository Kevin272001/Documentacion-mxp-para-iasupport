# Módulo de Inicialización (MXPi-init) - Facade-Extract-From-SP-Init-Transactional

## Propósito y alcance

MXPi-init es el sistema de inicialización y extracción de datos dentro del ecosistema MAXPOINT V2.0. Gestiona el proceso de extracción de datos desde sistemas legacy (SQL Server) y MongoDB, aplica transformaciones necesarias y carga los datos procesados en las colecciones de destino.

🔍 **Función principal:** Extraer información de menús transaccionales y categorías desde sistemas legacy y transformarlos al formato requerido por el nuevo ecosistema.

## Descripción general del sistema

* **Microservicio:** mxpv2.integration.init
* **Rol principal:** Coordinar procesos ETL (Extract, Transform, Load) para datos de categorías de menú transaccionales.
* **Función crítica:** Garantizar la correcta migración y transformación de datos desde sistemas legacy hacia el nuevo esquema de MAXPOINT V2.0.

## Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código | Descripción                                                                             |
| ------------------------- | ------ | --------------------------------------------------------------------------------------- |
| Interfaz de recepción ETL | F100   | Punto de entrada principal para solicitudes de procesamiento ETL de categorías de menú. |

### 3.2. Capa de extracción

| Componente                  | Código           | Descripción                                                                          |
| --------------------------- | ---------------- | ------------------------------------------------------------------------------------ |
| Extracción desde SQL Server | E001, E002       | Extrae datos de categorías de menú e imágenes desde stored procedures de SQL Server. |
| Extracción desde MongoDB    | E003, E006, E007 | Obtiene UUIDs, configuraciones de colores e íconos desde colecciones de MongoDB.     |

### 3.3. Capa de transformación

| Componente                   | Código | Descripción                                           |
| ---------------------------- | ------ | ----------------------------------------------------- |
| Transformación de categorías | T032   | Procesa y transforma datos de categorías de menú.     |
| Transformación de imágenes   | T031   | Procesa y transforma datos de imágenes de categorías. |

### 3.4. Capa de carga

| Componente      | Código | Descripción                                                            |
| --------------- | ------ | ---------------------------------------------------------------------- |
| Carga a MongoDB | L001   | Inserta los datos transformados en las colecciones destino de MongoDB. |

## Gestión de procesos

**Flujo de trabajo estándar:**

1. Recepción de solicitud → El endpoint F100 recibe la solicitud de procesamiento
2. Extracción → Se ejecutan los procesos E001-E007 para obtener datos desde fuentes diversas
3. Validación → Se verifica la estructura e integridad de los datos extraídos
4. Transformación → Los procesos T031 y T032 transforman los datos al formato destino
5. Carga → El proceso L001 inserta los datos en las colecciones MongoDB destino

## Puntos de integración

* **Entrada:**

  * SQL Server Legacy mediante stored procedures
  * MongoDB para UUIDs y configuraciones
* **Salida:**

  * Colecciones `Menus_BO_Category_Menu` y `Menus_BO_Category_Image` en MongoDB
* **Dependencias:**

  * Servicios de autenticación para tokens Bearer
  * Servicios de UUID para generación de identificadores

## Endpoint principal

**POST /api/v1/facade**

* **Descripción:** Procesamiento de categorías de menú transaccionales
* **Autenticación:** Bearer Token

### Headers

```
Authorization: Bearer {{token}}
Content-Type: application/json
```

### Ejemplo de JSON request para categorías de menú

```json
{
    "code": "F100",
    "withResponse": false,
    "withCache": false,
    "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "processes": {
        "EXTRACTS": [
            {
                "code": "E001",
                "withResponse": true,
                "withCache": false,
                "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "configuration": {
                    "connection": {
                        "server": "{{server_legacy}}",
                        "port": "",
                        "user": "{{user_legacy}}",
                        "password": "{{pass_legacy}}",
                        "repository": "{{repository_legacy}}"
```
