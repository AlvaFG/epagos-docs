# 📋 Documentación Técnica: Conciliación de Pagos vía API ePagos

## Información General

**Versión de API:** v2.1 (subversión de v2.0)  
**Tipo de API:** SOAP v1.2  
**Fecha de Documento:** Abril 2026

### URLs de Acceso

| Entorno | WSDL SOAP | Panel de Control |
|---------|-----------|------------------|
| Sandbox (Pruebas) | `https://sandbox.epagos.com/wsdl/2.1/index.php?wsdl` | `https://portalsandbox.epagos.com` |
| Producción | `https://api.epagos.com/wsdl/2.1/index.php?wsdl` | `https://portal.epagos.com` |

---

## 1. Arquitectura Recomendada para Conciliación

### Proceso Diario Recomendado

La metodología recomendada por ePagos es implementar un **proceso automático que se ejecute diariamente** con los siguientes pasos:

1. **Obtener Token de Autenticación** usando credenciales de firma
2. **Consultar pagos** con filtro de fecha de acreditación
3. **Procesar resultados** y reconciliar internamente
4. **Consultar pagos adicionales** para operaciones ya pagadas
5. **Verificar contracargos** en caso de tarjetas de crédito/débito
6. **Registrar novedades** en el sistema interno

---

## 2. Método: obtener_pagos

### Descripción
Este método lista las operaciones realizadas y sus detalles junto al estado actual. Es el método base para conciliación diaria.

### Parámetros de Entrada

#### Estructura: Credenciales

| Parámetro | Tipo | Obligatorio | Descripción |
|-----------|------|-------------|-------------|
| `id_organismo` | Numérico entero | Sí | Código del organismo asignado |
| `token` | Texto | Sí | Token devuelto por `obtener_token` |

#### Estructura: Pago (Criterios de Búsqueda)

| Parámetro | Tipo | Obligatorio | Descripción |
|-----------|------|-------------|-------------|
| `CodigoUnicoTransaccion` | Numérico entero | No | ID único de operación en ePagos |
| `ExternoId` | Texto | No | ID proporcionado por el organismo |
| `ExternoId_2` | Texto (100) | No | ID adicional proporcionado |
| `ExternoId_3` | Texto (512) | No | ID adicional (protocolo interno) |
| `Estado` | Set | No | Estado: A, O, V, C, R, P, D |
| `CodBarras` | Texto | No | Código de barras completo |
| `Implementador` | Numérico entero | No | ID del implementador (filtrado) |
| `FechaPagoDesde` | Fecha (aaaa-mm-dd) | No | Inicio rango fecha de pago |
| `FechaPagoHasta` | Fecha (aaaa-mm-dd) | No | Fin rango fecha de pago |
| `FechaAcreditacionDesde` | Fecha (aaaa-mm-dd) | No | Inicio rango acreditación en medio |
| `FechaAcreditacionHasta` | Fecha (aaaa-mm-dd) | No | Fin rango acreditación en medio |
| `FechaNovedadAcreditacionDesde` | Fecha (aaaa-mm-dd) | **Recomendado** | Inicio rango acreditación ePagos |
| `FechaNovedadAcreditacionHasta` | Fecha (aaaa-mm-dd) | **Recomendado** | Fin rango acreditación ePagos |
| `DevolverID4` | Booleano | No | Devuelve contenido del ID4 (default: false) |
| `Pagina` | Numérico entero | No | Número de página (100 resultados c/u) |

### Estructura de Respuesta

#### Respuesta Principal

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `token` | Texto | Token usado en la consulta |
| `id_organismo` | Numérico entero | Código de organismo |
| `pagos` | Array | Operaciones encontradas |
| `pagina` | Numérico entero | Página retornada |
| `cantidadTotal` | Numérico entero | Total de resultados |

