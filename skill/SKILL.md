---
name: epagos
description: ePagos Argentina payment gateway - complete API reference, integration patterns, and implementation guide for your project
---

# ePagos Payment Gateway - Complete Reference

## Overview

ePagos is an Argentine PSP (Payment Service Provider) regulated by BCRA. It consolidates 30+ payment methods (credit/debit cards, bank transfers, cash, QR, digital wallets) into a single integration.

- **Website:** epagos.com / epagos.com.ar
- **Support:** soporte@epagos.com.ar
- **Portal:** portal.epagos.net
- **No Node.js SDK exists** - we built our own TypeScript client

---

## Integration Options (3 approaches)

### Option A: Payment Button (JS only, simplest)
- Client-side only, loads `epagos.min.js`
- No backend needed for payment initiation
- Uses a **clave publica** (public key)
- ePagos hosts the payment form

### Option B: eCheckout (POST redirect) - **RECOMMENDED**
- Backend gets token via HTTP POST
- Redirects user to ePagos-hosted payment page
- User pays on ePagos, gets redirected back to ok_url/error_url
- No PCI certification needed (ePagos handles card data)

### Option C: SOAP API (full server-side)
- Direct server-to-server via SOAP/WSDL
- **Requires PCI-DSS** for card payments
- Use for querying payments, settlements, chargebacks (not for card processing)

---

## URLs

| Purpose | Sandbox | Production |
|---------|---------|------------|
| Token (POST) | `https://sandbox.epagos.com/post.php` | `https://api.epagos.com/post.php` |
| User redirect | `https://postsandbox.epagos.com` | `https://post.epagos.com` |
| SOAP WSDL | `https://sandbox.epagos.net/wsdl/2.0/index.php?wsdl` | `https://api.epagos.net/wsdl/2.0/index.php?wsdl` |
| JS library | `https://sandbox.epagos.com/quickstart/epagos.min.js` | `https://api.epagos.com/quickstart/epagos.min.js` |
| Portal (sandbox) | `https://portalsandbox.epagos.com` | `https://portal.epagos.com` |
| Logos | `https://sandbox.epagos.com/logos.php` | `https://api.epagos.com/logos.php` |
| Status page | `https://status.epagos.com.ar/` | |

**Note:** Both `.com` and `.net` domains appear in docs. Official docs use `.com`; PHP SDK uses `.net`. Both seem valid.

---

## Sandbox Credentials

| Key | Value |
|-----|-------|
| ID Organismo | `<provided by ePagos>` |
| ID Usuario | `<provided by ePagos>` |
| Hash | `<provided by ePagos>` |
| Password | `<provided by ePagos>` |
| Convenio (checkout) | `<provided by ePagos>` |
| Convenio (emision masiva) | `<provided by ePagos>` |

Request sandbox credentials from your ePagos sales rep. Production credentials go in Sealed Secrets, never in code.

---

## eCheckout Flow (for your project)

```
User clicks "Pay" → Backend gets token → Backend creates hidden form →
User is redirected to ePagos → User pays → ePagos redirects to ok_url/error_url →
Backend receives POST with result → Backend confirms via webhook/API
```

### Step 1: Get Token (backend)

```
POST https://sandbox.epagos.net/post.php
Content-Type: application/x-www-form-urlencoded

id_usuario=YOUR_USER_ID&id_organismo=YOUR_ORG_ID&password=YOUR_PASSWORD&hash=YOUR_HASH
```

Response: `{"token": "abc123..."}`

**Note:** A token from the POST endpoint is for eCheckout only. A token from SOAP `obtener_token` is for API calls only. They are different.

### Step 2: Redirect User (frontend)

Create a hidden form that POSTs to `https://postsandbox.epagos.net`:

```
version = "2.0"
operacion = "op_pago"
id_organismo = YOUR_ORG_ID
token = <token from step 1>
convenio = <convenio>
numero_operacion = <your order ID>
id_moneda_operacion = 1
monto_operacion = 480.00
detalle_operacion = URL-encoded JSON array of items
ok_url = https://yourdomain.com/payment/ok
error_url = https://yourdomain.com/payment/error
```

