
# Módulo de Propagación (MXP-Propagador)

## 1. Propósito y alcance
El módulo **MXP-Propagador** gestiona la propagación de configuraciones e impuestos en restaurantes dentro del ecosistema MAXPOINT V2.0.  

- Permite habilitar y configurar impuestos por PLU en restaurantes.  
- Asegura la coherencia de reglas fiscales entre franquicias y locales.  
- Coordina la propagación de impuestos con fechas de inicio y fin de vigencia.  

ℹ️ Para más detalles de módulos relacionados, consulta:  

- MXP POS  
- MXP Configuration  
- MXP Init  

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.propagator`  
- **Rol principal:** gestionar procesos de propagación de impuestos y configuraciones en restaurantes.  
- **Función crítica:** asegurar que los cambios de impuestos se apliquen correctamente en PLUs y restaurantes asociados.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de propagación
| Componente                       | Código | Descripción |
|----------------------------------|--------|-------------|
| API de propagación de impuestos  | P201   | Permite iniciar propagación de impuestos por restaurante y PLU. |

🔑 El **P201** asegura que las configuraciones de impuestos se propaguen en los restaurantes definidos, respetando fechas y tasas configuradas.  

---

## 4. Gestión de procesos

**Flujo de trabajo estándar:**
1. Solicitud de propagación → El endpoint recibe la configuración de impuestos por restaurante.  
2. Validación → Se verifican parámetros (`user`, `restaurant_id`, `plu_id`, `tax_id`, fechas y tasas).  
3. Procesamiento → El sistema aplica las reglas de propagación en los PLUs correspondientes.  
4. Resultado → Confirmación de propagación o error detallado.  

---

## 5. Puntos de integración
El propagador conecta configuraciones fiscales con el ecosistema:  

- **Entrada:** solicitudes desde módulos de administración y configuración.  
- **Salida:** impuestos habilitados en PLUs y restaurantes.  
- **Rol central:** mantener consistencia en la aplicación de reglas fiscales en toda la franquicia.  

---

## 6. Endpoints documentados

### 6.1 TaxesEmbedding – Start Restaurant Enabling Taxes Propagation  

- **Método:** `POST`  
- **URL:**  
  ```
  https://apim-uat-bo.azure-api.net/propagations/api/v1/propagationlegacy/startrestaurantenablingtaxespropagation?user={user_uuid}
  ```

**Query Params**  
- `user` (UUID) → ID del usuario que ejecuta la propagación.  

**Body (raw – JSON):**
```json
{
  "restaurantEnablingTaxes": [
    {
      "_id": "d9b3cd1e-25c4-4a1e-b634-9f89f26e14db",
      "user_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "cdn_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
      "restaurant_id": "e17d03da-b8b6-f424-febc-3a767b6401bb",
      "plus": [
        {
          "plu_id": "2d2ed803-ade4-eada-7f90-2af45a24ab0a",
          "enabling_taxes": [
            {
              "tax_id": "d00c4be9-a547-f72d-b87b-2abaebc07f4b",
              "configuration": [
                {
                  "start_date": "2025-07-10T19:19:42.021Z",
                  "end_date": "2025-07-14T19:19:42.021Z",
                  "tax_rate": 10
                }
              ]
            }
          ]
        }
      ],
      "created_user": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "updated_user": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "created_at": "2025-07-08T19:19:42.021Z",
      "updated_at": "2025-07-08T19:19:42.021Z"
    }
  ]
}
```

**Ejemplo de Request (cURL):**
```bash
curl --location 'https://apim-uat-bo.azure-api.net/propagations/api/v1/propagationlegacy/startrestaurantenablingtaxespropagation?user=3fa85f64-5717-4562-b3fc-2c963f66afa6' --header 'Content-Type: application/json' --data '{ ... }'
```

---

## 7. Escenarios de uso

✅ **Caso exitoso**  
- Solicitud correcta → validación de datos → propagación aplicada → confirmación OK.  

⚠️ **Posibles fallos**  
- Parámetros inválidos → `400 Bad Request`.  
- Usuario no autorizado → `401 Unauthorized`.  
- Error interno del servicio → `500 Internal Server Error`.  

---

## 8. Preguntas frecuentes (para IA Support)

- **Q:** ¿Qué ocurre si un PLU no tiene impuestos configurados?  
  **A:** La propagación lo ignora y continúa con el resto.  

- **Q:** ¿Se pueden configurar impuestos con vigencia limitada?  
  **A:** Sí, mediante `start_date` y `end_date`.  

- **Q:** ¿Qué pasa si un restaurante ya tiene configuraciones activas?  
  **A:** El propagador actualiza las configuraciones existentes.  

---

## 9. Referencias
Fuentes internas:  
- `mxp-propagator/taxes/readme.md`  
- `mxp-configuration/taxes/readme.md`  
- API Gateway: `apim-uat-bo.azure-api.net`  

---

## 10. Contacto
- **Equipo responsable:** Integraciones MAXPOINT – Propagador  
- **Canal de soporte:** #mxp-propagador (Slack interno)  
- **Escalamiento:** Arquitectura de POS → Liderazgo técnico  
