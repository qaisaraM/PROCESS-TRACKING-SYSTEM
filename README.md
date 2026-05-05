<div align="center">

<br/>

```
██████╗ ███████╗    ███╗   ███╗ █████╗  ██████╗██╗  ██╗██╗███╗   ██╗███████╗
██╔══██╗██╔════╝    ████╗ ████║██╔══██╗██╔════╝██║  ██║██║████╗  ██║██╔════╝
██████╔╝█████╗      ██╔████╔██║███████║██║     ███████║██║██╔██╗ ██║█████╗  
██╔═══╝ ██╔══╝      ██║╚██╔╝██║██╔══██║██║     ██╔══██║██║██║╚██╗██║██╔══╝  
██║     ███████╗    ██║ ╚═╝ ██║██║  ██║╚██████╗██║  ██║██║██║ ╚████║███████╗
╚═╝     ╚══════╝    ╚═╝     ╚═╝╚═╝  ╚═╝ ╚═════╝╚═╝  ╚═╝╚═╝╚═╝  ╚═══╝╚══════╝
                        T R A C K I N G   S Y S T E M
```

<br/>

**A production-grade desktop application for real-time CNC machining shop-floor management**

<br/>

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)
![PySide6](https://img.shields.io/badge/PySide6-Qt_Framework-41CD52?style=flat-square&logo=qt&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-QR_Scanning-5C3EE8?style=flat-square&logo=opencv&logoColor=white)
![Excel](https://img.shields.io/badge/openpyxl-Live_Data-217346?style=flat-square&logo=microsoftexcel&logoColor=white)
![ReportLab](https://img.shields.io/badge/ReportLab-PDF_Gen-E34F26?style=flat-square)
![Status](https://img.shields.io/badge/Status-Production-success?style=flat-square)

<br/>

</div>

---

## What This Is

A shop-floor WIP (Work-In-Progress) tracking system built for a precision machining facility. Operators scan QR codes at each machine station — the system automatically timestamps jobs, routes them to the next process, and gives planners a live bird's-eye view of every job on the floor.

No cloud. No browser. A self-contained desktop app that writes directly to a shared Excel workbook on a network drive.

---

## The Problem It Solves

In a CNC machining shop handling dozens of concurrent jobs across 8 process stations (Milling → Grinding → Lathe → EWAG → Levin → NC → R04 → QC), tracking WIP manually means:

- Paper travelers that get lost or misfiled
- No visibility into bottlenecks until it's too late
- Supervisors walking the floor to find job status
- No historical data for capacity planning

This system replaces all of that with a single scan.

---

## Core Features

### QR-Driven Job Routing
Operators scan a job's QR code at their station. The system detects whether this is a **start scan** (first touch) or **end scan** (job complete at this station) and acts accordingly — no mode-switching, no buttons to remember.

```
SCAN #1  →  Record Time Start, prompt operator for Machine No & PIC
SCAN #2  →  Auto-read Machine/PIC from DB, write Time End, route to next station
```

### Live WIP Board
The main screen shows every active job slotted into the column for its current process. Jobs awaiting operator pickup are highlighted in amber. Planners see the full floor at a glance.

```
┌─────────┬─────────┬──────────┬──────┬───────┬───────┬──────┬────┬──────────┐
│ Pending │ Milling │ Grinding │ R04  │ Lathe │ Levin │ EWAG │ NC │ QC & Assy│
├─────────┼─────────┼──────────┼──────┼───────┼───────┼──────┼────┼──────────┤
│ WSR-041 │ WSR-038 │ WSR-035  │      │WSR-029│       │      │    │ WSR-021  │
│ WSR-042 │         │ WSR-036  │      │       │WSR-031│      │    │          │
│ WSR-043 │         │          │      │       │       │      │    │          │
└─────────┴─────────┴──────────┴──────┴───────┴───────┴──────┴────┴──────────┘
                  ░ amber = waiting for operator
```

### Stats Panel
Sidebar shows Completed / In-Progress / Idle counts per station, filterable by Today / This Week / This Month / Custom Range. Double-click any station to drill down to job-level detail.

### Travel Sheet PDF
Generate a landscape A5 PDF travel document per job: QR code, process checklist with sign-off boxes, job info fields, printed-by metadata. Built with ReportLab with precise layout mirroring the UI.

### Planning Module
Password-protected planning page for supervisors to create new job routings — define WSR number, drawing, quantity, and sequence of machining steps.

---

## Architecture

```
pe-machine-tracking/
│
├── main.py                    # Entry point
│
├── ui/
│   ├── ui.py                  # App wiring — builds all pages, shared `ui` dict
│   ├── data_layer.py          # All Excel I/O lives here (openpyxl)
│   ├── process_config.py      # Single source of truth for all 8 processes
│   ├── process_page.py        # Generic process page factory (used by all 8 stations)
│   │
│   ├── ui_page_0.py           # Main WIP board + stats panel
│   ├── ui_page_2.py           # Travel Sheet + PDF generation
│   ├── ui_page_3.py           # Planning / job creation
│   └── ui_page_4.py           # View-all history
│
├── assets/
│   ├── fonts/
│   └── icons/
│
├── qr_codes/                  # QR image cache (auto-generated)
├── pdf_output/                # Generated travel sheets
└── DATA/
    └── DATA.xlsx              # Live data store (shared network drive)
```

### Key Design Decisions

**Single Excel file as database** — The facility already uses Excel for planning. Rather than introducing a separate DB, the app reads/writes directly to `DATA.xlsx` on the shared drive. `data_layer.py` is the only file that touches openpyxl — all pages go through it.

**Config-driven process pages** — All 8 machining stations share identical UI logic. Rather than 8 near-identical files, `process_config.py` defines a `ProcessConfig` dataclass and `process_page.py` is a factory that generates a fully-themed page from any config. Adding a new station is one new entry in the registry.

**Shared `ui` dict as event bus** — Pages communicate through a shared dict (`ui["selected_wsr"]`, `ui["refresh_wip_table"]`, etc.) rather than direct references. This keeps pages decoupled and makes the routing logic easy to follow.

**Sequence-aware row targeting** — When a WSR has the same process at multiple sequences (e.g. two grinding passes), the system uses `get_active_row()` to find the lowest-sequence row without a Time Start, rather than writing to all matching rows.

---

## Data Flow

```
Operator scans QR
        │
        ▼
  decode_qr_from_image()
  or QInputDialog (handheld scanner)
        │
        ▼
  _resolve_active_process(wsr, drawing)
  ── walks planned sequences in order
  ── finds first incomplete step
        │
        ├─── First incomplete step has no Time Start?
        │         → route to station page
        │           SCAN #1: operator fills Mc No + PIC, clicks Submit
        │           save_process_data() writes Time Start
        │
        └─── Step has Time Start but no Time End?
                  → SCAN #2: auto-read Mc/PIC, write Time End
                    get_next_process() → route to next station
                    refresh WIP table
```

---

## Process Configuration

Every machining process is defined once in `process_config.py`. The entire app — routing, column names, UI theme, Excel headers — derives from this single source of truth.

```python
ProcessConfig(
    name="GRINDING",
    page_index=10,
    route_key="grinding",               # matched against Process column in Excel
    col_mc_no="Mc No Grinding",         # exact Excel column header
    col_pic="PIC Grinding",
    col_time_start="Time Start Grinding",
    col_time_end="Time End Grinding",
    col_remarks="Remarks Grinding",
    tag_prefix="GRINDING",              # widget tag namespace
    accent=(128, 0, 77, 1),             # RGBA theme colour
    accent_light=(231, 141, 190, 0.28),
)
```

To add a new process station: add one `ProcessConfig` to the list. No other file changes required.

---

## Tech Stack

| Layer | Technology | Role |
|---|---|---|
| UI Framework | PySide6 (Qt) + pyvisual | Window, pages, widgets |
| Data Store | openpyxl | Read/write shared Excel workbook |
| QR Scanning | OpenCV (`cv2`) | Camera feed + image decode |
| PDF Generation | ReportLab | Landscape A5 travel sheets |
| QR Creation | qrcode / PIL | Generate job QR codes |
| Packaging | PyInstaller | Single-folder `.exe` for Windows |

---

## Setup

**Requirements:** Python 3.11+, Windows (tested), network access to shared drive

```bash
git clone https://github.com/yourusername/pe-machine-tracking
cd pe-machine-tracking
pip install -r requirements.txt
python main.py
```

**Network path** — The app expects `DATA.xlsx` at:
```
~/YOKOWO Group/Communication site - 13 IP/02 Non Disclosure/
10_PI/10_ImageAnalysis/PE MACHINE TRACKING/NO1/DATA/DATA.xlsx
```
Adjust `_get_base_dir()` in `data_layer.py` for your environment. Frozen builds look in `_internal/DATA/` next to the executable.

---

## Screenshots

> UI screenshots would go here — WIP board, a process station page, the stats drill-down, and a generated PDF travel sheet.

---

## What I Learned

Building this highlighted a few things that don't come up in tutorials:

- **Shared-file concurrency** — openpyxl's `load_workbook` / `save` cycle is not atomic. In a single-writer environment this is fine; multi-writer would need a lock file or migration to SQLite.
- **Qt layout in a resizable window** — PySide6's geometry management for dynamically-created widgets (QTableWidget inside a custom layout) requires careful cleanup and rebuild on resize. The debounced `resizeEvent` pattern here prevents thrashing.
- **Excel as a schema** — Columns are created on first write if missing (`_ensure_col`), which makes the schema self-healing but means column order isn't guaranteed across installs. A migration step would be cleaner for a larger team.
- **Config-driven UI factories** — The single `ProcessConfig` → `create_process_page_ui()` pattern cut ~1000 lines of duplicated code and made adding a new station a 10-line change.

---

<div align="center">

Built for a precision machining facility in Johor, Malaysia · 2024

</div>
