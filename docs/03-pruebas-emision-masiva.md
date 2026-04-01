# Pruebas - Emisión Masiva v2.0

## Plan de Pruebas

### Solicitud de Pago

| Id | Impl. | Prueba | Escenario | Descripción | Obligatorio | Observaciones |
|---|---|---|---|---|---|---|
| 12001 | API | Generación de Token | Token correcto | Usando las credenciales obtener un token | Si | |
| 12002 | API | Solicitud Pago | Operación Combinada | Generar operación con Forma de Pago 34 | Si | El usuario debe poder descargar la boleta con instrucciones |
| 12003 | API | Solicitud Pago | Código de Barra / QR | Obtener código de barra o QR de operación generada | Si | |
| 12004 | API | Solicitud Pago Lote | Generar un lote de operaciones | Generar múltiples operaciones con Forma de Pago 34 | No | |
| 12005 | API | Obtener Pagos | Búsqueda de operación puntual | Buscar por codigounicotransaccion o ID externo único | Si | Fundamental para conciliación y evitar duplicación |
| 12006 | API | Generar QR vinculado | QR para 1 o más operaciones | Generar QR que vincule operaciones existentes | No | |
| 12007 | API | Solicitud Pago | Operación con Fecha de Vencimiento | Generar operación con Fecha de Vencimiento | Si | |

### Conciliaciones

| Id | Impl. | Prueba | Escenario | Descripción | Obligatorio | Observaciones |
|---|---|---|---|---|---|---|
| 21001 | API | Obtener Pago | Búsqueda por rango de fechas | Buscar operaciones con fecha desde y hasta | Si | Fundamental para conciliación |
| 21002 | API | Obtener Pago Adicionales | Búsqueda por rango de fechas | Buscar rendiciones del día anterior o más | Si | |
| 21003 | API | Obtener Rendiciones | Por fecha de pago | | No | |
| 21004 | API | Obtener Rendiciones | Por fecha de depósito | Buscar rendiciones por fechas de depósitos | No | |
| 21005 | API | Obtener Contracargos | Por fecha | Buscar contracargos por fecha de recepción | No | |
| 22001 | Webhook | Informar Pagos | Pago recibido por primera vez | URL para recibir avisos de pago nuevo | No | |
| 22002 | Webhook | Informar Pagos | Pago ya registrado | URL para recibir avisos de pago existente | No | |
| 22003 | Webhook | Informar Devoluciones | URL para devoluciones | URL para avisos de devoluciones | No | |

### Rendiciones

| Id | Impl. | Prueba | Escenario | Descripción | Obligatorio | Observaciones |
|---|---|---|---|---|---|---|
| 31001 | Standard | Importar archivo de movimientos | Procesar archivo de rendición | Procesar para conciliar movimientos | Si | No obligatorio si se usa API Obtener Pago |
| 32001 | Personalizada | Importar archivo personalizado | Procesar archivo de rendición | Formato acordado | No | |

---

## Detalle de Pruebas

### 12001 - Generación de Token (método: `obtener_token`)

| Clave | Valor |
|---|---|
| id_organismo | (dato proporcionado por mail) |
| id_usuario | (dato proporcionado por mail) |
| password | (dato proporcionado por mail) |
| hash | (dato proporcionado por mail) |

**Resultado esperado:** Un número de token.

### 12002 - Solicitud Pago - Operación Combinada (método: `solicitud_pago`)

| Clave | Valor |
|---|---|
| id_fp | 34 |
| identificador_externo_2 | 12002 |
| monto_operacion | 12002 |

**Resultado esperado:** Nro de operación y estado **adeudado**.

### 12003 - Solicitud Pago - Código de Barra / QR (método: `solicitud_pago`)

| Clave | Valor |
|---|---|
| identificador_externo_2 | 12003 |
| opc_devolver_qr | true |
| opc_devolver_codbarras | true |

**Resultado esperado:** Datos de la operación en estado **adeudada** y campo `codigo_barras_fp` con el nro de código de barras (versión API 2.1).

### 12004 - Solicitud Pago Lote (método: `solicitud_pago_lote`)

**Operación 1:**

| Clave | Valor |
|---|---|
| Id_fp | 34 |
| identificador_externo_2 | 120051 |
| monto_operacion | 51 |

**Operación 2:**

| Clave | Valor |
|---|---|
| Id_fp | 34 |
| identificador_externo_2 | 120052 |
| monto_operacion | 52 |

**Operación 3:**

| Clave | Valor |
|---|---|
| Id_fp | 34 |
| identificador_externo_2 | 120053 |
| monto_operacion | 53 |

**Resultado esperado:** IDs de todas las operaciones generadas.

### 12006 - Generar QR vinculado (método: `generar_qr_vinculado`)

Utilizar los IDs de operación de la prueba 12004 para vincularlos.

### 12007 - Solicitud Pago - Operación con Fecha de Vencimiento

| Clave | Valor |
|---|---|
| id_fp | 4 |
| identificador_externo_2 | 12008 |
| monto_operacion | 12008 |
| opc_fecha_vencimiento | [fecha_de_hoy] formato aaaa-mm-dd |
