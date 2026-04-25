# Changelog

All notable releases of the maphy-elections dataset. Format loosely follows [Keep a Changelog](https://keepachangelog.com/). Versioning is semantic-data: MAJOR for schema-breaking changes, MINOR for added cycles or new metric types, PATCH for data-quality fixes that don't change schema.

## [v1.0.0] — 2026-04-25

Initial public release.

### Coverage
- 12 cycles 1992-2025
- 11 contests per recent cycle (president, vp, senator, partylist, governor, vice-governor, mayor, vice-mayor, councilor, district-rep, provincial-board)
- 4 cycles wired at barangay grain (2016 / 2019 / 2022 / 2025); pre-2016 at province-grain
- 158,219 source files / 8.4 GB raw; 7 gzipped tarballs / 447 MB / 63.4M CSV rows on release

### Sources merged
- figshare (Tumbagahan 2025, Negrite 2022, macoymejia 2016)
- GMA Halalan (all 12 cycles)
- ABS-CBN Halalan (2019 + 2022)
- Wikipedia canvass (pre-2010)
- UP CIDS Philippine Local Government Interactive Dataset (1992-2022)
- COMELEC Project of Precincts PDFs (2022 + 2025 crosswalks)

### Quality fixes baked in
- ABS-CBN multi-vote-contest turnout inflation — fixed via single-vote-contest gate; 0 impossible-turnout rows in deployed data
- Provincial-board file-pattern recognition — PSGC fill rate 0.1% → 97.2%
- 3-part location format (councilor / muni-district level) — splitter accepts both 2- and 3-part forms
- Region-name resolution for abbreviations (NCR, BARMM, CAR, REGION X (NORTHERN MINDANAO))

### Cross-validation receipts
- ABS-CBN × GMA top-12 partylist agreement: 0.57-3.64% delta (mean ~1.5%)
- 100% PSGC fill rate on 2022 + 2025 precinct→barangay crosswalks (107,765 + 85,183 precincts)
- Distribution sanity: turnout p50 by cycle matches canonical national figures (2025=73%, 2022=81%, 2019=73%, 2016=82%)

### Known gaps (documented, not bugs)
- 2019 barangay-grain Caraga: 0% (GMA 403's region 13 at barangay; ABS-CBN doesn't ship barangay)
- 2010-2013: province-grain only
- 1992-2007: same
- 2023 BSKE: no public data exists; FOI path open but probable denial

### Tarball layout
| Tarball | Size (gz) | Files | CSV rows |
|---|---|---|---|
| `elections-historical-1992-2010.tar.gz` | 0.4 MB | 59 | 18,388 |
| `elections-2013.tar.gz` | 0.04 MB | 5 | 584 |
| `elections-2016.tar.gz` | 39 MB | 48 | 5,753,246 |
| `elections-2019.tar.gz` | 37 MB | 42 | 4,680,320 |
| `elections-2022.tar.gz` | 194 MB | 85 | 29,144,958 |
| `elections-2025.tar.gz` | 161 MB | 35 | 22,674,514 |
| `elections-meta.tar.gz` | 16 MB | 124 | 1,105,320 |
| **Total** | **447 MB** | **398** | **63,377,330** |

### Verification (sha256, signed in release-manifest.json)
Run `sha256sum -c <(jq -r '.artifacts[] | "\(.sha256)  \(.filename)"' release-manifest.json)` after download.
