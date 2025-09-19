# Módulo de Carga de Restaurantes Legacy (MXPI-Server-Legacy-Restaurants)

## 1. Propósito y alcance

El módulo **MXPI-Server-Legacy-Restaurants** gestiona la extracción, transformación y carga de información de restaurantes provenientes del sistema **Legacy** hacia el ecosistema **MAXPOINT V2.0**.

Este microservicio asegura la homologación y coherencia de los datos de restaurantes entre sistemas antiguos y modernos, garantizando integridad en los procesos ETL (Extract, Transform, Load).

---

## 2. Descripción general del sistema

* **Microservicio:** `mxpv2.integration.restaurants`
* **Rol principal:** administrar el flujo ETL de información de restaurantes desde **Legacy** a **MAXPOINT V2.0**.
* **Función crítica:** homologar la información de restaurantes para evitar inconsistencias en la plataforma.

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente                | Código   | Descripción                                                                                    |
| ------------------------- | -------- | ---------------------------------------------------------------------------------------------- |
| Interfaz de recepción ETL | **R001** | Punto de entrada principal para iniciar la extracción de restaurantes desde el sistema Legacy. |

### 3.2. Capa de transformación

| Componente                   | Código   | Descripción                                                                                                              |
| ---------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------ |
| Homologación de restaurantes | **T001** | Procesa y transforma la información recibida desde Legacy, asegurando que cumpla con la estructura definida en MAXPOINT. |

### 3.3. Capa de carga

| Componente       | Código   | Descripción                                                                    |
| ---------------- | -------- | ------------------------------------------------------------------------------ |
| Carga en MongoDB | **L001** | Almacena los datos homologados de restaurantes en los repositorios de destino. |

---

## 4. Gestión de procesos

**Flujo estándar:**

1. **Ingesta** → El componente `R001` recibe la solicitud con datos de restaurantes desde el servidor Legacy.
2. **Validación** → Se verifica estructura e integridad de la data.
3. **Transformación** → Se homologan los datos mediante `T001`.
4. **Carga** → La información se persiste en MongoDB mediante `L001`.

---

## 5. Endpoint principal

### **POST - Load Restaurants Legacy**

```
{{server_legacy}}/api/v1/facade/restaurants
```

#### **Body (ejemplo):**

```json
{
  "code": "R001",
  "withResponse": true,
  "withCache": false,
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "country": "{{country}}",
  "franchise_id": {{legacy_cdn_id}},
  "processes": {
    "EXTRACTS": [
      {
        "code": "E001",
        "configuration": {
          "connection": {
            "server": "{{server_legacy}}",
            "user": "{{user_legacy}}",
            "password": "{{pass_legacy}}",
            "adapter": "SqlServer"
          },
          "entities": [
            {
              "name": "mxp_v2.View_Restaurants",
              "filters": [
                {
                  "name": "cdn_id",
                  "operator": "equals",
                  "value": "{{legacy_cdn_id}}"
                }
              ]
            }
          ]
        }
      }
    ],
    "TRANSFORMS": [
      {
        "code": "T001",
        "withResponse": true,
        "country": "{{country}}"
      }
    ],
    "LOADS": [
      {
        "code": "L001",
        "withResponse": true,
        "configuration": {
          "connection": {
            "server": "{{server_mongo}}",
            "user": "{{user_mongo}}",
            "password": "{{pass_mongo}}",
            "adapter": "MongoDB"
          },
          "entities": [
            {
              "name": "restaurants",
              "toName": "restaurants"
            }
          ]
        }
      }
    ]
  }
}
```

#### **Response esperado (200 OK):**

```json
{
  "syncId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "data": null,
  "cacheId": null
}
```

---

## 6. Escenarios de uso

✅ **Caso exitoso:** datos de restaurantes válidos → validación → transformación → carga exitosa en MongoDB.
⚠️ **Fallo en validación:** datos incorrectos → rechazo → log de error.
⚠️ **Fallo en transformación:** error en `T001` → rollback automático.
⚠️ **Fallo en carga:** `MongoDB` no disponible → reintento automático según política de retries.

---

## 7. Preguntas frecuentes (IA Support)

**Q:** ¿Qué pasa si falla la validación?
**A:** Se rechaza el paquete y se genera log de error detallado.

**Q:** ¿Dónde se persisten los datos?
**A:** En el repositorio de MongoDB definido en la configuración.

**Q:** ¿Qué pasa si MongoDB no responde?
**A:** El sistema aplica retry policy. Si falla de forma persistente, se debe revisar la conexión.

---

## 8. Referencias

* `mxp-server-legacy/process/readme.md` 
* `mxp-mongo/process/readme.md` 

**Contacto:**

* Equipo responsable: Integraciones MAXPOINT
* Canal de soporte: `#mxp-integraciones` (Slack interno)
* Escalamiento: Arquitectura de datos → Liderazgo técnico
