---
name: epagos-soap-payments
description: ePagos SOAP v2.1 payment creation - solicitud_pago, QR, batch, recurring, POS. Use when working on server-side payment creation, QR generation, bulk emission, or recurring charges.
---

# ePagos SOAP Payment Creation (v2.1)

## Overview

SOAP v2.1 is used for **creating payments server-side** (not for reconciliation -- that uses v2.0). Use cases:
- QR code generation for Transferencias 3.0
- Bulk/batch payment emission
- Recurring payments and subscriptions
- POS terminal payments

**Key difference from eCheckout:** SOAP v2.1 creates the payment operation directly on the server. eCheckout redirects the user to ePagos to select a payment method.

**Raw XML required:** Some SOAP libraries cannot produce `SOAP-ENC:Array` attributes that ePagos expects. You may need to build raw XML SOAP envelopes manually for `solicitud_pago`.

> For eCheckout flow see `epagos-echeckout`. For reconciliation see `epagos-reconciliation`. For reference tables see `epagos-reference`.

---

## URLs (SOAP v2.1)

| Purpose | Sandbox | Production | Domain |
|---------|---------|------------|--------|
| WSDL v2.1 | `https://sandbox.epagos.com/wsdl/2.1/index.php?wsdl` | `https://api.epagos.com/wsdl/2.1/index.php?wsdl` | .com |
| Endpoint (raw POST) | `https://sandbox.epagos.com/wsdl/2.1/index.php` | `https://api.epagos.com/wsdl/2.1/index.php` | .com |
| Namespace (tns) | `https://sandbox.epagos.net/` | `https://api.epagos.net/` | .net |
| SOAPAction | `https://sandbox.epagos.net/solicitud_pago` | `https://api.epagos.net/solicitud_pago` | .net |

**Important:** The endpoint uses `.com` but the namespace and SOAPAction use `.net`. This is NOT an error -- they are genuinely different servers.

---

## Token Management (SOAP v2.1)

SOAP v2.1 uses a **separate token** from v2.0 (different WSDL endpoint).

| Property | Value |
|----------|-------|
| Cache TTL | ~3000 seconds (50 minutes) |
| Auto-retry on | `02003` (token validation error) |
| Version param | `"2.0"` (always, even for v2.1) |

Token request uses the same `obtener_token` SOAP method as v2.0, but against the v2.1 WSDL.

---

## solicitud_pago (create payment)

```
Request (SOAP XML):
  version: "2.0"
  tipo_operacion: "op_pago"
  credenciales:
    id_organismo: from config
    token: from v2.1 obtener_token
  operacion:
    numero_operacion: string (payment UUID)
    identificador_externo_2: string (100 chars -- secondary reference)
    identificador_externo_3: string (512 chars -- tertiary reference)
    identificador_externo_4: string (65535 chars -- JSON metadata)
    identificador_cliente: string (for recurrence, optional)
    id_moneda_operacion: 1 (ARS)
    monto_operacion: decimal
    opc_pdf: boolean (default true)
    opc_fecha_vencimiento: "YYYY-MM-DD"
    opc_devolver_qr: boolean
    opc_devolver_codbarras: boolean
    opc_generar_pdf: boolean
    opc_fp_excluidas: "1,2,3"
    opc_tp_excluidos: "1,2"
    opc_fp_permitidas: "1,2,3"
    opc_T30_cerrado: boolean (closed amount QR)
    opc_T30_reutilizable: boolean (reusable QR)
    url_ok: string
    url_error: string
    detalle_operacion: array of items:
      - id_item: integer
      - desc_item: string
      - monto_item: decimal
      - cantidad_item: integer
    pagador:
      nombre_pagador, apellido_pagador, email_pagador
      identificacion_pagador: { tipo_doc_pagador, numero_doc_pagador, cuit_doc_pagador }
      domicilio_pagador: { calle, numero, adicional, cp, ciudad, provincia, pais }
      telefono_pagador: { codigo, numero }
  fp: array of payment methods:
    - id_fp: integer (payment method ID)
    - monto_fp: decimal
    - tarjeta: { numero, banco, vencimiento: {mes, anio}, codigo_seg, cuotas, titular, identificacion, fechanac, direccion }
  convenio: integer
```

