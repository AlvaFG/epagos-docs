# Documentación Técnica — Webhooks ePagos

**Fuente:** https://www.epagos.com/desarrolladores.php?secc=webhook

---

## 1. Configuración de la URL del Webhook

ePagos permite configurar la URL que el Organismo defina para recibir avisos de acreditaciones de pago y devoluciones. Esto optimiza la operatoria y elimina la necesidad de consultar vía API el estado de cada pago.

| Aspecto | Detalle |
|---|---|
| **Dónde se configura** | Sección **Desarrolladores** del panel de control |
| **Dónde se puede probar** | Sección **Desarrolladores** del panel de control |
| **Método HTTP** | `POST` |
| **Cantidad de URLs** | Se puede configurar **una sola URL** para ambos tipos, **o bien una URL para pagos y otra para devoluciones** |

---

## 2. Formato exacto del Payload (POST HTTP)

La URL configurada recibirá vía `POST` de HTTP los siguientes campos:

### 2.1 Campos raíz del payload

| Campo | Tipo | Descripción |
|---|---|---|
| `id_resp` | Numérico entero | El código de respuesta de E-Checkout. Valor `02001` si es un pago. **No se envía** si no es un pago. |
| `id_transaccion` | Numérico entero | El número de operación en ePagos |
| `id_fp` | Numérico entero | Identificador de la forma de pago que el usuario eligió |
| `fp` | JSON (texto) | Objeto JSON con los datos adicionales del pago (ver sección 2.2) |
| `id_organismo` | Numérico entero | Identificador del organismo al que pertenece |
| `convenio` | Numérico entero | El número de convenio de la transacción |
| `numero_operacion` | Texto | El número o código de transacción que el cliente informó al momento de iniciar la solicitud del pago |
| `identificador_2` | Texto (máx. 100 chars) | Identificador adicional 2 |
| `identificador_3` | Texto (máx. 512 chars) | Identificador adicional 3 |
| `codigo_barras` | Texto | El código de barras asociado a la operación |
| `fecha_pago` / `fecha_devolucion` | Fecha (`AAAA-MM-DD`) | La fecha en que se realizó el pago (acreditación en el medio de pago) / La fecha en que se realizó la devolución (reintegro) del pago al usuario |
| `monto_pagado` | Numérico decimal | El importe abonado por el usuario |
| `tipo` | Texto (1 char) | Indica el tipo de envío: `P` = Pago, `D` = Devolución |

### 2.2 Estructura del campo `fp` (JSON anidado)

El campo `fp` es un **texto que contiene un JSON** con los siguientes subcampos:

| Campo | Descripción | Notas |
|---|---|---|
| `nombre_fp` | El nombre de la forma de pago | — |
| `id_fp` | Identificador de la forma de pago que el usuario eligió | — |
| `tarjeta_fp` | El número de tarjeta **hasheado** | Solo en casos de pago con tarjeta |
| `importe_fp` | El importe abonado por el usuario | — |
| `tipo_doc_fp` | El tipo de documento del pagador (`CUIT` o `DNI`) | (**) No disponible en todos los medios de pago |
| `numero_doc_fp` | El número de documento del pagador | (**) No disponible en todos los medios de pago |

> **Nota:** Algunos de estos datos se encuentran también fuera de la estructura `fp` (en el payload raíz), pero se agregan aquí para **unificar la respuesta de los medios de pago**.

---

## 3. Diferencia entre Webhook de Pagos y Webhook de Devoluciones

| Aspecto | Webhook de Pago | Webhook de Devolución |
|---|---|---|
| **Campo `tipo`** | `P` | `D` |
| **Campo `id_resp`** | `02001` (siempre presente) | No se envía |
| **Campo de fecha** | `fecha_pago` (fecha de acreditación en el medio de pago) | `fecha_devolucion` (fecha del reintegro al usuario) |
| **URL de recepción** | Puede ser la misma o una URL dedicada para pagos | Puede ser la misma o una URL dedicada para devoluciones |

