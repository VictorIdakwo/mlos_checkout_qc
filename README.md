# MLOS Checkout QC

A Streamlit app for running automated Quality Control (QC) checks on MLOS (Master List of Settlements) checkout files across **4 QC layers**: schema alignment, MLoS data integrity, takeoffpoint cross-checks, and admin boundary validation.

---

## Features

- Upload `.sqlite`, `.csv`, `.xlsx`, or `.xls` checkout files
- **🔧 Auto Correct** runs before every QC — fixes common data issues and exports a corrected file
- **Step-by-step progress bar** displayed during QC run
- **4 QC Layers** run automatically on upload:
  - 🔎 **Schema Alignment** — verifies all required columns are present
  - 🏘️ **MLoS Rules** — data integrity and cross-table checks
  - 📍 **Takeoffpoint Rules** — 4 cross-table consistency checks
  - 🗺️ **Boundary Checks** — null ward code detection, ward code existence, coordinate, and state name validation against 9,410-ward admin boundary reference
- **Pass Rate %, Fail Rate %, and 🏆 Weighted QC Score** displayed on the dashboard
- Per-rule issue drilldown with expandable row-level detail tables
- **MLoS Issues — Longitudinal View** — one row per settlement, Yes/No per rule column, downloadable
- **Boundary Issues** — includes reference ward code, ward name, LGA, and state columns for comparison
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
| 🔧 Auto Correct | Correction log + full corrected MLoS download (always available) |
| 📊 QC Summary | Weighted QC Score breakdown, pass/fail per rule, failing row counts, Pass Rate %, Fail Rate % |
| 🏘️ MLoS Issues | Row-level drilldown per failing rule + Longitudinal View (Yes/No per rule) |
| 📍 Takeoffpoint Issues | Row-level drilldown for each failing takeoffpoint rule + download |
| 🗺️ Boundary Issues | Null/missing ward codes (B0), unmatched ward codes (B1), out-of-boundary coordinates (B2), state name mismatches (B3), with boundary reference comparison columns |
| 🔍 Raw Data | Filterable view of the full MLoS and Takeoffpoint datasets |
| 📄 Generate Report | Full QC verdict, 7-sheet Excel report download, and Send Email button |

---

## Process & Procedure

### 1. Upload Your File

Use the **sidebar uploader** to upload your MLOS checkout file. The tool accepts `.sqlite`, `.db`, `.csv`, `.xlsx`, and `.xls` files.

Once uploaded, the app:
1. Runs **Auto Correct** on the MLoS data (pre-step before QC)
2. Runs all 4 QC layers on the corrected data in sequence

A **labelled progress bar** tracks each step:

| Step | Progress | Layer |
|------|----------|-------|
| Pre-step | — | 🔧 Auto Correct |
| Step 1 / 4 | 5% → 25% | 🔎 Schema Alignment |
| Step 2 / 4 | 26% → 50% | 🏘️ MLoS Rules |
| Step 3 / 4 | 51% → 75% | 📍 Takeoffpoint Rules |
| Step 4 / 4 | 76% → 100% | 🗺️ Boundary Checks |

---

### 2. Auto Correct (Pre-step)

Auto Correct runs automatically **before** every QC. The corrected MLoS data is used for all 4 QC layers.

| # | Field(s) | Correction |
|---|----------|------------|
| 1 | `highrisk`, `slums`, `densely_populated`, `hard2reach`, `border`, `normadic`, `riverine`, `fulani` | NULL → `NA` |
| 2 | `scattered` | NULL or `NA` → `N` |
| 3 | `reasons_for_inaccessibility` | NULL → `NA` where `accessibility_status` is `Fully Accessible` |
| 4 | `source` | NULL or empty → `IE` |
| 5 | `globalid` | All `{` `}` stripped; any still-invalid UUID replaced with a fresh generated UUID |

The **🔧 Auto Correct tab** shows the correction log and a **⬇️ Download Full MLoS — Auto Corrected (Excel)** button (always visible).

---

### 3. Schema Alignment (Rules S1–S2)

Checks whether the uploaded file contains all required columns. Schema failures are reported without stopping the remaining checks.

#### Required MLoS Columns (41)

