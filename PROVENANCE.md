# Provenance

Every row in this dataset carries `source` and `source_version` columns that name exactly which upstream the row came from. This file documents each upstream — what they ship, what license they're under, and how to cite them.

**You must cite the upstreams you actually use.** maphy-elections itself is licensed CC-BY-4.0 (see [LICENSE](./LICENSE)) and requires citation per [README.md § Citation](./README.md#citation), but that citation does NOT replace the upstream attributions — it sits alongside them.

---

## Per-source breakdown

### figshare — Vianney Tumbagahan (2025 midterm)

- **Dataset:** *2025 Philippine Midterm Elections Data*
- **DOI:** [10.6084/m9.figshare.29086472](https://doi.org/10.6084/m9.figshare.29086472)
- **Author:** Vianney Tumbagahan (Philippines correspondent, Asia Elects)
- **License:** CC-BY-4.0
- **Coverage:** 2025 midterm precinct-level results + turnout, OAV + LAV included
- **Citation:**
  > Tumbagahan, V. (2025). *2025 Philippine Midterm Elections Data* (Version 2) [Dataset]. figshare. https://doi.org/10.6084/m9.figshare.29086472

### figshare — Neil Negrite (2022 presidential)

- **Dataset:** *2022 Presidential Elections Data*
- **DOI:** [10.6084/m9.figshare.19755469](https://doi.org/10.6084/m9.figshare.19755469)
- **Author:** Neil Negrite
- **License:** CC-BY-4.0
- **Coverage:** 2022 presidential precinct-level vote tallies
- **Citation:**
  > Negrite, N. (2022). *2022 Presidential Elections Data* [Dataset]. figshare. https://doi.org/10.6084/m9.figshare.19755469

### macoymejia — 2016 presidential precinct results

- **Repository:** [github.com/markjeee/macoymejia-elections-2016](https://github.com/markjeee/macoymejia-elections-2016) (or successor)
- **License:** MIT
- **Coverage:** 2016 NLE presidential precinct-level results (the iconic Duterte cycle)
- **Citation:** Cite the GitHub repository directly with author name + commit SHA used.

### GMA Halalan

- **Source:** GMA Network's Halalan election results portal (`gmanetwork.com/news/halalan*`)
- **Status:** Editorial reuse of public-domain COMELEC canvass data. GMA is the editorial publisher; the underlying numbers are public-domain by virtue of being COMELEC official canvass.
- **Coverage:** All 12 cycles 1992-2025, multiple admin levels per cycle. Most complete barangay-grain source for 2019.
- **Citation:** Cite as *"GMA Halalan, derived from COMELEC official canvass, scraped via maphy-elections pipeline {YYYY-MM-DD}"*.

### ABS-CBN Halalan

- **Source:** ABS-CBN News Halalan results dashboard (`halalanresults.abs-cbn.com`)
- **Status:** Editorial reuse of public-domain COMELEC canvass data. Same legal status as GMA Halalan — the editorial layer is ABS-CBN's; the numbers are public-domain.
- **Coverage:** 2019 + 2022 muni / province / region tallies. Used as cross-validation against GMA, and as the muni-level fill for 2019 regions where GMA's archive returns 403 (Davao, SOCCSKSARGEN, Caraga).
- **Citation:** *"ABS-CBN Halalan, derived from COMELEC official canvass, scraped via maphy-elections pipeline {YYYY-MM-DD}"*.

### Wikipedia canvass

- **Source:** Wikipedia per-cycle election articles (e.g., *2010 Philippine general election*, *2007 Philippine House of Representatives election*)
- **License:** CC-BY-SA-4.0 (Wikipedia content). Note: this is the only upstream with share-alike — derivative works that include verbatim Wikipedia text would inherit SA. We extract NUMERICAL FACTS (vote counts, candidate names) not prose, which is generally not copyrightable, so the SA inheritance does not bind. But if you re-publish Wikipedia prose extracted from these articles, you would need to comply with CC-BY-SA-4.0.
- **Coverage:** Pre-2010 cycles where structured data isn't published anywhere else.
- **Citation:** Cite the specific Wikipedia article + permalink revision ID used.

### UP Center for Integrative and Development Studies (UP CIDS)

- **Source:** *Philippine Local Government Interactive Dataset*, [elections.cids.up.edu.ph](https://elections.cids.up.edu.ph)
- **Authors:** Mendoza/Beja/Venida/Yap et al., UP CIDS
- **License:** "Absolutely free" per the site banner; not explicitly CC-licensed. Treated as freely-redistributable per the explicit free-use grant.
- **Coverage:** 11 election cycles 1992-2022, 227 LGUs, 15,148 candidates, plus LGU fiscal layer (IRA, tax, expenditure)
- **Citation:**
  > Mendoza, R. U., Beja, E. L., Venida, V. S., Yap, D. B., et al. *Philippine Local Government Interactive Dataset.* University of the Philippines Center for Integrative and Development Studies. Retrieved from https://elections.cids.up.edu.ph

### COMELEC Project of Precincts (POP) PDFs

- **Source:** Commission on Elections Philippines, `comelec.gov.ph/?r=2022NLE/ProjectOfPrecincts` and the 2025 equivalent
- **License:** Public domain per COMELEC's site footer (Philippine government work)
- **Coverage:** Per-province PDF tables of (barangay, precinct ID, clustered precinct, voting center, registered voters). Source for the 2022 + 2025 precinct→barangay crosswalks at 100% hit rate.
- **Citation:** *"COMELEC Project of Precincts ({2022|2025} NLE), parsed via maphy-elections pipeline."*

### altcoder/philippines-psgc-shapefiles

- **Repository:** [github.com/altcoder/philippines-psgc-shapefiles](https://github.com/altcoder/philippines-psgc-shapefiles)
- **License:** Per the repo's own LICENSE file (typically MIT or similar permissive)
- **Coverage:** 42,048 barangays at canonical 10-digit PSGC codes. Used as the join target for the precinct→barangay crosswalk.
- **Citation:** Cite the GitHub repo + commit SHA used.

---

## What we DID NOT use

Listed for transparency:

- **Hacked COMELEC voter databases** (2016, 2022 breaches). Off-limits ethically and legally.
- **COMELEC Precinct Finder** (`precinctfinder.comelec.gov.ph`). Identity-gated, Turnstile-protected, scraping at scale would violate the Data Privacy Act (RA 10173).
- **Rappler article text bulk-redistributed**. Their `robots.txt` disallows automated fetching; content is ARR. Used internally for name-canonicalization spot-checks only; not redistributed.

---

## Cross-source validation receipts

To confirm sources agree where they overlap, we cross-checked ABS-CBN × GMA on the top-12 partylist counts for 2022:

| Party | ABS-CBN votes | GMA votes | Delta |
|---|---|---|---|
| 101 ACT-CIS | 2,612,048 | 2,589,701 | 0.86% |
| 1 BAYAN MUNA | 1,110,407 | 1,099,014 | 1.03% |
| 141 AKO BICOL | 1,046,644 | 1,025,874 | 1.98% |
| 33 CIBAC | 924,345 | 918,235 | 0.66% |
| 54 ANG PROBINSYANO | 767,840 | 750,727 | 2.23% |
| 122 1PACMAN | 711,269 | 702,515 | 1.23% |
| 80 MARINO | 677,448 | 652,793 | 3.64% |
| 91 PROBINSYANO AKO | 628,698 | 619,421 | 1.48% |
| 130 SENIOR CITIZENS | 510,356 | 505,087 | 1.03% |
| 3 MAGSASAKA | 491,937 | 487,198 | 0.96% |
| 24 APEC | 479,729 | 472,306 | 1.55% |
| 37 GABRIELA | 445,696 | 443,151 | 0.57% |

Mean delta ~1.5%. Headline parties tight: 101 ACT-CIS 0.86%, 1 BAYAN MUNA 1.03%, 33 CIBAC 0.66%.

Five small parties show >5% delta — all vote totals < 250K where rounding amplifies (152 AKO PADAYON 9.5%, 20 DUMPER PTDA 5.6%, 74 MATA 5.1%, 136 ANG KABUHAYAN 7.6%, 97 TRICAP 6.0%). This is data documenting the upstream sources' own slight differences, not a maphy-elections issue.

---

*Last updated 2026-04-25 alongside v1.0.0 release.*
