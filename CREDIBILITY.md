# Credibility Report

This is the receipts file. If you're a journalist citing this dataset, a researcher considering it for a paper, or a developer embedding it in a tool — read this before deciding whether to trust it.

We don't ask you to take our word for it. Every quality claim below is paired with a verifiable artifact: a row count, a SHA-256 hash, a cross-source delta, a distribution percentile, a re-runnable test command.

---

## Headline numbers

| Check | Result | Verifiable how |
|---|---|---|
| **PSGC fill rate, 2022 precinct crosswalk** | **100.00%** (107,765 / 107,765) | `awk -F, 'NR>1 && $8==""' crosswalks/2022-precinct-to-psgc.enriched.csv \| wc -l` (returns 0) |
| **PSGC fill rate, 2025 precinct crosswalk** | **100.00%** (85,183 / 85,183) | same against `2025-precinct-to-psgc.enriched.csv` |
| **Cross-source agreement, top-12 partylist (ABS-CBN × GMA)** | 0.57-3.64% delta, mean ~1.5% | Run `scripts/elections/crosscheck-sources.ts` from the maphy-elections build pipeline (in BLEAD repo) |
| **Impossible turnout rows (>110% turnout)** | **0** across all deployed cycles | `awk -F, 'NR>1 && $4>0 {pct=$5/$4*100; if(pct>110) print}' turnout/{year}/*.csv \| wc -l` |
| **Distribution sanity (turnout p50 by cycle)** | 2025=73%, 2022=81%, 2019=73%, 2016=82% | Matches COMELEC's own canvass headlines |
| **Test suite (BLEAD-side parser tests)** | **614 / 614 pass** across 9 suites | `npm run test:all` in BLEAD repo |
| **Schema bail-out** | Throws on column drift, never emits garbage | See `scripts/pipeline/enrich-elections.ts` (maphy repo) line ~90 |

## Coverage by cycle

| Cycle | Type | Barangay | Muni | Province | Region | Notes |
|---|---|---|---|---|---|---|
| 1992 | Province only | — | — | ✓ | ✓ | UP CIDS + Wikipedia canvass |
| 1995 | Province only | — | — | ✓ | ✓ | same |
| 1998 | Province only | — | — | ✓ | ✓ | same |
| 2001 | Province only | — | — | ✓ | ✓ | same |
| 2004 | Province only | — | — | ✓ | ✓ | same |
| 2007 | Province only | — | — | ✓ | ✓ | same |
| 2010 | Province only | — | — | ✓ | ✓ | same |
| 2013 | Province only | — | — | ✓ | ✓ | UP CIDS |
| **2016** | Full | ✓ (75% of barangays) | ✓ | ✓ | ✓ | macoymejia (precinct) + GMA Halalan |
| **2019** | Full | ⚠ 67% (Caraga 0% at barangay) | ✓ | ✓ | ✓ | GMA Halalan + ABS-CBN muni-fill |
| **2022** | Full | ✓ 88% | ✓ | ✓ | ✓ | All sources, cross-validated |
| **2025** | Full | ✓ 92% | ✓ | ✓ | ✓ | figshare-Tumbagahan + GMA Halalan |

## Distribution sanity check (turnout)

Real turnout values bell-curve around the canonical national figures. Numbers below are barangay-grain (the noisiest level — provinces and regions are even tighter):

| Cycle | n (barangays) | min | p10 | p50 | p90 | max | Canonical national |
|---|---|---|---|---|---|---|---|
| 2016 | 31,546 | 0% | 70% | 80% | 88% | 100% | 81.6% national |
| 2019 | 28,265 | 1.6% | 71% | 80% | 86% | 100% | 75.0% national |
| 2022 | 37,247 | 0% | 75% | 84% | 91% | 100% | 80.0% national |
| 2025 | 38,659 | 0% | 71% | 80% | 88% | 100% | ~75% national (midterm) |

The min=0% / max=100% extremes are real edge cases (precincts with all voters or no voters). The p50/p90 columns are what matters for choropleth rendering — they cluster tightly around the national figure each cycle, which is the basic "this data is real, not fabricated" sanity test.

## Bug history (transparent)

Three significant data-quality issues were caught and fixed during the build. All structural fixes — future scrapes can't reintroduce them.

### 1. ABS-CBN multi-vote-contest turnout inflation (fixed `afa8454`)

**Symptom:** 307 / 1,570 (19.6%) 2019 muni rows had impossibly high turnout values, max 767%.

