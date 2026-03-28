# ClearPlan CPD Editor

A fully offline, single-file browser tool for editing IPMDAR Contract Performance Dataset (CPD) files. Built for program control professionals who need to review and modify Work Package earned value techniques, Control Account attributes, and other CPD metadata — then re-export a clean `.zip` — without installing software or transmitting data over a network.

---

## 🔒 Security & Data Handling

**This tool runs entirely in your browser. No data ever leaves your machine.**

- **Zero network activity** — JSZip and all application code are embedded directly in the HTML file. No CDN calls, no API requests, no telemetry, no analytics. Once the page loads, you can disconnect from the internet entirely.
- **No server component** — There is no backend. All file parsing, editing, validation, and ZIP generation happen client-side in browser memory.
- **Nothing is stored** — No cookies, no `localStorage`, no `sessionStorage`, no IndexedDB. When you close the tab, everything is gone.
- **Safe for sensitive data** — Designed for use with CUI (Controlled Unclassified Information), ITAR/EAR export-controlled data, proprietary contractor data, and other sensitive program information. Your CPD files are never uploaded, cached, or exposed to any external service.

You can verify this yourself: open the HTML file in a text editor — every line of code is readable, and you will find no external URLs, tracking pixels, or network calls of any kind.

---

## Quick Start

1. **Open** `ClearPlan_CPD_Editor.html` in any modern browser (Chrome, Edge, Firefox, Safari).
2. **Load** a CPD `.zip` file by dragging it onto the upload area or clicking to browse.
3. **Edit** Control Accounts and Work Packages using the tabbed interface.
4. **Validate** your changes against the IPMDAR specification.
5. **Export** an updated `.zip` file with your edits applied.

That's it. No installation, no accounts, no configuration.

---

## Features

### Control Account Editing
- Toggle **Summary-Level Planning Package (SLPP)** designation
- Edit name, manager, and all date fields (baseline, forecast, actual start/end)
- Edit custom field values (FIELD_01 through FIELD_10)
- Filter by WBS element; search by name, ID, or manager
- Sortable columns

### Work Package Editing
- Assign **Earned Value Techniques** from the full IPMDAR enumeration (Milestone, Percent Complete, LOE, 0/100, 50/50, Apportioned Effort, etc.)
- Toggle **Planning Package** designation
- `OtherEarnedValueTechnique` description field auto-appears for OTHER_DISCRETE and FIXED_X_Y
- Edit name, dates, and custom fields
- Filter by Control Account
- **Add new Work Packages** — even to CPD files that don't yet contain a `WorkPackages.json`

### Validation Engine
Checks against the IPMDAR v1.0 specification, including:
- Date consistency (start ≤ end for all date pairs)
- WBS/OBS leaf-node verification for Control Accounts
- Earned Value Technique enum validity
- `OtherEarnedValueTechnique` conditional enforcement (must be set for OTHER_DISCRETE / FIXED_X_Y; must not be set otherwise)
- Planning Package EV Technique warnings
- Duplicate ID detection
- Required field checks
- Custom field definition/value cross-referencing

### Changeset System
Enables repeatable edits across successive CPD deliverables:
- **Save Changeset** — Exports a `.cpd-changeset.json` file capturing every field-level edit you made, keyed by entity ID
- **Import Changeset** — Loads a previously saved changeset against a new CPD file
- **Review & Approve** — Modal diff view showing old → new values, match/mismatch status for every entity ID, and summary statistics
- **Apply** — One-click approval applies all matched changes; unmatched IDs are flagged and skipped

