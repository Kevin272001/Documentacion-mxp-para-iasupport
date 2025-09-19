# Módulo de Sincronización (MXP-Sync)

## 1. Propósito y alcance
El módulo **MXP-Sync** es el responsable de la **orquestación de sincronización de datos** dentro del ecosistema **MAXPOINT V2.0**.  

- Gestiona el flujo de datos procesados entre diferentes módulos.  
- Garantiza la coherencia del sistema en toda la plataforma.  
- Coordina los procesos **ETL (Extract, Transform, Load)**.  
- Recibe datos procesados de las tuberías de transformación y los distribuye a las interfaces de fachada (*facades*).  

ℹ️ **Módulos relacionados**:  
- MXP Init  
- MXP FE Build  
- Impresora MXP  

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.sync`  
- **Rol principal:** Gestionar procesos de integración y sincronización.  
- **Función crítica:** Asegurar que los datos transformados lleguen a las fachadas correctas dentro del flujo ETL.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de recepción ETL | `F008` | Punto de entrada principal para datos ETL. Valida estructura e integridad. |

### 3.2. Capa de transformación
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación de configuración de pagos | `T080` | Procesa y asegura la correcta estructura de la configuración de pagos. |

---

## 4. Gestión de procesos

**Flujo estándar de sincronización de pagos con tarjeta (QA):**  

1. **Ingestión de datos** → `F008` recibe la solicitud de sincronización.  
2. **Validación** → Se verifica integridad y estructura de los datos de configuración de pagos.  
3. **Extracción** → Se ejecuta el proceso `E001` desde **Mongo QA** (`QA_KFC_MXPi_Settings`).  
4. **Transformación** → Se aplica `T080` para garantizar la estructura requerida.  
5. **Carga** → Se inserta en el repositorio destino (`MXPi_Settings`) vía `L001`.  
6. **Distribución** → Los datos quedan listos para ser consumidos por fachadas de destino.  

---

## 5. Puntos de integración
- **Entrada:** Configuración de pagos con tarjeta desde **Mongo QA**.  
- **Salida:** Persistencia en `MXPi_Settings` con propiedades mapeadas (`_id`, `country_id`, `cdn_id`, `restaurant_id`, `cash_registers`).  
- **Rol central:** Mantener consistencia en la configuración de pagos multicanal.  

---

## 6. Patrones de sincronización
- **Recepción centralizada** → Todos los datos de pagos se reciben vía `F008`.  
- **Procesamiento de metadatos de pagos** → El transformador `T080` asegura coherencia.  
- **Distribución coordinada** → Los datos se cargan en el repositorio destino (`L001`).  
- **Garantía de coherencia** → Se asegura que la configuración de pagos sea uniforme entre entornos.  

---

## 7. Escenarios de uso

✅ **Caso exitoso**  
- Se recibe payload válido → `E001` extrae → `T080` transforma → `L001` carga → Configuración disponible en destino.  

⚠️ **Posibles fallos**  
- **Error en validación:** Paquete rechazado → revisar conexión y filtros (`country_id`, `cdn_id`, `restaurant_id`).  
- **Error en transformación (T080):** Se activa rollback y alerta en logs.  
- **Error en carga (L001):** Si el repositorio destino no está disponible, se activa política de reintentos.  

---

## 8. Endpoint documentado

### Sync-Config-Payments-For-Card-QA  
**Método:** `POST`  
**URL:**  
```http
{{server_sync}}/api/v1/facade
```

**Autorización:**  
```http
Bearer {{token}}
```

**Body (ejemplo):**
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
            "server": "{{server_mongo_qa}}",
            "user": "{{user_mongo_qa}}",
            "password": "{{pass_mongo_qa}}",
            "repository": "QA_KFC_MXPi_Settings",
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
            "server": "192.168.101.13",
            "port": "27020",
            "user": "admin",
            "password": "admin123",
            "repository": "MXPi_Settings",
            "adapter": "MongoLocal"
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

**Ejemplo de respuesta:**
```json
{
  "message": "Error processing request: 'name'"
}
```

---

## 9. Referencias
- `mxp-sync/process/readme.md (1-28)`  
- `mxp-feBuild/process/readme.md (9-10)`  
- Microservicio: `mxpv2.sync`  

---

## 10. Contacto
- **Equipo responsable:** Integraciones MAXPOINT  
- **Canal de soporte:** #mxp-integraciones (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico  
