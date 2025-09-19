# Módulo de Publicación de Ordenes COD (MXPI-COD)

## 1. Propósito y alcance

El módulo **MXPI-COD** es responsable de la orquestación y publicación de datos de órdenes de venta generadas en los puntos de venta (POS) dentro del ecosistema MAXPOINT V2.0. Gestiona el flujo de información entre los sistemas de venta y los módulos de integración, asegurando la coherencia y coordinación de los procesos ETL (Extract, Transform, Load). Recibe datos procesados de las aplicaciones de venta y los distribuye a las interfaces de integración.

ℹ️ Para más detalles de los módulos relacionados, consulta:
- MXP Init
- MXP FE Build
- Impresora MXP

## 2. Descripción general del sistema

- **Microservicio:** mxpv2.integration.cod
- **Rol principal:** Gestionar procesos de integración y publicación de órdenes de venta.
- **Función crítica:** Asegurar que los datos de las órdenes lleguen correctamente a los sistemas de integración dentro del flujo ETL.

## 3. Arquitectura y componentes

### 3.1. Capa de fachada

| Componente           | Código | Descripción                                              |
|----------------------|--------|----------------------------------------------------------|
| NewOrderRequest      | F200   | Publica datos de órdenes generadas en los puntos de venta.|

🔑 El endpoint actúa como gateway unificado para datos de órdenes, validando estructura e integridad.

### 3.2. Capa de transformación

| Componente | Código | Descripción                                      |
|------------|--------|--------------------------------------------------|
| Transformaciones varias | T201, T202, T203 | Procesan y preparan los datos de la orden para la carga en sistemas de integración. |

## 4. Gestión de procesos

**Flujo de trabajo estándar:**
- Ingestión de datos → Recepción de órdenes desde los puntos de venta.
- Validación → Verificación de estructura e integridad de la orden.
- Enrutamiento → Envío al proceso correcto según el contenido.
- Transformación → Procesamiento según el tipo de orden y productos.
- Distribución → Entrega a la interfaz de integración.

## 5. Puntos de integración

El publicador conecta el ecosistema MAXPOINT V2.0 con los sistemas de integración:
- **Entrada:** Recibe datos de órdenes desde los puntos de venta.
- **Salida:** Publica datos validados y transformados en el endpoint REST de integración.
- **Rol central:** Mantiene coherencia en procesos ETL inter-módulos.

## 6. Patrones de sincronización

- Recepción centralizada → Todas las órdenes llegan vía el endpoint de fachada.
- Procesamiento de metadatos → Cada transformación gestiona la información específica de la orden.
- Distribución coordinada → Los datos se envían según el tipo de orden y canal.
- Garantía de coherencia → Evita inconsistencias en la plataforma.

## 7. Escenarios de uso

### ✅ Caso exitoso
Orden válida → Recepción → Validación → Transformación → Distribución correcta a integración.

### ⚠️ Posibles fallos
- **Error en validación:** Datos rechazados, log de error, revisar fuente.
- **Error en transformación:** No se distribuyen, rollback y alerta.
- **Error en distribución:** Sistema de integración no disponible, reintentos automáticos.

## 8. Endpoint documentado

### POST NewOrderRequest

