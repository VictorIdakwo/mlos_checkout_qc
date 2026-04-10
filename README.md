# MLOS Checkout QC

A Streamlit app for running Quality Control (QC) checks on MLOS (Master List of Settlements) checkout SQLite files.

## Features

- Upload any `.sqlite` checkout file
- Automated QC checks on:
  - **MLoS layer** (`master_list_settlement_update_view`) — 15+ rules
  - **Takeoffpoint layer** (`mlos_takeoffpoint_view`) — 4 rules
- Per-rule issue drilldown with expandable tables
- Filterable raw data view
- **Generate Report tab** — full QC verdict + downloadable 7-sheet Excel report

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

## QC Rules

### MLoS Table
| Rule | Description |
|------|-------------|
| 2 | takeoffpoint must match takeoffpoint.name |
| 3 | takeoffpoint_code must match takeoffpoint.code |
| 4 | ward_code must match takeoffpoint.wardcode |
| 5 | Required fields must not be null |
| 6 | security_compromised = Y or N |
| 7 | accessibility_status valid value |
| 8 | Partially/Inaccessible must have reason |
| 9 | habitational_status valid value |
| 10 | set_target & number_of_houses ≤ set_population |
| 12 | day_of_activity valid code |
| 13 | urban/rural/scattered = Y or N |
| 14 | Profile flags = Y/N/NA |
| 15 | source = MLoS |
| 16 | editor = firstname.surname (lowercase) |
| 17 | globalid = valid UUID |

### Takeoffpoint Table
| Rule | Description |
|------|-------------|
| TP2 | name must match mlos.takeoffpoint |
| TP3 | code must match mlos.takeoffpoint_code |
| TP4 | wardcode must match mlos.ward_code |
| TP5 | globalid = valid UUID |

## Built By
eHealth Africa
