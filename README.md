🔗 [View Source Code](https://github.com/qaisaraM/PROCESS-TRACKING-SYSTEM)  
🌐 [Live Demo](https://qaisaram.github.io/PROCESS-TRACKING-SYSTEM/)

<div align="center">

<br/>

<pre>
██████╗ ███████╗    ███╗   ███╗ █████╗  ██████╗██╗  ██╗██╗███╗   ██╗███████╗
██╔══██╗██╔════╝    ████╗ ████║██╔══██╗██╔════╝██║  ██║██║████╗  ██║██╔════╝
██████╔╝█████╗      ██╔████╔██║███████║██║     ███████║██║██╔██╗ ██║█████╗  
██╔═══╝ ██╔══╝      ██║╚██╔╝██║██╔══██║██║     ██╔══██║██║██║╚██╗██║██╔══╝  
██║     ███████╗    ██║ ╚═╝ ██║██║  ██║╚██████╗██║  ██║██║██║ ╚████║███████╗
╚═╝     ╚══════╝    ╚═╝     ╚═╝╚═╝  ╚═╝ ╚═════╝╚═╝  ╚═╝╚═╝╚═╝  ╚═══╝╚══════╝
</pre>

### **Tracking System**

<br/>

**Real-time WIP tracking system for CNC machining shop-floor operations**

<br/>

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square\&logo=python\&logoColor=white)
![PySide6](https://img.shields.io/badge/PySide6-Qt_Framework-41CD52?style=flat-square\&logo=qt\&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-QR_Scanning-5C3EE8?style=flat-square\&logo=opencv\&logoColor=white)
![Excel](https://img.shields.io/badge/openpyxl-Data_Store-217346?style=flat-square\&logo=microsoft-excel\&logoColor=white)
![ReportLab](https://img.shields.io/badge/ReportLab-PDF_Generation-E34F26?style=flat-square)
![Status](https://img.shields.io/badge/Status-Production-success?style=flat-square)

<br/>

</div>

---

## 🧭 Overview

A desktop-based **Work-In-Progress (WIP) tracking system** built for a precision machining shop.

Operators scan QR codes at each station, and the system automatically:

* Records process start and end time
* Routes jobs to the next process
* Updates a live WIP dashboard

Runs as a **standalone desktop application** connected to a shared Excel file. No cloud or browser required.

---

## ⚠️ Problem

In a machining shop handling multiple jobs across several process stages:

* Paper travel sheets can be lost or delayed
* No real-time visibility of job status
* Supervisors need to manually check machines
* No structured data for performance tracking

---

## 💡 Solution

A simple QR-based workflow:

```
SCAN → RECORD → UPDATE → NEXT PROCESS
```

* First scan → start process
* Second scan → complete process
* System handles routing and logging automatically

---

## 🏭 Process Flow

```
Milling → Grinding → Lathe → EWAG → Levin → NC → R04 → QC
```

Each WSR moves step-by-step through defined stages.

---

## ⚙️ Core Features

### 🔹 QR-Based Tracking

```
SCAN #1 → Record Time Start + input Machine & PIC
SCAN #2 → Record Time End + auto route to next station
```

* No manual status updates
* Minimal operator interaction

---

### 🔹 Live WIP Board

* Real-time job tracking across all stations
* Jobs grouped by current process
* Waiting jobs clearly visible

```
Pending | Milling | Grinding | Lathe | ... | QC
```

---

### 🔹 WIP Status System

| Status         | Description                  |
| -------------- | ---------------------------- |
| 🟢 Completed   | Job finished and QC approved |
| 🟡 In Progress | Currently being processed    |
| 🔴 Idle        | No recent activity           |

---

### 🔹 Planning Module

* Create new WSR jobs
* Define process routing
* Assign job details

---

### 🔹 Travel Sheet (PDF)

* Auto-generated per job
* Includes QR code and checklist
* Used as physical tracking document

---

### 🔹 Stats & Monitoring

* Jobs per station
* Completed / WIP / Idle counts
* Filter by time range
* Drill-down into job-level details

---

## 🧠 Performance & Analytics

The system also provides basic production insights:

* Time taken per process stage
* Total completion time per WSR
* Idle time between stages
* PIC performance based on job completion speed

Helps identify:

* Bottlenecks
* Slow processes
* Workload distribution

---

## 🖥 UI System

Built using **PySide6 (Qt framework)** with custom UI components.

### Main Screens:

* WIP Dashboard
* Process tracking view
* Live scan activity
* Basic analytics panel

---

## 🏗 System Design

### Architecture

```
UI (PySide6)
   ↓
Data Layer (openpyxl)
   ↓
Excel File (shared storage)
```

---

### Key Design Decisions

**Excel as database**

* Matches existing workflow
* No extra infrastructure needed

**Config-driven process system**

* All stations defined in one place
* Easy to extend

**Event-based tracking**

* Every scan is a recorded event
* State is derived from data

---

## 🔄 Data Flow

```
Scan QR
   ↓
Identify WSR + Process
   ↓
Determine Start / End
   ↓
Write to Excel
   ↓
Update dashboard
```

---

## 🖥 Tech Stack

| Layer          | Technology              |
| -------------- | ----------------------- |
| UI             | PySide6 (Qt) + pyvisual |
| Data           | openpyxl (Excel)        |
| QR Scanning    | OpenCV                  |
| PDF Generation | ReportLab               |
| QR Code        | qrcode + PIL            |
| Packaging      | PyInstaller             |

---

## 📁 Project Structure

```
ui/                # UI pages and logic
data_layer.py      # Excel operations
process_config.py  # Process definitions
assets/            # Fonts, icons
qr_codes/          # Generated QR codes
pdf_output/        # Travel sheet output
DATA.xlsx          # Main data file
```

---

## 📚 Key Learnings

* Designing systems that integrate with real shop-floor workflows
* Handling Excel as a shared data source
* Building event-driven tracking from physical actions
* Reducing duplicated UI logic with config-based design
* Managing real-time updates in a desktop application

---

## 📍 Context

Built for internal use in a machining workshop environment, focused on improving:

* Production visibility
* Workflow tracking
* Communication between teams

---

<div align="center">

**Built for real-world production use · QaisaraM · 2026**

</div>
