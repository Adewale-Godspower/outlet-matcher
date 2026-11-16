# Outlet Matching — Step-by-Step Guide
## Existing outlets (blue house icons) → Recently surveyed outlets (green checks)

**Privacy rule:** Do NOT upload these files to AI, online matching websites, or online file converters.
Everything below is done **on your own computer, offline**. That is the whole point of this method.

---

## 1. The big idea

Do NOT match icons by eye on Google My Maps — you have 3,000+ points, so that will be slow and full of errors.

Instead, match the **tables behind the map** in 3 passes, from most certain to least certain:

| Pass | Rule | What happens |
|---|---|---|
| **1 — Phone** | Cleaned phone number is the same | Auto-match (unless the two points are far apart — see §5) |
| **2 — Name + owner + distance** | Outlet name and owner name are almost the same **and** the points are close (roughly < 300 m) | Auto-match |
| **3 — Fuzzy / close location** | Names are similar but not identical (typos, word order), or the points are very close but names differ | Goes to a REVIEW list — you check only these on the map |

Everything that matches nothing is reported separately:
- Surveyed but unmatched = probably a **genuinely new outlet**
- Existing but unmatched = **not re-found** (closed, moved, missed, or phone/name changed completely)

### Do you need to build a system?
No big system. You need one small, repeatable local tool that you re-run each time a new survey batch arrives.
The file **`Outlet_Matcher.html`** in this folder is that tool:
- Double-click it — it opens in your browser like a web page
- It makes **no internet connection** (you can switch off Wi-Fi after downloading it and it still works)
- Your CSVs are read by your browser on your machine and the result file is created on your machine. Nothing is sent anywhere.

If your team prefers Excel/Python, the same 3-pass logic below works there too — see §7.

---

## 2. Prepare your two files (15 minutes)

1. Find the **source spreadsheets** you uploaded to Google My Maps (your "Lagos Upload" sheets).
   The original sheets are better than a map export because they contain phone + owner columns.
   *Only have the map?* In Google My Maps use the `⋮` menu next to the map/layer → **Download KML / Export data**, then convert it — KML keeps coordinates + ID/name, but may not have phone/owner.
2. Work on **copies**, never the originals.
3. In Excel / Google Sheets: **File → Save As / Download → Comma Separated Values (.csv)** for each file:
   - `existing_outlets.csv`
   - `surveyed_outlets.csv`
4. Check each file has as many of these columns as possible (names don't have to be exact — the tool lets you map them):

   `ID | Outlet Name | Owner Name | Phone | Latitude | Longitude`

5. Quick clean-up in the sheet before exporting:
   - One row per outlet. Delete totally blank rows.
   - Don't merge cells. Don't put two outlets in one row.
   - Two phone numbers in one cell (e.g. `0803... / 0805...`) are OK — the tool reads both.
   - **Never change or re-use an ID.** The ID is the anchor everything joins back to.

---

## 3. The cleaning rules (why matches are missed without them)

The tool does these automatically, but you should understand them — this is where 80% of missed matches come from:

- **Phone:** remove spaces, dashes and `+`. Convert `+234 803…` and `234803…` to `0803…`.
  So `0803...`, `+234 803...` and `234803...` become one and the same number.
- **Outlet / owner names:** make lowercase, remove punctuation, ignore word order,
  and ignore filler words like *Shop, Store, Stores, Ltd, Enterprises, Provisions*.
  So "Adebayo Stores" ≈ "Store Adebayo", and "Chiamaka Provisions" ≈ "Chiamaka Provision Store".
- **Spelling:** fuzzy matching allows small typos (e.g. Adebayor vs Adebayo) — that's why it beats Excel VLOOKUP, which only finds *exact* matches.

---

## 4. Run the tool (5 minutes)

1. Download `Outlet_Matcher.html` to your computer and double-click it.
2. *(Recommended first time)* Practise with the fake CSVs in the `samples/` folder → **Run Matching**, so you can learn the flow with zero risk before touching real files.
3. Load `existing_outlets.csv` on the left and `surveyed_outlets.csv` on the right.
4. Check the column dropdowns the tool guessed (ID / outlet / owner / phone / lat / lon). Fix any that are wrong.
5. Leave it on **Balanced — recommended** (trust automatically at score 78+, send to Review at 55+, trust only within 300 m) and click **Run Matching**.
6. Download **`outlet_match_results.xlsx`** — one Excel workbook with 6 sheets:
   - **Summary** — counts, settings, and your XLOOKUP formulas
   - **Crosswalk** — `Existing_ID` (col A) → `New_Survey_ID` (col B), auto-matches only
   - **Matched** — auto-matched pairs with full evidence
   - **Review** — pairs you must check by eye (with an APPROVE/REJECT dropdown)
   - **Unmatched_New** — surveyed outlets with no match (probably new)
   - **Unmatched_Existing** — existing outlets not re-found

---

## 5. How to review (this is where your time should go)

Open the workbook's **Review** sheet (it's already sorted by **Score**, highest first), check each pair on Google My Maps, and pick from the `Review_Decision` dropdown:

