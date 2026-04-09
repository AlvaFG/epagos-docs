# obtener_entidades_pago

Devuelve los medios de pago disponibles para el Organismo. Opcionalmente puede filtrarse por formas de pago específicas.

**API SOAP v2.5** | Código de respuesta prefijo: `03xxx`

---

## Parámetros recibidos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `version` | Texto | Versión del protocolo. Valor fijo `"2.0"` |
| `credenciales` | Estructura `Credenciales` | Contiene la información para generar el token |
| `fp` | Array de `FormaPago` | **Opcional.** Filtra las formas de pago a mostrar |

### Estructura: Credenciales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_organismo` | Numérico entero | Código de organismo |
| `token` | Texto | El token devuelto por `obtener_token` |

### Estructura: FormaPago (filtro opcional)

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_fp` | Numérico entero | Código de forma de pago a incluir |

---

## Respuesta

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `token` | Texto | Token único generado para la solicitud |
| `id_organismo` | Numérico entero | Número de organismo |
| `fp` | Array de `FP` | Formas de pago disponibles |

### Estructura: FP

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `id_fp` | Numérico entero | Código de forma de pago |
| `nombre_fp` | Texto | Descripción de la forma de pago |
| `tipo_fp` | Set | Tipo de forma de pago. Valores posibles: `tipo_fp_presencial`, `tipo_fp_credito`, `tipo_fp_debito`, `tipo_fp_prepago`, `tipo_fp_publicacion`, `tipo_fp_billetera`, `tipo_fp_transferencia` |
| `estado_fp` | Set | Estado. Valores: `estado_fp_activo`, `estado_fp_desactivado` |
| `logos_fp` | Estructura `LogosFP` | URLs de los logos |
| `config_fp` | Estructura `ConfiguracionFP` | Datos de configuración adicionales |
| `adicional_fp` | Texto | Información adicional a solicitar al usuario. Opcional |
| `monto_minimo_fp` | Numérico decimal | Importe mínimo para que esté disponible. Opcional |
| `monto_maximo_fp` | Numérico decimal | Importe máximo que se puede abonar. Opcional |
| `tiempo_acreditacion_fp` | Numérico entero | Horas de acreditación de la forma de pago |

### Estructura: LogosFP

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `seguro_logos_fp` | Texto | URL del logo en HTTPS |
| `nombre_logos_fp` | Texto | URL del logo en HTTP |

### Estructura: ConfiguracionFP

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `patron_config_fp` | Texto | Patrón de validación de la forma de pago (si existe) |
| `longitud_config_fp` | Numérico entero | Longitud de las tarjetas (si disponible) |
| `validacion_config_fp` | Texto | Algoritmo de validación de tarjeta (si existe) |
| `codseg_config_fp` | Booleano | Indica si el código de seguridad es obligatorio |
| `codseg_long_config_fp` | Numérico entero | Longitud del código de seguridad (si disponible) |
| `codseg_ubic_config_fp` | Texto | Ubicación del código de seguridad (si disponible) |

---

## Respuestas posibles

| id_resp | Tipo | Descripción |
|---------|------|-------------|
| `03001` | ✅ Correcta | Entidades devueltas |
| `03002` | ❌ Error | Error al validar el token |
| `03003` | ❌ Error | Error interno al intentar devolver las entidades de pago |
| `03004` | ❌ Error | Error al validar el parámetro: `[_parametro_]` |
| `03005` | ❌ Error | Versión inválida del protocolo |

---

## Notas

- Útil para construir dinámicamente un selector de medios de pago en el front-end.
- Los logos en HTTPS (`seguro_logos_fp`) deben usarse en producción para evitar mixed content.
- El campo `adicional_fp` indica si hay información extra a pedir al usuario (ej. número de patente, CBU, etc.).
