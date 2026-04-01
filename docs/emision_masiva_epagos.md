# Documentación Técnica: Emisión Masiva en ePagos

## 1. Introducción a Emisión Masiva

La emisión masiva permite la publicación periódica de deuda por parte de organismos. Se integra con ePagos mediante códigos de barras y códigos QR en las boletas generadas. Disponible tanto para impresión como para envío por email en PDF.

**Características principales:**
- Códigos de barras (código ePagos)
- Códigos QR (con imagen base64)
- Procesamiento de lotes de hasta 50 operaciones
- Integración via Panel de Control o API SOAP

---

## 2. Métodos de API

### 2.1 Solicitud de Pago Individual

**Endpoint:** `solicitud_pago`

**Descripción:** Genera una nueva operación en el sistema.

#### Parámetros de Entrada

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo (valor fijo: "2.0") |
| `tipo_operacion` | Set | Tipo de operación (valor fijo: "op_pago") |
| `credenciales` | Estructura | Datos de credenciales |
| `operacion` | Estructura | Datos de la operación |
| `fp` | Array | Formas de pago |
| `convenio` | Numérico entero | Número de convenio asignado por ePagos |

#### Estructura: DatosCredenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | Token de acceso |

#### Estructura: DatosOperacionPago

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `numero_operacion` | Texto (100) | Código externo del cliente (opcional) |
| `identificador_externo_2` | Texto (100) | Identificador adicional (opcional) |
| `identificador_externo_3` | Texto (512) | Identificador adicional (opcional) |
| `identificador_externo_4` | Texto (65.535) | Identificador adicional (opcional) |
| `identificador_cliente` | Texto | Identificador para recurrencia (opcional) |
| `id_moneda_operacion` | Numérico entero | Código de moneda (valor fijo: 1) |
| `monto_operacion` | Numérico decimal | Importe con dos decimales |
| `opc_pdf` | Booleano | Incluir PDF del comprobante (defecto: true) |
| `opc_fecha_vencimiento` | Fecha | Vencimiento de boleta (AAAA-MM-DD) |
| `opc_devolver_qr` | Booleano | Devolver imagen QR (defecto: false) |
| `opc_devolver_codbarras` | Booleano | Devolver imagen código barras (defecto: false) |
| `opc_generar_pdf` | Booleano | Generar PDF de boleta (defecto: true) |
| `detalle_operacion` | Estructura | Detalles/items de la operación |
| `pagador` | Array | Datos del pagador |
| `fecha_2do_venc` | Fecha | Segundo vencimiento (AAAA-MM-DD) |
| `monto_operacion_2do_venc` | Numérico decimal | Importe segundo vencimiento |
| `tipo_operacion` | Numérico entero | Tipo de deuda para agrupar |
| `codigo_publicacion` | Numérico entero | Código de publicación homebanking |
| `url_boleta` | Texto (256) | URL de alojamiento de boleta |
| `opc_T30_cerrado` | Booleano | QR interoperable cerrado (defecto: true) |
| `opc_T30_reutilizable` | Booleano | QR interoperable reutilizable (defecto: false) |
| `opc_T30_requiere_orden` | Booleano | QR requiere orden previa (defecto: false) |

