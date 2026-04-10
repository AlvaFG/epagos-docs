# API v2.1+ — Métodos exclusivos (no disponibles en v1.0/v2.0)

> Estos 5 métodos fueron introducidos en v2.1 y están disponibles en v2.1, v2.4 y v2.5.
> NO existen en v1.0 ni v2.0.

---

## 14. pago_lote

Ejecuta el pago de un lote previamente generado con `solicitud_pago_lote`.

**Parámetros:**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | `"2.0"` |
| `credenciales` | Estructura | `id_organismo`, `token` |
| Datos del lote | Estructura | ID del lote a ejecutar |

> Nota: En v2.5, este método puede ser reemplazado automáticamente usando el campo `opc_operaciones_lote = true` en E-Checkout.

---

## 15. registrar_cuentas_cliente

Registra cuentas bancarias de clientes para débito automático.

**Parámetros:**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | `"2.0"` |
| `credenciales` | Estructura | `id_organismo`, `token` |
| Datos de la cuenta | Estructura | CBU, datos del titular, etc. |

---

## 16. obtener_resultados_debito

Consulta los resultados de débitos automáticos procesados.

**Parámetros:**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | `"2.0"` |
| `credenciales` | Estructura | `id_organismo`, `token` |
| Filtros | Estructura | Rango de fechas, estado |

---

## 17. obtener_cajas_qr

Lista las cajas configuradas para cobro con QRs interoperables estáticos.

**Parámetros:**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | `"2.0"` |
| `credenciales` | Estructura | `id_organismo`, `token` |

**Respuesta:**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | int | Código de respuesta |
| `respuesta` | Texto | Descripción |
| `cajas` | Array | Estructura CajasOrganismo |

**CajasOrganismo:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id_caja` | int | ID de la caja dentro del organismo |
| `nombre_caja` | Texto | Nombre o descripción |
| `tipo_caja` | Set | `F` (Física) o `V` (Virtual) |
| `monto_maximo_caja` | decimal | Importe máximo por operación (si definido) |

| Código | Tipo | Descripción |
|--------|------|-------------|
| 18002 | OK | Listado informado correctamente |
| 18003 | Error | Error al validar token |
| 18011 | Error | Organismo sin cajas configuradas |

---

## 18. generar_orden_qr

Genera una orden de pago asociada a una caja QR.

**Parámetros:**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | `"2.0"` |
| `credenciales` | Estructura | `id_organismo`, `token` |
| Datos de la orden | Estructura | `id_caja`, monto, datos del pagador |

---

## Resumen de disponibilidad

| Método | v1.0 | v2.0 | v2.1 | v2.4 | v2.5 |
|--------|:----:|:----:|:----:|:----:|:----:|
| `pago_lote` | ❌ | ❌ | ✅ | ✅ | ✅ |
| `registrar_cuentas_cliente` | ❌ | ❌ | ✅ | ✅ | ✅ |
| `obtener_resultados_debito` | ❌ | ❌ | ✅ | ✅ | ✅ |
| `obtener_cajas_qr` | ❌ | ❌ | ✅ | ✅ | ✅ |
| `generar_orden_qr` | ❌ | ❌ | ✅ | ✅ | ✅ |
