---
name: epagos-reference
description: ePagos reference tables - payment method IDs, error codes, payment states, URLs, credentials, test data. Use when looking up an ePagos code, ID, URL, or test credential.
---

# ePagos Reference Tables

## Overview

ePagos is an Argentine PSP (Payment Service Provider) regulated by BCRA. Consolidates 30+ payment methods into a single integration.

- **Website:** epagos.com / epagos.com.ar
- **Support:** soporte@epagos.com.ar
- **Status:** https://status.epagos.com.ar/
- **No Node.js SDK exists** -- build your own client using the SOAP/HTTP APIs below
- **Official SDKs:** PHP ([github.com/epagos/api_php](https://github.com/epagos/api_php)), .NET ([github.com/epagos/api_net_core](https://github.com/epagos/api_net_core))

> For payment flows see `epagos-echeckout`, for reconciliation see `epagos-reconciliation`, for SOAP payment creation see `epagos-soap-payments`.

---

## Complete URL Table

| Purpose | Sandbox | Production | Domain |
|---------|---------|------------|--------|
| eCheckout token (POST) | `https://sandbox.epagos.com/post.php` | `https://api.epagos.com/post.php` | .com |
| eCheckout redirect | `https://postsandbox.epagos.com` | `https://post.epagos.com` | .com |
| SOAP WSDL v2.0 (reconciliation) | `https://sandbox.epagos.net/wsdl/2.0/index.php?wsdl` | `https://api.epagos.net/wsdl/2.0/index.php?wsdl` | .net |
| SOAP WSDL v2.1 (payments) | `https://sandbox.epagos.com/wsdl/2.1/index.php?wsdl` | `https://api.epagos.com/wsdl/2.1/index.php?wsdl` | .com |
| SOAP namespace (tns) | `https://sandbox.epagos.net/` | `https://api.epagos.net/` | .net |
| SOAPAction header | `https://sandbox.epagos.net/solicitud_pago` | `https://api.epagos.net/solicitud_pago` | .net |
| JS library | `https://sandbox.epagos.com/quickstart/epagos.min.js` | `https://api.epagos.com/quickstart/epagos.min.js` | .com |
| Portal | `https://portalsandbox.epagos.com` | `https://portal.epagos.com` | .com |
| Logos | `https://sandbox.epagos.com/logos.php` | `https://api.epagos.com/logos.php` | .com |
| Status page | `https://status.epagos.com.ar/` | | .com.ar |

## Domain Mapping (IMPORTANT)

The `.com` and `.net` domains are NOT interchangeable -- they point to different servers:

- **`.com`** = eCheckout endpoints + SOAP v2.1 WSDL/endpoint (payment creation)
- **`.net`** = SOAP v2.0 WSDL (reconciliation) + SOAP namespace (`tns:`) + SOAPAction headers
- **`.com.ar`** = Status page only

---

## API Versions

ePagos has multiple API versions. The official website (epagos.com/desarrolladores.php) lists:

| Version | WSDL Path | Methods | Estado "L" | Status |
|---------|-----------|---------|------------|--------|
| v1.0 | `/wsdl/index.php?wsdl` | 13 | Unknown | Legacy (SOAP 1.2) |
| v2.0 | Not shown on website | 15 | Yes | Base version |
| v2.1 | `/wsdl/2.1/index.php?wsdl` | 18 | No | Marked [actual] on website |
| v2.4 | `/wsdl/2.4/index.php?wsdl` | 18 | Yes | Same methods as v2.1 + Estado L |
| v2.5 | `/wsdl/2.5/index.php?wsdl` | 20 | Yes | Adds POS: obtener_terminales_pos, solicitud_pago_pinpad |

**All versions from v2.0+ are "subversions" of v2.0** — the `version` parameter in requests is ALWAYS `"2.0"`.

**v2.1 added over v2.0:** obtener_cajas_qr, generar_orden_qr, obtener_resultados_debito
**v2.5 added over v2.1:** obtener_terminales_pos, solicitud_pago_pinpad (POS terminals)

**Official documentation PDFs:** [github.com/epagos/documentacion](https://github.com/epagos/documentacion) (latest: v2.9.3)

---

## Version Parameter

**ALWAYS `"2.0"` for ALL operations.** This includes:
- eCheckout POST form (`version = "2.0"`)
- SOAP v2.0 methods (`version: "2.0"`)
- SOAP v2.1 methods (`version: "2.0"` -- NOT "2.1")

The "2.1" in the WSDL path refers to the endpoint version, not the request parameter. Confirmed on official website: "se trata de una subversion de la v2.0 por lo que el numero a informar en los metodos sigue siendo el 2.0".

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

**Production credentials MUST be stored securely (environment variables, secrets manager) -- never in code.**

---

## Payment States

| Code | State | Description | Suggested Usage |
|------|-------|-------------|-----------------|
| A | Acreditada | Credited/Paid successfully | Mark as paid/credited |
| O | Adeudada | Owed -- pending payment (cash/transfer) | Mark as awaiting payment |
| V | Vencida | Expired -- past due date | Mark as failed/expired |
| C | Cancelada | Cancelled by user | Mark as cancelled |
| R | Rechazada | Rejected by payment method | Mark as failed/rejected |
| P | Pendiente | Request initiated, no payment method selected | Mark as pending |
| D | Devuelta | Refunded to user | Mark as refunded |
| L | Lote Acreditado | Batch credit (liquidated) | Mark as paid/credited |

---

## Chargeback States

| Code | State | Description |
|------|-------|-------------|
| P | Pendiente | Chargeback opened, awaiting response |
| R | Respondido | Response submitted |
| C | Confirmado | Chargeback confirmed |
| F | Finalizado | Process completed |
| FNF | No Favorable | Resolved against merchant |
| B | Baja | Withdrawn/cancelled |

---

## Payment Method IDs

### Online - Cards
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

### Digital Wallets
| ID | Provider | Type |
|----|----------|------|
| 27 | Billetera ePagos (*) | Wallet |
| 45 | Billetera Cripto (*) | Crypto |
| 47 | Billetera MercadoPago (*) | Wallet |
| 48 | Billetera MODO (*) | Wallet |

### Transfers
| ID | Provider | Type | Notes |
|----|----------|------|-------|
| 17 | E-Transferencia (*) | Bank transfer | |
| 38 | DEBIN (*) | Transfer | |
| 42 | Debito directo (*) | Direct debit | |
| 44 | Transferencias 3.0 (*) | Bank transfer | |
| 51 | Debito inmediato (*) (**) | Transfer | API SOAP only |

### Cash / In-person
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

### Homebanking / Publication
| ID | Provider | Type |
|----|----------|------|
| 6 | Pago Mis Cuentas | Homebanking |
| 7 | Red Link - Web (*) | Homebanking |
| 26 | Red Link | Homebanking |

### Other
| ID | Provider | Type |
|----|----------|------|
| 43 | IVR (*) | Phone payment |

(*) Not available for all organizations
(**) API SOAP only

---

## Payment Type IDs

Use in `tp_excluidos` to exclude entire categories.

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

---

## Document Type IDs

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

---

## Province Codes

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

## Operation Type Codes

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

---

## Error Response Codes

### Prefix-to-method mapping

| Prefix | Method |
|--------|--------|
| 01xxx | obtener_token |
| 02xxx | eCheckout responses / solicitud_pago |
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

### Common suffix pattern

| Suffix | Meaning |
|--------|---------|
| xxx1 | Success |
| xxx2 | Invalid token or credentials |
| xxx3 | Internal error (retry after 60s) |
| xxx4 | Invalid parameter |
| xxx5 | Invalid version / additional validation |

Token-expiry codes triggering auto-retry: `04002`, `05002`, `06002`.

---

## Test Cards

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

Expiry date is NOT validated in sandbox. Any future date works.

### Force rejection
Set **Nombre y Apellido Titular = "CALL"** to trigger a rejection.

---

## Test CBU/CUIT

### DEBIN
| CBU | Alias | CUIT | Type |
|-----|-------|------|------|
| 3220002204000040970011 | prueba.debin | 20123456781 | Fisica |
| 3220001101000040970011 | prueba.debin.2 | 27123456781 | Fisica |
| 3220001101000040970011 | prueba.debin.2 | 30111111110 | Juridica |
| 0070314530004001400167 | prueba.debin.3 | 20004940548 | Fisica |

### Debito Inmediato
| CBU | Scenario | CUIT | Type |
|-----|----------|------|------|
| 3220001801000020816200 | Accepted | 20312528046 | Fisica |
| 3220002204000040970011 | Pending | 20312528046 | Fisica |
| Any DEBIN CBU above | Rejected | -- | Fisica |

---

## External Identifiers

| Field | Max Length | Suggested Usage |
|-------|-----------|-----------------|
| `numero_operacion` | 100 | Your payment/order UUID |
| `identificador_externo_2` | 100 | Secondary reference (e.g., order ID, session ID) |
| `identificador_externo_3` | 512 | Tertiary reference (e.g., merchant ID, store ID) |
| `identificador_externo_4` | 65535 | JSON metadata (order details, extra context) |

---

## Minimum Amounts

| Method | Minimum |
|--------|---------|
| Credit cards | $40 ARS |
| Debit cards | No minimum |
| Transferencia 3.0 | $900 ARS |
| Billetera ePagos | No minimum |

---

## Logo Requirements

- Format: JPG
- Width: 142px
- Height: max 42px

---

## Integration Options

| Option | Complexity | PCI Required | Skill |
|--------|------------|-------------|-------|
| Payment Button (JS) | Basic | No | `epagos-echeckout` |
| eCheckout (POST redirect) | Intermediate | No | `epagos-echeckout` |
| SOAP API (server-side) | Advanced | Yes (for cards) | `epagos-soap-payments` |

**eCheckout is recommended for most integrations** -- no PCI needed, ePagos handles all card UI.

---

## Token Types

Two distinct token mechanisms exist -- do NOT mix them:

| Token Type | Obtained From | Used For | Caching |
|------------|---------------|----------|---------|
| POST token | `POST /post.php` (.com) | eCheckout redirect only | Single-use, no caching |
| SOAP v2.0 token | `obtener_token` via WSDL v2.0 (.net) | Reconciliation methods | Cache with ~50min TTL |
| SOAP v2.1 token | `obtener_token` via WSDL v2.1 (.com) | Payment creation methods | Cache with ~50min TTL |

---

## PHP SDK Reference

The official PHP SDK at github.com/epagos/api_php shows:
- SOAP client with `SoapClient($wsdl, options)`
- POST token (for eCheckout) uses `curl` to `post.php`
- SOAP token (for API) uses `SoapClient->obtener_token()`
- These are DIFFERENT tokens for different purposes
- `solicitud_pago_post()` generates a self-submitting HTML form
- Refunds query uses `obtener_pagos` with `Estado = 'D'`
