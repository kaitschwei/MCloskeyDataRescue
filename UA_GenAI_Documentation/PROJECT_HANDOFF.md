# McCloskey Data Rescue — Project Handoff

A complete brief for a fresh Claude project. As of 2026-04-30, pages 1–10 of the PDF are transcribed into the master CSVs (230 seed rows). Next page to process is page 11.

## 1. Project context

Ellen (ebledsoe@arizona.edu) is digitizing handwritten field data sheets from the **"Heteromyid Cheek Pouch Contents, Saguaro National Monument, 1975"** study. Each PDF page is a data sheet for one rodent's cheek-pouch seed sample. This is a data-rescue project — recovering historical ecology data trapped in paper records.

Repository: `/Users/ellenbledsoe/Documents/Git/MCloskeyDataRescue/`

Ellen primarily codes in R using tidyverse syntax. (Most of the work to date has been transcription, but any analysis code should be R/tidyverse.)

## 2. Source PDF

- Original: `Scanned from a Xerox Multifunction Printer.pdf` (114 pages)
- Pre-split chunks (use these — easier to load): `pdf_chunks/pages_001-010.pdf`, `pages_011-020.pdf`, … `pages_111-114.pdf`
- Sheets are landscape-oriented (rotated 270°); rendering + visual reading is required.

### Sheet structure

Header fields:
- **Species** (rodent, e.g. `P. amplus` = *Perognathus amplus*)
- **Identification** (animal ID, e.g. `LF1`)
- **Plot Number**
- **Station Number** (often blank)
- **Collection Date**
- **Total Number Seeds**

Data table columns (per row of the table):
- row #
- `W_L` — seed length
- `W_A` — major width
- `½W_B` — half the minor width (record literally as written)
- `Vol.` — calculated as `trunc(cbrt(L × major × ½W_B), 2)` (truncated to 2 decimal places, NOT rounded — confirmed on LF1 page 2 row 6: calc 1.157 → sheet 1.15)
- `Species` (of the seed, with handwritten annotations)

The right-hand "Species" column is **not** marginalia — it identifies the seed species per row and must be captured. Sheets sometimes use braces ("all but follow X") to propagate a formal genus to bracketed rows.

### Multi-page samples

One logical cheek-pouch sample can span multiple physical pages. When `Total Number Seeds` exceeds the rows on a page, the next page(s) are continuations with the same ID, species, plot, and date. Verify by summing row counts across continuation pages against the Total. (The June-15 LF1 sample spans pages 4–10: 27+28+28+28+28+28+13 = 180 seeds.)

## 3. Output files

All transcriptions go into two CSVs in `data_raw/`:

### `CheekPouchSeeds_1975_Claude.csv` — one row per seed measurement

Columns: `Day, Month, Year, Location, Plot, Station, Genus, Species, Identification, SeedLength, MajorWidth, MinorWidth, SeedGenus, SeedDescription, Flag`

- `Genus` / `Species` refer to the **rodent**.
- `SeedGenus` / `SeedDescription` refer to the **seed** (formal taxon + the informal as-written description, e.g. `"little canoe"`).
- `Flag` is an integer 0/1 column Ellen uses during line-by-line verification — set to `1` only when she explicitly says to flag a row.
- Blank Station cells stay empty.

### `CheekPouchSamples_1975_Claude.csv` — one row per unique cheek-pouch sample

Columns: `Day, Month, Year, Location, Plot, Station, Genus, Species, Identification, TotalSeeds`

