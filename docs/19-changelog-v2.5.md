# API v2.5 — Campos nuevos E-Checkout

> Estos campos son exclusivos de la versión 2.5 y aplican al flujo de E-Checkout.
> Endpoint WSDL: `https://api.epagos.com/wsdl/2.5/index.php?wsdl`

---

## Campos nuevos en solicitud E-Checkout

### Control de cuotas

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `opc_una_cuota` | Booleano | Si `true`, fuerza el pago en 1 cuota sin opción de cuotas para el usuario |

### Transferencias 3.0

Aplican solo cuando la operación incluye Transferencias 3.0 (`id_fp = 44`) como medio de pago disponible.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `opc_T30_cbu_destino` | Texto | CBU/CVU de la cuenta destino para Transferencias 3.0 |
| `opc_T30_cuit_destino` | Texto | CUIT del titular de la cuenta destino |
| `opc_T30_nombre_destino` | Texto | Nombre/razón social del titular destino |

### Operaciones en lote

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `opc_operaciones_lote` | Booleano | Si `true`, acredita automáticamente el lote al confirmar el pago E-Checkout. Alternativa automática al método SOAP `pago_lote` |

### Detalle de operación

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `detalle_operacion` | Array JSON (urlencode o base64) | Desglose del monto a pagar |
| `detalle_operacion_visible` | Booleano | Si se muestra el detalle al usuario (default: `true`) |

**Estructura de cada ítem en `detalle_operacion`:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id_item` | int | Identificador del elemento |
| `desc_item` | Texto | Descripción del elemento |
| `monto_item` | decimal | Monto unitario |
| `cantidad_item` | int | Cantidad |

> ⚠️ La plataforma valida que `SUM(monto_item × cantidad_item)` coincida con `monto_operacion`.

**Ejemplo de codificación (PHP):**

```php
// Opción 1: urlencode
$detalle_op = urlencode(json_encode([
    ['id_item' => '0', 'desc_item' => 'Cuota impuesto provincial', 'monto_item' => 200, 'cantidad_item' => 1],
    ['id_item' => '1', 'desc_item' => 'Recargo fuera de término',  'monto_item' => 10,  'cantidad_item' => 1]
]));

// Opción 2: base64
$detalle_op = base64_encode(json_encode([...]));
```

### Identificador adicional

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `identificador_externo_4` | Texto (65.535) | Cuarto identificador adicional para uso del cliente. Se suma a `identificador_externo_2` y `identificador_externo_3` |

> Se devuelve como `identificador_4` en las respuestas de E-Checkout para medios de pago con tarjeta.

---

## Comparativa de campos por versión

| Campo | v1.0 | v2.0 | v2.1 | v2.4 | v2.5 |
|-------|:----:|:----:|:----:|:----:|:----:|
| `opc_una_cuota` | ❌ | ❌ | ❌ | ❌ | ✅ |
| `opc_T30_cbu_destino` | ❌ | ❌ | ❌ | ❌ | ✅ |
| `opc_T30_cuit_destino` | ❌ | ❌ | ❌ | ❌ | ✅ |
| `opc_T30_nombre_destino` | ❌ | ❌ | ❌ | ❌ | ✅ |
| `opc_operaciones_lote` | ❌ | ❌ | ❌ | ❌ | ✅ |
| `detalle_operacion` (E-Checkout) | ❌ | ❌ | ❌ | ❌ | ✅ |
| `detalle_operacion_visible` | ❌ | ❌ | ❌ | ❌ | ✅ |
| `identificador_externo_4` | ❌ | ❌ | ❌ | ❌ | ✅ |
