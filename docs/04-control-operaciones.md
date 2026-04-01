# Control de Operaciones v2.0

ePagos ofrece 4 niveles de implementación para verificar que una operación fue debidamente acreditada y marcarla en el sistema para evitar pagos duplicados.

## Nivel 1: Callback

Tras finalizar la operación, el contribuyente es redirigido a la URL de ok o error definida por el Organismo. En esa URL se reciben todos los datos de la operación y su estado.

Para canales con **acreditación inmediata** desde el botón de pagos (Tarjetas de Crédito, Tarjetas de Débito, Billetera), se puede marcar la deuda como paga directamente.

## Nivel 2: Webhook

Mediante una URL brindada por el Organismo, ePagos devuelve en **tiempo real** la información de las operaciones acreditadas. El Organismo puede marcar la operación acreditada en sus sistemas.

**Vigente para TODOS los canales de pago.**

## Nivel 3: API

A través de un proceso en segundo plano, usando el método `obtener_pagos` de la API, se pueden buscar las operaciones realizadas y conocer su estado y detalle.

## Nivel 4: Rendición

- **Manual:** En base a los archivos de rendición diaria, enviados por mail o descargados desde el Panel de Control. Contienen todas las operaciones acreditadas del día anterior.
- **Automática:** Mediante el método `obtener_rendiciones` de la API.
