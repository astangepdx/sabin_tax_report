> **Claude:** to re-create this analysis from scratch, follow the step-by-step instructions in [CLAUDE.md](CLAUDE.md).

# PPS property tax in the Sabin catchment area, 2021-2025

How much property tax do the homes in the Portland Public Schools Sabin Elementary (K-5) catchment area pay to PPS, and how has it changed?
This repo reports the answer for tax years 2021 through 2025 using the Multnomah County tax statement for every residence in the area.
The scripts and the address-level records live in a separate private repo (`sabin_tax_records`); this one holds the findings and the aggregate data.

**In short:** the 2,729 residences in the area were billed **$6.65 million** for PPS in tax year 2025, up from **$5.44 million** in 2021.
That is **+22.3% over four years, or 5.2% a year**, compared with 3.5% to 4.4% a year for the main US inflation measures over the same span.

## 1. The area

![Sabin catchment area, with tax lots as dots](map/sabin_catchment_map.png)

The boundary is the PPS Sabin K-5 attendance zone (PPS school code 279), taken from the PPS attendance-zone map layer.
An interactive version of this map is in [`map/sabin_catchment_map.html`](map/sabin_catchment_map.html). Open the file in a browser, or, if GitHub Pages is enabled for this repo, at `https://astangepdx.github.io/sabin_tax_report/map/sabin_catchment_map.html`.
The boundary as GeoJSON is [`data/boundary_sabin_k5.geojson`](data/boundary_sabin_k5.geojson). The map shows lots only as dots and carries no street addresses.

## 2. Properties counted, and where the edges are fuzzy

A tax lot is counted as inside the area when the lot's center point falls inside the boundary polygon.
Of the 2,857 lots inside, **2,729 (95.5%) are residences** (county use code B single-family, X row/attached housing, or C 2-4 unit buildings, each with a street number) and **128 (4.5%)** are other lots: apartments over four units, commercial, vacant, parks, schools and similar.

| Item | Count | Share |
|---|---|---|
| Lots with center inside the boundary | 2,857 | 100% |
| Residences (included in all tax figures) | 2,729 | 95.5% of lots |
| &nbsp;&nbsp;single-family (B) / row-attached (X) / 2-4 unit (C) | 2,547 / 90 / 92 | 93.3% / 3.3% / 3.4% of residences |
| Other lots (excluded) | 128 | 4.5% of lots |
| &nbsp;&nbsp;of which carry a residential use code but no street number | 16 | 0.6% of lots |
| Lots that touch the boundary but have their center outside (not counted) | 43 | 1.5% of the 2,900 lots touching the area |
| Residences flagged by the odd/even street-side check | 10 | 0.37% of residences |
| Residences with a tax statement missing for at least one year | 15 | 0.55% of residences |
| Statements missing (of 13,645 expected = 2,729 x 5 years) | 55 | 0.40% of statements |
| Statements showing $0 to PPS | 173 | 1.3% of the 13,590 statements found |

What the discrepancies are:

- **Street-side check.** Many boundary streets split the area down the middle (one side of the street is in, the other out). Odd and even house numbers are normally on opposite sides, so each lot's side was cross-checked against its house-number parity. 62 lots disagree: 32 inside the boundary (10 residences and 22 non-residential) and 30 outside. The polygon decides membership; the flags are a review list, and the effect is at most 10 of 2,729 residences (0.37%).
- **Missing statements.** 15 properties have no county statement for some years. Twelve of them have statements only from tax year 2024 or 2025 on (their account numbers are among the highest, so they look like recently created accounts), 2 have no statement for any of the five years, and 1 lacks only 2025. That is why the number of residences billed grows from 2,715 in 2021 to 2,726 in 2025.
- **$0 statements.** 173 statements show no PPS tax. These are probably exempt properties (not individually verified).
- **Shared-lot units.** The lot layer carries one record per tax lot. Condominium or attached units that share a lot but have separate tax accounts may be missing, and a 2-4 unit lot counts as one property.
- **Street naming.** The county's address data calls one short block NE 14th Ave; the PPS School Finder map labels it NE 14th Pl. This does not change which lots are inside.

## 3. PPS tax by year

![PPS property tax by tax year](charts/sabin_pps_tax_by_year.png)

Each bar is the sum, across all residences with a statement that year, of one line of the county tax statement. Amounts are the tax **billed, before the 3% prompt-pay discount** (the discount reduces what a payer remits but not the funding PPS is levied). Tax year 2025 is the 2025-26 bill.

| Tax year | Properties | PPS | Local Option | Bonds | Total |
|---|---|---|---|---|---|
| 2021 | 2,715 | $2,894,001 | $1,203,899 | $1,343,442 | $5,441,343 |
| 2022 | 2,715 | $3,022,332 | $1,249,061 | $1,384,346 | $5,655,740 |
| 2023 | 2,715 | $3,314,309 | $1,291,451 | $1,501,400 | $6,107,159 |
| 2024 | 2,719 | $3,477,950 | $1,336,211 | $1,539,484 | $6,353,645 |
| 2025 | 2,726 | $3,573,594 | $1,383,157 | $1,696,338 | $6,653,089 |