**URL:** `{{server_cod}}/api/v1/order`  
**Autenticación:** Bearer Token  
**Body (ejemplo):**
```json
{
    "pointOfSale": {
        "id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
        "code": "EC-CAJA001",
        "cashierId": "7421A2EE-B13B-EA11-80EA-000D3A019254",
        "cashierName": "pepito",
        "platform": "ios",
        "account": "KFC",
        "accountId": 1,
        "source": "App",
        "channelId": 3,
        "channelName": "App"
    },
    "client": {
        "uid": "030B9503-85CF-E511-80C6-000D3A3261F3",
        "name": "María José ",
        "lastName": "Hernández Zurita ",
        "phone": "+593986610329",
        "email": "mari_jhz@hotmail.com",
        "govIdType": "CI",
        "govIdNumber": "0929691038",
        "externalId": "030B9503-85CF-E511-80C6-000D3A3261F3",
        "billingInformation": {
            "fullName": "María José  Hernández Zurita ",
            "govIdType": "CI",
            "govIdNumber": "0929691038",
            "phone": "0986610329",
            "email": "mari_jhz@hotmail.com",
            "address": "Guayaquil",
            "externalId": "030B9503-85CF-E511-80C6-000D3A3261F3"
        },
        "additionalInfo": {
            "birthdate": "",
            "gender": ""
        }
    },
    "order": {
        "orderId": "0001292432-010103",
        "createdAt": "2024-09-11 12:14:40",
        "selectedShippingMethod": "pickup",
        "accumulatePoints": false,
        "redeemPoints": false,
        "stock": false,
        "discount": false,
        "orderComment": "",
        "annulationComment": "",
        "products": [
            {
                "productId": "401",
                "product": "COMBO IDEAL",
                "quantity": 1,
                "productComment": "",
                "price": {
                    "unitPrice": {
                        "currencyCode": "USD",
                        "subtotalWithoutTaxes": 5.65,
                        "discountPercentage": 0,
                        "discountsValue": 0,
                        "subtotalIncludeDiscounts": 5.65,
                        "taxesPercentage": 15,
                        "taxValue": 0.85,
                        "total": 6.5,
                        "totalBeforeSale": 6.5,
                        "suggestedPrice": 0
                    },
                    "totalPrice": {
                        "currencyCode": "USD",
                        "subtotalWithoutTaxes": 5.65,
                        "discountPercentage": null,
                        "discountsValue": 0,
                        "subtotalIncludeDiscounts": 5.65,
                        "taxesPercentage": 15,
                        "taxValue": 0.85,
                        "total": 6.5,
                        "totalBeforeSale": 6.5,
                        "suggestedPrice": 0
                    }
                },
                "modifierGroups": [
                    {
                        "id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
                        "description": "Es necesario elegir uno",
                        "selectedModifiers": [
                            {
                                "productId": "33616",
                                "product": "CRISPY",
                                "quantity": 1,
                                "removedQuantity": null,
                                "price": {
                                    "unitPrice": {
                                        "currencyCode": "USD",
                                        "subtotalWithoutTaxes": 0,
                                        "discountPercentage": 0,
                                        "discountsValue": 0,
                                        "subtotalIncludeDiscounts": 0,
                                        "taxesPercentage": 15,
                                        "taxValue": 0,
                                        "total": 0,
                                        "totalBeforeSale": 0,
                                        "suggestedPrice": 0
                                    },
                                    "totalPrice": {
                                        "currencyCode": "USD",
                                        "subtotalWithoutTaxes": 0,
                                        "discountPercentage": null,
                                        "discountsValue": 0,
                                        "subtotalIncludeDiscounts": 0,
                                        "taxesPercentage": 15,
                                        "taxValue": 0,
                                        "total": 0,
                                        "totalBeforeSale": 0,
                                        "suggestedPrice": 0
                                    }
                                },
                                "modifierGroups": [],
                                "additionalInfo": {
                                    "redeemed": false,
                                    "referenceUnitPrice": {
                                        "currencyCode": "USD",
                                        "subtotalWithoutTaxes": 0,
                                        "discountPercentage": 0,
                                        "discountsValue": 0,
                                        "subtotalIncludeDiscounts": 0,
                                        "taxesPercentage": 15,
                                        "taxValue": 15,
                                        "totalBeforeSale": 0,
                                        "total": 0,
                                        "suggestedPrice": 0
                                    },
                                    "referenceTotalPrice": {
                                        "currencyCode": "USD",
                                        "subtotalWithoutTaxes": 0,
                                        "discountPercentage": 0,
                                        "discountsValue": 0,
                                        "subtotalIncludeDiscounts": 0,
                                        "taxesPercentage": 15,
                                        "taxValue": 0,
                                        "total": 0,
                                        "totalBeforeSale": 0,
                                        "suggestedPrice": 0
                                    }
                                }
                            }
                        ]
                    }
                ],
                "additionalInfo": {
                    "strategy": " ",
                    "relatedProductId": " ",
                    "redeemed": false,
                    "sort": 1,
                    "referenceUnitPrice": {
                        "currencyCode": "USD",
                        "subtotalWithoutTaxes": 5.65,
                        "discountPercentage": 0,
                        "discountsValue": 0,
                        "subtotalIncludeDiscounts": 5.65,
                        "taxesPercentage": 15,
                        "taxValue": 15,
                        "total": 6.5,
                        "totalBeforeSale": 6.5,
                        "suggestedPrice": 0
                    },
                    "referenceTotalPrice": {
                        "currencyCode": "USD",
                        "subtotalWithoutTaxes": 5.65,
                        "discountPercentage": 0,
                        "discountsValue": 0,
                        "subtotalIncludeDiscounts": 5.65,
                        "taxesPercentage": 15,
                        "taxValue": 15,
                        "total": 6.5,
                        "totalBeforeSale": 6.5,
                        "suggestedPrice": 0
                    }
                }
            }
        ],
        "stockDetails": [
            {
                "productId": 1890,
                "stock": 0
            }
        ]
    },
    "shippingMethod": {
        "store": {
            "id": 431,
            "name": "SHOPPING DAULE",
            "code": "K078",
            "vendorId": 10,
            "vendorName": "KENTUCKY FRIED CHICKEN",
            "latitude": -1.854157,
            "longitude": -79.975704
        },
        "delivery": {
            "deliveryDate": "2024-09-11 12:14:40",
            "latitude": "",
            "longitude": "",
            "country": "ECUADOR",
            "city": "DAULE",
            "mainStreet": "GUAYAS / DAULE / EMILIANO CAICEDO MARCOS / AV VICENTE PIEDRAHITA S/N",
            "secondaryStreet": "N/A",
            "reference": "Ref: payload.client.shippingInfo.reference",
            "propertyId": 1,
            "observationsAddress": "-",
            "numberContactAddress": "0986610329",
            "zipCode": "",
            "nickName": "",
            "externalId": ""
        },
        "pickup": {
            "pickupDate": "2024-09-11 12:19:40",
            "prepDate": "2024-09-11 12:19:40",
            "prepTimeUnit": "minute",
            "prepTime": 5,
            "propertyId": 1,
            "carryOutOptions": "PickUp-LLevar"
        }
    },
    "payments": {
        "totals": [
            {
                "currencyCode": "USD",
                "subtotalWithoutTaxes": 5.65,
                "discountPercentage": 0,
                "discountsValue": 0,
                "subtotalIncludeDiscounts": 5.65,
                "taxesPercentage": 15,
                "taxValue": 0.85,
                "totalBeforeSale": 6.5,
                "total": 6.5
            },
            {
                "currencyCode": "Points",
                "subtotalWithoutTaxes": 0,
                "discountPercentage": 0,
                "discountsValue": 0,
                "subtotalIncludeDiscounts": 0,
                "taxesPercentage": 15,
                "taxValue": 0,
                "totalBeforeSale": 0,
                "total": 0
            }
        ],
        "shippingCost": [
            {
                "quantity": 0,
                "productId": 43,
                "currencyCode": "USD",
                "subtotalWithoutTaxes": 0,
                "discountPercentage": 0,
                "discountsValue": 0,
                "subtotalIncludeDiscounts": 0,
                "taxesPercentage": null,
                "taxValue": 0,
                "total": 0
            }
        ],
        "extraCharges": [
            {
                "quantity": null,
                "productId": null,
                "description": null,
                "currencyCode": null,
                "subtotalWithoutTaxes": 0,
                "discountPercentage": 0,
                "discountsValue": 0,
                "subtotalIncludeDiscounts": 0,
                "taxesPercentage": 0,
                "taxValue": 0,
                "total": 0
            }
        ],
        "taxes": [
            {
                "name": "IVA 15%",
                "currencyCode": "USD",
                "subtotalWithoutTaxes": 5.65,
                "percentage": 15,
                "total": 0.85
            }
        ],
        "discounts": [
            {
                "externalId": null,
                "name": "OFF5APP",
                "percentageDiscount": false,
                "amountDiscount": true,
                "currencyCode": "USD",
                "totalBeforeSale": 0,
                "subtotalWithoutDiscounts": 4.65,
                "percentage": 0,
                "total": 4.3478,
                "totalWithTaxes": 5
            }
        ],
        "paymentMethods": [
            {
                "processor": "DEUNA",
                "currencyCode": "USD",
                "paymentMethodCode": "93",
                "transactionType": "credit",
                "transactionId": "",
                "transactionStatus": "APPROVED",
                "tid": null,
                "mid": null,
                "authorizationCode": null,
                "voucher": null,
                "referenceNumber": null,
                "exactPayment": true,
                "customerCashAmount": "0.0000",
                "totalBill": 6.5,
                "acquirer": {
                    "code": " ",
                    "name": " "
                },
                "card": {
                    "mask": "360857XXXXXXX2001",
                    "bin": "360857",
                    "lastFourDigits": "2001",
                    "brand": "diners_club",
                    "externalCardBrandId": "D20A9503-85CF-E511-80C6-000D3A3261F3",
                    "holder": " ",
                    "cardCountry": " "
                },
                "transactionDate": {
                    "date": "2024-09-11 05:14:41",
                    "timeZoneType": 3,
                    "timeZoneName": "America/Guayaquil"
                },
                "additionalInfo": {
                    "externalPaymentMethodId": "D20A9503-85CF-E511-80C6-000D3A3261F3"
                }
            }
        ]
    },
    "marketing": {
        "loyalty": {
            "accumulation": {
                "storeCost": 0,
                "marketingCost": 0,
                "totalPoints": 0
            },
            "redemption": {
                "storeCost": 0,
                "marketingCost": 0,
                "totalPoints": 0
            },
            "accountBalancePoints": 0
        }
    },
    "additionalInfo": {
        "kiosk": {
            "ip": ""
        },
        "pickupPhase": "INITIAL_PICKUP"
    }
}
```