#### detalle_operacion structure (JSON, then URL-encoded):
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

**Sum of (monto_item * cantidad_item) MUST equal monto_operacion.**

Set `detalle_operacion_visible = 1` to show items on the payment page.

#### Optional POST fields:
| Field | Description |
|-------|-------------|
| `fp_permitidas` | `serialize([1, 2, 3])` - Only allow these payment method IDs |
| `fp_excluidas` | `serialize([1, 2, 3])` - Exclude these payment method IDs |
| `tp_excluidos` | `serialize([1, 2])` - Exclude these payment type IDs |
| `opc_fecha_vencimiento` | `YYYY-MM-DD` - Due date for cash/homebanking payments |
| `detalle_operacion_visible` | `1` - Show item details on checkout page |

#### All eCheckout POST options:
| Field | Type | Description |
|-------|------|-------------|
| `opc_pdf` | boolean | Return PDF receipt in response (default: true) |
| `opc_devolver_qr` | boolean | Return QR image for cash/homebanking (default: true) |
| `opc_devolver_codbarras` | boolean | Return barcode image (default: false) |
| `opc_comision` | boolean | Return commission value (default: false) |
| `opc_fecha_vencimiento` | YYYY-MM-DD | Due date for cash payments (must be > today) |
| `opc_email_automatico` | boolean | Ask user for email if not provided (default: true) |
| `opc_descargar_pdf` | boolean | Show PDF download interface (default: false) |
| `opc_totem` | boolean | Show on-screen keyboard for kiosk/totem (default: false) |
| `opc_una_cuota` | boolean | Force single installment for credit cards (default: false) |
| `opc_guardado_obligatorio` | boolean | Force user to save payment method for recurrence |
| `opc_T30_monto_abierto` | boolean | Open amount QR (default: false) |
| `opc_T30_reutilizable` | boolean | Reusable QR (default: false) |
| `opc_T30_requiere_orden` | boolean | QR requires payment order (default: false) |
| `opc_operaciones_lote` | text | Comma-separated id_transaccion to auto-credit when this op is credited |
| `fp_permitidas` | text | Comma-separated or serialized array of allowed payment method IDs |
| `fp_excluidas` | text | Comma-separated or serialized array of excluded payment method IDs |
| `tp_excluidos` | text | Comma-separated or serialized array of excluded payment type IDs |
| `fp_defecto` | int | Default/preferred payment method ID |
| `fecha_2do_venc` | YYYY-MM-DD | Second due date |
| `monto_operacion_2do_venc` | decimal | Amount for second due date |

#### Payer data (optional, sent in POST):
| Field | Description |
|-------|-------------|
| `nombre_pagador`, `apellido_pagador` | Name |
| `email_pagador` | Email |
| `tipo_doc_pagador`, `numero_doc_pagador` | Document |
| `cuit_doc_pagador` | CUIT/CUIL |
| `cbu_pagador` | CBU for DEBIN |

### Step 3: Handle Callback (backend)

**IMPORTANT:** `ok_url` does NOT guarantee the operation is credited. It can be `02002` (pending). For cards/wallets it's instant (`02001`). For cash/homebanking/transfer it's deferred (`02002`). Always verify via webhook or API.

#### Callback for CARDS (ok_url) — POST fields:

| Field | Type | Description |
|-------|------|-------------|
| `id_resp` | int | Response code (02001=credited, 02002=pending) |
| `respuesta` | text | Response description |
| `convenio` | int | Agreement number |
| `fp` | JSON text | `{nombre_fp, tarjeta_fp, importe_fp, importe_original_fp, respuesta_entidad_cobro}` |
| `id_organismo` | int | Organization ID |
| `id_transaccion` | int | ePagos transaction ID |
| `id_fp` | int | Payment method ID chosen by user |
| `numero_operacion` | text | Your order ID (as sent) |
| `identificador_2` | text | Your identificador_externo_2 (as sent) |
| `identificador_3` | text | Your identificador_externo_3 (as sent) |
| `identificador_4` | text | Your identificador_externo_4 (as sent) |
| `token` | text | The token used to initiate |

