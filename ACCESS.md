# Access Guide

How to get this dataset, what to do if it doesn't have what you need, and how to file FOIs for the upstream gaps.

---

## You probably don't need to FOI anything

Election results in the Philippines are **public domain by law** — they're official COMELEC canvass figures. The reason most researchers think they need to file an FOI is because:

1. COMELEC's website (`comelec.gov.ph`) returns HTTP 403 to anything that isn't a real browser, making the data practically inaccessible despite being legally public.
2. PIDS holds the canonical 1970-2020 PSGC-vintage crosswalk and refuses to share it (researchers have been told to file FOIs; never delivered).
3. Pre-AES era results (1992-2007) were never published in structured form — only as PDF canvass returns.

This repo solves (1) and (2) for the 4 cycles where barangay-grain rendering is feasible (2016 / 2019 / 2022 / 2025). Pre-2016 cycles ship at province-grain because that's all that exists publicly.

**If your research question is at province-or-coarser grain for 1992-2025**: download a release tarball, you're done.

**If you need barangay-grain for 2016-2025**: download a release tarball, you're done.

**If you need precinct-grain raw counts for 2022-2025**: same — the `results/{year}/{contest}/precinct.csv` files are in there.

**If you need 2023 BSKE barangay results**: doesn't exist publicly. See § FOI guidance below.

**If you need pre-2010 barangay-grain anything**: doesn't exist publicly. Lost to history.

## How to download

### Option A — direct from GitHub Releases

```bash
# Single cycle
gh release download v1.0.0 -p 'elections-2025.tar.gz' \
  -R CelestialBrain/maphy-elections

# All cycles + manifest
gh release download v1.0.0 -R CelestialBrain/maphy-elections

# Verify integrity
sha256sum -c <(jq -r '.artifacts[] | "\(.sha256)  \(.filename)"' release-manifest.json)

# Extract
for f in elections-*.tar.gz; do tar -xzf "$f"; done
```

### Option B — clone and build from source

If you want to re-derive everything from the upstream sources yourself (audit purposes, or to add a cycle we haven't yet integrated):

```bash
# Code lives in BLEAD (the scraping platform) — this repo is the public extension
git clone https://github.com/CelestialBrain/blead
cd blead
git checkout feat/elections
npm install
# Read docs/ELECTIONS.md for the full pipeline
```

### Option C — bulk transfer for institutional use

If you need a single consolidated archive for a research project, journalism workflow, or institutional mirror, **email the maintainer directly**: `marangelonrevelo@gmail.com`. Bulk transfers (or links to a private mirror) are available for legitimate use cases — much friendlier on origin infrastructure than scripted downloads.

## How to cite

See [README.md § Citation](./README.md#citation) and [CITATION.cff](./CITATION.cff). Required attribution string for the dataset is:

> Revelo, M. A. N. (2026). *maphy-elections: Open Philippine election data, 12 cycles 1992-2025, PSGC-keyed at barangay grain.* https://github.com/CelestialBrain/maphy-elections

Plus the upstream sources you actually drew from — see [PROVENANCE.md](./PROVENANCE.md).

## What's NOT in this dataset (and what to do about it)

### 2023 BSKE barangay results

Neither this dataset nor any other public dataset has structured 2023 BSKE results. COMELEC chair Garcia promised Nov 2023 uploads; what actually exists is (a) campaign-finance SOCE forms at `comelec.gov.ph/?r=CampaignFinance/SOCEBSKE2023`, (b) scattered municipal press releases (Manila Bulletin Pateros list, SunStar Cebu, GMA Mandaue), (c) LENTE's 88-pp qualitative assessment PDF.

**To request the canonical data:**

- File an FOI at [foi.gov.ph](https://www.foi.gov.ph) with COMELEC as the target agency. Sample request body:
  > Pursuant to Executive Order No. 2 (s. 2016) on Freedom of Information, I respectfully request the official Statement of Votes by City/Municipality and by Barangay for the 2023 Barangay and Sangguniang Kabataan Elections, including a digital structured copy (CSV or Excel preferred) of the per-barangay candidate vote tallies.
- Realistic timeline: 3-6 months. Realistic outcome: probable denial citing internal-only / not-yet-finalised. Precedent: PSA denied the analogous 2022 precinct-SOV FOI.
- If you do get a response, please consider sharing the results back — open an [issue](https://github.com/CelestialBrain/maphy-elections/issues) or email the maintainer.

### PIDS 1970-2020 PSGC-vintage crosswalk

PIDS holds it but won't share. They told a researcher (Vianney Tumbagahan) to file an FOI; never delivered. The 2022/2025 precinct→barangay crosswalk in this repo (built independently from COMELEC POP PDFs at 100% hit rate) is functionally equivalent for those cycles. For 2016-2019 we use ABS-CBN/GMA name-based joins; for pre-2016 we ship province-grain only.

**To request the PIDS crosswalk anyway:**

- Email PIDS directly: research@pids.gov.ph (or via [their contact form](https://pids.gov.ph)). State that you want the *Translation Tables / Crosswalks for Coding Schemes* dataset cited in their methodology papers.
- Realistic timeline: weeks to months. Realistic outcome: redirected to FOI.
- If you do get a response: please share. We'll integrate it as an additional source.

### Pre-2010 precinct-level results

These were never published in structured form by COMELEC. Wikipedia + UP CIDS aggregate them at province-grain. There is no FOI path that recovers precinct-grain pre-2010 data — it doesn't exist in any digital archive at the relevant agencies. Lost to history.

## Reporting issues

- **Data integrity issues** (impossible values, mismatched checksums, schema violations): highest priority. [Open an issue](https://github.com/CelestialBrain/maphy-elections/issues) with the cycle, contest, polygon, and observed value. We re-derive and re-release with the fix.
- **Coverage gaps** in regions/cycles where coverage is documented as full: [open an issue](https://github.com/CelestialBrain/maphy-elections/issues). We investigate whether it's an upstream gap or a parser miss.
- **License / attribution questions**: read [LICENSE](./LICENSE) and [PROVENANCE.md](./PROVENANCE.md). If still unclear, [open an issue](https://github.com/CelestialBrain/maphy-elections/issues).
- **New upstream source you think we should integrate**: [open an issue](https://github.com/CelestialBrain/maphy-elections/issues) with the source URL + license terms + rough scope. We'll evaluate.

## Maintainer

Mar Angelo N. Revelo — `marangelonrevelo@gmail.com` — [@angelonrevelo](https://twitter.com/angelonrevelo)

This is a one-person side project funded by no one and beholden to no one. That's the point.
