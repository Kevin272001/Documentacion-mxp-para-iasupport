# Documentación Módulo MXPI-INIT Printer Settings DEV

## Propósito y alcance
MXPI-INIT Printer Settings DEV es el módulo de inicialización y configuración de dispositivos de impresión dentro del ecosistema MAXPOINT V2.0. Gestiona la sincronización de configuraciones de impresión, documentos, canales y parámetros de dispositivos para restaurantes. Garantiza la coherencia en la configuración de impresión en toda la plataforma.

## Descripción general del sistema
**Microservicio**: mxpv2.integration.init.printer  
**Rol principal**: gestionar procesos de inicialización y configuración de impresión  
**Función crítica**: asegurar que las configuraciones de impresión estén correctamente sincronizadas entre sistemas legacy y la nueva plataforma

## Arquitectura y componentes

### Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de configuración de impresión | F100 | Punto de entrada principal para procesos ETL de configuración de impresión |

### Capa de extracción (Extracts)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Extracción SQL Server | E001 | Extrae datos de stored procedures de SQL Server para documentos de impresión |
| Extracción MongoDB UUIDs | E002, E003, E004 | Obtiene UUIDs homologados para entidades del sistema |

### Capa de transformación (Transforms)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación documentos impresión | T124 | Procesa y transforma documentos de impresión |
| Transformación canales impresión | T125 | Procesa y transforma canales de impresión |
| Transformación canales con documentos | T126 | Procesa canales con sus documentos asociados |
| Transformación configuraciones impresión | T127 | Procesa configuraciones completas de impresión |

### Capa de carga (Loads)
| Componente | Código | Descripción |
|------------|--------|-------------|
| Carga MongoDB | L001 | Inserta datos transformados en colecciones de MongoDB |

## Endpoints documentados

### 1. Sincronización de Documentos de Impresión
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Sincronizar documentos de impresión desde sistemas legacy a MongoDB

**Flujo**:
- Extrae documentos de SQL Server (E001)
- Obtiene UUIDs para país, cadena y entidades (E003, E004)
- Transforma datos con T124
- Carga en colección `Settings_BO_Printing_Documents`

**Estructura de datos**:
```json
{
  "homologated_id": "uuid",
  "country_id": "uuid",
  "cdn_id": "uuid",
  "document_name": "string",
  "document_key": "string",
  "document_template": "string"
}
```

### 2. Sincronización de Canales de Impresión
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Sincronizar canales de impresión desde sistemas legacy a MongoDB

**Flujo**:
- Extrae canales de SQL Server (E001)
- Obtiene UUIDs para canal, país y cadena (E002, E003, E004)
- Transforma datos con T125
- Carga en colección `Settings_BO_Printing_Channels`

**Estructura de datos**:
```json
{
  "homologated_id": "uuid",
  "country_id": "uuid",
  "cdn_id": "uuid",
  "channel_name": "string",
  "documents_to_print": "array"
}
```

### 3. Sincronización de Canal VOUCHER con Documentos
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Asociar documentos al canal VOUCHER

**Flujo**:
- Extrae canal VOUCHER de MongoDB (E001)
- Extrae documentos específicos para vouchers (E002)
- Obtiene UUIDs para cadena y país (E004)
- Transforma con T126 (incluyendo datos)
- Actualiza canal en `Settings_BO_Printing_Channels`

**Documentos incluidos**:
- ARQUEO, CORTE_XX, DESASIGNADO_CAJERO, DESASIGNADO_CAJERO_DEN
- FIN_DE_DIA, RETIRO_FONDO_ASIGNADO, RETIRO_EFECTIVO, RETIROS_FORMASPAGO
- VOUCHER_CANCELADO_NO_APROBADO, VOUCHER_APROBADO, VOUCHER_ANULACION, VOUCHER_NO_APROBADO

### 4. Sincronización de Canal FACTURA con Documentos
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Asociar documentos al canal FACTURA

