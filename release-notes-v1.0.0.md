# maphy-elections v1.0.0 — initial public release

Open Philippine election data, ready to use. **CC-BY-4.0** licensed; citation to *Mar Angelo N. Revelo* + upstream sources required (see `LICENSE` + `PROVENANCE.md`).

## What's in this release

7 gzipped tarballs, 447 MB total, 63.4 M CSV rows.

| Tarball | Size (gz) | Files | CSV rows | Coverage |
|---|---|---|---|---|
| `elections-historical-1992-2010.tar.gz` | 0.4 MB | 59 | 18,388 | Pre-AES province-grain (UP CIDS + Wikipedia canvass) |
| `elections-2013.tar.gz` | 0.04 MB | 5 | 584 | UP CIDS only — transparency feeds dark this cycle |
| `elections-2016.tar.gz` | 39 MB | 48 | 5,753,246 | macoymejia precinct + GMA Halalan barangay |
| `elections-2019.tar.gz` | 37 MB | 42 | 4,680,320 | GMA Halalan + ABS-CBN muni-fill (Caraga / Davao / SOCCSKSARGEN) |
| `elections-2022.tar.gz` | 194 MB | 85 | 29,144,958 | All sources cross-validated (figshare + GMA + ABS-CBN + Wikipedia) |
| `elections-2025.tar.gz` | 161 MB | 35 | 22,674,514 | figshare-Tumbagahan + GMA Halalan |
| `elections-meta.tar.gz` | 16 MB | 124 | 1,105,320 | Candidates registry + turnout + precinct→PSGC crosswalks + manifests |

Plus `release-manifest.json` with sha256 hashes for all 7 tarballs.

## Quality receipts (full detail in CREDIBILITY.md)

- **PSGC fill rate, 2022 precinct crosswalk:** 100.00% (107,765 / 107,765)
- **PSGC fill rate, 2025 precinct crosswalk:** 100.00% (85,183 / 85,183)
- **ABS-CBN × GMA top-12 partylist agreement:** 0.57-3.64% delta (mean ~1.5%). Headline parties: ACT-CIS 0.86%, BAYAN MUNA 1.03%, CIBAC 0.66%.
- **Impossible turnout rows in any cycle:** 0 (after the multi-vote-contest gate fix landed)
- **Distribution sanity:** turnout p50 by cycle matches canonical national figures (2025=73%, 2022=81%, 2019=73%, 2016=82%)
- **Test suite:** 614 / 614 pass on the BLEAD-side parser

## Verify integrity after download

```bash
sha256sum -c <(jq -r '.artifacts[] | "\(.sha256)  \(.filename)"' release-manifest.json)
```

## Quick start

```bash
gh release download v1.0.0 -R CelestialBrain/maphy-elections
for f in elections-*.tar.gz; do tar -xzf "$f"; done
```

## Citation

> Revelo, M. A. N. (2026). *maphy-elections: Open Philippine election data, 12 cycles 1992-2025, PSGC-keyed at barangay grain.* https://github.com/CelestialBrain/maphy-elections

Plus the upstream sources you draw from — see `PROVENANCE.md` for full list (Tumbagahan, Negrite, GMA, ABS-CBN, COMELEC, UP CIDS, etc.).

## Known gaps (documented honestly in CREDIBILITY.md)

- **2019 barangay-grain Caraga: 0%** — GMA's 2019 archive HTTP 403's region 13 at barangay grain; ABS-CBN doesn't ship barangay grain at all.
- **2010-2013 turnout:** province-grain only (source data doesn't go finer publicly).
- **1992-2007:** same.
- **2023 BSKE:** no public data exists. See `ACCESS.md` for FOI guidance.
- **Pre-2010 precinct:** never published in structured form by COMELEC. Lost to history.

## What changed during the build

Three significant data-quality fixes during the integration audit, all structural so future scrapes can't regress:

1. **ABS-CBN multi-vote-contest turnout inflation** — `voter.actualCount` at muni level counted BALLOTS CAST not unique voters; for senator (12 picks/voter), partylist, councilor (6-12), provincial_board, the actuals were 4-6× inflated. Fix: turnout emission gated to single-vote contests (president, vp, mayor, vice_mayor, governor, vice_governor, district_rep). Verification post-fix: 0 broken rows, max 95.8% (2019), 98.1% (2022).
2. **Provincial-board file pattern** — filenames are `provincial-board-member-*-provincial-district-location-*.json`, not the 2-part `*-province-location-*.json` the parser was matching. PSGC fill 0.1% → 97.2%.
3. **Region-name resolution** — `NCR`, `BARMM`, `CAR`, `REGION X (NORTHERN MINDANAO)` etc. weren't reaching the resolver's lookup table. Fix: `expandRegionCandidates()` handles abbreviations + parenthesized variants.

## What's next (v1.1+)

- Per-candidate share indicators (Marcos % / Robredo % / Sara %, etc.) — needs source-canonicalization design call
- Winning-candidate categorical maps with party color palette — needs design call
- 2023 BSKE if FOI delivers
- Pre-2016 cycles at finer than province grain if PIDS opens its crosswalk

Open an issue with what you need: https://github.com/CelestialBrain/maphy-elections/issues

---

**Maintainer:** Mar Angelo N. Revelo — `marangelonrevelo@gmail.com`

**Live atlas (visualization):** https://bygelo.com/maphy/ (Politics category)
