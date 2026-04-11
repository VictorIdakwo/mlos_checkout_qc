# MLOS Checkout QC

A Streamlit app for running automated Quality Control (QC) checks on MLOS (Master List of Settlements) checkout files. The tool validates schema alignment, data integrity, and cross-table consistency before producing a downloadable QC report.

---

## Features

- Upload `.sqlite`, `.csv`, `.xlsx`, or `.xls` checkout files
- **Step 1 — Schema Validation:** checks that all required columns are present before running any QC
- **Step 2 — Automated QC Checks:**
  - **MLoS layer** (`master_list_settlement_update_view`) — 15+ rules
  - **Takeoffpoint layer** (`mlos_takeoffpoint_view`) — 4 rules
  - Cross-table consistency (ward codes, takeoffpoint names & codes)
- Per-rule issue drilldown with expandable row-level detail tables
- Filterable raw data view
- **Generate Report tab** — full QC verdict (CLEAN / FAILING) + downloadable 7-sheet Excel report

---

## Process & Procedure

### 1. Upload Your File

Use the **sidebar uploader** to upload your MLOS checkout file. Supported formats:

| Format | MLoS Data | Takeoffpoint Data |
|--------|-----------|-------------------|
| `.sqlite` / `.db` | `master_list_settlement_update_view` | `mlos_takeoffpoint_view` |
| `.xlsx` / `.xls` | Sheet 1 (or sheet named `mlos`) | Sheet 2 (or sheet named `takeoffpoint`) |
| `.csv` | Entire CSV file | Not available — takeoff cross-checks skipped |

> **Note:** For full QC coverage including takeoffpoint cross-checks, use `.sqlite` or `.xlsx` with two sheets.

---

### 2. Schema Validation (Automatic)

Before any QC rule is applied, the tool checks whether the uploaded file contains all required columns.

**If the schema does not align:**
- A red error banner is shown listing how many columns are missing
- An expandable table identifies each missing column, which table it belongs to, and the impact on QC rules
- A **Download Schema Error Report (.xlsx)** button is provided so you can share or log the issue
- QC execution is paused — no partial or misleading results are shown

**Fix the schema issues in the source file and re-upload to proceed.**

**Required columns — MLoS table:**

| Column | Used In |
|--------|---------|
| `takeoffpoint` | Rules 2, TP2 |
| `takeoffpoint_code` | Rules 3, TP3 |
| `ward_code` | Rules 4, TP4 |
| `settlement_name` | Base identifier |
| `security_compromised` | Rule 6 |
| `accessibility_status` | Rules 7, 8 |
| `reasons_for_inaccessibility` | Rule 8 |
| `habitational_status` | Rule 9 |
| `set_target` | Rule 10 |
| `number_of_houses` | Rule 10 |
| `set_population` | Rule 10 |
| `day_of_activity` | Rule 12 |
| `urban`, `rural`, `scattered` | Rule 13 |
| `highrisk`, `slums`, `densely_populated`, `hard2reach`, `border`, `normadic`, `riverine`, `fulani`, `team_code` | Rule 14 |
| `source` | Rule 15 |
| `editor` | Rule 16 |
| `globalid` | Rule 17 |

**Required columns — Takeoffpoint table:**

| Column | Used In |
|--------|---------|
| `name` | Rule TP2 |
| `code` | Rule TP3 |
| `wardcode` | Rule TP4 |
| `globalid` | Rule TP5 |

---

### 3. QC Checks

Once schema validation passes, QC checks run automatically across both tables.

#### MLoS Table Rules

| Rule | Check | Description |
|------|-------|-------------|
| 2 | Takeoffpoint Name Match | `takeoffpoint` must match `name` in the Takeoffpoint table |
| 3 | Takeoffpoint Code Match | `takeoffpoint_code` must match `code` in the Takeoffpoint table |
| 4 | Ward Code Match | `ward_code` must match `wardcode` in the Takeoffpoint table |
| 5 | No Null in Required Fields | All required fields must not be null or empty |
| 6 | Security Compromised Y/N | `security_compromised` must be `Y` or `N` |
| 7 | Accessibility Status Valid | Must be: `Fully Accessible`, `Partially Accessible`, or `Inaccessible` |
| 8 | Reason for Inaccessibility Required | Partially/Inaccessible settlements must have a reason |
| 9 | Habitational Status Valid | Must be: `Abandoned`, `Migrated`, `Inhabited`, or `Partially Inhabited` |
| 10 | Target & Houses ≤ Population | `set_target` and `number_of_houses` must not exceed `set_population` |
| 12 | Day of Activity Valid | Must be one of: `1`, `1_2`, `1_2_3`, `1_2_3_4`, `2`, `2_3`, `2_3_4`, `3`, `3_4`, `4`, `NA` |
| 13 | Urban / Rural / Scattered Y/N | Each must be `Y` or `N`; a settlement cannot be both Urban and Rural, or Urban and Scattered |
| 14 | Profile Flags Y/N/NA | `highrisk`, `slums`, `densely_populated`, `hard2reach`, `border`, `normadic`, `riverine`, `fulani`, `team_code` must be `Y`, `N`, or `NA` |
| 15 | Source = MLoS | `source` field must start with `MLoS` |
| 16 | Editor Format | `editor` must follow the format `firstname.surname` (all lowercase) |
| 17 | GlobalID is UUID | `globalid` must be a valid UUID (`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`) |

#### Takeoffpoint Table Rules

| Rule | Check | Description |
|------|-------|-------------|
| TP2 | Name matches MLoS | `name` must match `takeoffpoint` in the MLoS table |
| TP3 | Code matches MLoS | `code` must match `takeoffpoint_code` in the MLoS table |
| TP4 | Ward Code matches MLoS | `wardcode` must match `ward_code` in the MLoS table |
| TP5 | GlobalID is UUID | `globalid` must be a valid UUID |

> **Note:** If a CSV file is uploaded, Takeoffpoint data is unavailable and rules TP2–TP5 are skipped. A warning is displayed.

---

### 4. Review Results

Results are displayed across tabs:

- **QC Summary** — pass/fail status per rule with failing row counts and percentages
- **MLoS Issues** — row-level drilldown for each failing MLoS rule
- **Takeoffpoint Issues** — row-level drilldown for each failing takeoffpoint rule
- **Raw Data** — filterable view of the full MLoS dataset

---

### 5. Generate & Download Report

Go to the **Generate Report** tab to:

1. See the overall QC verdict: `✅ CLEAN` or `❌ FAILING`
2. Click **Generate Report** to produce a detailed 7-sheet Excel workbook
3. Download the report and share with the data team or programme leads

The report file is named: `{filename}_QC_Report.xlsx`

If the schema was invalid, a separate **Schema Error Report** (`.xlsx`) can be downloaded from the schema validation step before re-uploading.

---

## How to Run Locally

```bash
pip install -r requirements.txt
streamlit run app.py
```

## Streamlit Cloud

This app is deployable directly to [Streamlit Cloud](https://streamlit.io/cloud).

1. Fork or clone this repo
2. Go to [share.streamlit.io](https://share.streamlit.io)
3. Connect your GitHub repo
4. Set **Main file path** to `app.py`
5. Deploy!

---

## Built By
eHealth Africa
