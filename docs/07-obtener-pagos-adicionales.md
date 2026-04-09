# obtener_pagos_adicionales

Lista los pagos adicionales realizados por usuarios sobre operaciones ya acreditadas, dentro de un rango de fechas.

**API SOAP v2.5** | Código de respuesta prefijo: `07xxx`

---

## Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `pagos` | Estructura `DatosPagosAdicionales` | Criterios de búsqueda |

### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | El token devuelto por `obtener_token` |

### Estructura: DatosPagosAdicionales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `fecha_desde` | Fecha (`AAAA-MM-DD`) | Fecha de inicio de búsqueda (fecha novedad) |
| `fecha_hasta` | Fecha (`AAAA-MM-DD`) | Fecha de fin de búsqueda (fecha novedad) |

---

## Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `token` | Texto | Token con el que se hizo la consulta |
| `id_organismo` | Numérico entero | Código de organismo |
| `pagos_adicionales` | Estructura `PagosAdicionalesRespuestaArray` | Vector con los pagos adicionales encontrados |

### Estructura: PagosAdicionales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `CodigoUnicoTransaccion` | Numérico entero | Identificador del pago en ePagos |
| `FormaPago` | Texto | Nombre de la forma de pago utilizada |
| `Monto` | Numérico decimal | Monto del pago adicional |
| `FechaPago` | Fecha | Fecha de acreditación del pago |
| `FechaNovedad` | Fecha | Fecha en que se registra la novedad del pago |
| `IdPago` | Numérico entero | Identificador del pago adicional (se incrementa por cada pago adicional del sistema) |

---

## Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `07001` | ✅ Correcta | Pagos adicionales devueltos |
| `07002` | ❌ Error | Error al validar el token |
| `07003` | ❌ Error | Error interno al intentar devolver los pagos adicionales |
| `07004` | ❌ Error | El rango de fechas no es correcto |
| `07005` | ❌ Error | Error al validar el parámetro: `[_parametro_]` |
| `07006` | ❌ Error | Versión inválida del protocolo |

---

## Notas

- Un "pago adicional" es un pago voluntario extra que realiza un usuario sobre una operación ya acreditada (ej. propinas, cargos adicionales).
- El campo `FechaNovedad` es la fecha de registro del evento en el sistema, distinta de `FechaPago` (fecha de acreditación real).
- Útil para conciliación de pagos parciales o complementarios.
