# Módulo de Inicialización de Catálogos (MXPi-Init)

## 1. Propósito y alcance

**MXPi-Init Facade-Extract-From-SP-Init-Catalog-Data-For-Franchise** es el sistema de extracción y sincronización de datos de catálogos para franquicias dentro del ecosistema MAXPOINT V2.0.

- Gestiona la extracción de datos de catálogos desde sistemas legacy
- Transforma y homologa datos entre diferentes fuentes
- Sincroniza información de franquicias, restaurantes, configuraciones y clasificaciones
- Coordina procesos ETL específicos para datos de catálogos
- Distribuye datos procesados a las bases de datos de destino

ℹ️ Para más detalles de los módulos relacionados, consulta: MXP Sync, MXP FE Build

## 2. Descripción general del sistema

**Microservicio:** mxpv2.integration.init  
**Rol principal:** gestionar procesos de extracción e inicialización de catálogos  
**Función crítica:** asegurar que los datos de catálogos se extraigan, transformen y carguen correctamente en el flujo ETL

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de extracción de catálogos | F100 | Punto de entrada principal para procesos ETL de catálogos de franquicias |

🔑 El F100 actúa como gateway unificado para datos de catálogos, validando estructura e integridad antes del procesamiento.

### 3.2. Capa de extracción

| Componente | Código | Descripción |
|------------|--------|-------------|
| Extractor SQL Server | E001 | Extrae datos desde stored procedures del sistema legacy |
| Extractor MongoDB | E002 | Extrae datos de homologación desde MongoDB |

### 3.3. Capa de transformación

| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformador de franquicias | T017 | Procesa datos básicos de franquicias |
| Transformador de restaurantes | T018 | Procesa datos de restaurantes y ubicaciones |
| Transformador de configuraciones | T019 | Procesa configuraciones de franquicias |
| Transformador de clasificaciones | T020 | Procesa clasificaciones y categorías |
| Transformador de departamentos | T021 | Procesa departamentos internos |
| Transformador de media integración | T022 | Procesa configuraciones de media integración |
| Transformador de categorías PLU | T023 | Procesa categorías con precios |
| Transformador de dispositivos PLU | T024 | Procesa configuraciones de dispositivos para PLU |
| Transformador de estados categorías | T025 | Procesa estados de categorías PLU |
| Transformador de estados PLU | T026 | Procesa estados de PLU |
| Transformador de tipos pregunta | T027 | Procesa tipos de preguntas sugeridas |
| Transformador de dispositivos menú | T028 | Procesa dispositivos para imágenes de menú |
| Transformador de estados menú | T029 | Procesa estados de menú |
| Transformador de estados pregunta | T030 | Procesa estados de preguntas sugeridas |

### 3.4. Capa de carga

| Componente | Código | Descripción |
|------------|--------|-------------|
| Cargador MongoDB | L001 | Carga datos transformados a las colecciones de destino |

## 4. Gestión de procesos

### Flujo de trabajo estándar

1. **Ingestión de datos** → El F100 recibe las solicitudes de extracción de catálogos
2. **Extracción dual** → E001 extrae desde SQL Server, E002 desde MongoDB
3. **Validación** → Se verifica estructura e integridad de datos extraídos
4. **Transformación** → Los transformadores T017-T030 procesan según el tipo de catálogo
5. **Homologación** → Se aplican reglas de homologación entre sistemas
6. **Carga** → L001 inserta/actualiza datos en las colecciones de destino

## 5. Endpoints disponibles

### Catálogos de Franquicias

- **Facade-Extract-From-SP-Init-Catalog-Data-For-Franchise**
  - Extrae datos básicos de franquicias
  - Transforma con T017
  - Destino: `InternalClients_BO_Franchise`

- **Facade-Extract-From-SP-Init-Catalog-Data-For-Franchise-Restaurant**
  - Extrae datos de restaurantes por franquicia
  - Transforma con T018
  - Destino: `InternalClients_BO_Restaurant`

- **Facade-Extract-From-SP-Init-Catalog-Data-For-Franchise-Configurations**
  - Extrae configuraciones de franquicias
  - Transforma con T019
  - Destino: `InternalClients_BO_Franchise_Configurations`

- **Facade-Extract-From-SP-Init-Catalog-Data-For-Franchise-Classification**
  - Extrae clasificaciones
  - Transforma con T020
  - Destino: `Configurations_BO_Classification`

- **Facade-Extract-From-SP-Init-Catalog-Data-For-Franchise-Category-Price**
  - Extrae categorías con precios
  - Transforma con T023
  - Destino: `Menus_BO_Category_PLU`

