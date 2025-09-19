# Módulo de Inicialización de Usuarios POS (MXPi-init Users-Pos DEV)

## 1. Propósito y alcance

**MXPi-init Users-Pos** es el sistema de inicialización y homologación de identificadores de usuarios y perfiles POS dentro del ecosistema **MAXPOINT V2.0**.  
Este módulo garantiza la correcta conversión, validación y carga de identificadores desde sistemas legacy hacia la nueva plataforma (v2).  

- Gestiona el flujo ETL (**Extract, Transform, Load**) de usuarios POS y perfiles POS.  
- Garantiza la consistencia de los identificadores en toda la plataforma.  
- Permite homologar datos críticos para la operación de usuarios POS.  

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- MXP Init  
- MXP Homologation  
- MXP Sync  

---

## 2. Descripción general del sistema

- **Microservicio:** `mxpv2.integration.userspos.dev`  
- **Rol principal:** gestionar el proceso ETL de usuarios y perfiles POS.  
- **Función crítica:** homologar los IDs legacy (v1) a identificadores unificados (v2).  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Facade Init | H000 | Punto de entrada para homologación de usuarios y perfiles POS. |

### 3.2. Capa de extracción
| Componente | Código | Descripción |
|------------|--------|-------------|
| Extractor Usuarios POS | E001 | Extrae información de `mxp_v2.View_User_POS` en SQL Server. |
| Extractor Perfiles POS | E001 | Extrae datos de `Perfil_Pos` aplicando filtros (Administrador Local, Cajero, Mesero). |

### 3.3. Capa de transformación
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformador Homologación | H001 | Transforma datos v1 → v2 con validación por país. |

### 3.4. Capa de carga
| Componente | Código | Descripción |
|------------|--------|-------------|
| Loader MongoDB | L001 | Carga datos homologados en la colección `uuids` del repositorio `homologation_v1_v2`. |

---

## 4. Gestión de procesos

### Flujo de trabajo estándar
1. **Ingesta de datos** → Se recibe la solicitud en la fachada (`H000`).  
2. **Extracción (E001)** → Se consulta SQL Server para obtener `Users POS` o `Perfiles POS`.  
3. **Transformación (H001)** → Se homologan IDs v1 a IDs v2.  
4. **Carga (L001)** → Se almacenan resultados en MongoDB (`uuids`).  
5. **Distribución** → Los datos homologados quedan disponibles para otros módulos de la plataforma.  

---

## 5. Endpoints documentados

### 5.1. Homologación de Usuarios POS

**Endpoint:**  
```http
POST {{server_init}}/api/v1/facade
```

**Autorización:**  
`Bearer Token {{token}}`  

**Body ejemplo:**  
```json
{
  "code": "H000",
  "withResponse": false,
  "withCache": true,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "country": "{{country}}",
  "franchise_id": {{legacy_cdn_id}},
  "processes": {
    "EXTRACTS": [
      {
        "code": "E001",
        "withResponse": true,
        "withCache": true,
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "configuration": {
          "connection": {
            "server": "{{server_legacy}}",
            "port": "",
            "user": "{{user_legacy}}",
            "password": "{{pass_legacy}}",
            "repository": "{{repository_legacy}}",
            "adapter": "SqlServer"
          },
          "entities": [
            {
              "name": "mxp_v2.View_User_POS",
              "properties": [{ "name": "IDUsersPos", "type": "base64" }],
              "filters": [{ "name": "cdn_id", "operator": "equals", "value": "{{legacy_cdn_id}}" }]
            }
          ]
        }
      }
    ],
    "TRANSFORMS": [
      {
        "code": "H001",
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "withResponse": true,
        "withCache": false,
        "country": "{{country}}"
      }
    ],
    "LOADS": [
      {
        "code": "L001",
        "withCache": true,
        "withResponse": true,
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "configuration": {
          "connection": {
            "server": "{{server_mongo}}",
            "port": "",
            "user": "{{user_mongo}}",
            "password": "{{pass_mongo}}",
            "repository": "homologation_v1_v2",
            "adapter": "{{adapter_mongo}}"
          },
          "entities": [{ "name": "uuids", "toName": "uuids" }]
        },
        "dataInput": {}
      }
    ]
  }
}
```

