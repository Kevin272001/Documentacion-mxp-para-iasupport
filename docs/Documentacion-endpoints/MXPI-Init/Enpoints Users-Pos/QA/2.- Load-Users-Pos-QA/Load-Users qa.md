# Módulo de Carga de Usuarios POS (MXPi-init 2.- Load-Users-Pos-QA)

## Propósito y alcance
El módulo **Load-Users-Pos-QA** es un componente especializado del ecosistema MAXPOINT V2.0 diseñado para la extracción, transformación y carga (ETL) de datos de usuarios POS desde sistemas legacy hacia el entorno de QA. Su función principal es garantizar la migración controlada y homologación de usuarios con diferentes perfiles (Administrador Local, Cajero, Mesero) desde bases de datos SQL Server hacia MongoDB en el entorno de calidad.

## Descripción general del sistema
**Microservicio**: mxpv2.integration.init.load-users-pos-qa  
**Rol principal**: Ejecutar procesos ETL para migración de usuarios POS a entorno QA  
**Función crítica**: Asegurar la correcta homologación de identificadores entre sistemas legacy y V2, manteniendo la integridad referencial de los datos.

## Arquitectura y componentes

### Capa de fachada
| Componente | Código | Descripción |
|------------|--------|-------------|
| Interfaz de extracción POS | F100 | Punto de entrada principal para solicitudes de carga de usuarios POS |
| 🔑 El F100 actúa como orquestador del proceso ETL completo, coordinando extracciones, transformaciones y cargas |

### Capa de extracción (EXTRACTS)
| Código | Descripción | Origen |
|--------|-------------|---------|
| E001 | Extrae usuarios con perfil "Administrador Local" | SQL Server (SP) |
| E002 | Extrae usuarios con perfil "Cajero" | SQL Server (SP) |
| E003 | Extrae usuarios con perfil "Mesero" | SQL Server (SP) |
| E004 | Extrae homologaciones de Ciudad y Perfil_Pos | MongoDB QA |
| E005 | Extrae homologaciones de usuarios POS | MongoDB QA |
| E006 | Extrae homologaciones de restaurantes | MongoDB QA |
| E007 | Extrae homologaciones de cadenas | MongoDB QA |
| E008 | Extrae homologaciones de países | MongoDB QA |

### Capa de transformación (TRANSFORMS)
| Código | Descripción |
|--------|-------------|
| T120 | Transforma y homologa datos de usuarios POS utilizando las tablas de homologación |

### Capa de carga (LOADS)
| Código | Descripción | Destino |
|--------|-------------|---------|
| L001 | Carga usuarios POS transformados | MongoDB QA (Settings_BO_Users_Pos) |

## Gestión de procesos

### Flujo de trabajo estándar
1. **Invocación** → Se recibe solicitud en F100 con parámetros de configuración
2. **Extracción múltiple** → Ejecución paralela de extracciones E001-E008
3. **Homologación** → Transformación T120 utilizando tablas de homologación
4. **Validación** → Verificación de integridad referencial y consistencia de datos
5. **Carga** → Inserción/actualización en colección Settings_BO_Users_Pos
6. **Confirmación** → Retorno de resultados con estadísticas del proceso

### Parámetros de configuración requeridos
```json
{
    "server_init": "URL del servicio INIT",
    "token": "Token de autenticación",
    "server_legacy": "Servidor SQL Legacy",
    "user_legacy": "Usuario SQL Legacy",
    "pass_legacy": "Password SQL Legacy", 
    "repository_legacy": "Repositorio Legacy",
    "server_mongo_qa": "Servidor MongoDB QA",
    "user_mongo_qa": "Usuario MongoDB QA",
    "pass_mongo_qa": "Password MongoDB QA",
    "legacy_cdn_id": "ID de cadena en sistema legacy",
    "legacy_country_id": "ID de país en sistema legacy",
    "country": "Código de país (ej: 'MX')"
}
```

## Estructura de datos de salida

