# Entity Resolution — MSI Pipeline

Three independent tentacles, one shared pattern. Each resolves a noisy free-text name column from
the RC dataset into stable clusters with a canonical label.

| Tentacle | Input | Rows (raw) | Status |
|----------|-------|-----------:|--------|
| `owner_enrichment/` | `temp_unique_corporate_names` (`owner_name_cleaned`) | 349,413 | done |
| `financer_enrichment/` | `total_financers.csv` (`rc_financer`) | 738,115 | pipeline built, not yet run |
| `insurer_enrichment/` | _TBD_ | — | not started |

## Shared pattern

Every tentacle runs the same phase skeleton (`entity_resolution_pipeline.ipynb`):

```
clean -> typo/OCR fix -> canonical name -> embed (Qwen) -> FAISS top-K
      -> pair features -> rule engine (precision-first) -> union-find -> canonical label
```

What changes per tentacle is **only the domain knowledge**: the OCR dictionary, the
legal-form stopwords, the descriptor words, and the guards in the rule engine. The mechanics
(FAISS, union-find, feature set, incremental resolver) stay identical.

## Per-tentacle deltas

### `owner_enrichment`
Industry descriptors (`IMPEX`, `BUILDERS`, `TRAVELS`) are **identity** — they are the only thing
separating `A J IMPEX` from `A J BUILDERS`. Geography words (`INDIA`, `GLOBAL`) are throwaway.

### `financer_enrichment`
The reverse on both counts, plus one extra phase:

- **Phase 3 Institution Resolution** (new). `rc_financer` is captured per branch, so
  `STATE BANK OF INDIA HATHUR` / `SBI, PB BR DHANBAD` / `S.B.I KARPOORI THAKUR SADAN BR` are all
  one financer. A gazetteer of ~182 banks/NBFCs anchors the institution; the location residue moves
  to `branch_hint` and never affects clustering.
- **Abbreviation expansion** (`FIN`/`FINA`/`FINAN` -> `FINANCE`, `INV`/`INVE`/`INVST` -> `INVESTMENT`)
  and **co-operative vernacular folding** (`SAH` -> `SAHAKARI`, `PAT`/`PATH`/`PATTINA` -> `PATSANSTHA`).
- **`STATE` / `CENTRAL` / `NATIONAL` / `UNION` / `INDIA` stay discriminative** — without that,
  the four `* BANK OF INDIA` institutions collapse into one cluster.
- **`geo_conflict` guard** — `KERALA GRAMIN BANK` vs `KARNATAKA GRAMIN BANK` never merge.

### `insurer_enrichment`
Not started. Expect the financer shape more than the owner shape: a small number of real insurers
behind a long tail of branch/agent/office spellings, so the gazetteer-anchor approach should port
over directly.
