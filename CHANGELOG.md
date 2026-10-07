# Outside Event — Control de Versiones

Historial de cambios y mejoras de la aplicación de escaneo de UPCs para eventos externos (Outbound / Inbound).

---## v2.22
**Compare Excel: etiquetas de error en verde con el número primero**

- La columna Difference de la tabla de Compare Excel ahora muestra las filas de error en verde y con el número primero:
  - `N Product not scanned at outbound` cuando Outbound = 0 y hubo retorno o venta (UPC no escaneado en Outbound).
  - `N Unscanned difference` cuando hubo Outbound pero regresó más mercancía (over-return: Inbound + Sold > Outbound).
- Las diferencias negativas (shortage) siguen en rojo como antes.
- Para el total de Difference, el valor de Error(n) se trata como positivo y se suma (comportamiento actual conservado).
- `Only Error` muestra ambos tipos de error; las negativas no entran en `Only Error` (siguen solo en `Only negative Difference`).
- El cambio reutiliza el `diff`/`isError` por UPC ya calculado (`renderCmpCompareTable`); diff línea por línea contra v2.21: solo el label de versión y las 2 líneas de `diffClass`/`diffLabel` en Compare Excel.## v2.24
**Inbound y Compare Excel: checkbox "Only non-zero Difference"**

- Nuevo checkbox **Only non-zero Difference** en ambas tablas de comparación (Inbound y Compare Excel), desmarcado por defecto.
- Al marcarlo, la tabla muestra solo las filas con Difference distinto de cero.
- Se combina con los filtros existentes con el mismo patrón AND: en Inbound con "Only negative Difference"; en Compare Excel con "Only negative Difference" y "Only Error".
- "Clear filters" también reinicia los nuevos checkboxes.
- Cambio mínimo verificado con diff contra v2.23: solo el label de versión, los 2 checkboxes, el filtro `r.diff !== 0` en ambas tablas, el mensaje de filtros activos y el reset.## v2.26
**Compare Excel: confirmación para UPCs fuera del Outbound en modo "Paste list"**

- Nueva línea informativa fija bajo el textarea que se actualiza al escribir: muestra cuántos UPCs únicos de la lista no están en el Outbound.
- Al dar "Save sales" con UPCs desconocidos, pide confirmación: "¿guardarlos también?".
- Si se dice NO: se guardan los válidos y el textarea queda solo con los desconocidos (para revisarlos).
- Si se dice SÍ: se guarda todo sin restricción y el textarea se limpia por completo.
- Los desconocidos guardados aparecen en la comparación como Outbound=0 con la etiqueta "N Product not scanned at outbound".
- Solo Compare Excel; Inbound intacto.

## v2.25
**Compare Excel: modo "Paste list" en REGISTER SALES**

- Nuevo control de tabs segmentado en REGISTER SALES de Compare Excel: "Single" (por defecto) y "Paste list".
- El tab "Paste list" tiene un textarea ("Paste UPC list, one per line…") y botón "Save sales": cada línea no vacía cuenta como 1 venta del UPC.
- "Save sales" acumula en las ventas ya registradas (suma, no reemplaza), reutiliza las validaciones existentes (UPC válido, cantidad disponible y "Allow oversell"), actualiza la tabla, limpia el textarea y muestra un toast resumen.
- El checkbox "Allow oversell" quedó fuera de los tabs y aplica a ambos modos.
- Corrección propia: el split de líneas venía con regex escapada (`/\\r?\\n/`); se corrigió a `/\r?\n/`.
- Corrección propia (prueba en vivo): con `Allow oversell` marcado, los UPC con disponible=0 se contaban como inválidos porque `cmpAvailableUpcs` los excluía; ahora con oversell activo la validación acepta cualquier UPC del Outbound.
- Solo Compare Excel; Inbound intacto.

## v2.23
**Compare Excel: checkbox "Allow oversell" en Register Sales**

- Nuevo checkbox **Allow oversell** en el Register Sales de Compare Excel, desmarcado por defecto.
- Al marcarlo, se elimina la validación que impide vender más que la cantidad disponible (Outbound − Inbound − ventas ya registradas). Ejemplo: con Outbound ×2 se puede registrar una venta de ×3.
- Con el checkbox desmarcado, la validación actual sigue funcionando igual que siempre.
- Solo aplica a Compare Excel; el Register Sales de Inbound no se tocó.
- Cambio mínimo verificado con diff contra v2.22: solo el label de versión, el checkbox y la condición en `addCmpSoldManual()`.

