# Outside Event Dashboard

**Portfolio project by Miguel Salazar · MSD3sign** · [LinkedIn](https://www.linkedin.com/in/miguel-salazar-416703243/)

Web app for reconciling merchandise sent to off-site events (Outbound) and returned from them (Inbound): UPC scanning, sales registration, inventory reconciliation, inconsistency detection, charts, and Excel export.

**Live demo:** https://msd3sign.github.io/outside-event-dashboard/

## What it does

- **Outbound** — scan UPCs (USB scanner, manual entry, or phone camera) for merchandise leaving to an event; numbered scan list, per-UPC summary, save event, export to Excel.
- **Inbound** — pick the saved Outbound event, scan returned UPCs with over-scan protection, register/edit/delete manual sales with availability validation.
- **Compare Excel** — standalone flow: paste two UPC lists copied from Excel (one per line, duplicates kept) and get the full comparison without needing a saved event.
- **Reconciliation table** — UPC / Outbound / Inbound / Sold / Difference, where `Difference = (Inbound + Sold) − Outbound`. Rows with `Outbound = 0` but Inbound/Sold recorded are flagged as `Error (n)`, with an error counter and view-only filters (text search, negative difference, errors only).
- **Persistence** — save each event as its own `.json` file in a user-chosen local folder (File System Access API, shareable via OneDrive/Drive/Dropbox), with `localStorage` fallback.
- **Camera scanning** — hybrid detection: native `BarcodeDetector` first, lazy-loaded ZXing from CDN as fallback.

## Tech

Single-file HTML/CSS/vanilla JavaScript — no build step, no framework. Chart.js for charts, SheetJS for Excel export (CDN).

## Try it

Open the live demo, go to **🔀 Compare Excel**, and press **📥 Load sample data** — it fills both lists with fictional UPCs (including one inconsistent row to showcase the error detection) and runs the comparison.

## Version history

See the full changelog in [CHANGELOG.md](CHANGELOG.md). Current version: **v2.16**.