#### Estructura: DetalleOperacion

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_item` | Numérico entero | Código o número de item |
| `desc_item` | Texto | Descripción del item |
| `monto_item` | Numérico decimal | Importe del item |
| `cantidad_item` | Numérico entero | Cantidad de items |

#### Estructura: Pagador

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `nombre_pagador` | Texto | Nombre del pagador (opcional) |
| `apellido_pagador` | Texto | Apellido del pagador (opcional) |
| `fechanac_pagador` | Texto | Fecha de nacimiento (AAAA-MM-DD) (opcional) |
| `email_pagador` | Texto | Email del pagador |
| `identificacion_pagador` | Estructura | Identificación del pagador |
| `domicilio_pagador` | Estructura | Domicilio del pagador (opcional) |
| `telefono_pagador` | Estructura | Teléfono del pagador (opcional) |
| `cbu_pagador` | Numérico entero | CBU del banco (para DEBIN) |

#### Estructura: PagadorIdentificacion

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `tipo_doc_pagador` | Numérico entero | Tipo de documento (ver tabla documentos) |
| `numero_doc_pagador` | Numérico entero | Número de documento |
| `cuit_doc_pagador` | Numérico entero | Número de CUIT |

#### Estructura: PagadorDomicilio

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `calle_dom_pagador` | Texto | Nombre de la calle |
| `numero_dom_pagador` | Texto | Número de la calle |
| `adicional_dom_pagador` | Texto | Adicional de la calle |
| `cp_dom_pagador` | Texto | Código postal |
| `ciudad_dom_pagador` | Texto | Ciudad |
| `provincia_dom_pagador` | Numérico entero | Código de provincia |
| `pais_dom_pagador` | Numérico entero | Código de país (valor fijo: 13 = Argentina) |

#### Estructura: PagadorTelefono

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `codigo_telef_pagador` | Numérico entero | Código de área |
| `numero_telef_pagador` | Numérico entero | Número de teléfono |

#### Estructura: FormaPago

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_fp` | Numérico entero | Código de forma de pago |
| `monto_fp` | Numérico decimal | Importe de la forma de pago |
| `tarjeta` | Estructura | Datos de tarjeta (solo si es tarjeta) |

#### Estructura: FormaPagoTarjeta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `numero_tarjeta_fp` | Numérico entero | Número de tarjeta |
| `banco_tarjeta_fp` | Texto | Banco emisor |
| `vencimiento_tarjeta_fp` | Estructura | Fecha de vencimiento |
| `codigo_seg_tarjeta_fp` | Texto | Código de seguridad |
| `cuotas_tarjeta_fp` | Numérico entero | Cantidad de cuotas |
| `titular_tarjeta_fp` | Texto | Nombre del titular |
| `identificacion_tarjeta_fp` | Estructura | Identificación del titular |
| `fechanac_tarjeta_fp` | Fecha | Fecha de nacimiento (AAAA-MM-DD) |
| `direccion_tarjeta_fp` | Estructura | Dirección del titular |

#### Respuesta: RespuestaFormaPago

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de respuesta |
| `token` | Texto | Token único de la solicitud |
| `id_transaccion` | Numérico entero | Código único de operación |
| `id_organismo` | Numérico entero | Número de organismo |
| `convenio` | Numérico entero | Número de convenio |
| `numero_operacion` | Numérico entero | Número interno de operación |
| `codigo_pago_fp` | Numérico entero | Código corto de pago (efectivo/homebanking) |
| `codigo_barras_fp` | Texto | Código de barras completo |
| `fechavenc_fp` | Fecha | Fecha de vencimiento (AAAA-MM-DD) |
| `importe_fp` | Numérico decimal | Importe de la operación |
| `pdf` | Base64 | Contenido del comprobante PDF |
| `codigo_barras_imagen` | Base64 | Código de barras como PNG (base64) |
| `qr_imagen` | Base64 | Código QR como PNG (base64) |
| `qr_imagen_T30` | Base64 | Código QR Transferencias 3.0 como PNG |
| `qr_texto_T30` | Texto | Contenido del QR Transferencias 3.0 |
| `respuesta_entidad_cobro` | Texto | Respuesta del medio de pago |
| `codigo_pmc` | Texto | Código en Pago Mis Cuentas |
| `codigo_link` | Texto | Código en Red Link |
| `url_qr` | Texto | URL para pago en línea |
| `cuit_cuenta` | Numérico entero | CUIT de la cuenta |
| `cbu_cuenta` | Texto | CBU/CVU o alias de la cuenta |
| `barras_adicionales` | Array | Códigos de barras adicionales |

#### Estructura: BarrasAdicionales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `codigo_barra` | Texto | Código de barra adicional |
| `codigo_de_pago` | Texto | Código de pago sin factura |
| `fechavenc` | Fecha | Fecha de vencimiento (AAAA-MM-DD) |
| `importe` | Numérico decimal | Importe de la barra adicional |

#### Códigos de Respuesta