#### Callback for CASH (ok_url) — additional fields:

| Field | Type | Description |
|-------|------|-------------|
| `pdf` | text | Base64-encoded PDF payment slip |
| `codigo_barras_fp` | int | Barcode number |
| `codigo_pago_fp` | text | Payment code (no-invoice code) |
| `codigo_qr` | text | Base64-encoded QR image |
| `fp` | JSON text | `{nombre_fp, importe_fp, respuesta_entidad_cobro, fechavenc_fp}` |

#### Callback for HOMEBANKING/TRANSFER/DEBIN (ok_url) — additional fields:

| Field | Type | Description |
|-------|------|-------------|
| `pdf` | text | Base64-encoded PDF payment slip |
| `cuit_cuenta` | int | CUIT for homebanking publication or transfer source |
| `cbu_cuenta` | text | CBU/CVU for DEBIN |

#### Callback for ERROR (error_url) — POST fields:

| Field | Type | Description |
|-------|------|-------------|
| `id_resp` | int | Error response code |
| `respuesta` | text | Error description |
| `convenio` | int | Agreement number |
| `fp` | JSON text | `{nombre_fp, importe_fp, respuesta_entidad_cobro}` |
| `id_organismo` | int | Organization ID |
| `id_transaccion` | int | ePagos transaction ID |
| `numero_operacion` | text | Your order ID |
| `identificador_2`, `identificador_3` | text | Your identifiers |
| `token` | text | The token used |

#### eCheckout Response Codes (ok_url and error_url):

| Code | Type | Description |
|------|------|-------------|
| 02001 | OK | Pago acreditado (credited) |
| 02002 | OK | Pago pendiente (pending — cash/transfer) |
| 02003 | Error | Token validation error |
| 02004 | Error | Payment cancelled/rejected |
| 02005 | Error | Internal processing error |
| 02006 | Error | Parameter validation error: [param] |
| 02007 | Error | User cancelled payment |
| 02008 | Error | Amount/detail mismatch |
| 02009 | Error | Payment method [param] not available |
| 02010 | Error | Online service provider error |

### Step 4: Verify Payment (backend)

Use webhook (preferred) or API polling to confirm payment status.

---

## Webhook Integration (CRITICAL)

Configure in ePagos portal > Developers section. Can set one URL for both events or separate URLs.

### Webhook Payload (POST)

| Field | Type | Description |
|-------|------|-------------|
| `id_resp` | int | 02001 for payments. **NOT sent** for refunds |
| `id_transaccion` | int | ePagos transaction ID |
| `id_fp` | int | Payment method ID |
| `fp` | JSON text | Nested JSON with payment details (see below) |
| `id_organismo` | int | Organization ID |
| `convenio` | int | Agreement number |
| `numero_operacion` | text | Your order ID (as sent during payment creation) |
| `identificador_2` | text (100) | Your identifier 2 |
| `identificador_3` | text (512) | Your identifier 3 |
| `codigo_barras` | text | Barcode of the operation |
| `fecha_pago` | YYYY-MM-DD | Payment date (only for tipo=P) |
| `fecha_devolucion` | YYYY-MM-DD | Refund date (only for tipo=D) |
| `monto_pagado` | decimal | Amount paid |
| `tipo` | char | **P** = Payment, **D** = Refund |

### Webhook `fp` nested JSON:

| Field | Description |
|-------|-------------|
| `nombre_fp` | Payment method name |
| `id_fp` | Payment method ID |
| `tarjeta_fp` | Card number (hashed) — only for card payments |
| `importe_fp` | Amount paid |
| `tipo_doc_fp` | Payer document type (CUIT/DNI) — not all methods |
| `numero_doc_fp` | Payer document number — not all methods |

### Webhook verification

**ePagos does NOT provide HMAC or cryptographic verification.** Verify by:
1. Check `tipo` field: P=payment, D=refund
2. Check `id_resp` = 02001 for payments
3. Cross-reference `id_transaccion` with your DB
4. **ALWAYS** implement daily reconciliation via `obtener_pagos` as fallback

### Acreditación timing by payment method:

| Method | Timing | ok_url means |
|--------|--------|-------------|
| Credit/Debit cards | Immediate | Credited (02001) |
| Billetera ePagos | Immediate | Credited (02001) |
| Cash (PagoFacil, Rapipago) | Deferred | Pending (02002) — user still needs to pay |
| Homebanking | Deferred | Pending (02002) |
| Transfer/DEBIN | Deferred | Pending (02002) |

---

## Payment Button (JS) Integration

**Requirements:**
1. Request a **clave publica** (public key) from ePagos, specifying your domain(s)
2. Include the JS library

**Error handling:** Use `errorEpagos` event:
```javascript
window.addEventListener("errorEpagos", function(event) {
  console.error(event.detail.mensaje);
});
```

**Init methods:**
| Method | Description |
|--------|-------------|
| `ePagos.setClavePublica(key)` | Set public key credential |
| `ePagos.setOrganismo(id)` | Set organization ID |
| `ePagos.setAmbiente(env)` | `"T"` = Test/Sandbox, `"P"` = Production |

**datosOperacion (required):**
| Field | Required | Description |
|-------|----------|-------------|
| `convenio` | Yes | Agreement code |
| `ok_url` | Yes | Success redirect URL |
| `error_url` | Yes | Error redirect URL |
| `monto_operacion` | Yes | Amount |
| `numero_operacion` | No | Your operation ID |

**atributos (optional):**
| Field | Default | Description |
|-------|---------|-------------|
| `elemento_destino` | `"elementoEpagos"` | Target DOM element ID |
| `label` | `"Pagar"` | Button text |
| `id` | `"_Btn_epagos"` | Button element ID |
| `className` | none | CSS class |
| `style` | none | Inline CSS |

**estiloTooltip (optional):**
| Field | Description |
|-------|-------------|
| `background_color` | Tooltip background (hex) |
| `font_family` | Font family |
| `font_color` | Text color |
| `font_size` | Font size |

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
        "ok_url": "https://yourdomain.com/payment/ok",
        "error_url": "https://yourdomain.com/payment/error",
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

**Multiple buttons:** Call `botonPago()` once per button with different `elemento_destino`.

---

## SOAP API Methods

All SOAP methods require a token from `obtener_token` first (different from POST token).

### obtener_token

```
Request:
  version: "2.0"
  credenciales:
    id_organismo: YOUR_ORG_ID
    id_usuario: YOUR_USER_ID
    password: "YOUR_PASSWORD"
    hash: "YOUR_HASH"

Response:
  id_resp: 01001 (success) | 01002 (invalid credentials) | 01003 (internal error) | 01004 (invalid version)
  respuesta: text
  token: string
```

### solicitud_pago (create payment via API - non-card only without PCI)

```
Request:
  version: "2.0"
  tipo_operacion: "op_pago"
  credenciales: { id_organismo, token }
  operacion:
    numero_operacion: string (your order ID, optional)
    identificador_externo_2: string (100 chars, optional)
    identificador_externo_3: string (512 chars, optional)
    identificador_externo_4: string (65535 chars, optional)
    identificador_cliente: string (for recurrence, optional)
    id_moneda_operacion: 1 (ARS, required)
    monto_operacion: decimal (required)
    opc_pdf: boolean (include PDF in response, default true)
    opc_fecha_vencimiento: "YYYY-MM-DD" (due date, optional)
    opc_devolver_qr: boolean (return QR image, default false)
    opc_devolver_codbarras: boolean (return barcode, default false)
    opc_generar_pdf: boolean (generate PDF ticket, default true)
    opc_fp_excluidas: "1,2,3" (excluded payment method IDs)
    opc_tp_excluidos: "1,2" (excluded payment type IDs)
    opc_fp_permitidas: "1,2,3" (allowed payment method IDs)
    url_ok: string (redirect on success)
    url_error: string (redirect on error)
    detalle_operacion: array of items (required):
      - id_item: integer
      - desc_item: string
      - monto_item: decimal
      - cantidad_item: integer
    pagador: (required)
      - nombre_pagador, apellido_pagador, email_pagador
      - identificacion_pagador: { tipo_doc_pagador, numero_doc_pagador, cuit_doc_pagador }
      - domicilio_pagador: { calle, numero, adicional, cp, ciudad, provincia, pais }
      - telefono_pagador: { codigo, numero }
  fp: array of payment methods:
    - id_fp: integer (payment method ID)
    - monto_fp: decimal
    - tarjeta: { numero, banco, vencimiento: {mes, anio}, codigo_seg, cuotas, titular, identificacion, fechanac, direccion }
  convenio: integer
```

