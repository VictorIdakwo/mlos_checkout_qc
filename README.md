# MLOS Checkout QC

A Streamlit app for running automated Quality Control (QC) checks on MLOS (Master List of Settlements) checkout files across **5 QC layers**: schema alignment, MLoS data integrity, settlement spatial checks, takeoffpoint cross-checks, and admin boundary validation.

---

## Features

- Upload `.sqlite`, `.csv`, `.xlsx`, or `.xls` checkout files
- Optional **separate Takeoffpoint file** upload (`.csv` or `.xlsx`) — overrides the built-in sheet
- **▶️ Run QC button** — QC only starts after explicit submission
- **🔧 Auto Correct** runs before every QC — fixes common data issues and exports a corrected file
- **Step-by-step progress bar** displayed during QC run
- **5 QC Layers** run automatically on submission:
  - 🔎 **Schema Alignment** — verifies all required columns are present
  - 🏘️ **MLoS Rules** — data integrity and cross-table checks (Rules 1–14)
  - 📐 **Settlement QC** — duplicate, coordinate, stacking, and proximity checks
  - 📍 **Takeoffpoint Rules** — 4 cross-table consistency checks
  - 🗺️ **Boundary Checks** — null ward code detection, ward code existence, coordinate, and state name validation against 9,410-ward admin boundary reference
- **Pass Rate %, Fail Rate %, and 🏆 Weighted QC Score** displayed on the dashboard
- Per-rule issue drilldown with expandable row-level detail tables
- **MLoS Issues — Longitudinal View** — one row per settlement, Yes/No per rule column, downloadable
- **Boundary Issues** — includes reference ward code, ward name, LGA, and state columns for comparison
- **Generate Report tab** — full QC verdict (CLEAN / FAILING) + downloadable Excel report
- **Send QC Email** — sends summary to the data team on demand

---

## Supported File Formats

| Format | MLoS Data | Takeoffpoint Data |
|--------|-----------|-------------------|
| `.sqlite` / `.db` | `master_list_settlement_update_view` | `mlos_takeoffpoint_view` |
| `.xlsx` / `.xls` | Sheet 1 (or sheet named `mlos`) | Sheet 2 (or sheet named `takeoffpoint`) |
| `.csv` | Entire CSV file | Not available — upload a separate Takeoffpoint file |

> For full QC coverage including takeoffpoint and boundary checks, use `.sqlite` or `.xlsx`.

---

## Tab Overview

| Tab | Contents |
|-----|----------|
| 🔧 Auto Correct | Correction log + full corrected MLoS download (always available) |
| 📊 QC Summary | Weighted QC Score breakdown, pass/fail per rule, failing row counts, Pass Rate %, Fail Rate % |
| 🏘️ MLoS Issues | Row-level drilldown per failing rule + Longitudinal View (Yes/No per rule) |
| 📐 Settlement QC | Row-level drilldown for each failing settlement check (duplicates, coordinates, proximity) + download |
| 📍 Takeoffpoint Issues | Row-level drilldown for each failing takeoffpoint rule + download |
| 🗺️ Boundary Issues | Null/missing ward codes (B0), unmatched ward codes (B1), out-of-boundary coordinates (B2), state name mismatches (B3), with boundary reference comparison columns |
| 🔍 Raw Data | Filterable view of the full MLoS and Takeoffpoint datasets |
| 📄 Generate Report | Full QC verdict, Excel report download, and Send Email button |

---

## Process & Procedure

### 1. Upload Your File

Use the **sidebar uploader** to upload your MLOS checkout file. The tool accepts `.sqlite`, `.db`, `.csv`, `.xlsx`, and `.xls` files.

Optionally upload a separate **Takeoffpoint file** (`.csv` or `.xlsx`). This overrides the built-in Takeoffpoint sheet for Excel/SQLite files, and enables takeoffpoint cross-checks for CSV uploads.

Click **▶️ Run QC** in the sidebar to start. QC does not run until the button is clicked.

Once submitted, the app:
1. Runs **Auto Correct** on the MLoS data (pre-step before QC)
2. Runs all 5 QC layers on the corrected data in sequence

A **labelled progress bar** tracks each step:

