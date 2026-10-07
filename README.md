The app is live here:

https://msd3sign.github.io/outside-event-dashboard/

# Outside Event Dashboard — v2.27

## What changed in this version (v2.27)

**Compare Excel — "Delete all sales" button in Registered Sales:**

- Added a trash icon next to the "Actions" header of the Registered Sales table.
- Pressing it opens the existing "⚠️ Confirm Deletion" modal asking whether to delete all registered sales (Cancel/Delete).
- Confirming clears all sales, refreshes the sales list and the comparison table.
- Reuses the existing delete-modal flow; Inbound not modified.

---

## What changed in this version (v2.26)

- Added outbound confirmation support for Compare Excel Register Sales → Paste list mode.
- Paste list now shows a live warning when unique UPCs are not found in Outbound.
- Saving a Paste list with unknown UPCs asks for confirmation before processing.
- Users can choose to save unknown UPCs anyway or keep them in the textarea for later review.
- Existing validation rules, Allow oversell behavior, comparison rendering, and sales accumulation remain unchanged.

---

## What changed in this version (v2.25)

- Added a segmented Single / Paste list mode selector to REGISTER SALES in the Compare Excel panel.
- Added bulk UPC sales registration through a multiline paste box.
- Paste list mode registers one sale per non-empty UPC line and accumulates into existing registered sales.
- Reuses the existing UPC validation, remaining availability checks, oversell setting, toast notifications, and comparison refresh flow.
- The Allow oversell option remains visible outside the tabs and applies to both registration modes.

---

## What changed in this version (v2.24)

**Inbound and Compare Excel — "Only non-zero Difference" checkbox:**

- Added an **Only non-zero Difference** checkbox to both comparison tables (Inbound and Compare Excel), unchecked by default.
- When checked, the table shows only rows with Difference ≠ 0.
- It combines with the existing filters using the same AND pattern: in Inbound with "Only negative Difference"; in Compare Excel with "Only negative Difference" and "Only Error".
- "Clear filters" also resets the new checkboxes.

---

## What changed in this version (v2.23)

**Compare Excel — "Allow oversell" checkbox in Register Sales:**

- Added an **Allow oversell** checkbox to the Compare Excel Register Sales section, unchecked by default.
- When checked, it bypasses the validation that blocks sales larger than the available quantity (Outbound − Inbound − previously registered sales). Example: with Outbound ×2, a sale of ×3 can now be registered.
- When unchecked, the existing validation works exactly as before.
- The Inbound Register Sales was not modified.

---

## What changed in this version (v2.22)

- Updated Compare Excel Difference error labels to display the quantity first and use green error labels.
- Changed outbound-missing scan errors to display as `N Product not scanned at outbound`.
- Changed over-return errors to display as `N Unscanned difference`.
- Kept shortage differences unchanged: negative Difference values continue to display in red and remain available only through **Only negative Difference**.
- Kept **Only Error** behavior limited to the existing error classification flow and reused the existing UPC difference values.

---

## What changed in this version (v2.21)

**Compare Excel — "Only Error" now includes positive differences (over-return):**

- The **Only Error** filter now also includes POSITIVE differences: UPCs where there was Outbound but Inbound + Sold > Outbound (over-return / excess inventory). Those rows count as errors just like the negative ones.
- The error counter includes them as well.
- **Only negative Difference** still shows only negative differences (shortage).
- The classification reuses the already-computed per-UPC diff — no calculation logic was duplicated and no other part of the app was modified (verified with a line-by-line diff against v2.20).

---

## What changed in this version (v2.20)

**Compare Excel and Inbound — 4 info lines below the Event Chart:**

Below the chart (Event Chart) of the **Compare Excel** section and the main **Inbound** flow, four lines now appear with consistent styling (monospaced text, same color/emphasis as the rest of the app):

1. `Inbound (n) + Sold (n) = Total Inbound (n)`
2. `Outbound (n) − Total Inbound (n) = (result)`
3. `Difference (n)` — colored like in the table (green/red/gray depending on the sign)
4. `Total Items not Scanned in Outbound but were scanned in Inbound (n)` — same logic as the "Total items" added in v2.19, with Compare Excel data.

Everything is computed by reusing the totals (`totOut`, `totIn`, `totSold`, `totDiff`) the app already calculates in `renderCompareTable()` — no calculation logic was duplicated — and updates live together with the chart whenever the data changes. No other part of the app was modified (verified with a line-by-line diff against v2.19).

