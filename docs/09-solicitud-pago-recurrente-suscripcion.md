# solicitud_pago_recurrente_suscripcion

Planifica cobros recurrentes con periodicidad configurable sobre tarjetas o cuentas bancarias previamente guardadas por el cliente.

> ⚠️ **Este método no está disponible para todos los organismos.** Consultar con el ejecutivo comercial de ePagos.

**API SOAP v2.5** | Código de respuesta prefijo: `11xxx`

---

## Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `tipo_operacion` | Set | Valor fijo `"op_pago_recurrente"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `operacion` | Estructura `DatosOperacionPago` | Datos de la operación a realizar |
| `suscripcion` | Array de `DatosSuscripcion` | Planificaciones de cobros recurrentes (mínimo 1 elemento) |
| `suscripcion_modalidad` | Set | Modalidad: `P`=Personalizada, `M`=Mensual, `S`=Semanal, `A`=Anual |
| `descripcion` | Texto | Nombre o descripción de la suscripción |
| `convenio` | Numérico entero | Número de convenio asignado por ePagos |
| `medio` | Set | Medio de pago para los cobros: `op_pago_recurrente_medio_tarjeta`, `op_pago_recurrente_medio_debin`, `op_pago_recurrente_medio_debito_directo` |
| `clientes` | Array de `SuscripcionCliente` | Clientes incluidos en la suscripción |

### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | El token devuelto por `obtener_token` |

### Estructura: DatosSuscripcion

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `fecha` | Fecha (`AAAA-MM-DD`) | Fecha del cobro. Debe ser **mayor a la fecha actual** |
| `monto` | Numérico decimal | Monto diferencial para esta fecha. Si se omite, se usa el de `operacion` |

### Estructura: SuscripcionCliente

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `identificador_cliente` | Texto | Identificador de cliente usado al registrar las tarjetas/cuentas |
| `identificador_tarjeta` | Texto | Identificador de tarjeta obtenido con `obtener_tarjetas_cliente` (para medio tarjeta) |
| `identificador_cuenta` | Texto | Identificador de cuenta obtenido con `obtener_cuentas_cliente` (para DEBIN / débito directo) |

---

## Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `token` | Texto | Token con el que se hizo la consulta |
| `id_organismo` | Numérico entero | Código de organismo |

---

## Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `11001` | ✅ Correcta | Pago recurrente con suscripción planificado |
| `11002` | ❌ Error | Error al validar el token |
| `11003` | ❌ Error | Error interno al intentar procesar el pago recurrente con suscripción |
| `11004` | ❌ Error | Error al validar el parámetro: `[_parametro_]` |
| `11005` | ❌ Error | No coinciden los montos y los detalles |
| `11006` | ❌ Error | La fecha de planificación es inválida |
| `11007` | ❌ Error | No existe una tarjeta guardada con dichos datos |
| `11008` | ❌ Error | No existe una cuenta guardada con dichos datos |
| `11009` | ❌ Error | Versión inválida del protocolo |
| `15002` | ❌ Error | Método no autorizado para su organismo |

---

## Flujo típico de uso

```
1. obtener_token
2. solicitud_pago (con guardado de tarjeta/cuenta habilitado) → cliente paga y guarda datos
3. obtener_tarjetas_cliente / obtener_cuentas_cliente → recuperar identificadores
4. solicitud_pago_recurrente_suscripcion → planificar cobros futuros
5. obtener_resultados_debito → verificar estado de los débitos procesados
```

## Notas

- Las tarjetas/cuentas deben haber sido guardadas previamente en una `solicitud_pago` con la opción de guardado habilitada.
- La modalidad `P` (Personalizada) permite definir fechas y montos distintos para cada cobro.
- Ver también: `solicitud_pago_recurrente` (cobro único recurrente sin suscripción programada).