Source file: [`data/pps_tax_by_year.csv`](data/pps_tax_by_year.csv). The three PPS lines on a statement are "Portland Public Schools" (general), "Portland Public Schools Local Opt" (local option levy) and "Portland Public Schools Bonds".

## 4. Annual change, and how it compares with inflation

Total PPS tax in the area grew **22.3% from 2021 to 2025, a compound 5.15% a year** (four annual steps).

| Measure | Total change 2021 to 2025 | Per year (compound) |
|---|---|---|
| **Sabin PPS tax total** | **+22.3%** | **5.15%** |
| US CPI-U, annual average | +18.7% | 4.38% |
| US CPI-U, Dec to Dec | +16.4% | 3.86% |
| West Region CPI-U, Dec to Dec | +16.1% | 3.80% |
| US PCE price index, annual average | +16.5% | 3.89% |
| US PCE price index, Dec to Dec | +14.6% | 3.47% |

| Year over year | 2022 | 2023 | 2024 | 2025 |
|---|---|---|---|---|
| Sabin PPS tax total | +3.9% | +8.0% | +4.0% | +4.7% |
| US CPI-U, Dec to Dec | +6.5% | +3.4% | +2.9% | +2.7% |
| West Region CPI-U, Dec to Dec | +6.2% | +3.6% | +2.5% | +2.9% |
| US PCE price index, Dec to Dec | +5.6% | +2.9% | +2.6% | +2.8% |

Files: [`data/pps_tax_vs_inflation.csv`](data/pps_tax_vs_inflation.csv), [`data/inflation_indices.csv`](data/inflation_indices.csv).
Portland has no local CPI: BLS discontinued the Portland-Salem index after 2017, so the West Region CPI-U (13 states) is the closest local measure.

## 5. Commentary

- **Faster than inflation in total.** The area's PPS tax grew about 0.8 to 1.7 percentage points a year faster than the US inflation measures above. Inflation was front-loaded (2022 was the peak) while the tax growth was steadier, with one large step in 2023.
- **The components moved differently.** Over 2021-2025 the general PPS line grew 23.5% (5.4% a year), Local Option 14.9% (3.5% a year) and Bonds 26.3% (6.0% a year). Local Option rose by 3.4% to 3.8% every year; Bonds jumped 8.5% in 2023 and 10.2% in 2025; the general line jumped 9.7% in 2023.
- **Almost all of the growth is per property.** The number of residences billed grew only 0.4% (2,715 to 2,726), so the growth in dollars is almost entirely higher bills on the same lots.
- **What this data cannot say.** The statements give amounts, not reasons. Possible drivers that were not tested here include changes in assessed values, tax rates for the general fund and for voter-approved bonds and the local option, and new construction. Treat any explanation as a hypothesis.
- **Comparison caveats.** CPI and PCE measure consumer prices for calendar years; the tax is dollars billed to a fixed set of lots for tax years that run July to June. The comparison shows direction and rough size, not a like-for-like index.

## Data files

| File | Contents |
|---|---|
| `data/pps_tax_by_year.csv` | Yearly totals: properties billed, PPS, Local Option, Bonds, Total |
| `data/dataset_stats.json` | The counts used in section 2 |
| `data/boundary_sabin_k5.geojson` | The Sabin K-5 attendance-zone polygon |
| `data/inflation_indices.csv` | US CPI-U, West Region CPI-U and PCE inputs (and derived rates) |
| `data/pps_tax_vs_inflation.csv` | The comparison tables in section 4 |
| `charts/sabin_pps_tax_by_year.png` | The grouped bar chart |
| `map/sabin_catchment_map.html`, `.png` | Interactive and static map |

## Sources

- PPS attendance-zone layer: Portland Public Schools ArcGIS feature service (Sabin, K-5, `facilityid` 279).
- Tax lots: PortlandMaps Regional Taxlots (`Public/Taxlots/MapServer/0`).
- Tax statements: [Multnomah County property tax search](https://multcoproptax.com/).
- US CPI-U rates: [usinflationcalculator.com table of BLS data](https://www.usinflationcalculator.com/inflation/current-inflation-rates/).
- West Region CPI-U: [FRED series CUUR0400SA0](https://fred.stlouisfed.org/data/CUUR0400SA0.txt) (BLS).
- PCE price index: [FRED series PCEPI](https://fred.stlouisfed.org/data/PCEPI.txt) (BEA).
- Portland-Salem CPI (discontinued): [BLS table](https://www.bls.gov/regions/west/data/consumerpriceindex_portland_table.pdf).

Annual rates for West Region CPI-U and PCE, and all compound rates, are calculated from the index values.
October 2025 CPI data were not collected during the 2025 federal funding lapse; the US CPI-U 2025 annual-average figure is as published in the source table, and the December-to-December rates are not affected.
