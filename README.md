# MLOS Checkout QC

A Streamlit app for running automated Quality Control (QC) checks on MLOS (Master List of Settlements) checkout files across **4 QC layers**: schema alignment, MLoS data integrity, takeoffpoint cross-checks, and admin boundary validation.

---

## Features

- Upload `.sqlite`, `.csv`, `.xlsx`, or `.xls` checkout files
- **Step-by-step progress bar** displayed during QC run
- **4 QC Layers** run automatically on upload:
  - 🔎 **Schema Alignment** — verifies all required columns are present before running data checks
  - 🏘️ **MLoS Rules** — 15+ data integrity and cross-table checks
  - 📍 **Takeoffpoint Rules** — 4 cross-table consistency checks
  - 🗺️ **Boundary Checks** — ward code and coordinate validation against admin ward boundary reference
- **🔧 Auto Correct tab** — automatically fixes common data issues and exports a corrected file
- **Pass Rate % and Fail Rate %** displayed on the dashboard
- Per-rule issue drilldown with expandable row-level detail tables
- Downloadable Excel reports per QC layer and per rule
- **MLoS Issues — Longitudinal View** — one row per settlement, Yes/No per rule column, downloadable
- **Generate Report tab** — full QC verdict (CLEAN / FAILING) + downloadable 7-sheet Excel report
- **Send QC Email** — sends summary to the data team on demand

---

## Supported File Formats

| Format | MLoS Data | Takeoffpoint Data |
|--------|-----------|-------------------|
| `.sqlite` / `.db` | `master_list_settlement_update_view` | `mlos_takeoffpoint_view` |
| `.xlsx` / `.xls` | Sheet 1 (or sheet named `mlos`) | Sheet 2 (or sheet named `takeoffpoint`) |
| `.csv` | Entire CSV file | Not available — takeoff cross-checks skipped |

> For full QC coverage including takeoffpoint and boundary checks, use `.sqlite` or `.xlsx`.

---

## Tab Overview

| Tab | Contents |
|-----|----------|
| 🔧 Auto Correct | Automated data fixes with correction log + corrected file download |
| 📊 QC Summary | Pass/fail status per rule, failing row counts, Pass Rate %, Fail Rate % |
| 🏘️ MLoS Issues | Row-level drilldown per failing rule + Longitudinal View (Yes/No per rule column) |
| 📍 Takeoffpoint Issues | Row-level drilldown for each failing takeoffpoint rule + download |
| 🗺️ Boundary Issues | Row-level drilldown for ward code and coordinate failures + download |
| 🔍 Raw Data | Filterable view of the full MLoS and Takeoffpoint datasets |
| 📄 Generate Report | Full QC verdict, 7-sheet Excel report download, and Send Email button |

---

## Process & Procedure

### 1. Upload Your File

Use the **sidebar uploader** to upload your MLOS checkout file. The tool accepts `.sqlite`, `.db`, `.csv`, `.xlsx`, and `.xls` files.

Once uploaded, the app runs all 4 QC layers in sequence. A **labelled progress bar** tracks each step:

| Step | Progress | Layer |
|------|----------|-------|
| Step 1 / 4 | 5% → 25% | 🔎 Schema Alignment |
| Step 2 / 4 | 26% → 50% | 🏘️ MLoS Rules |
| Step 3 / 4 | 51% → 75% | 📍 Takeoffpoint Rules |
| Step 4 / 4 | 76% → 100% | 🗺️ Boundary Checks |

---

### 2. Schema Alignment (Rules S1–S2)

The first QC layer checks whether the uploaded file contains all required columns **before** running any data checks. Unlike a hard gate, schema results are reported as part of the QC summary so the process continues and shows all issues at once.

**If columns are missing:**
- Rule S1 or S2 shows as `❌ FAIL` in the QC Summary
- An expandable table lists each missing column, which table it belongs to, and the QC rules impacted
- A **Download Schema Error Report (.xlsx)** button appears

**Required columns — MLoS table (41 columns):**

| Column | Used In Rule(s) |
|--------|----------------|
| `state_code`, `state_name` | Base identifiers |
| `lga_code`, `lga_name` | Base identifiers |
| `ward_name`, `ward_code` | Rules 4, B1, B2 |
| `takeoffpoint` | Rules 2, TP2 |
| `takeoffpoint_code` | Rules 3, TP3 |
| `settlement_name` | Base identifier |
| `primarysettlement_name`, `alternate_name` | Nullable fields |
| `latitude`, `longitude` | Rule B2 |
| `security_compromised` | Rule 6 |
| `accessibility_status` | Rules 7, 8 |
| `reasons_for_inaccessibility` | Rule 8 |
| `habitational_status` | Rule 9 |
| `set_population`, `set_target`, `number_of_houses` | Rule 10 |
| `noncompliant_household` | Rule 10 |
| `team_code` | Rule 14 |
| `day_of_activity` | Rule 12 |
| `urban`, `rural`, `scattered` | Rule 13 |
| `highrisk`, `slums`, `densely_populated`, `hard2reach`, `border`, `normadic`, `riverine`, `fulani` | Rule 14 |
| `timestamp`, `last_updated` | Metadata |
| `source` | Rule 15 |
| `editor` | Rule 16 |
| `globalid`, `fc_globalid`, `settlementarea_globalid` | Rule 17 |

**Required columns — Takeoffpoint table (4 columns):**

| Column | Used In Rule(s) |
|--------|----------------|
| `name` | Rule TP2 |
| `code` | Rule TP3 |
| `wardcode` | Rule TP4 |
| `globalid` | Rule TP5 |

---

### 3. Auto Correct

After reviewing the QC Summary, go to the **🔧 Auto Correct** tab to apply automatic fixes to the uploaded MLoS data.