### solicitud_pago response

| Field | Type | Description |
|-------|------|-------------|
| `id_resp` | text | 02001=credited, 02002=pending |
| `respuesta` | text | Description |
| `id_transaccion` | int | ePagos unique ID |
| `fp[].codigo_pago_fp` | text | Short payment code (cash/homebanking) |
| `fp[].codigo_barras_fp` | text | Full barcode |
| `fp[].fechavenc_fp` | date | Due date |
| `fp[].importe_fp` | decimal | Amount |
| `fp[].pdf` | base64 | PDF receipt (if opc_pdf=true) |
| `fp[].codigo_barras_imagen` | base64 | Barcode PNG (if opc_devolver_codbarras) |
| `fp[].qr_imagen` | base64 | QR PNG (if opc_devolver_qr) |
| `fp[].qr_imagen_T30` | base64 | Transfers 3.0 QR PNG |
| `fp[].url_qr` | URL | Online card payment URL |
| `fp[].codigo_pmc` | text | Pago Mis Cuentas code |
| `fp[].codigo_link` | text | Red Link code |
| `fp[].cuit_cuenta`, `fp[].cbu_cuenta` | text | For DEBIN/transfer |

---

## QR / Transferencias 3.0 via solicitud_pago

Use `solicitud_pago` with FP 44 for QR payments:

| Setting | Value | Purpose |
|---------|-------|---------|
| `fp.id_fp` | `44` | Transferencias 3.0 |
| `fp.monto_fp` | Full amount | Must match monto_operacion |
| `opc_T30_cerrado` | `true` | Closed amount (user pays exact amount) |
| `opc_T30_reutilizable` | `false` | One-time QR |
| `opc_devolver_qr` | `true` | Return QR image in response |

Expected response: `02002` (pending) -- user hasn't scanned yet.

QR image returned in `qr_imagen_T30` (or `qr_imagen`), QR text in `qr_texto_T30`, URL in `url_qr`.

---

## obtener_entidades_pago (payment methods)

Returns available payment methods for the organization with logos, limits, and configuration.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  fp: array of { id_fp } (optional filter)

Response:
  fp: array of:
    id_fp: int
    nombre_fp: text
    tipo_fp: set (tipo_fp_presencial|tipo_fp_credito|tipo_fp_debito|...)
    estado_fp: set (estado_fp_activo|estado_fp_desactivado)
    logos_fp: { seguro_logos_fp (HTTPS URL), nombre_logos_fp (HTTP URL) }
    config_fp: { patron_config_fp, longitud_config_fp, validacion_config_fp, codseg_config_fp, codseg_long_config_fp, codseg_ubic_config_fp }
    adicional_fp: text (extra info needed, e.g. CBU, patente)
    monto_minimo_fp: decimal
    monto_maximo_fp: decimal
    tiempo_acreditacion_fp: int (hours)

Error codes: 03001=OK | 03002=invalid token | 03003=internal | 03004=invalid param | 03005=invalid version
```

Use case: dynamically build payment method selector with logos and limits.

---

## Batch Operations

### solicitud_pago_lote (batch -- up to 50)

Multiple payments in a single call.

```
Request:
  version: "2.0"
  tipo_operacion: "op_pago"
  credenciales: { id_organismo, token }
  lote: array of (max 50):
    - fp: array of DatosFormaPago
    - operacion: DatosOperacionPago
    - convenio: int

Response:
  lote: array of:
    - id_transaccion: int
    - numero_operacion: text
    - convenio: int
    - respuesta_forma_pago_array: array of RespuestaFormaPago

Error codes: 08001=OK | 08002=invalid token | 08003=internal | 08004=invalid param | 08005=batch size exceeds limit | 08006=processing error | 08007=amount/detail mismatch | 08008=invalid version
```

Response array positions correspond to request array positions.

### pago_lote (mark batch as paid)

Links multiple operations to an already-credited base operation. Requires authorization.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  pago_lote:
    id_transaccion: int (base operation -- must be credited, NOT settled)
    forma_pago: int (payment method ID)
    fecha_pago: date
    importe: decimal (must equal sum of operation amounts)
    operaciones: array of:
      - id_transaccion: int
      - importe: decimal

Error codes: 15001=OK | 15002=not authorized | 15003=invalid batch | 15004=invalid param | 15005=invalid token
```

