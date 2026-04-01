# **DOCUMENTACIÓN TÉCNICA - API SOLICITUD DE PAGO**
## **ePagos v2.1**

---

## **1. DESCRIPCIÓN GENERAL**

Genera una nueva operación en el sistema. Este método solo puede utilizarse para generar operaciones que **no sean de tarjeta de crédito ni débito**.

**Nota de seguridad:** Por motivos de seguridad, este tipo de integración no admite el pago con tarjetas de crédito y/o débito directamente enviándolos a la API. Si desea habilitar al contribuyente a pagar por estos medios, debe usar el QR generado.

---

## **2. PARÁMETROS RECIBIDOS**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| version | Texto | ✅ Sí | Versión, valor fijo: "2.0" |
| tipo_operacion | Set | ✅ Sí | Tipo de la operación. Valor fijo: "op_pago" |
| credenciales | Estructura DatosCredenciales | ✅ Sí | Los datos de las credenciales |
| operacion | Estructura DatosOperacionPago | ✅ Sí | Los datos de la operación a realizar |
| fp | Array de FormaPago | ✅ Sí | Los datos de las formas de pago |
| convenio | Numérico entero | ✅ Sí | El número de convenio asignado por ePagos |

---

## **3. ESTRUCTURAS DE REQUEST**

### **3.1 DatosCredenciales**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| id_organismo | Numérico entero | ✅ Sí | Código de organismo |
| token | Texto | ✅ Sí | El token generado mediante obtener_token |

---

### **3.2 DatosOperacionPago**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| numero_operacion | Texto (100) | ❌ No | Código externo del cliente. Permite relacionar un pago con sus transacciones |
| identificador_externo_2 | Texto (100) | ❌ No | Identificador adicional para uso del cliente |
| identificador_externo_3 | Texto (512) | ❌ No | Identificador adicional para uso del cliente |
| identificador_externo_4 | Texto (65.535) | ❌ No | Identificador adicional para uso del cliente |
| identificador_cliente | Texto | ❌ No | Identificador para usar en métodos de recurrencia |
| id_moneda_operacion | Numérico entero | ✅ Sí | Código de moneda, valor fijo: 1 (Peso Argentino) |
| monto_operacion | Numérico decimal | ✅ Sí | El importe de la operación con dos decimales (ej: 1234.56) |
| opc_pdf | Booleano | ❌ No | Determina si se incluye el PDF en la respuesta. Por defecto: true |
| opc_fecha_vencimiento | Fecha (AAAA-MM-DD) | ❌ No | Fecha de vencimiento de la boleta para pagos presenciales |
| opc_devolver_qr | Booleano | ❌ No | Determina si se devuelve la imagen del QR. Por defecto: false |
| opc_devolver_codbarras | Booleano | ❌ No | Determina si se devuelve la imagen del código de barras. Por defecto: false |
| opc_generar_pdf | Booleano | ❌ No | Determina si se genera la boleta en PDF. No generarla acelera el procesamiento. Por defecto: true |
| detalle_operacion | Estructura DetalleOperacion | ❌ No | Detalles o ítems que componen el importe |
| pagador | Array de Pagador | ✅ Sí | Los datos del pagador de la operación |
| fecha_2do_venc | Fecha (AAAA-MM-DD) | ❌ No | Fecha del segundo vencimiento de la boleta |
| monto_operacion_2do_venc | Numérico decimal | ❌ No | Importe del segundo vencimiento con dos decimales |
| tipo_operacion | Numérico entero | ❌ No | Tipo de deuda para agrupar. Consultar implementador |
| codigo_publicacion | Numérico entero (8 dígitos) | ❌ No | Código de publicación de homebanking. Si no se usa, se utiliza el CUIT del Pagador |
| url_boleta | Texto (256) | ❌ No | URL donde se aloja la boleta para enviar al usuario |
| opc_T30_cerrado | Booleano | ❌ No | **[Nuevo]** Determina si el QR interoperable es cerrado o abierto. Por defecto: true |
| opc_T30_reutilizable | Booleano | ❌ No | **[Nuevo]** Determina si el QR interoperable se puede pagar más de una vez. Por defecto: false |
| opc_T30_requiere_orden | Booleano | ❌ No | **[Nuevo]** Determina si el QR solo se paga mediante orden generada previamente. Solo en QRs cerrados. Por defecto: false |

