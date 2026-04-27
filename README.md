# banklens_credit_analyzer
Privacy-first Nigerian bank statement analyser. Runs entirely in your browser — no uploads, no server, no data leaves your device. Supports 11 banks including GTBank, Access, Kuda, OPay, Stanbic, TAJBank, Wema/ALAT, and more. Single HTML file, no install needed.

# BankLens

**A privacy-first Nigerian bank statement analyser that runs entirely in your browser.**

No uploads. No servers. No data ever leaves your device. Drop in a PDF, get structured transaction data, monthly summaries, integrity checks, and cashflow insights — all processed locally using PDF.js and ExcelJS.

---

## Features

- **Zero backend** — pure client-side HTML/JS, single file, no dependencies to install
- **12 bank parsers** covering the most common Nigerian statement formats
- **Balance-movement classifier** — derives credit/debit from running balance changes, not column labels, making it robust against formatting inconsistencies
- **Integrity checks** — cross-validates parsed totals against stated cover page figures
- **Monthly summary** — opening balance, total credits, total debits, net flow, and closing balance per month
- **Cashflow insights** — peak inflow/outflow months, average transaction size, flow patterns
- **Excel export** — formatted workbook with dark headers, frozen panes, colour-coded net flow
- **40-page batch processing** with garbage collection to handle large statements without crashing

---

## Supported Banks

| Bank | Format Notes |
|------|-------------|
| OPay | Individual and OWealth accounts; 2nd-header boundary detection |
| AltBank (Lima Square) | DEBIT/CREDIT/BALANCE column layout |
| Stanbic IBTC | Reversed chronology; auto-reversal applied |
| GTBank | Block-based line merging for multi-line narrations |
| Access Bank | Negative balance regex handling |
| Kuda | Standard NIP/mobile transfer format |
| Providus Bank | Standard columnar layout |
| TAJBank | DD-MMM-YY (2-digit year) date format |
| Wema Bank (DIAMONDXTRA) | Legacy Wema columnar format |
| Wema Bank / ALAT | DD-Mon-YYYY stream-order extraction; handles dual PDF encoding |
| Wema Bank ALAT Business | Balance-first column layout; DD-MM-YYYY numeric dates; coordinate-based extraction |
| Generic fallback | Score-based format detection; attempts best-effort parse |

---

## How It Works

1. **PDF extraction** — PDF.js extracts text items with X/Y coordinates. Items within ±2px on the Y-axis are grouped into rows and sorted left-to-right, reconstructing the visual table layout.

2. **Format detection** — The first 6,000 characters of extracted text are matched against bank-specific fingerprints (header labels, account type strings, email domains).

3. **Parsing** — Each bank parser uses a format-appropriate strategy:
   - Most parsers anchor on date patterns and collect amounts per row
   - The balance-movement classifier (`buildTxns`) determines whether each transaction is a credit or debit by comparing consecutive running balances — never relying on which column an amount appears in
   - Special formats (ALAT, ALAT Business) use stream-order extraction or pre-anchor balance strategies where coordinate extraction loses column adjacency

4. **Integrity checks** — Parsed totals are compared against stated cover-page figures. Mismatches above a threshold surface as warnings.

5. **Export** — ExcelJS renders a formatted workbook in-browser; the user downloads it directly.

---

## Usage

No installation. No build step.

```bash
# Clone the repo
git clone https://github.com/your-username/banklens.git

# Open the file directly in any modern browser
open banklens.html
```

Or just download `banklens.html` and open it. That's it.

**Requirements:** A modern browser with JavaScript enabled (Chrome, Firefox, Edge, Safari). PDF.js and ExcelJS are loaded from CDN at runtime.

---

## Privacy

All parsing runs locally on your device using the Web APIs available in any modern browser. The PDF file is read into memory, processed, and discarded — nothing is sent to any server. There are no analytics, no telemetry, and no network requests except the initial CDN load of PDF.js and ExcelJS libraries.

---

## Limitations

- **Scanned PDFs** (image-only, no embedded text) cannot be parsed. The error message will say the PDF may be scanned.
- **Password-protected PDFs** are not supported.
- **Very large statements** (500+ pages) may be slow depending on device memory.
- Parsers are calibrated against specific statement layouts. Banks sometimes change their PDF templates; if a known bank fails, the format may have changed.

---

## Known Edge Cases Handled

- TAJBank uses 2-digit years (`DD-MMM-YY`) — parser normalises to 4-digit UTC dates
- Stanbic statements are in reverse chronological order — parser reverses before computing balance movement
- OPay OWealth accounts embed a second header mid-statement — parser truncates at the second header boundary
- Wema ALAT statements use two different PDF encodings on the same page depending on narration length — parser detects and routes to stream-order extraction
- Wema ALAT Business has the Balance column as the leftmost column, meaning it appears before the narration and reference in PDF stream order — parser uses coordinate-grouped lines directly

---

## Project Structure

```
banklens.html          # The entire application — single self-contained file
README.md              # This file
```

The application is intentionally a single HTML file to maximise portability. There are no build tools, no package.json, and no dependencies to manage.

---

## Development Notes

**Adding a new bank parser:**

1. Add a detection key to `detectFormat()` — match against a unique string in the bank's cover page within the first 6,000 characters
2. Add cover stat extraction to `extractCoverStats()` — extract opening balance, closing balance, stated totals, account name/number
3. Write a `parseXxx(lines, opening)` function — return an array of `{date, balance, narration}` records and pass to `buildTxns(records, opening)`
4. Add a `case 'xxx':` to `dispatch()`
5. Add a display name to `FNAMES`

**The `buildTxns` invariant:** Never modify the balance-movement classifier. It is the single source of truth for whether a transaction is a credit or debit across all parsers. Parsers only need to supply a date, running balance, and narration string — the classifier handles the rest.

---

## Built With

- [PDF.js](https://mozilla.github.io/pdf.js/) — Mozilla's PDF rendering engine, used here for text extraction
- [ExcelJS](https://github.com/exceljs/exceljs) — In-browser Excel workbook generation

---

## License

MIT