#### Estructura: Pago (Respuesta)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `CodigoUnicoTransaccion` | Numérico entero | ID único de operación |
| `Externa` | Texto | ID del organismo |
| `Externa_2` | Texto | ID adicional del organismo |
| `Externa_3` | Texto | ID adicional del organismo |
| `ExternoId_4` | Texto (65.535) | ID 4 (solo si DevolverID4=true) |
| `Convenio` | Numérico entero | Número de convenio |
| `Importe` | Numérico decimal | Monto sin costos de financiación |
| `Estado` | Texto (1) | Estado actual de la operación |
| `Implementador` | Numérico entero | ID del implementador |
| `FechaPago` | Fecha-Hora (aaaa-mm-dd hh:mm:ss) | Inicio del pago |
| `FechaAcreditacion` | Fecha-Hora (aaaa-mm-dd hh:mm:ss) | Acreditación en el medio |
| `FechaNovedadAcreditacion` | Fecha-Hora (aaaa-mm-dd hh:mm:ss) | Acreditación en ePagos |
| `FormaPago` | Array | Información de forma de pago |
| `DatosPagador` | Estructura | Datos del usuario pagador |
| `DatosContribuyente` | Estructura | Datos del contribuyente |
| `PagosAdicionales` | Array | Pagos adicionales relacionados |
| `Recibo` | Texto (URL) | URL para descargar recibo |
| `Url_QR` | Texto (URL) | URL para pago online |

#### Estructura: FormaPago

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Identificador` | Numérico entero | Código de forma de pago |
| `Tipo` | Set | tipo_fp_presencial, tipo_fp_credito, tipo_fp_debito, tipo_fp_prepago, tipo_fp_publicacion, tipo_fp_billetera, tipo_fp_transferencia, tipo_fp_otros |
| `Importe` | Numérico decimal | Importe de la operación |
| `CodigoPago` | Texto | Número de pago corto |
| `CodigoBarras` | Texto | Código de barras |

#### Estructura: DatosPagador

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Nombre` | Texto | Nombre del pagador |
| `Apellido` | Texto | Apellido del pagador |
| `Email` | Texto | Email del pagador |
| `Identificacion` | Estructura | Datos de identificación |

#### Estructura: IdentificacionPagador

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `tipo_doc_pagador` | Numérico entero | Tipo de documento |
| `numero_doc_pagador` | Numérico entero | Número de documento |
| `cuit_doc_pagador` | Numérico entero | CUIT del pagador |

#### Estructura: DatosContribuyente

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Nombre` | Texto | Nombre del contribuyente |
| `Apellido` | Texto | Apellido del contribuyente |
| `Email` | Texto | Email del contribuyente |
| `Dominios` | Array Texto | Dominios relacionados |
| `Identificacion` | Estructura | Datos de identificación |
| `Turno` | Array | Información de turnos |

#### Estructura: TurnosContribuyente

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Fecha` | Fecha (AAAA-MM-DD) | Fecha del turno |
| `Hora` | Hora (HH:MM:SS) | Hora del turno |
| `Dominio` | Texto | Dominio asignado |

#### Estructura: PagosAdicionales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `CodigoUnicoTransaccion` | Numérico entero | ID de operación |
| `FormaPago` | Texto | Descripción de forma de pago |
| `Monto` | Numérico decimal | Importe del pago adicional |
| `FechaPago` | Fecha | Fecha de acreditación |
| `FechaNovedad` | Fecha | Fecha de novedad en ePagos |
| `IdPago` | Numérico entero | ID único del pago adicional |

### Códigos de Respuesta

| Código | Tipo | Significado |
|--------|------|-------------|
| 04001 | ✅ Correcta | Pagos devueltos exitosamente |
| 04002 | ❌ Error | Error al validar el token |
| 04003 | ❌ Error | Error interno al devolver pagos |
| 04004 | ❌ Error | El rango de fechas supera límite permitido |
| 04005 | ❌ Error | Error validando parámetro específico |
| 04006 | ❌ Error | Versión inválida del protocolo |

---

## 3. Método: obtener_pagos_adicionales

### Descripción
Lista pagos adicionales realizados sobre operaciones ya acreditadas. Considera casos como pagos en múltiples agencias o medios diferentes.

### Parámetros de Entrada

#### Estructura: DatosPagosAdicionales

| Parámetro | Tipo | Obligatorio | Descripción |
|-----------|------|-------------|-------------|
| `fecha_desde` | Fecha (AAAA-MM-DD) | Sí | Inicio de carga del pago adicional |
| `fecha_hasta` | Fecha (AAAA-MM-DD) | Sí | Fin de carga del pago adicional |

### Estructura de Respuesta

#### Respuesta Principal

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `token` | Texto | Token usado en consulta |
| `id_organismo` | Numérico entero | Código de organismo |
| `pagos_adicionales` | Array | Vector de pagos adicionales |

