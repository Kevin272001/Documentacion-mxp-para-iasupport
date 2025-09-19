---
id: fe-builder-factura
sidebar_label: Generación de Factura Electrónica
---

# Módulo de Facturación Electrónica (FE-Builder-Factura)

## 1. Propósito y alcance
El módulo **FE-Builder-Factura** se encarga de la generación de facturas electrónicas en el sistema MAXPOINT V2.0.  
Permite la creación de documentos electrónicos válidos según las normativas tributarias, con soporte para diferentes tipos de documentos y formatos.  
Integra la información de ventas con los sistemas de facturación electrónica autorizados.

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- Gestión de Clientes  
- Puntos de Venta  
- Catálogo de Productos  

## 2. Descripción general del sistema
- **Microservicio:** mxpv2.fe.builder  
- **Rol principal:** generación de documentos electrónicos  
- **Función crítica:** cumplimiento normativo tributario  

## 3. Arquitectura y componentes

### 3.1 Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz REST Factura | FE-FACT-001 | Punto de entrada para solicitudes de generación de facturas. Valida y procesa la información de la transacción. |

### 3.2 Tipos de documentos soportados
- `FACTURA-ECU`: Factura electrónica de venta
- `NOTA_CREDITO`: Nota de crédito electrónica
- `NOTA_DEBITO`: Nota de débito electrónica
- `GUIA_REMISION`: Guía de remisión electrónica

## 4. Gestión de procesos
### Flujo de trabajo estándar
1. **Recepción** → Se recibe la solicitud POST en `/api/v1/transaction/TX003`.  
2. **Validación** → Se verifica la estructura y los campos obligatorios.  
3. **Procesamiento** → Se genera el documento electrónico según normativa.  
4. **Autorización** → Se envía al SRI para su autorización.  
5. **Respuesta** → Se devuelve el documento electrónico generado.  

## 5. Puntos de integración
- **Entrada:** transacciones de puntos de venta.  
- **Salida:** documentos electrónicos autorizados.  
- **Rol central:** garantizar la generación de documentos electrónicos válidos.  

## 6. Patrones de operación
- **Validación estricta:** todos los campos obligatorios según normativa.  
- **Tolerancia a fallos:** reintentos automáticos en caso de error.  
- **Trazabilidad:** registro detallado de cada operación.  

## 7. Escenarios de uso
✅ **Caso exitoso:**  
Solicitud POST con datos correctos → validación → generación → autorización → respuesta con documento electrónico.  

⚠️ **Posibles fallos:**  
- Datos incompletos → error de validación.  
- Error en servicio SRI → reintento automático.  
- Clave de acceso duplicada → generación de nueva clave.  

## 8. Endpoints
### Generar Factura Electrónica
- **Método:** POST  
- **URL:** `/api/v1/transaction/TX003`  
- **Headers:**
  ```http
  Authorization: Bearer {{token}}
  Content-Type: application/json
  ```
- **Body:**
```json
{
    "type_document": "FACTURA-ECU",
    "transaction": {
        "transaction_date": "2025-07-17T15:33:20.123",
        "point_of_sale": {
            "cdn_id": "bc2d8ada-e25e-1bb7-55fe-32d1205ac4af",
            "restaurant_id": "e17d03da-b8b6-f424-febc-3a767b6401bb",
            "cash_register_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "cash_register_name": "EC-CAJA001",
            "user_seller_id": "7421A2EE-B13B-EA11-80EA-000D3A019254",
            "user_seller_name": "Daniel Llerena",
            "device_os_name": "ios",
            "source_app_name": "Maxpoint 2.0",
            "half_integration_id": 3,
            "half_integration_name": "App"
        },
        "client": {
            "client_id": "030B9503-85CF-E511-80C6-000D3A3261F3",
            "name": "María José",
            "last_name": "Hernández Zurita",
            "phone": "+593986610329",
            "email": "mari_jhz@hotmail.com",
            "gov_id_type": "CI",
            "gov_id_number": "0929691038",
            "external_id": "030B9503-85CF-E511-80C6-000D3A3261F3",
            "additional_info": {
                "birthdate": ""
            }
        },
        "order": {
            "order_id": "0001292432-010103",
            "createdAt": "2025-05-15 13:53:40",
            "products": [
                {
                    "plu_id": "401",
                    "plu_name": "COMBO IDEAL",
                    "plu_quantity": 1,
                    "price": {
                        "total_per_plu": {
                            "currency_code": "USD",
                            "subtotal_without_taxes": 5.65,
                            "tax_value": 0.85,
                            "total": 6.5,
                            "taxes": [
                                {
                                    "tax_name": "IVA 15%",
                                    "tax_code": "iva23313",
                                    "tax_percentage": 15,
                                    "total_tax_vulue": 0.85,
                                    "base_amount": 5.65
                                }
                            ]
                        }
                    }
                }
            ]
        },
        "payments": {
            "payment_methods": [
                {
                    "payment_methods_id": "D20A9503-85CF-E511-80C6-000D3A3261F3",
                    "processor": "DEUNA",
                    "currency_code": "USD",
                    "payment_method_code": "93",
                    "transaction_type": "OTROS CON UTILIZACIÓN DEL SISTEMA FINANCIERO",
                    "transaction_status": "APPROVED",
                    "total_bill": 6.5
                }
            ]
        }
    }
}
```