## v2.21
**Compare Excel: "Only Error" incluye diferencias positivas (over-return)**

- El filtro **Only Error** ahora también incluye las diferencias POSITIVAS: UPCs donde hubo Outbound pero Inbound + Sold > Outbound (mercancía de más / over-return). Esas filas cuentan como error igual que las negativas.
- El contador de errores también las incluye.
- **Only negative Difference** sigue mostrando solo diferencias negativas (shortage).
- El cambio reutiliza el `diff` por UPC ya calculado (`renderCmpCompareTable`); no se duplicó lógica ni se tocó ninguna otra parte de la app (diff línea por línea contra v2.20: solo el label de versión y la clasificación `isError`).

## v2.20
**Fórmulas informativas debajo del Event Chart — en Inbound y en Compare Excel**

- Debajo del **Event Chart** de la sección **Compare Excel** y del flujo principal de **Inbound** se muestran las mismas 4 líneas, con estilo coherente (`.chart-formulas`, texto monoespaciado):
  1. `Inbound (n) + Sold (n) = Total Inbound (n)`
  2. `Outbound (n) − Total Inbound (n) = (resultado)`
  3. `Difference (n)` — coloreado con las mismas clases `diff-pos`/`diff-neg`/`diff-zero` ya usadas en la tabla.
  4. `Total Items not Scanned in Outbound but were scanned in Inbound (n)` — suma de cantidades Inbound de los UPCs con Outbound=0 e Inbound>0.
- En Compare Excel usan los datos de `renderCmpCompareTable()` (`cmpOutboundSummary`/`cmpInboundSummary`/`cmpSoldManual`); en Inbound usan los de `renderCompareTable()` (`outboundList`/`inboundList`/`soldManual`). No se duplicó lógica de cálculo.
- Se actualizan en vivo junto con cada chart, dentro de la misma función que ya lo dibuja.
- No se modificó ninguna otra parte de la app.

## v2.19
**Compare Excel: total de unidades bajo el mensaje de Error**

- Debajo del mensaje "⚠️ Error (n) UPCs that were not scanned in Outbound but were scanned in Inbound" se agregó una segunda línea, con el mismo estilo (`error-count-row`): `📦 Total items: X unit(s) scanned in Inbound without an Outbound record`.
- **X** = suma de las cantidades Inbound de los mismos UPCs que cuenta ese Error (Outbound = 0, Inbound > 0) — total de **unidades**, no de UPCs distintos.
- Se calcula dentro de la misma función (`renderCmpCompareTable`) que ya arma el mensaje de Error (n), acumulando `inn` sobre las filas marcadas como error en el mismo `forEach` — se recalcula en vivo junto con el resto de la tabla.
- Se muestra/oculta en conjunto con el mensaje de Error (n): visible solo cuando `errorCount > 0`.
- No se modificó ninguna otra parte de la app — verificado con diff línea por línea contra v2.18 (9 líneas de diferencia en total: 2 por el bump de versión, 7 por esta mejora).

## v2.18
**Compare Excel: identificación de evento + guardar/reabrir**

- Se agregaron los campos **Store #**, **Event Name** y **Date** a la sección **Compare Excel**, con el mismo estilo (`work-topbar`) que Outbound e Inbound.
- Se agregó el botón **"💾 Save Event"**, que guarda las dos listas de UPCs pegadas (texto completo, tal cual) junto con los datos del evento.
- El guardado usa el **mismo almacenamiento y el mismo listado de eventos** que Outbound/Inbound (carpeta local / `window.storage` / `localStorage`, un archivo `.json` por evento) — el evento creado desde Compare Excel aparece en el índice general de eventos guardados.
- Se agregó el selector **"Switch Event"** (mismo patrón que Outbound) dentro de la propia sección Compare Excel, para reabrir cualquier evento guardado: repone Store #/Event Name/Date y ambas listas pegadas (`cmpOutboundText`/`cmpInboundText`, nuevos campos del modelo de datos). Si el evento fue creado en Outbound/Inbound y no tiene esos campos propios, reconstruye las listas a partir de `outboundList`/`inboundList` (un UPC por línea).
- Las ventas registradas en Compare Excel se guardan aparte (`cmpSoldManual`), sin tocar el `soldManual` que usa Inbound — evita que guardar desde un módulo pise los datos del otro si comparten el mismo evento.
- "Clear" ahora también limpia los campos del evento y el selector, para no sobrescribir por accidente un evento cargado.
- No se modificó ninguna otra parte de la app (se verificó con diff línea por línea contra v2.17).

