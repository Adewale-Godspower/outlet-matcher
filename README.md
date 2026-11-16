# Outlet Matcher

Match **existing outlets** to **newly surveyed outlets** — by phone number, outlet name, owner name, and GPS distance — and get a clean ID-to-ID crosswalk.

Single HTML file. Runs **100% locally in your browser**: no server, no upload, no build step, no dependencies.

## Why

Field teams often have a master list of outlets (from a census or CRM) and a fresh survey list of the “same” outlets, but the records rarely join cleanly:

- Phone numbers are written differently: `0803…`, `+234 803…`, `234803…`
- Outlet names have typos, swapped word order, or suffixes like *Shop / Stores / Ltd*
- One owner/agent phone may serve several outlets
- GPS points drift by a few metres between visits

Matching pins by eye on a map does not scale past a few hundred outlets. Outlet Matcher scores every plausible pair and only asks a human to review the ambiguous ones.

## Quick start

**Option A — just use it (recommended)**

1. Download `Outlet_Matcher.html` from this repo.
2. Double-click it — it opens in any modern browser.
3. (Optional, for confidential data) Disconnect from the internet. It still works.
4. Load your two CSVs — or practise first with the fake files in the [`samples/`](samples/) folder.

**Option B — Live site on Netlify (from GitHub)**

Push this repo to GitHub, then in Netlify: *Add new project → Import an existing project → GitHub → pick this repo*. Build command: *(leave empty)*. Publish directory: `.` (or just keep the included `netlify.toml`, which already sets this and makes the tool open at the site root). Every push to `main` redeploys automatically. Files loaded in the page are read locally by the browser and are never sent anywhere — but for confidential data, Option A (downloaded file, offline) is still the safer habit. **Never put real data files in this repo: a public Netlify site serves everything in it.**

## Input format

Two CSV files (export from Excel / Google Sheets via *File → Save As / Download → CSV*):

| File | Meaning |
|---|---|
| A | Existing outlets (master list) |
| B | Recently surveyed outlets |

Ideally each has: `ID`, outlet name, owner name, phone, latitude, longitude. Column names don't have to match exactly — the tool guesses, and you can correct the mapping in the UI. Missing phone/coordinate columns are handled, with reduced confidence.

Fake example files are in [`samples/`](samples/) so you can try the tool without real data.

## How matching works

1. **Cleaning** — Nigerian phone formats are normalised to one form; names are lowercased, punctuation/filler words removed, word order ignored.
2. **Candidate generation** — a surveyed outlet is only compared with existing outlets that share a phone number *or* are within ~1 km (spatial grid), so thousands of records stay fast.
3. **Scoring (0–100)** — weighted blend of outlet-name similarity (fuzzy), owner-name similarity, phone match, and closeness.
4. **Classification**
   - `AUTO_MATCH` — high score (or same phone at the same location): accept after a spot-check.
   - `REVIEW` — plausible but ambiguous, incl. *same phone, far apart* (possible shared phone) and *duplicate claims* on one existing outlet: check on a map and mark APPROVE / REJECT.
   - `UNMATCHED` — no safe partner: surveyed = likely new outlet; existing = not re-found.
5. **One-to-one rule** — if two surveyed outlets claim the same existing outlet, the higher score wins; the loser is flagged for review instead of silently double-matching.

Strictness presets (Conservative / Balanced / Lenient) and manual thresholds are in the UI.

## Output — one Excel workbook (`.xlsx`, generated locally)

| Sheet | Contents |
|---|---|
| `Summary` | Counts, files used, settings, reconciliation checks, and ready-to-paste XLOOKUP formulas |
| `Crosswalk` | `Existing_ID` in column A, `New_Survey_ID` in column B (AUTO_MATCH pairs) — built for `XLOOKUP` after converting the workbook to Google Sheets |
| `Matched` | Auto-matched pairs with full evidence: names, owners, phones, similarities, score, distance |
| `Review` | Ambiguous pairs with the same evidence plus a `Review_Decision` dropdown (APPROVE / REJECT) |
| `Unmatched_New` | Surveyed outlets with no match — probably genuinely new |
| `Unmatched_Existing` | Existing outlets not re-found |

IDs and phone numbers are stored as **text** so leading zeros and long IDs survive Excel/Sheets conversion. Headers are formatted, filtered, and frozen on every data sheet.

Typical flow: upload the workbook to Google Drive → convert to Google Sheets → in your main sheet use
`=XLOOKUP(A2, Crosswalk!$A:$A, Crosswalk!$B:$B, "Not Found")`.
After reviewing, copy the approved Review rows' A:B into the bottom of Crosswalk (the Summary sheet explains this, with a `FILTER` shortcut).

Keep the crosswalk as a permanent, dated asset: future survey rounds then only need to match previously unmatched outlets.

## Privacy & data safety

- The tool makes **no network calls**; verify by using it with Wi-Fi off.
- **Never commit real outlet data to this repo.** The included `.gitignore` blocks `*.csv` / `*.xlsx` and result files (only the fake files in `samples/` are allowed). Double-check `git status` before every push.
- Share row-level results only through your organisation's approved channels.

## Project structure

```
Outlet_Matcher.html   # the whole app — HTML + CSS + JS in one file
netlify.toml          # Netlify deploy config (publish root, serve tool at /)
README.md             # this file
HOW_TO_USE.md         # detailed step-by-step guide (incl. Excel-only fallback)
LICENSE               # MIT
samples/              # clearly fake demo CSVs
```

No build step, no `npm install`, nothing to deploy. To hack on it, edit the one HTML file and refresh your browser.

## Limitations

- Fuzzy matching will not catch an outlet that changed name, owner, *and* phone *and* moved far — those land in REVIEW/UNMATCHED by design.
- Phone-only matches across long distances are deliberately *not* auto-merged (shared phones are common).
- Very dense clusters (markets) may need the Conservative preset plus careful review of low-distance pairs.

## Contributing

Issues and PRs welcome — especially better name-normalisation rules for local naming patterns, and additional phone-format handling. Please do not attach real outlet data to issues; use the fake `samples/` format instead.

## License

MIT — see [LICENSE](LICENSE).
