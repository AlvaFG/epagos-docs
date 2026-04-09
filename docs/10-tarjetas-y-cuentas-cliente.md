# obtener_tarjetas_cliente / obtener_cuentas_cliente

Métodos para recuperar los instrumentos de pago guardados por un cliente del organismo, para su uso posterior en `solicitud_pago_recurrente` o `solicitud_pago_recurrente_suscripcion`.

**API SOAP v2.5** | Código de respuesta prefijo: `10xxx`

---

## obtener_tarjetas_cliente

Devuelve la lista de tarjetas activas guardadas para uno o más clientes del organismo.

### Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `datosCliente` | Array de `DatosClienteRecurrente` | Clientes a consultar (mínimo 1) |

#### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | El token devuelto por `obtener_token` |

#### Estructura: DatosClienteRecurrente

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `identificador_cliente` | Texto | Identificador usado al registrar las tarjetas |
| `identificador_tarjeta` | Texto | **Opcional.** Si se envía, devuelve solo esa tarjeta; sino, devuelve todas |
| `proximo_vencimiento` | Booleano | **Opcional.** Si `true`, devuelve solo tarjetas a vencer en los próximos meses |

### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `token` | Texto | Token con el que se hizo la consulta |
| `id_organismo` | Numérico entero | Código de organismo |
| `tarjetas` | Array de `TarjetasClientes` | Tarjetas por cliente |

#### Estructura: TarjetasClientes

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `identificador_cliente` | Texto | Identificador del cliente asignado por el organismo |
| `tarjetas` | Array de `TarjetasCliente` | Tarjetas del cliente |

#### Estructura: TarjetasCliente

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `identificador_tarjeta` | Texto | Identificador que ePagos asigna a la tarjeta |
| `marca` | Texto | Marca de la tarjeta (VISA, Mastercard, etc.) |
| `id_fp` | Numérico entero | Código de la forma de pago |
| `fecha_alta` | Fecha (`AAAA-MM-DD`) | Fecha de alta de la tarjeta |
| `fecha_vencimiento` | Texto | Fecha de vencimiento de la tarjeta en formato `MM/AA` |

---

## obtener_cuentas_cliente

Devuelve la lista de cuentas bancarias activas guardadas para uno o más clientes del organismo (DEBIN o Débito directo).

### Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `datosCliente` | Array de `DatosClienteCuentaRecurrente` | Clientes a consultar (mínimo 1) |

#### Estructura: DatosClienteCuentaRecurrente

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `identificador_cliente` | Texto | Identificador usado al registrar las cuentas |
| `medio` | Set | Tipo de cuenta: `op_pago_recurrente_medio_debin` o `op_pago_recurrente_medio_debito_directo` |
| `identificador_cuenta` | Texto | **Opcional.** Si se envía, devuelve solo esa cuenta; sino, devuelve todas |

### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `token` | Texto | Token con el que se hizo la consulta |
| `id_organismo` | Numérico entero | Código de organismo |
| `cuentas` | Array de `CuentasClientes` | Cuentas por cliente |

#### Estructura: CuentasClientes

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `identificador_cliente` | Texto | Identificador del cliente asignado por el organismo |
| `cuentas` | Array de `CuentasCliente` | Cuentas del cliente |

#### Estructura: CuentasCliente

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `identificador_cuenta` | Texto | Identificador que ePagos asigna a la cuenta |
| `id_fp` | Numérico entero | Código de la forma de pago |
| `tipo_operacion` | Numérico entero | Tipo de impuesto usado al momento de generar la adhesión |
| `cbu` | Texto | CBU de la cuenta guardada |
| `email` | Texto | **Opcional.** E-mail relacionado con el `identificador_cliente` |
| `medio` | Set | Forma de pago: `op_pago_recurrente_medio_debin` o `op_pago_recurrente_medio_debito_directo` |
| `fecha_alta` | Fecha (`AAAA-MM-DD`) | Fecha de alta de la cuenta |

---

## Respuestas posibles (ambos métodos)

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `10001` | ✅ Correcta | Tarjetas / cuentas del cliente devueltas |
| `10002` | ❌ Error | Error al validar el token |
| `10003` | ❌ Error | Error interno al intentar devolver las tarjetas / cuentas |
| `10004` | ❌ Error | Error al validar el parámetro: `[_parametro_]` |
| `10005` | ❌ Error | Versión inválida del protocolo |

---

## Notas

- Los `identificador_tarjeta` / `identificador_cuenta` obtenidos aquí son los que se usan en `solicitud_pago_recurrente_suscripcion` (campo `SuscripcionCliente`).
- El campo `proximo_vencimiento` en tarjetas es útil para notificar a clientes que actualicen su tarjeta antes de que venza.
- Para cuentas, una combinación de `cbu` + `identificador_cliente` + `tipo_operacion` es única.
