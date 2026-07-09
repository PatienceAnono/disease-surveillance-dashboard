# Kenya Disease Surveillance Dashboard

**Malaria fell 78% in 22 years. The TB detection gap nearly doubled during COVID. Rwanda — with a lower GDP than Kenya — achieved disease outcomes that the entire region is trying to replicate. This project maps all of it.**

---

## What this is

A full disease surveillance analytics project covering five communicable diseases across Kenya, built on real WHO Global Health Observatory data with a live API connection that auto-refreshes when WHO publishes updated figures. The analysis covers the national disease burden from 2000 to 2022, three documented cholera outbreaks, a composite risk index for all 46 Kenyan counties and an East Africa regional comparison across 10 countries.

I built this as part of my health analytics portfolio because disease surveillance is one of those areas where the data genuinely exists — WHO, IHME, Kenya NLTP, NASCOP all publish it — but almost nobody brings it together into a form that a county health officer or programme manager can actually use. That is what this dashboard does.

---

## The data

**Primary source: WHO Global Health Observatory API**

The API is publicly accessible with no authentication. I built a `fetch_who_gho()` function that pulls any indicator for any country or group of countries and returns a clean pandas DataFrame. The same URL structure works directly in Power BI's OData feed connector, which means the dashboard auto-refreshes when WHO updates its data.

```
Base URL: https://ghoapi.azureedge.net/api
Example: https://ghoapi.azureedge.net/api/MALARIA_EST_INCIDENCE_PER1000?$filter=SpatialDim eq 'KEN'
```

Indicator codes I used:

| Disease | Indicator | Code |
|:---|:---|:---|
| Malaria | Incidence per 1,000 | `MALARIA_EST_INCIDENCE_PER1000` |
| Malaria | Deaths per 100,000 | `MALARIA_EST_DEATHS` |
| TB | Incidence per 100,000 | `MDG_0000000020` |
| TB | Treatment success | `MDG_0000000022` |
| HIV | Prevalence % | `HIVPREV_0000000029` |
| HIV | New infections | `HIV_0000000001` |
| HIV | ART coverage | `HIV_0000000007` |
| Cholera | Cases reported | `CHOLERA_0000000001` |

**Secondary sources:** IHME Global Burden of Disease 2021 (DALY rates), UNAIDS (HIV treatment cascade), Kenya National Leprosy and TB Programme (county notification rates), Kenya NASCOP (HIV county prevalence), Kenya IDSR weekly bulletins (outbreak data).

---

## Five datasets, five analytical angles

| Sheet | What it contains |
|:---|:---|
| `Kenya_Disease_Burden_Trend` | 23 years × 5 diseases × 15 indicators — national annual figures 2000–2022 |
| `Cholera_Outbreak_Detail` | Weekly epidemic curve data for 3 documented outbreaks (2015, 2019, 2022) |
| `East_Africa_Regional` | 10 countries × 13 indicators — 2022 disease snapshot with DALY rates |
| `Kenya_County_Risk_Index` | All 46 counties with malaria endemicity zone, TB notification rate, HIV prevalence and composite risk score |
| `WHO_GHO_API_Reference` | Indicator codes and OData URLs — paste directly into Power BI |

---

## What the data showed

**Malaria — the progress story**

Kenya reduced malaria incidence from 265 per 1,000 population at risk in 2000 to 58 in 2022, a 78% reduction. The steepest decline happened between 2005 and 2015, driven by mass ITN distribution and the national rollout of artemisinin-based combination therapy in 2008. There is a visible post-2019 uptick: COVID disrupted vector control programmes and ITN distribution missed its 2020 targets. This is not noise — it is a real signal that the programme needs watching.

**TB — the detection gap problem**

The gap between TB incidence and TB notifications is the most operationally important number in TB surveillance. In a normal year, Kenya misses roughly 7 cases per 100,000 — people with active TB who go undiagnosed. In 2020, that gap widened dramatically as people stopped presenting to facilities. Undiagnosed TB patients remain infectious. The gap has narrowed since but had not fully recovered to pre-COVID levels by 2022.

**HIV — the ART success**

The inverse relationship between ART coverage and new HIV infections in this dataset is one of the clearest natural experiments I have seen in health data. As ART coverage scaled from effectively zero in 2000 to 83% by 2022, new infections fell from 82,000 per year to 25,000. The causal chain is well understood biologically. Seeing it this clearly in country-level data over 22 years is still striking. At the current trajectory, Kenya will not reach the UNAIDS 2030 target of 15,000 new infections per year.

**County risk index**

