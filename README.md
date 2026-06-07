# Development Timeline & Platting History Map
**Author:** Martins
**Date:** May 11, 2026
**Software:** QGIS (EPSG:2227 — NAD83 / California Zone 3, US Feet)
**Project File:** `Platting_History.qgz`

---

## 1. Project Overview

This project reconstructs the **digital cadastral history** of 1,904 land parcels
within the study area by analysing the `datemap_ad` and `daterec_ad` date fields
from the parcels layer. Building footprints from OSM are overlaid to identify
whether each parcel retains its original building stock or has been subject to
subdivision activity and potential redevelopment.

---

## 2. Critical Data Note — Date Field Interpretation

The original project brief assumed the date fields would span historical platting
eras (pre-1950 through post-2000). Field inspection revealed a different reality:

| Field | Coverage | Range | Interpretation |
|-------|----------|-------|----------------|
| `datemap_ad` | 1,904 parcels (100%) | 1998–2016 | Date parcel record was entered or last updated in the **digital cadastral system** |
| `daterec_ad` | 500 parcels (26%) | 1975–2016 | Date a **formal subdivision or lot-line adjustment** was recorded for the parcel |

`datemap_ad` does not represent the original land grant or subdivision date —
it reflects digital record-keeping activity. Parcels with older `datemap_ad`
values were part of the initial cadastral digitisation (1998–1999); those with
recent values have been modified, re-subdivided, or administratively updated.

`daterec_ad` is a stronger signal of active land change — its presence indicates
the parcel was formally re-recorded at some point, implying a lot split, merger,
or boundary adjustment. The 1,404 parcels without a `daterec_ad` have not been
formally re-recorded and are assumed to retain their original parcel geometry.

---

## 3. Era Cohort Classification (`map_era`)

Parcels are classified into four digital-era cohorts based on `datemap_ad`:

| Era | Year Range | Parcels | Interpretation |
|-----|-----------|---------|----------------|
| Legacy | 1998–1999 | 1,583 | Original cadastral fabric — digitised at dataset creation |
| Early Digital | 2000–2004 | 97 | First wave of parcel updates post-digitisation |
| Mid Digital | 2005–2009 | 83 | Second wave — active subdivision period |
| Recent Update | 2010–2016 | 141 | Most recent modifications or administrative updates |

---

## 4. Redevelopment Flag (`redeveloped`)

Each parcel is assigned a redevelopment classification based on the combination
of `daterec_ad` presence, `datemap_ad` era, and building coverage:

| Flag | Criteria | Count |
|------|----------|-------|
| Likely Redeveloped | Has `daterec_ad` + `datemap_year` ≥ 2005 + has buildings | 214 |
| Potentially Redeveloped | Has `daterec_ad` + has buildings | 277 |
| Original Stock | No `daterec_ad` + has buildings | 1,391 |
| Vacant / No Data | No intersecting building footprint | 22 |

Parcels flagged **Likely Redeveloped** are the most analytically significant —
they carry formal recording evidence of parcel change in the modern era and
currently have a building present, suggesting new construction followed the
subdivision event.

---

## 5. Building Coverage (BCR)

Building footprints were intersected with parcel polygons to compute:
- `bldg_sqft` — total building footprint area within each parcel (sq ft, EPSG:2227)
- `bcr_pct` — building coverage ratio capped at 100%
- `bldg_count` — number of distinct building pieces intersecting the parcel

This enables cross-tabulation of development intensity against cadastral era —
for example, whether Legacy parcels have higher or lower BCR than Recent Update
parcels, which would indicate infill or densification pressure.

---

## 6. Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `blklot` | String | Block-lot parcel identifier |
| `block_num` | String | Block number |
| `lot_num` | String | Lot number |
| `street` | String | Street name |
| `zoning_sim` | String | Simplified zoning code |
| `districtna` | String | Planning district |
| `datemap_ad` | String | Raw map-added date (YYYYMMDD) |
| `datemap_year` | Integer | Year parsed from `datemap_ad` |
| `daterec_ad` | String | Raw record-added date (YYYYMMDD) |
| `daterec_year` | Integer | Year parsed from `daterec_ad` |
| `map_era` | String | Era cohort label |
| `recorded` | String | Yes = has `daterec_ad` |
| `parcel_sqft` | Double | Parcel area (sq ft) |
| `bldg_sqft` | Double | Building footprint area within parcel (sq ft) |
| `bcr_pct` | Double | Building coverage ratio (%) |
| `bldg_count` | Integer | Building intersection pieces per parcel |
| `redeveloped` | String | Redevelopment classification |

---

## 7. Map Symbology

### Primary Layer — Platting History (map_era)
| Era | Colour |
|-----|--------|
| Legacy (1998–1999) | `#7b3294` Purple |
| Early Digital (2000–2004) | `#c2a5cf` Light purple |
| Mid Digital (2005–2009) | `#a6dba0` Light green |
| Recent Update (2010–2016) | `#008837` Dark green |

Building footprints are overlaid in grey to show physical building presence
against the cadastral era map.

---

## 8. Output Files

```
Development Timeline & Platting History Map/
├── Platting_History.qgz              ← QGIS project (open this)
├── platting_history.gpkg
│   ├── platting_history              ← Primary output layer
│   ├── building_footprints           ← OSM buildings (clipped, EPSG:2227)
│   ├── boundary                      ← Study area boundary
│   └── parcels                       ← Original cadastral parcels
└── README.md
```

---

## 9. Recommended Next Steps

- **Source original deed/grant records** — San Francisco's historic subdivision
  maps are held by the Assessor-Recorder's Office and would enable genuine
  pre-1950 platting history reconstruction.
- **Cross with suitability scores** — join `platting_history` to the v2
  suitability output on `blklot` to identify parcels that are both high-scoring
  and carry recent recording activity — the strongest redevelopment candidates.
- **Cross with BCR analysis** — compare mean BCR across era cohorts to test
  whether recently updated parcels are more or less developed than the legacy
  fabric — a direct measure of infill pressure.
- **Map block-level cohort dominance** — aggregate `map_era` by `block_num`
  to produce a block-polygon layer showing which city blocks are predominantly
  legacy fabric versus recently modified.

---

*Generated with QGIS MCP Plugin + Claude (Anthropic) — May 11, 2026*