| id_resp | Tipo | Respuesta |
|---------|------|-----------|
| 02001 | Correcta | Pago acreditado |
| 02002 | Correcta | Pago pendiente |
| 02003 | Error | Error al validar el token |
| 02004 | Error | Pago cancelado/rechazado |
| 02005 | Error | Error interno al procesar pago |
| 02006 | Error | Error al validar parámetro |
| 02007 | Error | Usuario canceló el pago |
| 02008 | Error | Montos y detalles no coinciden |
| 02009 | Error | Forma de pago no disponible |
| 02010 | Error | Versión inválida del protocolo |

---

### 2.2 Solicitud de Pago por Lote

**Endpoint:** `solicitud_pago_lote`

**Descripción:** Realiza emisión de múltiples operaciones en una sola solicitud.

> ⚠️ **Límite:** Máximo **50 operaciones** por lote.

#### Parámetros de Entrada

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo (valor fijo: "2.0") |
| `tipo_operacion` | Set | Tipo de operación (valor fijo: "op_pago") |
| `credenciales` | Estructura | Datos de credenciales |
| `lote` | Array | Vector de estructuras TipoLote |

#### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | Token devuelto por obtener_token |

#### Estructura: TipoLote

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `fp` | Array | Array de estructuras DatosFormaPago |
| `operacion` | Estructura | Datos de la operación (DatosOperacionPago) |
| `convenio` | Numérico entero | Número de convenio asignado por ePagos |

#### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de respuesta |
| `token` | Texto | Token utilizado en consulta |
| `id_organismo` | Numérico entero | Código de organismo |
| `lote` | Array | Array de RespuestaLote |

#### Estructura: RespuestaLote

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_transaccion` | Numérico entero | Número único de operación |
| `numero_operacion` | Texto | Número de operación del cliente |
| `convenio` | Numérico entero | Número de convenio ePagos |
| `respuesta_forma_pago_array` | Array | Array de RespuestaFormaPago |

#### Códigos de Respuesta

| id_resp | Tipo | Respuesta |
|---------|------|-----------|
| 08001 | Correcta | Pagos en lotes procesados |
| 08002 | Error | Error al validar token |
| 08003 | Error | Error interno al procesar pagos |
| 08004 | Error | Error al validar parámetro: [_parametro_] |
| 08005 | Error | El tamaño del lote supera el límite permitido de [_limite_] |
| 08006 | Error | Error al procesar pagos |
| 08007 | Error | Montos y detalles no coinciden |
| 08008 | Error | Versión inválida del protocolo |

---

### 2.3 Generar QR Vinculado

**Endpoint:** `generar_qr_vinculado`

**Descripción:** Genera un QR selector para múltiples operaciones en una boleta. Útil para boletas con varias cuotas (Cuota 1, Cuota 2, Anual, etc.).

#### Parámetros de Entrada

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo (valor fijo: "2.0") |
| `credenciales` | Estructura | Datos de credenciales |
| `operaciones_qr` | Array | Array de estructuras OperacionQR (mínimo 1) |

#### Estructura: OperacionQR

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_transaccion` | Numérico entero | ID de transacción devuelto por ePagos |
| `etiqueta` | Texto | Texto que representa la operación en el selector |

#### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de respuesta |
| `qr` | Base64 | Imagen del QR codificada en base64 |

#### Códigos de Respuesta

| id_resp | Tipo | Respuesta |
|---------|------|-----------|
| 14001 | Correcta | Codigo QR generado con exito |
| 14002 | Error | Error al registrar la operación vinculada: #[_parametro_] |
| 14003 | Error | La Operación: #[_parametro_] no pertenece al organismo |
| 14004 | Error | El hash [_parametro_] ya se encuentra registrado |
| 14005 | Error | Hay operaciones duplicadas en el envío |
| 14006 | Error | Versión inválida del protocolo |

---

## 3. Tablas de Referencia

### 3.1 Medios de Pago Disponibles

