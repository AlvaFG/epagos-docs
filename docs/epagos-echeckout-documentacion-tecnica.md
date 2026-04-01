# ePagos — Documentación Técnica E-Checkout

**Fuente:** https://www.epagos.com/desarrolladores.php?secc=checkout

---

## 1. URLs de Entornos

| Entorno | Sandbox (pruebas) | Producción |
|---|---|---|
| **Obtener token** | `https://sandbox.epagos.com/post.php` | `https://api.epagos.com/post.php` |
| **Enviar solicitud de pago** | `https://postsandbox.epagos.com` | `https://post.epagos.com` |
| **Panel de control** | `https://portalsandbox.epagos.com` | `https://portal.epagos.com` |

---

## 2. Código Fuente de Ejemplo — SDK PHP

Repositorio oficial: https://github.com/epagos/api_php/tree/master/post

- Ejemplo de redirección: https://github.com/epagos/api_php/blob/master/post/redireccion/inicio.php
- Ejemplo con iframe: https://github.com/epagos/api_php/blob/master/post/iframe/inicio.php

---

## 3. Paso 1 — Obtención del Token

El primer paso es obtener la autorización para iniciar el pago. Se debe realizar un **POST HTTP** a la URL de obtención de token.

### 3.1. Parámetros del POST para obtener Token

| Nombre | Tipo | Descripción |
|---|---|---|
| `id_usuario` | Texto | Usuario |
| `id_organismo` | Numérico entero | Código de Organismo asignado |
| `password` | Texto | Contraseña de su usuario |
| `hash` | Texto | Hash correspondiente a su usuario |

### 3.2. Respuesta de obtención de Token (JSON)

| Nombre | Tipo | Descripción |
|---|---|---|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de respuesta |
| `id_organismo` | Numérico entero | Código de Organismo |
| `Token` | Texto | El token generado (solamente en caso de éxito) |

### 3.3. Códigos de respuesta — Obtención de Token

| id_resp | Tipo | Respuesta |
|---|---|---|
| `01001` | Correcta | Token generado |
| `01002` | Error | Error al validar el usuario o sus credenciales. |
| `01003` | Error | Error interno al generar el token. |

---

## 4. Paso 2 — Formulario POST de Checkout (Inicio de Solicitud de Pago)

Una vez obtenido el token, se realiza el POST a la URL correspondiente. Los campos marcados con `(*)` son **obligatorios**.

### 4.1. Campos base

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `version` | Texto | ✅ Obligatorio | La versión del protocolo. Valor fijo: `"1.0"` |
| `operacion` | Set | ✅ Obligatorio | Tipo de operación. Valor fijo: `"op_pago"` |
| `convenio` | Numérico | ✅ Obligatorio | El número de convenio asignado. Ej: `00001`. En algunos casos la plataforma puede inferirlo automáticamente; en ese caso enviar `null`. |

### 4.2. Credenciales

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `id_organismo` | Numérico | ✅ Obligatorio | Código del Organismo |
| `token` | Texto | ✅ Obligatorio | Token generado previamente |

### 4.3. URLs de respuesta

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `ok_url` | Texto | ✅ Obligatorio | URL de respuesta positiva. No indica que la operación esté acreditada. |
| `error_url` | Texto | ✅ Obligatorio | URL de respuesta de error |

### 4.4. Datos de la operación

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `numero_operacion` | Texto (100) | Opcional | Identificación externa. Permite relacionar una solicitud de pago con el sistema del Organismo. |
| `id_moneda_operacion` | Numérico entero | ✅ Obligatorio | Moneda. Valor fijo: `"1"` |
| `monto_operacion` | Numérico decimal | ✅ Obligatorio | Monto |
| `identificador_cliente` | Texto (6–100) | Opcional | Solo para uso de guardado de medios de pago. Ver pagos recurrentes. |
| `fecha_2do_venc` | Fecha (AAAA-MM-DD) | Opcional | La fecha del segundo vencimiento de la solicitud de pago. |
| `monto_operacion_2do_venc` | Numérico decimal | Opcional | El importe del segundo vencimiento de la operación, con dos decimales. |
| `tipo_operacion` | Numérico entero | Opcional | El tipo de deuda para agrupar la deuda. Consultar al implementador para ver los valores posibles. |

