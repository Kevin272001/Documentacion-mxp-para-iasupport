# Módulo de Carga de Usuarios POS (MXPi-init Users-Pos DEV 2.- Load-Users-Pos-DEV)

## Propósito y alcance

El módulo **Load-Users-Pos-DEV** es un componente especializado dentro del ecosistema MAXPOINT V2.0 diseñado para la extracción, transformación y carga (ETL) de datos de usuarios POS desde sistemas legacy hacia la nueva plataforma. Este módulo gestiona específicamente la migración de usuarios con diferentes perfiles (Administrador Local, Cajero, Mesero) y su homologación a la estructura de datos V2.

## Descripción general del sistema

**Microservicio**: mxpv2.integration.init.users-pos  
**Rol principal**: Ejecutar procesos ETL para la migración de usuarios POS desde sistemas legacy  
**Función crítica**: Garantizar la correcta extracción, transformación y carga de datos de usuarios con preservación de las relaciones y homologación de identificadores.

## Arquitectura y componentes

### 1. Capa de extracción (Extract)

| Componente | Código | Descripción |
|------------|--------|-------------|
| Extracción usuarios Admin Local | E001 | Extrae usuarios con perfil "Administrador Local" desde SQL Server |
| Extracción usuarios Cajero | E002 | Extrae usuarios con perfil "Cajero" desde SQL Server |
| Extracción usuarios Mesero | E003 | Extrae usuarios con perfil "Mesero" desde SQL Server |
| Homologación ciudades y perfiles | E004 | Obtiene homologación de IDs para ciudades y perfiles POS desde MongoDB |
| Homologación usuarios | E005 | Obtiene homologación de IDs para usuarios desde MongoDB |
| Homologación restaurantes | E006 | Obtiene homologación de IDs para restaurantes desde MongoDB |
| Homologación cadenas | E007 | Obtiene homologación de IDs para cadenas desde MongoDB |
| Homologación países | E008 | Obtiene homologación de IDs para países desde MongoDB |

### 2. Capa de transformación (Transform)

| Componente | Código | Descripción |
|------------|--------|-------------|
| Transformación usuarios POS | T120 | Procesa y transforma los datos de usuarios POS para la estructura destino |

### 3. Capa de carga (Load)

| Componente | Código | Descripción |
|------------|--------|-------------|
| Carga usuarios POS | L001 | Inserta los datos transformados en la colección MongoDB destino |

## Gestión de procesos

### Flujo de trabajo estándar

1. **Extracción múltiple**: 
   - Se ejecutan simultáneamente las extracciones E001-E003 desde SQL Server
   - Se ejecutan simultáneamente las homologaciones E004-E008 desde MongoDB

2. **Transformación**:
   - Los datos extraídos se procesan con el transformador T120
   - Se aplican las homologaciones de IDs correspondientes
   - Se estructura la información según el schema destino

3. **Carga**:
   - Los datos transformados se insertan en la colección `Settings_BO_Users_Pos`
   - Se preservan las relaciones jerárquicas (país → cadena → restaurante → usuario)

## Puntos de integración

El módulo se integra con:

- **Sistemas legacy**: SQL Server mediante stored procedures
- **Base de datos de homologación**: MongoDB con la colección `homologation_v1_v2`
- **Base de datos destino**: MongoDB con la colección `DEV_KFC_MXPi_Settings`

## Configuraciones técnicas

### Conexiones utilizadas

```json
{
  "sql_server": {
    "adapter": "SqlServerSP",
    "repository": "{{repository_legacy}}",
    "stored_procedure": "mxp_v2.USP_UsuariosPos_JSON"
  },
  "mongo_homologation": {
    "adapter": "Mongo",
    "repository": "homologation_v1_v2",
    "collection": "uuids"
  },
  "mongo_destination": {
    "adapter": "Mongo",
    "repository": "DEV_KFC_MXPi_Settings",
    "collection": "Settings_BO_Users_Pos"
  }
}
```

### Estructura de destino