---

### **3.3 DetalleOperacion**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| id_item | Numérico entero | ✅ Sí | Código o número de ítem |
| desc_item | Texto | ✅ Sí | Texto a mostrar al usuario sobre el ítem |
| monto_item | Numérico decimal | ✅ Sí | Importe del ítem (suma × cantidad = monto operación) |
| cantidad_item | Numérico entero | ✅ Sí | Cantidad de estos ítems |

---

### **3.4 Pagador**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| nombre_pagador | Texto | ❌ No | El nombre del pagador |
| apellido_pagador | Texto | ❌ No | El apellido del pagador |
| fechanac_pagador | Texto (AAAA-MM-DD) | ❌ No | La fecha de nacimiento del pagador |
| email_pagador | Texto | ✅ Sí | La dirección de email del pagador |
| identificacion_pagador | Estructura PagadorIdentificacion | ✅ Sí | La identificación del pagador |
| domicilio_pagador | Estructura PagadorDomicilio | ❌ No | La información del domicilio del pagador |
| telefono_pagador | Estructura PagadorTelefono | ❌ No | La información del teléfono del pagador |
| cbu_pagador | Numérico entero | ❌ No | Número de CBU del banco del pagador (usado cuando la forma de pago es DEBIN) |

---

### **3.5 PagadorIdentificacion**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| tipo_doc_pagador | Numérico entero | ✅ Sí | El tipo de documento del pagador. [Ver tabla documentos](#tabla-tipos-documentos) |
| numero_doc_pagador | Numérico entero | ✅ Sí | El número de documento del pagador |
| cuit_doc_pagador | Numérico entero | ❌ No | El número de CUIT del pagador |

---