## v2.17
**Qty sold por defecto = 1 en Register Sales**

- En el módulo **Register Sales**, tanto en **Inbound** como en **Compare Excel**, el campo "Qty sold" ahora se precarga con el valor **1** automáticamente cada vez que se selecciona un UPC en el combobox buscable (clic en una opción del dropdown, o Enter sobre un UPC exacto).
- El usuario puede presionar "✓ Register Sale" directamente sin escribir nada, o cambiar el número manualmente si la venta fue de una cantidad distinta a 1.
- No se modificó ninguna otra parte de la app.

## v2.16
**ZXing incrustado — escaneo por cámara 100% offline**

- Se recibió el archivo real de **@zxing/library v0.20.0** (build UMD `index.min.js`, 336 KB) y se verificó íntegro (`node --check` pasó, expone `ZXing.BrowserMultiFormatReader`, sin `</script>` literal que rompiera el HTML).
- La librería se **incrustó directamente dentro del `<head>`** del `index.html`, en su propio bloque `<script>`, eliminando por completo la dependencia de `cdn.jsdelivr.net` (o cualquier CDN) para el fallback de escaneo por cámara en navegadores sin `BarcodeDetector` nativo (Safari/iPad).
- `loadZxing()` ahora solo verifica que `window.ZXing` ya esté definido (lo está, de forma síncrona, apenas carga la página) — se eliminó el arreglo `ZXING_CDNS` y la lógica de cascada específica para ZXing, que quedó obsoleta.
- **Chart.js y SheetJS/XLSX siguen igual que en v2.15** (carga en cascada multi-CDN) — no se incrustaron en esta versión, ya que el pedido fue específico para ZXing.
- El archivo pasó de ~96 KB a ~428 KB por la librería incrustada; sigue siendo un único `.html` portátil.
- Entregado como `index.html` + `README.md` (documentación de este cambio específico).

## v2.15
**Carga resiliente multi-CDN (sin bloqueo por firewall de empresa)**

- Se detectó que en la wifi de la empresa la cámara abría y detectaba visualmente el UPC, pero nunca lo escaneaba: el firewall corporativo bloqueaba `cdn.jsdelivr.net`, el único origen del que se cargaba la librería ZXing (fallback para navegadores sin `BarcodeDetector` nativo, como Safari/iPad) y también Chart.js.
- Se evaluó incrustar la librería ZXing directamente en el `.html` (100% offline), pero no fue posible traerla de forma confiable y verificable con las herramientas disponibles en este entorno (sin acceso a red en la terminal, y la herramienta de fetch está pensada para extraer texto de páginas, no para descargar archivos binarios/JS grandes byte a byte) — incrustar una copia potencialmente corrupta habría sido peor que el problema original.
- En su lugar se implementó **carga en cascada multi-CDN**: ZXing, Chart.js y SheetJS (motor de exportación a Excel) ahora intentan cargarse desde varios orígenes distintos en secuencia (`cdn.jsdelivr.net` → `unpkg.com` → `cdnjs.cloudflare.com`, u orden equivalente) — si el firewall bloquea uno, prueba automáticamente el siguiente, sin intervención del usuario.
- Se quitaron los `<script src>` bloqueantes de XLSX y Chart.js del `<head>`; ahora cargan de forma perezosa y en segundo plano apenas abre la app, sin retrasar el uso normal (escanear y guardar nunca dependen de estas librerías).
- Si las tres fuentes fallan: el gráfico muestra un mensaje de "cargando…" y se dibuja solo automáticamente en cuanto la librería esté disponible (sin recargar la página); la exportación a Excel muestra un error claro explicando qué dominios pedir a IT que desbloqueen, en vez de fallar en silencio.
- El escaneo por USB y el botón manual "+ Add" nunca dependieron de internet y siguen funcionando igual sin importar el estado de estas librerías.

