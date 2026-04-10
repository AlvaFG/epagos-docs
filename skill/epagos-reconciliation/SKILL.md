---
name: epagos-reconciliation
description: ePagos daily reconciliation via SOAP v2.0 - obtener_pagos, rendiciones, contracargos. Use when working on payment reconciliation, settlement sync, chargeback monitoring, or daily sync jobs.
---

# ePagos Reconciliation (SOAP v2.0)

## Overview

Daily reconciliation is **mandatory** -- ePagos recommends it as the definitive source of truth for payment status. Webhooks can fail; reconciliation catches everything.

**Recommended schedule:**
- Main sync: every 24h (off-peak hours)
- Chargeback alerts: every 6h

**Reconciliation methods use SOAP v2.0** on the `.net` domain. The official website marks v2.1 as [actual] and these methods are available on `.com` WSDLs too, but the v2.0/`.net` endpoint is stable and includes Estado "L" (Lote Acreditado) in responses.

> For API version matrix see `epagos-reference`. For eCheckout flow see `epagos-echeckout`. For SOAP payment creation see `epagos-soap-payments`.

---

## URLs (Reconciliation only)

| Purpose | Sandbox | Production |
|---------|---------|------------|
| SOAP WSDL v2.0 | `https://sandbox.epagos.net/wsdl/2.0/index.php?wsdl` | `https://api.epagos.net/wsdl/2.0/index.php?wsdl` |

Note: The official website only shows `.com` WSDLs (v2.1+). The `.net` v2.0 endpoint is a legacy alternative that still works. SOAP v2.1 (`.com`) is used for payment creation -- see `epagos-soap-payments`.

---

## Token Management (SOAP v2.0)

```
Request:
  version: "2.0"
  credenciales:
    id_organismo: <provided by ePagos>
    id_usuario: <provided by ePagos>
    password: "<provided by ePagos>"
    hash: "<provided by ePagos>"

Response:
  id_resp: 01001 (success) | 01002 (invalid credentials) | 01003 (internal) | 01004 (invalid version)
  respuesta: text
  token: string
```

**Caching:** SOAP tokens should be cached (e.g., in Redis) with ~50-minute TTL.

**Token expiry auto-retry:** If a method returns `04002`, `05002`, or `06002` (invalid token), clear the cached token, get a fresh one, and retry the call once.

**Version parameter:** Always `"2.0"` -- this never changes.

---

## obtener_pagos (query payments)

The primary reconciliation method. Use `FechaNovedadAcreditacion` filters for daily sync.

```
Request:
  version: "2.0"
  credenciales: { id_organismo, token }
  pago (all optional filters):
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
    FechaNovedadAcreditacionDesde: "YYYY-MM-DD HH:MM:SS" (RECOMMENDED)
    FechaNovedadAcreditacionHasta: "YYYY-MM-DD HH:MM:SS"
    DevolverID4: boolean (default false)
    Pagina: integer (starts at 1, 100 results per page)

Response:
  id_resp, respuesta, token, id_organismo, pagina, cantidadTotal
  pagos: array of:
    CodigoUnicoTransaccion: int
    Externa, Externa_2, Externa_3: string (your identifiers)
    ExternoId_4: string (only if DevolverID4=true)
    Convenio: int
    Importe: decimal
    Estado: char (A/O/V/C/R/P/D/L)
    FechaPago: datetime
    FechaAcreditacion: datetime
    FechaNovedadAcreditacion: datetime
    FormaPago: array of { Identificador, Tipo, Importe, CodigoPago, CodigoBarras }
    DatosPagador: { Nombre, Apellido, Email, Identificacion: {tipo_doc, numero_doc, cuit} }
    PagosAdicionales: array of { CodigoUnicoTransaccion, FormaPago, Monto, FechaPago, FechaNovedad, IdPago }
    Recibo: URL
    Url_QR: URL

Error codes:
  04001 = OK
  04002 = Invalid token (auto-retry)
  04003 = Internal error (retry after 60s)
  04004 = Date range exceeded (max 48h per query)
  04005 = Invalid parameter
  04006 = Invalid version
```