**Flujo**:
- Extrae canal FACTURA de MongoDB (E001)
- Extrae documentos específicos para facturas (E002)
- Obtiene UUIDs para cadena y país (E004)
- Transforma con T126 (incluyendo datos)
- Actualiza canal en `Settings_BO_Printing_Channels`

**Documentos incluidos**:
- FACTURA, NOTA_CREDITO

### 5. Sincronización de Configuraciones de Impresión Completas
**Endpoint**: `POST {{server_init}}/api/v1/facade`

**Propósito**: Sincronizar configuraciones completas de impresión para cajas registradoras

**Flujo**:
- Extrae configuraciones de SQL Server (E001)
- Obtiene UUIDs para caja registradora, restaurante, cadena y canal (E002)
- Extrae canales de impresión existentes (E004)
- Transforma con T127
- Carga en colección `Settings_BO_Printing_Settings`

**Estructura de datos**:
```json
{
  "country_id": "uuid",
  "cdn_id": "uuid",
  "restaurant_id": "uuid",
  "cash_registers": [
    {
      "cash_register_id": "uuid",
      "cash_register_name": "string",
      "printing_channel": [
        {
          "channel_id": "uuid",
          "channel_name": "string",
          "documents_to_print": "array",
          "printers": "array"
        }
      ]
    }
  ]
}
```

## Gestión de procesos

### Flujo de trabajo estándar
1. **Iniciación** → El F100 recibe la solicitud de sincronización
2. **Extracción** → Múltiples extractores (E001-E004) obtienen datos de diversas fuentes
3. **Homologación** → Los UUIDs se resuelven para mantener consistencia entre sistemas
4. **Transformación** → Los transformadores (T124-T127) procesan y estructuran los datos
5. **Carga** → El cargador L001 inserta los datos en MongoDB
6. **Confirmación** → Se retorna estado de la operación

## Puntos de integración
El módulo conecta con múltiples sistemas:
- **SQL Server**: Datos legacy de configuración de impresión
- **MongoDB UUIDs**: Homologación de identificadores
- **MongoDB Settings**: Almacenamiento de configuraciones finales

## Patrones de sincronización
- **Extracción múltiple**: Datos obtenidos de varias fuentes simultáneamente
- **Homologación centralizada**: UUIDs gestionados consistentemente
- **Transformación especializada**: Diferentes transformadores para distintos tipos de datos
- **Carga estructurada**: Datos cargados en schemas específicos

## Escenarios de uso

### ✅ Caso exitoso
Solicitud F100 válida → extracción exitosa → homologación completa → transformación correcta → carga exitosa en MongoDB

### ⚠️ Posibles fallos
- **Error en extracción**: Datos no disponibles en sistemas source
- **Error en homologación**: UUIDs no encontrados para entidades
- **Error en transformación**: Datos incompatibles con transformadores
- **Error en carga**: MongoDB no disponible o schema incompatible

## Preguntas frecuentes (para IA Support)

**Q: ¿Qué hacer si falla la homologación de UUIDs?**  
A: Verificar que las entidades existan en la colección `uuids` con los filtros correctos.

**Q: ¿Cómo verificar que una sincronización fue exitosa?**  
A: Revisar los logs de transformación (T124-T127) y confirmar la carga en MongoDB.

**Q: ¿Los procesos son idempotentes?**  
A: Sí, pueden ejecutarse múltiples veces sin afectar la consistencia de datos.

**Q: ¿Qué pasa si MongoDB no está disponible durante la carga?**  
A: El sistema reintentará según la política de reintentos configurada.

## Referencias
- **Fuentes internas**: mxpi-init/printer-settings/readme.md
- **Microservicio**: mxpv2.integration.init.printer
- **Colecciones MongoDB**: DEV_KFC_MXPi_Settings (Settings_BO_Printing_*)

## Contacto
**Equipo responsable**: Impresión y Dispositivos MAXPOINT  
**Canal de soporte**: #mxp-impresion (Slack interno)  
**Escalamiento**: Arquitectura de Dispositivos → Liderazgo técnico