---
name: epagos
description: ePagos Argentina payment gateway - complete API reference, integration patterns, and implementation guide for Aramal
---

# ePagos Payment Gateway - Complete Reference

## Table of Contents

1. [Overview](#overview)
2. [Integration Options](#integration-options)
3. [URLs](#urls)
4. [Sandbox Credentials](#sandbox-credentials)
5. [eCheckout Flow](#echeckout-flow)
6. [Webhook Integration](#webhook-integration)
7. [Payment Button JS](#payment-button-js)
8. [SOAP API - Core Methods](#soap-api---core-methods)
9. [SOAP API - Payment Entities](#soap-api---payment-entities)
10. [SOAP API - Batch Operations](#soap-api---batch-operations)
11. [SOAP API - Recurring & Subscriptions](#soap-api---recurring--subscriptions)
12. [SOAP API - QR Methods](#soap-api---qr-methods)
13. [SOAP API - POS Terminals](#soap-api---pos-terminals)
14. [Payment States](#payment-states)
15. [Payment Verification (4 Levels)](#payment-verification-4-levels)
16. [Operation Type Codes](#operation-type-codes)
17. [Reference Tables](#reference-tables)
18. [Test Cards & Data](#test-cards--data)
19. [Minimum Amounts](#minimum-amounts)
20. [Error Response Codes](#error-response-codes)
21. [Architecture Decision](#architecture-decision)
22. [Daily Reconciliation](#daily-reconciliation)
23. [Test Scenarios Checklist](#test-scenarios-checklist)
24. [PHP SDK Reference](#php-sdk-reference)

---

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

### Option B: eCheckout (POST redirect) - **RECOMMENDED FOR ARAMAL**
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
| ID Organismo | 22812 |
| ID Usuario | 2989 |
| Hash | 9f082f37d55f18a06385b3ad0521c4ae |
| Password | ced10222e76aa7f99824dd28455f4bbe |
| Convenio (checkout) | null |
| Convenio (emision masiva) | 24812 |

**IMPORTANT:** Production credentials go in Sealed Secrets, never in code.

---

## eCheckout Flow (for Aramal)

```
User clicks "Pay" → Backend gets token → Backend creates hidden form →
User is redirected to ePagos → User pays → ePagos redirects to ok_url/error_url →
Backend receives POST with result → Backend confirms via webhook/API
```

### Step 1: Get Token (backend)

```
POST https://sandbox.epagos.net/post.php
Content-Type: application/x-www-form-urlencoded

id_usuario=2989&id_organismo=22812&password=ced10222e76aa7f99824dd28455f4bbe&hash=9f082f37d55f18a06385b3ad0521c4ae
```

Response: `{"token": "abc123..."}`

**Note:** A token from the POST endpoint is for eCheckout only. A token from SOAP `obtener_token` is for API calls only. They are different.

### Step 2: Redirect User (frontend)

Create a hidden form that POSTs to `https://postsandbox.epagos.net`:

```
version = "2.0"
operacion = "op_pago"
id_organismo = 22812
token = <token from step 1>
convenio = <convenio>
numero_operacion = <your order ID>
id_moneda_operacion = 1
monto_operacion = 480.00
detalle_operacion = URL-encoded or base64-encoded JSON array of items
ok_url = https://aramal.co/payment/ok
error_url = https://aramal.co/payment/error
```

#### detalle_operacion structure (JSON, then URL-encoded or base64-encoded):
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

**Encoding:** Use `urlencode(JSON)` or `base64_encode(JSON)` — both are accepted.

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

### Acreditacion timing by payment method:

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
    ePagos.setOrganismo('22812');
    ePagos.setAmbiente("T");

    ePagos.botonPago({
      "datosOperacion": {
        "convenio": "00000",
        "ok_url": "https://aramal.co/payment/ok",
        "error_url": "https://aramal.co/payment/error",
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

## SOAP API - Core Methods

All SOAP methods require a token from `obtener_token` first (different from POST token).

### obtener_token

```
Request:
  version: "2.0"
  credenciales:
    id_organismo: 22812
    id_usuario: 2989
    password: "ced10222e76aa7f99824dd28455f4bbe"
    hash: "9f082f37d55f18a06385b3ad0521c4ae"

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

### solicitud_pago response (for reference)

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
  version: "2.0"
  credenciales: { id_organismo, token }
  criteria:
    fecha_desde: "YYYY-MM-DD" (required)
    fecha_hasta: "YYYY-MM-DD" (required)

Response:
  pagos_adicionales: array of:
    CodigoUnicoTransaccion: int
    FormaPago: text
    Monto: decimal
    FechaPago: datetime
    FechaNovedad: datetime (when ePagos received the payment notification)
    IdPago: int

Error codes: 07001=OK | 07002=invalid token | 07003=internal | 07004=invalid dates | 07005=invalid param | 07006=invalid version
```

**Note:** `FechaNovedad` is the date ePagos learned about the payment, not the date the user paid. Use this for reconciliation date ranges.

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

## SOAP API - Payment Entities

### obtener_entidades_pago (get available payment methods)

Returns available payment methods for the organization with logos, config, limits.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  fp: array of { id_fp } (optional filter — only return these methods)

Response:
  id_resp, respuesta, token, id_organismo
  fp: array of:
    id_fp: int (payment method ID)
    nombre_fp: text (name)
    tipo_fp: set (tipo_fp_presencial | tipo_fp_credito | tipo_fp_debito | tipo_fp_prepago | tipo_fp_publicacion | tipo_fp_billetera | tipo_fp_transferencia)
    estado_fp: set (estado_fp_activo | estado_fp_desactivado)
    logos_fp: { seguro_logos_fp (HTTPS URL), nombre_logos_fp (HTTP URL) }
    config_fp: { patron_config_fp, longitud_config_fp, validacion_config_fp, codseg_config_fp (bool), codseg_long_config_fp, codseg_ubic_config_fp }
    adicional_fp: text (extra info to request from user, e.g. CBU, patente)
    monto_minimo_fp: decimal (minimum amount)
    monto_maximo_fp: decimal (maximum amount)
    tiempo_acreditacion_fp: int (accreditation hours)

Error codes: 03001=OK | 03002=invalid token | 03003=internal | 03004=invalid param | 03005=invalid version
```

**Use case:** Dynamically build a payment method selector in the frontend with logos and limits.

---

## SOAP API - Batch Operations

### solicitud_pago_lote (batch payment requests — up to 50)

Generates multiple payment operations in a single call. Equivalent to calling `solicitud_pago` multiple times.

```
Request:
  version: "2.0"
  tipo_operacion: "op_pago"
  credenciales: { id_organismo, token }
  lote: array of TipoLote (max 50):
    - fp: array of DatosFormaPago (same as solicitud_pago)
    - operacion: DatosOperacionPago (same as solicitud_pago)
    - convenio: int

Response:
  id_resp, respuesta, token, id_organismo
  lote: array of RespuestaLote:
    - id_transaccion: int (generated operation ID)
    - numero_operacion: text (your operation ID)
    - convenio: int
    - respuesta_forma_pago_array: array of RespuestaFormaPago (same as solicitud_pago response)

Error codes: 08001=OK | 08002=invalid token | 08003=internal | 08004=invalid param | 08005=batch size exceeds limit | 08006=processing error | 08007=amount/detail mismatch | 08008=invalid version
```

**Note:** Response array positions correspond to request array positions.

### pago_lote (mark batch as paid — link to credited base operation)

Marks multiple operations as paid, linking them to an already-credited base operation. The linked operations join the base operation's settlement batch.

> Not available for all organizations — requires authorization.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  pago_lote:
    id_transaccion: int (base operation — must be credited and NOT yet settled)
    forma_pago: int (payment method ID to assign)
    fecha_pago: date (payment date)
    importe: decimal (total amount — must equal sum of operation amounts)
    operaciones: array of:
      - id_transaccion: int (operation to mark as paid)
      - importe: decimal (amount for this operation)

Response:
  id_resp, respuesta

Error codes: 15001=OK | 15002=not authorized | 15003=invalid batch data | 15004=invalid param | 15005=invalid token
```

**Key constraint:** Base operation must be credited but NOT yet settled (rendida).

---

## SOAP API - Recurring & Subscriptions

> Not available for all organizations — must request from ePagos sales rep.

### Flow overview

```
1. solicitud_pago (with opc_guardado_obligatorio=true + identificador_cliente)
   → User pays and saves their card/account on ePagos
2. obtener_tarjetas_cliente / obtener_cuentas_cliente
   → Retrieve saved payment instrument identifiers
3. solicitud_pago_recurrente → charge stored card/account (one-time)
4. solicitud_pago_recurrente_suscripcion → schedule recurring charges
5. obtener_resultados_debito → verify bank response for processed debits
```

### obtener_tarjetas_cliente (get saved cards)

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  datosCliente: array of:
    - identificador_cliente: text (your client ID, min 6 chars)
    - identificador_tarjeta: text (optional — filter to specific card)
    - proximo_vencimiento: boolean (optional — only cards expiring soon)

Response:
  id_resp, respuesta, token, id_organismo
  tarjetas: array of TarjetasClientes:
    - identificador_cliente: text
    - tarjetas: array of:
      - identificador_tarjeta: text (use this in recurring charges)
      - marca: text (VISA, Mastercard, etc.)
      - id_fp: int (payment method ID)
      - fecha_alta: date
      - fecha_vencimiento: text (MM/YY)

Error codes: 10001=OK | 10002=invalid token | 10003=internal | 10004=invalid param | 10005=invalid version
```

### obtener_cuentas_cliente (get saved bank accounts)

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  datosCliente: array of:
    - identificador_cliente: text
    - medio: set (op_pago_recurrente_medio_debin | op_pago_recurrente_medio_debito_directo)
    - identificador_cuenta: text (optional — filter to specific account)

Response:
  id_resp, respuesta, token, id_organismo
  cuentas: array of CuentasClientes:
    - identificador_cliente: text
    - cuentas: array of:
      - identificador_cuenta: text (use this in recurring charges)
      - id_fp: int
      - tipo_operacion: int
      - cbu: text
      - email: text (optional)
      - medio: set
      - fecha_alta: date

Error codes: 10001=OK | 10002=invalid token | 10003=internal | 10004=invalid param | 10005=invalid version
```

### registrar_cuentas_cliente (bulk register bank accounts — up to 100)

Registers pre-existing bank account adhesions for direct debit, without requiring the client to go through a payment flow.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  cuentas: array of CuentaClienteAgregar (max 100):
    - identificador_cliente: text
    - tipo_operacion: int (operation type ID)
    - cuit: int (optional — validates CBU ownership if sent)
    - cbu: text (22 digits)
    - fecha_adhesion: date (original adhesion date)

Response:
  id_resp, respuesta
  cuentas: array of:
    - identificador_cliente: text
    - identificador_cuenta: text (ePagos-assigned ID — use in recurring charges)

Error codes: 16001=OK | 16002=invalid version | 16003=invalid param | 16004=invalid token | 16005=account adhesion error
```

**Uniqueness:** The combination `cbu` + `identificador_cliente` + `tipo_operacion` must be unique.

### solicitud_pago_recurrente_suscripcion (schedule recurring charges)

Plans recurring charges with configurable periodicity on saved cards/accounts.

```
Request:
  version: "2.0"
  tipo_operacion: "op_pago_recurrente"
  credenciales: { id_organismo, token }
  operacion: DatosOperacionPago (same as solicitud_pago)
  suscripcion: array of (min 1):
    - fecha: "YYYY-MM-DD" (must be future date)
    - monto: decimal (optional — overrides operacion.monto_operacion for this charge)
  suscripcion_modalidad: set
    - P = Personalizada (custom dates/amounts)
    - M = Mensual (monthly)
    - S = Semanal (weekly)
    - A = Anual (annual)
  descripcion: text (subscription name)
  convenio: int
  medio: set
    - op_pago_recurrente_medio_tarjeta (card)
    - op_pago_recurrente_medio_debin (DEBIN)
    - op_pago_recurrente_medio_debito_directo (direct debit)
  clientes: array of:
    - identificador_cliente: text
    - identificador_tarjeta: text (for card medio)
    - identificador_cuenta: text (for DEBIN/direct debit medio)

Response:
  id_resp, respuesta, token, id_organismo

Error codes: 11001=OK | 11002=invalid token | 11003=internal | 11004=invalid param | 11005=amount/detail mismatch | 11006=invalid date | 11007=card not found | 11008=account not found | 11009=invalid version | 15002=not authorized
```

**Card rejection codes (do NOT retry):** `cc_rejected_blacklist`, `cc_rejected_card_disabled`, `cc_rejected_bad_filled_date`, `cc_rejected_bad_filled_other`

### obtener_resultados_debito (bank response codes for processed debits)

Returns bank-level response codes for debits processed via `solicitud_pago_recurrente`. This is the debit-specific equivalent of reconciliation.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  datosDebito: array of int (operation numbers to query)

Response:
  id_resp, respuesta
  resultados: array of:
    - operacion: int (operation ID)
    - codigo: text (bank response code)
    - resultado: text (bank response description)

Error codes: 17001=OK | 17002=invalid version | 17003=invalid param | 17004=invalid token | 17005=invalid operation
```

**Note:** The `codigo` is the bank's response code, not an ePagos code. Poll periodically after debits are processed to catch bank rejections.

---

## SOAP API - QR Methods

### generar_qr_vinculado (linked QR for multiple operations)

Generates a single QR that acts as a selector — user scans and chooses which operation(s) to pay.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  operaciones_qr: array of:
    - id_transaccion: int (from solicitud_pago)
    - etiqueta: text (label shown to user in selector)

Response:
  id_resp, respuesta
  qr: base64 (PNG image — use as <img src="data:image/png;base64,{qr}" />)

Error codes: 14001=OK | 14002=registration error | 14003=operation not owned by org | 14004=hash already registered | 14005=duplicate operations | 14006=invalid version
```

**Key rules:**
- All operations must belong to the same organization
- No duplicate `id_transaccion` in the same request
- Each unique combination generates a unique hash — re-generating returns error 14004

### obtener_cajas_qr (list QR payment boxes)

Lists the static interoperable QR payment boxes configured for the organization (T3.0).

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }

Response:
  id_resp, respuesta
  cajas: array of:
    - id_caja: int
    - nombre_caja: text
    - tipo_caja: set (F=Fisica, V=Virtual)
    - monto_maximo_caja: decimal (max amount per operation)

Error codes: 18002=OK | 18003=invalid token | 18011=no boxes configured
```

### generar_orden_qr (create T3.0 payment order for a box)

Creates a dynamic payment order for a static QR box. The customer scans the box's static QR, sees the order, and pays via their wallet app (MODO, MercadoPago, etc.).

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  orden: array of:
    - id_caja: int (optional — ignored if id_transaccion sent)
    - id_transaccion: int (optional — existing T3.0 operation, overrides id_caja)
    - importe: decimal (> 0, 2 decimals)
    - concepto: text (150 chars, optional)
    - identificador_2: text (100, optional)
    - identificador_3: text (512, optional)
    - identificador_4: text (65535, optional)
    - email_pagador: text (255, optional)
    - detalle_orden: text (255, optional)
    - vencimiento: "YYYY-MM-DD" (optional)

Response:
  id_resp, respuesta

Error codes: 18001=OK | 18003=invalid token | 18004=exceeds box max amount | 18005=invalid param | 18006=invalid expiry date | 18007=bad date format | 18008=internal error | 18009=invalid box | 18010=amount must be > 0 | 18011=no boxes configured | 18012=invalid operation
```

#### QR Interoperable Flow:
```
1. obtener_cajas_qr → list boxes, get id_caja
2. Print static QR at physical/virtual payment points
3. On each payment: generar_orden_qr (id_caja + importe + metadata)
4. Customer scans static QR with wallet app (MODO, MercadoPago, etc.)
5. Customer sees the order and confirms payment
6. ePagos notifies via webhook
7. Reconcile with obtener_pagos
```

---

## SOAP API - POS Terminals

Methods for in-person card payments via physical POS terminals (PINPAD). Requires POS terminals assigned by ePagos.

> Not currently used by Aramal (we use eCheckout). Documented for future reference.

### obtener_terminales_pos (list POS terminals)

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }

Response:
  id_resp, respuesta
  terminales: array of:
    - numero: text (serial number — use in solicitud_pago_pinpad)
    - descripcion: text
    - marca: text
    - modelo: text

Error codes: 21001=OK | 21002=invalid version | 21003=invalid param | 21004=invalid token
```

### solicitud_pago_pinpad (initiate POS payment)

Sends a payment instruction to a specific POS terminal. The terminal prompts the customer to tap/insert/swipe their card.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  id_transaccion: int (from solicitud_pago — must be pre-generated)
  monto: decimal (must match the operation's amount exactly)
  terminal: text (serial number from obtener_terminales_pos)

Response:
  id_resp, respuesta

Error codes: 21001=OK | 21002=invalid version | 21003=invalid param | 21004=invalid token
```

#### POS Flow:
```
1. obtener_token
2. solicitud_pago → generates operation, gets id_transaccion
3. obtener_terminales_pos → list available terminals (optional)
4. solicitud_pago_pinpad (id_transaccion + monto + terminal serial)
5. Customer taps/inserts/swipes card on POS
6. ePagos processes and notifies via webhook
7. Reconcile with obtener_pagos
```

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

## Operation Type Codes

Standard codes for grouping operations by debt type. Request new codes via email to ePagos if needed.

| ID | Sigla | Nombre |
|----|-------|--------|
| 1 | TSG | Tasa de servicios generales |
| 2 | TISH | Tasa de Inspeccion de Seguridad e Higiene |
| 3 | TCRVM | Tasa de Conservacion de la Red Vial Municipal |
| 4 | ROD | Patente |
| 5 | CEM | Tasa de Cementerio |
| 6 | COM | Tasa de Comercio |
| 7 | COM_PUB | Tasa de Comercio - Publicidad y Propaganda |
| 8 | COM_SEG | Tasa de Comercio - Tasa de Seguridad |
| 9 | COM_HAB | Tasa de Comercio - Tasa de Habilitacion |
| 10 | COM_ANT | Tasa de Comercio - Inspeccion de Seguridad de Antenas |
| 11 | TASAS | Tasas Varias |
| 12 | INF | Infracciones |
| 13 | INF_TRA | Infracciones - Transito |
| 14 | INF_COM | Infracciones - Comercio |
| 15 | CONTRA | Contravenciones Varias |
| 16 | DER_OFI | Derechos de oficina |
| 17 | CONTRIB | Contribuciones Varias |
| 18 | CONTRAC | Contracargo descontado |
| 19 | SANIT | Sanitario |
| 20 | SERV | Servicio |

**Note:** These are primarily municipal tax codes. Aramal may use `20` (Servicio) or request a custom code for restaurant orders.

---

## Reference Tables

### Payment Method IDs (complete active list)

#### Online - Cards
| ID | Provider | Type |
|----|----------|------|
| 1 | VISA | Credit |
| 2 | AMEX | Credit |
| 3 | Mastercard | Credit |
| 9 | ArgenCard | Credit |
| 10 | Cabal Credito | Credit |
| 11 | Diners Club | Credit |
| 54 | Naranja X | Credit |
| 14 | VISA Debito | Debit |
| 15 | Maestro Debito | Debit |
| 28 | Cabal Debito | Debit |
| 41 | Mastercard Debito | Debit |
| 52 | Habitualista Debito | Debit |

#### Digital Wallets
| ID | Provider | Type |
|----|----------|------|
| 27 | Billetera ePagos (*) | Wallet |
| 45 | Billetera Cripto (*) | Crypto |
| 47 | Billetera MercadoPago (*) | Wallet |
| 48 | Billetera MODO (*) | Wallet |

#### Transfers
| ID | Provider | Type |
|----|----------|------|
| 17 | E-Transferencia (*) | Bank transfer |
| 38 | DEBIN (*) | Transfer |
| 42 | Debito directo (*) | Direct debit |
| 44 | Transferencias 3.0 (*) | Bank transfer |
| 51 | Debito inmediato (*) (**) | Transfer |

#### Cash / In-person
| ID | Provider | Type |
|----|----------|------|
| 4 | Pago Facil | Cash |
| 5 | Rapipago | Cash |
| 8 | Banco Nacion | Cash |
| 12 | BanCor - Banco de Cordoba | Cash |
| 24 | Cobro Express | Cash |
| 25 | E-Cajero (*) | Cash |
| 35 | Tarjeta mandataria (*) | Cash |
| 36 | Multipago | Cash |
| 39 | Banco de Corrientes (*) | Cash |
| 40 | Banco de Neuquen | Cash |
| 50 | Banco Santander (*) | Cash |
| 53 | Banco Supervielle | Cash |

#### Homebanking / Publication
| ID | Provider | Type |
|----|----------|------|
| 6 | Pago Mis Cuentas | Homebanking |
| 7 | Red Link - Web (*) | Homebanking |
| 26 | Red Link | Homebanking |

#### Other
| ID | Provider | Type |
|----|----------|------|
| 43 | IVR (*) | Phone payment |

(*) Not available for all organizations
(**) API SOAP only — ideal for bulk emission or recurring charges

### Payment Type IDs

Use in `tp_excluidos` to exclude entire payment categories.

| ID | Type |
|----|------|
| 1 | Presencial (cash) |
| 2 | Credit card |
| 3 | Debit card |
| 4 | Prepago |
| 5 | Homebanking (publication) |
| 6 | Billetera ePagos |
| 7 | Bank transfer |
| 8 | Phone payment (cobro telefonico) |
| 9 | Crypto |

### Document Type IDs

Use in `tipo_doc_pagador`.

| ID | Type |
|----|------|
| 1 | DNI |
| 2 | LE |
| 3 | LC |
| 4 | DNI Extranjero |
| 5 | Cedula Extranjero |
| 6 | Pasaporte |
| 7 | No consta |
| 8 | Cedula de identidad |
| 9 | CUIT |

### Province Codes

Use in `provincia_dom_pagador`.

| ID | Province |
|----|----------|
| 1 | Buenos Aires |
| 2 | Ciudad Autonoma de Buenos Aires |
| 3 | Catamarca |
| 4 | Cordoba |
| 5 | Corrientes |
| 6 | Chaco |
| 7 | Chubut |
| 8 | Entre Rios |
| 9 | Formosa |
| 10 | Jujuy |
| 11 | La Pampa |
| 12 | La Rioja |
| 13 | Mendoza |
| 14 | Misiones |
| 15 | Neuquen |
| 16 | Rio Negro |
| 17 | Salta |
| 18 | San Juan |
| 19 | San Luis |
| 20 | Santa Cruz |
| 21 | Santa Fe |
| 22 | Santiago del Estero |
| 23 | Tierra del Fuego |
| 24 | Tucuman |

---

## Test Cards & Data

### Credit Cards
| Provider | Number | CVV | Expiry |
|----------|--------|-----|--------|
| VISA | 4507 9900 0000 4905 | 123 | 12/2026 |
| AMEX | 3766 341249 71005 | 1234 | 12/2026 |
| Mastercard | 5323 6222 7777 7785 | 123 | 12/2026 |

### Debit Cards
| Provider | Number | CVV | Expiry |
|----------|--------|-----|--------|
| VISA Debito | 4517 7210 0485 6075 | 123 | 12/2026 |
| MC Debito | 5299 9100 1000 0015 | 123 | 12/2026 |

**Note:** Expiry date is not validated in sandbox. Any future date works.

### Test rejection
Set **Nombre y Apellido Titular = "CALL"** in checkout to trigger a rejection.

### Test CBU/CUIT — DEBIN
| CBU | Alias | CUIT | Type |
|-----|-------|------|------|
| 3220002204000040970011 | prueba.debin | 20123456781 | Fisica |
| 3220001101000040970011 | prueba.debin.2 | 27123456781 | Fisica |
| 3220001101000040970011 | prueba.debin.2 | 30111111110 | Juridica |
| 0070314530004001400167 | prueba.debin.3 | 20004940548 | Fisica |

### Test CBU/CUIT — Debito Inmediato
| CBU | Scenario | CUIT | Type |
|-----|----------|------|------|
| 3220001801000020816200 | Accepted | 20312528046 | Fisica |
| 3220002204000040970011 | Pending | 20312528046 | Fisica |
| Any DEBIN CBU above | Rejected | — | Fisica |

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

| Field | Max Length | Recommended use in Aramal |
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

### Summary by method prefix

| Prefix | Method |
|--------|--------|
| 01xxx | obtener_token |
| 02xxx | eCheckout responses |
| 03xxx | obtener_entidades_pago |
| 04xxx | obtener_pagos |
| 05xxx | obtener_rendiciones |
| 06xxx | obtener_contracargos |
| 07xxx | obtener_pagos_adicionales |
| 08xxx | solicitud_pago_lote |
| 10xxx | obtener_tarjetas_cliente / obtener_cuentas_cliente |
| 11xxx | solicitud_pago_recurrente_suscripcion |
| 14xxx | generar_qr_vinculado |
| 15xxx | pago_lote |
| 16xxx | registrar_cuentas_cliente |
| 17xxx | obtener_resultados_debito |
| 18xxx | obtener_cajas_qr / generar_orden_qr |
| 21xxx | obtener_terminales_pos / solicitud_pago_pinpad |

### Common pattern (most methods follow this)

| Suffix | Meaning |
|--------|---------|
| xxx1 | Success |
| xxx2 | Invalid token or credentials |
| xxx3 | Internal error |
| xxx4 | Invalid parameter |
| xxx5 | Invalid version / additional validation |

Specific error codes are documented inline with each method above.

---

## Architecture Decision for Aramal

### Chosen approach: eCheckout (POST redirect)

**Why:**
- No PCI certification needed
- ePagos handles all card UI and security
- Works with all payment methods (cards, wallets, transfers, cash)
- User pays on a trusted ePagos page
- Simple redirect flow compatible with our web app

**Flow in Aramal:**
1. Customer completes order -> clicks "Pay with ePagos"
2. Backend creates a `payment` record in DB with order details
3. Backend calls `POST sandbox.epagos.net/post.php` to get token
4. Backend returns a redirect URL/form data to frontend
5. Frontend redirects user to ePagos checkout
6. User pays -> ePagos redirects to our `ok_url` or `error_url`
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

---

## Test Scenarios Checklist

### eCheckout / Payment Button Tests

| ID | Test | Expected Result |
|----|------|-----------------|
| 11001 | Token generation | Valid token returned |
| 11002 | Card payment success | State=acreditada, redirect ok_url |
| 11003 | Card payment rejection (Titular="CALL") | State=rechazada, redirect error_url |
| 11004 | Card payment cancellation | State=cancelada, redirect error_url |
| 11005 | Cash/homebanking generation | State=adeudada, payment slip downloadable |
| 11006 | Cash/homebanking cancellation | State=cancelada, redirect error_url |
| 11007 | Operation with due date | State=acreditada with opc_fecha_vencimiento |
| 11008 | E-Transfer generation (CUIT=30715084291) | State=acreditada |
| 11009 | DEBIN generation | State=adeudada, use test CBU/CUIT |

### Emision Masiva (API) Tests

| ID | Test | Expected Result |
|----|------|-----------------|
| 12001 | SOAP token generation | Valid token |
| 12002 | solicitud_pago with fp=34 (combined) | Operation ID, state=adeudada |
| 12003 | solicitud_pago with QR/barcode | codigo_barras_fp + qr returned |
| 12004 | solicitud_pago_lote (3 operations) | 3 operation IDs returned |
| 12005 | obtener_pagos by ID or external ID | Operation found, correct state |
| 12006 | generar_qr_vinculado (link ops from 12004) | QR base64 image returned |
| 12007 | solicitud_pago with due date | Operation with vencimiento set |

### Conciliation Tests (shared)

| ID | Test | Expected Result |
|----|------|-----------------|
| 21001 | obtener_pagos by date range | Paginated results |
| 21002 | obtener_pagos_adicionales by date | Additional payments found |
| 21003 | obtener_rendiciones by payment date | Settlement data |
| 21004 | obtener_rendiciones by deposit date | Settlement data |
| 21005 | obtener_contracargos by date | Chargeback data |
| 22001 | Webhook — new payment | POST received, tipo=P |
| 22002 | Webhook — existing payment | Idempotent handling |
| 22003 | Webhook — refund | POST received, tipo=D |

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
