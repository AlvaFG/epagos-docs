# obtener_resultados_debito

Devuelve los códigos de respuesta bancaria del procesamiento de débitos iniciados con `solicitud_pago_recurrente`.

**API SOAP v2.5** | Código de respuesta prefijo: `17xxx`

---

## Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `datosDebito` | Array de enteros | Números de operaciones de las que se desea conocer el estado |

### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | El token devuelto por `obtener_token` |

---

## Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `resultados` | Array de `ResultadosDebito` | Estado de cada débito solicitado |

### Estructura: ResultadosDebito

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `operacion` | Número entero | Identificador de la operación |
| `codigo` | Texto | Código de respuesta del banco |
| `resultado` | Texto | Descripción del resultado del banco |

---

## Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `17001` | ✅ Correcta | Resultados devueltos correctamente |
| `17002` | ❌ Error | Versión inválida del protocolo |
| `17003` | ❌ Error | Error al validar el parámetro: `[_parametro_]` |
| `17004` | ❌ Error | Error al validar el token |
| `17005` | ❌ Error | La operación `[_operacion_]` es inválida o no pertenece a su organismo |

---

## Notas

- Este método es el equivalente de conciliación para débitos bancarios (DEBIN / Débito directo), distinto de `obtener_pagos` que aplica a todos los medios.
- El `codigo` devuelto es el código de respuesta que retorna el banco del pagador (no un código interno de ePagos).
- Consultar periódicamente luego de que se procesen los débitos para verificar rechazos bancarios y reintentar o notificar al cliente.