#### Estructura: PagosAdicionales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `CodigoUnicoTransaccion` | Numérico entero | ID de la operación |
| `FormaPago` | Texto | Nombre de la forma de pago |
| `Monto` | Numérico decimal | Monto del pago adicional |
| `FechaPago` | Fecha | Fecha de acreditación del pago |
| `FechaNovedad` | Fecha | Fecha de novedad en ePagos |
| `IdPago` | Numérico entero | ID único del pago adicional |

### Códigos de Respuesta

| Código | Tipo | Significado |
|--------|------|-------------|
| 07001 | ✅ Correcta | Pagos adicionales devueltos |
| 07002 | ❌ Error | Error al validar el token |
| 07003 | ❌ Error | Error interno al devolver pagos adicionales |
| 07004 | ❌ Error | El rango de fechas no es correcto |
| 07005 | ❌ Error | Error validando parámetro específico |
| 07006 | ❌ Error | Versión inválida del protocolo |

---

## 4. Método: obtener_contracargos

### Descripción
Lista contracargos/reclamos por pagos no reconocidos en tarjetas de crédito/débito. Permite gestionar disputas y conocer su estado.

### Parámetros de Entrada

#### Estructura: DatosContracargos

| Parámetro | Tipo | Obligatorio | Descripción |
|-----------|------|-------------|-------------|
| `numero` | Numérico entero | No | Número único del contracargo |
| `estado` | Texto | No | Estado: R, C, P, F, FNF, B |
| `fecha_desde` | Fecha | No | Inicio de carga del contracargo |
| `fecha_hasta` | Fecha | No | Fin de carga del contracargo |

### Estructura de Respuesta

#### Respuesta Principal

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id_resp` | Numérico entero | Código de respuesta |
| `respuesta` | Texto | Descripción de la respuesta |
| `token` | Texto | Token usado en consulta |
| `id_organismo` | Numérico entero | Código de organismo |
| `contracargos` | Array | Vector de contracargos |

#### Estructura: DatosContracargosRespuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Numero` | Numérico entero | Número único de contracargo |
| `Estado` | Texto | Estado actual (R/C/P/F/FNF/B) |
| `Medio` | Texto | Nombre del medio de pago |
| `Transaccion` | Numérico entero | Número de operación reclamada |
| `Monto` | Numérico decimal | Monto de la operación reclamada |
| `Tarjeta` | Texto | Número de tarjeta reclamada |
| `Lote` | Numérico entero | Número de lote |
| `Cupon` | Numérico entero | Número de cupón |
| `Respuesta` | Base64 | Comprobante de respuesta (codificado) |
| `Respuesta_formato` | Texto | Extensión del comprobante (ej: PDF) |
| `Comprobante` | Base64 | Comprobante del reclamo (codificado) |
| `Comprobante_formato` | Texto | Extensión del comprobante |
| `Fecha` | Fecha | Fecha de alta del contracargo |
| `Fecha_vencimiento` | Fecha | Fecha de vencimiento |
| `Fecha_resolucion` | Fecha | Fecha de resolución (si fue respondido) |
| `Fecha_confirmacion` | Fecha | Fecha de confirmación si no fue respondido |
| `Fecha_finalizacion` | Fecha | Fecha de finalización si fue solucionado |

### Estados de Contracargo

| Código | Descripción |
|--------|-------------|
| R | Respondido - Se envió respuesta al contracargo |
| C | Confirmado - Pendiente de confirmación de resolución |
| P | Pendiente - Aún sin respuesta |
| F | Finalizado - Resuelto favorablemente |
| FNF | Finalizado No Favorable - Resuelto desfavorablemente |
| B | Baja - Contracargo eliminado |

### Códigos de Respuesta

| Código | Tipo | Significado |
|--------|------|-------------|
| 06001 | ✅ Correcta | Contracargos devueltos |
| 06002 | ❌ Error | Error al validar el token |
| 06003 | ❌ Error | Error interno al devolver contracargos |
| 06004 | ❌ Error | El rango de fechas no es correcto |
| 06005 | ❌ Error | Error validando parámetro específico |
| 06006 | ❌ Error | Versión inválida del protocolo |

---

## 5. Estados de Operación

### Descripción de Estados

