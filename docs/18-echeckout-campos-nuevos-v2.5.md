# E-Checkout: campos nuevos (actualización v2.5)

Este documento complementa `epagos-echeckout-documentacion-tecnica.md` con los campos incorporados en versiones recientes que no estaban documentados originalmente. Todos pertenecen al POST de inicio de solicitud de pago.

---

## Campos nuevos en opciones (`opc_*`)

| Nombre | Tipo | Default | Descripción |
|--------|------|---------|-------------|
| `opc_operaciones_lote` | Texto | — | Lista de `id_transaccion` separados por comas. Cuando la operación E-Checkout se acredita, traslada esa acreditación automáticamente a las operaciones indicadas (equivalente a llamar `pago_lote` manualmente) |
| `opc_T30_monto_abierto` | Booleano | `false` | Si `true`, el QR interoperable generado es de **monto abierto** (el pagador elige cuánto pagar) |
| `opc_T30_reutilizable` | Booleano | `false` | Si `true`, el QR interoperable generado puede ser pagado **más de una vez** |
| `opc_T30_requiere_orden` | Booleano | `false` | Si `true`, el QR interoperable requiere una **orden de pago** para poder cobrarse (ver `generar_orden_qr`) |
| `opc_una_cuota` | Booleano | `false` | 🆕 Fuerza a que los pagos con **tarjeta de crédito solo puedan realizarse en una cuota** |

---

## Campos nuevos en datos de operación

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `detalle_operacion` | Texto (JSON codificado) | **Opcional.** Detalle del desglose del monto a pagar. Debe ser un array JSON de objetos `DetalleOperacion` codificado con `urlencode()` o `base64_encode()` |
| `detalle_operacion_visible` | Booleano | **Opcional.** Determina si se muestra el detalle al usuario. Default: `true` |

### Estructura: DetalleOperacion

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_item` | Numérico entero | Identificador del elemento del detalle |
| `desc_item` | Texto | Descripción del elemento |
| `monto_item` | Numérico decimal | Monto unitario del elemento |
| `cantidad_item` | Numérico entero | Cantidad del elemento |

> ⚠️ La plataforma valida que `SUM(monto_item * cantidad_item)` coincida con `monto_operacion`.

### Ejemplo de codificación (PHP)

```php
// Opción 1: urlencode
$detalle_op = urlencode(json_encode([
    ['id_item' => '0', 'desc_item' => 'Cuota impuesto provincial', 'monto_item' => 200, 'cantidad_item' => 1],
    ['id_item' => '1', 'desc_item' => 'Recargo fuera de término',  'monto_item' => 10,  'cantidad_item' => 1]
]));

// Opción 2: base64
$detalle_op = base64_encode(json_encode([...]));
```

---

## Nuevo identificador adicional

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `identificador_externo_4` | Texto (65.535) | Identificador adicional para uso del cliente (se suma a los existentes `identificador_externo_2` y `identificador_externo_3`) |

Este campo se devuelve como `identificador_4` en las respuestas de E-Checkout para los medios de pago con tarjeta.

---

## Notas de integración

- Los campos `opc_T30_*` aplican solo cuando la operación incluye Transferencias 3.0 (id_fp = 44) como medio de pago disponible.
- `opc_operaciones_lote` es una alternativa automática al método SOAP `pago_lote` para acreditar lotes al confirmar un pago E-Checkout.
- `opc_una_cuota` es útil para organismos que no quieren manejar financiamiento en cuotas.