---

## Recurring & Subscriptions

> Not available for all organizations -- must request from ePagos.

### Flow

```
1. solicitud_pago (with opc_guardado_obligatorio=true + identificador_cliente)
   -> User pays and saves their card/account
2. obtener_tarjetas_cliente / obtener_cuentas_cliente
   -> Retrieve saved payment instrument identifiers
3. solicitud_pago_recurrente -> charge stored card/account (one-time)
4. solicitud_pago_recurrente_suscripcion -> schedule recurring charges
5. obtener_resultados_debito -> verify bank response for processed debits
```

### obtener_tarjetas_cliente (saved cards)

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  datosCliente: array of:
    - identificador_cliente: text (min 6 chars)
    - identificador_tarjeta: text (optional filter)
    - proximo_vencimiento: boolean (optional)

Response:
  tarjetas: array of:
    - identificador_cliente: text
    - tarjetas: array of:
      - identificador_tarjeta: text (use in recurring charges)
      - marca: text (VISA, Mastercard, etc.)
      - id_fp: int
      - fecha_alta: date
      - fecha_vencimiento: text (MM/YY)

Error codes: 10001=OK | 10002=invalid token | 10003=internal | 10004=invalid param | 10005=invalid version
```

### obtener_cuentas_cliente (saved bank accounts)

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  datosCliente: array of:
    - identificador_cliente: text
    - medio: set (op_pago_recurrente_medio_debin | op_pago_recurrente_medio_debito_directo)
    - identificador_cuenta: text (optional filter)

Response:
  cuentas: array of:
    - identificador_cliente: text
    - cuentas: array of:
      - identificador_cuenta: text (use in recurring charges)
      - id_fp: int
      - tipo_operacion: int
      - cbu: text
      - email: text (optional)
      - medio: set
      - fecha_alta: date

Error codes: 10001=OK | 10002=invalid token | 10003=internal | 10004=invalid param | 10005=invalid version
```

### registrar_cuentas_cliente (bulk register -- up to 100)

Pre-register bank account adhesions without a payment flow.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  cuentas: array of (max 100):
    - identificador_cliente: text
    - tipo_operacion: int
    - cuit: int (optional -- validates CBU ownership)
    - cbu: text (22 digits)
    - fecha_adhesion: date

Response:
  cuentas: array of:
    - identificador_cliente: text
    - identificador_cuenta: text (ePagos-assigned ID)

Error codes: 16001=OK | 16002=invalid version | 16003=invalid param | 16004=invalid token | 16005=adhesion error
```

Uniqueness constraint: `cbu` + `identificador_cliente` + `tipo_operacion` must be unique.

### solicitud_pago_recurrente_suscripcion (schedule recurring)

```
Request:
  version: "2.0"
  tipo_operacion: "op_pago_recurrente"
  credenciales: { id_organismo, token }
  operacion: DatosOperacionPago
  suscripcion: array of (min 1):
    - fecha: "YYYY-MM-DD" (future date)
    - monto: decimal (optional override)
  suscripcion_modalidad: P (custom) | M (monthly) | S (weekly) | A (annual)
  descripcion: text
  convenio: int
  medio: op_pago_recurrente_medio_tarjeta | op_pago_recurrente_medio_debin | op_pago_recurrente_medio_debito_directo
  clientes: array of:
    - identificador_cliente: text
    - identificador_tarjeta: text (for cards)
    - identificador_cuenta: text (for DEBIN/direct debit)