### 4.5. Opciones

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `opc_pdf` | Booleano | Opcional | Indica si se devuelve el contenido del comprobante PDF. Por defecto: `true` |
| `opc_devolver_qr` | Booleano | Opcional | Indica si se devuelve el contenido del código QR (solo para pagos en Efectivo y Homebanking). Por defecto: `true` |
| `opc_devolver_codbarras` | Booleano | Opcional | Indica si se devuelve el contenido del código de barras como imagen. Por defecto: `false` |
| `opc_comision` | Booleano | Opcional | Indica si se debe retornar el valor de la comisión de la transacción. Por defecto: `false` |
| `opc_fecha_vencimiento` | Fecha (AAAA-MM-DD) | Opcional | Fecha del primer vencimiento de la solicitud de pago. Debe ser mayor que la fecha de hoy. |
| `opc_guardado_obligatorio` | Booleano | Opcional | Indica si el usuario está obligado a guardar el medio de pago para recurrencia. |
| `opc_totem` | Booleano | Opcional | Indica si debe mostrarse teclado en pantalla, útil para interfaces táctiles (Tótem o Kiosk digital). |
| `opc_email_automatico` | Booleano | Opcional | Indica si el `email_pagador` se solicita al usuario cuando no se informa. Por defecto: `true` |
| `opc_descargar_pdf` | Booleano | Opcional | Indica si se muestra una interfaz de ePagos para la descarga de las instrucciones o boleta de pago. Por defecto: `false` |
| `opc_operaciones_lote` | Texto | Opcional | Lista de `id_transaccion` separados por comas. Si se indica, cuando la operación de E-Checkout se acredite, se traslada esa acreditación a las operaciones indicadas (sin invocar manualmente al método `pago_lote` de la API). |
| `opc_T30_monto_abierto` | Booleano | Opcional | Indica si el QR interoperable generado es de monto abierto o cerrado. Por defecto: `false` |
| `opc_T30_reutilizable` | Booleano | Opcional | Indica si el QR interoperable generado se puede pagar más de una vez. Por defecto: `false` |
| `opc_T30_requiere_orden` | Booleano | Opcional | Indica si el QR interoperable generado requiere una orden de pago para su cobro. Por defecto: `false` |
| `opc_una_cuota` | Booleano | Opcional | **Nuevo.** Fuerza a que los pagos con Tarjeta de crédito solo puedan realizarse en una cuota. Por defecto: `false` |

### 4.6. Detalle de la operación

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `detalle_operacion` | Texto (JSON codificado) | Opcional | Estructura con el detalle de cómo se compone el monto a pagar. Se debe enviar un array con al menos un elemento. Ver sección 5. |
| `detalle_operacion_visible` | Booleano | Opcional | Determina si se debe mostrar al usuario el detalle informado. Por defecto: `true` |

### 4.7. Forma de pago (opcional)

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `fp_excluidas` | Array numérico serializado o texto con números separados por comas | Opcional | Las formas de pago excluidas. |
| `tp_excluidos` | Array numérico serializado o texto con números separados por comas | Opcional | Los tipos de pago excluidos. |
| `fp_permitidas` | Array numérico serializado o texto con números separados por comas | Opcional | Las formas de pago permitidas. |
| `fp_defecto` | Numérico | Opcional | La forma de pago preferida. |

### 4.8. Identificadores adicionales (opcional)

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `identificador_externo_2` | Texto (100) | Opcional | Identificador adicional para uso del cliente. |
| `identificador_externo_3` | Texto (512) | Opcional | Identificador adicional para uso del cliente. |
| `identificador_externo_4` | Texto (65.535) | Opcional | Identificador adicional para uso del cliente. |

