# MediCare-Connect-Hospital-Network-Power-BI-Analytics-Report

**A production-grade Power BI capstone project** solving real business problems in healthcare using data modelling, DAX, and interactive dashboards.

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?style=flat&logo=powerbi&logoColor=black)
![Excel](https://img.shields.io/badge/Dataset-Excel%20%7C%20Star%20Schema-217346?style=flat&logo=microsoft-excel&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-28%20Measures-0078D4?style=flat)
![Healthcare](https://img.shields.io/badge/Industry-Healthcare-E8183F?style=flat)
![Status](https://img.shields.io/badge/Status-Complete-2EA043?style=flat)


## Project Overview

MediCare Connect is a **5-branch private hospital network** operating across Lagos, Abuja, Rivers, and Kano (Nigeria). Despite rapid growth, leadership had **no unified view** of clinical operations, patient experience, or financial performance across branches — they were consolidating 5 Excel files manually, a process that took 2 weeks and was already out of date on arrival.

**This project replaces that manual process with a single, interactive Power BI report** that gives every level of the business — from branch managers to the executive committee — a clear, trustworthy, and near-real-time view of how the hospital network is performing.


| | |
|---|---|
| **Sector** | Healthcare — Multi-Branch Hospital Operations |
| **Client** | MediCare Connect Hospital Network (simulated) |
| **Data period** | FY2024 – FY2025 (2 full years) |
| **Total encounters** | 18,000 patient visits |
| **Total revenue billed** | ₦1.92 billion |
| **Report pages** | 4 (Executive Overview · Operations · Financial · Quality of Care) |
| **Data model** | Star schema — 2 fact tables, 7 dimension tables |
| **DAX measures** | 28 across 5 themes |


## Business Problems Solved

The Group Medical Director and CFO raised four specific concerns before this project started:

1. **Operational blind spots** — no easy visibility into patient volumes, wait times, or LOS trends by branch
2. **Revenue leakage** — finance suspected significant uncollected revenue but couldn't quantify or locate it
3. **Inconsistent patient experience** — satisfaction and wait times appeared to vary by branch, but had never been measured systematically
4. **Manual, slow reporting** — 2-week Excel consolidation cycle was already stale before leadership read it

This report delivers a specific, data-backed answer to each of those four problems.

---

## Repository Structure

```
medicare-connect-powerbi/
│
├── README.md                          ← You are here
│
├── docs/
│   ├── 01_Business_Scenario_and_Objectives.docx   ← Project brief & scope
│   ├── 02_DAX_Insights_ReportDesign.docx          ← DAX library + formatting spec
│   └── 03_ReportMockups_AllPages.docx             ← All 4 page mockups (22 pages)
│
├── data/
│   └── MediCareConnect_PowerBI_Dataset.xlsx       ← Star schema dataset (9 tabs)
│       ├── 0_START_HERE                           ← Model guide & data dictionary
│       ├── Dim_Date
│       ├── Dim_Patient
│       ├── Dim_Physician
│       ├── Dim_Department
│       ├── Dim_Hospital
│       ├── Dim_Diagnosis
│       ├── Dim_Payer
│       ├── Fact_Encounters
│       └── Fact_Billing
│
└── report/
    └── MediCareConnect.pbix                       ← Power BI report file
```

---

## Data Model — Star Schema

The dataset is structured as a **star schema**, the industry standard for Power BI and analytical modelling. This design ensures fast DAX calculations, clean relationships, and a model that can be extended without restructuring.



                    ┌─────────────┐
                    │  Dim_Date   │
                    └──────┬──────┘
                           │


┌──────────────┐    ┌──────┴──────────┐    ┌──────────────┐
│  Dim_Patient │────│                 │────│  Dim_Payer   │
└──────────────┘    │  Fact_Encounters│    └──────────────┘
┌──────────────┐    │  (18,000 rows)  │    ┌──────────────┐
│ Dim_Physician│────│                 │────│ Dim_Diagnosis│
└──────────────┘    └──────┬──────────┘    └──────────────┘
          
                           │    │
          ┌────────────────┘    └───────────────┐
          │                                     │
   ┌──────┴──────┐                    ┌─────────┴──────┐
   │ Dim_Hospital│                    │ Dim_Department │
   └─────────────┘                    └────────────────┘
                           │
                    ┌──────┴──────────┐
                    │  Fact_Billing   │
                    │  (18,000 rows)  │
                    └─────────────────┘


 ### Table Reference

| Table | Type | Rows | Grain / Description |
|---|---|---|---|
| `Fact_Encounters` | Fact | 18,000 | One row per patient visit (outpatient, inpatient, emergency) |
| `Fact_Billing` | Fact | 18,000 | One billing record per encounter — 1:1 relationship |
| `Dim_Date` | Dimension | 731 | Every calendar day Jan 2024 – Dec 2025 |
| `Dim_Patient` | Dimension | 2,500 | Unique registered patients |
| `Dim_Physician` | Dimension | 60 | Physicians across 5 branches |
| `Dim_Department` | Dimension | 12 | Clinical departments |
| `Dim_Hospital` | Dimension | 5 | Hospital branches by region |
| `Dim_Diagnosis` | Dimension | 15 | ICD-10-style diagnosis categories |
| `Dim_Payer` | Dimension | 6 | Payer types (NHIS, HMOs, Self-Pay, Corporate) |

> **How to build relationships:** Open Power BI Model view. Drag `DateID` from `Fact_Encounters` onto `DateID` in `Dim_Date` and repeat for each foreign key. All relationships are Many-to-One from the fact table to the dimension. Full guide is in the `0_START_HERE` tab of the dataset.


## DAX Measure Library

All 28 measures live in a dedicated `_Measures` table. Create it with **Modeling → New Table**, name it `_Measures`, then add each measure below.

### Volume & Patient Flow

```dax
Total Encounters = COUNTROWS(Fact_Encounters)

Total Patients (Distinct) = DISTINCTCOUNT(Fact_Encounters[PatientID])

Outpatient Encounters =
    CALCULATE([Total Encounters], Fact_Encounters[EncounterType] = "Outpatient")

Emergency Encounters =
    CALCULATE([Total Encounters], Fact_Encounters[EncounterType] = "Emergency")

Emergency Share % = DIVIDE([Emergency Encounters], [Total Encounters])

Encounters MoM Growth % =
    DIVIDE([Total Encounters] - CALCULATE([Total Encounters],
    PREVIOUSMONTH(Dim_Date[Date])),
    CALCULATE([Total Encounters], PREVIOUSMONTH(Dim_Date[Date])))
```

### Operational Efficiency

```dax
Avg Wait Time =
    AVERAGE(Fact_Encounters[WaitTimeMinutes])

Avg LOS =
    CALCULATE(
        AVERAGE(Fact_Encounters[LengthOfStayDays]),
        Fact_Encounters[LengthOfStayDays] > 0
    )

Encounters per Physician =
    DIVIDE([Total Encounters], DISTINCTCOUNT(Fact_Encounters[PhysicianID]))

High Wait Flag =
    IF([Avg Wait Time] > 30, "⚠ Above Target", "On Target")
```

### Financial Performance

```dax
Total Billed = SUM(Fact_Billing[BilledAmountNGN])

Total Collected = SUM(Fact_Billing[AmountPaidNGN])

Collection Rate % = DIVIDE([Total Collected], [Total Billed])

Revenue Gap = [Total Billed] - [Total Collected]

Revenue per Encounter = DIVIDE([Total Billed], [Total Encounters])

Denied Claims =
    CALCULATE(COUNTROWS(Fact_Billing), Fact_Billing[ClaimStatus] = "Denied")

Denied Claims % = DIVIDE([Denied Claims], COUNTROWS(Fact_Billing))

Self-Pay Revenue Share % =
    VAR SelfPayBilled =
        CALCULATE([Total Billed], Dim_Payer[PayerType] = "Self-Pay")
    RETURN DIVIDE(SelfPayBilled, [Total Billed])
```

### Quality of Care

```dax
Avg Satisfaction =
    AVERAGE(Fact_Encounters[PatientSatisfactionScore])

Readmissions = SUM(Fact_Encounters[ReadmittedWithin30Days])

Readmission Rate % = DIVIDE([Readmissions], [Total Encounters])

AMA Discharge % =
    DIVIDE(
        CALCULATE([Total Encounters],
        Fact_Encounters[DischargeStatus] = "Left Against Medical Advice"),
        [Total Encounters]
    )

Satisfaction Flag =
    IF([Avg Satisfaction] < 3.8, "⚠ Below Target", "On Target")
```

### Time Intelligence

```dax
Revenue YTD = TOTALYTD([Total Billed], Dim_Date[Date])

Revenue PY = CALCULATE([Total Billed], SAMEPERIODLASTYEAR(Dim_Date[Date]))

Revenue YoY % = DIVIDE([Total Billed] - [Revenue PY], [Revenue PY])

Rolling 3M Avg Encounters =
    AVERAGEX(
        DATESINPERIOD(Dim_Date[Date], MAX(Dim_Date[Date]), -3, MONTH),
        [Total Encounters]
    )
```

---

## Report Pages

### Page 1 — Executive Overview
**Audience:** CEO and Executive Committee

> *"The network treated 18,000 patients across 5 branches in 2024–2025, billing ₦1.92bn and collecting 92% of it — but collection performance and patient experience vary meaningfully by branch."*

| Visual | Fields | Insight |
|---|---|---|
| 4 KPI Cards | Total Encounters, Total Billed, Collection Rate %, Avg Satisfaction | 18,000 encounters · ₦1.92bn · 92.2% · 3.96/5 |
| Combo Chart | Encounters + Collection Rate % by HospitalName | Ikeja (5,237) and Lekki (5,154) lead volume; Kano highest collection rate (92.7%) |
| Line Chart | Encounters by Month | Seasonal spike Apr–Sep peaking at 1,865 in August |
| Donut Chart | Encounters by EncounterType | Outpatient 54% · Inpatient 26% · Emergency 20% |

---


### Page 2 — Operations & Patient Flow
**Audience:** Branch Managers and Department Heads

> *"Emergency Medicine carries the highest volume and the longest wait time (34.5 min vs. a 24.4-min network average) — this is where staffing pressure is concentrated."*

| Visual | Fields | Insight |
|---|---|---|
| KPI Cards | Avg Wait Time, Avg LOS, Encounters per Physician | 24.4 min · 3.2 days · 300 enc/physician |
| Horizontal Bar | Avg Wait Time by DepartmentName | Emergency at 34.5 min is 57% above every other department |
| Scatter Chart | Wait Time vs. Satisfaction, bubble = Encounters | Emergency sits alone in the bottom-right danger zone |
| Clustered Column | Encounters by Department × HospitalName | Ikeja and Lekki drive load in every department |
| Top Diagnoses | Total Encounters by DiagnosisDescription (Top 8) | Road Traffic Trauma (2,351) and Pneumonia (1,698) lead |

---

### Page 3 — Financial Performance
**Audience:** CFO and Finance Team

> *"The network collects 92% overall, but Self-Pay patients fall to 81% — that gap is responsible for most of the ₦150.6m outstanding."*

| Visual | Fields | Insight |
|---|---|---|
| 4 KPI Cards | Total Billed, Total Collected, Collection Rate %, Revenue Gap | ₦1.92bn · ₦1.77bn · 92.2% · ₦150.6m gap |
| Horizontal Bar | Collection Rate % by PayerName | HMOs at 94–95%; Self-Pay the sole laggard at 81% |
| Stacked Column | Total Billed by Hospital × PayerType | Self-Pay segment proportionally similar across all branches |
| Donut | Count of Claims by ClaimStatus | 10,581 Partially Paid · 7,291 Paid · 128 Denied |
| Line Chart | Total Billed vs. Collected by YearMonth | 8% gap holds steady across both years — not widening |
| Matrix | Dept × Revenue per Encounter | Oncology ₦275K · Surgery ₦200K · Orthopedics ₦171K |

---

### Page 4 — Quality of Care
**Audience:** Clinical Governance and Quality Team

> *"Network-wide readmission is a healthy 2.9%, but Emergency Medicine alone sits at 4.7% — paired with the lowest satisfaction score (3.58/5), it is the clearest quality signal in the data."*

| Visual | Fields | Insight |
|---|---|---|
| KPI Cards w/ icons | Readmission Rate %, Avg Satisfaction, AMA % | 2.9% ✓ · 3.96/5 ⚠ · 2.4% ✓ |
| Bar Chart | Readmission Rate % by DepartmentName | Emergency (4.7%) is double every other department |
| Bar Chart | Avg Satisfaction by Department | Emergency (3.58) is only dept below 3.8 target |
| Bar Chart | Encounters by DischargeStatus | 93%+ routine/recovered; AMA and Deceased as watch-list |
| Table | Diagnosis × Readmit % × Satisfaction × Encounters | Typhoid (4.1%), Malaria (3.6%), MI (3.5%) lead readmissions |
| Decomposition Tree | Readmissions → Hospital → Dept → Diagnosis | Traces 522 readmissions to a specific, investigable cluster |

---

## Key Findings

The report surfaces four headline findings, each supported by multiple independent visuals across different pages:

### 1. Emergency Medicine is the single biggest operational risk
Across **three independent metrics** — wait time (Page 2), satisfaction score (Page 4), and 30-day readmission rate (Page 4) — Emergency Medicine is the only department flagged on all three. It carries the highest volume of any department and the worst outcomes on every quality measure. The recommended first action is a staffing and triage protocol review specifically for Emergency Medicine.

### 2. The ₦150.6m revenue gap is a collections problem, not a billing problem
Only 128 of 18,000 claims were denied (<1%) — the billing and coding process is sound. The gap is concentrated in **Self-Pay patients (81% collection vs. 94–95% for all HMO payers)**. Introducing an upfront deposit or installment-plan policy for self-pay patients is a specific, costed intervention the data directly supports.

### 3. Seasonal staffing should be pre-positioned from late March
Patient volume spikes 35%+ from April through September, driven by malaria and respiratory cases. The network should plan staffing, bed allocation, and medication supply around this window rather than reacting to it after the fact.

### 4. Ikeja and Lekki run the network, but volume and collection efficiency don't correlate
Ikeja and Lekki together handle 57% of encounters. Kano has the smallest volume but the highest collection rate. This tells the CFO that smaller branches are not financially underperforming — they may need different management strategies rather than volume growth targets.

---

## Report Design Standards

| Element | Specification |
|---|---|
| **Canvas size** | 1280 × 720 px (16:9 widescreen) |
| **Primary color** | Navy `#1F4E79` |
| **Accent** | Blue `#2E75B6` |
| **On-target** | Green `#548235` |
| **Monitor** | Amber `#BF8F00` |
| **Alert** | Red `#C00000` |
| **Font** | Segoe UI (all elements) |
| **Page title** | 24pt · Bold · Navy |
| **KPI values** | 28–32pt · Bold · status-colored |
| **Body / subtitles** | 10–11pt · Regular · Dark Grey |
| **Conditional rule** | Color is never used alone — always paired with a label or icon |

---

## How to Use This Repository

**Prerequisites:** Power BI Desktop (free download from Microsoft)

```bash
# 1. Clone the repo
git clone https://github.com/yourusername/medicare-connect-powerbi.git
cd medicare-connect-powerbi

# 2. Read the brief first (this is how real analytics engagements start)
open docs/01_Business_Scenario_and_Objectives.docx

# 3. Load the data
# Open Power BI Desktop → Get Data → Excel Workbook → select data/MediCareConnect_PowerBI_Dataset.xlsx
# Import all 9 sheets. Read the 0_START_HERE tab before building relationships.

# 4. Build the model
# Follow the relationship map in 0_START_HERE. All Many-to-One from Fact → Dim.

# 5. Write the DAX
# Create a _Measures table. Paste measures from docs/02_DAX_Insights_ReportDesign.docx.
# Verify every measure before building visuals.

# 6. Build the report
# Use docs/03_ReportMockups_AllPages.docx as your visual target.
# Match layout, colors, and KPI card design exactly.
```

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| Power BI Desktop | Report development, DAX, data modelling |
| Microsoft Excel | Dataset delivery (star schema, 9 sheets) |
| DAX | 28 measures across volume, operations, financial, quality, and time intelligence themes |
| Python (pandas, faker, openpyxl) | Synthetic dataset generation with realistic seasonal patterns |
| Microsoft Word | Business scenario documentation, design specification, mockup briefs |

---

## License

This project is released for **educational and portfolio use**. All data is fully synthetic — no real patients, physicians, or financial records are represented. The MediCare Connect brand is fictional.

---

## Connect

If you found this project useful or have questions about the methodology, connect with me on [LinkedIn](#) or open a GitHub Issue.

*Built with precision. Documented with intent. Designed to tell a clear story.*

                    