I built a composite 0–100 risk score for all 46 counties incorporating malaria endemicity zone, TB notification rate, HIV prevalence and cholera risk classification. Fourteen counties scored above 70. They fell into two distinct clusters with different disease drivers: the Lake Victoria basin (Siaya 82, Kisumu 84, Migori 80) where HIV compounds holoendemic malaria, and the ASAL north (Turkana 82, Mandera 80, Garissa 78) where TB and cholera risk are the primary concerns alongside epidemic-prone malaria.

**Rwanda**

Rwanda's numbers sit in every comparison chart: malaria at 12 per 1,000, TB at 56 per 100,000, ART coverage at 89%. A lower-income country in the same WHO region consistently outperforming its neighbours by margins that cannot be explained by resources alone. It is the most useful benchmark in the dataset.

---

## Charts produced

- `disease_burden_overview.png` — indexed trend for all three diseases (2000=100), ART vs new infections panel, percentage reduction summary
- `malaria_surveillance.png` — incidence/mortality dual axis, endemicity zone box plots, TB-HIV syndemic scatter, estimated cases with uncertainty band
- `tb_hiv_surveillance.png` — TB cascade with detection gap shaded, UNAIDS 95-95-95 treatment cascade, new infections projection, ART scale-up vs PLHIV
- `epidemic_curves.png` — three Kenya cholera outbreak epidemic curves + comparison panel with alert thresholds
- `county_risk_dashboard.png` — all 46 counties ranked by composite risk score, disease risk matrix scatter
- `regional_disease.png` — malaria, TB and HIV comparison across 10 East African countries

---

## Technical approach

```python
# Python stack
pandas · numpy · matplotlib · seaborn · requests

# The API function
fetch_who_gho(indicator_code='MALARIA_EST_INCIDENCE_PER1000',
              countries=['KEN', 'UGA', 'TZA'],
              years=(2000, 2022))
# Returns clean DataFrame — works for any WHO GHO indicator, any countries

# Power BI live connection
# Get Data → OData Feed → paste any URL from the WHO_GHO_API_Reference sheet
# No authentication. Refreshes automatically when WHO updates data.
```

The API function handles connection errors gracefully — if the API is unreachable, it prints an informative message and returns an empty DataFrame so the rest of the notebook continues running from local data.

---

## Project structure

```
disease-surveillance-dashboard/
├── Disease_Surveillance_EDA_v2.ipynb    # Main analysis notebook
├── Disease_Surveillance_Dataset.xlsx    # Five-sheet dataset
│   ├── Kenya_Disease_Burden_Trend       # National 23-year trend
│   ├── Cholera_Outbreak_Detail          # Epidemic curve data
│   ├── East_Africa_Regional             # 10-country comparison
│   ├── Kenya_County_Risk_Index          # 46-county risk scoring
│   └── WHO_GHO_API_Reference            # API codes and OData URLs
├── outputs/
│   ├── disease_burden_overview.png
│   ├── malaria_surveillance.png
│   ├── tb_hiv_surveillance.png
│   ├── epidemic_curves.png
│   ├── county_risk_dashboard.png
│   └── regional_disease.png
└── README.md
```

---

## Running the notebook

```bash
pip install pandas numpy matplotlib seaborn openpyxl requests
jupyter notebook Disease_Surveillance_EDA_v2.ipynb
```

The notebook runs fully offline from the local dataset. If you have internet access, the `fetch_who_gho()` function calls will also return live WHO data. The WHO GHO API requires no registration or API key.

---

## Skills this project demonstrates

- **REST API integration** — live WHO GHO OData API, parameterised queries, error handling, Power BI-ready URL construction
- **Public health data literacy** — WHO indicator codes, IHME GBD methodology, IDSR reporting framework, UNAIDS 95-95-95 targets
- **Composite index construction** — multi-disease county risk scoring across categorical and continuous variables
- **Epidemic curve analysis** — outbreak detection logic, CFR calculation, propagated vs point source transmission interpretation
- **Multi-country data wrangling** — 10-country regional comparison, ISO3 country codes, regional aggregation
- **Dashboard design** — 5-page Power BI structure mapped to operational use cases for county health officers and programme managers

---

## Data sources

- WHO Global Health Observatory: `ghoapi.azureedge.net/api`
- IHME Global Burden of Disease 2021: `vizhub.healthdata.org/gbd-results`
- UNAIDS AIDSinfo: `aidsinfo.unaids.org`
- Kenya National Leprosy and TB Programme Annual Report
- Kenya NASCOP HIV Estimates Report 2022
- Kenya Integrated Disease Surveillance and Response (IDSR) bulletins

---

**Patience Anono** · PA Data Analytics · [padataanalytics.com](https://padataanalytics.com) · hello@padataanalytics.com · @pa_analytics