### 4.9. Datos del pagador (opcional)

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `nombre_pagador` | Texto | Opcional | Nombre |
| `apellido_pagador` | Texto | Opcional | Apellido |
| `fechanac_pagador` | Fecha (AAAA-MM-DD) | Opcional | Fecha de nacimiento |
| `email_pagador` | Texto | Opcional | Email |
| `tipo_doc_pagador` | Numérico | Opcional | Tipo de documento |
| `numero_doc_pagador` | Numérico entero | Opcional | Número de documento |
| `cuit_doc_pagador` | Numérico entero | Opcional | CUIT / CUIL |
| `calle_dom_pagador` | Texto | Opcional | Calle del domicilio |
| `numero_dom_pagador` | Texto | Opcional | Número de la calle del domicilio |
| `adicional_dom_pagador` | Texto | Opcional | Datos adicionales de la dirección |
| `cp_dom_pagador` | Texto | Opcional | Código postal del domicilio |
| `ciudad_dom_pagador` | Texto | Opcional | Ciudad del domicilio |
| `provincia_dom_pagador` | Numérico entero | Opcional | Provincia del domicilio |
| `país_dom_pagador` | Numérico entero | Opcional | País del domicilio |
| `codigo_telef_pagador` | Numérico entero | Opcional | Código de área del teléfono |
| `numero_telef_pagador` | Numérico entero | Opcional | Número del teléfono |
| `banco_pagador` | Numérico entero | Opcional | El código de banco del pagador |
| `cbu_pagador` | Numérico entero | Opcional | El número de CBU del banco del pagador. Usado cuando la forma de pago es DEBIN. |

---

## 5. Estructura `detalle_operacion`

El campo `detalle_operacion` es un **array de objetos JSON** con la siguiente estructura por elemento:

| Nombre | Tipo | Descripción |
|---|---|---|
| `id_item` | Numérico entero | Identificador del elemento del detalle |
| `desc_item` | Texto | Descripción del elemento del detalle |
| `monto_item` | Numérico decimal | Monto unitario del elemento del detalle |
| `cantidad_item` | Numérico entero | Cantidad del elemento del detalle |

> **⚠️ Atención:** Si se informan los detalles de la operación, la plataforma validará que la **suma de `monto_item × cantidad_item` de cada elemento coincida con el `monto_operacion`** total.

### Notas técnicas de codificación

- El parámetro `detalle_operacion` debe enviarse **codificando el JSON** con `urlencode()` de PHP (o función equivalente).
- Otra opción es codificar en **Base64** el texto JSON resultante. En este caso, el JSON debe ser un **array de objetos** con la estructura indicada.
- Los campos de tipo `float` se envían **sin separadores de miles** y con **punto (`.`) como separador decimal**. Ejemplo: `1234.12`. Si es un entero puede enviarse como `123` o `123.00`.

### Ejemplo en PHP

```php
// Opción 1: urlencode
$detalle_op = urlencode(json_encode([
    [
        'id_item'       => '0',
        'desc_item'     => 'Cuota pura del impuesto provincial',
        'monto_item'    => 200,
        'cantidad_item' => 1
    ],
    [
        'id_item'       => '1',
        'desc_item'     => 'Recargo pago fuera de término',
        'monto_item'    => 10,
        'cantidad_item' => 1
    ]
]));

// Opción 2: Base64
$detalle_op = base64_encode(json_encode([ [...], ..., [...] ]));
```

---

## 6. Ejemplo de Formulario HTML — Inicio de Solicitud de Pago

```html
<form name='pago' method='post' action='<url_post>'>
    <input type='hidden' name='version'                   value='2.0' />
    <input type='hidden' name='operacion'                 value='op_pago' />
    <input type='hidden' name='id_organismo'              value='<id_organismo>' />
    <input type='hidden' name='convenio'                  value='<nro_convenio>' />
    <input type='hidden' name='token'                     value='<token_generado>' />
    <input type='hidden' name='numero_operacion'          value='<nro_operacion>' />
    <input type='hidden' name='id_moneda_operacion'       value='1' />
    <input type='hidden' name='monto_operacion'           value='<monto>' />
    <input type='hidden' name='detalle_operacion'         value='<detalle_encodeado>' />
    <input type='hidden' name='detalle_operacion_visible' value='1' />
    <input type='hidden' name='ok_url'                    value='<su_url_ok>' />
    <input type='hidden' name='error_url'                 value='<su_url_error>' />
    <br/>
    <input type='submit' value='Enviar pago' />
</form>
```

El envío redirige al usuario a ePagos para que complete el pago. A la vuelta, recibirá la respuesta en la `ok_url` o `error_url` según el resultado.

---

