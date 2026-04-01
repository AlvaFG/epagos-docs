# ePagos — Documentación Técnica: Introducción

## Elige la forma de integrarte

En esta sección encontrarás toda la información necesaria para integrar la plataforma de ePagos en tu Organismo. De acuerdo a las funcionalidades que se requieran brindar, existen diferentes alternativas de implementación.

---

## Proceso de integración

El proceso de integración habitualmente consta de las siguientes instancias:

| # | Instancia |
|---|-----------|
| 1 | Inicio de la solicitud de pago |
| 2 | Conciliación de los pagos recibidos |
| 3 | Emisión masiva |

> ePagos provee herramientas para simplificar cada una de las tareas antes mencionadas.

---

## 1. Inicio de la solicitud de pago

Este proceso comienza en el sitio web del Organismo. El contribuyente realiza la consulta de la deuda a abonar o se determinan las obligaciones que debe cancelar. En ese momento se debe iniciar la solicitud del pago. Con cada solicitud se genera una operación en la plataforma, que permitirá luego realizar el seguimiento de su estado y la posterior conciliación.

### Métodos disponibles para iniciar la solicitud

| Método | Descripción | Control | URL de referencia |
|--------|-------------|---------|-------------------|
| **Botón de Pago** | Forma simple de incluir dinámicamente un botón en el sitio web. Se puede definir el estilo y texto. Con un solo click el contribuyente es redirigido a ePagos y retorna al sitio del Organismo con los resultados. | Básico | `/desarrolladores.php?secc=boton_pago` |
| **E-Checkout** | En solo dos pasos permite tener más control sobre la redirección. El contribuyente es redirigido al sitio del Organismo una vez finalizada la acción. | Intermedio | `/desarrolladores.php?secc=checkout` |
| **API** | Permite generar pagos con medios de pago que no incluyan tarjeta de crédito y/o débito. Permite generar boletas de pago sin necesidad de que el usuario visite la plataforma. Siempre se genera una operación para seguir el estado del cobro. | Avanzado | `/desarrolladores.php?secc=api` |

---

## 2. Conciliación de los pagos

Una vez que el Organismo comienza a operar, la segunda tarea de integración es la obtención de los pagos y las imputaciones internas requeridas.

### Comportamiento por tipo de medio de pago

| Medio de pago | Acreditación | Consideraciones |
|---------------|-------------|-----------------|
| Tarjeta de crédito / débito | Inmediata | Cada redirección a la URL de pago exitoso indica operación acreditada |
| Billetera ePagos | Inmediata | Ídem tarjeta |
| Efectivo | Diferida | El usuario debe realizar acciones posteriores al inicio |
| Homebanking | Diferida | El usuario debe realizar acciones posteriores al inicio |

### Recomendación en la conciliación

> Se recomienda utilizar una **combinación de métodos** para evitar que existan operaciones cuya información no sea procesada. Deben contemplarse situaciones como: falla de conectividad en la redirección, o error en el sitio del Organismo al procesar la respuesta.

### Métodos de conciliación disponibles

| # | Método | URL de referencia |
|---|--------|-------------------|
| 1 | Implementando los **Webhooks** de ePagos | `/desarrolladores.php?secc=webhook` |
| 2 | Consultando vía **API** por las operaciones iniciadas | `/desarrolladores.php?secc=conciliacion_api` |
| 3 | Procesando los **archivos de rendiciones** | `/desarrolladores.php?secc=rendicion` |

---

## 3. Emisión masiva

La emisión periódica de las obligaciones del Organismo puede integrarse para ser cobrada a través de ePagos.

| Funcionalidad | Descripción |
|---------------|-------------|
| **Código de barras** | Obtenible vía API para incluir en boletas impresas o distribuidas digitalmente. Centraliza todos los cobros en la plataforma ePagos. |
| **Código QR** | Se genera en cada operación. Permite al contribuyente pagar con su celular boletas de emisión masiva mediante tarjeta de crédito/débito o billetera, con acreditación inmediata. |

---

## Estructura de la documentación (índice)

| Sección | Sub-sección | URL |
|---------|-------------|-----|
| Introducción | — | `/desarrolladores.php?secc=introduccion` |
| Solicitud de pago | Botón de pago | `/desarrolladores.php?secc=boton_pago` |
| Solicitud de pago | E-Checkout | `/desarrolladores.php?secc=checkout` |
| Solicitud de pago | API | `/desarrolladores.php?secc=api` |
| Conciliación de los pagos | Webhooks | `/desarrolladores.php?secc=webhook` |
| Conciliación de los pagos | API | `/desarrolladores.php?secc=conciliacion_api` |
| Conciliación de los pagos | Rendición | `/desarrolladores.php?secc=rendicion` |
| Emisión masiva | — | `/desarrolladores.php?secc=emision_masiva` |
| Recurrencia | — | `/desarrolladores.php?secc=recurrencia` |
| Recursos gráficos | — | `/desarrolladores.php?secc=recursos` |
| Referencia de API | — | `/templates/desarrolladores/referencia.php` |

---

## Información legal

| Campo | Valor |
|-------|-------|
| Razón social | E-Pagos S.A. |
| CUIT | 30-71508429-1 |
| Nota regulatoria | E-Pagos S.A. ofrece servicios de pago y **no está autorizado por el Banco Central** a operar como entidad financiera. Los fondos acreditados en cuentas de pago no constituyen depósitos en una entidad financiera ni están garantizados conforme la legislación aplicable a depósitos en entidades financieras. |