| Proveedor | Modalidad | ID | Notas |
|-----------|-----------|-----|-------|
| VISA | Tarjeta de crédito - Online | 1 | |
| American Express (AMEX) | Tarjeta de crédito - Online | 2 | |
| Mastercard | Tarjeta de crédito - Online | 3 | |
| ArgenCard | Tarjeta de crédito - Online | 9 | |
| Cabal Crédito | Tarjeta de crédito - Online | 10 | |
| Diners Club | Tarjeta de crédito - Online | 11 | |
| Naranja X | Tarjeta de crédito - Online | 54 | |
| VISA Débito | Tarjeta de débito - Online | 14 | |
| Maestro Débito | Tarjeta de débito - Online | 15 | |
| Cabal Débito | Tarjeta de débito - Online | 28 | |
| Mastercard Débito | Tarjeta de débito - Online | 41 | |
| Habitualista Débito | Tarjeta de débito - Online | 52 | |
| Pago Fácil | Presencial - Efectivo | 4 | |
| RapiPago | Presencial - Efectivo | 5 | |
| Banco Nación | Presencial - Efectivo | 8 | |
| BanCor - Banco de Córdoba | Presencial - Efectivo | 12 | |
| Banco de Corrientes | Presencial - Efectivo | 39 | (*) |
| Banco Santander | Presencial - Efectivo | 50 | (*) |
| Banco de Neuquén | Presencial - Efectivo | 40 | |
| E-Cajero | Presencial - Efectivo | 25 | (*) |
| Tarjeta mandataria | Presencial - Efectivo | 35 | (*) |
| Multipago | Presencial - Efectivo | 36 | |
| Cobro Express | Presencial - Efectivo | 24 | |
| Banco Supervielle | Presencial - Efectivo | 53 | |
| Pago Mis Cuentas | Publicación de deuda | 6 | |
| Red Link - Web | Publicación de deuda | 7 | (*) |
| Red Link | Publicación de deuda | 26 | |
| Billetera ePagos | Billetera online | 27 | (*) |
| Billetera MODO | Billetera online | 48 | (*) |
| Billetera MercadoPago | Billetera online | 47 | (*) |
| E-Transferencia | Transferencia bancaria | 17 | (*) |
| DEBIN | Transferencia bancaria | 38 | (*) |
| Débito Directo | Transferencia bancaria | 42 | (*) |
| Débito Inmediato | Transferencia bancaria | 51 | (**) |
| Transferencias 3.0 | Transferencia bancaria | 44 | (*) |
| IVR | Cobro telefónico | 43 | (*) |
| Billetera Cripto | Criptomonedas | 45 | (*) |
| Combinado | Presencial + Publicación | 34 | (**) |

> (*) No disponible para todos los organismos  
> (**) Solo desde API SOAP — ideal para emisiones masivas o cobros recurrentes

### 3.2 Tipos de Forma de Pago

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

### 3.3 Códigos de Provincia

| Descripción | ID |
|-------------|-----|
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
| Rio Negro | 16 |
| Salta | 17 |
| San Juan | 18 |
| San Luis | 19 |
| Santa Cruz | 20 |
| Santa Fe | 21 |
| Santiago del Estero | 22 |
| Tierra del Fuego | 23 |
| Tucumán | 24 |

### 3.4 Tipos de Documento

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

## 4. Códigos de Barras y Códigos QR

### 4.1 Códigos de Barras

| Campo | Descripción |
|-------|-------------|
| `codigo_barras_fp` | Código de barras completo en texto |
| `codigo_barras_imagen` | Imagen PNG del código en Base64 |
| `opc_devolver_codbarras` | Activar para recibir la imagen (defecto: false) |
| `codigo_pago_fp` | Código corto para pagos en efectivo/homebanking |

### 4.2 Tipos de Código QR

| Tipo | Campo Respuesta | Activación | Formato |
|------|----------------|------------|---------|
| QR Estándar | `qr_imagen` | `opc_devolver_qr: true` | PNG / Base64 |
| QR Transferencias 3.0 | `qr_imagen_T30` | `opc_devolver_qr: true` | PNG / Base64 |
| Texto QR T30 | `qr_texto_T30` | `opc_devolver_qr: true` | Texto plano |
| URL de pago | `url_qr` | Siempre disponible | URL |
| QR Vinculado | Vía `generar_qr_vinculado` | Método separado | PNG / Base64 |

### 4.3 Opciones para QR Interoperable (T30)

| Parámetro | Tipo | Defecto | Descripción |
|-----------|------|---------|-------------|
| `opc_T30_cerrado` | Booleano | true | El usuario no puede modificar el monto |
| `opc_T30_reutilizable` | Booleano | false | Permite múltiples pagos con el mismo QR |
| `opc_T30_requiere_orden` | Booleano | false | Solo válido en QRs cerrados; requiere orden previa |

