# Documentación Técnica de Pagos Recurrentes - ePagos API v2.1

## Resumen Ejecutivo

ePagos ofrece una solución completa para pagos recurrentes que permite a organizaciones cobrar de forma automática sobre tarjetas de crédito/débito o débitos en cuenta bancaria previamente guardados por los clientes. La solución incluye métodos para:

- Autorización mediante tokens
- Registro y gestión de tarjetas y cuentas bancarias
- Ejecución de cargos recurrentes puntuales
- Planificación de suscripciones con múltiples cobros
- Consulta y listado de métodos de pago guardados

---

## 1. Flujo General de Pagos Recurrentes

```
┌─────────────────────────────────────────────────────────────┐
│                     FLUJO GENERAL                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Obtener Token (obtener_token)                           │
│     ├─ Credenciales: id_organismo, id_usuario, password    │
│     └─ Respuesta: Token de autorización                    │
│                                                              │
│  2. Registro de Medio de Pago (primera solicitud)          │
│     ├─ solicitud_pago (con identificador_cliente)          │
│     │  └─ Usuario guarda tarjeta o cuenta                  │
│     ├─ Tarjeta de crédito/débito O                         │
│     └─ Débito directo (adhesión de cuenta)                 │
│                                                              │
│  3. Gestionar Medios de Pago Guardados                     │
│     ├─ obtener_tarjetas_cliente (listar tarjetas)          │
│     ├─ obtener_cuentas_cliente (listar cuentas)            │
│     └─ registrar_cuentas_cliente (cargar cuentas)          │
│                                                              │
│  4. Ejecutar Cargos Recurrentes                            │
│     ├─ solicitud_pago_recurrente (cargo puntual)           │
│     └─ solicitud_pago_recurrente_suscripcion (plan)        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Método: Obtener Token

**Descripción:** Valida las credenciales y devuelve el token de autorización requerido para invocar todos los métodos de la API.

### Parámetros de Entrada

| Nombre | Tipo | Descripción | Requerido |
|--------|------|-------------|-----------|
| version | Texto | Versión del protocolo | ✓ Sí |
| id_organismo | Numérico entero | Código de organismo | ✓ Sí |
| id_usuario | Numérico entero | Código de usuario | ✓ Sí |
| password | Texto | Password asignado | ✓ Sí |
| hash | Texto | Hash asignado | ✓ Sí |

**Valores fijos:** version = "2.0"

### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| id_resp | Numérico entero | Código de respuesta |
| respuesta | Texto | Descripción de la respuesta |
| token | Texto | Token único para autorización |

### Códigos de Respuesta

| Código | Tipo | Descripción |
|--------|------|-------------|
| 01001 | Correcta | Token generado exitosamente |
| 01002 | Error | Error al validar usuario o credenciales |
| 01003 | Error | Error interno al generar token |
| 01004 | Error | Versión inválida del protocolo |

---

## 3. Método: Solicitud de Pago (Registro Inicial)

**Descripción:** Crea una operación de pago que permite al usuario registrar una tarjeta o cuenta bancaria para pagos recurrentes. Es el primer paso obligatorio para habilitar recurrencia.

### Parámetros Relevantes para Recurrencia

| Nombre | Tipo | Descripción | Requerido |
|--------|------|-------------|-----------|
| version | Texto | Versión del protocolo | ✓ Sí |
| tipo_operacion | Set | Tipo de operación. Valor: "op_pago" | ✓ Sí |
| id_organismo | Numérico entero | Código de organismo | ✓ Sí |
| token | Texto | Token de autorización | ✓ Sí |
| identificador_cliente | Texto | ID único del cliente (mín. 6 caracteres) | ✓ Recurrencia |
| email_pagador | Texto | Email del pagador | ✓ Recurrencia |
| monto_operacion | Numérico decimal | Importe a cobrar | ✓ Sí |
| opc_guardado_obligatorio | Booleano | Fuerza guardar el medio de pago | - |
| fp_permitidas | Set | Formas de pago permitidas (42=débito/tarjeta) | - |
| tp_excluidos | Texto | Tipos de operación excluidos ("1,3,4,5,6,7") | - |

### Estructura: DatosOperacionPago

| Campo | Tipo | Descripción |
|-------|------|-------------|
| numero_operacion | Texto (100) | ID externo de la operación |
| identificador_externo_2 | Texto (100) | Identificador adicional |
| identificador_externo_3 | Texto (512) | Identificador adicional |
| identificador_externo_4 | Texto (65535) | Identificador adicional |
| identificador_cliente | Texto | ID único del cliente para recurrencia |
| id_moneda_operacion | Numérico entero | Código de moneda (fijo "1" = ARS) |
| monto_operacion | Numérico decimal | Importe con dos decimales |
| opc_fecha_vencimiento | Fecha (AAAA-MM-DD) | Fecha de vencimiento de la boleta |
| opc_devolver_qr | Booleano | Si se devuelve imagen QR (default: false) |
| opc_devolver_codbarras | Booleano | Si se devuelve imagen código de barras |
| opc_generar_pdf | Booleano | Si se genera boleta PDF (default: true) |

### Estructura: Pagador

| Campo | Tipo | Descripción | Recurrencia |
|-------|------|-------------|-------------|
| nombre_pagador | Texto | Nombre del pagador | Opcional |
| apellido_pagador | Texto | Apellido del pagador | Opcional |
| email_pagador | Texto | Email (vincula con billetera) | **Obligatorio** |
| tipo_doc_pagador | Numérico | Tipo de documento | Opcional |
| numero_doc_pagador | Numérico | Número de documento | Opcional |
| cuit_doc_pagador | Numérico | CUIT | Opcional |
| cbu_pagador | Numérico | CBU del banco del pagador (para DEBIN) | Condicional |

### Respuesta solicitud_pago

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| id_resp | Numérico entero | Código de respuesta |
| respuesta | Texto | Descripción de respuesta |
| token | Texto | Token de la solicitud |
| id_transaccion | Numérico entero | ID único de operación en ePagos |
| id_organismo | Numérico entero | Código de organismo |
| convenio | Numérico entero | Número de convenio |
| numero_operacion | Numérico entero | Número interno de operación |
| fp | Array | Array de RespuestaFormaPago |

### Estructura: RespuestaFormaPago

| Campo | Tipo | Descripción |
|-------|------|-------------|
| codigo_pago_fp | Numérico entero | Código de pago corto |
| codigo_barras_fp | Texto | Código de barras completo |
| fechavenc_fp | Fecha | Fecha de vencimiento |
| importe_fp | Numérico decimal | Importe de la operación |
| pdf | Base64 | Comprobante en PDF |
| qr_imagen | Base64 | Imagen QR en PNG (si opc_devolver_qr=true) |
| url_qr | Texto | URL para pago en línea |
| respuesta_entidad_cobro | Texto | Respuesta del medio de pago |

### Códigos de Respuesta solicitud_pago

| Código | Tipo | Descripción |
|--------|------|-------------|
| 02001 | Correcta | Pago acreditado |
| 02002 | Correcta | Pago pendiente |
| 02003 | Error | Error al validar token |
| 02004 | Error | Pago cancelado / rechazado |
| 02005 | Error | Error interno al procesar el pago |
| 02006 | Error | Error al validar parámetro |
| 02007 | Error | El usuario canceló el pago |
| 02008 | Error | No coinciden montos y detalles |
| 02009 | Error | Forma de pago no disponible |
| 02010 | Error | Versión inválida del protocolo |

---

## 4. Método: solicitud_pago_recurrente

**Descripción:** Ejecuta un cobro puntual sobre una tarjeta o cuenta previamente guardada por un cliente.

> ⚠️ **Este método no está disponible para todos los organismos. Consulte a su ejecutivo comercial.**

### Parámetros de Entrada

| Nombre | Tipo | Descripción | Requerido |
|--------|------|-------------|-----------|
| version | Texto | Versión del protocolo (fijo "2.0") | ✓ Sí |
| tipo_operacion | Set | Valor fijo "op_pago_recurrente" | ✓ Sí |
| id_organismo | Numérico entero | Código de organismo | ✓ Sí |
| token | Texto | Token de autorización | ✓ Sí |
| operacion | Estructura | DatosOperacionPago | ✓ Sí |
| convenio | Numérico entero | Número de convenio | ✓ Sí |
| medio | Set | Forma de pago para los cobros | ✓ Sí |
| cliente | Estructura | SuscripcionCliente | ✓ Sí |
| fecha_debito | Fecha (AAAA-MM-DD) | Fecha de débito directo (solo Débito Directo, mín. 3 días hábiles) | - |

### Estructura: Credenciales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id_organismo | Numérico entero | Código de organismo |
| token | Texto | Token devuelto por obtener_token |

### Parámetro: Medio de Pago

| Valor | Descripción |
|-------|-------------|
| op_pago_recurrente_medio_tarjeta | Tarjeta de crédito/débito |
| op_pago_recurrente_medio_debin | Débito Inmediato (DEBIN) |
| op_pago_recurrente_medio_debito_directo | Débito Directo bancario |
| op_pago_recurrente_medio_debito_inmediato | Débito Inmediato alternativo |

### Estructura: SuscripcionCliente

| Campo | Tipo | Descripción | Condición |
|-------|------|-------------|-----------|
| identificador_cliente | Texto | ID del cliente asignado al registrarse | ✓ Siempre |
| identificador_tarjeta | Texto | ID de tarjeta (obtener_tarjetas_cliente) | Solo si medio = tarjeta |
| identificador_cuenta | Texto | ID de cuenta (obtener_cuentas_cliente) | Solo si medio = banco |

### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| id_resp | Numérico entero | Código de respuesta |
| respuesta | Texto | Descripción de respuesta |
| token | Texto | Token utilizado |
| id_transaccion | Numérico entero | ID único de operación |
| id_organismo | Numérico entero | Código de organismo |
| convenio | Numérico entero | Número de convenio |
| numero_operacion | Numérico entero | Número interno de operación |
| fp | Array | Array de RespuestaFormaPago |

### Códigos de Respuesta

| Código | Tipo | Descripción |
|--------|------|-------------|
| 09001 | Correcta | Pago acreditado |
| 09002 | Correcta | Pago pendiente |
| 09003 | Correcta | Pago rechazado / cancelado |
| 09004 | Error | Error al validar token |
| 09005 | Error | Error interno en procesamiento |
| 09006 | Error | Error al validar parámetro |
| 09007 | Error | No coinciden montos y detalles |
| 09008 | Error | Versión inválida del protocolo |
| 09009 | Error | La fecha del débito es incorrecta |
| 18012 | Error | Fecha de vencimiento obligatoria para débitos directos recurrentes |
| 18013 | Error | El campo "fecha_debito" debe estar vacío cuando se usa "opc_fecha_vencimiento" |
| 15002 | Error | Método no autorizado para el organismo |

---

## 5. Método: solicitud_pago_recurrente_suscripcion

**Descripción:** Ejecuta cobros periódicos con planificación definida (semanal, mensual, anual o personalizada) sobre un medio de pago guardado.

> ⚠️ **Este método no está disponible para todos los organismos. Consulte a su ejecutivo comercial.**

### Parámetros de Entrada

| Nombre | Tipo | Descripción | Requerido |
|--------|------|-------------|-----------|
| version | Texto | Versión del protocolo (fijo "2.0") | ✓ Sí |
| tipo_operacion | Set | Valor fijo "op_pago_recurrente" | ✓ Sí |
| id_organismo | Numérico entero | Código de organismo | ✓ Sí |
| token | Texto | Token de autorización | ✓ Sí |
| operacion | Estructura | DatosOperacionPago | ✓ Sí |
| suscripcion | Array | Array de DatosSuscripcion (mín. 1 elemento) | ✓ Sí |
| suscripcion_modalidad | Set | Modalidad de cobro: P, M, S, A | ✓ Sí |
| descripcion | Texto | Nombre de la suscripción | ✓ Sí |
| convenio | Numérico entero | Número de convenio | ✓ Sí |
| medio | Set | Forma de pago | ✓ Sí |
| clientes | Array | Array de SuscripcionCliente | ✓ Sí |

### Estructura: DatosSuscripcion

| Campo | Tipo | Descripción | Ejemplo |
|-------|------|-------------|---------|
| fecha | Fecha (AAAA-MM-DD) | Fecha del cobro (mayor a la actual) | 2026-05-01 |
| monto | Numérico decimal | Monto diferencial (si no se define, usa el de operacion) | 1500.50 |

### Modalidades de Suscripción

| Código | Descripción |
|--------|-------------|
| P | Personalizada (fechas específicas definidas manualmente) |
| M | Mensual |
| S | Semanal |
| A | Anual |

### Estructura: SuscripcionCliente

| Campo | Tipo | Descripción | Condición |
|-------|------|-------------|-----------|
| identificador_cliente | Texto | ID del cliente usado al registrar tarjetas | ✓ Siempre |
| identificador_tarjeta | Texto | ID de tarjeta de crédito/débito | Solo si medio = tarjeta |
| identificador_cuenta | Texto | ID de cuenta bancaria | Solo si medio = banco |

### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| id_resp | Numérico entero | Código de respuesta |
| respuesta | Texto | Descripción de respuesta |
| token | Texto | Token utilizado |
| id_organismo | Numérico entero | Código de organismo |

### Códigos de Respuesta

| Código | Tipo | Descripción |
|--------|------|-------------|
| 11001 | Correcta | Pago recurrente con suscripción planificado |
| 11002 | Error | Error al validar token |
| 11003 | Error | Error interno en procesamiento |
| 11004 | Error | Error al validar parámetro |
| 11005 | Error | No coinciden montos y detalles |
| 11006 | Error | Fecha de planificación inválida |
| 11007 | Error | No existe tarjeta guardada con esos datos |
| 11008 | Error | No existe cuenta guardada con esos datos |
| 11009 | Error | Versión inválida del protocolo |
| 15002 | Error | Método no autorizado para el organismo |

---

## 6. Método: obtener_tarjetas_cliente

**Descripción:** Devuelve la lista de tarjetas activas guardadas para un cliente, para usar en solicitud_pago_recurrente.

### Parámetros de Entrada

| Nombre | Tipo | Descripción | Requerido |
|--------|------|-------------|-----------|
| version | Texto | Versión del protocolo (fijo "2.0") | ✓ Sí |
| id_organismo | Numérico entero | Código de organismo | ✓ Sí |
| token | Texto | Token de autorización | ✓ Sí |
| datosCliente | Array | Array de DatosClienteRecurrente (mín. 1) | ✓ Sí |

### Estructura: DatosClienteRecurrente

| Campo | Tipo | Descripción | Requerido |
|-------|------|-------------|-----------|
| identificador_cliente | Texto | ID del cliente | ✓ Sí |
| identificador_tarjeta | Texto | ID de tarjeta específica | - Opcional |

### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| id_resp | Numérico entero | Código de respuesta |
| respuesta | Texto | Descripción de respuesta |
| token | Texto | Token utilizado |
| id_organismo | Numérico entero | Código de organismo |
| tarjetas | Array | Array de TarjetasClientes |

### Estructura: TarjetasClientes

| Campo | Tipo | Descripción |
|-------|------|-------------|
| identificador_cliente | Texto | ID del cliente asignado por el Organismo |
| tarjetas | Array | Array de TarjetasCliente |

### Estructura: TarjetasCliente

| Campo | Tipo | Descripción |
|-------|------|-------------|
| identificador_tarjeta | Texto | ID asignado por ePagos a la tarjeta |
| marca | Texto | Marca de tarjeta (ej. VISA, MASTERCARD) |
| id_fp | Numérico entero | Código de forma de pago |
| fecha_alta | Fecha (AAAA-MM-DD) | Fecha de registro de la tarjeta |

### Códigos de Respuesta

| Código | Tipo | Descripción |
|--------|------|-------------|
| 10001 | Correcta | Tarjetas del cliente devueltas |
| 10002 | Error | Error al validar token |
| 10003 | Error | Error interno en devolución |
| 10004 | Error | Error al validar parámetro |
| 10005 | Error | Versión inválida del protocolo |

---

## 7. Método: obtener_cuentas_cliente

**Descripción:** Devuelve la lista de cuentas bancarias activas guardadas para un cliente (DEBIN o Débito Directo), para usar en solicitud_pago_recurrente.

### Parámetros de Entrada

| Nombre | Tipo | Descripción | Requerido |
|--------|------|-------------|-----------|
| version | Texto | Versión del protocolo (fijo "2.0") | ✓ Sí |
| id_organismo | Numérico entero | Código de organismo | ✓ Sí |
| token | Texto | Token de autorización | ✓ Sí |
| datosCliente | Array | Array de DatosClienteCuentaRecurrente (mín. 1) | ✓ Sí |

### Estructura: DatosClienteCuentaRecurrente

| Campo | Tipo | Descripción | Requerido |
|-------|------|-------------|-----------|
| identificador_cliente | Texto | ID del cliente | ✓ Sí |
| medio | Set | Tipo de cuenta a devolver | ✓ Sí |
| identificador_cuenta | Texto | ID de cuenta específica | - Opcional |

### Parámetro: Medio

| Valor | Descripción |
|-------|-------------|
| op_pago_recurrente_medio_debin | Débito Inmediato (DEBIN) |
| op_pago_recurrente_medio_debito_directo | Débito Directo bancario |

### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| id_resp | Numérico entero | Código de respuesta |
| respuesta | Texto | Descripción de respuesta |
| token | Texto | Token utilizado |
| id_organismo | Numérico entero | Código de organismo |
| cuentas | Array | Array de CuentasClientes |

### Estructura: CuentasClientes

| Campo | Tipo | Descripción |
|-------|------|-------------|
| identificador_cliente | Texto | ID del cliente asignado por el Organismo |
| cuentas | Array | Array de CuentasCliente |

### Estructura: CuentasCliente

| Campo | Tipo | Descripción |
|-------|------|-------------|
| identificador_cuenta | Texto | ID asignado por ePagos a la cuenta |
| id_fp | Numérico entero | Código de forma de pago |
| tipo_operacion | Numérico entero | Tipo de impuesto usado en la adhesión |
| cbu | Texto | CBU de la cuenta guardada |
| email | Texto | Email relacionado con el identificador_cliente |
| medio | Set | Forma de pago (DEBIN / Débito Directo) |
| fecha_alta | Fecha (AAAA-MM-DD) | Fecha de registro de la cuenta |

### Códigos de Respuesta

| Código | Tipo | Descripción |
|--------|------|-------------|
| 10001 | Correcta | Cuentas del cliente devueltas |
| 10002 | Error | Error al validar token |
| 10003 | Error | Error interno en devolución |
| 10004 | Error | Error al validar parámetro |
| 10005 | Error | Versión inválida del protocolo |

---

## 8. Método: registrar_cuentas_cliente

**Descripción:** Permite cargar cuentas bancarias ya adheridas a Débito Directo en la organización, para usarlas en solicitud_pago_recurrente. Soporta hasta 100 cuentas por invocación.

### Parámetros de Entrada

| Nombre | Tipo | Descripción | Requerido |
|--------|------|-------------|-----------|
| version | Texto | Versión del protocolo (fijo "2.0") | ✓ Sí |
| id_organismo | Numérico entero | Código de organismo | ✓ Sí |
| token | Texto | Token de autorización | ✓ Sí |
| cuentas | Array | Array de CuentaClienteAgregar (máx. 100) | ✓ Sí |

### Estructura: CuentaClienteAgregar

| Campo | Tipo | Descripción | Requerido |
|-------|------|-------------|-----------|
| identificador_cliente | Texto | ID del cliente (contribuyente) | ✓ Sí |
| tipo_operacion | Número entero | ID del tipo de impuesto | ✓ Sí |
| cuit | Número entero | CUIT del contribuyente (validado contra CBU) | - Opcional |
| cbu | Texto | CBU de la cuenta bancaria | ✓ Sí |
| fecha_adhesion | Fecha | Fecha original de adhesión a Débito Directo | ✓ Sí |

> **Nota:** La combinación de cbu + identificador_cliente + tipo_operacion debe ser única.

### Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| id_resp | Numérico entero | Código de respuesta |
| respuesta | Texto | Descripción de respuesta |
| cuentas | Array | Array de CuentaClienteGenerada |

### Estructura: CuentaClienteGenerada

| Campo | Tipo | Descripción |
|-------|------|-------------|
| identificador_cliente | Texto | ID del cliente asignado por el Organismo |
| identificador_cuenta | Texto | ID generado por ePagos para la cuenta |

### Códigos de Respuesta

| Código | Tipo | Descripción |
|--------|------|-------------|
| 16001 | Correcta | Cuentas adheridas correctamente |
| 16002 | Error | Versión inválida del protocolo |
| 16003 | Error | Error al validar parámetro |
| 16004 | Error | Error al validar token |
| 16005 | Error | Error al intentar adherir la cuenta |

---

## 9. Respuestas de Rechazo de Medio de Pago

Cuando un cobro es rechazado, el campo `respuesta_entidad_cobro` contiene el detalle. Los siguientes códigos indican que el medio de pago ya no es válido y **no debe reintentarse**:

| Código | Medio | Explicación |
|--------|-------|-------------|
| cc_rejected_blacklist | Tarjeta C/D | Tarjeta en lista negra (robada, perdida, etc.) |
| cc_rejected_card_disabled | Tarjeta C/D | Tarjeta no activada por el usuario |
| cc_rejected_bad_filled_date | Tarjeta C/D | Problema con la fecha de vencimiento de la tarjeta |
| cc_rejected_bad_filled_other | Tarjeta C/D | Datos de la tarjeta incorrectos, deberá volver a cargarla |

---

## 10. Flujos de Integración

### 10.1 Flujo: Registro de Tarjeta y Primer Cobro

```
PASO 1 — Obtener Token
  Request:  version="2.0", id_organismo, id_usuario, password, hash
  Response: token="ABC123..."