## 7. Respuesta POST a `ok_url` y `error_url`

Las respuestas **siempre** se realizan mediante un **POST HTTP** a las URLs indicadas. Los campos devueltos varían según el medio de pago elegido.

### 7.1. Respuesta — Tarjetas de crédito / débito (a `ok_url`)

Datos que ingresa el usuario en el formulario de pago con tarjeta:
- Cantidad de cuotas (solo crédito)
- Número de tarjeta
- Código de seguridad
- Fecha de vencimiento (año y mes)
- Nombre del titular (como aparece en la tarjeta)
- Tipo de documento
- Número de documento
- E-mail
- Fecha de nacimiento
- Número de puerta
- Datos del domicilio (solo si la operación es de alto riesgo)

**Campos devueltos en el POST:**

| Nombre | Tipo | Descripción |
|---|---|---|
| `id_resp` | Numérico entero | Código de Respuesta |
| `respuesta` | Texto | Texto de la respuesta |
| `convenio` | Numérico entero | Número de convenio asignado a la operación |
| `fp` | Texto (JSON) | Vector con los datos: `nombre_fp` (Texto), `tarjeta_fp` (Texto), `importe_fp` (Numérico decimal), `importe_original_fp` (Numérico decimal), `respuesta_entidad_cobro` (Texto) |
| `id_organismo` | Numérico | Código del Organismo |
| `id_transaccion` | Numérico entero | Identificador de la transacción |
| `id_fp` | Numérico entero | Identificador de la forma de pago elegida |
| `numero_operacion` | Texto | Dato enviado al iniciar la operación |
| `identificador_2` | Texto | Dato enviado al iniciar la operación en el atributo `identificador_externo_2` |
| `identificador_3` | Texto | Dato enviado al iniciar la operación en el atributo `identificador_externo_3` |
| `identificador_4` | Texto | Dato enviado al iniciar la operación en el atributo `identificador_externo_4` |
| `token` | Texto | El token con el que inició la solicitud |

### 7.2. Respuesta — Medios de pago en efectivo (a `ok_url`)

El usuario ingresa solo su **E-mail**. El sistema genera código de barras y código de pago sin factura.

**Campos devueltos en el POST:**

| Nombre | Tipo | Descripción |
|---|---|---|
| `id_resp` | Numérico entero | Código de Respuesta |
| `respuesta` | Texto | Texto de la respuesta |
| `convenio` | Numérico entero | Número de convenio asignado a la operación |
| `fp` | Texto (JSON) | Vector con los datos: `nombre_fp` (Texto), `importe_fp` (Numérico decimal), `respuesta_entidad_cobro` (Texto), `fechavenc_fp` (Fecha AAAA-MM-DD) |
| `id_organismo` | Numérico | Código del Organismo |
| `id_transaccion` | Numérico entero | Identificador de la transacción |
| `id_fp` | Numérico entero | Identificador de la forma de pago elegida |
| `numero_operacion` | Texto | Dato enviado al iniciar la operación en el atributo `numero_operacion` |
| `identificador_2` | Texto | Dato enviado al iniciar la operación en el atributo `identificador_externo_2` |
| `identificador_3` | Texto | Dato enviado al iniciar la operación en el atributo `identificador_externo_3` |
| `token` | Texto | El token con el que inició la solicitud |
| `pdf` | Texto | Boleta de pago (PDF). Contenido del PDF codificado en **base64** |
| `codigo_barras_fp` | Numérico entero | Código de barras de la forma de pago |
| `codigo_pago_fp` | Texto | Código de pago de la forma de pago (código de pago sin factura) |
| `codigo_qr` | Texto | Código QR. Contenido de la imagen del QR codificado en **base64** |
| `barras_adicionales` | Vector | Vector con la información de las barras adicionales de la operación, si existen |

### 7.3. Respuesta — Homebanking / Transferencias / DEBIN (a `ok_url`)

El usuario ingresa: **CUIT/CUIL** (de la cuenta bancaria), **CBU/CVU o alias** (solo para DEBIN), y **E-mail**.

**Campos devueltos en el POST:**

