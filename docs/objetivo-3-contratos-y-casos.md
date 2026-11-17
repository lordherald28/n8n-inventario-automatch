# Objetivo 3 — Contratos de datos y reglas de conciliación

**Alcance:** definición documental. No implementa SQL, endpoints, workflows ni integraciones. Los datos de ejemplo son sintéticos y se basan en la factura de prueba adjunta.

## Alcance y evidencias

- Se revisaron `AGENTS.md`, `README.md`, `docs/project-structure.md`, las skills ODD y verification, y `docs/git-flow.md`. El repositorio es un harness sin implementación funcional previa.
- La factura de prueba tiene una página e identifica al proveedor por nombre e ID fiscal; contiene factura `F-1540`, fecha `16/11/2026`, orden `OC-2026-0087`, recepción `RC-115` y cuatro líneas con SKU, código de barras, descripción, cantidad y valores monetarios.
- El proveedor del ejemplo se completa con un ID interno sintético `PROV-001`, conforme al acuerdo de este objetivo. Ese ID no aparece en el PDF y debe corresponder a la identidad interna compartida por los sistemas.
- Las cantidades y valores monetarios transcritos del PDF se verificaron: 12×3.25=39.00; 20×2.10=42.00; 15×4.80=72.00; 30×0.95=28.50. Los subtotales suman USD 181.50, igual al subtotal y total del PDF; impuestos 0.00 USD.

## Contrato propuesto: recepción externa por Webhook POST

El emisor envía una recepción JSON. `supplier.id` es el identificador interno del proveedor; `invoice_number` permite localizar la factura única asociada. `receipt_reference` identifica el documento de recepción. SKU es la clave principal del producto.

```json
{
  "supplier": {
    "id": "PROV-001",
    "name": "Suministros Delta Caribe S.R.L.",
    "tax_id": "SDC-482901"
  },
  "invoice_number": "F-1540",
  "receipt_reference": "RC-115",
  "received_at": "2026-11-16T10:30:00-05:00",
  "items": [
    { "sku": "ACE-500", "barcode": "7501234567890", "description": "Aceite vegetal 500 ml", "quantity": 12 },
    { "sku": "JAB-250", "barcode": "7501234567891", "description": "Jabón líquido 250 ml", "quantity": 20 },
    { "sku": "DET-1000", "barcode": "7501234567892", "description": "Detergente 1 L", "quantity": 15 },
    { "sku": "AGU-1500", "barcode": "7501234567893", "description": "Agua mineral 1.5 L", "quantity": 30 }
  ]
}
```

| Campo | Tipo | Regla documentada |
|---|---|---|
| `supplier.id` | string | ID interno estable del proveedor. |
| `supplier.name` | string | Nombre del proveedor. |
| `supplier.tax_id` | string | NIF/ID fiscal del proveedor. |
| `invoice_number` | string | Número de factura; parte de la identidad de la factura junto con el ID interno del proveedor. |
| `receipt_reference` | string | Identificador de la recepción usado para detectar reenvíos/conflictos del mismo documento. |
| `received_at` | fecha/hora ISO 8601 con zona | Fecha/hora declarada de recepción. |
| `items` | array no vacío | Líneas recibidas. |
| `items[].sku` | string | Identificador principal del producto y clave de comparación. |
| `items[].barcode` | string | Dato adicional; no reemplaza el SKU. |
| `items[].description` | string | Dato descriptivo. |
| `items[].quantity` | número positivo | Cantidad recibida que se compara con la factura. No se compara el precio. |

## Modelo de factura de proveedor

Modelo normalizado de los campos de prueba visibles en el PDF, más el ID interno acordado. Los valores monetarios se conservan como datos de factura, pero no intervienen en la conciliación.

```json
{
  "supplier": {
    "id": "PROV-001",
    "name": "Suministros Delta Caribe S.R.L.",
    "tax_id": "SDC-482901"
  },
  "invoice_number": "F-1540",
  "invoice_date": "2026-11-16",
  "purchase_order_reference": "OC-2026-0087",
  "receipt_reference": "RC-115",
  "currency": "USD",
  "items": [
    { "line_number": "001", "sku": "ACE-500", "barcode": "7501234567890", "description": "Aceite vegetal 500 ml", "quantity": 12, "unit_price": 3.25, "subtotal": 39.00 },
    { "line_number": "002", "sku": "JAB-250", "barcode": "7501234567891", "description": "Jabón líquido 250 ml", "quantity": 20, "unit_price": 2.10, "subtotal": 42.00 },
    { "line_number": "003", "sku": "DET-1000", "barcode": "7501234567892", "description": "Detergente 1 L", "quantity": 15, "unit_price": 4.80, "subtotal": 72.00 },
    { "line_number": "004", "sku": "AGU-1500", "barcode": "7501234567893", "description": "Agua mineral 1.5 L", "quantity": 30, "unit_price": 0.95, "subtotal": 28.50 }
  ],
  "subtotal": 181.50,
  "taxes": 0.00,
  "total_amount": 181.50
}
```