PASO 2 — Solicitud de Pago Inicial (registrar tarjeta)
  Request:  token, identificador_cliente="CLI_001", email_pagador="user@mail.com"
            monto_operacion=1.00, opc_guardado_obligatorio=true, fp_permitidas=42
  Response: url_qr → usuario paga y guarda tarjeta

PASO 3 — Obtener Tarjetas Guardadas
  Request:  token, datosCliente=[{identificador_cliente:"CLI_001"}]
  Response: identificador_tarjeta="TAR_001", marca="VISA"

PASO 4 — Ejecutar Cobro Recurrente
  Request:  token, convenio=11111, medio="op_pago_recurrente_medio_tarjeta"
            identificador_cliente="CLI_001", identificador_tarjeta="TAR_001"
            monto_operacion=1500.00
  Response: id_resp=09001 → "Pago acreditado"
```

### 10.2 Flujo: Suscripción Mensual Automática

```
Request:
  version: "2.0"
  tipo_operacion: "op_pago_recurrente"
  credenciales: { id_organismo: 12345, token: "ABC123..." }
  descripcion: "Plan Mensual - Impuesto ABC"
  suscripcion_modalidad: "M"
  suscripcion:
    - { fecha: "2026-05-01", monto: 1500.00 }
    - { fecha: "2026-06-01", monto: 1500.00 }
    - { fecha: "2026-07-01", monto: 1500.00 }
  convenio: 11111
  medio: "op_pago_recurrente_medio_tarjeta"
  clientes:
    - { identificador_cliente: "CLI_001", identificador_tarjeta: "TAR_001" }

