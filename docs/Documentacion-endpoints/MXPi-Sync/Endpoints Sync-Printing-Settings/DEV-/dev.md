# Módulo de Sincronización de Impresión (MXPI-Sync – Printing Settings)

## 1. Propósito y alcance
El módulo **MXPI-Sync Printing Settings** es responsable de la **sincronización de configuraciones de impresión** dentro del ecosistema **MAXPOINT V2.0**.

- Gestiona la extracción, transformación y carga de parámetros relacionados con la configuración de impresoras.  
- Garantiza que los ajustes de impresión estén disponibles en los puntos de venta (POS) de forma unificada.  
- Coordina procesos ETL para mantener consistencia en las configuraciones de impresión entre entornos de desarrollo y productivos.  

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- MXP Init  
- MXP FE Build  
- MXP Printer  

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.sync`  
- **Rol principal:** sincronizar configuraciones de impresión desde repositorios Mongo hacia sistemas locales.  
- **Función crítica:** asegurar que la información de impresión (p. ej. cajas registradoras y parámetros de impresión) llegue correctamente a los restaurantes definidos.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de sincronización de impresión | F008 | Punto de entrada principal para solicitudes de sincronización de configuraciones de impresión. |

🔑 El componente `F008` valida integridad de la petición y orquesta los procesos ETL.  

### 3.2. Capa de transformación
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación de datos de impresión | T080 | Normaliza las configuraciones de impresión y garantiza compatibilidad entre repositorios. |

---

## 4. Gestión de procesos

**Flujo de trabajo estándar (Sync-Printing-Settings – DEV):**  
1. **Ingestión de datos** → `F008` recibe parámetros de sincronización.  
2. **Extracción** → Se conecta al repositorio Mongo (`DEV_KFC_MXPi_Settings`) para obtener `Settings_BO_Printing_Settings`.  
3. **Validación** → Filtra registros por `country_id`, `cdn_id` y `restaurant_id`.  
4. **Transformación** → Con `T080`, procesa los datos de impresión, asegurando su estructura.  
5. **Carga** → Envía configuraciones transformadas al repositorio local (`MXPi_Settings`).  
6. **Distribución** → Pone disponibles los parámetros en los POS del restaurante destino.  

---

## 5. Puntos de integración
- **Entrada:** configuraciones desde `DEV_KFC_MXPi_Settings` (Mongo).  
- **Salida:** repositorio local `MXPi_Settings`.  
- **Rol central:** mantener actualizadas las configuraciones de impresión para los restaurantes.  

---

## 6. Patrones de sincronización
- Recepción vía fachada `F008`.  
- Procesamiento de configuraciones con `T080`.  
- Distribución hacia repositorios locales Mongo.  
- Validación de consistencia mediante filtros de país, cadena y restaurante.  

---

## 7. Escenarios de uso

### ✅ Caso exitoso
1. Se envía request a `POST {{server_sync}}/api/v1/facade` con código `F008`.  
2. Se extraen configuraciones de impresión desde Mongo (DEV).  
3. Se transforman con `T080`.  
4. Se cargan en `MXPi_Settings`.  
5. Los POS reciben la configuración actualizada.  

### ⚠️ Posibles fallos
- **Error en validación:** filtros inválidos → datos rechazados.  
- **Error en transformación (`T080`):** inconsistencia en estructura → rollback automático.  
- **Error en distribución:** fallo de conexión con repositorio destino → reintentos activados.  

---

## 8. Endpoint documentado

### Sync-Printing-Settings (DEV)

**Método:** `POST`  
**URL:** `{{server_sync}}/api/v1/facade`  
**Auth:** `Bearer Token {{token}}`  

#### 📥 Body ejemplo
```json
{
  "code": "F008",
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "chunkLoad": 100,
  "processes": {
    "EXTRACTS": [
      {
        "code": "E001",
        "configuration": {
          "connection": {
            "server": "{{server_mongo}}",
            "repository": "DEV_KFC_MXPi_Settings",
            "adapter": "Mongo"
          },
          "entities": [
            {
              "name": "Settings_BO_Printing_Settings",
              "filters": [
                { "name": "country_id", "operator": "equals", "datatype": "uuid", "value": "d6cac83d-6ce5-f62e-2628-1e2e9ae7b51f" },
                { "name": "cdn_id", "operator": "equals", "datatype": "uuid", "value": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af" },
                { "name": "restaurant_id", "operator": "equals", "datatype": "uuid", "value": "24260579-faf8-763f-aac7-5e16faff96d1" }
              ]
            }
          ]
        }
      }
    ],
    "TRANSFORMS": [
      { "code": "T080", "country": "ECU", "withCache": true }
    ],
    "LOADS": [
      {
        "code": "L001",
        "configuration": {
          "connection": {
            "server": "192.168.101.13",
            "port": "27020",
            "user": "admin",
            "password": "admin123",
            "repository": "MXPi_Settings",
            "adapter": "MongoLocal"
          },
          "entities": [
            { "name": "Settings_BO_Printing_Settings", "toName": "Settings_BO_Printing_Settings" }
          ]
        }
      }
    ],
    "PUBLISHERS": []
  }
}
```

#### 📤 Example Response
```json
{
  "message": "Error processing request: 'name'"
}
```

---

## 9. Referencias
- `mxp-sync/process/readme.md`  
- `mxp-printer/config/printing/readme.md`  

---

## 10. Contacto
- **Equipo responsable:** Integraciones MAXPOINT  
- **Canal de soporte:** #mxp-integraciones (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico  
