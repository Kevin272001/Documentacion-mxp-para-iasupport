# Módulo de Inicialización de Usuarios POS (MXPI-INIT QA)
1.- Homologation-Users-Pos-QA  

---

## 1. Propósito y alcance
**MXPI-INIT Load-Users-Pos-DEV** es el microservicio responsable de la homologación, carga e integración de usuarios y perfiles POS dentro del ecosistema **MAXPOINT V2.0**.  
Gestiona el proceso de extracción desde sistemas **legacy (SQL Server)**, transforma los identificadores y finalmente los carga en **MongoDB** para su uso en la plataforma.  

Funciones principales:
- Homologación de identificadores entre sistemas legacy (v1) y el nuevo sistema (v2).
- Carga y persistencia de datos homologados en repositorios unificados.
- Asegurar coherencia de identificadores de usuarios y perfiles POS.
- Integración con procesos ETL (Extract, Transform, Load).

ℹ️ Para más detalles de los módulos relacionados, consulta:
- **MXP Init**
- **MXP FE Build**
- **MXP Printer**

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.init.users.pos`
- **Rol principal:** gestionar procesos ETL para homologación de usuarios y perfiles POS.
- **Función crítica:** garantizar que los usuarios POS del sistema legacy se homologuen y persistan correctamente en el sistema v2.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de homologación | F001 | Punto de entrada principal para homologación de usuarios y perfiles POS. |

### 3.2. Capa de transformación
| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación de IDs | H001 | Aplica reglas de homologación de IDs v1 → v2. |

### 3.3. Capa de carga
| Componente | Código | Descripción |
|------------|--------|-------------|
| Carga Mongo | L001 | Inserta registros homologados en MongoDB (`homologation_v1_v2`). |

---

## 4. Gestión de procesos
**Flujo de trabajo estándar:**  
1. **Extracción:** Se consultan usuarios y perfiles POS desde SQL Server (`mxp_v2.View_User_POS`, `Perfil_Pos`).  
2. **Validación:** Se asegura integridad de datos (filtros por `cdn_id`, `prf_descripcion`).  
3. **Transformación:** Se homologan los IDs con reglas definidas (`H001`).  
4. **Carga:** Se insertan datos en MongoDB (`uuids`).  
5. **Confirmación:** El proceso retorna logs de homologación y estado de carga.

---

## 5. Puntos de integración
- **Entrada:**  
  - Usuarios POS (`mxp_v2.View_User_POS`)  
  - Perfiles POS (`Perfil_Pos`)  
- **Salida:**  
  - MongoDB colección `uuids`  
- **Rol central:** asegurar la homologación correcta de usuarios y perfiles POS.

---

## 6. Patrones de sincronización
- Extracción controlada desde SQL Server.  
- Homologación unificada con reglas en `H001`.  
- Persistencia centralizada en MongoDB.  
- Validación de integridad antes de cargar.  

---

## 7. Escenarios de uso

### ✅ Caso exitoso
1. Extracción correcta desde SQL Server.  
2. Homologación en `H001`.  
3. Carga exitosa en MongoDB.  
4. Respuesta de confirmación al cliente.  

### ⚠️ Posibles fallos
- **Error en extracción:** conexión o credenciales inválidas.  
- **Error en homologación:** reglas de transformación fallidas → rollback.  
- **Error en carga:** indisponibilidad de MongoDB → política de reintentos.  

---

## 8. Endpoints documentados

### 🔹 1. Homologation-Users-Pos-QA
**Endpoint:**  
```http
POST {{server_init}}/api/v1/facade
```

**Autorización:**
```
Bearer {{token}}
```

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
    "EXTRACTS": [...],
    "TRANSFORMS": [...],
    "LOADS": [...]
  }
}
```

- **Entidad extraída:** `mxp_v2.View_User_POS`  
- **Entidad cargada:** `uuids` (MongoDB)

---

### 🔹 2. Homologation-Users-Profile
**Endpoint:**  
```http
POST {{server_init}}/api/v1/facade
```

**Autorización:**
```
Bearer {{token}}
```

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
    "EXTRACTS": [...],
    "TRANSFORMS": [...],
    "LOADS": [...]
  }
}
```

- **Entidad extraída:** `Perfil_Pos` (con filtros por *Administrador Local, Cajero, Mesero*)  
- **Entidad cargada:** `uuids` (MongoDB)

---

## 9. Preguntas frecuentes (IA Support)

**Q:** ¿Qué pasa si falla la extracción?  
**A:** Se genera un log de error y no continúa el flujo.  

**Q:** ¿Dónde se almacenan los datos homologados?  
**A:** En MongoDB (`homologation_v1_v2.uuids`).  

**Q:** ¿Qué debo revisar si falla la carga?  
**A:** Conexión a MongoDB y logs de proceso `L001`.  

---

## 10. Referencias
Fuentes internas:  
- `mxpi-init/process/readme.md`  
- `mxp-sync/process/readme.md`  
- `mxp-feBuild/process/readme.md`  

---

## 11. Contacto
- **Equipo responsable:** Integraciones MAXPOINT  
- **Canal de soporte:** `#mxp-integraciones` (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico  
