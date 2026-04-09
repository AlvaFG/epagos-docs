# QR Interoperable: obtener_cajas_qr + generar_orden_qr

Métodos para el flujo de cobro mediante QR interoperable estático con órdenes de pago dinámicas (Transferencias 3.0 / T3.0).

**API SOAP v2.5** | Código de respuesta prefijo: `18xxx`

---

## obtener_cajas_qr

Lista las cajas de cobro configuradas para el organismo, con sus QRs interoperables estáticos.

### Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |

#### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | El token devuelto por `obtener_token` |

### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `cajas` | Array de `CajasOrganismo` | Cajas configuradas para el organismo |

#### Estructura: CajasOrganismo

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_caja` | Numérico entero | Número ID que identifica la caja dentro del organismo |
| `nombre_caja` | Texto | Nombre o descripción de la caja |
| `tipo_caja` | Set | Tipo de caja: `F` = Física, `V` = Virtual |
| `monto_maximo_caja` | Numérico decimal | Importe máximo por operación (si está definido) |

### Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `18002` | ✅ Correcta | Listado de cajas informado correctamente |
| `18003` | ❌ Error | Error al validar el token |
| `18011` | ❌ Error | El organismo no tiene cajas configuradas |

---

## generar_orden_qr

Genera una orden de cobro (T3.0) para una caja específica o para una operación existente. El cliente paga escaneando el QR estático de la caja, que carga la orden generada.

### Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `orden` | Array de `Orden` | Datos de la orden de cobro |

#### Estructura: Orden

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_caja` | Numérico entero | **Opcional.** ID de la caja destino. Si se envía `id_transaccion`, este campo se ignora |
| `id_transaccion` | Numérico entero | **Opcional.** ID de operación T3.0 existente donde se enviará la orden. Si se envía, `id_caja` se ignora |
| `importe` | Numérico decimal | Importe de la operación (con 2 decimales, mayor a 0) |
| `concepto` | Texto (150) | **Opcional.** Descripción de la operación |
| `identificador_2` | Texto (100) | **Opcional.** Identificador adicional para uso del cliente |
| `identificador_3` | Texto (512) | **Opcional.** Identificador adicional para uso del cliente |
| `identificador_4` | Texto (65.535) | **Opcional.** Identificador adicional para uso del cliente |
| `email_pagador` | Texto (255) | **Opcional.** Correo del cliente pagador |
| `detalle_orden` | Texto (255) | **Opcional.** Detalle de la operación |
| `vencimiento` | Fecha (`AAAA-MM-DD`) | **Opcional.** Fecha de vencimiento de la orden |

### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |

### Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `18001` | ✅ Correcta | Operación generada con éxito |
| `18003` | ❌ Error | Error al validar el token |
| `18004` | ❌ Error | El importe supera el monto máximo configurado para la caja |
| `18005` | ❌ Error | Error al validar el parámetro: `[_parametro_]` |
| `18006` | ❌ Error | La fecha de vencimiento debe ser igual o posterior a la fecha actual |
| `18007` | ❌ Error | Formato de fecha erróneo. Debe ser `AAAA-MM-DD` |
| `18008` | ❌ Error | Error interno al intentar generar la orden QR T3.0 |
| `18009` | ❌ Error | La caja indicada no es válida |
| `18010` | ❌ Error | El importe debe ser mayor a cero |
| `18011` | ❌ Error | El organismo no tiene cajas configuradas |
| `18012` | ❌ Error | La operación indicada no es válida |

---

## Flujo típico

```
1. obtener_cajas_qr → listar cajas disponibles con sus id_caja
2. [Imprimir QR estático de cada caja en los puntos de cobro físicos o en la web]
3. Cuando el cliente va a pagar:
   → generar_orden_qr (con id_caja + importe + datos de la operación)
   → La orden queda disponible en la caja para que el cliente la pague escaneando el QR
4. El cliente abre su billetera (MODO, MercadoPago, etc.), escanea el QR estático
   → Ve la orden generada y confirma el pago
5. ePagos notifica el pago por webhook
6. Conciliar con obtener_pagos
```

## Notas

- El QR de la caja es **estático** (no cambia). Lo dinámico es la orden generada para cada cobro.
- Los campos `identificador_2`, `identificador_3`, `identificador_4` son útiles para asociar la orden a datos internos del sistema (número de factura, ID de pedido, etc.).
- Si se define `vencimiento`, la orden no podrá pagarse después de esa fecha.
- La diferencia entre caja `Física` (F) y `Virtual` (V) la define el organismo según su caso de uso.
