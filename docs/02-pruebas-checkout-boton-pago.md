# Pruebas - eCheckout / Botón de Pago v2.0

## Plan de Pruebas

### Solicitud de Pago

| Id | Implementación | Prueba | Escenario | Descripción | Obligatorio | Observaciones |
|---|---|---|---|---|---|---|
| 11001 | Botón de Pago / eCheckout | Generación de Token | Token correcto | Usando las credenciales obtener un token | Si | |
| 11002 | Botón de Pago / eCheckout | Tarjeta crédito/débito | Prueba de acreditación | El usuario completa correctamente todos los datos | Si | Verificar manejo de múltiples solicitudes sobre la misma deuda para evitar pagos duplicados |
| 11003 | Botón de Pago / eCheckout | Tarjeta crédito/débito | Prueba de rechazo | El usuario ingresa algún nro de tarjeta inválido | Si | Ingresar algún dato incorrecto para que rechace la operación |
| 11004 | Botón de Pago / eCheckout | Tarjeta crédito/débito | Prueba de cancelación | El usuario cancela o abandona la operación | Si | |
| 11005 | Botón de Pago / eCheckout | Efectivo / Homebanking | Prueba de generación | La operación debe quedar en estado Adeudada | Si | El usuario debe poder descargar la boleta con instrucciones de pago |
| 11006 | Botón de Pago / eCheckout | Efectivo / Homebanking | Prueba de cancelación | El usuario cancela o abandona la operación | Si | |
| 11007 | Botón de Pago / eCheckout | Efectivo / Homebanking | Operación con Fecha de Vencimiento | Generar una operación con Fecha de Vencimiento | Si | |
| 11008 | Botón de Pago / eCheckout | Transferencia | Prueba de generación | Operación en estado Adeudada, CUIT debe ser el de la cuenta de origen | No | |
| 11009 | Botón de Pago / eCheckout | Debin | Prueba de generación | Operación en estado Adeudada | No | Indicar CUIT y CBU de la tabla de pruebas |

### Conciliaciones

| Id | Implementación | Prueba | Escenario | Descripción | Obligatorio | Observaciones |
|---|---|---|---|---|---|---|
| 21001 | API | Obtener Pago | Búsqueda por rango de fechas | Buscar operaciones con fecha desde y hasta | Si | Fundamental para conciliación y evitar duplicación |
| 21002 | API | Obtener Pago Adicionales | Búsqueda por rango de fechas | Buscar rendiciones en base a fecha del día anterior o más | Si | |
| 21003 | API | Obtener Rendiciones | Por fecha de pago | | No | |
| 21004 | API | Obtener Rendiciones | Por fecha de depósito | Buscar rendiciones por fechas de depósitos | No | |
| 21005 | API | Obtener Contracargos | Por fecha | Buscar contracargos por fecha de recepción | No | |
| 22001 | Webhook | Informar Pagos | Pago recibido por primera vez | URL para recibir avisos de pago (pago nuevo) | No | |
| 22002 | Webhook | Informar Pagos | Pago ya registrado | URL para recibir avisos de pago (pago existente) | No | |
| 22003 | Webhook | Informar Devoluciones | URL para recibir devoluciones | URL para avisos de devoluciones | No | |

### Rendiciones

| Id | Implementación | Prueba | Escenario | Descripción | Obligatorio | Observaciones |
|---|---|---|---|---|---|---|
| 31001 | Standard | Importar archivo de movimientos | Procesar archivo de rendición | Procesar para conciliar movimientos | Si | No obligatorio si se usa API Obtener Pago |
| 32001 | Personalizada | Importar archivo personalizado | Procesar archivo de rendición | Formato acordado, verificar datos para conciliación | No | |

---

## Detalle de Pruebas

### 11001 - Generación de Token

**Método:** POST HTTP

| Clave | Valor |
|---|---|
| id_organismo | (dato proporcionado por mail) |
| id_usuario | (dato proporcionado por mail) |
| password | (dato proporcionado por mail) |
| hash | (dato proporcionado por mail) |

**Resultado esperado:** Un número de token.

### 11002 - Tarjeta crédito/débito - Prueba de acreditación

| Clave | Valor |
|---|---|
| Identificador_externo_2 | 11002 |
| monto_operacion | 11002 |

Usar las tarjetas de pruebas de la documentación.

**Resultado esperado:** Operación en estado **acreditada** y redirección a `url_ok`.

### 11003 - Tarjeta crédito/débito - Prueba de rechazo

| Clave | Valor |
|---|---|
| Identificador_externo_2 | 11003 |
| monto_operacion | 11003 |

Completar en la interfaz de checkout: **Nombre y Apellido Titular = "CALL"**

**Resultado esperado:** Operación en estado **rechazada** y redirección a `url_error`.

### 11004 - Tarjeta crédito/débito - Prueba de cancelación

| Clave | Valor |
|---|---|
| Identificador_externo_2 | 11004 |
| monto_operacion | 11004 |

Iniciar la solicitud y cancelarla con el botón Cancelar en la interfaz de checkout.

**Resultado esperado:** Operación en estado **cancelada** y redirección a `url_error`.

### 11005 - Efectivo / Homebanking - Prueba de generación

| Clave | Valor |
|---|---|
| Identificador_externo_2 | 11005 |
| monto_operacion | 11005 |

Usar las cuentas de prueba de la documentación.

**Resultado esperado:** Operación en estado **adeudada**.

### 11006 - Efectivo / Homebanking - Prueba de cancelación

| Clave | Valor |
|---|---|
| Identificador_externo_2 | 11006 |
| monto_operacion | 11006 |

Iniciar la solicitud y cancelarla con el botón Cancelar.

**Resultado esperado:** Operación en estado **cancelada** y redirección a `url_error`.

### 11007 - Operación con Fecha de Vencimiento

| Clave | Valor |
|---|---|
| identificador_externo_2 | 11007 |
| monto_operacion | 11007 |
| opc_fecha_vencimiento | [fecha_del_dia] formato aaaa-mm-dd |

**Resultado esperado:** Operación en estado **acreditada** y redirección a `url_ok`.

### 11008 - E-Transferencias - Prueba de generación

| Clave | Valor |
|---|---|
| Identificador_externo_2 | 11008 |
| monto_operacion | 11008 |

Completar en la interfaz de checkout: **CUIT = 30715084291**

**Resultado esperado:** Operación en estado **acreditada** y redirección a `url_ok`.

### 11009 - Debin - Prueba de generación

| Clave | Valor |
|---|---|
| Identificador_externo_2 | 11009 |
| monto_operacion | 11009 |

Usar las cuentas de prueba de la documentación.