| Estado | Código | Descripción |
|--------|--------|-------------|
| Acreditada | A | Pago completado y acreditado en el medio |
| Adeudada | O | Operación pendiente de pago |
| Vencida | V | Operación vencida sin pago |
| Cancelada por usuario | C | Usuario canceló el proceso de pago |
| Rechazada por medio | R | Medio de pago rechazó la transacción |
| Pendiente | P | Solicitud iniciada sin selección de medio |
| Devuelta | D | Operación devuelta, importe reintegrado |

---

## 6. Criterios de Búsqueda Recomendados

### Para Búsqueda Diaria (Recomendado)

```
- Usar: FechaNovedadAcreditacionDesde y FechaNovedadAcreditacionHasta
- Rango: Días anteriores (ej: últimas 24-48 horas)
- Ventaja: Obtiene todas las acreditaciones procesadas por ePagos
```

### Para Búsqueda por Período de Pago

```
- Usar: FechaPagoDesde y FechaPagoHasta
- Rango: Período específico de iniciación de pagos
```

### Para Búsqueda por Acreditación en Medio

```
- Usar: FechaAcreditacionDesde y FechaAcreditacionHasta
- Rango: Período de acreditación en medio de pago externo
```

### Para Búsqueda por Identificador

```
- Usar: ExternoId, ExternoId_2, ExternoId_3
- O: CodigoUnicoTransaccion
- O: CodBarras
```

### Limitaciones Importantes

- ⚠️ El rango de fechas **no debe superar un límite específico** (consultar documentación detallada)
- ⚠️ Se devuelven **100 resultados por página**
- ⚠️ Usar paginación para resultados mayores a 100

---

## 7. Recomendaciones de Implementación

### Arquitectura del Proceso

```
1. AUTENTICACIÓN (Diaria)
   └─ Obtener token usando credenciales
   
2. CONSULTA PAGOS (Diaria)
   ├─ Filtrar por FechaNovedadAcreditacion (últimas 24-48h)
   ├─ Iterar sobre páginas si > 100 resultados
   └─ Procesar cada pago
   
3. CONSULTA PAGOS ADICIONALES (Diaria)
   └─ Filtrar por fecha_desde y fecha_hasta
   
4. CONSULTA CONTRACARGOS (Diaria/Semanal)
   ├─ Filtrar por estado (P, C, R)
   └─ Dar seguimiento a disputas
```

### Mejores Prácticas

| Práctica | Descripción |
|----------|-------------|
| **Frecuencia Diaria** | Ejecutar conciliación mínimo 1x diario |
| **Hora Recomendada** | Fuera de horarios pico (ej: 02:00-06:00) |
| **Rango de Fechas** | Máximo 24-48 horas para mayor rendimiento |
| **Paginación** | Procesar en lotes si hay >100 resultados |
| **Reintentos** | Implementar lógica de reintentos ante errores |
| **Logging** | Registrar todas las consultas y cambios |
| **Reconciliación** | Comparar con registros internos |
| **Token Cache** | Reutilizar token (duración: ~1 hora) |
| **Manejo Errores** | Respetar códigos de error específicos |
| **Validación** | Validar todos los parámetros antes de enviar |

### Casos de Uso Especiales

#### Pagos Adicionales
- **Escenario:** Boletas de efectivo pagadas en 2+ agencias/medios
- **Solución:** Consultar `obtener_pagos_adicionales` diariamente
- **Acción:** Registrar como crédito adicional

#### Contracargos
- **Escenario:** Disputas por tarjeta de crédito/débito
- **Estados a Monitorear:** P (Pendiente), C (Confirmado), R (Respondido)
- **Acción:** Notificar si estado es FNF (desfavorable)

#### Paginación
- **Caso:** Más de 100 resultados en consulta
- **Solución:** Usar parámetro `Pagina` con incrementos
- **Control:** Verificar `cantidadTotal` en respuesta

---

## 8. Estructura JSON de Ejemplo (Respuesta obtener_pagos)

