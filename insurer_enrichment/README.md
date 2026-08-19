# Insurer Enrichment — not started

Placeholder tentacle. Drop the raw insurer name export here (one column, same shape as
`financer_enrichment/total_financers.csv`), then port
`financer_enrichment/entity_resolution_pipeline.ipynb`.

## Porting checklist

The mechanics carry over unchanged. Only these need domain work:

- [ ] `CONFIG["input_file"]` / `input_col`
- [ ] `OCR_DICT` — run a token-frequency count on the raw column first, then fix the top misspellings
- [ ] `ABBREV_DICT` — expect `GEN`/`GENL` -> `GENERAL`, `INS` -> `INSURANCE`, `ASSUR` -> `ASSURANCE`
- [ ] `BRAND_ALIASES` — `NIA`/`NIACL`, `UIIC`, `OICL`, `TAGIC`, `ICICI LOMBARD`, `HDFC ERGO`, `LIC`
- [ ] `INSTITUTION_LIST` — the ~30 general insurers and ~25 life insurers are a closed list, so the
      gazetteer anchor should cover far more than the 48% it reaches on financers
- [ ] `LEGAL_STOPWORDS` / `DESCRIPTOR_WORDS` — keep `GENERAL` vs `LIFE` vs `HEALTH` **discriminative**
      (`SBI GENERAL` and `SBI LIFE` are different insurers), same reasoning as
      `STATE`/`CENTRAL`/`UNION` in the financer pipeline
- [ ] rule engine — the `desc_conflict` guard already covers the `GENERAL` vs `LIFE` split once
      those words are out of `DESCRIPTOR_WORDS`