La carga resultante en la colección `Settings_BO_Users_Pos` sigue esta estructura:

```json
{
    "_id": "uuid",
    "country": {
        "country_id": "uuid",
        "country": "string"
    },
    "user_pos": {
        "first_name": "string",
        "last_name": "string",
        "email": "string",
        "phone": "string",
        "address": "string",
        "birth_date": "date",
        "user": "string",
        "password": "string",
        "identification": "string"
    },
    "user_pos_profiles": [
        {
            "profile_id": "uuid",
            "profile_name": "string",
            "franchises": [
                {
                    "franchise_id": "uuid",
                    "franchise_name": "string",
                    "restaurants": [
                        {
                            "restaurant_id": "uuid",
                            "restaurant_name": "string",
                            "address": "string",
                            "phone": "string",
                            "city_id": "uuid"
                        }
                    ]
                }
            ]
        }
    ],
    "IsActive": "boolean"
}
```

## Puntos de integración

El módulo se integra con:

- **Sistemas Legacy**: SQL Server con stored procedures específicos
- **Homologación V1-V2**: MongoDB QA con colección de homologación
- **Settings POS**: MongoDB QA para almacenamiento final de usuarios
- **Servicios INIT**: Para autenticación y coordinación de procesos

## Patrones de procesamiento

1. **Extracción por perfil**: Separación de usuarios por tipo de perfil (E001-E003)
2. **Homologación centralizada**: Uso de tablas de equivalencia ID V1 → ID V2 (E004-E008)
3. **Transformación especializada**: Módulo T120 para conversión de estructuras
4. **Carga documental**: Inserción en MongoDB con estructura documental optimizada

## Escenarios de uso

### ✅ Caso exitoso
1. Solicitud válida a F100 con parámetros correctos
2. Extracciones E001-E008 completadas exitosamente
3. Transformación T120 con homologación completa
4. Carga L001 en MongoDB con todos los usuarios procesados
5. Retorno de estadísticas de procesamiento

### ⚠️ Posibles fallos

**Error en extracción Legacy**:
- Fallo en conexión SQL Server
- Stored procedure no disponible
- Datos inconsistentes en legacy

**Error en homologación**:
- IDs sin equivalencia en tablas de homologación
- Estructuras de datos incompatibles

**Error en carga**:
- Conexión MongoDB no disponible
- Violación de restricciones de schema

**Acciones recomendadas**:
- Verificar conectividad con sistemas origen
- Validar existencia de tablas de homologación
- Revisar logs de transformación T120
- Confirmar permisos de escritura en MongoDB QA

## Preguntas frecuentes (para IA Support)

**Q: ¿Qué hacer si falla la extracción de usuarios?**  
A: Verificar que los stored procedures `mxp_v2.USP_UsuariosPos_JSON` existan y sean accesibles con las credenciales proporcionadas.

**Q: ¿Cómo validar que la homologación fue correcta?**  
A: Revisar los logs de la transformación T120 que debe indicar el porcentaje de IDs homologados exitosamente.

**Q: ¿Qué significa "ID sin homologación"?**  
A: Indica que un ID del sistema legacy no tiene equivalencia en la tabla de homologación V1-V2. Se debe ejecutar el proceso de homologación primero.

**Q: ¿El proceso sobreescribe usuarios existentes?**  
A: Sí, el proceso upsert basado en el `_id` que corresponde al `user_id` homologado.

**Q: ¿Cómo verificar el resultado de la carga?**  
A: Consultar la colección `Settings_BO_Users_Pos` en MongoDB QA y validar conteos con el sistema legacy.

## Referencias

- mxp-init/process/load-users-pos.md
- mxp-homologation/process/readme.md
- mxp-settings/schema/users-pos-schema.v2.json

## Contacto

**Equipo responsable**: Migración POS - MAXPOINT  
**Canal de soporte**: #mxp-pos-migration (Slack interno)  
**Escalamiento**: Líder de migración → Arquitecto de datos