- A multi-page sample gets ONE row here (don't add a new sample row when continuing onto the next physical page).
- Shared columns mean the same thing as in the seeds file.

The `_Claude` suffix flags these as Claude-assisted transcriptions; keep it.

### Conventions

- `Location` encodes plot area as `SNM_East` (Saguaro NM + direction).
- `Plot` holds just the number (e.g. `6`).
- Preserve column order and existing row order when appending.

## 4. Seed-species decode (informal sheet names → identification)

| Informal name on sheet | Formal taxon |
|---|---|
| "little canoe" / "canoe" | *Plantago* |
| "teardrop" | *Erodium* |
| "pistachio" | *Zinnia* |
| "lotus" | Deervetch (*Lotus*) |
| "tetra" / "little tetra" | Deervetch |
| "bladderpod" | *Lesquerella* |
| "little round" | unspecified — ASK Ellen when encountered |

Expect new informal name → taxon mappings as work progresses. Add them to this list as Ellen confirms them.

## 5. Validation approach (CRITICAL)

**Always read all four numbers on a row, including Vol. The Vol column is a built-in 2-decimal checksum on L / Major / Minor.**

Workflow per row:
1. Read L, Major (W_A), Minor (½W_B), AND Vol.
2. Compute `trunc(cbrt(L × major × ½W_B), 2)` and compare to the sheet's Vol value. (`round(., 2)` is also a valid match — some rows match under either; truncation is the observed default.)
3. **If cbrt matches Vol:** present the row confidently; no flag needed.
4. **If cbrt doesn't match Vol:** ALWAYS surface the discrepancy to Ellen. Do NOT silently correct or guess, even if a digit substitution seems obvious. Show: what was read, what the formula gives, and any plausible alternate readings. Let Ellen decide.
5. Keep originals in the CSV until Ellen confirms an alternate. Don't silently rewrite.

**Why this matters:** In an early verification of LF1 (50 rows), 8 corrections were needed and in every case Vol already disagreed with the original read. Using Vol as a pre-check up front catches these without bothering Ellen.

### Known handwriting digit confusions (ranked by frequency)

- `1 ↔ 4` — most common. Script "1" with a top flag vs. thin "4". Examples: 2.13→2.43, 2.14→2.44.
- `8 ↔ 9` — 2.18→2.19, 2.68→2.69, 0.78→0.79; also seen on Wa: 0.96→0.98.
- `2 ↔ 7` — script "7" with a curl can read as "2" in hundredths of L. Page 6 had two: 2.32→2.37.
- `2 ↔ 9`, `5 ↔ 9` — closed-loop "9" with a tail. 2.42→2.49, 2.45→2.49, 2.15→2.19.
- `7 ↔ 3` — 2.57→2.53.
- `4 ↔ 8` — 2.14→2.18.
- `1 ↔ 6` — page 6 row 8 L: 2.06→2.01 (Vol 1.02 forced the disambiguation).

### Where errors cluster

- Tenths and hundredths digits of L (most common).
- Major and Minor are rarely mis-read.
- Day/Month/Year/IDs occasionally mis-read.

### Sanity reminder

- If cbrt is too low by ~0.04, L is probably ~0.3 higher (single-digit jump in tenths).
- If too low by ~0.01, look at the hundredths of L or Minor.

### Animal-ID handwriting: L vs 4

A leading script capital "L" easily reads as "4". What was first transcribed as `4F1` on page 2 is actually `LF1` per Ellen, and the same form continues across many pages. Bias toward `L` unless the sheet shows a closed top loop or straight-edged numeral. If ambiguous, surface it before committing. Cross-check by looking at neighbouring pages — consistent IDs across a Total-Seeds spillover confirm the reading.

### Row-counting reminder

Always count physical rows on the sheet and confirm they match what gets appended to the CSV. Don't trust running cumulative-row math alone — page 6 originally had 28 rows on the sheet but only 27 in the CSV; the missing row was caught only on re-review.

## 6. Progress to date (as of 2026-04-30)

All completed pages are appended to `CheekPouchSeeds_1975_Claude.csv` and the corresponding samples to `CheekPouchSamples_1975_Claude.csv`.

| Pages | Date | Rodent | Rows | Notes |
|---|---|---|---|---|
| 1–3 | 1975-06-13 | LF1 (*P. amplus*), Plot 6 SNM_East | 50 | Full 50-seed sample |
| 4 | 1975-06-15 | LF1, same | 27 | Start of 180-seed sample |
| 5 | 1975-06-15 | LF1, same | 28 | continuation |
| 6 | 1975-06-15 | LF1, same | 28 | originally transcribed as 27; missing row inserted on re-review 2026-04-27 |
| 7 | 1975-06-15 | LF1, same | 28 | all Plantago "little canoe"; zero corrections on re-verify |
| 8 | 1975-06-15 | LF1, same | 28 | all Plantago "little canoe"; one flag at row 15 (Vol disagrees with cbrt: sheet 1.04 vs computed 1.08, Ellen confirmed values as-read). Row 25 L corrected from 2.75 to 2.35 by Ellen |
| 9 | 1975-06-15 | LF1, same | 28 | all Plantago "little canoe"; one flag at row 12 (Ellen revised L 2.12→2.19 with flag). Other Ellen corrections: row 8 L 2.26→2.21 (no flag); row 14 L 1.58→1.98 (no flag) |
| 10 | 1975-06-15 | LF1, same | 13 | final continuation; closes 180-seed June-15 sample (27+28+28+28+28+28+13 = 180); zero flags; bottom of page is blank — no second sample on this sheet |

Total seed rows in CSV: **230** (231 lines including the header).

Samples CSV: 2 rows so far — the 50-seed June-13 sample and the 180-seed June-15 sample, both LF1.

### Next page

**Page 11.** Source chunk: `pdf_chunks/pages_011-020.pdf`. The LF1 / June-15 sample is COMPLETE (180/180), so page 11 will begin a new cheek-pouch sample (different rodent and/or date). Read the header carefully and add a new row to `CheekPouchSamples_1975_Claude.csv`. Watch for new informal seed names that may need decoding.

## 7. Working with Ellen

- She codes in R/tidyverse — any analysis snippets should match.
- During verification she's hands-on: she'll override readings (e.g. "row 25 L is 2.35, not 2.75"), set flags explicitly ("flag this one"), and accept Vol-disagreement rows as-read when the original recorder's math was off.
- Always surface every Vol discrepancy. Never silently correct.
- Don't guess on `little round` or other unknown informal seed names — ask.
- Keep Claude-suffixed filenames (`_Claude`) intact.

## 8. Reference link

Project Google Doc (background, decode notes): `https://docs.google.com/document/d/1XURwwMb-Y9wm-yZjCHeIoY8Ih998kTHB2AgJvNQmbks/edit?usp=drivesdk` — "McCloskey Rodent Data Plant Code"
