# CausalT5K clean exports

This folder holds **deduplicated, evaluation-ready** JSON for Pearl levels L1 (association), L2 (intervention), and L3 (counterfactual). Each file is a single array of case objects with a shared schema.

## Files

| File | Unique cases | Source |
|------|-------------:|--------|
| `CausalT5K_L1_clean.json` | 743 | All unique L1 cases in `D*/D*_L1.json` |
| `CausalT5K_L2_clean.json` | 3302 | All unique L2 cases in `D*/D*_L2.json` |
| `CausalT5K_L3_clean.json` | 1536 | All unique L3 cases in `D*/D*_L3.json` |

Regenerate all three levels:

```bash
python3 final_dataset/clean/build_clean_exports.py
```

## Build rules (L1, L2, L3)

From all domain shards under `final_dataset/D*/`:

1. **Deduplicate** to one row per logical case:
   - primary key: `case_id` when set;
   - otherwise: domain row `id` (for rows that lack `case_id`).
2. **No score filter** — every unique case in the domain files is included.
3. **Tie-break** when multiple rows share the same key: highest `final_score` / `final_score_2`, then adjudicated annotation, then longer `gold_rationale`.
4. **Assign** sequential clean ids `L1-0000`, `L2-0000`, … (sorted by domain, `case_id`, then `id`).
5. **Normalize** `trap` to `{canonical, raw_type_name, raw_type, raw_subtype}` and flatten nested `variables` where needed.
6. **Preserve** `original_id` and `original_case_id` from the chosen domain row.

The script asserts `len(clean) ==` domain unique-key count so exports cannot silently drop cases.

## Legacy curated L2 slice

`CausalT5K_L2_clean_small.json` (1360 rows, ~1101 unique `case_id`s, some duplicates) is an older hand-curated subset kept for reference. **`CausalT5K_L2_clean.json` is the full unique L2 export** (3302 cases), not a copy of `_small`.

## Schema (per case)

Core fields used in experiments and writeups:

- **Identity:** `id` (clean id), `case_id`, `original_id`, `original_case_id`, `pearl_level`, `domain`, `bucket`
- **Task:** `scenario`, `claim`, `label`, `variables` (`X`, `Y`, optional `Z`)
- **Trap:** `trap.canonical` plus raw codes/names (`raw_type`, `raw_type_name`, `raw_subtype`)
- **Reasoning:** `gold_rationale`, `key_insight`, `causal_structure`, `wise_refusal`, `conditional_answers`
- **Provenance:** `initial_author`, `validator`, `annotation`, optional `source`

Many legacy columns are present (often `null`) so the clean files align with the historical L2 export template.

## Comparison with domain JSON (`D*/D*_L*.json`)

| | Clean (`CausalT5K_*_clean.json`) | Domain (`D*/D*_L*.json`) |
|--|--|--|
| Rows | One per `case_id` | Multiple rows per `case_id` possible |
| IDs | `L1-0000`, `L2-0000`, … | Authoring ids (`T3-BucketF-…`, numeric `case_id`) |
| Traps | `trap.canonical` + raw fields | Often `trap.type` / `type_name` only |
| Scores | Not exported in clean files | `final_score`, `final_score_2`, etc. |

Use **clean** files for training/eval splits and paper examples; use **domain** files for auditing scores, duplicates, and adjudication history.

## Unlabelled holdout (not in clean exports)

Cases under `final_dataset/unlabelled/` are **excluded** from `CausalT5K_*_clean.json` and from `build_clean_exports.py` (which only reads `D*/D*_L*.json`, same as `scripts/causalt5k_data.py`).

| Item | Value |
|------|------:|
| File | `unlabelled/NO_cases_missing_traps.json` |
| Rows | 332 |
| Unique `case_id` | 40 |
| Pearl split | L1: 42, L2: 240, L3: 50 (row counts; many duplicate rows per id) |
| Domain | Medicine & Health (D4) only |
| Reason | **NO** cases removed because `trap` is missing (violates “exactly one trap type” for NO) |

These cases were **moved out of** `D4/D4_L*.json` but often still have `final_score ≥ 9`. They are kept for manual relabelling, not for the main benchmark slice.

### How this relates to README / paper statistics

The root [README.md](../../README.md) table **Valid Total** (e.g. D4 = 1,233, all domains = 5,147) aligns with counting **valid rows in `D*/` plus the unlabelled holdout**, not with `D*/` alone:

| Metric | `D*/` only (current files) | `D*/` + `unlabelled` |
|--------|---------------------------:|---------------------:|
| D4 valid rows (`final_score ≥ 9`) | ~902 | **~1,234** (≈ table 1,233) |
| All domains valid rows (`final_score ≥ 9`) | ~4,855 | **~5,187** (≈ table 5,147) |

**Clean exports** use globally deduplicated unique cases in **`D*/` only** (5,581 total: 743 + 3,302 + 1,536). That is the labeled, trap-complete benchmark. The 40 unique unlabelled ids are not included until traps are restored and cases are moved back into `D4/`.

See `unlabelled/README.md` for remediation steps.

## Layout

```
final_dataset/
  clean/
    README.md
    build_clean_exports.py
    CausalT5K_L1_clean.json
    CausalT5K_L2_clean.json
    CausalT5K_L2_clean_small.json   # alias / source for L2 clean
    CausalT5K_L3_clean.json
  D1/ … D10/
    D*_L1.json, D*_L2.json, D*_L3.json
  unlabelled/
    NO_cases_missing_traps.json   # holdout; not merged into clean/
```