## v2.14
**Filtro "Only Error" en Compare Excel**

- Se agregó un checkbox **"Only Error"** en la barra de filtros de la tabla "Outbound / Inbound Comparison" del módulo **Compare Excel**, junto a "Only negative Difference".
- Al marcarlo, la tabla muestra únicamente las filas marcadas como `Error (n)` (UPCs con Outbound = 0 pero con Inbound o Sold registrado).
- Es combinable con el filtro de texto por UPC y con "Only negative Difference" al mismo tiempo.
- Es un filtro solo de vista: los totales, el contador de errores y el gráfico siguen reflejando siempre el conjunto completo de datos, no la vista filtrada.
- Se limpia junto con los demás filtros al presionar "Clear filters" o al correr un nuevo "Compare"/"Clear".
- No se modificó el módulo **Inbound** en esta versión.

## v2.13
**Filtros de tabla reincorporados en Inbound**

- Se agregó de nuevo, en el panel **Inbound**, la barra de filtros sobre la tabla "Outbound / Inbound Comparison": campo de texto para filtrar por UPC (completo o parcial), checkbox "Only negative Difference" y botón "Clear filters" — igual a la que ya existía en Compare Excel.
- Los filtros se resetean automáticamente al cambiar de evento.
- No se modificó ninguna otra parte del panel de Inbound.

## v2.12
**Revert de filtros de tabla en Inbound**

- Se revirtió en el panel **Inbound** la barra de filtros de la tabla comparativa (campo de texto por UPC y "Only negative Difference") introducida por error en la v2.11, dejando el panel igual que en la v2.10.
- Se conservó únicamente el combobox buscable de UPC en "Register Sales", que era el único cambio solicitado para Inbound.
- El módulo **Compare Excel** no se modificó y conserva ambos filtros.

## v2.11
**Búsqueda tipo-ahead en Register Sales + filtros en la tabla comparativa**

- El selector de UPC del bloque **"Register Sales"** (Inbound y Compare Excel) se reemplazó por un **combobox buscable**: al escribir cualquier parte de un UPC, la lista desplegable se filtra en vivo mostrando solo los UPCs disponibles que lo contienen, junto con su cantidad disponible — resuelve el problema de listas muy largas donde el `<select>` nativo era difícil de recorrer.
- Se agregó una barra de filtros sobre la tabla "Outbound / Inbound Comparison" (Inbound y Compare Excel) con: campo de texto para filtrar filas por UPC (completo o parcial) y checkbox "Only negative Difference" para ver solo filas con Diferencia negativa.
- Ambos filtros son solo de vista: los totales, el contador de errores y el gráfico siempre reflejan el conjunto completo de datos del evento, nunca la vista filtrada.
- Se agregó indicador "showing X/Y" en el contador de la tabla cuando hay un filtro activo, y un mensaje de "sin resultados" dentro de la tabla si el filtro no coincide con nada.

## v2.10
**Contador de errores en la tabla comparativa**

- Se agregó un contador al final del panel "Outbound / Inbound Comparison", tanto en **Inbound** como en **Compare Excel**, que muestra: "⚠️ Error (n) UPCs that were not scanned in Outbound but were scanned in Inbound" — donde `n` es la cantidad de UPCs con datos inconsistentes (Outbound = 0 pero con Inbound o Vendidos registrados).
- Solo se muestra cuando hay al menos un error; permanece oculto si todos los datos son consistentes.
- Se corrigió otro texto suelto en español ("vendidos") que había quedado sin traducir en el JS desde la v2.7.

## v2.9
**Nuevo módulo: Compare Excel Data**

- Se agregó un tercer flujo en la pantalla principal, junto a Outbound e Inbound: **"🔀 Compare Excel"**.
- Dos áreas de texto ("Outbound" e "Inbound") donde se pega directamente la lista completa de UPCs copiada de Excel (una por línea, con duplicados incluidos); un botón "Compare" procesa ambas listas y calcula el conteo por UPC.
- Incluye el mismo panel de "Register Sales" / "Registered Sales" (editable y eliminable) que existe en Inbound, con las mismas validaciones de disponibilidad.
- Muestra la misma tabla "Outbound / Inbound Comparison" (con la fórmula `(Inbound + Sold) − Outbound` y el formato "Error (n)" para datos inconsistentes) y el mismo "Event Chart" de Inbound.
- A diferencia de Inbound, este módulo **no tiene escaneo ni "Scanned UPC List"** — es una herramienta independiente basada 100% en pegar datos ya exportados de Excel, sin depender de un evento guardado.
- Se corrigió un texto suelto en español ("0 vendidos") que quedó sin traducir en la v2.7.