### obtener_pagos (query payments)

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  pago (search criteria, all optional):
    CodigoUnicoTransaccion: integer (ePagos transaction ID)
    ExternoId: string (your numero_operacion)
    ExternoId_2: string (100)
    ExternoId_3: string (512)
    Estado: "A"|"O"|"V"|"C"|"R"|"P"|"D"|"L"
    CodBarras: string
    Implementador: integer
    FechaPagoDesde: "YYYY-MM-DD"
    FechaPagoHasta: "YYYY-MM-DD"
    FechaAcreditacionDesde: "YYYY-MM-DD HH:MM:SS"
    FechaAcreditacionHasta: "YYYY-MM-DD HH:MM:SS"
    FechaNovedadAcreditacionDesde: "YYYY-MM-DD HH:MM:SS" (RECOMMENDED for daily queries)
    FechaNovedadAcreditacionHasta: "YYYY-MM-DD HH:MM:SS"
    DevolverID4: boolean (default false)
    Pagina: integer (100 results per page, starts at 1)

Response:
  id_resp, respuesta, token, id_organismo, pagina, cantidadTotal
  pagos: array of:
    CodigoUnicoTransaccion: int (ePagos transaction ID)
    Externa, Externa_2, Externa_3: string (your identifiers)
    ExternoId_4: string (only if DevolverID4=true)
    Convenio: int
    Importe: decimal (amount without financing costs)
    Estado: char (A/O/V/C/R/P/D)
    FechaPago: datetime (payment initiation)
    FechaAcreditacion: datetime (credit at payment method)
    FechaNovedadAcreditacion: datetime (credit at ePagos)
    FormaPago: array of { Identificador, Tipo, Importe, CodigoPago, CodigoBarras }
      Tipo values: tipo_fp_presencial, tipo_fp_credito, tipo_fp_debito,
                   tipo_fp_prepago, tipo_fp_publicacion, tipo_fp_billetera,
                   tipo_fp_transferencia, tipo_fp_otros
    DatosPagador: { Nombre, Apellido, Email, Identificacion: {tipo_doc, numero_doc, cuit} }
    PagosAdicionales: array of { CodigoUnicoTransaccion, FormaPago, Monto, FechaPago, FechaNovedad, IdPago }
    Recibo: URL (receipt download)
    Url_QR: URL (online payment)

Error codes: 04001=OK | 04002=invalid token | 04003=internal | 04004=date range exceeded | 04005=invalid param | 04006=invalid version
```

### obtener_pagos_adicionales (additional payments on already-credited ops)

```
Request:
  credenciales: { id_organismo, token }
  criteria: { fecha_desde: "YYYY-MM-DD", fecha_hasta: "YYYY-MM-DD" }

Response:
  pagos_adicionales: array of { CodigoUnicoTransaccion, FormaPago, Monto, FechaPago, FechaNovedad, IdPago }

Error codes: 07001=OK | 07002=invalid token | 07003=internal | 07004=invalid dates | 07005=invalid param | 07006=invalid version
```

### obtener_rendiciones (daily settlements)

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  rendicion:
    numero: integer (unique settlement number)
    secuencia: integer (incremental per org)
    fecha_desde: "YYYY-MM-DD"
    fecha_hasta: "YYYY-MM-DD"
    fecha_deposito_desde: "YYYY-MM-DD"
    fecha_deposito_hasta: "YYYY-MM-DD"

Response:
  rendicion: array of:
    Numero, Secuencia, Convenio
    Estado: "A" (Active) | "D" (Deposited)
    Fecha_desde, Fecha_hasta, Fecha_estimada_deposito, Fecha_deposito
    Monto (gross), Monto_depositado, Monto_comision, Monto_IVA, Monto_IIBB
    Monto_CC (chargebacks), Monto_ND (non-depositable)
    Cantidad (transaction count)
    Contenido: URL (ZIP file download)
    Detalles: array of { Codigo_unico_transaccion, Monto, Numero_operacion, Depositable }
    Contracargos: array of { Codigo_unico_transaccion, Monto, Numero_rendicion }
    Devoluciones: array of { Codigo_unico_transaccion, Monto, Numero_rendicion }

Error codes: 05001=OK | 05002=invalid token | 05003=internal | 05004=date range exceeded | 05005=invalid param
```