**Ejemplo de request:**
```bash
curl --location -g '{{server_cod}}/api/v1/order' \
--header 'Authorization: Bearer {{token}}' \
--header 'Content-Type: application/json' \
--data '{ ... }'
```

**Ejemplo de respuesta:**
```json
{
  "code": 200,
  "status": "success",
  "message": "Orden registrada exitosamente",
  "orderId": "0001292432-010103",
  "data": { ... }
}
```

## 9. Preguntas frecuentes (para IA Support)

- **¿Qué pasa si falla la validación?**  
  El paquete se descarta y se registra en logs. Revisar la fuente de datos.

- **¿Cómo sé que la transformación fue exitosa?**  
  Revisar el log del proceso de transformación.

- **¿El publicador almacena datos?**  
  No, solo orquesta flujos. La persistencia depende de los módulos de carga.

- **¿Qué hacer si la carga falla?**  
  Se activa la política de reintentos. Si persiste, revisar la interfaz de integración y los logs.

## 10. Referencias

- mxp-sync/process/readme.md (1-28)
- mxp-feBuild/process/readme.md (9-10)
- Microservicio: mxpv2.integration.cod

## 11. Contacto

- **Equipo responsable:** Integraciones MAXPOINT
- **Canal de soporte:** #mxp-integraciones (Slack interno)
- **Escalamiento:** Arquitectura de datos → Liderazgo técnico