---

### 5.2. Homologación de Perfiles POS

**Endpoint:**  
```http
POST {{server_init}}/api/v1/facade
```

**Autorización:**  
`Bearer Token {{token}}`  

**Body ejemplo:**  
```json
{
  "code": "H000",
  "withResponse": false,
  "withCache": true,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "country": "{{country}}",
  "franchise_id": 0,
  "processes": {
    "EXTRACTS": [
      {
        "code": "E001",
        "withResponse": true,
        "withCache": true,
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "configuration": {
          "connection": {
            "server": "{{server_legacy}}",
            "port": "",
            "user": "{{user_legacy}}",
            "password": "{{pass_legacy}}",
            "repository": "{{repository_legacy}}",
            "adapter": "SqlServer"
          },
          "entities": [
            {
              "name": "Perfil_Pos",
              "properties": [{ "name": "IDPerfilPos", "type": "base64" }],
              "filters": [{ "name": "prf_descripcion", "operator": "equals", "value": "Administrador Local" }]
            },
            {
              "name": "Perfil_Pos",
              "properties": [{ "name": "IDPerfilPos", "type": "base64" }],
              "filters": [{ "name": "prf_descripcion", "operator": "equals", "value": "Cajero" }]
            },
            {
              "name": "Perfil_Pos",
              "properties": [{ "name": "IDPerfilPos", "type": "base64" }],
              "filters": [{ "name": "prf_descripcion", "operator": "equals", "value": "Mesero" }]
            }
          ]
        }
      }
    ],
    "TRANSFORMS": [
      {
        "code": "H001",
        "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "withResponse": true,
        "withCache": false,
        "country": "{{country}}"
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
            "port": "",
            "user": "{{user_mongo}}",
            "password": "{{pass_mongo}}",
            "repository": "homologation_v1_v2",
            "adapter": "{{adapter_mongo}}"
          },
          "entities": [{ "name": "uuids", "toName": "uuids" }]
        },
        "dataInput": {}
      }
    ]
  }
}
```

---

## 6. Escenarios de uso

- ✅ **Caso exitoso:** Usuarios POS o Perfiles extraídos → transformados → cargados en MongoDB.  
- ⚠️ **Error en extracción:** credenciales incorrectas o entidad no encontrada → log de error generado.  
- ⚠️ **Error en transformación:** IDs no válidos → rollback automático.  
- ⚠️ **Error en carga:** MongoDB no disponible → reintentos según política configurada.  

---

## 7. Preguntas frecuentes (para IA Support)

- **Q:** ¿Qué pasa si falla la validación en la transformación (H001)?  
  **A:** El registro no se procesa y se envía alerta al sistema de monitoreo.  

- **Q:** ¿El microservicio almacena datos de manera persistente?  
  **A:** No, únicamente orquesta la homologación. La persistencia ocurre en MongoDB.  

- **Q:** ¿Qué hacer si MongoDB no responde?  
  **A:** Revisar conexión (`server_mongo`, credenciales). El sistema reintenta antes de fallar definitivamente.  

---

## 8. Referencias

- **Fuentes internas:**  
  - `mxp-init/process/readme.md`  
  - `mxp-homologation/process/readme.md`  

- **Microservicio:** `mxpv2.integration.userspos.dev`  

- **Contacto:**  
  - Equipo responsable: **Integraciones MAXPOINT**  
  - Canal de soporte: `#mxp-integraciones` (Slack interno)  
  - Escalamiento: **Arquitectura de datos → Liderazgo técnico**  