### Startup Diagnostics
On launch, the tool verifies browser capabilities (File API, Blob support, JSZip availability) and displays a checklist. If any requirement isn't met, actionable troubleshooting guidance is shown.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  ClearPlan_CPD_Editor.html               │
│                    (~195 KB, single file)                 │
│                                                          │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────┐  │
│  │  JSZip 3.10  │  │  App CSS     │  │  App JavaScript│  │
│  │  (embedded)  │  │  (embedded)  │  │  (embedded)    │  │
│  │  97 KB       │  │  ~12 KB      │  │  ~85 KB        │  │
│  └──────┬───────┘  └──────────────┘  └───────┬────────┘  │
│         │                                     │          │
│         ▼                                     ▼          │
│  ZIP read/write              Parsing, editing, validation │
│  (client-side)               diffing, changeset engine    │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │              Browser (runtime environment)          │  │
│  │  File API ──▶ ArrayBuffer ──▶ JSZip ──▶ JSON parse │  │
│  │  User edits ──▶ State mutation ──▶ Validation       │  │
│  │  Export ──▶ JSZip generate ──▶ Blob ──▶ Download    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│         ▲                              │                 │
│         │                              ▼                 │
│    ┌────┴─────┐                 ┌──────┴──────┐          │
│    │ CPD .zip │                 │ CPD .zip    │          │
│    │ (input)  │                 │ (output)    │          │
│    └──────────┘                 └─────────────┘          │
│                                                          │
│    ┌──────────────┐           ┌──────────────────┐       │
│    │ Changeset    │◄─────────▶│ Changeset        │       │
│    │ .json import │           │ .json export     │       │
│    └──────────────┘           └──────────────────┘       │
└─────────────────────────────────────────────────────────┘

         Network: NONE (fully air-gapped capable)
```

### Data Flow

1. User drops a CPD `.zip` file onto the page
2. `File.arrayBuffer()` reads the raw bytes into browser memory
3. JSZip parses the ZIP archive and extracts each JSON file
4. JSON files are parsed into JavaScript objects held in application state
5. A deep copy of the original state is retained for change tracking
6. User edits mutate the working state; the UI re-renders from state
7. The validation engine compares state against IPMDAR v1.0 rules
8. On export, JSZip re-serializes all JSON files and generates a new ZIP blob
9. The browser triggers a file download from the blob — no upload occurs

### Changeset Flow

1. The diff engine compares current state against the original snapshot field-by-field
2. Changes are serialized as a structured JSON document keyed by entity IDs
3. On import, each changeset entry is matched against the current CPD by ID
4. Unmatched IDs are flagged; matched changes are previewed in a diff view
5. On approval, changes are applied to the working state in a single pass

---

## IPMDAR Compliance

This tool reads and writes CPD files conforming to the **IPMDAR Contract Performance Dataset Version 1.0** specification (March 2020), including:

- `FileType.txt` — verified on load, written on export as `IPMDAR_CONTRACT_PERFORMANCE_DATASET/1.0`
- All 24 JSON file entries defined in the File Format Specification
- Singleton tables (DatasetConfiguration, DatasetMetadata, SourceSoftwareMetadata, ContractData) preserved as single objects
- Array tables preserved with record ordering
- Null/empty field omission per the JSON conventions in the spec
- Files for empty tables are omitted from the exported ZIP per spec

Files that this tool does not edit (performance data tables like BCWS_ToDate, BCWP_ToDate, ACWP_ToDate, etc.) are passed through byte-for-byte from input to output.

---

## Browser Compatibility

| Browser | Status |
|---------|--------|
| Chrome / Chromium 80+ | ✅ Fully supported |
| Microsoft Edge 80+ | ✅ Fully supported |
| Firefox 78+ | ✅ Fully supported |
| Safari 14+ | ✅ Fully supported |
| Internet Explorer | ❌ Not supported |

The tool includes a `File.arrayBuffer()` polyfill for older browser builds that support FileReader but not the newer ArrayBuffer API.

---

## File Descriptions

| File | Purpose |
|------|---------|
| `ClearPlan_CPD_Editor.html` | The complete tool — open in any browser to use |
| `README.md` | This file |

---

## Distribution

This is a single HTML file. You can distribute it by:
- Emailing it as an attachment
- Hosting it on a SharePoint or internal file share
- Committing it to a Git repository
- Placing it on a USB drive

No build step, no package manager, no dependencies to install. Recipients double-click the file and it works.

---

## License

JSZip is dual-licensed under the [MIT License](https://opensource.org/licenses/MIT) and [GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html).

---

*Built by [ClearPlan, LLC](https://clearplanconsulting.com/) — the absolute go-to company for effective program planning and control solutions.*