## v2.8
**Escaneo de UPC con la cámara del celular**

- Se agregó un botón "📷" junto al campo de escaneo, tanto en Outbound como en Inbound.
- Al presionarlo, se abre un modal con la cámara trasera del celular y un recuadro guía para apuntar al código de barras.
- **Detección híbrida**: usa primero la API nativa `BarcodeDetector` (Chrome/Android, sin descargar nada extra); si el navegador no la soporta (ej. iPhone/Safari), descarga automáticamente la librería ZXing desde un CDN solo en ese momento (carga perezosa, cero peso si no hace falta).
- Cada código detectado pasa por la misma función `commitScan()` que ya usan el escáner USB y el botón manual — por lo tanto conserva automáticamente todas las validaciones existentes (UPC no registrado en el evento, límites de disponibilidad para Inbound/Ventas, anti-duplicado).
- El modal permite escaneo continuo (varios productos seguidos) y apaga la cámara automáticamente al cerrarlo o al cambiar de pantalla.
- Se corrigió el botón "+Agregar", que había quedado sin traducir en la v2.7 — ahora dice "+ Add".

## v2.7
**Traducción al inglés + validaciones de integridad de datos**

- Se tradujeron al inglés todos los títulos, etiquetas, botones, placeholders, mensajes de error/confirmación, encabezados de tabla, etiquetas del gráfico y del Excel exportado. El nombre "Outside Event" se mantuvo sin traducir.
- El número de versión se movió a la misma línea del título, justo después de la palabra "Event" (antes aparecía en una posición distinta).
- **Nueva fórmula de "Diferencia"**: ahora es `(Inbound + Sold) − Outbound`, en vez de `Outbound − Inbound − Sold`. Un producto que salió y no regresó ni se vendió ahora se muestra correctamente como una pérdida en **rojo (negativo)**, en vez de en verde.
- **Detección de datos inconsistentes**: si un UPC tiene 0 en Outbound pero existen registros de Inbound o Ventas para él (dato huérfano/erróneo), la tabla comparativa lo muestra como **"Error (n)"** en rojo en vez de un número normal.
- **Nueva validación en Outbound**: ya no se puede eliminar un UPC del listado de Outbound si ese UPC ya tiene unidades registradas en Inbound o en Ventas del evento guardado. Se muestra un mensaje pidiendo eliminar primero esas entradas en Inbound.

## v2.6
**Guardado en carpeta local + módulo de ventas editable + identidad de marca**

- **Guardado en carpeta (File System Access API)**: botón "Conectar Carpeta" que permite elegir una carpeta local (o sincronizada vía OneDrive/Google Drive/Dropbox/red compartida). Cada evento se guarda como su propio archivo `.json`, nombrado `{Store#} - {Nombre del Evento}.json`.
  - El listado de "Eventos guardados" se arma leyendo en vivo los archivos de la carpeta — así, si otra computadora conecta la misma carpeta, ve todos los eventos automáticamente.
  - Si se renombra un evento, el archivo se renombra en disco (sin duplicados).
  - Si el navegador no soporta esta función (Firefox/Safari), se usa `localStorage` como respaldo automático.
  - La carpeta elegida se recuerda entre sesiones (vía IndexedDB); requiere un clic de "Reconectar" por seguridad del navegador.
- **Módulo "Ventas Registradas"**: listado editable de cada venta manual registrada por UPC, con:
  - ✏️ Editar cantidad en línea (con validación de máximo permitido).
  - 🗑 Eliminar con confirmación.
- Se agregó el número de versión visible junto al título de la app.
- Se agregó **footer**: año de creación (2026), crédito al creador (Miguel Salazar) y enlace "Para sugerencias" que abre el correo.

