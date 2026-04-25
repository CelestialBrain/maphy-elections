# maphy-elections

Open Philippine election data — 12 cycles 1992-2025 — multi-source-validated, PSGC-keyed at barangay grain, cross-checked against canonical canvass figures.

This repo is the **public extension** of the [maphy](https://github.com/CelestialBrain/maphy) civic atlas. It exists for one reason: PH election data is in the public domain by law, but the canonical crosswalks (precinct → barangay) are gatekept by the institutions that hold them. PIDS holds the 1970-2020 PSGC-vintage crosswalk and refused to share it when asked, telling researchers to file FOI requests. COMELEC's "public" pages return HTTP 403 to anything that isn't a real browser. Six different agencies hold overlapping pieces and none publishes them as machine-readable, versioned, openly-licensed data.

So one person rebuilt the layer from scratch. This repo is that rebuild, made public.

If you're a journalist, researcher, civic developer, or candidate who needs PH election data, **you don't need to file an FOI**. The data is here, citable, with provenance receipts.

---

## What's in here

| Asset | Coverage | Format |
|---|---|---|
| **Turnout** per polygon, per cycle | 12 cycles 1992-2025 | CSV |
| **Per-candidate vote tallies** | 11 contests per recent cycle (president, VP, senator, partylist, governor, vice-governor, mayor, vice-mayor, councilor, district rep, provincial board) | CSV |
| **Candidate registries** | Stable IDs across cycles | CSV |
| **Precinct → barangay crosswalks** | 2022 + 2025 (107,765 + 85,183 precincts, **100% PSGC hit rate**) | CSV |
| **Per-source provenance** | Every row carries `source` + `source_version` columns | column metadata |

Data is grouped into 7 per-cycle gzipped tarballs (each <2 GB, fits GitHub Release single-file limit). See [CHANGELOG.md](./CHANGELOG.md) for current release.

## Quick start

```bash
# Download a single cycle
gh release download v1.0.0 -p 'elections-2025.tar.gz'

# Or all of them
gh release download v1.0.0

# Extract
tar -xzf elections-2025.tar.gz

# Verify integrity (sha256 in release-manifest.json)
sha256sum -c <(jq -r '.artifacts[] | "\(.sha256)  \(.filename)"' release-manifest.json)
```

CSV schemas are documented in [`schemas/`](./schemas/).

## Why you should trust this data

Three orthogonal quality signals — the full receipts are in [CREDIBILITY.md](./CREDIBILITY.md).

1. **100% PSGC fill rate** on 2022 + 2025 precinct→barangay crosswalks (107,765 / 107,765 and 85,183 / 85,183). The data tested PIDS hasn't published.
2. **Cross-source validation:** ABS-CBN × GMA agree to **0.57-3.64% delta** on the top-12 partylist counts (mean ~1.5%). Headline parties: ACT-CIS 0.86%, BAYAN MUNA 1.03%, CIBAC 0.66%.
3. **Distribution sanity:** turnout values match canonical national figures (2025 muni p50 = 73%, 2022 = 81%, 2016 = ~82% — consistent with COMELEC's own canvass headlines).

After multiple parser fix passes (see [CHANGELOG.md](./CHANGELOG.md)) the deployed data has **0 impossible turnout rows** across all cycles. Max turnout values: 95.8% (2019), 98.1% (2022) — physically achievable, no >100% leaks.

## Citation

If you use this dataset in research, journalism, or a derivative product, please cite:

> Revelo, M. A. N. (2026). *maphy-elections: Open Philippine election data, 12 cycles 1992-2025, PSGC-keyed at barangay grain.* https://github.com/CelestialBrain/maphy-elections

Plus the upstream sources you actually drew from — see [PROVENANCE.md](./PROVENANCE.md) for the full list (Tumbagahan, Negrite, GMA, ABS-CBN, COMELEC, UP CIDS, etc., each with their own citation strings).

A machine-readable [`CITATION.cff`](./CITATION.cff) is provided for tools that auto-resolve citations (Zotero, GitHub's "Cite this repository" button, etc.).

## License

- **Data**: [CC-BY-4.0](./LICENSE) — free to use, modify, distribute, including commercially. **You must give credit** (see Citation above) and indicate if changes were made.
- **Code** (parsers, schemas, glue scripts): [Apache-2.0](./LICENSE-CODE) — permissive, with an explicit patent grant.

The CC-BY-4.0 choice is deliberate: it matches the upstream license used by figshare datasets (Tumbagahan 2025, Negrite 2022) and aligns with how COMELEC publishes its own canvass — public domain data, attribution-required derivative work. CC-BY-SA was rejected because share-alike scares off academic and journalistic reuse without strengthening the credibility signal.

## Need data this repo doesn't ship?

Two paths:

1. **For 2023 BSKE and pre-2010 precinct data:** these don't exist publicly in structured form. COMELEC never published them. You can [file an FOI](https://www.foi.gov.ph) requesting the masterlist; expect a 3-6 month timeline and probable denial (precedent: PSA denied the analogous 2022 precinct-SOV FOI).
2. **For PIDS's 1970-2020 crosswalk:** PIDS holds it but won't share. The 2022/2025 precinct→barangay crosswalk in this repo is functionally equivalent for those cycles (built independently from COMELEC's POP PDFs). Pre-2022 cycles in this repo are province-grain only.

Or [open an issue](https://github.com/CelestialBrain/maphy-elections/issues) with what you need — happy to help if it's something we can derive from the data here.

## Contact

- **Maintainer:** Mar Angelo N. Revelo — `marangelonrevelo@gmail.com`
- **Live atlas (visualization layer):** https://bygelo.com/maphy/ (Politics category)
- **Issues / data requests:** https://github.com/CelestialBrain/maphy-elections/issues

---

> "The work is genuinely good. The reason it's necessary is genuinely a state failure. Both things can be true."