| Nombre | Tipo | Descripción |
|---|---|---|
| `id_resp` | Numérico entero | Código de Respuesta |
| `respuesta` | Texto | Texto de la respuesta |
| `convenio` | Numérico entero | Número de convenio asignado a la operación |
| `fp` | Texto (JSON) | Vector con los datos: `nombre_fp` (Texto), `importe_fp` (Numérico decimal), `respuesta_entidad_cobro` (Texto) |
| `id_organismo` | Numérico | Código del Organismo |
| `id_transaccion` | Numérico entero | Identificador de la transacción |
| `id_fp` | Numérico entero | Identificador de la forma de pago elegida |
| `numero_operacion` | Texto | Dato enviado al iniciar la operación |
| `identificador_2` | Texto | Dato enviado al iniciar la operación en `identificador_externo_2` |
| `identificador_3` | Texto | Dato enviado al iniciar la operación en `identificador_externo_3` |
| `token` | Texto | El token con el que inició la solicitud |
| `pdf` | Texto | Boleta de pago (PDF). Contenido del PDF codificado en **base64** |
| `cuit_cuenta` | Numérico entero | El CUIT a donde se publicará (Homebanking) o desde donde deberá transferir (Transferencia Bancaria) |
| `cbu_cuenta` | Texto | Solo en caso de DEBIN: el CBU/CVU o alias de la cuenta a donde se publicará el DEBIN |

### 7.4. Respuesta — Error (a `error_url`)

**Campos devueltos en el POST:**

| Nombre | Tipo | Descripción |
|---|---|---|
| `id_resp` | Numérico entero | Código de Respuesta |
| `respuesta` | Texto | Texto de la respuesta |
| `convenio` | Numérico entero | Número de convenio asignado a la operación |
| `fp` | Texto (JSON) | Vector con los datos: `nombre_fp` (Texto), `importe_fp` (Numérico decimal), `respuesta_entidad_cobro` (Texto) |
| `id_organismo` | Numérico | Código del Organismo |
| `id_transaccion` | Numérico entero | Identificador de la transacción |
| `numero_operacion` | Texto | Dato enviado al iniciar la operación en el atributo `numero_operacion` |
| `identificador_2` | Texto | Dato enviado al iniciar la operación en el atributo `identificador_externo_2` |
| `identificador_3` | Texto | Dato enviado al iniciar la operación en el atributo `identificador_externo_3` |
| `token` | Texto | El token con el que inició la solicitud |

---

## 8. Códigos de Respuesta — Operación de Pago

Aplican tanto a la `ok_url` como a la `error_url`.

| id_resp | Tipo | Respuesta |
|---|---|---|
| `02001` | Correcta | Pago acreditado |
| `02002` | Correcta | Pago pendiente |
| `02003` | Error | Error al validar el token |
| `02004` | Error | Pago cancelado / rechazado |
| `02005` | Error | Error interno al intentar procesar el pago |
| `02006` | Error | Error al validar el parámetro: `[_parametro_]` |
| `02007` | Error | El usuario canceló el pago |
| `02008` | Error | No coinciden los montos y los detalles |
| `02009` | Error | La forma de pago `[_parametro_]` no se encuentra disponible |
| `02010` | Error | Error del proveedor de servicios online |

---

## 9. Notas Técnicas Adicionales

- La integración requiere **credenciales de firma** para el Organismo antes de poder utilizar E-Checkout.
- El flujo es: **Obtener Token → POST del formulario → ePagos procesa el pago → POST de retorno a `ok_url` o `error_url`**.
- Las respuestas de retorno **siempre se realizan mediante POST HTTP**, nunca GET.
- La `ok_url` indica respuesta positiva del flujo, **pero no garantiza que la operación esté acreditada** (puede estar pendiente, código `02002`).
- El campo `fp` en las respuestas es un **JSON embebido en el POST** que contiene los datos de la forma de pago seleccionada. Su estructura varía según el medio de pago.
- Los valores `fp_excluidas`, `tp_excluidos` y `fp_permitidas` pueden enviarse como array PHP serializado con `serialize()` o como texto con los números separados por comas.
- **Nota legal:** E-Pagos S.A. (CUIT: 30-71508429-1) ofrece servicios de pago y no está autorizado por el Banco Central a operar como entidad financiera. Los fondos acreditados en cuentas de pago no constituyen depósitos en entidades financieras.