**Pagination:** Max 100 results per page. Use `Pagina` starting at 1. Check `cantidadTotal` vs processed count to know if more pages exist.

---

## Payment State Mapping

| ePagos Code | State | Description | Suggested Usage |
|-------------|-------|-------------|-----------------|
| A | Acreditada | Paid successfully | Mark as paid/credited |
| O | Adeudada | Pending (cash/transfer) | Mark as awaiting payment |
| V | Vencida | Expired past due date | Mark as failed/expired |
| C | Cancelada | Cancelled by user | Mark as cancelled |
| R | Rechazada | Rejected by payment method | Mark as failed/rejected |
| P | Pendiente | Initiated, no method selected | Mark as pending |
| D | Devuelta | Refunded to user | Mark as refunded |
| L | Lote Acreditado | Batch credit (liquidated) | Mark as paid/credited |

**Note on Estado "L":** The v2.1 docs on epagos.com do NOT list Estado L, but v2.4+ docs do. The v2.0 endpoint returns L in actual API responses. Handle it as credited.

---

## obtener_pagos_adicionales

Additional payments on already-credited operations (e.g., user pays twice on a cash slip).

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
    FechaNovedad: datetime
    IdPago: int

Error codes: 07001=OK | 07002=invalid token | 07003=internal | 07004=invalid dates | 07005=invalid param | 07006=invalid version
```

**Note:** `fecha_desde/fecha_hasta` filter by the date ePagos received the payment notification (`FechaNovedad`), NOT the date the user paid (`FechaPago`).

---

## obtener_rendiciones (daily settlements)

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

---

## obtener_contracargos (chargebacks)

```
Request:
  credenciales: { id_organismo, token }
  contracargos:
    numero: integer
    estado: "P"|"R"|"C"|"F"|"FNF"|"B"
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

### Chargeback States

| Code | State | Description |
|------|-------|-------------|
| P | Pendiente | Chargeback opened, awaiting response |
| R | Respondido | Response submitted |
| C | Confirmado | Chargeback confirmed |
| F | Finalizado | Process completed |
| FNF | No Favorable | Resolved against merchant |
| B | Baja | Withdrawn/cancelled |

---

## Recommended Lookback Windows

| Data Type | Lookback |
|-----------|----------|
| Payments | 5 days |
| Settlements | 7 days |
| Chargebacks | 30 days |

---

## Reconciliation Process

```
1. obtener_token -> get fresh SOAP v2.0 token (or use cached, ~50min TTL)
2. obtener_pagos -> filter FechaNovedadAcreditacion last 5 days, paginate if >100
3. Process each payment -> update internal DB (use batch queries, no N+1)
4. obtener_pagos_adicionales -> catch payments on already-credited ops
5. obtener_contracargos -> monitor disputes (30-day window)
6. obtener_rendiciones -> sync settlement files (7-day window)
7. Generate alerts for chargebacks pending >7 days
```

**Key rules:**
- Max date range per query: 48 hours -- split longer ranges into 48h chunks
- 100 results per page, use `Pagina` to iterate
- Always retry on xxx3 (internal error) after 60 seconds
- Amount validation: compare ePagos `Importe` with your payment amount
- Idempotent: look up by ePagos transaction ID before updating
- Batch queries: use `WHERE id = ANY($1)` pattern for efficiency

---

## Chargeback Alerting

Recommended chargeback monitoring approach:
- Query `obtener_contracargos` with 30-day lookback every 6 hours
- If a chargeback has been in `Pendiente` (P) state for >7 days: send alert notification to merchant
- Track alert state to prevent duplicate notifications

---

## Refunds

Refunds are queried using `obtener_pagos` with `Estado = 'D'` filter. There is no separate refund API -- refunds are initiated through the ePagos portal.
