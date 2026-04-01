# Documentacion Tecnica: Rendiciones ePagos

## Tabla de Contenidos
1. [Conceptos Generales](#conceptos-generales)
2. [Formatos de Archivos](#formatos-de-archivos)
3. [Campos de Datos](#campos-de-datos)
4. [Frecuencia de Generacion](#frecuencia-de-generacion)
5. [Procesamiento de Rendiciones](#procesamiento-de-rendiciones)
6. [Obtencion de Rendiciones](#obtencion-de-rendiciones)
7. [Referencia API](#referencia-api)
8. [Codigos de Respuesta](#codigos-de-respuesta)

---

## Conceptos Generales

### ¿Qué es una Rendicion?

Todos los días la plataforma genera para su Organismo un cierre de las operaciones recaudadas por convenio, a esto se denomina **rendicion**.

### Contenido de las Rendiciones

Las **rendiciones** contienen:
- Todos los pagos recibidos y acreditados en ese día
- Pagos que pudieran haber quedado de días anteriores sin rendir
- Otros conceptos que suman a lo recaudado (ej: pagos adicionales)
- Conceptos que restan del monto a rendir (ej: contracargos de tarjetas)

---

## Formatos de Archivos

### Descarga Manual del Panel de Control

#### Archivo ZIP
- Formato: ZIP
- Contenido: Caratula e informacion de movimientos
- Frecuencia: Diaria
- Ubicacion: Panel de control de la plataforma

#### Archivo CSV
- Formato: CSV (Comma-Separated Values)
- Contenido: Detalle de cada pago y descuento recibido
- Ubicacion: Dentro del archivo ZIP
- Uso: Puede importarse directamente en sistemas

### Obtencion via API

- Formato de Respuesta: SOAP XML
- Contenido: Estructura de datos detallada
- Ventaja: No requiere procesar archivos separados

| Aspecto | Descarga Manual | API |
|---------|----------------|-----|
| Formato principal | ZIP + CSV | XML/SOAP |
| Automatizacion | Manual | Automatica |
| Procesamiento | Requiere importar CSV | Datos en respuesta |
| Recomendacion | No recomendada | Recomendada |

---

## Campos de Datos

### Estructura de Respuesta de Rendicion

#### Datos Principales (DatosRendicionRespuesta)

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| Numero | Numerico entero | Numero unico de rendicion |
| Secuencia | Numerico entero | Numero de rendicion para el organismo |
| Convenio | Numerico entero | Numero de convenio |
| Estado | Texto | Estado: A=Activa, D=Depositada |
| Fecha_desde | Fecha | Fecha desde de la rendicion |
| Fecha_hasta | Fecha | Fecha hasta de la rendicion |
| Fecha_estimada_deposito | Fecha | Fecha estimada del deposito |
| Fecha_deposito | Fecha | Fecha real del deposito |
| Monto | Numerico decimal | Monto bruto |
| Monto_depositado | Numerico decimal | Monto depositado |
| Monto_comision | Numerico decimal | Monto de comision |
| Monto_IVA | Numerico decimal | IVA de la comision |
| Monto_IIBB | Numerico decimal | IIBB de la comision |
| Monto_CC | Numerico decimal | Monto de contracargos |
| Monto_ND | Numerico decimal | Monto no depositable |
| Cantidad | Numerico entero | Cantidad de transacciones |
| Detalles | Array | Array de DetalleRendicion |
| Contracargos | Array | Array de DetalleContracargo |
| Devoluciones | Array | Array de DetalleContracargo |
| Contenido | Texto | URL del archivo ZIP |

#### Detalle de Operacion (DetalleRendicion)

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| Codigo_unico_transaccion | Numerico entero | ID unico de la operacion |
| Monto | Numerico decimal | Importe de la operacion |
| Numero_operacion | Texto | Identificacion externa |
| Depositable | Booleano | Si sera depositada |

#### Detalle de Contracargo (DetalleContracargo)

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| Codigo_unico_transaccion | Numerico entero | ID unico |
| Monto | Numerico decimal | Importe |
| Numero_rendicion | Numerico entero | Rendicion asociada |

---

## Frecuencia de Generacion

### Generacion Diaria

- Frecuencia: Todos los días
- Período: Se genera un cierre por cada día calendario
- Acreditacion: Incluye pagos acreditados en ese día
- Retroactivos: Puede incluir pagos de días anteriores no rendidos

---

## Procesamiento de Rendiciones

### Alternativa 1: Conciliacion Manual (No recomendada)

Procedimiento:
1. Ingresar al panel de control diariamente
2. Descargar el archivo .zip de rendiciones
3. Extraer el archivo .csv del .zip
4. Importar el .csv en el sistema

Desventajas:
- Requiere proceso manual
- Propenso a errores
- Menos automatizado

### Alternativa 2: Conciliacion via API (Recomendada)

Procedimiento:
1. Obtener token mediante obtener_token
2. Consultar rendiciones con obtener_rendiciones
3. Procesar operaciones en la respuesta
4. Acreditar pagos automaticamente

Ventajas:
- Totalmente automatizado
- Datos en respuesta de API
- No requiere procesar archivos
- Mas seguro y confiable

---

## Obtencion de Rendiciones

### Metodo API: obtener_rendiciones

#### URLs de Acceso

| Entorno | WSDL SOAP |
|---------|-----------|
| Sandbox | https://sandbox.epagos.com/wsdl/2.1/index.php?wsdl |
| Produccion | https://api.epagos.com/wsdl/2.1/index.php?wsdl |

#### Panel de Control

| Entorno | URL |
|---------|-----|
| Sandbox | https://portalsandbox.epagos.com |
| Produccion | https://portal.epagos.com |

#### Parametros de Entrada

##### Credenciales

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| id_organismo | Numerico entero | Codigo de organismo |
| token | Texto | Token devuelto |

##### Filtro de Rendicion

| Parametro | Tipo | Descripcion | Formato |
|-----------|------|-------------|---------|
| numero | Numerico entero | Numero unico (opcional) | 12345 |
| secuencia | Numerico entero | Numero incremental (opcional) | 1 |
| fecha_desde | Fecha | Inicio de período | AAAA-MM-DD |
| fecha_hasta | Fecha | Fin de período | AAAA-MM-DD |
| fecha_deposito_desde | Fecha | Inicio de deposito (opt) | AAAA-MM-DD |
| fecha_deposito_hasta | Fecha | Fin de deposito (opt) | AAAA-MM-DD |

##### Version

| Parametro | Tipo | Valor |
|-----------|------|-------|
| version | Texto | "2.0" |

#### Parametros de Respuesta

| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| id_resp | Numerico entero | Codigo de respuesta |
| respuesta | Texto | Descripcion |
| token | Texto | Token utilizado |
| id_organismo | Numerico entero | Codigo de organismo |
| rendicion | Array | Array de DatosRendicionRespuesta |

---

## Referencia API

### Informacion General

- Tipo: API SOAP v1.2
- Autenticacion: Token basado en credenciales
- Version API: v2.1 (actual)
- Version a reportar: 2.0 (subversion de v2.0)

### Requisitos Previos

1. Solicitar credenciales de firma al ejecutivo
2. Obtener token mediante obtener_token
3. Utilizar token en todas las invocaciones

---

## Codigos de Respuesta

| Codigo | Tipo | Descripcion |
|--------|------|-------------|
| 05001 | Correcta | Rendiciones devueltas |
| 05002 | Error | Error al validar token |
| 05003 | Error | Error interno |
| 05004 | Error | Rango de fechas fuera de limite |
| 05005 | Error | Error en validacion de parametros |

---

## Notas Importantes

1. Adaptacion de Formato: Consulte con ejecutivo
2. Depositos: Incluye informacion de depositos
3. Contracargos y Devoluciones: En arrays separados
4. Monto No Depositable: Indica movimientos no depositables
5. Usar API: Implementar conciliacion automatica

---

Documento generado: Abril 2026
Version API: 2.1
Ultima actualizacion: 2026-04-01