### obtener_contracargos (chargebacks)

```
Request:
  credenciales: { id_organismo, token }
  contracargos:
    numero: integer
    estado: "R"(Respondido)|"C"(Confirmado)|"P"(Pendiente)|"F"(Finalizado)|"FNF"(No Favorable)|"B"(Baja)
    fecha_desde, fecha_hasta: "YYYY-MM-DD"

Response:
  contracargos: array of:
    Numero, Estado, Medio, Transaccion, Monto
    Tarjeta, Lote, Cupon
    Respuesta (base64), Respuesta_formato (file ext)
    Comprobante (base64), Comprobante_formato (file ext)
    Fecha, Fecha_vencimiento, Fecha_resolucion, Fecha_confirmacion, Fecha_finalizacion

Error codes: 06001=OK | 06002=invalid token | 06003=internal | 06004=invalid dates | 06005=invalid param | 06006=invalid version
```

### obtener_devoluciones (refunds)

Uses `obtener_pagos` with `Estado = 'D'` filter internally.

---

## Payment States

| Code | State | Description |
|------|-------|-------------|
| A | Acreditada | Credited/Paid successfully |
| O | Adeudada | Owed - pending payment (cash/transfer) |
| V | Vencida | Expired - past due date |
| C | Cancelada | Cancelled by user |
| R | Rechazada | Rejected by payment method |
| P | Pendiente | Request initiated, no payment method selected yet |
| D | Devuelta | Refunded to user |
| L | Lote Acreditado | Batch credit |

---

## Payment Verification (4 Levels)

### Level 1: Callback (redirect)
After payment, user is redirected to ok_url/error_url. For **instant** methods (cards, wallets), you can mark as paid. For **deferred** methods (cash, transfer), operation is "adeudada" until actually paid.

### Level 2: Webhook (real-time, recommended)
Configure a URL in ePagos portal. ePagos POSTs payment notifications in real-time for ALL payment methods. Configure at: portal.epagos.net > Developers section.

**IMPORTANT:** ePagos recommends implementing a daily reconciliation process using `obtener_pagos` as fallback, since webhook notifications may fail to be delivered or processed. Never rely solely on webhooks.

### Level 3: API Polling
Use `obtener_pagos` in a background job to query payment status. Use `FechaNovedadAcreditacionDesde/Hasta` for daily reconciliation.

### Level 4: Settlement (rendicion)
Daily settlement files with all credited operations. Available via `obtener_rendiciones` API or email/portal download.

---

## Test Cards

### Credit
| Provider | Number | CVV | Expiry |
|----------|--------|-----|--------|
| VISA | 4507 9900 0000 4905 | 123 | 12/2026 |
| AMEX | 3766 341249 71005 | 1234 | 12/2026 |
| Mastercard | 5323 6222 7777 7785 | 123 | 12/2026 |

### Debit
| Provider | Number | CVV | Expiry |
|----------|--------|-----|--------|
| VISA Debito | 4517 7210 0485 6075 | 123 | 12/2026 |
| MC Debito | 5299 9100 1000 0015 | 123 | 12/2026 |

**Note:** Expiry date is not validated in sandbox. Any future date works.

### Test rejection
Set **Nombre y Apellido Titular = "CALL"** in checkout to trigger a rejection.

### Test CBU/CUIT (for DEBIN/transfers)
| CBU | CUIT | Type |
|-----|------|------|
| 3220002204000040970011 | 20123456781 | Fisica |
| 3220001101000040970011 | 27123456781 | Fisica |
| 0070314530004001400167 | 20004940548 | Fisica |

