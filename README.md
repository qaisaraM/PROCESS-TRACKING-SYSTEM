<div align="center">

# QR-Based Manufacturing Process Tracking System

*Replacing paper travel sheets with a real-time, scan-driven WIP tracking system*
*for precision CNC machining shop-floor operations*

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![PySide6](https://img.shields.io/badge/PySide6-Qt_Framework-41CD52?style=flat-square&logo=qt&logoColor=white)](https://doc.qt.io/qtforpython/)
[![OpenCV](https://img.shields.io/badge/OpenCV-QR_Scanning-5C3EE8?style=flat-square&logo=opencv&logoColor=white)](https://opencv.org)
[![openpyxl](https://img.shields.io/badge/openpyxl-Data_Store-217346?style=flat-square&logo=microsoft-excel&logoColor=white)](https://openpyxl.readthedocs.io)
[![ReportLab](https://img.shields.io/badge/ReportLab-PDF_Generation-E34F26?style=flat-square)](https://reportlab.com)
[![Status](https://img.shields.io/badge/Status-Production-success?style=flat-square)](https://github.com/qaisaraM/PROCESS-TRACKING-SYSTEM)

**[🌐 Live Demo](https://qaisaram.github.io/PROCESS-TRACKING-SYSTEM/) · [💻 Source Code](https://github.com/qaisaraM/PROCESS-TRACKING-SYSTEM)**

</div>

---

## Abstract

Manual paper-based travel sheets used in precision CNC machining workshops create visibility gaps, data loss, and communication delays between process stations. This project describes the design and deployment of a desktop-based Work-In-Progress (WIP) tracking system that digitises the entire job routing workflow through QR code scanning. Operators perform a two-scan interaction per process stage — scan to start, scan to complete — and the system automatically records timestamps, routes jobs to the next station, updates a live WIP dashboard, and maintains a structured audit trail in a shared Excel data store. Built for a real machining environment with no cloud infrastructure, the system integrates with existing workflows, generates physical travel sheet PDFs with embedded QR codes, and provides production analytics including per-stage cycle times, idle detection, and workload distribution.

> **Keywords:** QR scanning · workflow automation · WIP tracking · PySide6 · event-driven architecture · shop-floor digitisation · manufacturing informatics

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Solution Architecture](#2-solution-architecture)
3. [System Workflow](#3-system-workflow)
4. [Core Features](#4-core-features)
5. [Performance & Analytics](#5-performance--analytics)
6. [Software Architecture](#6-software-architecture)
7. [Discussion](#7-discussion)
8. [Getting Started](#8-getting-started)
9. [Future Work](#9-future-work)

---

## 1. Problem Statement

In a precision machining shop handling multiple concurrent jobs across sequential process stages, paper-based travel sheets are the primary mechanism for tracking job status. This approach introduces several operational failure modes:

| Pain Point | Operational Impact |
|---|---|
| Travel sheets lost or misfiled | Job status becomes unknown mid-process |
| No real-time visibility | Supervisors must physically walk the floor |
| Manual status checks | Delays in identifying bottlenecks |
| No structured data | Performance analysis is impossible |
| Handoff gaps between stations | Idle time goes undetected and unmeasured |

> The core problem is not the absence of data — every machining event *happens* — but the absence of a system to *capture* it automatically without burdening operators.

---

## 2. Solution Architecture

The system replaces paper travel sheets with a **two-scan digital workflow**: one scan starts a process stage, a second scan ends it. Everything else — routing, timestamps, dashboard updates, data logging — is handled automatically.

![End-to-End Pipeline](https://github.com/qaisaraM/PROCESS-TRACKING-SYSTEM/blob/main/docs/process_diagram.png)


**Key architectural decision:** Excel as the data store — not a traditional database. This matches the existing tool ecosystem on the shop floor, requires no additional infrastructure, and allows supervisors to open and inspect data directly without any special software.

---

## 3. System Workflow

### Process Pipeline

```
Milling → Grinding → Lathe → EWAG → Levin → NC → R04 → QC
```

Each Work Sheet Record (WSR) moves sequentially through defined stages. The system enforces this routing automatically — an operator cannot complete a stage that was never started, and cannot skip stages.

### Scan Event Logic

```python
# Two-scan state machine per WSR per process
SCAN #1  →  No active record found
         →  Create START event
         →  Record: timestamp, machine ID, PIC name

SCAN #2  →  Active START record found
         →  Create END event
         →  Calculate cycle time
         →  Route WSR to next process stage
         →  Update dashboard
```

### Job Status Model

| Status | Condition | Visual |
|--------|-----------|--------|
| Completed | QC stage END recorded | 🟢 |
| In Progress | Active START with no END | 🟡 |
| Idle | No scan activity beyond threshold | 🔴 |
| Waiting | Routed but not yet started | ⚪ |

---

## 4. Core Features

### QR-Based Scan Tracking

- OpenCV reads QR codes from a connected camera feed in real time
- First scan opens a process record (logs start time, machine, PIC)
- Second scan closes it (logs end time, auto-routes to next station)
- No manual data entry — operator interaction is scan-only

### Live WIP Dashboard

- All active jobs displayed across process columns simultaneously
- Grouped by current station: `Pending | Milling | Grinding | ... | QC`
- Job cards show elapsed time, PIC, and status colour
- Refreshes automatically on every scan event

### Planning Module

- Create new WSR jobs with process routing definition
- Assign job details: part number, quantity, priority, due date
- Generates a unique QR code per WSR on creation

### Auto-Generated Travel Sheet (PDF)

```
Per-job PDF includes:
  - WSR identifier and job details
  - Embedded QR code for scanning
  - Process checklist with sign-off fields
  - Serves as both digital trigger and physical document
```

Generated via ReportLab — no manual document preparation required.

### Statistics & Monitoring Panel

- Jobs per station (current snapshot)
- Completed / In-Progress / Idle counts
- Filter by date range and process stage
- Drill-down into individual WSR event history

---

## 5. Performance & Analytics

Because every scan event is timestamped and stored, the system derives production insights that were previously unavailable:

| Metric | Derivation |
|--------|-----------|
| Cycle time per stage | `END timestamp − START timestamp` |
| Total job completion time | `QC END − first Milling START` |
| Idle time between stages | `Next START − Previous END` |
| PIC throughput | Jobs completed per operator per shift |

These metrics enable identification of:

- **Bottleneck stations** — stages with consistently high cycle times
- **Handoff delays** — excessive idle time between sequential stations
- **Workload imbalance** — uneven job distribution across operators

> This transforms the tracking system from a logging tool into a continuous improvement instrument — the same scan data that routes jobs also generates the evidence base for process optimisation decisions.

---

## 6. Software Architecture

### Project Structure

```
PROCESS-TRACKING-SYSTEM/
│
├── ui/                    # PySide6 screens and components
│   ├── dashboard.py       # Live WIP board
│   ├── scan_view.py       # QR scan interface
│   ├── planning.py        # Job creation module
│   └── analytics.py       # Stats and monitoring
│
├── data_layer.py          # All Excel read/write operations
├── process_config.py      # Station definitions and routing rules
├── event_processor.py     # Start/End logic and state transitions
│
├── assets/                # Fonts, icons, UI resources
├── qr_codes/              # Generated QR code images
├── pdf_output/            # Travel sheet PDFs
└── DATA.xlsx              # Shared data store
```

### Data Flow

```
QR Scan (OpenCV)
      ↓
event_processor.py  ←  process_config.py (routing rules)
      ↓
data_layer.py  →  DATA.xlsx (write event)
      ↓
UI refresh  →  dashboard.py (read + render)
```

### Technology Stack

| Component | Technology | Role |
|-----------|------------|------|
| GUI Framework | PySide6 (Qt) | All UI screens and components |
| QR Scanning | OpenCV | Real-time camera QR decode |
| Data Store | openpyxl | Excel read/write operations |
| PDF Output | ReportLab | Travel sheet generation |
| QR Generation | qrcode + Pillow | WSR QR code creation |
| Deployment | PyInstaller | Single-executable packaging |

---

## 7. Discussion

### Event-Driven Design in Physical Systems

The most important architectural decision was treating every scan as an immutable event rather than a mutable status update. Rather than storing "current state" and overwriting it, the system appends events and derives state from the event log. This means the full history of every job is always queryable, scan errors can be identified and corrected without data loss, and the dashboard state can be reconstructed from scratch at any point.

### Excel as Infrastructure

Using Excel as a data store is an unusual choice by software engineering standards, but the right one for this deployment context. The system runs on existing shop-floor PCs without IT infrastructure changes. Supervisors can open `DATA.xlsx` directly to inspect or export data using tools they already know. This reduced adoption friction significantly compared to introducing a new database or web interface.

### Configuration-Driven Process Routing

Defining all process stations and routing rules in `process_config.py` rather than hardcoding them means the system can be adapted to a different shop layout or process sequence by editing a single file — no code changes required. This proved important when the client requested a minor routing adjustment after initial deployment.

### Operator-Minimal Interaction

The two-scan interaction model was a deliberate constraint. Every additional required input (form field, button press, confirmation dialog) represents a potential point of operator error or frustration on a busy shop floor. Reducing the entire tracking action to two scans — with the system handling all routing and logging — was essential to adoption.

---

## 8. Getting Started

### Requirements

- Windows 10/11
- Python 3.11+
- USB or IP camera for QR scanning
- Shared network path for `DATA.xlsx` (multi-workstation use)

### Installation

```bash
git clone https://github.com/qaisaraM/PROCESS-TRACKING-SYSTEM
cd PROCESS-TRACKING-SYSTEM
pip install -r requirements.txt
python main.py
```

### Configuration

Edit `process_config.py` to define your process stations and routing sequence:

```python
PROCESS_STAGES = [
    "Milling", "Grinding", "Lathe",
    "EWAG", "Levin", "NC", "R04", "QC"
]
```

### Output Structure

```
pdf_output/
└── WSR_<id>_<date>.pdf       # Travel sheet with QR code

qr_codes/
└── WSR_<id>.png              # QR code image

DATA.xlsx                     # All scan events and job records
```

---

## 9. Future Work

- Web-based dashboard for remote supervisor monitoring without desktop app install
- Push notifications to supervisor on job completion or idle-time threshold breach
- Multi-site support with centralised data aggregation across workshops
- Integration with ERP/MES systems via REST API for bidirectional job data sync
- Predictive scheduling based on historical cycle time distributions
- Barcode and RFID support as alternatives to QR scanning

---

<div align="center">

*Built for real production use · Deployed in a precision machining workshop · 2026*

**[LinkedIn](https://linkedin.com/in/qaisara-mardhiah) · [Portfolio](https://qaisaram.github.io/PROCESS-TRACKING-SYSTEM/) · [GitHub](https://github.com/qaisaraM)**

</div>