> Se puede configurar **una única URL** para recibir ambos tipos de webhook, o bien **dos URLs separadas**: una para pagos y otra para devoluciones. La distinción entre el tipo de evento se realiza mediante el campo `tipo` (`P` o `D`).

---

## 4. Verificación/Validación del Webhook

La página de documentación **no detalla un mecanismo de firma o token de validación criptográfico** (como HMAC o header de autorización). La única información disponible sobre validación es:

| Aspecto | Detalle |
|---|---|
| **Identificación del tipo de evento** | Campo `tipo`: `P` = Pago, `D` = Devolución |
| **Identificación del pago** | Campo `id_resp`: valor `02001` confirma que es un pago de E-Checkout |
| **Configuración y prueba** | La URL puede **probarse** desde la sección Desarrolladores del panel de control |
| **Recomendación oficial** | Implementar un proceso complementario que consulte periódicamente (idealmente **una vez por día**) los pagos acreditados en el período, como resguardo ante envíos no recibidos o no procesados |

---

## 5. Códigos de Respuesta Esperados

| Campo | Valor | Descripción |
|---|---|---|
| `id_resp` | `02001` | Código de respuesta de E-Checkout que indica un pago exitoso |
| `id_resp` | *(no se envía)* | Cuando el evento es una devolución, este campo no está presente en el payload |

> La documentación no especifica códigos de respuesta HTTP que el servidor receptor deba retornar a ePagos (ej. `200 OK`). Solo se documenta el campo `id_resp` dentro del payload enviado.

---

## 6. Ejemplos de Payload JSON

La documentación de ePagos **no incluye ejemplos de payload JSON o XML explícitos** en esta página. Sin embargo, a partir de los campos documentados, la estructura del payload POST sería:

### Ejemplo — Pago (`tipo: P`)

```json
{
  "id_resp": 2001,
  "id_transaccion": 123456789,
  "id_fp": 5,
  "fp": "{\"nombre_fp\":\"Visa\",\"id_fp\":5,\"tarjeta_fp\":\"HASH_DE_TARJETA\",\"importe_fp\":1500.00,\"tipo_doc_fp\":\"DNI\",\"numero_doc_fp\":\"12345678\"}",
  "id_organismo": 42,
  "convenio": 1001,
  "numero_operacion": "OP-20260331-001",
  "identificador_2": "ID_ADICIONAL_2",
  "identificador_3": "ID_ADICIONAL_3_LARGO",
  "codigo_barras": "9501234500090",
  "fecha_pago": "2026-03-31",
  "monto_pagado": 1500.00,
  "tipo": "P"
}
```

### Ejemplo — Devolución (`tipo: D`)

```json
{
  "id_transaccion": 123456789,
  "id_fp": 5,
  "fp": "{\"nombre_fp\":\"Visa\",\"id_fp\":5,\"tarjeta_fp\":\"HASH_DE_TARJETA\",\"importe_fp\":1500.00}",
  "id_organismo": 42,
  "convenio": 1001,
  "numero_operacion": "OP-20260331-001",
  "identificador_2": "ID_ADICIONAL_2",
  "identificador_3": "ID_ADICIONAL_3_LARGO",
  "codigo_barras": "9501234500090",
  "fecha_devolucion": "2026-03-31",
  "monto_pagado": 1500.00,
  "tipo": "D"
}
```

> **Nota:** Los ejemplos son una reconstrucción fiel de los campos documentados. ePagos no publica ejemplos de payload en esta sección de la documentación.

---

## 7. Advertencia Oficial de ePagos

> Debido a la naturaleza de este tipo de notificaciones, puede darse que algún envío no lo reciba o que el mismo sea enviado y no pueda procesarse de su lado. Por lo que **recomendamos que además de implementar el Webhook, exista un proceso que regularmente (una vez por día es ideal), obtenga los pagos acreditados en el período.**