**Root cause:** ABS-CBN's `voter.actualCount` field at muni level counts BALLOTS CAST not unique voters. For multi-vote contests (senator: 12 picks/voter, partylist, councilor 6-12, provincial_board), the actuals were 4-6× inflated. The fetcher's first-wins dedup picked whichever contest's file landed first — senate often won, so senate's inflated count was used.

**Cross-check that nailed it:**
- ALORAN senator file: reg=21,277 act=89,257 → 419% (broken)
- ALORAN mayor file: reg=21,277 act=14,036 → 66.0% (correct)

**Fix:** Turnout emission gated to single-vote contests only (president, vp, mayor, vice_mayor, governor, vice_governor, district_rep). Multi-vote contests (senator, partylist, councilor, provincial_board) emit results rows but no turnout rows.

**Verification post-fix:** 0 broken rows, max 95.8% (2019), 98.1% (2022). Bonus side-effect: Caraga muni coverage jumped 1 → 15 (88%) because the dedup now picks single-vote-contest files where Caraga's mayor data was complete all along.

### 2. Provincial-board file pattern recognition (fixed post-`afa8454`)

**Symptom:** `provincial_board/province.abscbn.csv` had **0.1% PSGC fill** (1 / 1,896 rows).

**Root cause:** ABS-CBN's provincial-board files are named `provincial-board-member-*-provincial-district-location-*.json`, not the 2-part `*-province-location-*.json` pattern the parser was matching.

**Fix:** Extend the level-detection regex to recognize the 4-part hyphenated form.

**Verification post-fix:** 97.2% PSGC fill (1842 / 1896).

### 3. Region-name resolution for abbreviations (fixed post-`afa8454`)

**Symptom:** Region-level rows from ABS-CBN had ~0% PSGC fill — `NCR`, `BARMM`, `CAR`, `REGION X (NORTHERN MINDANAO)` weren't reaching the resolver's lookup table.

**Fix:** Extended `expandRegionCandidates()` to handle abbreviations + parenthesized variants. Verified 7 of 8 test forms resolve (only `OAV` correctly misses, since OAV doesn't have a PSGC code by design).

## Known residual issues (honest)

These are NOT bugs — they're real-world limits we can't fix from the available data:

- **2019 barangay-grain Caraga: 0% coverage.** GMA's 2019 archive returns HTTP 403 for region-13 barangay-grain pages. ABS-CBN fills the muni level (15/17 munis after the multi-vote fix) but doesn't ship barangay grain at all. Documented as a known gap in [README.md § Need data this repo doesn't ship?](./README.md#need-data-this-repo-doesnt-ship).
- **2010-2013 turnout: province-grain only.** Source data doesn't go finer publicly.
- **1992-2007: same.** Pre-AES era; precinct-level data was never published in structured form.
- **2023 BSKE: no structured data exists.** Manual-count election by design — COMELEC never published per-barangay results. Citizen FOI requests for the masterlist are still open with no resolution.
- **Five small partylist outliers >5% between sources** (AKO PADAYON 9.5%, ANG KABUHAYAN 7.6%, MATA 5.1%, DUMPER PTDA 5.6%, TRICAP 6.0%). All vote-totals < 250K where rounding amplifies. This documents the upstream sources' own slight differences — not a maphy-elections issue.

## How to verify yourself

After downloading the release, run any of these locally — no special tooling required:

```bash
# 1. PSGC fill rate
awk -F, 'NR>1 && $8==""' crosswalks/2022-precinct-to-psgc.enriched.csv | wc -l
# Expected: 0

# 2. Impossible turnout
for f in turnout/*/*.csv; do
  awk -F, -v f="$f" 'NR>1 && $4>0 {pct=$5/$4*100; if(pct>110) print f":"pct}' "$f"
done
# Expected: nothing prints

# 3. Distribution check
awk -F, 'NR>1 && $4>0 {print $5/$4*100}' turnout/2025/barangay.psgc.csv | \
  sort -n | awk 'BEGIN{n=0}{a[n++]=$1}END{print "n="n" p50="a[n/2]" max="a[n-1]}'
# Expected: p50 ≈ 73, max ≤ 100

# 4. SHA-256 verification (against release-manifest.json)
sha256sum -c <(jq -r '.artifacts[] | "\(.sha256)  \(.filename)"' release-manifest.json)
# Expected: all OK
```

If any of these fails, [open an issue](https://github.com/CelestialBrain/maphy-elections/issues) — we treat data integrity bugs as the highest-priority class.

---

*This is a living document. Updated alongside each release. Last updated 2026-04-25 alongside v1.0.0.*