- **Facade-Extract-From-SP-Init-Catalog-Data-For-Franchise-Departments**
  - Extrae departamentos internos
  - Transforma con T021
  - Destino: `Configurations_BO_Departament_Internal`

- **Facade-Extract-From-SP-Init-Catalog-Data-For-Franchise-Half-Integration**
  - Extrae configuraciones de media integración
  - Transforma con T022
  - Destino: `Menus_BO_Half_Integration`

### Catálogos MXPv2

- **Facade-Extract-From-SP-Init-Catalog-Data-MXPv2-Status-Suggested-Question**
  - Extrae estados de preguntas sugeridas
  - Transforma con T030
  - Destino: `Menus_BO_Status_Suggested_Question`

- **Facade-Extract-From-SP-Init-Catalog-Data-MXPv2-Status-Menu**
  - Extrae estados de menú
  - Transforma con T029
  - Destino: `Menus_BO_Status_Menu`

- **Facade-Extract-From-SP-Init-Catalog-Data-MXPv2-Device-Images-Menu**
  - Extrae dispositivos para imágenes de menú
  - Transforma con T028
  - Destino: `Menus_BO_Device_Images_Menu`

- **Facade-Extract-From-SP-Init-Catalog-Data-MXPv2-Type-Question**
  - Extrae tipos de preguntas
  - Transforma con T027
  - Destino: `Menus_BO_Type_Suggested_Question`

- **Facade-Extract-From-SP-Init-Catalog-Data-MXPv2-Status-Plu**
  - Extrae estados de PLU
  - Transforma con T026
  - Destino: `Menus_BO_Status_PLU`

- **Facade-Extract-From-SP-Init-Catalog-Data-MXPv2-Status-Category-Price-Plu**
  - Extrae estados de categorías PLU
  - Transforma con T025
  - Destino: `Menus_BO_Status_Category_PLU`

- **Facade-Extract-From-SP-Init-Catalog-Data-MXPv2-Device-Images-Plu**
  - Extrae dispositivos para imágenes PLU
  - Transforma con T024
  - Destino: `Menus_BO_Devices_Images_PLU`

## 6. Puntos de integración

El módulo MXPi-Init conecta múltiples sistemas:

- **Entrada:** recibe datos desde SQL Server legacy y MongoDB de homologación
- **Salida:** distribuye datos a múltiples bases MongoDB especializadas
- **Rol central:** mantiene coherencia en la inicialización de catálogos

## 7. Patrones de sincronización

- **Extracción dual** → Combina datos de SQL Server y MongoDB
- **Transformación especializada** → Cada tipo de catálogo tiene su transformador
- **Carga dirigida** → Los datos se cargan en colecciones específicas según su tipo
- **Homologación automática** → Mapeo de IDs entre sistemas legacy y v2

## 8. Escenarios de uso

### ✅ Caso exitoso
Solicitud válida → extracción E001/E002 → transformación T0XX → carga L001 → datos sincronizados

### ⚠️ Posibles fallos

**Error en extracción SQL Server:**
- Datos rechazados por stored procedure inválido
- Acción recomendada: verificar parámetros de país y cdn_id

**Error en transformación:**
- Datos no pasan a carga por falta de homologación
- Se activa rollback y alerta al sistema de monitoreo

**Error en carga MongoDB:**
- Colección de destino no disponible
- El sistema reintenta según política de reintentos

## 9. Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si falla la extracción en E001?**
A: El proceso se detiene, se registra en logs y se debe verificar la conexión al sistema legacy y los parámetros.

**Q: ¿Cómo sé que la transformación se procesó correctamente?**
A: Revisa el log del transformador correspondiente (T017-T030). Si hay éxito, se marca como transformation_ok.

**Q: ¿El módulo almacena datos permanentemente?**
A: No, solo orquesta flujos ETL. La persistencia final está en las colecciones MongoDB de destino.

**Q: ¿Qué hacer si la carga falla?**
A: Se activa la política de reintentos, pero si persiste, revisar la colección de destino y los logs de L001.

## Referencias

**Fuentes internas:**
- mxp-init/process/readme.md
- mxp-sync/process/readme.md

**Microservicio:** mxpv2.integration.init

## Contacto

**Equipo responsable:** Integraciones MAXPOINT  
**Canal de soporte:** #mxp-integraciones (Slack interno)  
**Escalamiento:** Arquitectura de datos → Liderazgo técnico