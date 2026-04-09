# Tablas de referencia

Tablas de valores de referencia para usar al integrar la API SOAP, E-Checkout y Botón de Pago. Extraídas de la Referencia de API v2.5.

---

## Tarjetas de prueba (Sandbox)

### Tarjetas de crédito

| Proveedor | Número | Código seguridad | Vencimiento |
|-----------|--------|-----------------|-------------|
| VISA | 4507 9900 0000 4905 | 123 | 12/2026 |
| American Express (AMEX) | 3766 341249 71005 | 1234 | 12/2026 |
| Mastercard | 5323 6222 7777 7785 | 123 | 12/2026 |

### Tarjetas de débito

| Proveedor | Número | Código seguridad | Vencimiento |
|-----------|--------|-----------------|-------------|
| Visa Débito | 4517 7210 0485 6075 | 123 | 12/2026 |
| Mastercard Débito | 5299 9100 1000 0015 | 123 | 12/2026 |

> La fecha de vencimiento **no se valida** en Sandbox. Cualquier fecha mayor a la actual funciona.
> Para forzar un rechazo, usar **Nombre Titular = "CALL"**.

---

## CBU y CUITs de prueba (Sandbox)

### Para DEBIN

| CBU | Alias | CUIT | Tipo persona |
|-----|-------|------|-------------|
| 3220002204000040970011 | prueba.debin | 20123456781 | Física |
| 3220001101000040970011 | prueba.debin.2 | 27123456781 | Física |
| 3220001101000040970011 | prueba.debin.2 | 30111111110 | Jurídica |
| 0070314530004001400167 | prueba.debin.3 | 20004940548 | Física |

### Para Débito inmediato

| CBU | Escenario | CUIT | Tipo persona |
|-----|-----------|------|-------------|
| 3220001801000020816200 | Débito inmediato aceptado | 20312528046 | Física |
| 3220002204000040970011 | Débito inmediato pendiente | 20312528046 | Física |
| cualquier CBU de la lista DEBIN | Débito inmediato rechazado | — | Física |

---

## Medios de pago (IDs)

Usar en `fp_permitidas`, `fp_excluidas`, `fp_defecto` del E-Checkout / Botón de Pago.

| Proveedor | Modalidad | ID |
|-----------|-----------|-----|
| VISA | Tarjeta de crédito – Online | 1 |
| American Express (AMEX) | Tarjeta de crédito – Online | 2 |
| Mastercard | Tarjeta de crédito – Online | 3 |
| Pago Fácil | Presencial – Efectivo | 4 |
| Rapi Pago | Presencial – Efectivo | 5 |
| Pago Mis Cuentas | Publicación de deuda | 6 |
| Red Link – Web (*) | Publicación de deuda | 7 |
| Banco Nación | Presencial – Efectivo | 8 |
| ArgenCard | Tarjeta de crédito – Online | 9 |
| Cabal Crédito | Tarjeta de crédito – Online | 10 |
| Diners Club | Tarjeta de crédito – Online | 11 |
| BanCor – Banco de Córdoba | Presencial – Efectivo | 12 |
| E-Transferencia (*) | Transferencia bancaria | 17 |
| VISA Débito | Tarjeta de débito – Online | 14 |
| Maestro Débito | Tarjeta de débito – Online | 15 |
| Red Link | Publicación de deuda | 26 |
| Billetera ePagos (*) | Billetera online | 27 |
| Cabal Débito | Tarjeta de débito – Online | 28 |
| Cobro Express | Presencial – Efectivo | 24 |
| E-Cajero (*) | Presencial – Efectivo | 25 |
| Banco de Neuquén | Presencial – Efectivo | 40 |
| Mastercard Débito | Tarjeta de débito – Online | 41 |
| Débito directo (*) | Transferencia bancaria | 42 |
| IVR (*) | Cobro telefónico | 43 |
| Transferencias 3.0 (*) | Transferencia bancaria | 44 |
| Billetera Cripto (*) | Criptomonedas | 45 |
| Billetera MercadoPago (*) | Billetera online | 47 |
| Billetera MODO (*) | Billetera online | 48 |
| Débito inmediato (*) (**) | Transferencia bancaria | 51 |
| Habitualista Débito | Tarjeta de débito – Online | 52 |
| Banco Supervielle | Presencial – Efectivo | 53 |
| Naranja X | Tarjeta de crédito – Online | 54 |
| Tarjeta mandataria (*) | Presencial – Efectivo | 35 |
| Multipago | Presencial – Efectivo | 36 |
| Banco de Corrientes (*) | Presencial – Efectivo | 39 |
| Banco Santander (*) | Presencial – Efectivo | 50 |

> (*) No disponible para todos los organismos.
> (**) Solo para uso desde la API SOAP. Ideal para emisiones masivas o cobros recurrentes.

### Medios desactivados (referencia histórica)

| Proveedor | ID |
|-----------|-----|
| RIPSA Pagos | 16 |
| Provincia NET | 13 |
| Plus Pagos | 18 |
| Entre Ríos Servicios (Plus pagos) | 19 |
| Santa Fe Servicios (Plus pagos) | 20 |
| San Juan Servicios (Plus pagos) | 21 |
| Santa Cruz Servicios (Plus pagos) | 22 |
| Corrientes Servicios (Plus pagos) | 23 |
| Chubut Pagos (Provincia NET) | 33 |
| Pronto Pagos (Provincia NET) | 32 |
| Pampa Pagos (Provincia NET) | 31 |
| Formo Pagos (Provincia NET) | 30 |
| Banco Piano (Provincia NET) | 29 |
| BICA Ágil | 37 |

---

## Tipos de forma de pago (IDs)

Usar en `tp_excluidos` del E-Checkout.

| Descripción | ID |
|-------------|-----|
| Presencial (cobro en efectivo) | 1 |
| Tarjeta de crédito | 2 |
| Tarjeta de débito | 3 |
| Prepago | 4 |
| Publicación de deuda (cajeros y homebanking) | 5 |
| Billetera ePagos | 6 |
| Transferencias bancarias | 7 |
| Cobro telefónico | 8 |
| Criptomonedas | 9 |

> Tip: Para restringir a solo tarjetas y débito directo (recurrencia), usar `tp_excluidos = "1,3,4,5,6,7"` o `fp_permitidas = 42`.

---

## Tipos de documento (IDs)

Usar en `tipo_doc_pagador` del E-Checkout / API.

| Descripción | ID |
|-------------|-----|
| DNI | 1 |
| LE | 2 |
| LC | 3 |
| DNI Extranjero | 4 |
| Cédula Extranjero | 5 |
| Pasaporte | 6 |
| No consta | 7 |
| Cédula de identidad | 8 |
| CUIT | 9 |

---

## Códigos de provincia (IDs)

Usar en `provincia_dom_pagador` del E-Checkout / API.

| Provincia | ID |
|-----------|-----|
| Buenos Aires | 1 |
| Ciudad Autónoma de Buenos Aires | 2 |
| Catamarca | 3 |
| Córdoba | 4 |
| Corrientes | 5 |
| Chaco | 6 |
| Chubut | 7 |
| Entre Ríos | 8 |
| Formosa | 9 |
| Jujuy | 10 |
| La Pampa | 11 |
| La Rioja | 12 |
| Mendoza | 13 |
| Misiones | 14 |
| Neuquén | 15 |
| Río Negro | 16 |
| Salta | 17 |
| San Juan | 18 |
| San Luis | 19 |
| Santa Cruz | 20 |
| Santa Fe | 21 |
| Santiago del Estero | 22 |
| Tierra del Fuego | 23 |
| Tucumán | 24 |
