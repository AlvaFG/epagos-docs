# Botón de Pago ePagos — Documentación Técnica

## Descripción general

ePagos permite iniciar una solicitud de pago en un solo paso mediante un botón JavaScript. Para usarlo se requiere:

- Solicitar una **credencial pública** (clave pública) indicando el o los dominios desde donde se invocará el botón.
- Incluir la librería JavaScript de ePagos en el código fuente.

---

## Ubicación de la librería

| Entorno | URL |
|---|---|
| **Sandbox (pruebas)** | `https://sandbox.epagos.com/quickstart/epagos.min.js` |
| **Producción** | `https://api.epagos.com/quickstart/epagos.min.js` |

---

## Métodos de inicialización

Antes de invocar `botonPago()`, se deben configurar tres métodos sobre el objeto `ePagos`:

| Método | Descripción |
|---|---|
| `ePagos.setClavePublica(hash)` | Indica la clave pública proporcionada por ePagos. |
| `ePagos.setOrganismo(id)` | Especifica el identificador numérico del organismo asignado. |
| `ePagos.setAmbiente(valor)` | Define el entorno de ejecución. Valores posibles: `"T"` (Test/Sandbox) o `"P"` (Producción). |

---

## Método principal: `ePagos.botonPago(config)`

Acepta un objeto de configuración con tres nodos principales: `datosOperacion`, `atributos` y `estiloTooltip`.

---

### `datosOperacion` — Parámetros obligatorios de la operación

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `convenio` | Numérico entero (string) | ✅ Sí | Número de convenio asignado. Ej: `"36589"` |
| `ok_url` | Texto (URL) | ✅ Sí | URL de redirección en caso de pago exitoso. |
| `error_url` | Texto (URL) | ✅ Sí | URL de redirección en caso de error. |
| `monto_operacion` | Numérico decimal | ✅ Sí | Monto de la operación. Ej: `1500` |
| `numero_operacion` | Texto | ❌ No | Identificador interno de la operación. Ej: `"1234"` |

> Para más parámetros opcionales de operación, consultar la sección **E-Checkout** de la documentación.

---

### `atributos` — Configuración de diseño del botón

Todos los atributos son opcionales. Si no se definen, el botón usará un estilo por defecto.

| Nombre | Tipo | Descripción | Valor por defecto |
|---|---|---|---|
| `elemento_destino` | Texto | ID del elemento HTML donde se posicionará el botón. | `elementoEpagos` |
| `label` | Texto | Texto visible en el botón. Ej: resultado → `<button>Iniciar Pago</button>` | `"Pagar"` |
| `id` | Texto | ID del elemento botón generado. Ej: resultado → `id="miBtn_Btn_epagos"` | `"_Btn_epagos"` |
| `className` | Texto | Clase CSS del botón. Ej: resultado → `class="claseMiBtn"` | *(sin clase)* |
| `style` | Texto | Estilos CSS inline adicionales. Ej: `"font-size: 11px; color: #fff; background-color: #e40044"` | *(sin estilo)* |

---

### `estiloTooltip` — Configuración del tooltip informativo

Al pasar el cursor sobre el botón se muestra un tooltip con información de ePagos. Todos los atributos son opcionales.

| Nombre | Tipo | Descripción |
|---|---|---|
| `background_color` | Texto | Color de fondo del tooltip. Ej: `"#A3E4D7"` |
| `font_family` | Texto | Fuente del texto del tooltip. Ej: `"Arial, Times New Roman"` |
| `font_color` | Texto | Color del texto del tooltip. Ej: `"black"` |
| `font_size` | Texto | Tamaño del texto del tooltip. Ej: `"17px"` |

---

## Eventos

| Evento | Descripción | Propiedad del detalle |
|---|---|---|
| `errorEpagos` | Se dispara cuando `botonPago()` detecta errores de estructura o contenido en los parámetros. | `event.detail.mensaje` — Mensaje de error en texto plano. |

---

## Ejemplos de código

### 1. Implementación básica

