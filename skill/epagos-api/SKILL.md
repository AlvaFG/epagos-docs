---
name: epagos-api
description: Centralized API reference - all ePagos methods, endpoints, types, and versions in one place. Use when you need to find which API method to use, check version availability, or get a quick overview of the full ePagos API surface.
---

# ePagos API — Referencia Centralizada

Índice completo de todos los métodos y endpoints de la plataforma ePagos. Usa esta skill para orientarte rápidamente sobre qué método necesitás y en qué versión está disponible. Para detalle de implementación, consultá las skills específicas.

## URLs base

| Entorno | E-Checkout / SOAP v2.1+ | SOAP v2.0 (conciliación) |
|---------|--------------------------|--------------------------|
| Sandbox | `https://sandbox.epagos.com` | `https://sandbox.epagos.net` |
| Producción | `https://api.epagos.com` | `https://api.epagos.net` |

| Entorno | Token POST | Redirect E-Checkout |
|---------|------------|---------------------|
| Sandbox | `https://sandbox.epagos.com/post.php` | `https://postsandbox.epagos.com` |
| Producción | `https://api.epagos.com/post.php` | `https://post.epagos.com` |

## Tabla completa de APIs

### E-Checkout y Botón de Pago

| # | Método / Endpoint | Tipo | Descripción | Versiones | Skill |
|---|-------------------|------|-------------|-----------|-------|
| 1 | `POST /post.php` | POST | Token single-use para iniciar E-Checkout | Todas | `epagos-echeckout` |
| 2 | Redirect a checkout | POST (redirect) | Formulario oculto que redirige al usuario al checkout | Todas | `epagos-echeckout` |
| 3 | `ok_url` / `error_url` | POST (inbound) | Callback al navegador con resultado del pago | Todas | `epagos-echeckout` |
| 4 | Webhook | POST (inbound) | Notificación server-to-server (sin firma HMAC) | Todas | `epagos-echeckout` |
| 5 | Botón de Pago JS | JS (client-side) | Widget JS con clave pública, abre checkout en iframe | Todas | `epagos-echeckout` |

### SOAP — Autenticación

| # | Método | Tipo | Descripción | Versiones | Skill |
|---|--------|------|-------------|-----------|-------|
| 6 | `obtener_token` | SOAP | Valida credenciales, devuelve token (~50 min TTL) | v1.0–v2.5 | `epagos-soap-payments` / `epagos-reconciliation` |

### SOAP — Creación de Pagos

| # | Método | Tipo | Descripción | Versiones | Skill |
|---|--------|------|-------------|-----------|-------|
| 7 | `solicitud_pago` | SOAP | Genera operación de pago (boleta, barras, QR, tarjeta) | v1.0–v2.5 | `epagos-soap-payments` |
| 8 | `solicitud_pago_lote` | SOAP | Emisión masiva: hasta 50 operaciones en una llamada | v1.0–v2.5 | `epagos-soap-payments` |
| 9 | `pago_lote` | SOAP | Ejecuta/acredita lote generado con `solicitud_pago_lote` | v2.1+ | `epagos-soap-payments` |

### SOAP — Consultas

| # | Método | Tipo | Descripción | Versiones | Skill |
|---|--------|------|-------------|-----------|-------|
| 10 | `obtener_pagos` | SOAP | Lista operaciones con estado y detalle (paginado, max 100) | v1.0–v2.5 | `epagos-reconciliation` |
| 11 | `obtener_entidades_pago` | SOAP | Medios de pago habilitados (IDs, logos, config) | v1.0–v2.5 | `epagos-soap-payments` |
| 12 | `obtener_pagos_adicionales` | SOAP | Pagos adicionales sobre operaciones ya acreditadas | v1.0–v2.5 | `epagos-reconciliation` |

### SOAP — Conciliación

| # | Método | Tipo | Descripción | Versiones | Skill |
|---|--------|------|-------------|-----------|-------|
| 13 | `obtener_rendiciones` | SOAP | Rendiciones diarias: comisiones, retenciones, depósitos | v1.0–v2.5 | `epagos-reconciliation` |
| 14 | `obtener_contracargos` | SOAP | Contracargos con estados (P/R/C/F/FNF/B) | v1.0–v2.5 | `epagos-reconciliation` |

### SOAP — Recurrentes y Suscripciones

| # | Método | Tipo | Descripción | Versiones | Skill |
|---|--------|------|-------------|-----------|-------|
| 15 | `solicitud_pago_recurrente` | SOAP | Pago recurrente con tarjeta tokenizada | v1.0–v2.5 | `epagos-soap-payments` |
| 16 | `solicitud_pago_recurrente_suscripcion` | SOAP | Suscripción de cobro recurrente automático | v1.0–v2.5 | `epagos-soap-payments` |

### SOAP — Tarjetas y Cuentas