## v2.5
**Corrección de IDs de eventos duplicados**

- Se detectó que dos eventos distintos con el mismo Store # y misma Fecha (pero distinto nombre) generaban el mismo ID interno, causando que el segundo evento **sobrescribiera** al primero en el listado.
- Se corrigió generando un ID único para cada evento nuevo (con sufijo automático si hay colisión), y distinguiendo claramente entre "estoy creando un evento nuevo" vs. "estoy editando uno existente".

## v2.4
**Compatibilidad fuera de Claude.ai**

- Se detectó que el guardado fallaba ("Error guardando el evento") al abrir el archivo `.html` directamente en Chrome, porque `window.storage` solo existe dentro del entorno de artifacts de Claude.ai.
- Se agregó una capa de compatibilidad: si `window.storage` no está disponible, la app usa automáticamente `localStorage` del navegador como respaldo, sin que el usuario tenga que hacer nada distinto.

## v2.3
**Validación de sobre-escaneo en Inbound**

- Se bloqueó la posibilidad de escanear más unidades de un UPC en Inbound de las que realmente salieron en el Outbound del evento (evita que la Diferencia baje a valores negativos por error de escaneo duplicado).
- Mensaje claro indicando cuántas unidades ya se registraron y que no se puede escanear más.

## v2.2
**Validación de disponibilidad para "Vendidos"**

- El selector de "Cantidad Vendida" ahora solo muestra UPCs que **todavía tienen unidades disponibles** (Outbound − Inbound − Vendido > 0). Un UPC que ya salió y regresó completo (ej. salió 1, regresó 1) desaparece de la lista, evitando ventas registradas por error.
- Si se intenta registrar una venta mayor a la disponible, se bloquea con un mensaje indicando el máximo permitido.

## v2.1
**Correcciones críticas de escaneo en Inbound**

- **Bug de concatenación de UPCs**: al escanear dos códigos seguidos, el segundo se pegaba al primero formando un número incorrecto. Se corrigió con un mecanismo de auto-confirmación por temporización (igual que presionar "+Agregar"), sin depender de que el escáner envíe "Enter".
- **Validación de UPC no registrado**: si se escanea un UPC que no salió en el Outbound del evento, se rechaza con un mensaje de error y no se agrega a ninguna lista.
- Se agregó el **listado de UPCs escaneados en Inbound** (numerado, con eliminación y confirmación), igual que ya existía en Outbound — antes solo se veía el resumen comparativo, sin el detalle UPC por UPC.
- Se corrigió un error de consola (`classList.add` con un token inválido) que rompía el efecto visual de "flash" al escanear.

## v2.0
**Reestructuración completa: Dashboard con flujo Main → Outbound → Inbound**

Reemplazo de los prototipos iniciales (`index.html`, `OutsideEvent (claude).html`) por una nueva arquitectura de 3 pantallas:

- **Main Page (kiosco)**: selector de flujo (Outbound/Inbound), opción de Evento Nuevo o Evento Existente, listado de eventos guardados, botón Start.
- **Outbound**: Store #, Nombre del Evento, Fecha; escaneo automático con lista numerada y borrado con confirmación; resumen por UPC; guardar evento; exportar a Excel.
- **Inbound**: filtro por Store #, selección del evento Outbound correspondiente, tabla comparativa **UPC / Outbound / Inbound / Vendidos / Diferencia**, bloque para registrar ventas manuales, gráfico de totales, guardar y exportar a Excel.
- Diseño visual propio: tema oscuro con motivo de "código de barras", tipografía Space Grotesk/IBM Plex Mono, colores diferenciados por flujo (ámbar = Outbound, cian = Inbound).

## v1.0
**Idea original del proyecto**

- Documento inicial (`Proyecto Outside Event.docx`) con los requerimientos base: capturar Store #, escanear UPCs con lector USB, listado automático con opción de borrar duplicados, resumen por UPC, y guardado en Excel con hojas separadas "Outbound UPC's" / lógica de Inbound comparando lo que salió vs. lo que regresó.
- Primeros prototipos funcionales en HTML (`index.html`, `OutsideEvent (claude).html`) usando lectura/escritura directa de archivos Excel (SheetJS) como mecanismo de guardado.

---

*Este documento se actualizará con cada nueva versión de Outside Event.*
