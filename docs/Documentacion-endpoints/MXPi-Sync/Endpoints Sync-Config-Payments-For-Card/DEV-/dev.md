# Módulo de SYNC (MXPI-Sync) DEV

## 1. Propósito y alcance
El módulo **MXPI-Sync** permite orquestar procesos de sincronización complejos dentro del ecosistema **MAXPOINT V2.0**, gestionando integraciones ETL (Extract, Transform, Load) con múltiples fuentes y repositorios.  

- Gestiona configuraciones de pagos con tarjeta.  
- Coordina procesos de extracción desde SQL Server y MongoDB.  
- Aplica transformaciones intermedias.  
- Carga resultados en repositorios locales o remotos.  

ℹ️ Módulos relacionados:  
- MXP Init  
- MXP FE Build  
- MXP Printer  

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.sync`  
- **Rol principal:** Sincronización de configuraciones críticas (ej. pagos con tarjeta).  
- **Función crítica:** Asegurar que los datos extraídos, transformados y cargados mantengan integridad y coherencia.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de recepción ETL | F008 | Recibe y valida procesos ETL de configuración de pagos. |

### 3.2. Capa de extracción
| Componente | Código | Descripción |
|------------|--------|-------------|
| Extractor SQL Server | E001 | Obtiene datos de **clasificaciones** desde repositorio Azure SQL. |
| Extractor Mongo | E002 | Obtiene datos de homologación (`uuids`) desde MongoDB. |

### 3.3. Capa de transformación
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformador de clasificaciones | T008-1 | Aplica reglas de transformación para homologar clasificaciones y entidades. |

### 3.4. Capa de carga
| Componente | Código | Descripción |
|------------|--------|-------------|
| Loader Mongo | L001 | Inserta datos transformados en colecciones de Mongo (ej. `Settings_BO_Payment_Card_Settings`, `transformation_classifications`). |

---

## 4. Gestión de procesos
**Flujo de trabajo estándar**  
1. Ingestión → F008 recibe la configuración.  
2. Extracción → E001 (SQL Server) y E002 (Mongo) obtienen datos.  
3. Transformación → T008-1 procesa homologaciones.  
4. Carga → L001 persiste resultados en repositorio destino.  
5. Distribución → Los datos sincronizados quedan listos para las fachadas.  

---

## 5. Endpoints del sistema

### 📌 Sincronización de Configuración de Pagos con Tarjeta
#### 1. Ejecutar sincronización
```http
POST {{server_sync}}/api/v1/facade
Authorization: Bearer {{token}}
Content-Type: application/json
```

**Body ejemplo**
```json
{
  "code": "F008",
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "chunkLoad": 100,
  "processes": {
    "EXTRACTS": [
      {
        "code": "E001",
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "withResponse": true,
        "withCache": false,
        "configuration": {
          "connection": {
            "server": "{{server_mongo}}",
            "user": "{{user_mongo}}",
            "password": "{{pass_mongo}}",
            "repository": "DEV_KFC_MXPi_Settings",
            "adapter": "Mongo"
          },
          "entities": [
            {
              "name": "Settings_BO_Payment_Card_Settings",
              "filters": [
                { "name": "country_id", "operator": "equals", "datatype": "uuid", "value": "d6cac83d-6ce5-f62e-2628-1e2e9ae7b51f" },
                { "name": "cdn_id", "operator": "equals", "datatype": "uuid", "value": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af" },
                { "name": "restaurant_id", "operator": "equals", "datatype": "uuid", "value": "e17d03da-b8b6-f424-febc-3a767b6401bb" }
              ]
            }
          ]
        }
      }
    ],
    "TRANSFORMS": [
      {
        "code": "T080",
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "withResponse": true,
        "withCache": true,
        "country": "ECU"
      }
    ],
    "LOADS": [
      {
        "code": "L001",
        "withCache": false,
        "withResponse": true,
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "configuration": {
          "connection": {
            "server": "{{server_mongo}}",
            "user": "{{user_mongo}}",
            "password": "{{pass_mongo}}",
            "repository": "MXP_integrations",
            "adapter": "Mongo"
          },
          "entities": [
            {
              "name": "Settings_BO_Payment_Card_Settings",
              "toName": "Settings_BO_Payment_Card_Settings",
              "properties": [
                { "fromName": "_id", "toName": "_id", "toType": "uuid", "isPrimaryKey": "true" },
                { "fromName": "country_id", "toName": "country_id", "toType": "uuid" },
                { "fromName": "cdn_id", "toName": "cdn_id", "toType": "uuid" },
                { "fromName": "restaurant_id", "toName": "restaurant_id", "toType": "uuid" },
                { "fromName": "cash_registers", "toName": "cash_registers", "toType": "array" }
              ]
            }
          ]
        }
      }
    ],
    "PUBLISHERS": []
  }
}
```

**Ejemplo de respuesta**
```json
{
  "message": "Error processing request: 'name'"
}
```

---

## 6. Patrones de sincronización
- Recepción centralizada → F008 concentra el inicio del flujo.  
- Extracción distribuida → SQL Server y Mongo.  
- Transformación → Procesos homologadores en T080/T008-1.  
- Carga coordinada → Persistencia en repositorios Mongo.  

---

## 7. Escenarios de uso
✅ **Caso exitoso:**  
Extracción (E001/E002) → Transformación (T080/T008-1) → Carga (L001) → Configuración disponible en fachada.  

⚠️ **Posibles fallos:**  
- **Error en extracción:** repositorio no disponible.  
- **Error en transformación:** datos inconsistentes.  
- **Error en carga:** problemas de escritura en destino.  

---

## 8. Preguntas frecuentes (IA Support)

**Q:** ¿Qué ocurre si falla la conexión a SQL Server/Mongo?  
**A:** El proceso E001/E002 retorna error y se aborta la sincronización.  

**Q:** ¿Se pueden reintentar procesos fallidos?  
**A:** Sí, relanzando el endpoint con el mismo `syncId`.  

**Q:** ¿Dónde se almacenan los datos finales?  
**A:** En el repositorio configurado en **LOADS** (`MXP_integrations`).  

---

## 9. Referencias
- `mxpi-sync/facade/readme.md`  
- Colección Postman: `Sync-Config-Payments-For-Card`  

---

## 10. Contacto
- **Equipo responsable:** Integraciones MAXPOINT  
- **Canal de soporte:** `#mxp-integraciones` (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico  