---

## 5. Ejemplos de Solicitudes y Respuestas

### 5.1 Solicitud de Pago Individual

```json
{
  "version": "2.0",
  "tipo_operacion": "op_pago",
  "credenciales": {
    "id_organismo": 70,
    "token": "token_abc123xyz"
  },
  "operacion": {
    "numero_operacion": "OP-2026-001",
    "identificador_cliente": "CLIENTE-001",
    "id_moneda_operacion": 1,
    "monto_operacion": 1500.50,
    "opc_devolver_qr": true,
    "opc_devolver_codbarras": true,
    "opc_pdf": true,
    "opc_generar_pdf": true,
    "opc_fecha_vencimiento": "2026-05-15",
    "detalle_operacion": {
      "id_item": 1,
      "desc_item": "Cuota de servicios",
      "monto_item": 1500.50,
      "cantidad_item": 1
    },
    "pagador": {
      "nombre_pagador": "Juan",
      "apellido_pagador": "Pérez",
      "email_pagador": "juan.perez@example.com",
      "identificacion_pagador": {
        "tipo_doc_pagador": 1,
        "numero_doc_pagador": 12345678
      }
    }
  },
  "fp": [
    {
      "id_fp": 4,
      "monto_fp": 1500.50
    }
  ],
  "convenio": 123
}
```

### 5.2 Respuesta de Pago Individual

```json
{
  "id_resp": "02002",
  "respuesta": "Pago pendiente",
  "id_transaccion": 987654321,
  "codigo_pago_fp": "1234567890",
  "codigo_barras_fp": "12345678901234567890123456789012345",
  "fechavenc_fp": "2026-05-15",
  "importe_fp": 1500.50,
  "codigo_barras_imagen": "iVBORw0KGgoAAAANSUhEUgAA...",
  "qr_imagen": "iVBORw0KGgoAAAANSUhEUgAA...",
  "qr_imagen_T30": "iVBORw0KGgoAAAANSUhEUgAA...",
  "qr_texto_T30": "00020126580014...",
  "url_qr": "https://portal.epagos.com/pagar/qr/abc123xyz",
  "codigo_pmc": "PMC-123456",
  "codigo_link": "RL-654321",
  "token": "token_abc123xyz",
  "id_organismo": 70,
  "convenio": 123
}
```

### 5.3 Solicitud de Lote (múltiples operaciones)

```json
{
  "version": "2.0",
  "tipo_operacion": "op_pago",
  "credenciales": {
    "id_organismo": 70,
    "token": "token_abc123xyz"
  },
  "lote": [
    {
      "operacion": {
        "numero_operacion": "OP-2026-001",
        "monto_operacion": 1000.00,
        "id_moneda_operacion": 1,
        "opc_devolver_qr": true,
        "opc_generar_pdf": false,
        "pagador": {
          "email_pagador": "usuario1@example.com",
          "identificacion_pagador": {
            "tipo_doc_pagador": 1,
            "numero_doc_pagador": 12345678
          }
        }
      },
      "fp": [{ "id_fp": 4, "monto_fp": 1000.00 }],
      "convenio": 123
    },
    {
      "operacion": {
        "numero_operacion": "OP-2026-002",
        "monto_operacion": 2000.00,
        "id_moneda_operacion": 1,
        "opc_devolver_qr": true,
        "opc_generar_pdf": false,
        "pagador": {
          "email_pagador": "usuario2@example.com",
          "identificacion_pagador": {
            "tipo_doc_pagador": 1,
            "numero_doc_pagador": 87654321
          }
        }
      },
      "fp": [{ "id_fp": 4, "monto_fp": 2000.00 }],
      "convenio": 123
    }
  ]
}
```

### 5.4 Generar QR Vinculado (múltiples cuotas en una boleta)

```json
{
  "version": "2.0",
  "credenciales": {
    "id_organismo": 70,
    "token": "token_abc123xyz"
  },
  "operaciones_qr": [
    { "id_transaccion": 987654321, "etiqueta": "Cuota 1" },
    { "id_transaccion": 987654322, "etiqueta": "Cuota 2" },
    { "id_transaccion": 987654323, "etiqueta": "Cuota Anual" }
  ]
}
```