---

## Payment Method IDs (most relevant for your project)

### Online - Cards
| ID | Provider | Type |
|----|----------|------|
| 1 | VISA | Credit |
| 2 | AMEX | Credit |
| 3 | Mastercard | Credit |
| 10 | Cabal | Credit |
| 11 | Diners Club | Credit |
| 54 | Naranja X | Credit |
| 14 | VISA Debito | Debit |
| 15 | Maestro Debito | Debit |
| 41 | Mastercard Debito | Debit |

### Digital Wallets
| ID | Provider | Type |
|----|----------|------|
| 27 | Billetera ePagos (*) | Wallet |
| 48 | Billetera MODO (*) | Wallet |
| 47 | Billetera MercadoPago (*) | Wallet |

### Transfers
| ID | Provider | Type |
|----|----------|------|
| 17 | E-Transferencia (*) | Bank transfer |
| 44 | Transferencias 3.0 (*) | Bank transfer (min $900) |
| 38 | Debin (*) | Transfer |

### Cash
| ID | Provider | Type |
|----|----------|------|
| 4 | Pago Facil | Cash |
| 5 | Rapipago | Cash |
| 24 | Cobro Express | Cash |

(*) Not available for all organizations

### Payment Type IDs
| ID | Type |
|----|------|
| 1 | Cash (presencial) |
| 2 | Credit card |
| 3 | Debit card |
| 5 | Homebanking |
| 6 | Billetera ePagos |
| 7 | Bank transfer |
| 9 | Crypto |

---

## Minimum Amounts

| Method | Minimum |
|--------|---------|
| Credit cards | $40 ARS |
| Debit cards | No minimum |
| Transferencia 3.0 | $900 ARS |
| Billetera ePagos | No minimum |

---

## External Identifiers (for tracking)

ePagos provides 4 external identifier fields per operation:

| Field | Max Length | Recommended use |
|-------|-----------|--------------------------|
| `numero_operacion` | 100 | Order ID (e.g., `order_123`) |
| `identificador_externo_2` | 100 | Session ID (e.g., `session_456`) |
| `identificador_externo_3` | 512 | Restaurant ID + metadata |
| `identificador_externo_4` | 65535 | Full order JSON for reconciliation |

---

## Logo Requirements

For the payment page branding:
- Format: JPG
- Width: 142px
- Height: max 42px

---

## Error Response Codes

### obtener_token
| Code | Type | Description |
|------|------|-------------|
| 01001 | OK | Token generado |
| 01002 | Error | Credenciales invalidas |
| 01003 | Error | Error interno |
| 01004 | Error | Version invalida |

### generar_qr_vinculado
| Code | Type | Description |
|------|------|-------------|
| 14001 | OK | QR generado con exito |
| 14002 | Error | Error al registrar operacion |
| 14003 | Error | Operacion no pertenece al organismo |
| 14004 | Error | Hash ya registrado |
| 14005 | Error | Operaciones duplicadas |
| 14006 | Error | Version invalida |

---

## Architecture Decision for your project

### Chosen approach: eCheckout (POST redirect)

**Why:**
- No PCI certification needed
- ePagos handles all card UI and security
- Works with all payment methods (cards, wallets, transfers, cash)
- User pays on a trusted ePagos page
- Simple redirect flow compatible with our web app

**Typical flow:**
1. Customer completes order → clicks "Pay with ePagos"
2. Backend creates a `payment` record in DB with order details
3. Backend calls `POST sandbox.epagos.net/post.php` to get token
4. Backend returns a redirect URL/form data to frontend
5. Frontend redirects user to ePagos checkout
6. User pays → ePagos redirects to our `ok_url` or `error_url`
7. Backend receives callback POST, updates payment status
8. Webhook confirms payment asynchronously (for cash/transfer methods)
9. Background job polls `obtener_pagos` for reconciliation