Error codes: 11001=OK | 11002=token | 11003=internal | 11004=param | 11005=amount mismatch | 11006=invalid date | 11007=card not found | 11008=account not found | 11009=version | 15002=not authorized
```

Card rejection codes (do NOT retry): `cc_rejected_blacklist`, `cc_rejected_card_disabled`, `cc_rejected_bad_filled_date`, `cc_rejected_bad_filled_other`

### obtener_resultados_debito (bank responses)

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  datosDebito: array of int (operation numbers)

Response:
  resultados: array of:
    - operacion: int
    - codigo: text (bank response code, NOT ePagos code)
    - resultado: text (bank description)

Error codes: 17001=OK | 17002=version | 17003=param | 17004=token | 17005=invalid operation
```

Poll periodically after debits are processed to catch bank rejections.

---

## QR Methods

### generar_qr_vinculado (linked QR for multiple operations)

Single QR that acts as a selector -- user scans and chooses which operation(s) to pay.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  operaciones_qr: array of:
    - id_transaccion: int (from solicitud_pago)
    - etiqueta: text (label shown in selector)

Response:
  qr: base64 (PNG image)

Error codes: 14001=OK | 14002=registration error | 14003=not owned by org | 14004=hash already registered | 14005=duplicate operations | 14006=version
```

All operations must belong to the same organization. No duplicate `id_transaccion`.

### obtener_cajas_qr (list QR boxes)

Static interoperable QR boxes (T3.0).

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }

Response:
  cajas: array of:
    - id_caja: int
    - nombre_caja: text
    - tipo_caja: F (Fisica) | V (Virtual)
    - monto_maximo_caja: decimal

Error codes: 18002=OK | 18003=invalid token | 18011=no boxes
```

### generar_orden_qr (T3.0 payment order)

Dynamic payment order for a static QR box.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  orden: array of:
    - id_caja: int (or id_transaccion)
    - importe: decimal (> 0, 2 decimals)
    - concepto: text (150 chars, optional)
    - identificador_2/3/4: text (optional)
    - email_pagador: text (optional)
    - detalle_orden: text (255, optional)
    - vencimiento: "YYYY-MM-DD" (optional)

Error codes: 18001=OK | 18003=token | 18004=exceeds max | 18005=param | 18006=expiry | 18007=date format | 18008=internal | 18009=invalid box | 18010=amount <= 0 | 18011=no boxes | 18012=invalid operation
```

### QR Interoperable Flow
```
1. obtener_cajas_qr -> list boxes, get id_caja
2. Print static QR at physical/virtual payment points
3. On each payment: generar_orden_qr (id_caja + importe + metadata)
4. Customer scans static QR with wallet app (MODO, MercadoPago, etc.)
5. Customer sees order and confirms payment
6. ePagos notifies via webhook
7. Reconcile with obtener_pagos
```

---

## POS Terminals

> **Note:** POS methods (obtener_terminales_pos, solicitud_pago_pinpad) were added in API v2.5. They are available on WSDL v2.5 (`/wsdl/2.5/index.php?wsdl`) but NOT on v2.1. Use the v2.5 WSDL endpoint if POS support is needed.

### obtener_terminales_pos

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }

Response:
  terminales: array of:
    - numero: text (serial number)
    - descripcion: text
    - marca: text
    - modelo: text

Error codes: 21001=OK | 21002=version | 21003=param | 21004=token
```

### solicitud_pago_pinpad

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  id_transaccion: int (from solicitud_pago)
  monto: decimal (must match exactly)
  terminal: text (serial number)

Error codes: 21001=OK | 21002=version | 21003=param | 21004=token
```

### POS Flow
```
1. obtener_token
2. solicitud_pago -> get id_transaccion
3. obtener_terminales_pos -> list terminals (optional)
4. solicitud_pago_pinpad (id_transaccion + monto + terminal)
5. Customer taps/inserts card
6. ePagos notifies via webhook
7. Reconcile with obtener_pagos
```

---

## Raw XML SOAP Example

The `solicitud_pago` call may require raw XML because some SOAP libraries cannot produce `SOAP-ENC:Array` type attributes.

Key structural elements:
- SOAP 1.2 envelope with `xsi:type` annotations on every field
- `SOAP-ENC:Array` for `detalle_operacion` and `fp` arrays
- Namespace: `tns:` pointing to `.net` domain
- SOAPAction header: `.net` domain + `/solicitud_pago`