| Step | Progress | Layer |
|------|----------|-------|
| Pre-step | — | 🔧 Auto Correct |
| Step 1 / 5 | 5% → 20% | 🔎 Schema Alignment |
| Step 2 / 5 | 21% → 40% | 🏘️ MLoS Rules |
| Step 3 / 5 | 41% → 60% | 📐 Settlement QC |
| Step 4 / 5 | 61% → 80% | 📍 Takeoffpoint Rules |
| Step 5 / 5 | 81% → 100% | 🗺️ Boundary Checks |

---

### 2. Auto Correct (Pre-step)

Auto Correct runs automatically **before** every QC. The corrected MLoS data is used for all 5 QC layers.

| # | Field(s) | Correction |
|---|----------|------------|
| 1 | `highrisk`, `slums`, `densely_populated`, `hard2reach`, `border`, `nomadic`, `riverine`, `fulani` | NULL → `NA` |
| 2 | `scattered` | NULL or `NA` → `N` |
| 3 | `reasons_for_inaccessibility` | NULL → `NA` where `accessibility_status` is `Fully Accessible` |
| 4 | `source` | NULL or empty → `IE` |
| 5 | `eha_guid` | All `{` `}` stripped; any still-invalid UUID replaced with a fresh generated UUID |

The **🔧 Auto Correct tab** shows the correction log and a **⬇️ Download Full MLoS — Auto Corrected (Excel)** button (always visible).

---

### 3. Schema Alignment (Rules S1–S2)

Checks whether the uploaded file contains all required columns. Schema failures are reported without stopping the remaining checks.

#### Required MLoS Columns (43)

| Column | Description | Used In |
|--------|-------------|---------|
| `state_code` | 2-letter state code (e.g. `NA`) | Base identifier, Rule 1 |
| `state_name` | Full state name (e.g. `Nasarawa`) | Base identifier, Rule 1, B1, B3 |
| `lga_code` | LGA numeric code | Base identifier, Rule 1 |
| `lga_name` | LGA name | Base identifier, Rule 1 |
| `ward_name` | Ward name | Base identifier, Rule 1 |
| `ward_code` | Ward code | Rules 1, 4, B0, B1, B2, B3 |
| `takeoffpoint` | Takeoffpoint name | Rules 2, TP2 |
| `takeoffpoint_code` | Takeoffpoint code | Rules 3, TP3 |
| `settlement_name` | Settlement name | Base identifier, Rule 1 |
| `primarysettlement_name` | Primary settlement name | Nullable (can be null) |
| `alternate_name` | Alternate settlement name | Nullable (can be null) |
| `latitude` | GPS latitude | Rules 1, SQ2, SQ2b, SQ3, SQ4, B2 |
| `longitude` | GPS longitude | Rules 1, SQ2, SQ2b, SQ3, SQ4, B2 |
| `security_compromised` | Y or N | Rules 1, 5 |
| `accessibility_status` | Fully Accessible / Partially Accessible / Inaccessible | Rules 1, 6, 7 |
| `reasons_for_inaccessibility` | Reason text | Rule 7, Auto Correct (Nullable — can be null for Fully Accessible settlements) |
| `habitational_status` | Abandoned / Migrated / Inhabited / Partially Inhabited | Rule 8 (Nullable) |
| `set_population` | Total settlement population | Rules 1, 9, 10 |
| `set_target` | Target count | Rules 1, 9 |
| `number_of_household` | Household count | Rules 1, 10 |
| `noncompliant_household` | Non-compliant household count | Base field (Nullable) |
| `team_code` | Team code (varchar) | Base field (Nullable) |
| `day_of_activity` | Day of activity | Base field (Nullable) |
| `urban` | Y or N | Rules 1, 11 |
| `rural` | Y or N | Rules 1, 11 |
| `scattered` | Y or N | Rules 1, 11, Auto Correct |
| `highrisk` | Y, N, or NA | Rules 1, 12 |
| `slums` | Y, N, or NA | Rules 1, 12 |
| `densely_populated` | Y, N, or NA | Rules 1, 12 |
| `hard2reach` | Y, N, or NA | Rules 1, 12 |
| `border` | Y, N, or NA | Rules 1, 12 |
| `nomadic` | Y, N, or NA | Rules 1, 12 |
| `riverine` | Y, N, or NA | Rules 1, 12 |
| `fulani` | Y, N, or NA | Rules 1, 12 |
| `timestamp` | Record timestamp | Metadata, Rule 1 |
| `last_updated` | Last update timestamp | Metadata (Nullable) |
| `source` | Data source | Auto Correct (→ `IE` if empty) (Nullable) |
| `editor` | Editor username | Rule 13 (Nullable) |
| `validation_status` | Validation status | Base field (Nullable) |
| `master_id` | Master record ID | Base field (Nullable) |
| `mlos_id` | MLoS record ID | Base field (Nullable) |
| `eha_guid` | Record UUID | Rules 1, 14, Auto Correct |
| `settlementarea_globalid` | Settlement area UUID | Rule 1 |

