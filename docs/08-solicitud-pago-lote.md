# solicitud_pago_lote

Permite generar múltiples operaciones de pago en una sola llamada. Equivale a invocar `solicitud_pago` varias veces pero en un único request. **Límite: 50 operaciones por lote.**

**API SOAP v2.5** | Código de respuesta prefijo: `08xxx`

---

## Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `tipo_operacion` | Set | Tipo de operación. Valor fijo `"op_pago"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `lote` | Estructura `ArrayLote` | Vector de `TipoLote` con el detalle de cada operación |

### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | El token devuelto por `obtener_token` |

### Estructura: TipoLote

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `fp` | Array de `DatosFormaPago` | Datos de la forma de pago de la operación |
| `operacion` | Estructura `DatosOperacionPago` | Datos de la operación (igual que en `solicitud_pago`) |
| `convenio` | Numérico entero | Número de convenio asignado por ePagos |

---

## Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta general del lote |
| `respuesta` | Texto | Descripción de la respuesta |
| `token` | Texto | Token con el que se hizo la consulta |
| `id_organismo` | Numérico entero | Código de organismo |
| `lote` | Array de `RespuestaLote` | Detalle de cada operación generada |

### Estructura: RespuestaLote

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_transaccion` | Numérico entero | Número único de la operación generada |
| `numero_operacion` | Texto | Número de operación informado por el cliente |
| `convenio` | Numérico entero | Número de convenio informado por ePagos |
| `respuesta_forma_pago_array` | Array de `RespuestaFormaPago` | Detalles de la operación generada |

---

## Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `08001` | ✅ Correcta | Pagos en lotes procesados |
| `08002` | ❌ Error | Error al validar el token |
| `08003` | ❌ Error | Error interno al intentar procesar los pagos |
| `08004` | ❌ Error | Error al validar el parámetro: `[_parametro_]` |
| `08005` | ❌ Error | El tamaño del lote supera el límite permitido de `[_limite_]` |
| `08006` | ❌ Error | Error al procesar pagos |
| `08007` | ❌ Error | No coinciden los montos y los detalles |
| `08008` | ❌ Error | Versión inválida del protocolo |

---

## Notas

- Límite máximo: **50 operaciones por request**. Para volúmenes mayores, invocar el método múltiples veces.
- Cada elemento del array `lote` en la respuesta corresponde posicionalmente al elemento enviado en el request.
- Ver `emision_masiva_epagos.md` para el flujo completo de emisión masiva con archivos de lote.
