# Documentación Módulo MXPI-INIT Printer Settings QA

## Propósito y alcance
MXPI-INIT Printer Settings QA es el módulo de configuración de impresión para el entorno de calidad (QA) dentro del ecosistema MAXPOINT V2.0. Gestiona la sincronización de configuraciones de impresión, documentos, canales y parámetros de dispositivos específicos para el ambiente de control de calidad. Garantiza la coherencia en la configuración de impresión en el entorno QA antes de pasar a producción.

## Descripción general del sistema
**Microservicio**: mxpv2.integration.init.printer.qa  
**Rol principal**: gestionar procesos de inicialización y configuración de impresión en QA  
**Función crítica**: asegurar que las configuraciones de impresión estén correctamente sincronizadas entre sistemas legacy y la plataforma V2.0 en ambiente de calidad

## Arquitectura y componentes

### Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de configuración de impresión QA | F100 | Punto de entrada principal para procesos ETL de configuración de impresión en QA |

### Capa de extracción (Extracts)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Extracción SQL Server | E001 | Extrae datos de stored procedures de SQL Server para documentos de impresión |
| Extracción MongoDB UUIDs | E002, E003, E004 | Obtiene UUIDs homologados para entidades del sistema desde QA |
| Extracción MongoDB Settings | E001, E002 | Lee configuraciones existentes en MongoDB QA |

### Capa de transformación (Transforms)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación documentos impresión | T124 | Procesa y transforma documentos de impresión para QA |
| Transformación canales impresión | T125 | Procesa y transforma canales de impresión para QA |
| Transformación canales con documentos | T126 | Procesa canales con sus documentos asociados para QA |
| Transformación configuraciones impresión | T127 | Procesa configuraciones completas de impresión para QA |

### Capa de carga (Loads)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Carga MongoDB QA | L001 | Inserta datos transformados en colecciones de MongoDB QA |

## Endpoints documentados

### 1. Sincronización de Documentos de Impresión (QA)
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Sincronizar documentos de impresión desde sistemas legacy a MongoDB QA

**Flujo**:
- Extrae documentos de SQL Server (E001)
- Obtiene UUIDs para país, cadena y entidades (E003, E004)
- Transforma datos con T124
- Carga en colección `Settings_BO_Printing_Documents` en BD QA

**Base de datos destino**: `QA_KFC_MXPi_Settings`

### 2. Sincronización de Canales de Impresión (QA)
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Sincronizar canales de impresión desde sistemas legacy a MongoDB QA

**Flujo**:
- Extrae canales de SQL Server (E001)
- Obtiene UUIDs para canal, país y cadena (E002, E003, E004)
- Transforma datos con T125
- Carga en colección `Settings_BO_Printing_Channels` en BD QA

**Base de datos destino**: `QA_KFC_MXPi_Settings`

### 3. Sincronización de Canal VOUCHER con Documentos (QA)
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Asociar documentos al canal VOUCHER en ambiente QA

**Flujo**:
- Extrae canal VOUCHER de MongoDB QA (E001)
- Extrae documentos específicos para vouchers (E002)
- Obtiene UUIDs para cadena y país (E004)
- Transforma con T126
- Actualiza canal en `Settings_BO_Printing_Channels` en QA

**Documentos incluidos**:
- ARQUEO, CORTE_XX, DESASIGNADO_CAJERO, DESASIGNADO_CAJERO_DEN
- FIN_DE_DIA, RETIRO_FONDO_ASIGNADO, RETIRO_EFECTIVO, RETIROS_FORMASPAGO
- VOUCHER_CANCELADO_NO_APROBADO, VOUCHER_APROBADO, VOUCHER_ANULACION, VOUCHER_NO_APROBADO

### 4. Sincronización de Canal FACTURA con Documentos (QA)
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Asociar documentos al canal FACTURA en ambiente QA

**Flujo**:
- Extrae canal FACTURA de MongoDB QA (E001)
- Extrae documentos específicos para facturas (E002)
- Obtiene UUIDs para cadena y país (E004)
- Transforma con T126
- Actualiza canal en `Settings_BO_Printing_Channels` en QA

**Documentos incluidos**:
- FACTURA, NOTA_CREDITO