---

## What changed in v2.19

**Compare Excel section — unit total below the Error message:**

Below the message "⚠️ Error (n) UPCs that were not scanned in Outbound but were scanned in Inbound", a second line now appears with consistent styling:

```
📦 Total items: X unit(s) scanned in Inbound without an Outbound record
```

Where **X** is the sum of the Inbound quantities of those same UPCs (Outbound = 0, Inbound > 0) — that is, a total of **units**, not of distinct UPCs. It is calculated and shown/hidden together with the error message, inside the same function (`renderCmpCompareTable`), so it recalculates live whenever the data changes. No other part of the app was modified (verified with a line-by-line diff against v2.18).

---

## What changed in v2.18

**Compare Excel section — now saves and reopens events, just like Outbound/Inbound:**

1. Added the **Store #**, **Event Name**, and **Date** fields at the top of the section, with the same style (`work-topbar`) as Outbound and Inbound.
2. Added the **"💾 Save Event"** button next to "Compare" and "Clear". It saves both pasted UPC lists (Outbound and Inbound, full text exactly as pasted) plus the event data.
3. Saving uses **exactly the same storage** as Outbound/Inbound (local folder / `window.storage` / `localStorage`, same `.json` file per event, same event index). An event saved from Compare Excel shows up in the general saved-events list.
4. Added a **"Switch Event"** selector (same as in Outbound) to reopen any saved event directly in Compare Excel — it restores Store #/Event Name/Date and both pasted lists. If the event was originally created in Outbound/Inbound (with no Compare Excel data of its own), it rebuilds the lists from the scanned UPCs.
5. The "Clear" button now also clears the event fields and the selector, to avoid accidentally overwriting a loaded event by pressing "Save Event" after clearing.

Nothing else in the app was modified.

---

## What changed in v2.17

**Register Sales module (Inbound and Compare Excel):** the **"Qty sold"** field now comes pre-filled with **1** as soon as a UPC is selected in the searchable combobox. The user can press "✓ Register Sale" directly without typing the quantity — and can still change the number manually if the sale was for more than 1 unit. Nothing else in the app was touched.

---

## What changed in v2.16

**Problem fixed:** on the company wifi, the tablet opened the camera and showed the UPC, but never scanned it. Cause: the tablet's browser has no native `BarcodeDetector` (typical on Safari/iPadOS), so the app depended on loading the **ZXing** library from a CDN (`cdn.jsdelivr.net`) at the moment the camera opened — and the corporate firewall blocked that domain, so ZXing never loaded.

**v2.16 fix:** the ZXing library (`@zxing/library` v0.20.0, UMD build `index.min.js`, 336 KB) is now **embedded directly inside `index.html`**, in a `<script>` block within `<head>`. Nothing is downloaded from the internet for camera scanning to work — the file is 100% self-contained in that regard, regardless of the company firewall.

### What did NOT change (kept as in v2.15)
- **Chart.js** and **SheetJS/XLSX** (Excel export) still use the v2.15 multi-CDN cascade loading (they try several origins in sequence: `jsdelivr` → `unpkg` → `cdnjs`). They were not embedded in this version because this round's specific request was ZXing only.
- The rest of the app (Outbound, Inbound, Compare Excel, filters, searchable combobox, folder saving, etc.) was not touched.

### Technical verification performed
- The uploaded `.js` file was validated with `node --check` → valid syntax.
- Confirmed it exposes `ZXing.BrowserMultiFormatReader` (the class the app's code uses).
- Verified the file does not contain the literal string `</script>` (which would break the HTML if not escaped).
- Validated the full syntax of the resulting HTML (both `<script>` blocks: the embedded library and the app logic).

### How to verify it no longer depends on the internet for the camera
1. Open `index.html` on a tablet/device without `BarcodeDetector` support (e.g., Safari/iPad).
2. Disconnect wifi or turn on airplane mode.
3. Press the 📷 scan button — the camera should open and detect the code just like with internet.
   (The rest of the UI, like Excel export or viewing the chart, will still need internet until those two libraries are embedded as well.)

### File size
The HTML went from ~96 KB to **~428 KB** due to the embedded library. It remains a single portable file, with no changes to how it is opened or shared.

---
*See the project's `CHANGELOG.md` for the full version history.*