#### Required Takeoffpoint Columns (4)

| Column | Description | Used In |
|--------|-------------|---------|
| `name` | Takeoffpoint name | Rule TP2 |
| `code` | Takeoffpoint code | Rule TP3 |
| `wardcode` | Ward code | Rule TP4 |
| `globalid` | Record UUID | Rule TP5 |

---

### 4. MLoS QC Checks (Rules 1–14)

Data integrity rules applied to the MLoS table. Rules are numbered sequentially 1–14.

| Rule | Check | Description |
|------|-------|-------------|
| 1 | Required Fields — Not Null | All 28 schema `NOT NULL` columns must not be empty. The detail table includes a **Null Fields** column listing exactly which column(s) are null for each failing row. Covered fields: `state_code`, `state_name`, `lga_code`, `lga_name`, `ward_name`, `ward_code`, `settlement_name`, `latitude`, `longitude`, `security_compromised`, `accessibility_status`, `set_population`, `set_target`, `number_of_household`, `urban`, `rural`, `scattered`, `highrisk`, `slums`, `densely_populated`, `hard2reach`, `border`, `nomadic`, `riverine`, `fulani`, `timestamp`, `eha_guid`, `settlementarea_globalid` |
| 2 | Takeoffpoint Name Match | `takeoffpoint` must match `name` in the Takeoffpoint table |
| 3 | Takeoffpoint Code Match | `takeoffpoint_code` must match `code` in the Takeoffpoint table |
| 4 | Ward Code Match | `ward_code` must match `wardcode` in the Takeoffpoint table |
| 5 | Security Compromised Y/N | `security_compromised` must be `Y` or `N` |
| 6 | Accessibility Status Valid | Must be: `Fully Accessible`, `Partially Accessible`, or `Inaccessible` |
| 7 | Reason for Inaccessibility Required | Partially/Inaccessible settlements must have a reason |
| 8 | Habitational Status Valid | Must be: `Abandoned`, `Migrated`, `Inhabited`, or `Partially Inhabited` |
| 9 | set_target ≤ set_population | `set_target` must not exceed `set_population` |
| 10 | number_of_household ≤ set_population | `number_of_household` must not exceed `set_population` |
| 11 | Urban / Rural / Scattered Y/N | Each must be `Y` or `N`; cannot be both Urban and Rural (11a), or Urban and Scattered (11b) |
| 12 | Profile Flags Y/N/NA | `highrisk`, `slums`, `densely_populated`, `hard2reach`, `border`, `nomadic`, `riverine`, `fulani` must be `Y`, `N`, or `NA` |
| 13 | Editor Format | `editor` must follow the format `firstname.surname` (all lowercase) |
| 14 | eha_guid is UUID | `eha_guid` must be a valid UUID (`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`) |

**Nullable fields** (schema allows NULL — not subject to Rule 1): `primarysettlement_name`, `alternate_name`, `reasons_for_inaccessibility`, `habitational_status`, `noncompliant_household`, `team_code`, `day_of_activity`, `source`, `last_updated`, `editor`, `validation_status`, `master_id`, `mlos_id`.

> Previously removed rules: Day of Activity (was R12), team_code numeric (was R14t) — `team_code` is varchar; Source = MLoS (was R15) — handled by Auto Correct (`source` NULL → `IE`).

---

### 5. Settlement QC Checks (Rules SQ1–SQ4)

Spatial integrity checks applied to the MLoS settlement records.

