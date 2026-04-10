# ePagos API — Mapa de Versiones

> Última actualización: 2026-04-10
> Fuente: https://www.epagos.com/templates/desarrolladores/referencia.php

---

## Resumen de versiones disponibles

La API SOAP de ePagos tiene **5 versiones** publicadas. La versión marcada como **[actual]** es la **v2.1**.

| Versión | Estado | WSDL Sandbox | WSDL Producción | Valor `version` en métodos |
|---------|--------|-------------|-----------------|---------------------------|
| v1.0 | Legacy | `https://sandbox.epagos.com/wsdl/index.php?wsdl` | `https://api.epagos.com/wsdl/index.php?wsdl` | `"1.0"` |
| v2.0 | Legacy | `https://sandbox.epagos.com/wsdl/2.0/index.php?wsdl` | `https://api.epagos.com/wsdl/2.0/index.php?wsdl` | `"2.0"` |
| v2.1 | **[actual]** | `https://sandbox.epagos.com/wsdl/2.1/index.php?wsdl` | `https://api.epagos.com/wsdl/2.1/index.php?wsdl` | `"2.0"` (*) |
| v2.4 | Disponible | `https://sandbox.epagos.com/wsdl/2.4/index.php?wsdl` | `https://api.epagos.com/wsdl/2.4/index.php?wsdl` | `"2.0"` (*) |
| v2.5 | Disponible | `https://sandbox.epagos.com/wsdl/2.5/index.php?wsdl` | `https://api.epagos.com/wsdl/2.5/index.php?wsdl` | `"2.0"` (*) |

> (*) **Nota importante:** v2.1, v2.4 y v2.5 son subversiones de v2.0. El valor del parámetro `version` en los métodos SOAP sigue siendo `"2.0"`. La versión se selecciona por el **endpoint WSDL** al que se apunta.

---

## Métodos disponibles por versión

| # | Método | v1.0 | v2.0 | v2.1 | v2.4 | v2.5 | Códigos resp. |
|---|--------|:----:|:----:|:----:|:----:|:----:|---------------|
| 1 | `obtener_token` | ✅ | ✅ | ✅ | ✅ | ✅ | 01xxx |
| 2 | `solicitud_pago` | ✅ | ✅ | ✅ | ✅ | ✅ | 02xxx |
| 3 | `obtener_entidades_pago` | ✅ | ✅ | ✅ | ✅ | ✅ | 03xxx |
| 4 | `obtener_pagos` | ✅ | ✅ | ✅ | ✅ | ✅ | 04xxx |
| 5 | `obtener_rendiciones` | ✅ | ✅ | ✅ | ✅ | ✅ | 05xxx |
| 6 | `obtener_contracargos` | ✅ | ✅ | ✅ | ✅ | ✅ | 06xxx |
| 7 | `obtener_pagos_adicionales` | ✅ | ✅ | ✅ | ✅ | ✅ | 07xxx |
| 8 | `solicitud_pago_lote` | ✅ | ✅ | ✅ | ✅ | ✅ | 08xxx |
| 9 | `solicitud_pago_recurrente` | ✅ | ✅ | ✅ | ✅ | ✅ | 09xxx |
| 10 | `solicitud_pago_recurrente_suscripcion` | ✅ | ✅ | ✅ | ✅ | ✅ | 10xxx |
| 11 | `obtener_tarjetas_cliente` | ✅ | ✅ | ✅ | ✅ | ✅ | 11xxx |
| 12 | `obtener_cuentas_cliente` | ✅ | ✅ | ✅ | ✅ | ✅ | 12xxx |
| 13 | `generar_qr_vinculado` | ✅ | ✅ | ✅ | ✅ | ✅ | 13xxx |
| 14 | `pago_lote` | ❌ | ❌ | ✅ | ✅ | ✅ | 14xxx |
| 15 | `registrar_cuentas_cliente` | ❌ | ❌ | ✅ | ✅ | ✅ | 15xxx |
| 16 | `obtener_resultados_debito` | ❌ | ❌ | ✅ | ✅ | ✅ | 16xxx |
| 17 | `obtener_cajas_qr` | ❌ | ❌ | ✅ | ✅ | ✅ | 18xxx |
| 18 | `generar_orden_qr` | ❌ | ❌ | ✅ | ✅ | ✅ | 19xxx |

**Total:** v1.0/v2.0 = 13 métodos — v2.1/v2.4/v2.5 = 18 métodos

---

## Changelog de diferencias entre versiones

### v1.0 → v2.0

- Cambio de endpoint WSDL: `/wsdl/index.php` → `/wsdl/2.0/index.php`
- Cambio del parámetro `version`: `"1.0"` → `"2.0"`
- Mismos 13 métodos

### v2.0 → v2.1 [actual]

- Endpoint WSDL: `/wsdl/2.0/` → `/wsdl/2.1/`
- Se mantiene `version = "2.0"` (es subversión)
- **5 métodos nuevos:**
  - `pago_lote` — Ejecutar el pago de un lote generado con `solicitud_pago_lote`
  - `registrar_cuentas_cliente` — Registrar cuentas bancarias para débito automático
  - `obtener_resultados_debito` — Consultar resultados de débitos automáticos
  - `obtener_cajas_qr` — Listar cajas QR interoperables estáticas del organismo
  - `generar_orden_qr` — Generar orden de pago asociada a una caja QR
- **Nuevo código de error en `obtener_token`:** `01004` — Versión inválida del protocolo

### v2.1 → v2.4

- Endpoint WSDL: `/wsdl/2.1/` → `/wsdl/2.4/`
- Mismos 18 métodos
- Posibles cambios menores en parámetros opcionales o validaciones

### v2.4 → v2.5

- Endpoint WSDL: `/wsdl/2.4/` → `/wsdl/2.5/`
- Mismos 18 métodos
- **Campos nuevos para E-Checkout** (ver archivo `19-changelog-v2.5.md`):
  - `opc_una_cuota` — Forzar pago en 1 cuota
  - `opc_T30_cbu_destino` — CBU destino para Transferencias 3.0
  - `opc_T30_cuit_destino` — CUIT destino
  - `opc_T30_nombre_destino` — Nombre destino
  - `opc_operaciones_lote` — Acreditación automática de lotes
  - `detalle_operacion` — Desglose de ítems en E-Checkout
  - `detalle_operacion_visible` — Visibilidad del desglose
  - `identificador_externo_4` — Cuarto identificador adicional

---

## ¿Qué versión usar?

| Caso de uso | Versión recomendada |
|-------------|--------------------|
| Nueva integración estándar | **v2.1** (actual, recomendada por ePagos) |
| E-Checkout con campos v2.5 (cuotas, T30, lotes) | **v2.5** |
| Integración legacy existente en v1.0 | Migrar a v2.1 cuando sea posible |
| QR interoperable estático / débito automático | **v2.1** mínimo (métodos no disponibles en v1.0/v2.0) |

---

## Panel de control

| Entorno | URL |
|---------|-----|
| Sandbox (pruebas) | https://portalsandbox.epagos.com |
| Producción | https://portal.epagos.com |

---

## Tablas de referencia

Las tablas de referencia (IDs de medios de pago, tipos de documento, provincias, tarjetas de prueba) son **compartidas entre todas las versiones**. Ver `17-tablas-de-referencia.md`.
