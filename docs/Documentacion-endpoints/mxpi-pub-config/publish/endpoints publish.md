# Módulo de Configuración Pública (MXPI-pub-config)

## 1. Propósito y alcance
El módulo **MXPI-pub-config** gestiona las configuraciones públicas del ecosistema **MAXPOINT V2.0**.  
Permite consultar y validar catálogos de configuración críticos, asegurando que los servicios dependientes accedan a información consistente y actualizada.  

ℹ️ Para más detalles de los módulos relacionados, consulta:  
- MXP Init  
- MXP FE Build  
- Impresora MXP 2  

---

## 2. Descripción general del sistema
- **Microservicio:** `mxpv2.integration.pub-config`  
- **Rol principal:** Exponer endpoints para consultar configuraciones públicas.  
- **Función crítica:** Validar la estructura y disponibilidad de catálogos de configuración.  

---

## 3. Arquitectura y componentes

### 3.1. Capa de fachada
| Componente                  | Código | Descripción                                                                 |
|------------------------------|--------|-----------------------------------------------------------------------------|
| Interfaz de consulta         | P001   | Permite obtener catálogos de configuración según IP y código padre.         |

### 3.2. Capa de procesamiento
| Componente                  | Código | Descripción                                                                 |
|------------------------------|--------|-----------------------------------------------------------------------------|
| Validación de catálogos      | V020   | Verifica estructura, campos obligatorios y estado de los catálogos.         |

---

## 4. Gestión de procesos

### Flujo de trabajo estándar
1. **Consulta de catálogo** → El endpoint `P001` recibe `ip` y `fatherCode`.  
2. **Validación** → Se verifica la estructura del response y los campos de cada catálogo.  
3. **Distribución** → Se retorna al cliente la información validada en formato estandarizado.  

---

## 5. Endpoints disponibles

### 📌 Obtener catálogos de configuración
**Método:** `GET`  
**URL:** `{{server_publisher}}/P001?ip=10.23.42.43&fatherCode=config_pago`

#### Response esperado
```json
{
  "code": 20,
  "messages": [
    "Encontrado Correctamente"
  ],
  "data": {
    "catalogs": [
      {
        "code": "pp_Clave",
        "name": "Clave de Servicio de Pago",
        "value": "NCeNl5H3FRXb3A703eQVBGJzThdbVeW458tjK0K48F4=",
        "fatherCode": "config_pago",
        "detail": "",
        "status": "A",
        "ip": "10.23.63.95"
      },
      {
        "code": "pp_Usuario",
        "name": "Usuario de Servicio de Pago",
        "value": "usr_pago",
        "fatherCode": "config_pago",
        "detail": "",
        "status": "I",
        "ip": "10.23.63.95"
      }
    ]
  }
}
```

#### Validaciones principales
- **Código HTTP:** 200 o 202.  
- **Estructura del response:** Debe contener `code` (20, 200, 202), `messages` (array no vacío), `data.catalogs` (objeto con array de catálogos).  
- **Mensajes:** Cada mensaje debe contener "Encontrado".  
- **Catálogos:** Cada catálogo debe contener los campos:  
  - `code`, `name`, `value`, `fatherCode`, `detail`, `status`, `ip`.  
  - `status` debe ser `A` (activo) o `I` (inactivo).  
  - `ip` debe tener formato válido IPv4.  
  - `value` debe ser string no vacío.  

---

## 6. Puntos de integración
El microservicio **MXPI-pub-config** asegura que los catálogos de configuración sean accesibles y válidos:  
- **Entrada:** Parámetros `ip` y `fatherCode` desde aplicaciones cliente.  
- **Salida:** Catálogos verificados para consumo por servicios dependientes.  
- **Rol central:** Garantizar consistencia y disponibilidad de la configuración pública.  

---

## 7. Escenarios de uso

### ✅ Caso exitoso
- Se realiza la consulta con IP y `fatherCode` válido → catálogos retornados con código 20 → mensajes correctos → todos los campos presentes.  

### ⚠️ Posibles fallos
- **IP o fatherCode inválido:** Response con `code` distinto y `messages` indicando error.  
- **Catálogo incompleto:** Se genera log de validación con detalle de los campos faltantes.  

---

## 8. Preguntas frecuentes (para IA Support)

**Q: ¿Qué pasa si no se encuentra el fatherCode?**  
A: El response devuelve un mensaje de error y el código indica fallo.  

**Q: ¿Qué significa el estado `A` o `I` en un catálogo?**  
A: `A` = activo, `I` = inactivo.  

**Q: ¿Puedo usar este endpoint para cualquier IP?**  
A: Solo IP autorizadas en el sistema pueden obtener catálogos válidos.  

---

## 9. Referencias
- **Fuentes internas:**  
  - `mxpi-pub-config/endpoints/P001`  

- **Microservicio:** `mxpv2.integration.pub-config`  

- **Contacto:**  
  - **Equipo responsable:** Integraciones MAXPOINT  
  - **Canal de soporte:** #mxp-integraciones (Slack interno)  
  - **Escalamiento:** Arquitectura de datos → Liderazgo técnico