### **3.6 PagadorDomicilio**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| calle_dom_pagador | Texto | ✅ Sí | El nombre de la calle del pagador |
| numero_dom_pagador | Texto | ✅ Sí | El número de la calle del pagador |
| adicional_dom_pagador | Texto | ❌ No | El adicional de la calle del pagador (piso, depto, etc) |
| cp_dom_pagador | Texto | ✅ Sí | El código postal del domicilio |
| ciudad_dom_pagador | Texto | ✅ Sí | La ciudad del domicilio |
| provincia_dom_pagador | Numérico entero | ✅ Sí | El código de la provincia. [Ver tabla provincias](#tabla-provincias) |
| país_dom_pagador | Numérico entero | ✅ Sí | El código de país. Valor fijo: 13 (Argentina) |

---

### **3.7 PagadorTelefono**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| codigo_telef_pagador | Numérico entero | ✅ Sí | El código de área del teléfono del pagador |
| numero_telef_pagador | Numérico entero | ✅ Sí | El número de teléfono del pagador |

---

### **3.8 FormaPago**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| id_fp | Numérico entero | ✅ Sí | Código de la forma de pago |
| monto_fp | Numérico decimal | ✅ Sí | El importe de la forma de pago |
| tarjeta | Estructura FormaPagoTarjeta | ❌ Condicional | **Obligatoria solo si la forma de pago es tarjeta de crédito o débito** |

---

### **3.9 FormaPagoTarjeta**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| numero_tarjeta_fp | Numérico entero | ✅ Sí | El número de tarjeta de crédito |
| banco_tarjeta_fp | Texto | ✅ Sí | El nombre del banco emisor |
| vencimiento_tarjeta_fp | Estructura FormaPagoTarjetaVencimiento | ✅ Sí | Fecha de vencimiento |
| codigo_seg_tarjeta_fp | Texto | ✅ Sí | Código de seguridad (CVV) de la tarjeta |
| cuotas_tarjeta_fp | Numérico entero | ✅ Sí | La cantidad de cuotas a realizar |
| titular_tarjeta_fp | Texto | ✅ Sí | Nombre y apellido como aparece en la tarjeta |
| identificación_tarjeta_fp | Estructura FormaPagoTarjetaIdentificacion | ✅ Sí | La identificación del titular |
| fechanac_tarjeta_fp | Fecha (AAAA-MM-DD) | ✅ Sí | Fecha de nacimiento del titular |
| direccion_tarjeta_fp | Estructura FormaPagoTarjetaDireccion | ✅ Sí | Dirección del titular (donde se envía resumen) |

---

### **3.10 FormaPagoTarjetaVencimiento**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| mes_vencimiento_tarjeta_fp | Numérico entero | ✅ Sí | Mes de vencimiento (MM) - ej: 12 |
| anio_vencimiento_tarjeta_fp | Numérico entero | ✅ Sí | Año de vencimiento (AAAA) - ej: 2025 |

---

### **3.11 FormaPagoTarjetaIdentificacion**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| tipo_identificacion_tarjeta_fp | Numérico entero | ✅ Sí | El tipo de identificación del titular. [Ver tabla documentos](#tabla-tipos-documentos) |
| numero_identificacion_tarjeta_fp | Numérico entero | ✅ Sí | El número de identificación del titular |

---

### **3.12 FormaPagoTarjetaDireccion**

| Nombre | Tipo | Obligatorio | Descripción |
|--------|------|------------|-------------|
| calle_direccion_tarjeta_fp | Texto | ✅ Sí | La calle de la dirección del titular |
| numero_direccion_tarjeta_fp | Texto | ✅ Sí | El número de la dirección |

---

## **4. NOTAS IMPORTANTES SOBRE FORMATOS**

- Los campos de tipo **float/decimal** se envían **sin separadores de miles** y con el **punto (.)** como separador decimal
- Ejemplo correcto: `1234.56`
- Si el número es entero, puede enviarse sin `.00`: `123` o `123.00`
- Las fechas siempre en formato: `AAAA-MM-DD` (ej: 2025-12-31)

---

## **5. RESPUESTA**

### **5.1 Parámetros de Respuesta**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| id_resp | Numérico entero | El código de respuesta (ver tabla de respuestas) |
| respuesta | Texto | La descripción de la respuesta |
| token | Texto | El token único generado para la solicitud |
| id_transaccion | Numérico entero | Código único de operación asignado por ePagos |
| id_organismo | Numérico entero | Su número de organismo proporcionado por ePagos |
| convenio | Numérico entero | El número de convenio proporcionado por ePagos |
| numero_operacion | Numérico entero | El número interno de operación informado al iniciar |
| fp | Array de RespuestaFormaPago | Array con información de las formas de pago procesadas |

---

### **5.2 RespuestaFormaPago**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| codigo_pago_fp | Numérico entero | Para pago en efectivo o Homebanking, código corto que identifica el pago |
| codigo_barras_fp | Texto | El código de barras completo que identifica el pago |
| fechavenc_fp | Fecha (AAAA-MM-DD) | La fecha de vencimiento para el pago de la operación |
| importe_fp | Numérico decimal | El importe de la operación generada |
| pdf | Base64 | Contenido del comprobante en PDF (si opc_pdf = true). Codificado en base64 |
| codigo_barras_imagen | Base64 | Imagen del código de barras en PNG (solo si opc_devolver_codbarras = true) |
| qr_imagen | Base64 | Imagen del QR en PNG (solo si opc_devolver_qr = true) |
| qr_imagen_T30 | Base64 | QR para transferencias 3.0 en PNG (solo si opc_devolver_qr = true) |
| qr_texto_T30 | Texto | Texto contenido del QR para transferencias 3.0 |
| respuesta_entidad_cobro | Texto | Detalle de la respuesta del medio de pago elegido |
| codigo_pmc | Texto | Código de publicación de la deuda en Pago Mis Cuentas |
| codigo_link | Texto | Código de publicación de la deuda en Red Link |
| url_qr | Texto | URL a usar para pago en línea (tarjeta de crédito y/o débito) |
| cuit_cuenta | Numérico entero | Número de CUIT de la cuenta |
| cbu_cuenta | Texto | Número de CBU/CVU o alias de la cuenta de destino (DEBIN) |
| barras_adicionales | Array de BarrasAdicionales | **[Nuevo]** Array con códigos de barras adicionales |

---

### **5.3 BarrasAdicionales**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| codigo_barra | Texto | El código de barra adicional de la operación |
| codigo_pago | Texto | El código de pago sin factura asociado a la barra |
| fechavenc | Fecha (AAAA-MM-DD) | Fecha de vencimiento de la barra adicional |
| importe | Numérico decimal | El importe de la barra adicional |

---

## **6. CÓDIGOS DE RESPUESTA Y ERRORES**

| ID Respuesta | Tipo | Descripción |
|-------------|------|-------------|
| **02001** | ✅ Correcta | Pago acreditado |
| **02002** | ✅ Correcta | Pago pendiente |
| **02003** | ❌ Error | Error al validar el token |
| **02004** | ❌ Error | Pago cancelado / rechazado |
| **02005** | ❌ Error | Error interno al intentar procesar el pago |
| **02006** | ❌ Error | Error al validar el parámetro: [_parametro_] |
| **02007** | ❌ Error | El usuario canceló el pago |
| **02008** | ❌ Error | No coinciden los montos y los detalles |
| **02009** | ❌ Error | La forma de pago [_parametro_] no se encuentra disponible |
| **02010** | ❌ Error | Versión inválida del protocolo |

---

## **7. FLUJO DE INTEGRACIÓN**

1. **Obtener Token**: Realiza una petición SOAP al método `obtener_token` con tus credenciales
2. **Obtener Entidades de Pago** (Opcional): Llama a `obtener_entidades_pago` para obtener medios de pago disponibles
3. **Crear Solicitud de Pago**: Llama a `solicitud_pago` con el token y forma de pago seleccionada
4. **Procesar Respuesta**: Captura el código de barras, QR y PDF de la boleta generada

---

## **8. CONSIDERACIONES DE SEGURIDAD**

- ⚠️ **No envíes tarjetas de crédito/débito directamente a la API**
- ⚠️ **Siempre valida el token antes de procesar**
- ⚠️ **Usa HTTPS para todas las comunicaciones**
- ⚠️ **Protege tus credenciales de acceso**
- ✅ **Usa el QR generado para pagos con tarjeta**

---

## **9. TABLA DE REFERENCIA - TIPOS DE DOCUMENTOS**

Para `tipo_doc_pagador` y `tipo_identificacion_tarjeta_fp`:

| Código | Documento |
|--------|-----------|
| 1 | DNI (Documento Nacional de Identidad) |
| 2 | Pasaporte |
| 3 | Cédula Extranjera |
| 4 | Libreta Cívica |
| 5 | Libreta de Enrolamiento |

---

## **10. TABLA DE REFERENCIA - PROVINCIAS DE ARGENTINA**

| Código | Provincia |
|--------|-----------|
| 1 | Buenos Aires |
| 2 | Catamarca |
| 3 | Chaco |
| 4 | Chubut |
| 5 | Córdoba |
| 6 | Corrientes |
| 7 | Entre Ríos |
| 8 | Formosa |
| 9 | Jujuy |
| 10 | La Pampa |
| 11 | La Rioja |
| 12 | Mendoza |
| 13 | Misiones |
| 14 | Neuquén |
| 15 | Río Negro |
| 16 | Salta |
| 17 | San Juan |
| 18 | San Luis |
| 19 | Santa Cruz |
| 20 | Santa Fe |
| 21 | Santiago del Estero |
| 22 | Tierra del Fuego |
| 23 | Tucumán |
| 24 | CABA (Ciudad Autónoma de Buenos Aires) |

---

## **FUENTE**
Documentación extraída de: https://www.epagos.com/templates/desarrolladores/referencia.php?met=solicitud_pago
Versión API: v2.1
Fecha de extracción: 2026-04-01
