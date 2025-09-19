# Módulo de Pagos (MXP-POS)

## 1. Propósito y alcance

MXP-POS es el microservicio encargado de la **gestión y consulta de
métodos de pago y su estatus dentro del ecosistema MAXPOINT V2.0**.\
Este módulo asegura que los puntos de venta (POS) dispongan de la
información actualizada sobre los métodos de pago disponibles,
restricciones aplicables y estados de las transacciones.

Funciones principales:\
- Centralizar y exponer información sobre **métodos y tipos de pago**
disponibles por restaurante, dispositivo y caja.\
- Proveer el estado de los pagos registrados en el sistema.\
- Garantizar coherencia en la validación y disponibilidad de medios de
pago.

------------------------------------------------------------------------

## 2. Descripción general del sistema

-   **Microservicio:** `mxp-pos`\
-   **Rol principal:** Exponer la información de medios de pago y
    estatus de pagos en el ecosistema.\
-   **Función crítica:** Permitir que los POS validen restricciones de
    pagos y consulten estados de forma confiable y centralizada.

------------------------------------------------------------------------

## 3. Arquitectura y componentes

### 3.1. Capa de pagos

  -----------------------------------------------------------------------
  Componente                Código            Descripción
  ------------------------- ----------------- ---------------------------
  API de Métodos de Pago    P001              Permite obtener los métodos
                                              de pago habilitados y sus
                                              restricciones.

  API de Estado de Pagos    P002              Retorna el listado de
                                              estados posibles de los
                                              pagos procesados en el
                                              sistema.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 4. Gestión de procesos

### Flujo estándar de consulta

1.  **POS realiza solicitud** → al endpoint correspondiente.\
2.  **Validación de parámetros** → se verifica franquicia, restaurante,
    caja y dispositivo.\
3.  **Procesamiento** → el sistema determina los métodos de pago
    disponibles o estados de pagos.\
4.  **Respuesta** → se devuelve información estructurada y lista para
    consumo en el POS.

------------------------------------------------------------------------

## 5. Puntos de integración

-   **Entrada:** solicitudes de POS (cajas registradoras, terminales de
    restaurantes, dispositivos móviles).\
-   **Salida:**
    -   Métodos y restricciones de pago disponibles.\
    -   Estados de pagos definidos en el ecosistema.

------------------------------------------------------------------------

## 6. Endpoints documentados

### 6.1. Endpoint: **get_payment_methods_and_types_restrictions**

-   **Método:** `GET`\
-   **URL:**\

``` http
https://apim-uat-ec.azure-api.net/payments/api/v1/paymenttype/get_payment_methods_and_types_restrictions
```

#### Query Params

  ----------------------------------------------------------------------------------
  Param                  Ejemplo de valor                       Descripción
  ---------------------- -------------------------------------- --------------------
  franchise              bc2d8ada-e25e-1bb7-55fe-32d1205ac4af   Identificador de la
                                                                franquicia.

  device                 b4645748-78c2-0160-5667-79105889c1d7   Identificador único
                                                                del dispositivo POS.

  restaurant             35b71d9e-7e9c-8f4e-25c0-472cfbbff75c   Identificador del
                                                                restaurante.

  cash_register          ca51f8e2-c2ab-5df4-8abe-bc90fbe11edc   Caja registradora
                                                                asociada al POS.
  ----------------------------------------------------------------------------------

📌 **Función:** Retorna los métodos de pago disponibles para un
restaurante específico, considerando restricciones de dispositivo,
franquicia y caja.

------------------------------------------------------------------------

### 6.2. Endpoint: **paymentsstatus**

-   **Método:** `GET`\
-   **URL:**\

``` http
https://apim-uat-ec.azure-api.net/payments/api/v1/paymentsstatus/get_all_status_payment
```

📌 **Función:** Retorna el listado de todos los estados de pagos
soportados en el sistema (ej. pendiente, aprobado, rechazado, etc.).

------------------------------------------------------------------------

## 7. Escenarios de uso

✅ **Caso exitoso:**\
- El POS solicita métodos de pago con los parámetros correctos.\
- El servicio responde con lista de métodos habilitados y restricciones.

⚠️ **Posibles fallos:**\
- **Parámetros inválidos:** el servicio no retorna información.\
- **Restricciones aplicadas:** algunos métodos no aparecen disponibles.\
- **Falla en conexión:** el POS no obtiene respuesta → requiere
reintento.

------------------------------------------------------------------------

## 8. Preguntas frecuentes (IA Support)

**Q: ¿Qué pasa si no envío el parámetro `cash_register`?**\
A: La API no podrá determinar correctamente los métodos de pago
disponibles, ya que dependen de la caja específica.

**Q: ¿El endpoint `paymentsstatus` cambia según restaurante?**\
A: No, este endpoint devuelve estados de pagos globales, aplicables a
todo el ecosistema.

**Q: ¿Cómo manejar restricciones de métodos de pago?**\
A: El POS debe validar las restricciones recibidas y deshabilitar
visualmente los métodos no permitidos.

**Q: ¿La API persiste datos de pagos?**\
A: No, solo consulta y devuelve información. La persistencia se gestiona
en módulos de pagos transaccionales.

------------------------------------------------------------------------

## 9. Referencias

-   Documentación interna de pagos: `mxp-pos/payments/readme.md`\
-   API Gateway: `apim-uat-ec.azure-api.net`\
-   Microservicio: `mxp.pos`

------------------------------------------------------------------------

## 10. Contacto

-   **Equipo responsable:** Integraciones MAXPOINT -- Pagos\
-   **Canal de soporte:** #mxp-pos (Slack interno)\
-   **Escalamiento:** Arquitectura de Pagos → Liderazgo técnico