| # | Método | Tipo | Descripción | Versiones | Skill |
|---|--------|------|-------------|-----------|-------|
| 17 | `obtener_tarjetas_cliente` | SOAP | Tarjetas tokenizadas del cliente | v1.0–v2.5 | `epagos-soap-payments` |
| 18 | `obtener_cuentas_cliente` | SOAP | Cuentas bancarias registradas (DEBIN, débito) | v1.0–v2.5 | `epagos-soap-payments` |
| 19 | `registrar_cuentas_cliente` | SOAP | Registrar hasta 100 adhesiones para débito automático | v2.1+ | `epagos-soap-payments` |
| 20 | `obtener_resultados_debito` | SOAP | Resultados de débitos procesados por el banco | v2.1+ | `epagos-soap-payments` |

### SOAP — QR

| # | Método | Tipo | Descripción | Versiones | Skill |
|---|--------|------|-------------|-----------|-------|
| 21 | `generar_qr_vinculado` | SOAP | QR vinculado a una o más operaciones existentes | v1.0–v2.5 | `epagos-soap-payments` |
| 22 | `obtener_cajas_qr` | SOAP | Lista cajas QR interoperables estáticas (tipo F/V) | v2.1+ | `epagos-soap-payments` |
| 23 | `generar_orden_qr` | SOAP | Orden de pago dinámica para caja QR estática | v2.1+ | `epagos-soap-payments` |

### SOAP — Terminales POS

| # | Método | Tipo | Descripción | Versiones | Skill |
|---|--------|------|-------------|-----------|-------|
| 24 | `obtener_terminales_pos` | SOAP | Lista terminales POS/PinPad configuradas | v2.5 | `epagos-soap-payments` |
| 25 | `solicitud_pago_pinpad` | SOAP | Pago presencial en terminal POS/PinPad | v2.5 | `epagos-soap-payments` |

## Versiones disponibles

| Versión | WSDL path | Métodos | Estado | Param `version` |
|---------|-----------|:-------:|--------|:---------------:|
| v1.0 | `/wsdl/index.php?wsdl` | 13 | Legacy | `"1.0"` |
| v2.0 | `/wsdl/2.0/index.php?wsdl` | 13 | Legacy | `"2.0"` |
| v2.1 | `/wsdl/2.1/index.php?wsdl` | 18 | **Actual** | `"2.0"` |
| v2.4 | `/wsdl/2.4/index.php?wsdl` | 18 | Disponible | `"2.0"` |
| v2.5 | `/wsdl/2.5/index.php?wsdl` | 20 | Disponible | `"2.0"` |

> La versión real se selecciona por el **endpoint WSDL**, no por el parámetro `version`.

## ¿Qué método necesito?

| Quiero... | Método | Skill |
|-----------|--------|-------|
| Cobrar online (redirect) | E-Checkout (`POST /post.php` + redirect) | `epagos-echeckout` |
| Cobrar online (widget JS) | Botón de Pago JS | `epagos-echeckout` |
| Generar boleta/código de barras | `solicitud_pago` | `epagos-soap-payments` |
| Emitir muchas boletas a la vez | `solicitud_pago_lote` + `pago_lote` | `epagos-soap-payments` |
| Cobrar con QR dinámico | `generar_orden_qr` | `epagos-soap-payments` |
| Cobrar con QR vinculado | `generar_qr_vinculado` | `epagos-soap-payments` |
| Cobrar recurrente con tarjeta | `solicitud_pago_recurrente` | `epagos-soap-payments` |
| Crear suscripción automática | `solicitud_pago_recurrente_suscripcion` | `epagos-soap-payments` |
| Débito automático en cuenta | `registrar_cuentas_cliente` → `obtener_resultados_debito` | `epagos-soap-payments` |
| Cobrar en terminal POS | `solicitud_pago_pinpad` | `epagos-soap-payments` |
| Consultar estado de pagos | `obtener_pagos` | `epagos-reconciliation` |
| Conciliar rendiciones | `obtener_rendiciones` | `epagos-reconciliation` |
| Monitorear contracargos | `obtener_contracargos` | `epagos-reconciliation` |
| Ver medios de pago habilitados | `obtener_entidades_pago` | `epagos-soap-payments` |
| Buscar IDs, códigos, tarjetas test | Tablas de referencia | `epagos-reference` |

## Skills disponibles

| Skill | Cubre |
|-------|-------|
| `epagos-echeckout` | Flujo E-Checkout, callbacks, webhook, botón JS |
| `epagos-soap-payments` | Todos los métodos SOAP de creación de pagos, QR, lotes, recurrentes, POS |
| `epagos-reconciliation` | Conciliación diaria: obtener_pagos, rendiciones, contracargos |
| `epagos-reference` | Tablas de referencia: IDs de medios de pago, códigos de error, test cards, URLs |
| `epagos-api` | Esta skill: índice centralizado de todos los endpoints |