| Rule | Check | Description |
|------|-------|-------------|
| SQ1 | Duplicate Settlement Name in Ward | `settlement_name` must not repeat within the same `ward_code` |
| SQ2 | Latitude/Longitude — Not Null | `latitude` and `longitude` must not be null or missing |
| SQ2b | Latitude/Longitude — Not Zero | `latitude` and `longitude` must not be zero (checked only on non-null rows) |
| SQ3 | Stacked Coordinates | No two settlements may share identical `latitude`/`longitude` coordinates |
| SQ4 | Settlements Too Close (< 30 m) | Every settlement must be more than 30 metres from all other settlements (Haversine distance) |

---

### 6. Takeoffpoint QC Checks (Rules TP2–TP5)

Cross-table consistency checks between the Takeoffpoint and MLoS tables.

| Rule | Check | Description |
|------|-------|-------------|
| TP2 | Name matches MLoS | `name` must match `takeoffpoint` in the MLoS table |
| TP3 | Code matches MLoS | `code` must match `takeoffpoint_code` in the MLoS table |
| TP4 | Ward Code matches MLoS | `wardcode` must match `ward_code` in the MLoS table |
| TP5 | GlobalID is UUID | `globalid` must be a valid UUID |

> If a CSV file is uploaded without a separate Takeoffpoint file, rules TP2–TP5 are skipped with a warning.

---

### 7. Boundary Checks (Rules B0–B3)

Spatial and reference validation against the admin ward boundary dataset (9,410 wards).

| Rule | Check | Description |
|------|-------|-------------|
| B0 | Ward Code — Not Available on Data | `ward_code` is null or empty in the uploaded data |
| B1 | Ward Code — Boundary Reference | `ward_code` must exist in the boundary reference for the file's state(s) (only checked for rows with a non-null ward_code) |
| B2 | Coordinates — Within Ward Boundary | `latitude`/`longitude` must fall within the bounding box of the declared `ward_code` |
| B3 | State Name — Boundary Reference Match | `state_name` in MLoS must match the `state_name` the boundary reference assigns to the same `ward_code` |

**State filtering:** The boundary reference is pre-filtered by `state_name` (e.g. "Nasarawa", "Kano") matched case-insensitively against the uploaded file's `state_name` column. Fallback to the full 9,410-ward reference if no match is found.

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

### 8. Weighted QC Score

After all 5 layers run, the app calculates a **Weighted QC Score** that reflects the relative importance of each layer:

| QC Layer | Weight |
|----------|--------|
| 🔎 Schema Alignment | 10% |
| 🏘️ MLoS Rules | 40% |
| 📐 Settlement QC | 20% |
| 📍 Takeoffpoint Rules | 20% |
| 🗺️ Boundary Checks | 10% |

**How it's calculated:**

For each layer, the *layer pass rate* = number of passing rules ÷ total rules in that layer. The weighted contribution = layer pass rate × layer weight × 100.

```
Weighted Score = (Schema Pass Rate × 10%) + (MLoS Pass Rate × 40%)
               + (Settlement Pass Rate × 20%) + (TP Pass Rate × 20%)
               + (Boundary Pass Rate × 10%)
```

> If a layer has no applicable checks (e.g. Takeoffpoint rules skipped for CSV uploads), that layer receives **full credit** (100% pass rate) so the score is not unfairly penalised.

The score and a breakdown table are shown in the **📊 QC Summary** tab. The 🏆 score also appears in the top metric bar on the dashboard.

| Score Range | Interpretation |
|-------------|---------------|
| 80% – 100% | Green — file is in good shape |
| 60% – 79% | Amber — notable issues to review |
| 0% – 59% | Red — significant data quality problems |

---

### 9. MLoS Issues — Longitudinal View

The **MLoS Issues** tab includes a longitudinal (wide-format) view of all settlement rows that failed at least one rule:

- Each row represents one settlement
- Each QC rule appears as a column (e.g. `Rule_5 | Security Compromised Y/N`)
- **Yes** = error present on that row for that rule
- **No** = no error on that rule for that row
- Click **Download MLoS Issues — Longitudinal (Excel)** to export the workbook

---

### 10. Generate & Download Report

Go to the **Generate Report** tab to:

1. See the overall QC verdict: `✅ CLEAN` or `❌ FAILING`
2. Click **Generate Report** to produce a detailed Excel workbook
3. Download and share with the data team or programme leads

The report file is named: `{filename}_QC_Report.xlsx`

---

### 11. Send QC Email

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
