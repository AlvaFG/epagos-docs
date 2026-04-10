# ePagos Argentina - Documentacion Tecnica y Skills para Claude Code

Documentacion completa de la API de [ePagos](https://www.epagos.com), pasarela de pagos argentina regulada por BCRA. Incluye skills para [Claude Code](https://claude.ai/claude-code) que permiten a agentes de IA integrar pagos con ePagos de forma autonoma.

## Que es ePagos?

ePagos es un PSP (Payment Service Provider) argentino que consolida 30+ medios de pago en una sola integracion:

- Tarjetas de credito/debito (VISA, Mastercard, AMEX, Cabal, Naranja X, etc.)
- Billeteras digitales (MODO, MercadoPago, ePagos)
- Transferencias bancarias (Transferencias 3.0, DEBIN)
- Efectivo (PagoFacil, Rapipago, Cobro Express)
- QR interoperable
- Pagos recurrentes y suscripciones

## Contenido

### `/docs/` - Documentacion tecnica completa

Extraida de la documentacion oficial de ePagos (abril 2026):

| Archivo | Contenido |
|---------|-----------|
| `01-credenciales-sandbox.md` | Credenciales y configuracion del sandbox |
| `02-pruebas-checkout-boton-pago.md` | Plan de pruebas para checkout y boton de pago |
| `03-pruebas-emision-masiva.md` | Plan de pruebas para emision masiva |
| `04-control-operaciones.md` | 4 niveles de verificacion de pagos |
| `05-tipo-operaciones.md` | Codigos de tipo de operacion |
| `epagos-documentacion-introduccion.md` | Introduccion y overview de la plataforma |
| `epagos-boton-pago-documentacion.md` | Boton de pago JavaScript (integracion client-side) |
| `epagos-echeckout-documentacion-tecnica.md` | E-Checkout completo (redirect flow, callbacks, response codes) |
| `epagos-webhook-documentacion.md` | Webhooks (payload, tipos, configuracion) |
| `epagos-recursos-tecnicos.md` | URLs, logos, recursos graficos |
| `ePagos-API-Solicitud-Pago.md` | API SOAP: solicitud de pago |
| `ePagos_Conciliacion_API_Documentacion.md` | API SOAP: conciliacion (obtener_pagos, contracargos) |
| `Documentacion_Tecnica_Rendiciones_ePagos.md` | API SOAP: rendiciones diarias |
| `ePagos_Documentacion_Pagos_Recurrentes.md` | Pagos recurrentes y suscripciones |
| `emision_masiva_epagos.md` | Emision masiva (lotes, codigo de barras, QR) |

### `/skill/` - Skills para Claude Code (4 skills focalizadas)

| Skill | Usar cuando... | Lineas |
|-------|---------------|--------|
| `epagos-echeckout/SKILL.md` | Implementar checkout, callbacks ok/error, webhooks, boton de pago JS | ~380 |
| `epagos-reconciliation/SKILL.md` | Conciliacion diaria, obtener_pagos, rendiciones, contracargos | ~270 |
| `epagos-soap-payments/SKILL.md` | Crear pagos via SOAP, QR, emision masiva, pagos recurrentes, POS | ~480 |
| `epagos-reference/SKILL.md` | Buscar un ID de medio de pago, codigo de error, URL, tarjeta de test | ~400 |

Cada skill es independiente y se carga solo cuando es relevante. Esto reduce tokens vs un unico archivo monolitico.

## Como usar los Skills en Claude Code

Copia las 4 skills a tu proyecto:

```bash
cp -r skill/* .claude/skills/
```

Claude Code cargara automaticamente el skill relevante cuando trabajes en integracion de pagos con ePagos.

## Opciones de integracion

ePagos ofrece 3 formas de integrar:

| Opcion | Complejidad | PCI requerido | Skill | Descripcion |
|--------|-------------|---------------|-------|-------------|
| **Boton de Pago** | Basica | No | `epagos-echeckout` | JavaScript client-side, 1 paso |
| **E-Checkout** | Intermedia | No | `epagos-echeckout` | Redirect server-side, 2 pasos |
| **API SOAP** | Avanzada | Si (para tarjetas) | `epagos-soap-payments` | Server-to-server completo |

Para la mayoria de los casos, **E-Checkout** es la opcion recomendada: no requiere PCI, ePagos maneja toda la UI de pago, y soporta todos los medios de pago.

## URLs

| Proposito | Sandbox | Produccion | Dominio |
|-----------|---------|------------|---------|
| Token (POST) | `https://sandbox.epagos.com/post.php` | `https://api.epagos.com/post.php` | .com |
| Redirect checkout | `https://postsandbox.epagos.com` | `https://post.epagos.com` | .com |
| SOAP WSDL v2.0 (conciliacion) | `https://sandbox.epagos.net/wsdl/2.0/index.php?wsdl` | `https://api.epagos.net/wsdl/2.0/index.php?wsdl` | .net |
| SOAP WSDL v2.1 (pagos) | `https://sandbox.epagos.com/wsdl/2.1/index.php?wsdl` | `https://api.epagos.com/wsdl/2.1/index.php?wsdl` | .com |
| JS library | `https://sandbox.epagos.com/quickstart/epagos.min.js` | `https://api.epagos.com/quickstart/epagos.min.js` | .com |
| Portal | `https://portalsandbox.epagos.com` | `https://portal.epagos.com` | .com |

**Nota:** Los dominios `.com` y `.net` NO son intercambiables. `.com` = eCheckout + SOAP v2.1. `.net` = SOAP v2.0 (conciliacion) + namespace SOAP.

## Tarjetas de prueba (sandbox)

| Proveedor | Numero | CVV | Vencimiento |
|-----------|--------|-----|-------------|
| VISA (credito) | 4507 9900 0000 4905 | 123 | 12/2026 |
| Mastercard (credito) | 5323 6222 7777 7785 | 123 | 12/2026 |
| AMEX (credito) | 3766 341249 71005 | 1234 | 12/2026 |
| VISA (debito) | 4517 7210 0485 6075 | 123 | 12/2026 |
| MC (debito) | 5299 9100 1000 0015 | 123 | 12/2026 |

> En sandbox la fecha de vencimiento no se valida. Para forzar un rechazo, usar Nombre Titular = "CALL".

## SDK oficial

ePagos solo tiene SDK para PHP y .NET:
- PHP: [github.com/epagos/api_php](https://github.com/epagos/api_php)
- .NET: [github.com/epagos/api_net_core](https://github.com/epagos/api_net_core)

No hay SDK de Node.js/JavaScript. Los skills incluyen toda la informacion necesaria para construir un cliente propio en cualquier lenguaje.

## Recursos

- [Documentacion oficial](https://www.epagos.com/desarrolladores.php)
- [API Reference](https://www.epagos.com/templates/desarrolladores/referencia.php)
- [Status page](https://status.epagos.com.ar/)
- Soporte: soporte@epagos.com.ar

## Licencia

La documentacion de ePagos pertenece a E-Pagos S.A. (CUIT: 30-71508429-1). Este repositorio es una recopilacion con fines de integracion y referencia.
