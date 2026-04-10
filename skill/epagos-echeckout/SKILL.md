---
name: epagos-echeckout
description: ePagos eCheckout payment flow - token, redirect, callbacks, webhooks, test cards. Use when working on payment initiation, checkout, ok_url/error_url callbacks, webhook handling, or payment button.
---

# ePagos eCheckout Flow

## Overview

eCheckout is a POST-redirect flow where the user pays on ePagos-hosted pages. No PCI certification needed -- ePagos handles all card data.

```
User clicks "Pay" -> Backend gets token -> Backend creates hidden form ->
User redirected to ePagos -> User pays -> ePagos redirects to ok_url/error_url ->
Backend receives POST with result -> Webhook confirms asynchronously
```

> For reference tables (payment method IDs, error codes, test cards) see `epagos-reference`. For reconciliation see `epagos-reconciliation`.

---

## URLs (eCheckout only)

| Purpose | Sandbox | Production |
|---------|---------|------------|
| Token (POST) | `https://sandbox.epagos.com/post.php` | `https://api.epagos.com/post.php` |
| User redirect | `https://postsandbox.epagos.com` | `https://post.epagos.com` |
| JS library | `https://sandbox.epagos.com/quickstart/epagos.min.js` | `https://api.epagos.com/quickstart/epagos.min.js` |
| Portal (webhook config) | `https://portalsandbox.epagos.com` | `https://portal.epagos.com` |

All eCheckout URLs use `.com` domain.

---

## Step 1: Get Token (backend)

```
POST https://sandbox.epagos.com/post.php
Content-Type: application/x-www-form-urlencoded

id_usuario=<provided by ePagos>&id_organismo=<provided by ePagos>&password=<provided by ePagos>&hash=<provided by ePagos>
```

Response: `{"token": "abc123..."}`

**Rules:**
- eCheckout tokens are **single-use** -- get a fresh token for EVERY payment. No caching.
- This is a DIFFERENT token from SOAP `obtener_token`. Do not mix them.

---

## Step 2: Redirect User (frontend)

Create a hidden form that POSTs to `https://postsandbox.epagos.com`:

### Required fields

| Field | Value | Notes |
|-------|-------|-------|
| `version` | `"2.0"` | Always "2.0" (not "1.0") |
| `operacion` | `"op_pago"` | Fixed value |
| `id_organismo` | From credentials | |
| `token` | From step 1 | Single-use |
| `convenio` | From credentials (or empty) | |
| `numero_operacion` | Your payment UUID | Unique identifier for this payment |
| `id_moneda_operacion` | `1` | ARS (Argentine Peso) |
| `monto_operacion` | `480.00` | Decimal, 2 places, period separator |
| `detalle_operacion` | JSON array (URL-encoded or base64) | Sum of items MUST equal monto_operacion |
| `ok_url` | Callback URL for success | |
| `error_url` | Callback URL for errors | |

### detalle_operacion structure

```json
[
  {
    "id_item": "1",
    "desc_item": "Hamburguesa Classic",
    "monto_item": "350",
    "cantidad_item": "1"
  },
  {
    "id_item": "2",
    "desc_item": "Coca Cola 500ml",
    "monto_item": "130",
    "cantidad_item": "1"
  }
]
```

**Sum of (monto_item * cantidad_item) MUST equal monto_operacion.** Encoding: `urlencode(JSON)` or `base64_encode(JSON)`.

Set `detalle_operacion_visible = 1` to show items on the payment page.

### Optional fields

| Field | Type | Description |
|-------|------|-------------|
| `fp_permitidas` | text | Comma-separated or serialized array of allowed payment method IDs |
| `fp_excluidas` | text | Excluded payment method IDs |
| `tp_excluidos` | text | Excluded payment type IDs |
| `fp_defecto` | int | Default/preferred payment method ID |
| `opc_fecha_vencimiento` | YYYY-MM-DD | Due date for cash/homebanking (must be > today) |
| `opc_una_cuota` | boolean | Force single installment for credit cards |
| `opc_guardado_obligatorio` | boolean | Force user to save payment method (for recurrence) |
| `opc_pdf` | boolean | Return PDF receipt (default: true) |
| `opc_devolver_qr` | boolean | Return QR image for cash/homebanking (default: true) |
| `opc_devolver_codbarras` | boolean | Return barcode image (default: false) |
| `opc_comision` | boolean | Return commission value (default: false) |
| `opc_email_automatico` | boolean | Ask user for email if not provided (default: true) |
| `opc_descargar_pdf` | boolean | Show PDF download interface (default: false) |
| `opc_totem` | boolean | On-screen keyboard for kiosk (default: false) |
| `opc_T30_monto_abierto` | boolean | Open amount QR (default: false) |
| `opc_T30_reutilizable` | boolean | Reusable QR (default: false) |
| `opc_T30_requiere_orden` | boolean | QR requires payment order (default: false) |
| `opc_operaciones_lote` | text | Comma-separated id_transaccion to auto-credit |
| `fecha_2do_venc` | YYYY-MM-DD | Second due date |
| `monto_operacion_2do_venc` | decimal | Amount for second due date |