**Respuesta:**

```json
{
  "id_resp": "14001",
  "respuesta": "Codigo QR generado con exito",
  "qr": "iVBORw0KGgoAAAANSUhEUgAA..."
}
```

---

## 6. Endpoints y Ambientes

### Sandbox (Pruebas)

| Elemento | URL |
|----------|-----|
| WSDL SOAP | https://sandbox.epagos.com/wsdl/2.1/index.php?wsdl |
| Panel de Control | https://portalsandbox.epagos.com |

### Producción

| Elemento | URL |
|----------|-----|
| WSDL SOAP | https://api.epagos.com/wsdl/2.1/index.php?wsdl |
| Panel de Control | https://portal.epagos.com |

> **Nota:** Se trata de una subversión de la v2.0. El número a informar en los métodos sigue siendo **2.0**.

---

## 7. Validaciones Importantes

| Regla | Detalle |
|-------|---------|
| Formato decimal | Punto (.) como separador. Ej: `1234.12` o `123` |
| Formato fecha | AAAA-MM-DD |
| Límite de lote | Máximo 50 operaciones por solicitud |
| Suma de items | `sum(monto_item × cantidad_item)` debe igualar `monto_operacion` |
| Suma de formas de pago | `sum(monto_fp)` debe igualar `monto_operacion` |
| PDF en masivas | Desactivar `opc_generar_pdf` acelera el procesamiento |
| Código de barras | Usar un único código por boleta para claridad del usuario |

---

## 8. Tarjetas de Prueba (Sandbox)

### Tarjetas de Crédito

| Proveedor | Número | Código Seguridad | Vencimiento |
|-----------|--------|-----------------|-------------|
| VISA | 4507 9900 0000 4905 | 123 | 12-2026 |
| American Express | 3766 341249 71005 | 1234 | 12-2026 |
| Mastercard | 5323 6222 7777 7785 | 123 | 12-2026 |

### Tarjetas de Débito

| Proveedor | Número | Código Seguridad | Vencimiento |
|-----------|--------|-----------------|-------------|
| Visa Débito | 4517 7210 0485 6075 | 123 | 12-2026 |
| Mastercard Débito | 5299 9100 1000 0015 | 123 | 12-2026 |

> En Sandbox, la fecha de vencimiento no se valida.

---

## 9. CBUs y CUITs de Prueba

### Para DEBIN

| CBU | Alias | CUIT | Tipo |
|-----|-------|------|------|
| 3220002204000040970011 | prueba.debin | 20123456781 | Física |
| 3220001101000040970011 | prueba.debin.2 | 27123456781 | Física |
| 3220001101000040970011 | prueba.debin.2 | 30111111110 | Jurídica |
| 0070314530004001400167 | prueba.debin.3 | 20004940548 | Física |

### Para Débito Inmediato

| CBU | Escenario | CUIT | Tipo |
|-----|-----------|------|------|
| 3220001801000020816200 | Aceptado | 20312528046 | Física |
| 3220002204000040970011 | Pendiente | 20312528046 | Física |
| Cualquier CBU de la lista | Rechazado | - | Física |

---

## 10. Consideraciones para Emisión Masiva

1. **Generación de PDF:** Desactivar `opc_generar_pdf: false` para acelerar el procesamiento en emisiones grandes.
2. **Códigos de barras:** Usar un único código por boleta para claridad del usuario.
3. **QR vinculado:** Usar `generar_qr_vinculado` para boletas con múltiples cuotas/pagos.
4. **Identificadores:** Usar `numero_operacion` para relacionar los pagos con los sistemas propios.
5. **Lotes:** Dividir operaciones en grupos de máximo 50 por petición.
6. **Panel de Control:** Alternativa sin integración técnica para importar archivos CSV/Excel.
7. **Recomendación QR:** El código QR de ePagos permite pagar con tarjeta de crédito/débito sin presencia física.

---

**API:** SOAP v1.2  
**Versión de API:** 2.1 (número a informar en métodos: 2.0)  
**Última actualización:** 2026-04-01  
**Fuente:** https://www.epagos.com/desarrolladores.php?secc=emision_masiva