```json
{
  "id_resp": 4001,
  "respuesta": "Pagos devueltos",
  "token": "token_actualizado",
  "id_organismo": 12345,
  "pagina": 1,
  "cantidadTotal": 250,
  "pagos": [
    {
      "CodigoUnicoTransaccion": 987654321,
      "Externa": "REF001",
      "Externa_2": "REF002",
      "Externa_3": "REF003",
      "Convenio": 123,
      "Importe": 1500.00,
      "Estado": "A",
      "Implementador": null,
      "FechaPago": "2026-04-01 10:30:45",
      "FechaAcreditacion": "2026-04-01 11:00:00",
      "FechaNovedadAcreditacion": "2026-04-01 11:15:30",
      "FormaPago": [
        {
          "Identificador": 5,
          "Tipo": "tipo_fp_credito",
          "Importe": 1500.00,
          "CodigoPago": "123456",
          "CodigoBarras": null
        }
      ],
      "DatosPagador": {
        "Nombre": "Juan",
        "Apellido": "Pérez",
        "Email": "juan.perez@example.com",
        "Identificacion": {
          "tipo_doc_pagador": 96,
          "numero_doc_pagador": 12345678,
          "cuit_doc_pagador": 20123456780
        }
      },
      "DatosContribuyente": {
        "Nombre": "Juan",
        "Apellido": "Pérez",
        "Email": "juan.perez@example.com",
        "Dominios": ["dominio1"],
        "Identificacion": {
          "tipo_doc_pagador": 96,
          "numero_doc_pagador": 12345678,
          "cuit_doc_pagador": 20123456780
        },
        "Turno": []
      },
      "PagosAdicionales": [],
      "Recibo": "https://api.epagos.com/recibo/123456789",
      "Url_QR": "https://pagar.epagos.com/qr/123456789"
    }
  ]
}
```

---

## 9. Flujo de Integración Completa

```
┌─────────────────────────────────────────────────────┐
│ INICIO PROCESO DIARIO DE CONCILIACIÓN              │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │  Obtener Token       │
        │  (obtener_token)     │
        │  Validar credenciales│
        └──────────┬───────────┘
                   │
                   ▼
    ┌──────────────────────────────┐
    │ Consultar Pagos              │
    │ (obtener_pagos)              │
    │ - Filtro: Fecha Novedad      │
    │ - Rango: últimas 24-48h      │
    │ - Manejo paginación          │
    └──────────┬───────────────────┘
               │
               ▼
    ┌──────────────────────────────┐
    │ Procesar cada Pago           │
    │ - Validar estado             │
    │ - Reconciliar internamente   │
    │ - Descargar recibo si existe │
    └──────────┬───────────────────┘
               │
               ▼
    ┌──────────────────────────────┐
    │ Consultar Pagos Adicionales  │
    │ (obtener_pagos_adicionales)  │
    │ - Registrar como crédito     │
    └──────────┬───────────────────┘
               │
               ▼
    ┌──────────────────────────────┐
    │ Consultar Contracargos       │
    │ (obtener_contracargos)       │
    │ - Filtrar estados críticos   │
    │ - Realizar seguimiento       │
    └──────────┬───────────────────┘
               │
               ▼
    ┌──────────────────────────────┐
    │ Generar Reporte              │
    │ - Resumen de operaciones     │
    │ - Alertas de issues          │
    │ - Registro de auditoria      │
    └──────────┬───────────────────┘
               │
               ▼
        ┌──────────────────┐
        │ FIN PROCESO      │
        └──────────────────┘
```

---

## 10. Tabla de Errores Comunes

| Error | Causa Probable | Solución |
|-------|----------------|----------|
| 04002 / 06002 / 07002 | Token inválido o expirado | Generar nuevo token con obtener_token |
| 04004 / 06004 / 07004 | Rango de fechas muy grande | Reducir a máximo 48 horas |
| 04005 / 06005 / 07005 | Parámetro mal formateado | Validar formato (aaaa-mm-dd) |
| 04006 / 06006 / 07006 | Versión protocolo incorrecta | Usar "2.0" en parámetro version |
| 04003 / 06003 / 07003 | Error interno en servidor | Reintentar después de 60 segundos |
| Sin respuesta | Timeout de conexión | Verificar conectividad y firewall |

---

## Conclusión

La API de Conciliación de ePagos proporciona tres métodos clave para automatizar completamente el proceso de reconciliación de pagos. La implementación de un proceso diario recomendado utilizando `obtener_pagos` como base, complementado con `obtener_pagos_adicionales` y `obtener_contracargos`, garantiza una visibilidad completa y actualizad del estado de todos los pagos recibidos.

---

**Documento compilado:** Abril 2026  
**Versión de API documentada:** v2.1  
**Fuente:** https://www.epagos.com/desarrolladores.php
