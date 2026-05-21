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