### 5. Sincronización de Canal LINEA con Documentos (QA)
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Asociar documentos al canal LINEA en ambiente QA

**Flujo**:
- Extrae canal LINEA de MongoDB QA (E001)
- Extrae documentos específicos para línea (E002)
- Obtiene UUIDs para cadena y país (E004)
- Transforma con T126
- Actualiza canal en `Settings_BO_Printing_Channels` en QA

**Documentos incluidos**:
- ORDEN_PEDIDO

### 6. Sincronización de Configuraciones de Impresión Completas (QA)
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Sincronizar configuraciones completas de impresión para cajas registradoras en QA

**Flujo**:
- Extrae configuraciones de SQL Server (E001)
- Obtiene UUIDs para caja registradora, restaurante, cadena y canal (E002)
- Extrae canales de impresión existentes (E004)
- Transforma con T127
- Carga en colección `Settings_BO_Printing_Settings` en QA

## Gestión de procesos

### Flujo de trabajo estándar
1. **Iniciación** → El F100 recibe la solicitud de sincronización
2. **Extracción** → Múltiples extractores (E001-E004) obtienen datos de diversas fuentes
3. **Homologación** → Los UUIDs se resuelven para mantener consistencia entre sistemas
4. **Transformación** → Los transformadores (T124-T127) procesan y estructuran los datos
5. **Carga** → El cargador L001 inserta los datos en MongoDB QA
6. **Confirmación** → Se retorna estado de la operación

### Configuración de conexiones QA
- **SQL Server**: Conexión legacy con autenticación por usuario/contraseña
- **MongoDB QA**: Conexión al entorno de calidad con credenciales específicas de QA
- **Repository**: `QA_KFC_MXPi_Settings` (base de datos de QA)

## Puntos de integración
El módulo conecta con múltiples sistemas en QA:
- **SQL Server**: Datos legacy de configuración de impresión
- **MongoDB UUIDs QA**: Homologación de identificadores en ambiente de calidad
- **MongoDB Settings QA**: Almacenamiento de configuraciones finales en QA

## Patrones de sincronización
- **Extracción múltiple**: Datos obtenidos de varias fuentes simultáneamente
- **Homologación centralizada**: UUIDs gestionados consistentemente
- **Transformación especializada**: Diferentes transformadores para distintos tipos de datos
- **Carga estructurada**: Datos cargados en schemas específicos en ambiente QA

## Escenarios de uso

### ✅ Caso exitoso
Solicitud F100 válida → extracción exitosa → homologación completa → transformación correcta → carga exitosa en MongoDB QA

### ⚠️ Posibles fallos
- **Error en extracción**: Datos no disponibles en sistemas source
- **Error en homologación**: UUIDs no encontrados para entidades en QA
- **Error en transformación**: Datos incompatibles con transformadores
- **Error en carga**: MongoDB QA no disponible o schema incompatible

## Preguntas frecuentes (para IA Support)

**Q: ¿Qué diferencia hay entre el entorno QA y DEV?**  
A: QA utiliza bases de datos específicas de calidad (`QA_KFC_MXPi_Settings`) y credenciales de conexión diferentes para asegurar el aislamiento del ambiente de testing.

**Q: ¿Cómo verificar que una sincronización en QA fue exitosa?**  
A: Revisar los logs de transformación (T124-T127) y confirmar la carga en MongoDB QA.

**Q: ¿Los procesos son idempotentes en QA?**  
A: Sí, pueden ejecutarse múltiples veces sin afectar la consistencia de datos en el ambiente de calidad.

**Q: ¿Qué hacer si MongoDB QA no está disponible durante la carga?**  
A: El sistema reintentará según la política de reintentos configurada para el ambiente QA.

## Referencias
- **Fuentes internas**: mxpi-init/printer-settings-qa/readme.md
- **Microservicio**: mxpv2.integration.init.printer.qa
- **Colecciones MongoDB QA**: QA_KFC_MXPi_Settings (Settings_BO_Printing_*)

## Contacto
**Equipo responsable**: Impresión y QA MAXPOINT  
**Canal de soporte**: #mxp-impresion-qa (Slack interno)  
**Escalamiento**: Arquitectura de Dispositivos → Liderazgo técnico