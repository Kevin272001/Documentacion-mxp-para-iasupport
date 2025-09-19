# Módulo de Carga de Números PLU (MXPi-init 1. Load-Number-Plu QA)

## 1. Propósito y alcance

**MXPi-init 1. Load-Number-Plu QA** es el microservicio encargado de la **carga, homologación y sincronización de números PLU** dentro del ecosistema **MAXPOINT V2.0**.  
Su función principal es recibir datos desde sistemas **legacy (SQL Server)**, transformarlos y cargarlos en **MongoDB**, asegurando consistencia y disponibilidad en el entorno QA.

Funciones principales:
- Cargar y homologar información de **números PLU**.
- Procesar medios de venta asociados a menús.
- Coordinar procesos **ETL (Extract, Transform, Load)** en QA.

ℹ️ Módulos relacionados:
- **MXP Init**
- **MXP FE Build**
- **MXP Printer**

---

## 2. Descripción general del sistema

- **Microservicio:** `mxpv2.integration.init.load.number.plu.qa`
- **Rol principal:** gestionar la carga y sincronización de números PLU en QA.
- **Función crítica:** transformar y persistir datos de PLU y medios de venta en MongoDB para pruebas y QA.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código | Descripción                                      |
|---------------------------|--------|--------------------------------------------------|
| Interfaz de carga de PLU  | F100   | Punto de entrada principal para procesos de PLU. |

### 3.2. Capa de transformación

| Componente                    | Código | Descripción                                     |
|-------------------------------|--------|-------------------------------------------------|
| Transformación PLU (general)  | T128   | Procesa datos de homologación PLU.              |
| Transformación medios de venta| T129   | Procesa datos de medios de venta asociados.     |

### 3.3. Capa de carga

| Componente | Código | Descripción                                           |
|------------|--------|-------------------------------------------------------|
| Carga Mongo| L001   | Inserta datos procesados en MongoDB (`QA_KFC_MXPi_Settings`). |

---

## 4. Gestión de procesos

**Flujo de trabajo estándar:**  
1. **Extracción** → desde SQL Server y MongoDB (uuids).  
2. **Transformación** → con procesos `T128` y `T129`.  
3. **Carga** → en MongoDB QA (`plu_num_plu` y `plu_means_sale`).  
4. **Confirmación** → logs y respuesta de éxito o error.  

---

## 5. Puntos de integración

- **Entrada:**  
  - `sp_GetNumPluForPlu` desde SQL Server.  
  - `uuids` desde MongoDB.  
  - `mxp_v2.Medio_Venta` desde SQL Server.  

- **Salida:**  
  - MongoDB colección `QA_KFC_MXPi_Settings` (`plu_num_plu`, `plu_means_sale`).  

- **Rol central:** asegurar que los datos de PLU y medios de venta se sincronicen y transformen correctamente en QA.

---

## 6. Patrones de sincronización

- Extracción combinada desde **SQL Server** y **MongoDB**.  
- Transformación de IDs (`T128`, `T129`).  
- Carga centralizada en **MongoDB QA**.  
- Validación y confirmación antes de la persistencia.  

---

## 7. Escenarios de uso

### ✅ Caso exitoso
1. Se extraen datos de PLU desde SQL Server y uuids en MongoDB.  
2. Transformación en `T128`.  
3. Carga en `plu_num_plu`.  
4. Confirmación en logs de éxito.  

### ⚠️ Posibles fallos
- **Extracción:** credenciales inválidas o SP no accesible.  
- **Transformación:** error en `T128` o `T129`.  
- **Carga:** error de conexión a MongoDB QA.  

---

## 8. Endpoints documentados

### 🔹 1. Facade-Init-Plu-Num-Plu

**Método:** `POST`  
**URL:**  
```http
{{init}}/api/v1/facade
```

**Autorización:**  
```
Bearer {{token}}
```

**Body ejemplo:**  
```json
{ ... JSON completo de E001, E002, T128 y L001 ... }
```

**Ejemplo cURL:**  
```bash
curl --location -g '{{init}}/api/v1/facade' --data '{ ... JSON completo ... }'
```

---

### 🔹 2. Facade-Init-Plu-Menu-Medio

**Método:** `POST`  
**URL:**  
```http
{{init}}/api/v1/facade
```

**Autorización:**  
```
Bearer {{token}}
```

**Body ejemplo:**  
```json
{ ... JSON completo de E001, T129 y L001 ... }
```

**Ejemplo cURL:**  
```bash
curl --location -g '{{init}}/api/v1/facade' --data '{ ... JSON completo ... }'
```

---

## 9. Preguntas frecuentes (IA Support)

**Q:** ¿Qué pasa si falla la extracción?  
**A:** Se genera log y el flujo se detiene.  

**Q:** ¿Dónde se almacenan los datos?  
**A:** En MongoDB (`QA_KFC_MXPi_Settings`).  

**Q:** ¿Qué hago si falla la transformación?  
**A:** Revisar reglas de `T128`/`T129` y consultar los logs.  

**Q:** ¿Qué pasa si MongoDB QA no responde?  
**A:** Se activa la política de reintentos (retry).  

---

## 10. Referencias

- `mxpi-init/process/readme.md`  
- `mxp-sync/process/readme.md`  
- `mxp-feBuild/process/readme.md`  

---

## 11. Contacto

- **Equipo responsable:** Integraciones MAXPOINT  
- **Canal de soporte:** `#mxp-integraciones` (Slack interno)  
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico  