**What we need to build:**
- `backend/src/services/epagos.service.ts` - ePagos API client
- `backend/src/controllers/payments.controller.ts` - Payment endpoints
- `backend/src/routes/payments.routes.ts` - Routes
- `backend/src/models/payments.model.ts` - DB model
- DB migration for `payments` table
- `web/src/pages/customer/PaymentResultPage.tsx` - ok/error landing pages
- Webhook endpoint for async payment notifications

---

## Daily Reconciliation Process (MANDATORY)

ePagos recommends a daily automated process. Run at 02:00-06:00 (off-peak).

```
1. obtener_token → get fresh token (~1h cache)
2. obtener_pagos → filter FechaNovedadAcreditacion last 24-48h, paginate if >100
3. Process each payment → update internal DB
4. obtener_pagos_adicionales → catch payments on already-credited ops
5. obtener_contracargos → monitor disputes (states P, C, R)
6. Generate report / alerts
```

**Key rules:**
- Max date range: 48 hours per query
- 100 results per page, use `Pagina` to iterate
- Token can be reused for ~1 hour
- Always retry on 04003/05003/06003/07003 (internal errors) after 60s

### solicitud_pago API response (for reference)

The API `solicitud_pago` response includes:
- `id_transaccion` — ePagos unique ID
- `fp[].codigo_pago_fp` — Short payment code (cash/homebanking)
- `fp[].codigo_barras_fp` — Full barcode
- `fp[].fechavenc_fp` — Due date
- `fp[].importe_fp` — Amount
- `fp[].pdf` — Base64 PDF receipt (if opc_pdf=true)
- `fp[].codigo_barras_imagen` — Barcode PNG (if opc_devolver_codbarras=true)
- `fp[].qr_imagen` — QR PNG (if opc_devolver_qr=true)
- `fp[].qr_imagen_T30` — Transfers 3.0 QR PNG
- `fp[].url_qr` — URL for online card payment
- `fp[].codigo_pmc` — Pago Mis Cuentas code
- `fp[].codigo_link` — Red Link code
- `fp[].cuit_cuenta`, `fp[].cbu_cuenta` — For DEBIN/transfer

---

## Recurring Payments (for future subscription billing)

ePagos supports recurring charges on stored cards/accounts. Useful for billing restaurants.
**Not available for all orgs — must request from ePagos sales rep.**

### Flow:
1. First payment with `identificador_cliente` (min 6 chars) + `opc_guardado_obligatorio=true`
2. User pays and saves their card on ePagos
3. `obtener_tarjetas_cliente` → get `identificador_tarjeta`
4. `solicitud_pago_recurrente` → charge stored card (one-time)
5. `solicitud_pago_recurrente_suscripcion` → schedule recurring (M=monthly, S=weekly, A=annual, P=custom)

### Key parameters:
- `medio`: `op_pago_recurrente_medio_tarjeta` | `op_pago_recurrente_medio_debin` | `op_pago_recurrente_medio_debito_directo`
- `cliente`: `{ identificador_cliente, identificador_tarjeta }` or `{ identificador_cliente, identificador_cuenta }`
- `suscripcion_modalidad`: M/S/A/P
- `suscripcion`: array of `{ fecha, monto }` for each charge

### Error codes: 09xxx (recurrente), 10xxx (tarjetas/cuentas), 11xxx (suscripcion), 15002 (not authorized)
### Card rejection codes: `cc_rejected_blacklist`, `cc_rejected_card_disabled`, `cc_rejected_bad_filled_date`, `cc_rejected_bad_filled_other` — do NOT retry these.

Full docs: `docs/epagos-sandbox/ePagos_Documentacion_Pagos_Recurrentes.md`

---

## PHP SDK Reference (for implementation guidance)

The official PHP SDK at github.com/epagos/api_php shows:
- SOAP client instantiation with `SoapClient($wsdl, options)`
- Token must be obtained before every API call
- POST token (for eCheckout) uses `curl` to `post.php`
- SOAP token (for API queries) uses `SoapClient->obtener_token()`
- These are DIFFERENT tokens for different purposes
- `solicitud_pago_post()` generates a self-submitting HTML form
- `solicitud_pago_post_iframe()` does the same but targets an iframe
- Refunds query uses `obtener_pagos` with `Estado = 'D'`