La carga final sigue este schema:

```javascript
{
  "_id": "uuid", // ID único del usuario
  "country": {
    "country_id": "uuid", // ID homologado del país
    "country": "string" // Nombre del país
  },
  "user_pos": {
    "first_name": "string",
    "last_name": "string",
    "email": "string",
    "phone": "string",
    "address": "string",
    "birth_date": "date",
    "user": "string", // Usuario/login
    "password": "string",
    "identification": "string" // Identificación/documento
  },
  "user_pos_profiles": [
    {
      "profile_id": "uuid", // ID homologado del perfil
      "profile_name": "string", // Nombre del perfil
      "franchises": [
        {
          "franchise_id": "uuid", // ID homologado de la cadena
          "franchise_name": "string", // Nombre de la cadena
          "restaurants": [
            {
              "restaurant_id": "uuid", // ID homologado del restaurante
              "restaurant_name": "string", // Nombre del restaurante
              "address": "string", // Dirección del restaurante
              "phone": "string", // Teléfono del restaurante
              "city_id": "uuid" // ID homologado de la ciudad
            }
          ]
        }
      ]
    }
  ],
  "IsActive": true // Estado activo/inactivo
}
```

## Parámetros requeridos

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| {{server_init}} | URL del servidor INIT | https://init.mxpoint.com |
| {{token}} | Token de autenticación Bearer | eyJhbGciOiJIUzI1NiIs... |
| {{server_legacy}} | Servidor SQL Server legacy | sqlserver.legacy.com |
| {{user_legacy}} | Usuario SQL Server | user_legacy |
| {{pass_legacy}} | Password SQL Server | password_legacy |
| {{repository_legacy}} | Base de datos legacy | LegacyDB |
| {{server_mongo}} | Servidor MongoDB | mongodb.server.com |
| {{user_mongo}} | Usuario MongoDB | user_mongo |
| {{pass_mongo}} | Password MongoDB | password_mongo |
| {{country}} | Código de país | MX, CO, AR |
| {{legacy_cdn_id}} | ID de cadena en sistema legacy | 123 |
| {{legacy_country_id}} | ID de país en sistema legacy | 1 |

## Escenarios de uso

### ✅ Caso exitoso
1. Extracción exitosa desde SQL Server y MongoDB
2. Transformación correcta de datos con homologación de IDs
3. Carga completa en la base de datos destino

### ⚠️ Posibles fallos
- **Error de conexión SQL Server**: Falla extracción de usuarios
- **Error de conexión MongoDB**: Falla homologación o carga final
- **Homologación incompleta**: Falta de IDs homologados para alguna entidad
- **Error de validación**: Datos inconsistentes o incompletos

## Políticas de reintento

- Reintentos automáticos en fallos de conexión (3 intentos)
- Log detallado de errores con identificación del paso fallido
- Alertas al sistema de monitoreo en fallos persistentes

## Preguntas frecuentes (para IA Support)

**Q: ¿Qué hacer si falla la extracción desde SQL Server?**  
A: Verificar conectividad con el servidor legacy y validar credenciales.

**Q: ¿Cómo verificar que la homologación fue exitosa?**  
A: Revisar los logs de los extractores E004-E008 y validar que se obtuvieron todos los IDs necesarios.

**Q: ¿Qué pasa si falta algún ID en la homologación?**  
A: El proceso de transformación (T120) registrará un error y no procederá con la carga.

**Q: ¿Dónde se registran los errores del proceso?**  
A: En los logs del microservicio mxpv2.integration.init.users-pos con identificador de syncId.

## Referencias

- Documentación técnica: mxp-init/process/users-pos/readme.md
- Esquemas de base de datos: mxp-schemas/users-pos/v2.md
- API de homologación: mxp-homologation/api/v1/readme.md

## Contacto

**Equipo responsable**: Migración de Datos MAXPOINT  
**Canal de soporte**: #mxp-migracion-datos (Slack interno)  
**Escalamiento**: Líder de Migración → Arquitecto de Datos