El PDF también muestra email, dirección y teléfono del proveedor, además de datos del cliente/almacén receptor. No son necesarios para la relación o conciliación y se omiten del modelo mínimo. El PDF no declara unidad de medida por línea. La precisión de cantidades y su representación quedan sujetas al formato que acuerden los sistemas; no se convierten unidades en esta definición.

## Relación, identidad e idempotencia

1. La factura se identifica por `(supplier.id, invoice_number)` y la recepción por `receipt_reference`. `supplier.name` y `supplier.tax_id` identifican/describen al proveedor, pero la relación usa su ID interno.
2. Se admite una sola factura por recepción y una sola recepción por factura; no se admiten entregas parciales ni agregación de documentos. Ambos documentos pueden llegar en cualquier orden: el documento inicial queda pendiente de su contraparte.
3. Para un mismo identificador de documento, un reenvío con contenido idéntico es idempotente: se reconoce como duplicado y no se reprocesa, no vuelve a generar informes ni incidencias, no vuelve a notificar y no modifica existencias.
4. Si se recibe contenido diferente con un identificador ya registrado, el resultado es conflicto. No se sobrescribe el documento ni se concilia ese contenido. Resolver una reclamación o autorizar una corrección requiere intervención humana.
5. Con factura y recepción relacionadas, se comparan cantidades por SKU. Los SKU presentes solo en un documento se reportan como faltantes o sobrantes. No se consideran precios, subtotales, impuestos ni total.
6. Si no hay diferencias, el resultado es `conciliado`. Si las hay, el proceso determinista registra las incidencias, genera automáticamente el informe de recepción y el informe de reclamación, y notifica al responsable. La generación de informes no requiere aprobación humana.
7. La notificación se considera enviada solo con confirmación del envío. Hasta confirmarla, la incidencia permanece `incidencia_detectada`; confirmada, pasa a `incidencia_notificada`. La misma incidencia no debe originar una notificación duplicada. La identidad/idempotency key de notificación debe ser estable por incidencia y el resultado del envío debe registrarse para hacer efectiva esta regla.
8. Ningún estado o resultado modifica existencias automáticamente. La aprobación humana se reserva para resolver reclamaciones o autorizar correcciones.

## Estados funcionales

| Estado | Significado |
|---|---|
| `pendiente_de_factura` | Recepción registrada sin la factura correspondiente. |
| `pendiente_de_recepcion` | Factura registrada sin la recepción correspondiente. |
| `incidencia_detectada` | Comparación con diferencias; incidencias registradas e informes generados. El envío de notificación aún no está confirmado. |
| `incidencia_notificada` | Se confirmó el envío de la notificación al responsable. La reclamación aún requiere resolución humana. |
| `conciliado` | Factura y recepción únicas relacionadas coinciden en cantidades por SKU. |

La resolución humana de la reclamación y la autorización de una corrección son decisiones/acciones humanas, no nuevos estados definidos en este objetivo. Las reglas de reintento técnico ante un envío fallido deben mantener la clave idempotente para evitar duplicados.

## Matriz de pruebas

Datos base de la factura: proveedor `PROV-001` / Suministros Delta Caribe S.R.L. / `SDC-482901`; factura `F-1540`; recepción `RC-115`; SKU y cantidades ACE-500: 12, JAB-250: 20, DET-1000: 15, AGU-1500: 30.

