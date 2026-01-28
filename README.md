# 🧪 Mass Balance Calculator  
### AI-Assisted Pharmaceutical Stability Analysis Tool

A **dual-mode Mass Balance Calculator** (Web App + Excel) built for **forced degradation studies** in pharmaceuticals, aligned with **ICH Q1A(R2)** guidelines.

This project enables analysts to **accurately calculate mass balance**, **select the correct method automatically**, and **generate professional, audit-ready PDF reports in seconds**.

---

## 🚨 Problem Statement

In pharmaceutical stability testing:

- Mass balance calculations are **manual and error-prone**
- Multiple calculation methods exist, but **no clear guidance** on which to use
- Excel sheets are inconsistent and hard to audit
- Report generation is slow and non-standard

❌ These issues lead to **delays, rework, and regulatory risk**.

---

## ✅ Our Solution

An **intelligent Mass Balance Calculator** that:

- Supports **four industry-accepted methods**
- Automatically **recommends the optimal method**
- Applies **correction factors (λ, ω)** where required
- Generates **professional PDF reports**
- Works both **online (web app)** and **offline (Excel)**

---

## 🔬 Supported Calculation Methods

| Method | Description |
|------|------------|
| SMB | Simple Mass Balance |
| AMB | Absolute Mass Balance |
| RMB | Relative Mass Balance |
| LK-IMB | Lukulay-Körner Improved Mass Balance (Recommended) |

The system automatically selects the most appropriate method based on degradation behavior and data quality.

---

## 🧠 Key Features

- 🎯 Intelligent method recommendation
- 🎨 Color-coded risk assessment (PASS / ALERT / OOS)
- 📊 Interactive charts and visual gauges
- 📄 Professional PDF report export
- 🕒 Calculation history tracking
- 📈 Trend-analysis ready
- 📋 ICH Q1A(R2) compliant logic

---

## 🖥️ Web Application

### Tech Stack
- **Frontend:** React, Vite, Tailwind CSS
- **Backend:** Node.js, Express
- **Database:** SQLite
- **Charts:** Recharts
- **PDF Generation:** jsPDF

### Capabilities
- Real-time calculations
- Save and review past runs
- Download audit-ready PDF reports
- Clean, responsive UI

---

## 📊 Excel Tool (Offline)

- Yellow input cells with validation
- Formula-based calculations (no macros)
- Conditional formatting (Green / Yellow / Red)
- Diagnostic summary sheet
- Trend tracking support
- Compatible with Excel and Google Sheets

---

## 📂 Project Structure

typeshi/
├── backend/ # Express API + SQLite
├── frontend/ # React frontend
├── README.md # Project documentation


---

## 🚀 Running the Project Locally

### Backend

cd backend
npm install
npm run dev


**Frontend**
cd frontend
npm install
npm run dev
