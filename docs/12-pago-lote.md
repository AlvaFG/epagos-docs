# pago_lote

Marca como pagadas un conjunto de operaciones vinculándolas a una operación base ya acreditada. Las operaciones vinculadas quedan dentro del lote de acreditación de la operación original.

> ⚠️ **Este método no está disponible para todos los organismos.** Consultar con el ejecutivo comercial de ePagos.

**API SOAP v2.5** | Código de respuesta prefijo: `15xxx`

---

## Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `pago_lote` | Estructura `PagoLote` | Datos de las operaciones a acreditar y la operación base |

### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | El token devuelto por `obtener_token` |

### Estructura: PagoLote

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_transaccion` | Numérico entero | Identificador de la operación base ya acreditada (y **aún no rendida**) |
| `forma_pago` | Numérico entero | Identificador de la forma de pago a asignar a las operaciones a acreditar |
| `fecha_pago` | Fecha | Fecha en que deben marcarse como pagadas las operaciones |
| `importe` | Numérico decimal | Monto total de todas las operaciones a acreditar |
| `operaciones` | Array de `OperacionPagoLote` | Detalle de cada operación a acreditar |

### Estructura: OperacionPagoLote

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_transaccion` | Numérico entero | Identificador ePagos de la operación a acreditar |
| `importe` | Numérico decimal | Monto de la operación. **La suma debe coincidir con `importe` de `PagoLote`** |

---

## Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |

---

## Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `15001` | ✅ Correcta | El pago del lote fue aceptado |
| `15002` | ❌ Error | Método no autorizado para su organismo |
| `15003` | ❌ Error | Datos inválidos de lote recibidos |
| `15004` | ❌ Error | Error al validar el parámetro: `[_parametro_]` |
| `15005` | ❌ Error | Error al validar el token |

---

## Notas

- La operación base (`id_transaccion` de `PagoLote`) debe estar **acreditada pero no rendida** aún.
- La suma de los `importe` de cada `OperacionPagoLote` debe ser exactamente igual al `importe` total de `PagoLote`.
- Este método es útil para organismos que reciben pagos externos (ej. cajas recaudadoras propias) y necesitan informar a ePagos que un conjunto de operaciones fue cobrado bajo un pago consolidado.
