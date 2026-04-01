# 📦 Recursos Técnicos — ePagos Desarrolladores

> Fuente: https://www.epagos.com/desarrolladores.php?secc=recursos

---

## 🖼️ Logos de Medios de Pago

ePagos provee una URL dinámica para generar imágenes con los logos de los medios de pago operados por el organismo. Debe incluirse en el atributo `src` de un tag `<img>`.

### Endpoints por entorno

| Entorno | URL |
|---|---|
| **Sandbox (Testing)** | `https://sandbox.epagos.com/logos.php` |
| **Producción** | `https://api.epagos.com/logos.php` |

---

### Parámetros de la URL

Los parámetros marcados con `(*)` son **obligatorios**.

| Nombre | Tipo | Descripción |
|---|---|---|
| `id_organismo` (*) | Numérico entero | Identificador del organismo asignado por ePagos |
| `tipo_fp` (*) | Texto | Identificador(es) de las formas de pago a incluir en la imagen. Se envían separados por comas. Ej: `1,2` |
| `variante` | Texto | Tamaño de la imagen: `chica` (500px de ancho), `mediana` (1000px de ancho), `grande` (2200px de ancho) |

---

### Ejemplo de uso

```
https://api.epagos.com/logos.php?id_organismo=1234&tipo_fp=1&variante=chica
```

---

## 📚 Manual de Marca (Brandbook)

| Recurso | Tipo | URL de descarga |
|---|---|---|
| ePagos Brandbook v1.0 | PDF descargable | `https://www.epagos.com/recursos/epagos_brandbook_v1.0.pdf` |

---

## 🔗 Navegación de la Documentación Técnica

| Sección | URL |
|---|---|
| Introducción | `https://www.epagos.com/desarrolladores.php?secc=introduccion` |
| Botón de pago | `https://www.epagos.com/desarrolladores.php?secc=boton_pago` |
| E-Checkout | `https://www.epagos.com/desarrolladores.php?secc=checkout` |
| API de Solicitud de Pago | `https://www.epagos.com/desarrolladores.php?secc=api` |
| Webhooks | `https://www.epagos.com/desarrolladores.php?secc=webhook` |
| Conciliación — API | `https://www.epagos.com/desarrolladores.php?secc=conciliacion_api` |
| Rendición | `https://www.epagos.com/desarrolladores.php?secc=rendicion` |
| Emisión masiva | `https://www.epagos.com/desarrolladores.php?secc=emision_masiva` |
| Recurrencia | `https://www.epagos.com/desarrolladores.php?secc=recurrencia` |
| Recursos gráficos | `https://www.epagos.com/desarrolladores.php?secc=recursos` |
| Referencia de API | `https://www.epagos.com/templates/desarrolladores/referencia.php` |
| Plugins y SDKs (GitHub) | `https://github.com/epagos/sdk` |

---

## 🛠️ Herramientas y Accesos

| Herramienta | URL |
|---|---|
| Portal / Inicio de sesión | `https://portal.epagos.net` |
| Ayuda / FAQ | `https://www.epagos.com/ayuda.php` |
| Status del servicio | `https://status.epagos.com.ar/` |

---

## 📱 Apps móviles

| Plataforma | URL |
|---|---|
| Android (Google Play) | `https://play.google.com/store/apps/details?id=com.epagos.epagos` |
| iOS (App Store) | `https://apps.apple.com/us/app/ePagos/id1477245060?l=es&ls=1` |

---

> **Nota legal:** E-Pagos S.A., CUIT: 30-71508429-1. No está autorizado por el BCRA a operar como entidad financiera. Los fondos en cuentas de pago no constituyen depósitos bancarios.
