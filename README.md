# 🧾 Northwind Group – VAT Compliance Dashboard (Power BI)

![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?logo=powerbi&logoColor=black)
![Power Query](https://img.shields.io/badge/Power%20Query-ETL-217346)
![DAX](https://img.shields.io/badge/DAX-23%20measures-0B2D6B)
![Data](https://img.shields.io/badge/Data-50K%20invoice%20lines-4FA3E3)

An end-to-end **tax technology case study**: a multi-country VAT compliance solution for a fictional retail group, built the way a consulting team would deliver it. The project runs from requirements and process mapping, through data cleaning and validation, to a two-page Power BI dashboard that answers five questions from the Head of Tax.

---

# :octocat: Dashboard 👇

### Page 1 – Case Overview
![Case Overview](Images/dashboard_page1_case_overview.png)

### Page 2 – Key Findings
![Key Findings](Images/dashboard_page2_key_findings.png)

---

## 📌 Business Problem

**Northwind Group** is a retailer with five VAT-registered entities in the **UK, Ireland, Germany, France and the Netherlands**. Each entity files a quarterly VAT return.

The Group Tax team combined five country Excel exports by hand, checked only a sample of VAT rates and tracked filing deadlines by email. The Head of Tax needed **one reliable view** of three things:
- the group's VAT position
- errors in the VAT data
- whether returns are filed on time

## ❓ Questions Answered

| # | Question | Answer from the data |
|---|---|---|
| Q1 | How much VAT does the group owe each quarter? | **€1.1M net VAT payable** (€3.6M output − €2.5M input) from Jan 2024 to Jun 2026. Quarterly liability ranges from €66K (2025-Q4) to €143K (2025-Q1). |
| Q2 | Which entities drive the VAT liability? | **NW-UK accounts for 37%** (€410K). The other four entities each account for 14–17%. |
| Q3 | Where are the VAT errors, and how much VAT is at risk? | **976 lines (2.0%) have errors**: 497 use the wrong VAT rate and 479 have no VAT code. Wrong rates put **€97.2K of VAT at risk**, and NW-UK alone accounts for 194 of those lines (€47K). |
| Q4 | Are VAT returns being filed on time? | **76% are filed on time** (38 of 50). 9 were filed late, and NW-DE was late in 5 of 10 quarters. **3 returns for 2026-Q2 are overdue** (UK, IE, FR). |
| Q5 | Does the VAT filed match the VAT calculated from transactions? | Filed VAT is **€19.4K higher** than the transactions support. NW-UK over-declared by €13.1K, and NW-IE under-declared by €1.2K. |

## 💡 Recommendations
1. **Fix data at source:** give country finance teams the exception list (export from the dashboard) before each filing.
2. **Prioritise NW-UK**, which has the largest liability, the most wrong-rate errors and the biggest filed-vs-calculated gap.
3. **File the three overdue 2026-Q2 returns now**, and review NW-DE's filing process, which has a repeated late pattern.
4. **Replace the manual Excel process** with the automated refresh. That cuts manual data-preparation steps from 6 to 1 and checks 100% of transactions instead of a sample.

---

## 🔄 Project Workflow

| Step | What was done |
|---|---|
| **1. Requirements** | Business problem, stakeholders, 8 user stories with testable acceptance criteria ([Requirements Document](Documentation/Requirements_Document.pdf)) |
| **2. Process mapping** | As-is and to-be swimlane maps showing the manual pain points and the automated future state |
| **3. Data profiling** | Data dictionary for every column, plus an issues log of 10 data quality findings ([Data Dictionary](Documentation/Data_Dictionary_and_Issues_Log.xlsx)) |
| **4. ETL (Power Query)** | Combined 5 country files, parsed 3 date formats, removed 248 duplicates, converted GBP→EUR with monthly FX rates |
| **5. Validation** | Checked every line against the correct VAT rate for its country and code. Reverse-charge (RC) lines were treated as valid. |
| **6. Data model** | Star schema with two fact tables sharing date and entity dimensions |
| **7. Reporting** | Two-page dashboard with filters for year, quarter, entity and exception type, and page navigation buttons |

### As-is vs to-be process
![As-is process](Images/process_as_is.png)
![To-be process](Images/process_to_be.png)

---

## 🗂️ Data Model

![Data model](Images/data_model.png)

- **FactVATTransactions:** 49,926 cleaned invoice lines (output and input VAT, expected VAT, exception flag)
- **FactFilings:** 50 quarterly VAT returns (due date, filed date, amount filed)
- **Dimensions:** DimDate, DimEntity, DimCounterparty, DimVATRate
- All relationships are one-to-many and single-direction, so one set of filters drives both fact tables.

## 🧹 Data Quality Issues Found & Handled

| Issue | Rows | How it was handled |
|---|---|---|
| Wrong VAT amount (rate mismatch) | 497 | Flagged by comparing actual VAT with net × correct rate (tolerance €0.05) |
| Missing VAT code | 479 | Flagged as an exception for correction at source |
| Duplicate invoice lines | 248 | Removed in Power Query |
| 3 different date formats | 50,174 | Parsed per format (DD/MM/YYYY, DD.MM.YYYY, YYYY-MM-DD) |
| UK amounts in GBP | 10,059 | Converted to EUR at the monthly average rate |
| Trailing spaces in names | 2,434 | Trimmed |
| Late and overdue returns | 9 + 3 | Shown in a colour-coded filing tracker |

## 📐 Sample DAX Measures

```DAX
Net VAT Payable = [Output VAT] - [Input VAT]

VAT at Risk =
SUMX (
    FILTER ( FactVATTransactions, FactVATTransactions[ExceptionType] = "Wrong VAT rate" ),
    ABS ( FactVATTransactions[VATAmountEUR] - FactVATTransactions[ExpectedVATEUR] )
)

Filing Status =
VAR DueDate = SELECTEDVALUE ( FactFilings[DueDate] )
VAR FiledDate = SELECTEDVALUE ( FactFilings[FiledDate] )
RETURN
    SWITCH ( TRUE (),
        NOT ISBLANK ( FiledDate ) && FiledDate <= DueDate, "Filed on time",
        NOT ISBLANK ( FiledDate ), "Filed late",
        DueDate < TODAY (), "Overdue",
        "Not yet due" )
```
All 23 measures are in [DAX/Measures.dax](DAX/Measures.dax).

---

## 🛠️ Tools & Skills
**Power BI Desktop** · **Power Query (M)** · **DAX** · **Star-schema data modelling** · **Power BI Project (PBIP / TMDL)** · **Requirements gathering** · **Process mapping (draw.io)** · **Data quality assessment** · **Excel**

## 📁 Repository Structure
```
├── Northwind_VAT_Dashboard.pbix     # Ready-to-open Power BI report
├── PowerBI_Project/                 # Same report as a PBIP project (text files, Git-friendly)
├── Datasets/                        # 10 raw CSV files (5 country exports + reference data)
├── Power Query/                     # M scripts for every query
├── DAX/Measures.dax                 # All 23 DAX measures
├── Documentation/                   # Requirements, data dictionary & issues log, process maps
└── Images/                          # Dashboard screenshots, process maps, data model
```

## ▶️ How to Use
1. Download or clone this repository.
2. Open `Northwind_VAT_Dashboard.pbix` in **Power BI Desktop** (free).
3. To refresh the data, go to **Home → Transform data → Edit parameters** and set **DataFolder** to the full path of the `Datasets` folder on your computer, ending with `\`. Then click **Refresh**.

> **Note:** All company names and figures are **fictional**. The dataset was generated for this case study, with realistic data quality issues built in.

---

## 👤 Author
**Nusrat** – University College Dublin
<!-- Add your LinkedIn link here -->

# Don't forget to leave a star ⭐️