Four corrections are applied automatically:

| # | Field(s) | Correction |
|---|----------|------------|
| 1 | `highrisk`, `slums`, `densely_populated`, `hard2reach`, `border`, `normadic`, `scattered`, `riverine`, `fulani` | NULL values replaced with `NA` |
| 2 | `reasons_for_inaccessibility` | Filled with `NA` where `accessibility_status` is `Fully Accessible` and the field is NULL |
| 3 | `source` | NULL or empty values replaced with `IE` |
| 4 | `globalid` | All `{` and `}` characters stripped; any still-invalid UUID replaced with a freshly generated UUID |

The tab displays a **correction log** (column, correction type, rows fixed). A **Download Corrected MLoS (Excel)** button exports the fixed dataset as `{filename}_corrected.xlsx`.

> If no corrections are needed, a green "No corrections needed" message is shown.

---

### 4. MLoS QC Checks (Rules 2–17)

Data integrity rules applied to the MLoS table.

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
| 10 | set_target ≤ set_population | `set_target` must not exceed `set_population` |
| 11 | number_of_houses ≤ set_population | `number_of_houses` must not exceed `set_population` |
| 12 | Day of Activity Valid | Must be one of: `1`, `1_2`, `1_2_3`, `1_2_3_4`, `2`, `2_3`, `2_3_4`, `3`, `3_4`, `4`, `NA` |
| 13 | Urban / Rural / Scattered Y/N | Each must be `Y` or `N`; cannot be both Urban and Rural, or Urban and Scattered |
| 14 | Profile Flags Y/N/NA | `highrisk`, `slums`, `densely_populated`, `hard2reach`, `border`, `normadic`, `riverine`, `fulani`, `team_code` must be `Y`, `N`, or `NA` |
| 16 | Editor Format | `editor` must follow the format `firstname.surname` (all lowercase) |
| 17 | GlobalID is UUID | `globalid` must be a valid UUID (`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`) |

---

### 5. Takeoffpoint QC Checks (Rules TP2–TP5)

Cross-table consistency checks between the Takeoffpoint and MLoS tables.

| Rule | Check | Description |
|------|-------|-------------|
| TP2 | Name matches MLoS | `name` must match `takeoffpoint` in the MLoS table |
| TP3 | Code matches MLoS | `code` must match `takeoffpoint_code` in the MLoS table |
| TP4 | Ward Code matches MLoS | `wardcode` must match `ward_code` in the MLoS table |
| TP5 | GlobalID is UUID | `globalid` must be a valid UUID |

> If a CSV file is uploaded, Takeoffpoint data is unavailable and rules TP2–TP5 are skipped with a warning.

---

### 6. Boundary Checks (Rules B1–B2)

Spatial and reference validation against the admin ward boundary dataset (9,410 wards).

| Rule | Check | Description |
|------|-------|-------------|
| B1 | Ward Code — Boundary Reference | `ward_code` must exist in the admin ward boundary reference dataset |
| B2 | Coordinates — Within Ward Boundary | `latitude`/`longitude` must fall within the bounding box of the declared `ward_code` |

**Performance optimisation:** The boundary search is pre-filtered by `state_code` from the uploaded file, reducing the search space to only the wards within the relevant state(s).

> `lga_code` is intentionally excluded from the pre-filter — lga_code formats can differ between the uploaded file and the reference, which previously caused valid ward codes to be incorrectly flagged by B1.

Reference files bundled in the repo:
- `ward_boundary_ref.csv` — 9,410 ward codes with state, LGA, and ward metadata
- `ward_boundary_bbox.csv` — bounding box (min/max lon/lat) per ward code extracted from the admin boundary dataset

---

### 7. MLoS Issues — Longitudinal View

The **MLoS Issues** tab includes a longitudinal (wide-format) view of all settlement rows that failed at least one rule:

- Each row represents one settlement
- Each QC rule appears as a column (e.g. `Rule_6 | Security Compromised Y/N`)
- **Yes** = error present on that row for that rule
- **No** = no error on that rule for that row
- Click **Download MLoS Issues — Longitudinal (Excel)** to export the workbook

---

### 8. Generate & Download Report

Go to the **Generate Report** tab to:

1. See the overall QC verdict: `✅ CLEAN` or `❌ FAILING`
2. Click **Generate Report** to produce a detailed Excel workbook
3. Download and share with the data team or programme leads

The report file is named: `{filename}_QC_Report.xlsx`

---

### 9. Send QC Email

Click **Send QC Email** in the Generate Report tab to notify the data team.

| Field | Value |
|-------|-------|
| **To** | adanna.alex@ehealthnigeria.org |
| **CC** | fashoto.busayo@ehealthnigeria.org, victor.idakwo@ehealthnigeria.org, oluwadamilare.akindipe@ehealthnigeria.org |
| **Subject** | MLoS QC checks for `{filename}` |
| **Body** | Full check-by-check summary with verdict, issue counts, and missing columns |

SMTP credentials must be configured in Streamlit secrets:

```toml
smtp_host = "smtp.office365.com"   # or smtp.gmail.com for Gmail
smtp_port = 587
smtp_user = "your@email.com"
smtp_pass = "your-app-password"
```

> **Note:** `smtp_host` must be the mail server hostname — not an email address.

---

## How to Run Locally

```bash
pip install -r requirements.txt
streamlit run app.py
```

## Streamlit Cloud

1. Fork or clone this repo
2. Go to [share.streamlit.io](https://share.streamlit.io)
3. Connect your GitHub repo
4. Set **Main file path** to `app.py`
5. Add SMTP credentials under **App secrets**
6. Deploy!

---

## Built By
eHealth Africa
