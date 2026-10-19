# CLAUDE.md - re-creating the Sabin catchment property tax study

Instructions for Claude (or a person) to reproduce [this report](README.md) for the Sabin K-5 catchment area, or to adapt it to
another PPS elementary school or another year range. Read it all before starting. The numbers in "Expected results" at the end
are what the original run produced, so use them as a sanity check (a different school or year will differ, of course).

## What the study answers

For every residence inside a school's attendance area, read the Portland Public Schools lines of the Multnomah County property tax
statement for the last five billed tax years, add them up by year, and compare the growth with inflation.

## Ground rules

- **Never retype data.** Coordinates, IDs and amounts are moved programmatically and verified with checksums. A single mistyped polygon vertex (lon -122.65155 instead of -122.66155) once sent a boundary edge diagonally across the map until the user spotted it. Checksums: count, sum, and sum modulo 1,000,000,007 for ID lists.
- **Confirm scope with the user first.** Which school; which years (the last 5 tax years that have been billed, excluding the current year); amounts **before** the 3% prompt-pay discount (the levy, "the actual amount of funding") or after; spreadsheet layout. The original choices: Sabin K-5, tax years 2025 to 2021 (2025 = the 2025-26 bill), before discount, four columns per year (PPS, PPS Local Option, PPS Bonds, Total as a `=SUM` formula), newest year leftmost.
- **Ask before downloading** any file, and say the filename, source and size. Do not put results in a public place without asking.
- **A person starts the county site's guest session** (it shows a CAPTCHA and a disclaimer). Never automate or bypass these.
- **Be gentle with the county site**: at most 3 parallel workers with short pauses; the full run of about 13,600 statements takes many hours. Check the site's terms of use.
- **Publish address-level data only privately.** Street addresses next to tax amounts belong in a private repo; public outputs are aggregates, and the public map carries no addresses.
- The cloud sandbox may not be able to reach the county, PortlandMaps or ArcGIS hosts (HTTP 403 from the egress proxy). Do not route around that. Run the fetches as JavaScript in a browser page the user's computer provides (for example the Claude desktop app's browser pane, or Claude in Chrome) on the relevant site's own origin, and move results out by having that page save a JSON file the user attaches.
- State assumptions plainly and keep a log of decisions (they end up on the workbook's "Boundary Notes" sheet).

## Tools

- A browser that can run page JavaScript (see above), with the user's permission for: `portlandmaps.com`, `multcoproptax.com`, `services.arcgis.com` (ArcGIS REST), `cdnjs.cloudflare.com` (pdf.js).
- Python 3 with `openpyxl` and `matplotlib`; LibreOffice (headless) to recalculate the workbook's formulas.

## Step 1 - Get the attendance-zone polygon

The PPS School Finder is backed by an ArcGIS feature service:

```
https://services.arcgis.com/10N2YvfR3dqhPYAK/arcgis/rest/services/Schools_51e5ab7b167e457ebc668581dda23731/FeatureServer/2
```

Query `where=OBJECTID=33&returnGeometry=true&outSR=4326&f=geojson` for Sabin (`name` = "Sabin", `zonetype` = "K5", `facilityid` = "279").
For another school, find its row with `where=name='<School>' AND zonetype='K5'`. (Some PPS schools are K-8: `zonetype` K8.)
Take `features[0].geometry.coordinates[0]` as the ring. Sabin's ring has 98 vertices and is clockwise (the street-side logic relies on this; assert it).
Checksum of the verified Sabin polygon: sum of longitudes = -12019.471120605, sum of latitudes = 4464.091333391.

### Visual review of the boundary (do not skip)

Draw the polygon over a street map (Leaflet + OpenStreetMap tiles in a local HTML page shown to the user) and ask the user to check **every edge** against the PPS School Finder and their own knowledge. The review caught a real error once (the MLK Blvd edge cut diagonally to NE 14th & Prescott because of the mistyped vertex above). Fix by re-fetching and verifying the checksum, never by hand-editing coordinates. Note any place where the boundary leaves the street grid (a diagonal stair-step, a mid-block jog) so it can be described in the notes.

## Step 2 - Find every property inside

Portland's tax lots: `https://www.portlandmaps.com/arcgis/rest/services/Public/Taxlots/MapServer/0/query` (run the query from a page on `www.portlandmaps.com`).
POST (form-encoded) with:

```
geometry   = {"rings":[<ring>],"spatialReference":{"wkid":4326}}
geometryType = esriGeometryPolygon     inSR = 4326     spatialRel = esriSpatialRelIntersects
outFields  = OBJECTID,SITEADDR,SITECITY,SITEZIP,RNO,PROPERTYID,STATE_ID,PROP_CODE,PRPCD_DESC,LANDUSE,UNITS,YEARBUILT,COUNTY
returnGeometry = true     outSR = 4326     orderByFields = OBJECTID
resultOffset = 0,1000,2000...    resultRecordCount = 1000    f = json
```

Page until `exceededTransferLimit` is false. **The paged result repeats rows**, so de-duplicate by `OBJECTID`.
For each lot compute the area-weighted centroid of its largest ring and test it with a point-in-polygon check. A lot is "inside" when its **centroid** is inside.
Sabin: 2,900 lots intersect the polygon; 2,857 have their centroid inside; 43 touch the edge with the centroid outside and are not counted. The server's own count of intersecting lots must equal the unique rows you fetched.

Classify the inside lots. **Residence** = county use code (`PROP_CODE`) in `B` (residential improved), `X` (row/attached housing), `C` (2-4 unit multi-family) or `W` (residential condo) **and** an address that starts with a street number. Everything else goes to an "Excluded - Review" list with a reason, never silently dropped (16 lots carry a residential code but no street number). The property ID column is `PROPERTYID` (an "R" plus six digits, such as `R123456`).

Known limitation: the layer has one record per tax lot, so condo or attached units sharing a lot with separate accounts may be missing.

## Step 3 - Street-by-street boundary notes and the odd/even check

Pull the lots within 70 m of the polygon too (same query with `distance=70, units=esriSRUnit_Meter`) and check that its "inside" set equals Step 2's.
Walk the polygon edges clockwise from the northwest corner and group them into "runs" by street: classify each edge as north-south or east-west, match it to the nearest street name among nearby lot addresses (a street needs at least 6 lots; north-south edges match Ave/Blvd/Pl/Hwy, east-west edges match St/Ct/Dr/Ln/Ter; edges more than 30 m from any street are "midblock"; diagonal edges are named by a vote of lots within 45 m). Describe the result as a street walk, for example "NE Wygant St -> NE 22nd Ave -> NE Prescott St -> ... -> NE MLK Blvd".

On a boundary street one side is inside and the other outside, and in this part of Portland odd and even house numbers are on opposite sides. For each street run, find the majority parity among inside lots within 45 m, then flag inside lots with the other parity and outside lots with the inside parity. The polygon decides membership; the flags are a review list (Sabin: 62 flags, 10 of them residences inside the area).
Because the clockwise polygon has the inside on the right-hand side of travel, each run's inside side can also be stated as a compass side.

## Step 4 - The address list (workbook)

Build an Excel workbook with `openpyxl` (Arial 10, no hard-coded totals; recalculate and require zero formula errors):

1. **Boundary Notes**: the boundary walk table, the street-walk sentence, live counts by formula (inside lots, residences, excluded, and a check that residences + excluded = inside lots), legend, and the decisions list with each design decision tagged `[required: user]` or `[choice]`.
2. **Properties**: one row per residence, sorted by street then house number. Columns: Street Address (e.g. "123 NE 10th Ave."), Property ID, then four columns for each tax year (newest leftmost), then Use Code, Property Use, Boundary Check. Use a two-row header (year over component names), freeze panes at C3, autofilter.
3. **Summary**: one row per tax year with formulas over Properties (count of statements, sum of each component, and Total = sum of the three), plus a clustered column chart of years on the x axis with every bar labelled in millions of dollars, rounded to $0.1M (number format `$0.0,,"M"`; years run left to right in time order).
4. **Statement Notes**: properties with a missing statement. 5. **Excluded - Review**. 6. **Boundary Check**.

Address style: pretty-case tokens ("NE 10th Ave."), keep qualifiers such as "N/" in the county record as is.

## Step 5 - Read the tax statements

Do this on `https://multcoproptax.com/Property-Search` after a person has started a guest session. For each property R-number:

1. **Search by R-number.** The site is an ASP.NET (DNN) app. GET `/Property-Search`, copy every hidden form input, then POST the form back with `__EVENTTARGET=btnSearch`, `__EVENTARGUMENT=<R>`, `dnn$ctr410$MultnomahGuestView$SearchTextBox=<R>`. Parse the hidden input `dnn_ctr410_MultnomahGuestView_SearchResultJson`; its `ResultList[]` entry with `PropertyQuickRefID == R` has `PartyQuickRefID`. Refresh the form template every ~150 searches. (Typing into the page's search box via UI automation duplicated text; use the form POST.)
2. **Internal property ID.** GET `/Property-Detail/PropertyQuickRefID/<R>/PartyQuickRefID/<party>` and read `PropertyID:\s*"(\d+)"` from the HTML. A valid party ID is required; a cold-loaded detail URL without a guest session returns a "critical error".
3. **Statement PDF for each tax year.** POST `/Proxy/tax/TaxStatement` (form-encoded) with `PropertyID`, `TaxYear`, `EffectiveDate=11-15-<year>` and header `Accept: application/json`. A 200 returns a filename like `TaxStatement-<id>-<year>-11-15-<year>.pdf`; **HTTP 500 means there is no statement** for that year (record it as missing, do not retry forever). GET `/Proxy/documents/pdf/<filename>/`.
4. **Parse the PDF text** with pdf.js (cdnjs `pdf.js/3.11.174`; set `workerSrc`). In the "CURRENT TAX BY DISTRICT" block, match `PORTLAND PUBLIC SCHOOLS?([A-Z \/\.]*?)\s+\$?([\d,]+\.\d\d)`: an empty label is the general PPS line, a label containing `LOC` is Local Option, containing `BOND` is Bonds; anything else goes to a warnings list. Also read `TAX \(Before Discount\)\s+\$([\d,]+\.\d\d)` and `ACCOUNT NO:\s+(R\d+)` (must equal the property you asked for).
5. **Run with 3 workers**, 60-100 ms pauses, three retries per call, state saved to `localStorage` every 20 properties so a page reload can resume, and a halt after 12 consecutive failures or a "no session" error. Expect about 0.1 to 0.5 properties per second. The browser tab can drop mid-run; re-inject the script and resume (completed properties are skipped). Re-paste the property list in checksummed chunks, not by retyping.
6. **Validate**: (a) for every statement, the three PPS lines must sum to no more than the statement total; (b) spot-check one property you can verify by hand against a second source, such as the county site's own page or `taxgraph.multco.us`; (c) no parser warnings; (d) every property has a record for all five years (either amounts or "no statement").
7. Save a JSON file with the amounts and have the user attach it (see Ground rules).

Statement facts: with a full-year payment by the November due date the county gives a 3% discount (2/3 payment: 2%), and the statement shows the pre-discount total. All amounts in this study are pre-discount.

## Step 6 - Summaries, chart, map

- Aggregate by tax year over residences with a statement: properties billed, PPS, Local Option, Bonds, Total (`data/pps_tax_by_year.csv`).
- Chart: a grouped bar chart, tax years left to right in time order, four bars per year (PPS, PPS Local Option, PPS Bonds, Total), a label above each bar in millions rounded to $0.1M, a legend, and a plain subtitle ("Itemized property tax payments for 2,715-2,726 properties"). Colours: blue `#2a78d6`, orange `#eb6834`, green `#1baf7a`, amber `#eda100`.
- Map: a single self-contained HTML page (SVG, no tiles, so it works where external resources are blocked) with the boundary, one dot per lot (residences vs other lots), labelled boundary streets, pan/zoom, scale bar and north arrow. The public version carries no street addresses and no home marker.

## Step 7 - Inflation comparison

Compare compound annual growth of the total with: US CPI-U (annual average and December-to-December), West Region CPI-U (Portland-Salem CPI was discontinued after 2017), and the PCE price index. Use FRED series `CUUR0400SA0` (West) and `PCEPI`, and a BLS-based table for US CPI-U. Compute rates from index values yourself and say which figures are calculated. Compound over the four annual steps from the first to the last year.

## Step 8 - Report

Write the README with: the map; the property counts and every discrepancy with its percentage; the by-year chart and table; annual change against inflation; commentary that separates what the data shows from untested explanations. Keep address-level data out of the public repo.

## Expected results (Sabin K-5, tax years 2021-2025)

| Check | Value |
|---|---|
| Polygon vertices | 98 (clockwise) |
| Lots intersecting / centroid inside | 2,900 / 2,857 |
| Residences / excluded | 2,729 (B 2,547, X 90, C 92) / 128 |
| Boundary flags | 62 (32 inside, of which 10 residences; 30 outside) |
| Statements expected / found / missing | 13,645 / 13,590 / 55 (15 properties) |
| Statements with $0 to PPS | 173 |
| Residences billed per year, 2021 to 2025 | 2,715; 2,715; 2,715; 2,719; 2,726 |
| Total PPS tax, 2021 to 2025 | $5,441,343; $5,655,740; $6,107,159; $6,353,645; $6,653,089 |
| Compound annual growth 2021 to 2025 | 5.15% |