- `APPROVE` — same outlet
- `REJECT` — different outlets

When you're done: filter Review by `APPROVE`, copy columns A:B of those rows, and paste them at the bottom of the **Crosswalk** sheet. (In Google Sheets, `=FILTER(Review!A:B, Review!Q:Q="APPROVE")` lists them for you — column Q is `Review_Decision`.)

**Using the Crosswalk in Google Sheets:** upload the workbook to Drive and convert it to Google Sheets, then in your main sheet:

- Existing → New: `=XLOOKUP(A2, Crosswalk!$A:$A, Crosswalk!$B:$B, "Not Found")`
- New → Existing: `=XLOOKUP(A2, Crosswalk!$B:$B, Crosswalk!$A:$A, "Not Found")`

(change `A2` to the cell holding your ID). IDs are stored as text — keep them as text in your sheet too, or lookups will miss.

**Watch out for these traps:**

1. **Shared phone:** a distributor/agent/owner's number used for many outlets. Same phone but points far apart (> ~1 km) ≠ automatic match. The tool flags these as REVIEW for exactly this reason.
2. **Same name, different place:** many outlets share generic names. Name alone is not a match — name **+ close distance** is.
3. **Very close, different name:** outlet may have been rebranded or taken over by a new owner. Check on the map/street-level detail before deciding.
4. **Duplicate surveys:** two surveyed rows claiming the same existing outlet. The higher score wins; check whether the other is a duplicate survey or a second nearby outlet.
5. **Moved outlet:** same phone + name, but 200–500 m away. Usually APPROVE with a note "relocated" and update the existing coordinates.

Sanity checks before you finish:
- `Existing total = matched + not re-found` and `Surveyed total = matched + new`. The tool shows these counts — make them reconcile.
- Spot-check ~20 AUTO_MATCH rows on the map. If you find errors, raise the Auto threshold (e.g. 78 → 85) and re-run — that's what the settings are for.

---

## 6. Keep the crosswalk (your permanent asset)

Your final deliverable is one master file, kept by you:

`Existing_ID | New_Survey_ID | Match_Method | Score | Distance_m | Review_Decision`

- Join on this file whenever you need to connect old and new data — don't overwrite either ID.
- Next survey round, you only need to match the **unmatched** outlets against the new batch. Each round gets easier.
- Keep a dated backup before each matching round (e.g. `crosswalk_2026-11-10.xlsx`).

---

## 7. If you'd rather do it in Excel (no tool)

The logic is identical, just more manual — fine for a one-off small batch, painful for 3,000+ rows:

1. Add a `Phone_Clean` column to both sheets (strip spaces/`+`, replace leading `234` with `0`).
2. **Pass 1:** `XLOOKUP`/`VLOOKUP` on `Phone_Clean` → record matches + matched ID.
3. Remove matched rows from both sides.
4. **Pass 2:** normalise names (lowercase, remove "shop/store/ltd"), then exact-match on name → record matches.
5. Remove matched rows again.
6. Everything left in step 5 has *no exact match* — in Excel you'd have to eyeball it, which is exactly why the fuzzy + distance tool exists. Excel's **Merge Queries → Fuzzy matching** (Power Query) can approximate Pass 3 if you already use Power Query.

A Python script (pandas + RapidFuzz, run locally) does the same thing as the HTML tool — use it if someone on your team already writes Python and wants Excel files in/out directly. Never use a website that asks you to upload the files to do this.

---

## 8. Confidentiality checklist

- [ ] Files stay on your machine / your company drive — not in AI chats, not on matching websites
- [ ] Tool downloaded once, run offline; no add-ons from unknown publishers installed
- [ ] Access to the crosswalk file limited like the source data — it links identities to locations
- [ ] Share results as counts/summaries externally, not the row-level file, unless approved