| Column | Description | Used In |
|--------|-------------|---------|
| `state_code` | 2-letter state code (e.g. `NA`) | Base identifier |
| `state_name` | Full state name (e.g. `Nasarawa`) | Base identifier, B1, B3 |
| `lga_code` | LGA numeric code | Base identifier |
| `lga_name` | LGA name | Base identifier |
| `ward_name` | Ward name | Base identifier |
| `ward_code` | Ward code | Rules 4, B1, B2, B3 |
| `takeoffpoint` | Takeoffpoint name | Rules 2, TP2 |
| `takeoffpoint_code` | Takeoffpoint code | Rules 3, TP3 |
| `settlement_name` | Settlement name | Base identifier |
| `primarysettlement_name` | Primary settlement name | Nullable (can be null) |
| `alternate_name` | Alternate settlement name | Nullable (can be null) |
| `latitude` | GPS latitude | Rule B2 |
| `longitude` | GPS longitude | Rule B2 |
| `security_compromised` | Y or N | Rule 6 |
| `accessibility_status` | Fully Accessible / Partially Accessible / Inaccessible | Rules 7, 8 |
| `reasons_for_inaccessibility` | Reason text | Rule 8, Auto Correct (Nullable — can be null for Fully Accessible settlements) |
| `habitational_status` | Abandoned / Migrated / Inhabited / Partially Inhabited | Rule 9 |
| `set_population` | Total settlement population | Rules 10, 11 |
| `set_target` | Target count | Rule 10 |
| `number_of_houses` | House count | Rule 11 |
| `noncompliant_household` | Non-compliant household count | Base field |
| `team_code` | Team code (varchar) | Base field |
| `day_of_activity` | Day of activity | Base field |
| `urban` | Y or N | Rule 13 |
| `rural` | Y or N | Rule 13 |
| `scattered` | Y or N | Rule 13, Auto Correct |
| `highrisk` | Y, N, or NA | Rule 14 |
| `slums` | Y, N, or NA | Rule 14 |
| `densely_populated` | Y, N, or NA | Rule 14 |
| `hard2reach` | Y, N, or NA | Rule 14 |
| `border` | Y, N, or NA | Rule 14 |
| `normadic` | Y, N, or NA | Rule 14 |
| `riverine` | Y, N, or NA | Rule 14 |
| `fulani` | Y, N, or NA | Rule 14 |
| `timestamp` | Record timestamp | Metadata |
| `last_updated` | Last update timestamp | Metadata |
| `source` | Data source | Auto Correct (→ `IE` if empty) |
| `editor` | Editor username | Rule 16 |
| `globalid` | Record UUID | Rule 17, Auto Correct |
| `fc_globalid` | Feature class UUID | Base field |
| `settlementarea_globalid` | Settlement area UUID | Base field |

#### Required Takeoffpoint Columns (4)

| Column | Description | Used In |
|--------|-------------|---------|
| `name` | Takeoffpoint name | Rule TP2 |
| `code` | Takeoffpoint code | Rule TP3 |
| `wardcode` | Ward code | Rule TP4 |
| `globalid` | Record UUID | Rule TP5 |

---

### 4. MLoS QC Checks

Data integrity rules applied to the MLoS table.

| Rule | Check | Description |
|------|-------|-------------|
| 2 | Takeoffpoint Name Match | `takeoffpoint` must match `name` in the Takeoffpoint table |
| 3 | Takeoffpoint Code Match | `takeoffpoint_code` must match `code` in the Takeoffpoint table |
| 4 | Ward Code Match | `ward_code` must match `wardcode` in the Takeoffpoint table |
| 6 | Security Compromised Y/N | `security_compromised` must be `Y` or `N` |
| 7 | Accessibility Status Valid | Must be: `Fully Accessible`, `Partially Accessible`, or `Inaccessible` |
| 8 | Reason for Inaccessibility Required | Partially/Inaccessible settlements must have a reason |
| 9 | Habitational Status Valid | Must be: `Abandoned`, `Migrated`, `Inhabited`, or `Partially Inhabited` |
| 10 | set_target ≤ set_population | `set_target` must not exceed `set_population` |
| 11 | number_of_houses ≤ set_population | `number_of_houses` must not exceed `set_population` |
| 13 | Urban / Rural / Scattered Y/N | Each must be `Y` or `N`; cannot be both Urban and Rural, or Urban and Scattered |
| 14 | Profile Flags Y/N/NA | `highrisk`, `slums`, `densely_populated`, `hard2reach`, `border`, `normadic`, `riverine`, `fulani` must be `Y`, `N`, or `NA` |
| 16 | Editor Format | `editor` must follow the format `firstname.surname` (all lowercase) |
| 17 | GlobalID is UUID | `globalid` must be a valid UUID (`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`) |

