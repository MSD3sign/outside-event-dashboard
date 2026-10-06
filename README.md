# Outside Event Dashboard — v2.20

## Qué cambió en esta versión (v2.20)

**Flujo principal Inbound — 4 líneas informativas bajo el Event Chart:**

Debajo del gráfico (Event Chart) del flujo principal de **Inbound** ahora aparecen, con estilo coherente (texto monoespaciado, mismo color/énfasis que el resto de la app):

1. `Inbound (n) + Sold (n) = Total Inbound (n)`
2. `Outbound (n) − Total Inbound (n) = (resultado)`
3. `Difference (n)` — coloreado igual que en la tabla (verde/rojo/gris según el signo)
4. `Total Items not Scanned in Outbound but were scanned in Inbound (n)` — misma lógica del "Total items" agregado en v2.19 para Compare Excel, aplicada ahora al flujo Inbound.

Todo se calcula reutilizando los totales (`totOut`, `totIn`, `totSold`, `totDiff`) que la app ya calcula en `renderCompareTable()` — no se duplicó ninguna lógica de cálculo — y se actualiza en vivo junto con el chart, cada vez que cambian los datos. No se modificó ninguna otra parte de la app (verificado con diff línea por línea contra v2.19).

---

## Qué cambió en v2.19

**Sección Compare Excel — total de unidades bajo el mensaje de Error:**

Debajo del mensaje "⚠️ Error (n) UPCs that were not scanned in Outbound but were scanned in Inbound" ahora aparece una segunda línea con estilo coherente:

```
📦 Total items: X unit(s) scanned in Inbound without an Outbound record
```

Donde **X** es la suma de las cantidades Inbound de esos mismos UPCs (Outbound = 0, Inbound > 0) — es decir, total de **unidades**, no de UPCs distintos. Se calcula y se muestra/oculta junto con el mensaje de error, dentro de la misma función (`renderCmpCompareTable`), así que se recalcula en vivo cada vez que cambian los datos. No se modificó ninguna otra parte de la app (verificado con diff línea por línea contra v2.18).

---

## Qué cambió en v2.18

**Sección Compare Excel — ahora guarda y reabre eventos, igual que Outbound/Inbound:**

1. Se agregaron los campos **Store #**, **Event Name** y **Date** al inicio de la sección, con el mismo estilo (`work-topbar`) que Outbound e Inbound.
2. Se agregó el botón **"💾 Save Event"** junto a "Compare" y "Clear". Guarda las dos listas de UPCs pegadas (Outbound e Inbound, texto completo tal cual se pegó) más los datos del evento.
3. El guardado usa **exactamente el mismo almacenamiento** que Outbound/Inbound (carpeta local / `window.storage` / `localStorage`, mismo archivo `.json` por evento, mismo índice de eventos). El evento guardado desde Compare Excel aparece en el listado general de eventos guardados.
4. Se agregó un selector **"Switch Event"** (igual que en Outbound) para volver a abrir cualquier evento guardado directamente en Compare Excel — repone Store #/Event Name/Date y ambas listas pegadas. Si el evento fue creado originalmente en Outbound/Inbound (sin datos propios de Compare Excel), reconstruye las listas a partir de los UPCs escaneados.
5. El botón "Clear" ahora también limpia los campos del evento y el selector, para evitar sobrescribir por accidente un evento cargado al presionar "Save Event" después de limpiar.

No se modificó nada más de la app.

---

## Qué cambió en v2.17

**Módulo Register Sales (Inbound y Compare Excel):** el campo **"Qty sold"** ahora viene precargado con el valor **1** apenas se selecciona un UPC en el combobox buscable. El usuario puede presionar directamente "✓ Register Sale" sin tener que escribir la cantidad — y sigue pudiendo cambiar el número manualmente si la venta fue de más de 1 unidad. No se tocó nada más de la app.

---

## Qué cambió en v2.16

**Problema resuelto:** en la wifi de la empresa, la tablet abría la cámara y mostraba el UPC, pero nunca lo escaneaba. Causa: el navegador de la tablet no tiene `BarcodeDetector` nativo (típico en Safari/iPadOS), así que la app dependía de cargar la librería **ZXing** desde un CDN (`cdn.jsdelivr.net`) en el momento de abrir la cámara — y el firewall corporativo bloqueaba ese dominio, por lo que ZXing nunca llegaba a cargar.

**Solución v2.16:** la librería ZXing (`@zxing/library` v0.20.0, build UMD `index.min.js`, 336 KB) ahora está **incrustada directamente dentro del `index.html`**, en un bloque `<script>` dentro del `<head>`. Ya no se descarga nada de internet para que el escaneo por cámara funcione — el archivo es 100% autónomo en ese aspecto, sin importar el firewall de la empresa.

### Qué NO cambió (se mantuvo igual que v2.15)
- **Chart.js** y **SheetJS/XLSX** (exportar a Excel) siguen usando la carga en cascada multi-CDN de la v2.15 (prueban varios orígenes en secuencia: `jsdelivr` → `unpkg` → `cdnjs`). No se incrustaron en esta versión porque el pedido específico de esta ronda fue solo ZXing.
- El resto de la app (Outbound, Inbound, Compare Excel, filtros, combobox buscable, guardado en carpeta, etc.) no se tocó.

### Verificación técnica realizada
- El archivo `.js` subido se validó con `node --check` → sintaxis válida.
- Se confirmó que expone `ZXing.BrowserMultiFormatReader` (la clase que usa el código de la app).
- Se verificó que el archivo no contiene la cadena `</script>` literal (que rompería el HTML si no se escapara).
- Se validó la sintaxis completa del HTML resultante (ambos bloques `<script>`: la librería incrustada y la lógica de la app).

### Cómo probar que ya no depende de internet para la cámara
1. Abra `index.html` en una tablet/dispositivo con `BarcodeDetector` no soportado (ej. Safari/iPad).
2. Desconecte el wifi o actívelo en modo avión.
3. Presione el botón 📷 para escanear — la cámara debe abrir y detectar el código igual que con internet.
   (El resto de la UI, como exportar a Excel o ver el gráfico, seguirá necesitando internet mientras esas dos librerías no estén también incrustadas.)

### Tamaño del archivo
El HTML pasó de ~96 KB a **~428 KB** por la librería incrustada. Sigue siendo un solo archivo portátil, sin cambios en cómo se abre o se comparte.

---
*Ver `CHANGELOG.md` del proyecto para el historial completo de versiones.*