| Caso | Entrada | Resultado esperado |
|---|---|---|
| Recepción correcta | Factura y recepción con mismos identificadores y cantidades para los cuatro SKU. | `conciliado`; no generar reclamación ni notificación; existencias intactas. |
| Faltante | ACE-500 recibido: 10 en vez de 12. | Registrar faltante de 2; generar automáticamente informe de recepción e informe de reclamación; notificar al responsable. `incidencia_detectada` hasta confirmar envío; luego `incidencia_notificada`. Sin aprobación para generar informes y sin cambio de existencias. |
| Sobrante | JAB-250 recibido: 22 en vez de 20. | Registrar sobrante de 2 y aplicar la misma generación de informes y confirmación de notificación que en faltante. |
| SKU facturado ausente | No aparece AGU-1500 en la recepción. | Registrar faltante de 30; generar ambos informes y notificar según confirmación. |
| SKU recibido ausente en factura | Se agrega SKU `NUEVO-1`, cantidad 1, a la recepción. | Registrar sobrante de 1; generar ambos informes y notificar según confirmación. |
| Factura pendiente | Solo llega la recepción. | `pendiente_de_factura`; no comparar ni generar reclamación hasta que llegue la factura correspondiente. |
| Recepción pendiente | Solo llega la factura. | `pendiente_de_recepcion`; no comparar ni generar reclamación hasta que llegue la recepción correspondiente. |
| Orden de llegada inverso | Llega primero la factura y después la recepción coincidente. | Primero `pendiente_de_recepcion`; luego ejecutar la comparación determinista y terminar `conciliado` o con la incidencia correspondiente. |
| Proveedor distinto | Coincide `invoice_number`, difiere `supplier.id`. | No relacionar documentos; mantener cada uno pendiente de contraparte. |
| Reenvío idéntico | Reenviar exactamente una recepción o factura ya registrada. | Idempotente: reconocer duplicado sin reprocesar, duplicar informe/incidencia/notificación ni modificar existencias. |
| Mismo identificador, contenido distinto | Reutilizar `receipt_reference` o `(supplier.id, invoice_number)` con cualquier campo de contenido distinto. | Conflicto; no sobrescribir ni conciliar. Revisión humana para resolver/autorizar corrección; existencias intactas. |
| Más de una recepción para una factura | Registrar otra `receipt_reference` para la misma pareja `(supplier.id, invoice_number)`. | Conflicto con la cardinalidad uno a uno; no tratarlo como entrega parcial ni agregar cantidades. Requiere resolución humana. |
| Notificación no confirmada | Se detectó incidencia y generaron informes, pero el envío falla o no se confirma. | Mantener `incidencia_detectada`; registrar resultado no confirmado. Reintento usa la misma clave idempotente y no crea una notificación duplicada. |
| Notificación confirmada | El envío de la notificación de la incidencia recibe confirmación. | Pasar a `incidencia_notificada`; un reprocesamiento no envía una segunda notificación para esa incidencia. |
| Datos inválidos | JSON mal formado; falta ID/nombre/NIF, número de factura, referencia de recepción, items o SKU; cantidad no positiva. | Rechazar como inválido; no relacionar, conciliar ni modificar existencias. El formato de respuesta HTTP queda fuera de alcance. |
| Corrección solicitada | Se necesita cambiar un documento ya registrado. | No aceptar como actualización silenciosa: requiere autorización humana; tras autorizar, aplicar la política de corrección que se defina. No modificar existencias automáticamente. |

## Decisiones aún necesarias

- Acordar cómo se asigna/mappea `supplier.id` interno en ambos sistemas y cómo se corrigen diferencias de nombre/NIF sin cambiar la identidad.
- Acordar formato y precisión de cantidades y política de SKU repetido en varias líneas (comparar sumas por SKU o rechazar duplicados de línea).
- Definir destinatario responsable, canal y qué señal técnica constituye confirmación del envío; especificar persistencia de clave idempotente y tratamiento de resultado de envío desconocido.
- Definir plantillas/contenido de informes y datos obligatorios de reclamación, respetando que su generación automática no espera aprobación.
- Definir procedimiento autorizado para resolver una reclamación y aplicar correcciones, incluyendo su efecto posterior sobre la comparación. La decisión no puede mutar existencias automáticamente.
- Definir respuesta HTTP para entradas inválidas y conflictos de idempotencia.

## Verificación y límites

- Se validó la sintaxis de los dos ejemplos JSON y la aritmética de las cuatro líneas contra el PDF.
- Revisión de coherencia: las identidades de proveedor aparecen en ambos documentos; SKU es la clave de comparación; la relación es uno a uno; duplicados idénticos no reprocesan y cambios con igual identidad producen conflicto; diferencias producen incidencias e informes automáticamente; el estado notificado depende de confirmación; los casos mantienen las existencias sin cambios.
- Revisión de alcance: el cambio es documental. No se añaden SQL, workflows, endpoints, integraciones ni lógica de IA.
- `ConvertFrom-Json` sobre los dos bloques JSON: válido. Comprobación aritmética con PowerShell: los subtotales suman USD 181.50. `git diff --cached --check`: limpio (solo avisó que Git puede convertir LF a CRLF en un acceso posterior).
- Revisión final del diff: solo contiene este documento. No hay suite de aplicación para esta documentación y no se ejecutaron pruebas de software.
