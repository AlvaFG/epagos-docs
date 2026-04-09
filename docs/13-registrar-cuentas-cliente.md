# registrar_cuentas_cliente

Carga cuentas bancarias ya adheridas al organismo para Débito directo. Permite registrar hasta 100 cuentas por llamada y puede invocarse múltiples veces.

**API SOAP v2.5** | Código de respuesta prefijo: `16xxx`

---

## Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `cuentas` | Array de `CuentaClienteAgregar` | Cuentas a registrar (mínimo 1, máximo 100 por request) |

### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | El token devuelto por `obtener_token` |

### Estructura: CuentaClienteAgregar

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `identificador_cliente` | Texto | Identificador del cliente (contribuyente) a utilizar. Se usará luego para recuperar las cuentas con `obtener_cuentas_cliente` |
| `tipo_operacion` | Número entero | Identificador del tipo de impuesto al que se relacionará la cuenta. La combinación `cbu` + `identificador_cliente` + `tipo_operacion` debe ser única |
| `cuit` | Número entero | **Opcional.** CUIT del contribuyente. Si se envía, se valida contra el CBU para verificar titularidad |
| `cbu` | Texto | CBU de la cuenta bancaria (22 dígitos) |
| `fecha_adhesion` | Fecha | Fecha original de adhesión de la cuenta a Débito directo en el organismo |

---

## Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `cuentas` | Array de `CuentaClienteGenerada` | Cuentas registradas con sus identificadores |

### Estructura: CuentaClienteGenerada

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `identificador_cliente` | Texto | Identificador del cliente asignado por el organismo |
| `identificador_cuenta` | Texto | Identificador que ePagos asigna a la cuenta (usar en `solicitud_pago_recurrente_suscripcion`) |

---

## Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `16001` | ✅ Correcta | Cuentas adheridas correctamente |
| `16002` | ❌ Error | Versión inválida del protocolo |
| `16003` | ❌ Error | Error al validar el parámetro: `[_parametro_]` |
| `16004` | ❌ Error | Error al validar el token |
| `16005` | ❌ Error | Error al intentar adherir la cuenta `[_cuenta_]` |

---

## Notas

- Diferencia con el guardado de cuenta vía `solicitud_pago`: `registrar_cuentas_cliente` permite cargar cuentas en bulk desde el backend, sin que el cliente pase por un flujo de pago.
- La combinación `cbu` + `identificador_cliente` + `tipo_operacion` debe ser única para evitar duplicados.
- Si se envía `cuit`, ePagos valida que ese CUIT sea titular de la cuenta con ese CBU.
- Límite: 100 cuentas por request. Para cargas masivas, invocar el método múltiples veces.
- Una vez registradas, usar `obtener_cuentas_cliente` para recuperar los `identificador_cuenta` y usarlos en débitos recurrentes.