### Respuesta Exitosa (200) - Factura Generada
```json
{
    "code": 200,
    "status": "success",
    "alert": "Integración completada exitosamente",
    "messages": [
        "Integración completada exitosamente"
    ],
    "data": {
        "companyTaxData": {
            "name": "INT FOOD SERVICES CORP SA",
            "identification_number": "1234567890001",
            "special_taxpayer": "GRAN CONTRIBUYENTE",
            "isSpecial_taxpayer": "Sí",
            "special_taxpayer_code": "RESOL. N°: NAC-GCFOIOC21-00000900-E",
            "type_tax": "3243fdw23",
            "obligatory_to_keep_accounts": "Sí",
            "resolution_code": "155",
            "company_address": "Av. Amazonas y Naciones Unidas, Quito, Ecuador",
            "restaurnt_address": "PICHINCHA / QUITO / AV. AMAZONAS 6121 Y EL INCA",
            "establishment_code": "001",
            "point_of_emission_code": "001"
        },
        "dataInvoice": {
            "sequential": "050-070-000470031",
            "transaction": "K024F001529034",
            "access_key": "091020240117914151320011024050000470020412615331",
            "legend": "Estimado cliente: Por favor verifique los datos de su factura, únicamente se aceptarán cambios el mismo día de emisión.",
            "link_label": "Para obtener su factura electrónica ingrese a:",
            "url_document": "https://facturacion.kfc.ec",
            "access_key_label": "(Usuario: CI/RUC, Clave: CI/RUC) o a la página web del SRI con la Clave de Acceso:",
            "created_at": "2024-09-11 12:14:40"
        }
    }
}
```

### Respuesta de Error (400) - Validación Fallida
```json
{
    "code": 400,
    "status": "failed",
    "alert": "Error en la validación de datos",
    "messages": [
        "El campo 'gov_id_number' es requerido",
        "El campo 'email' no tiene un formato válido"
    ],
    "data": {}
}
```

### Respuesta de Error (500) - Error del Servidor
```json
{
    "code": 500,
    "status": "error",
    "alert": "Error en el servidor",
    "messages": [
        "No se pudo conectar con el servicio de facturación"
    ]
}
```

## 9. Preguntas frecuentes (Soporte IA)
Q: ¿Qué hago si recibo un error de "Clave de Acceso ya utilizada"?  
A: Este error ocurre cuando se intenta generar una factura con una clave de acceso que ya existe. El sistema debe generar automáticamente una nueva clave de acceso y reintentar el proceso.

Q: ¿Cuánto tiempo debo esperar para obtener la autorización del SRI?  
A: Normalmente la respuesta es inmediata, pero en casos de alta demanda puede tomar hasta 24 horas. Se recomienda implementar un sistema de reintentos con backoff exponencial.

Q: ¿Qué debo hacer si el cliente no proporciona su correo electrónico?  
A: El correo electrónico es obligatorio para la emisión de facturas electrónicas. Se debe solicitar al cliente o generar un correo genérico con el RUC/CI del cliente.

## 10. Referencias
- Reglamento para la Aplicación de la Ley Orgánica de Régimen Tributario Interno  
- Manual de Facturación Electrónica del SRI  
- Especificaciones técnicas de documentos electrónicos  

## 11. Contacto
- **Equipo responsable:** Facturación Electrónica  
- **Canal de soporte:** `#soporte-facturacion` (Slack interno)  
- **Escalamiento:** Líder de Facturación → Gerente de TI
