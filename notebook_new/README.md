# notebook_new: annotated notebooks

These are copies of `notebooks/` with every cell explained and every question answered. Original code cells and their saved outputs are unchanged. The added cells are:

- **📘 Explanation**: what the cell does, line by line, and how to read its output.
- **✅ Answer**: a full answer to each interview angle and each "Before you move on" question, using the real numbers from this data pack.

One line of code was changed in each notebook from 01 on, and in A.Data_Loading. The `ROOT` detection now also recognises a folder called `notebook_new`, so the notebooks find `data/` when run from here.

Cells in `05_methods` from section 2 on, and all of `06_answer_cold_run`, have no saved outputs, because those cells were never run. Their explanation cells quote the values they produce, recomputed on the same clean data with the notebooks' own filters and seed.

| Notebook | What the annotations add |
|---|---|
| 00_template | How the CONFIG and MAP/DO/SEE/CHECK/SUM structure works, with a filled-in example |
| 01_profile | Encoding, dtypes, grain and id audits; answers on encodings, `category` and payment-reference normalisation |
| 02_load_parallel_validate | Processes vs threads, the pydantic contract, and what the 3,107 rejects actually are |
| 03_clean_reconcile | The `pd.NA` trap, the merge flags (`validate=`, `indicator=`), patient de-duplication, and the full decisions bridge |
| 04_sql_transformations | Logical query order, window functions, indexes; the pandas vs SQL mismatch caused by 30 June |
| 05_methods | Tree, ladder, bridge, shuffle test, fair comparison, with recomputed outputs and slide-3 numbers per brief |
| 06_answer_cold_run | The board number, a claim sentence and a caveat to paste in, and the unit-label mismatch |
| A.Data_Loading | The standalone loader, and a warning that it overwrites `bookings_all.csv.gz` |

## Issues found while annotating (explained in the notebooks; code not changed)

1. **30 June is dropped.** `between("2025-01-01", "2025-06-30")` stops at 00:00 on 30 June, so pandas undercounts each half by about 1,000 bookings compared with SQL. Full-day headline: **455,383 vs 433,611 tests, +5.0%**.
2. **Reschedules lose their collection.** The phlebotomy app logs collections against the *original* nova booking id. Dropping the originals makes all 1,500 reschedules look "cancelled_or_missing", although 1,449 have a collection record. A tested fix is in 03.
3. **Labels don't match the code.** The headline unit in 06 and the chart axis in 05 say "collected or no-show", but the code counts all bookings. With that filter, growth is +4.2%.
4. **Logs are appended on every run.** The `logs.*` calls in 01 have already added the same rows three times to `docs/decisions-log.md` and `docs/approach-log.md`.
