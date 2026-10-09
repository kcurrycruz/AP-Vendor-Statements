# Statement Desk

A browser-based prototype for checking many vendor statement PDFs against BILL records. Built for the Daily Deposit Process / AP Vendor Statements project.

## Scope

Batch PDF uploads; digital text extraction and local OCR for scans; multi-page statements; invoice, credit, payment and balance-forward activity; separate original amount and outstanding balance; exception review, correction, notes and CSV export.

All PDF processing happens in the browser. The hosted application serves static code and OCR assets. It does not upload statement PDFs to a server. The application has no BILL credentials or live connection. Sample matches are explicitly simulated. Actual uploads remain unverified until BILL records are imported. Optional Save session stores extracted records and notes in this browser, not the PDFs; Clear workspace removes that saved session. Data is not shared across users or devices.

## Architecture

- `dist/core.mjs`: statement interpretation, decimal-money handling and matching rules.
- `dist/extraction.mjs`: PDF.js text extraction and Tesseract OCR, producing position-aware lines.
- `dist/app.mjs`: browser UI and review workflow.
- `dist/demo.mjs`: fictional statement and BILL fixtures.
- `scripts/build.mjs`: copy pinned PDF.js and OCR assets into the static build.

Saved vendor name mappings are local to the browser. Six initial layout adapters cover Ace Endico, Adventure in Food, Antonucci's, Argyle, BakeMark and A.J. Rinella. Other layouts use a conservative generic invoice-table adapter; unsupported or incomplete results require manual review. No guarantee is made that these adapters handle every historical version or vendor.

## Run

Install the pinned dependencies with pnpm, run `pnpm build`, `pnpm test`, then `pnpm dev`. The development server prints its local URL. Node 22 or newer is recommended.

## BILL integration boundary

The CSV record provider is a temporary input adapter. A future server-side BILL v3 provider should return the same normalized records: `vendor`, `vendorId`, `invoiceNumber`, `amount`, `currency`, `status`, `clearedPaymentAmount`, `scheduledAmount`, `creditAmount`, `id`. Keep production developer keys and authentication credentials exclusively on the server. Only retrieve data; no bill creation, payment initiation, or accounting writes are in scope.

Live matching must paginate complete results, include historical paid records and older outstanding items, retain data coverage/refresh timestamps, and inspect full payment lifecycle and credit applications. BILL's dueAmount subtracts scheduled payments; do not treat a scheduled payment as settled. Statement-date balances must be reconstructed from dated activity when history permits, otherwise mark them unverified. Matching in BILL does not verify QuickBooks synchronization.

## Validation

Domain tests cover leading zeros and suffixes, paid and scheduled payments, credits and repeated invoice activity, duplicates, missing records versus unavailable data, statement totals and carried-forward balances. Synthetic BILL fixtures demonstrate outcomes but do not validate actual vendor records. Browser QA and supplied PDFs must additionally be tested before operational use.
