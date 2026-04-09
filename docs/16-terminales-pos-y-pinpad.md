# Terminales POS: obtener_terminales_pos + solicitud_pago_pinpad

Métodos para integración con terminales POS físicas (PINPAD). Permiten iniciar cobros presenciales con tarjeta directamente desde el sistema del organismo hacia una terminal POS asignada.

> 🆕 **Métodos nuevos en v2.5**

**API SOAP v2.5** | Código de respuesta prefijo: `21xxx`

---

## obtener_terminales_pos

Lista las terminales POS asignadas al organismo.

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
| `terminales` | Array de `TerminalesPOS` | Terminales POS disponibles para el organismo |

#### Estructura: TerminalesPOS

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `numero` | Texto | Número de serie de la terminal POS |
| `descripcion` | Texto | Descripción de la terminal |
| `marca` | Texto | Marca de la terminal POS |
| `modelo` | Texto | Modelo de la terminal POS |

### Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `21001` | ✅ Correcta | Terminales devueltas correctamente |
| `21002` | ❌ Error | Versión inválida del protocolo |
| `21003` | ❌ Error | Error al validar el parámetro |
| `21004` | ❌ Error | Error al validar el token |

---

## solicitud_pago_pinpad

Inicia el cobro de una operación previamente generada en una terminal POS específica. La terminal recibe la instrucción y solicita al cliente que inserte/pase/acerque su tarjeta.

### Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `id_transaccion` | Numérico entero | Número de la operación a pagar (generada previamente con `solicitud_pago`) |
| `monto` | Numérico decimal | Monto a cobrar (**debe coincidir exactamente** con el de la operación) |
| `terminal` | Texto | Número de serie de la terminal POS destino |

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

### Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `21001` | ✅ Correcta | Operación creada correctamente |
| `21002` | ❌ Error | Versión inválida del protocolo |
| `21003` | ❌ Error | Error al validar el parámetro |
| `21004` | ❌ Error | Error al validar el token |

---

## Flujo típico

```
1. obtener_token
2. solicitud_pago → genera la operación, obtiene id_transaccion
3. [Opcional] obtener_terminales_pos → listar terminales disponibles y sus números de serie
4. solicitud_pago_pinpad (id_transaccion + monto + numero_serie_terminal)
   → La terminal POS muestra el monto y solicita la tarjeta al cliente
5. El cliente pasa/inserta/acerca su tarjeta en la POS
6. ePagos procesa el pago y notifica por webhook
7. Conciliar con obtener_pagos
```

## Notas

- La operación debe haberse generado previamente con `solicitud_pago` antes de llamar a `solicitud_pago_pinpad`.
- El `monto` enviado debe ser **exactamente igual** al de la operación original; de lo contrario el pago es rechazado.
- El `terminal` es el número de serie devuelto por `obtener_terminales_pos` en el campo `numero`.
- Este flujo es para pagos **presenciales con tarjeta** y requiere que el organismo tenga terminales POS asignadas por ePagos.
- Dado que ambos métodos comparten el prefijo `21xxx`, los códigos de respuesta son los mismos para `obtener_terminales_pos` y `solicitud_pago_pinpad`.