> Rules 5 (No Null in Required Fields), 12 (Day of Activity), 14t (team_code numeric), and 15 (Source = MLoS) have been removed. Rule 5 was removed as null checks are not enforced — `primarysettlement_name`, `alternate_name`, and `reasons_for_inaccessibility` are explicitly nullable; Rule 12 was dropped as a QC requirement; Rule 14t was removed as `team_code` is a varchar field; Rule 15 is handled by Auto Correct (`source` NULL → `IE`).

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

### 6. Boundary Checks (Rules B0–B3)

Spatial and reference validation against the admin ward boundary dataset (9,410 wards).

| Rule | Check | Description |
|------|-------|-------------|
| B0 | Ward Code — Not Available on Data | `ward_code` is null or empty in the uploaded data |
| B1 | Ward Code — Boundary Reference | `ward_code` must exist in the boundary reference for the file's state(s) (only checked for rows with a non-null ward_code) |
| B2 | Coordinates — Within Ward Boundary | `latitude`/`longitude` must fall within the bounding box of the declared `ward_code` |
| B3 | State Name — Boundary Reference Match | `state_name` in MLoS must match the `state_name` the boundary reference assigns to the same `ward_code` |

**State filtering:** The boundary reference is pre-filtered by `state_name` (e.g. "Nasarawa", "Kano") matched case-insensitively against the uploaded file's `state_name` column. Fallback to the full 9,410-ward reference if no match is found.

> `state_code` is not used for filtering — the 2-letter code `"NA"` (Nasarawa) is silently converted to `NaN` by pandas when reading from SQLite, making the filter unreliable.

**Boundary Issues tab** includes reference comparison columns:

| Extra Column | Meaning |
|-------------|---------|
| `Ref Ward Code` | Ward code as it appears in the boundary reference |
| `Ref Ward Name` | Ward name from the reference |
| `Ref LGA Code` / `Ref LGA Name` | LGA from the reference |
| `Ref State Code` | State from the reference |
| `In Boundary Reference` | `Yes` or `No — not found in reference` |

Reference files bundled in the repo:
- `ward_boundary_ref.csv` — 9,410 ward codes with state, LGA, and ward metadata
- `ward_boundary_bbox.csv` — bounding box (min/max lon/lat) per ward code

---

### 7. Weighted QC Score

After all 4 layers run, the app calculates a **Weighted QC Score** that reflects the relative importance of each layer:

| QC Layer | Weight |
|----------|--------|
| 🔎 Schema Alignment | 10% |
| 🏘️ MLoS Rules | 50% |
| 📍 Takeoffpoint Rules | 30% |
| 🗺️ Boundary Checks | 10% |

**How it's calculated:**

For each layer, the *layer pass rate* = number of passing rules ÷ total rules in that layer. The weighted contribution = layer pass rate × layer weight × 100.

```
Weighted Score = (Schema Pass Rate × 10%) + (MLoS Pass Rate × 50%)
               + (TP Pass Rate × 30%)      + (Boundary Pass Rate × 10%)
```

> If a layer has no applicable checks (e.g. Takeoffpoint rules skipped for CSV uploads), that layer receives **full credit** (100% pass rate) so the score is not unfairly penalised.

The score and a breakdown table are shown in the **📊 QC Summary** tab. The 🏆 score also appears in the top metric bar on the dashboard.

| Score Range | Interpretation |
|-------------|---------------|
| 80% – 100% | Green — file is in good shape |
| 60% – 79% | Amber — notable issues to review |
| 0% – 59% | Red — significant data quality problems |

---

### 8. MLoS Issues — Longitudinal View

The **MLoS Issues** tab includes a longitudinal (wide-format) view of all settlement rows that failed at least one rule:

- Each row represents one settlement
- Each QC rule appears as a column (e.g. `Rule_6 | Security Compromised Y/N`)
- **Yes** = error present on that row for that rule
- **No** = no error on that rule for that row
- Click **Download MLoS Issues — Longitudinal (Excel)** to export the workbook

---

### 9. Generate & Download Report

Go to the **Generate Report** tab to:

1. See the overall QC verdict: `✅ CLEAN` or `❌ FAILING`
2. Click **Generate Report** to produce a detailed Excel workbook
3. Download and share with the data team or programme leads

The report file is named: `{filename}_QC_Report.xlsx`

---

### 10. Send QC Email

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

> `smtp_host` must be the mail server hostname — **not** an email address.

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
