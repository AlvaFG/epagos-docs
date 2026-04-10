# ePagos — Tabla Centralizada de APIs

> Referencia rápida de todos los métodos y endpoints disponibles en la plataforma ePagos.
> Última actualización: 2026-04-10

---

## URLs base

| Entorno | Dominio E-Checkout / SOAP v2.1+ | Dominio SOAP v2.0 (conciliación) |
|---------|----------------------------------|----------------------------------|
| Sandbox | `https://sandbox.epagos.com` | `https://sandbox.epagos.net` |
| Producción | `https://api.epagos.com` | `https://api.epagos.net` |

---

## Tabla maestra de APIs

### E-Checkout y Botón de Pago

| # | Método / Endpoint | Tipo | Descripción | Versiones |
|---|-------------------|------|-------------|-----------|
| 1 | `POST /post.php` | POST | Obtener token single-use para iniciar E-Checkout | Todas |
| 2 | `POST → postsandbox.epagos.com` / `post.epagos.com` | POST (redirect) | Redirección del usuario al checkout de ePagos con formulario oculto | Todas |
| 3 | `ok_url` / `error_url` (callback) | POST (inbound) | Callback que ePagos envía al navegador del usuario con resultado del pago | Todas |
| 4 | Webhook ePagos | POST (inbound) | Notificación server-to-server con resultado del pago (sin firma HMAC) | Todas |
| 5 | Botón de Pago JS | JS (client-side) | Widget JavaScript que abre checkout en iframe/popup usando clave pública | Todas |

### SOAP — Autenticación

| # | Método | Tipo | Descripción | Versiones |
|---|--------|------|-------------|-----------|
| 6 | `obtener_token` | SOAP | Valida credenciales y devuelve token de autorización (~50 min TTL) | v1.0, v2.0, v2.1, v2.4, v2.5 |

### SOAP — Creación de Pagos

| # | Método | Tipo | Descripción | Versiones |
|---|--------|------|-------------|-----------|
| 7 | `solicitud_pago` | SOAP | Genera nueva operación de pago (boleta, código de barras, QR, tarjeta) | v1.0, v2.0, v2.1, v2.4, v2.5 |
| 8 | `solicitud_pago_lote` | SOAP | Genera hasta 50 operaciones de pago en una sola llamada (emisión masiva) | v1.0, v2.0, v2.1, v2.4, v2.5 |
| 9 | `pago_lote` | SOAP | Ejecuta/acredita un lote previamente generado con `solicitud_pago_lote` | v2.1, v2.4, v2.5 |

### SOAP — Consultas de Pagos

| # | Método | Tipo | Descripción | Versiones |
|---|--------|------|-------------|-----------|
| 10 | `obtener_pagos` | SOAP | Lista operaciones con estado, datos del pagador y formas de pago (paginado, max 100) | v1.0, v2.0, v2.1, v2.4, v2.5 |
| 11 | `obtener_entidades_pago` | SOAP | Devuelve medios de pago habilitados para el organismo (IDs, logos, config) | v1.0, v2.0, v2.1, v2.4, v2.5 |
| 12 | `obtener_pagos_adicionales` | SOAP | Lista pagos adicionales sobre operaciones ya acreditadas | v1.0, v2.0, v2.1, v2.4, v2.5 |

### SOAP — Conciliación

| # | Método | Tipo | Descripción | Versiones |
|---|--------|------|-------------|-----------|
| 13 | `obtener_rendiciones` | SOAP | Obtiene rendiciones diarias con comisiones, retenciones y depósitos | v1.0, v2.0, v2.1, v2.4, v2.5 |
| 14 | `obtener_contracargos` | SOAP | Lista contracargos recibidos con estados (P/R/C/F/FNF/B) | v1.0, v2.0, v2.1, v2.4, v2.5 |

### SOAP — Recurrentes y Suscripciones

| # | Método | Tipo | Descripción | Versiones |
|---|--------|------|-------------|-----------|
| 15 | `solicitud_pago_recurrente` | SOAP | Genera pago recurrente con tarjeta tokenizada | v1.0, v2.0, v2.1, v2.4, v2.5 |
| 16 | `solicitud_pago_recurrente_suscripcion` | SOAP | Crea suscripción de cobro recurrente automático | v1.0, v2.0, v2.1, v2.4, v2.5 |

### SOAP — Tarjetas y Cuentas

| # | Método | Tipo | Descripción | Versiones |
|---|--------|------|-------------|-----------|
| 17 | `obtener_tarjetas_cliente` | SOAP | Obtiene tarjetas tokenizadas del cliente | v1.0, v2.0, v2.1, v2.4, v2.5 |
| 18 | `obtener_cuentas_cliente` | SOAP | Obtiene cuentas bancarias registradas del cliente (DEBIN, débito) | v1.0, v2.0, v2.1, v2.4, v2.5 |
| 19 | `registrar_cuentas_cliente` | SOAP | Registra hasta 100 adhesiones de cuentas bancarias para débito automático | v2.1, v2.4, v2.5 |
| 20 | `obtener_resultados_debito` | SOAP | Consulta resultados de débitos automáticos procesados por el banco | v2.1, v2.4, v2.5 |

### SOAP — QR

| # | Método | Tipo | Descripción | Versiones |
|---|--------|------|-------------|-----------|
| 21 | `generar_qr_vinculado` | SOAP | Genera QR vinculado a una o más operaciones existentes | v1.0, v2.0, v2.1, v2.4, v2.5 |
| 22 | `obtener_cajas_qr` | SOAP | Lista cajas QR interoperables estáticas del organismo (tipo F/V) | v2.1, v2.4, v2.5 |
| 23 | `generar_orden_qr` | SOAP | Genera orden de pago dinámica asociada a una caja QR estática | v2.1, v2.4, v2.5 |

### SOAP — Terminales POS (v2.5)

| # | Método | Tipo | Descripción | Versiones |
|---|--------|------|-------------|-----------|
| 24 | `obtener_terminales_pos` | SOAP | Lista terminales POS/PinPad configuradas | v2.5 |
| 25 | `solicitud_pago_pinpad` | SOAP | Procesa pago presencial en terminal POS/PinPad | v2.5 |

---

## Resumen por versión

| Versión | WSDL path | Métodos SOAP | Estado |
|---------|-----------|:------------:|--------|
| v1.0 | `/wsdl/index.php?wsdl` | 13 | Legacy |
| v2.0 | `/wsdl/2.0/index.php?wsdl` | 13 | Legacy |
| v2.1 | `/wsdl/2.1/index.php?wsdl` | 18 | **Actual** |
| v2.4 | `/wsdl/2.4/index.php?wsdl` | 18 | Disponible |
| v2.5 | `/wsdl/2.5/index.php?wsdl` | 20 | Disponible (POS + campos E-Checkout) |

> **Nota:** El parámetro `version` en los métodos SOAP es siempre `"1.0"` para v1.0 y `"2.0"` para v2.0/v2.1/v2.4/v2.5. La versión real se determina por el endpoint WSDL.

---

## Referencias

- Detalle de cada método SOAP: `docs/06-*.md` a `docs/16-*.md`
- Campos nuevos v2.5: `docs/18-echeckout-campos-nuevos-v2.5.md`, `docs/19-changelog-v2.5.md`
- Tablas de referencia (IDs medios de pago, provincias, etc.): `docs/17-tablas-de-referencia.md`
- Mapa de versiones: `docs/00-versiones-api.md`