Response:
  id_resp: 11001 → "Pago recurrente con suscripción planificado"
```

### 10.3 Flujo: Débito Directo con Fecha Específica

```
Request:
  version: "2.0"
  tipo_operacion: "op_pago_recurrente"
  credenciales: { id_organismo: 12345, token: "ABC123..." }
  convenio: 11111
  medio: "op_pago_recurrente_medio_debito_directo"
  operacion: { monto_operacion: 25000.00, numero_operacion: "OP_001" }
  cliente:
    identificador_cliente: "CLI_002"
    identificador_cuenta: "CTA_001"
  fecha_debito: "2026-04-25"  ← mínimo 3 días hábiles desde hoy

Response:
  id_resp: 09001 → "Pago acreditado" (débito procesado en fecha indicada)
```

### 10.4 Flujo: Agregar Tarjeta sin Operación (Billetera)

```
Request:
  id_organismo: 70  ← ePagos Billetera (credenciales específicas)
  monto_operacion: 1.00  ← cargo simbólico
  identificador_cliente: "CLI_001"
  email_pagador: "user@mail.com"
  opc_guardado_obligatorio: true

Resultado: Tarjeta registrada para futuros cobros recurrentes
           El cargo simbólico puede tomarse como pago a cuenta
```

---

## 11. Endpoints SOAP

| Entorno | URL WSDL |
|---------|----------|
| Sandbox (Pruebas) | https://sandbox.epagos.com/wsdl/2.1/index.php?wsdl |
| Producción | https://api.epagos.com/wsdl/2.1/index.php?wsdl |
| Panel Sandbox | https://portalsandbox.epagos.com |
| Panel Producción | https://portal.epagos.com |

- **Protocolo:** SOAP v1.2
- **Versión API:** v2.1 (informar version="2.0" en métodos)
- **Decimales:** Punto (.) como separador, sin miles (ej: 1234.50)
- **Fechas:** ISO 8601 — AAAA-MM-DD

---

## 12. Consideraciones de Diseño

### identificador_cliente

| Regla | Detalle |
|-------|---------|
| Longitud mínima | 6 caracteres |
| Tipo | Alfanumérico |
| Unicidad | Único por contribuyente |
| Responsabilidad | Generación y gestión del Organismo |
| Alcance | Puede ser global o variar por convenio |
| Billetera | Con email_pagador vincula a cuenta ePagos |

### Formas de Pago — Restricción para Recurrencia

| Parámetro | Valor | Efecto |
|-----------|-------|--------|
| fp_permitidas | 42 | Solo Débito Directo y Tarjetas C/D |
| tp_excluidos | "1,3,4,5,6,7" | Excluye tipos específicos de pago |

### Adhesión de Cuenta Bancaria

| Dato solicitado | Descripción |
|----------------|-------------|
| CUIT / CUIL | El CUIT o CUIL del usuario |
| CBU / Alias | CBU o Alias de la cuenta bancaria |

> Se valida que el CBU/Alias sea válido y esté asociado a esa cuenta.

---

## 13. Errores Comunes y Soluciones

| Código Error | Causa | Solución |
|-------------|-------|----------|
| 01002 | Credenciales inválidas | Verificar id_organismo, id_usuario, password y hash |
| 02006 / 09006 | Parámetro inválido | Revisar tipos de datos y formato enviado |
| 09009 | Fecha de débito incorrecta | Asegurar mínimo 3 días hábiles desde hoy |
| 18012 | Falta fecha de vencimiento | Para débito directo, fecha_debito es obligatoria |
| 18013 | Conflicto de fechas | No usar fecha_debito junto con opc_fecha_vencimiento |
| 11007 | Tarjeta no encontrada | Verificar identificador_tarjeta con obtener_tarjetas_cliente |
| 11008 | Cuenta no encontrada | Verificar identificador_cuenta con obtener_cuentas_cliente |
| 15002 | Método no autorizado | Consultar con ejecutivo comercial de ePagos |
| cc_rejected_* | Tarjeta inválida | No reintentar; solicitar nuevo medio de pago |

---

## 14. Seguridad y Buenas Prácticas

- **No almacenar números de tarjeta:** Usar solo identificadores de ePagos (identificador_tarjeta)
- **Tokens temporales:** Regenerar token antes de cada sesión de API
- **Tokens exclusivos:** Un token generado via API no se reutiliza en E-Checkout
- **HTTPS obligatorio:** Todas las comunicaciones deben ser cifradas (TLS)
- **Guardar id_transaccion:** Registrar el ID de cada operación para trazabilidad y auditoría
- **Implementar Webhooks:** Para recibir confirmaciones asíncronas de pago
- **Reintentos controlados:** Implementar lógica de backoff exponencial con límite de intentos
- **Monitorear respuesta_entidad_cobro:** Detectar tarjetas inválidas y dejar de intentar cobros

---

*Documentación extraída de ePagos API v2.1 — Actualizada: Abril 2026*
*Fuente: https://www.epagos.com/desarrolladores.php?secc=recurrencia*