### Payer data (optional)

| Field | Description |
|-------|-------------|
| `nombre_pagador`, `apellido_pagador` | Name |
| `email_pagador` | Email |
| `tipo_doc_pagador`, `numero_doc_pagador` | Document |
| `cuit_doc_pagador` | CUIT/CUIL |
| `cbu_pagador` | CBU for DEBIN |

### External identifier fields

| ePagos Field | Max Length | Suggested Usage |
|-------------|-----------|-----------------|
| `numero_operacion` | 100 | Your payment UUID |
| `identificador_externo_2` | 100 | Secondary reference (e.g., order ID, session ID) |
| `identificador_externo_3` | 512 | Tertiary reference (e.g., merchant ID, store ID) |
| `identificador_externo_4` | 65535 | JSON metadata (order details, extra context) |

---

## Step 3: Handle Callback (backend)

**IMPORTANT:** `ok_url` does NOT guarantee credited payment. Code `02002` = pending (user hasn't paid yet for cash/transfer). Always verify via webhook or API.

### Callback for CARDS (ok_url POST)

| Field | Type | Description |
|-------|------|-------------|
| `id_resp` | int | 02001=credited, 02002=pending |
| `respuesta` | text | Response description |
| `convenio` | int | Agreement number |
| `fp` | JSON text | `{nombre_fp, tarjeta_fp, importe_fp, importe_original_fp, respuesta_entidad_cobro}` |
| `id_organismo` | int | Organization ID |
| `id_transaccion` | int | ePagos transaction ID |
| `id_fp` | int | Payment method ID chosen by user |
| `numero_operacion` | text | Your payment UUID (as sent) |
| `identificador_2` | text | Your identificador_externo_2 |
| `identificador_3` | text | Your identificador_externo_3 |
| `identificador_4` | text | Your identificador_externo_4 |
| `token` | text | The token used |

### Callback for CASH (ok_url) -- additional fields

| Field | Type | Description |
|-------|------|-------------|
| `pdf` | text | Base64-encoded PDF payment slip |
| `codigo_barras_fp` | int | Barcode number |
| `codigo_pago_fp` | text | Payment code |
| `codigo_qr` | text | Base64-encoded QR image |
| `fp` | JSON | `{nombre_fp, importe_fp, respuesta_entidad_cobro, fechavenc_fp}` |

### Callback for HOMEBANKING/TRANSFER/DEBIN (ok_url)

| Field | Type | Description |
|-------|------|-------------|
| `pdf` | text | Base64-encoded PDF payment slip |
| `cuit_cuenta` | int | CUIT for homebanking/transfer |
| `cbu_cuenta` | text | CBU/CVU for DEBIN |

### Callback for ERROR (error_url POST)

| Field | Type | Description |
|-------|------|-------------|
| `id_resp` | int | Error response code |
| `respuesta` | text | Error description |
| `convenio` | int | Agreement number |
| `fp` | JSON | `{nombre_fp, importe_fp, respuesta_entidad_cobro}` |
| `id_organismo` | int | Organization ID |
| `id_transaccion` | int | ePagos transaction ID |
| `numero_operacion` | text | Your payment UUID |
| `identificador_2`, `identificador_3` | text | Your identifiers |
| `token` | text | The token used |

### Response Codes (02xxx)

| Code | Type | Description |
|------|------|-------------|
| 02001 | OK | Pago acreditado (credited) -- instant for cards/wallets |
| 02002 | OK | Pago pendiente (pending) -- deferred for cash/transfer |
| 02003 | Error | Token validation error |
| 02004 | Error | Payment cancelled/rejected |
| 02005 | Error | Internal processing error |
| 02006 | Error | Parameter validation error |
| 02007 | Error | User cancelled payment |
| 02008 | Error | Amount/detail mismatch |
| 02009 | Error | Payment method not available |
| 02010 | Error | Online service provider error |

---

## Step 4: Verify Payment

4 levels of verification (use all):

1. **Callback** (redirect) -- immediate, but 02002 is just "pending"
2. **Webhook** (real-time) -- preferred for confirmation
3. **API Polling** -- `obtener_pagos` background job (see `epagos-reconciliation`)
4. **Settlement** -- `obtener_rendiciones` daily files (see `epagos-reconciliation`)

---

## Webhook Integration (CRITICAL)

Configure URL in ePagos portal > Developers section. Can set one URL for both events or separate URLs.

### Webhook Payload (POST)

| Field | Type | Description |
|-------|------|-------------|
| `id_resp` | int | 02001 for payments. **NOT sent** for refunds |
| `id_transaccion` | int | ePagos transaction ID |
| `id_fp` | int | Payment method ID |
| `fp` | JSON text | Nested JSON with payment details (see below) |
| `id_organismo` | int | Organization ID |
| `convenio` | int | Agreement number |
| `numero_operacion` | text | Your payment UUID |
| `identificador_2` | text (100) | Your identifier 2 |
| `identificador_3` | text (512) | Your identifier 3 |
| `codigo_barras` | text | Barcode |
| `fecha_pago` | YYYY-MM-DD | Payment date (only for tipo=P) |
| `fecha_devolucion` | YYYY-MM-DD | Refund date (only for tipo=D) |
| `monto_pagado` | decimal | Amount paid |
| `tipo` | char | **P** = Payment, **D** = Refund |

### Webhook `fp` nested JSON

| Field | Description |
|-------|-------------|
| `nombre_fp` | Payment method name |
| `id_fp` | Payment method ID |
| `tarjeta_fp` | Card number (hashed) -- cards only |
| `importe_fp` | Amount paid |
| `tipo_doc_fp` | Payer document type -- not all methods |
| `numero_doc_fp` | Payer document number -- not all methods |

### Webhook Security

**ePagos does NOT provide HMAC or cryptographic verification.** Verify by:
1. Check `tipo` field: P=payment, D=refund
2. Check `id_resp` = 02001 for payments
3. Cross-reference `id_transaccion` with your DB
4. **ALWAYS** implement daily reconciliation via `obtener_pagos` as fallback

### Acreditacion timing

| Method | Timing | ok_url code |
|--------|--------|------------|
| Credit/Debit cards | Immediate | 02001 (credited) |
| Billetera ePagos/MODO/MercadoPago | Immediate | 02001 (credited) |
| Cash (PagoFacil, Rapipago) | Deferred | 02002 (pending -- user hasn't paid) |
| Homebanking | Deferred | 02002 (pending) |
| Transfer/DEBIN | Deferred | 02002 (pending) |

---

## Payment Button (JS) Integration

Alternative to eCheckout redirect -- loads a button directly in the page.

**Requirements:**
1. Request a **clave publica** (public key) from ePagos, specifying your domain(s)
2. Include the JS library

### Init methods

| Method | Description |
|--------|-------------|
| `ePagos.setClavePublica(key)` | Set public key credential |
| `ePagos.setOrganismo(id)` | Set organization ID |
| `ePagos.setAmbiente(env)` | `"T"` = Sandbox, `"P"` = Production |

### Error handling

```javascript
window.addEventListener("errorEpagos", function(event) {
  console.error(event.detail.mensaje);
});
```

### Complete example

```html
<div id="epagos_btn"></div>

<script type="text/javascript">
  var script = document.createElement("script");
  script.addEventListener("load", function() {
    window.addEventListener("errorEpagos", function(event) {
      console.error(event.detail.mensaje);
    });

    ePagos.setClavePublica("YOUR_PUBLIC_KEY");
    ePagos.setOrganismo('YOUR_ORG_ID');
    ePagos.setAmbiente("T");

    ePagos.botonPago({
      "datosOperacion": {
        "convenio": "00000",
        "ok_url": "https://yoursite.com/payment/ok",
        "error_url": "https://yoursite.com/payment/error",
        "monto_operacion": 1500
      },
      "atributos": {
        "label": "Pagar $1500",
        "elemento_destino": "epagos_btn",
        "className": "btn btn-primary"
      },
      "estiloTooltip": {
        "background_color": "#333",
        "font_family": "Arial",
        "font_color": "#fff",
        "font_size": "14px"
      }
    });
  });
  script.src = "https://sandbox.epagos.com/quickstart/epagos.min.js";
  script.async = true;
  document.getElementsByTagName("script")[0].parentNode.appendChild(script);
</script>
```

### datosOperacion (required)

| Field | Required | Description |
|-------|----------|-------------|
| `convenio` | Yes | Agreement code |
| `ok_url` | Yes | Success redirect URL |
| `error_url` | Yes | Error redirect URL |
| `monto_operacion` | Yes | Amount |
| `numero_operacion` | No | Your operation ID |

### atributos (optional)

| Field | Default | Description |
|-------|---------|-------------|
| `elemento_destino` | `"elementoEpagos"` | Target DOM element ID |
| `label` | `"Pagar"` | Button text |
| `id` | `"_Btn_epagos"` | Button element ID |
| `className` | none | CSS class |
| `style` | none | Inline CSS |

### estiloTooltip (optional)

| Field | Description |
|-------|-------------|
| `background_color` | Tooltip background (hex) |
| `font_family` | Font family |
| `font_color` | Text color |
| `font_size` | Font size |

Multiple buttons: call `botonPago()` once per button with different `elemento_destino`.

---

## Test Cards (quick reference)

| Provider | Number | CVV | Expiry |
|----------|--------|-----|--------|
| VISA (credit) | 4507 9900 0000 4905 | 123 | 12/2026 |
| AMEX (credit) | 3766 341249 71005 | 1234 | 12/2026 |
| Mastercard (credit) | 5323 6222 7777 7785 | 123 | 12/2026 |
| VISA (debit) | 4517 7210 0485 6075 | 123 | 12/2026 |
| MC (debit) | 5299 9100 1000 0015 | 123 | 12/2026 |

Expiry not validated in sandbox. Force rejection: Titular = "CALL".

> For DEBIN test CBU/CUIT, see `epagos-reference`.