```html
<html>
  <head>
    <!-- el contenido del head de su página -->
  </head>
  <body>
    <!-- contenido de su web antes del botón -->
    <div id="elementoEpagos"></div>

    <script>
      var script = document.createElement("script");
      script.addEventListener("load", function() {

        // Captura de errores
        window.addEventListener("errorEpagos", function(event) {
          console.error(event.detail.mensaje);
        });

        // Inicialización
        ePagos.setClavePublica("988083f870919095d2661ae5f59a523e");
        ePagos.setOrganismo(99);
        ePagos.setAmbiente("T"); // "T" = Test, "P" = Producción

        // Configuración del botón
        ePagos.botonPago({
          "datosOperacion": {
            "convenio": "36589",
            "numero_operacion": "1234",
            "ok_url": "http://www.su_sitio.com.ar/ok.php",
            "error_url": "http://www.su_sitio.com.ar/error.php",
            "monto_operacion": 1500
          }
        });

      });

      script.src = "https://sandbox.epagos.com/quickstart/epagos.min.js";
      script.async = true;
      document.getElementsByTagName("script")[0].parentNode.appendChild(script);
    </script>

    <!-- contenido de su web después del botón -->
  </body>
</html>
```

---

### 2. Botón con personalización de diseño y tooltip

```javascript
ePagos.botonPago({
  "datosOperacion": {
    "convenio": "36589",
    "ok_url": "http://www.su_sitio.com.ar/ok.php",
    "error_url": "http://www.su_sitio.com.ar/error.php",
    "monto_operacion": 1500
  },
  "atributos": {
    "elemento_destino": "miDiv",
    "label": "Iniciar Pago",
    "id": "miBtn",
    "className": "claseMiBtn",
    "style": "font-size: 11px; color: #ffffff; background-color: #e40044"
  },
  "estiloTooltip": {
    "background_color": "#A3E4D7",
    "font_family": "Arial, Times New Roman",
    "font_color": "black",
    "font_size": "17px"
  }
});
```

---

### 3. Captura y manejo de errores

```javascript
window.addEventListener("errorEpagos", function(event) {
  console.error(event.detail.mensaje);
});
```

---

### 4. Múltiples botones en la misma página

Para mostrar más de un botón (por ejemplo, con diferentes convenios o montos), se llama a `botonPago()` una vez por cada botón, usando `elemento_destino` para posicionarlos en distintos contenedores:

```html
<div id="elementoEpagos"></div>
<div id="miDiv"></div>

<script>
  var script = document.createElement("script");
  script.addEventListener("load", function() {

    window.addEventListener("errorEpagos", function(event) {
      console.error(event.detail.mensaje);
    });

    // Inicialización (se realiza una sola vez)
    ePagos.setClavePublica("988083f870919095d2661ae5f59a523e");
    ePagos.setOrganismo(99);
    ePagos.setAmbiente("T");

    // OPERACION 1 — se renderiza en #elementoEpagos (por defecto)
    ePagos.botonPago({
      "datosOperacion": {
        "convenio": "36589",
        "numero_operacion": "12345",
        "ok_url": "http://www.su_sitio.com.ar/ok_operacion1.php",
        "error_url": "http://www.su_sitio.com.ar/error_operacion1.php",
        "monto_operacion": 737
      }
    });

    // OPERACION 2 — se renderiza en #miDiv
    ePagos.botonPago({
      "datosOperacion": {
        "convenio": "10000",
        "numero_operacion": "12346",
        "ok_url": "http://www.su_sitio.com.ar/ok_operacion2.php",
        "error_url": "http://www.su_sitio.com.ar/error_operacion2.php",
        "monto_operacion": 36
      },
      "atributos": {
        "elemento_destino": "miDiv",
        "style": "background-color: #FFFFE0;"
      },
      "estiloTooltip": {
        "background_color": "#A3E4D7",
        "font_family": "Arial, Times New Roman",
        "font_color": "white"
      }
    });

  });

  script.src = "https://sandbox.epagos.com/quickstart/epagos.min.js";
  script.async = true;
  document.getElementsByTagName("script")[0].parentNode.appendChild(script);
</script>
```

---

## Notas de integración

- El `<div id="elementoEpagos"></div>` es el contenedor **por defecto** donde se renderiza el botón si no se especifica `elemento_destino`.
- La librería se carga de forma **asíncrona** (`script.async = true`), por lo que toda la configuración debe realizarse dentro del listener del evento `load`.
- Se recomienda **siempre** implementar el listener de `errorEpagos` para capturar errores de validación antes del renderizado del botón.
