# generar_qr_vinculado

Genera un QR que funciona como selector de operaciones a pagar. Es útil para boletas con más de una operación donde se quiere mostrar un único QR que permita al pagador elegir cuál abonar.

**API SOAP v2.5** | Código de respuesta prefijo: `14xxx`

---

## Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `operaciones_qr` | Array de `OperacionQR` | Operaciones a incluir en el selector (mínimo 1) |

### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | El token devuelto por `obtener_token` |

### Estructura: OperacionQR

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_transaccion` | Numérico entero | Identificador de la transacción devuelto por ePagos al generarla |
| `etiqueta` | Texto | Texto que representará a la operación en el selector que ve el pagador |

---

## Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `qr` | Base64 | Imagen del QR codificada en base64 |

---

## Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `14001` | ✅ Correcta | Código QR generado con éxito |
| `14002` | ❌ Error | Error al registrar la operación vinculada: `#[_parametro_]` |
| `14003` | ❌ Error | La operación `#[_parametro_]` no pertenece al organismo |
| `14004` | ❌ Error | El hash `[_parametro_]` ya se encuentra registrado |
| `14005` | ❌ Error | Hay operaciones duplicadas en el envío |
| `14006` | ❌ Error | Versión inválida del protocolo |

---

## Ejemplo de uso

```
Escenario: Boleta de servicios con 3 impuestos (ABL, Patente, Automotor)
→ Generar 3 operaciones con solicitud_pago → obtener 3 id_transaccion
→ Llamar a generar_qr_vinculado con las 3 operaciones y sus etiquetas:
   - id_transaccion: 100001, etiqueta: "ABL - Cuota 1/2025"
   - id_transaccion: 100002, etiqueta: "Patente Vehículo"
   - id_transaccion: 100003, etiqueta: "Automotor"
→ El pagador escanea el QR y ve un selector para elegir qué pagar
```

## Notas

- El QR devuelto en `qr` es una imagen en base64 lista para incrustar en HTML: `<img src="data:image/png;base64,{qr}" />`
- Cada par `id_transaccion` + operaciones combinadas genera un hash único. Si se intenta re-generar el mismo vínculo, se obtiene el error `14004`.
- Las operaciones vinculadas deben pertenecer al mismo organismo (`14003` si no).
- No se puede incluir la misma `id_transaccion` dos veces en el mismo request (`14005`).
