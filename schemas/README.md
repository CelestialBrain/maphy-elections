# CSV Schemas

Every CSV in this dataset follows one of four schemas. All are PSGC-keyed where possible; every row carries `source` + `source_version` columns for full provenance.

## `results/{year}/{contest}/{level}.{source}.csv`

Per-candidate vote counts at the specified admin level.

```
psgc_code           string    10-digit PSGC for level≠precinct/national
level               string    region | province | muni | barangay | precinct | national
precinct_id         string?   only when level=precinct (clustered precinct code)
candidate_id        string    BLEAD-stable ID, matches candidates/{year}.csv
candidate_name      string    exact-as-printed on ballot
party               string?   abbreviation (PFP, PDP-Laban, etc.)
votes               integer   ≥ 0
rank_within_level   integer?  1 = top vote-getter at this polygon
source              string    figshare | gma | abscbn | wikipedia-canvass | upcids | macoymejia
source_version      string    DOI version, commit hash, or scrape date
extracted_at_utc    ISO 8601  when this row was extracted from upstream
```

## `turnout/{year}/{level}.{source}.csv`

Voter participation at the specified admin level. Note: turnout rows are emitted only from single-vote contests (president, vp, mayor, vice_mayor, governor, vice_governor, district_rep) to avoid the multi-vote-contest inflation bug where senate's 12-picks-per-voter ballots-cast count is divided by registered_voters → impossible >100% turnout. See [CREDIBILITY.md § 1](../CREDIBILITY.md) for the full bug history.

```
psgc_code           string    10-digit PSGC
level               string    region | province | muni | barangay | precinct
precinct_id         string?   only when level=precinct
registered_voters   integer   ≥ 0; 0 means upstream didn't carry registration counts
actual_voters       integer   ≥ 0
undervotes          integer?  contest-specific; not all sources populate
overvotes           integer?  same
valid_votes         integer?  same
oav_included        boolean   true if overseas absentee included in actual_voters
lav_included        boolean   true if local absentee included
source              string    figshare | gma | abscbn | bare
source_version      string    
```

## `candidates/{year}.csv`

Canonical candidate registry. One row per candidate-contest pair.

```
candidate_id        string    Stable across cycles; see § Candidate ID schema below
comelec_coc_id      string?   when COC PDF carries one (rare)
canonical_name      string    FirstName MiddleName LastName
alias_names         string    pipe-separated: "Manny Pacquiao|Sen. Manny|Pacman"
ballot_name         string    exactly as printed on the official ballot
contest             string    president | vp | senator | partylist | governor | ...
party               string?
is_substitute       boolean
substitute_for      string?   candidate_id (if is_substitute=true)
coc_filed_at        date?
sex                 string?   male | female | X | null
date_of_birth       date?
wikidata_qid        string?   Q12345 if Wikidata enrichment found a match
source              string    same enum as results/turnout
```

## `crosswalks/{year}-precinct-to-psgc.{enriched.}csv`

Precinct → barangay PSGC map. Two variants: bare (name-tuple only, `psgc_code` empty) and `.enriched` (PSGC-resolved at 100% hit rate on 2022+2025).

```
precinct_id              string    8-digit COMELEC precinct ID
clustered_precinct_id    string?   comma-separated original precinct codes
voting_center_id         string?   polling-center name + address
region_name              string    canonical (e.g., "CAR", "REGION IV-A")
province_name            string    canonical
muni_name                string    canonical
barangay_name            string    canonical
psgc_code                string    10-digit (empty in non-enriched variant)
match_method             string    exact | fuzzy | crosswalk | region_only
match_confidence         integer   0-100; ≥90 = high confidence
source                   string    pop-crosswalk
source_version           string    pop-{year}-scraped-{date}
```

## Candidate ID schema (3-tier stable)

There is no authoritative national ID for Philippine candidates. We mint a stable `candidate_id` and reuse it across contests + years whenever the same person is identified.

Algorithm (priority order):

1. If `comelec_coc_id` exists → `candidate_id = "coc:{comelec_coc_id}"`
2. Else if Wikidata match with ≥3 corroborating facts (name + DOB + last office held) → `candidate_id = "wd:{qid}"`
3. Else deterministic hash of `(normalized_canonical_name, date_of_birth)` → `candidate_id = "h:{sha1[:12]}"`

When a higher-tier ID later becomes available (e.g., we find the COC PDF for a candidate previously identified by hash), an alias row is emitted in `candidates/_aliases.csv` with `(from_id, to_id, merged_at_utc)` so historical references can be rewritten downstream.

This means:
- `candidate_id` is stable across releases — never silently changes
- Cross-source merges work (figshare uses "Marcos", ABS-CBN uses "MARCOS, BONGBONG (PFP)" — both can resolve to the same `wd:Q1...` after Wikidata enrichment)
- Tier prefixes (`coc:` / `wd:` / `h:`) tell you the confidence level at a glance
