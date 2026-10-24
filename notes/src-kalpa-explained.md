# `src/kalpa/`, file by file

`src/kalpa/` is a small Python package that the analysis notebooks share. It has four working modules and an `__init__.py`. Data moves through them in this order:

```
raw files ──► loaders.py ──► models.py ──► checks.py
              (read)         (validate)    (verify counts and joins)
                                  │
                                  └──► logs.py (record decisions in docs/*.md)
```

---

## [`__init__.py`](../src/kalpa/__init__.py)

This file holds no code, only a docstring. It does two things:

- It marks `kalpa` as a package, so notebooks can run `from kalpa import loaders, models, checks, logs`.
- It shows how to import it from a notebook: add `../src` to `sys.path` first, because the package isn't pip-installed.

It also says every function has a test in `tests/`.

---

## [`loaders.py`](../src/kalpa/loaders.py): reading the raw files

There is one reader for each source format. Each returns a pandas DataFrame with one row per booking-test.

| Function | What it does |
|---|---|
| `list_pack(root)` | Lists every file in the data pack with its path, family (`legacy` / `nova` / `results` / folder name), size and extension. Use it to see what's in the pack. |
| `read_legacy_file(path)` | Reads a gzipped legacy CSV export, which is in **Latin-1** encoding. It removes **header lines that repeat mid-file** and counts them in `df.attrs["repeated_header_lines"]`. It parses `dd/mm/yyyy HH:MM` dates (bad dates become `NaT`), strips whitespace from `booking_id` and adds `source_file` and `system="legacy"`. |
| `read_nova_file(path)` | Reads a gzipped nova **JSON-lines** export. Each line is one booking with its tests nested inside, so it creates one row per test to match the legacy layout. It lowercases `channel` and `source`. Timestamps are converted from UTC to IST and the timezone is then removed, so they compare cleanly with legacy times. `patient_name` is NA because nova doesn't export it. |
| `read_simple_csv(path)` | A thin wrapper around `pd.read_csv` that adds a `source_file` column. |
| `READERS`, `_dispatch` | `READERS` maps `"legacy"` / `"nova"` to their reader. `_dispatch` is a top-level function, so worker processes can pickle and call it. |
| `read_many(paths, kind, workers)` | Reads many files at once in a **process pool**. The docstring explains why processes rather than threads: gzip, CSV and JSON parsing are CPU-bound, and threads would wait on Python's global interpreter lock. Results go back into their original slots, so row order is the same on every run. It also adds up the repeated-header counts. |
| `iter_chunks(path, chunksize)` | Reads a CSV in chunks, for files too big to load into memory at once. |
| `normalise_phone(s)` | Removes non-digits and keeps the last 10, so `+91 98765 43210` becomes `9876543210`. Works on a whole column at once. |
| `normalise_invoice_ref(s)` | The column version of `Payment.invoice_id_normalised` (see below). It rewrites variants like `inv2026000123` as `INV-2026-000123`, and returns anything unrecognised just stripped of whitespace. |
| `ID_PATTERNS` | Regexes for legacy and nova booking IDs. They duplicate the ones in `models.py`. |

---

## [`models.py`](../src/kalpa/models.py): validating records (Pydantic 2)

This module checks each record before it enters the analysis. A record is either accepted, possibly after a harmless clean-up, or **rejected with a reason**. It is never quietly patched. That keeps `input = accepted + rejected` true at every step.

**Regexes:**

- `LEGACY_ID`: `BK-ABC-123456-123456`
- `NOVA_ID`: `nova_abc_1234567`
- `TEST_CODE`: `T###` or `PK##`

**`TestLine`**: one test within a booking.

- `code` is stripped, uppercased and must match `TEST_CODE`.
- `price` must be ≥ 0.
- Extra fields are not allowed (`extra="forbid"`).

**`Booking`**: one visit, from either system.

- **Cleaned up:** whitespace on all strings is stripped, `channel` and `source` are lowercased, and `"Bangalore"` becomes `"Bengaluru"`.
- **Rejected when:**
  - the booking ID matches neither format;
  - `channel` or `source` isn't an allowed value;
  - the booking has no tests (`min_length=1`);
  - `slot_at` is earlier than `created_at`. Both are compared without timezones, because nova includes `+05:30` and legacy has no offset, but both are IST.
- **Derived properties:** `n_tests` and `amount_rs` (sum of test prices + `home_fee`).

**`Payment`**: one row from the payment gateway export.

- Amounts arrive in paise and must be ≥ 0. `amount_rs` converts them to rupees.
- `status` must be `captured` or `failed`, and `method` must be one of the four allowed values.
- An empty or NaN `settlement_id` becomes `None`.
- `invoice_id_normalised` fixes case, dashes and whitespace in the invoice reference. On purpose, it does **not** try to fix typos like a transposed digit. Those refs fail the join, as the docstring intends.

**`validate_many(model, records)`**: runs a model over many dicts and returns `(accepted, rejects)`. Each reject is `{"record": original, "errors": [(field, message), ...]}`. It never raises an exception.

---

## [`checks.py`](../src/kalpa/checks.py): checks that print PASS/FAIL

These are assertions that print a result instead of raising, so a notebook keeps running and you see every failure at once. Every result is also stored in a module-level list, `_RESULTS`.

| Function | What it checks |
|---|---|
| `check(name, condition, detail)` | The basic check. It prints `PASS`/`FAIL`, stores the result and returns the bool. |
| `reconciles(name, n_in, n_kept, n_rejected)` | That `input = kept + rejected`. It prints all three numbers so you can check the sum yourself. |
| `join_did_not_fan_out(name, left, joined, key)` | That a LEFT join didn't add rows. If it did, the right table has duplicate keys, and it reports how many rows sit on repeated keys. |
| `bridge_balances(name, start, steps, end, tol=0.5)` | That `start + sum(steps) = end` to within ₹0.50. It prints each step, so the whole bridge from one total to another is visible. |
| `summary()` | Prints "X of Y checks passed" and returns every result as a DataFrame. |

---

## [`logs.py`](../src/kalpa/logs.py): recording decisions as you go

It adds rows to the markdown tables in `docs/` (two levels above this folder), so a decision is written down when it's made rather than afterwards.

- **`_append_row(path, header_fragment, cells)`**: the helper all the others use.
  - It finds the first markdown table whose header contains `header_fragment`, skips past the `|---|` separator line and adds the new row after the table's last row.
  - It never edits existing rows.
  - It replaces `|` in cell text with `/` so the table stays intact.
  - It raises an error if it can't find the table.
- **The public functions** each write to one table:

| Function | File and table |
|---|---|
| `decision(...)` | `decisions-log.md`, decisions to drop, default or fix records |
| `reconciliation(...)` | `decisions-log.md`, row counts in, kept and rejected; it computes the `yes`/`NO` balance flag itself |
| `join(...)` | `decisions-log.md`, joins made and row counts after |
| `cold_run(...)` | `decisions-log.md`, notes from re-running everything in a fresh environment |
| `challenge(...)` | `challenges-log.md`, what broke and what you tried |
| `approach(...)` | `approach-log.md`, options considered and the one chosen |

---

## `__pycache__/`

Python creates this folder automatically. It holds compiled bytecode (`*.cpython-313.pyc`, so you're on Python 3.13). You can ignore it, and it shouldn't be committed.

---

## Things worth knowing

1. **Checks pile up when you re-run cells.** `checks._RESULTS` lives in the module, so re-running a notebook cell in the same kernel adds duplicate entries and inflates `summary()`. Restart the kernel before a final run, or add a `reset()` function.
2. **The repeated-header test needs an exact match.** `read_legacy_file` only drops a repeated header that matches the first line character for character. A copy with trailing spaces or `\r` would stay in as a data row. Those rows would probably fail parsing and validation later, so they'd land in the rejects instead of being counted as headers.
3. **Bad legacy dates may get past the model.** Legacy dates that fail to parse become `NaT` before they reach the `Booking` model. Any comparison with `NaT` is `False`, so the slot-before-creation check could miss these rows instead of rejecting them. It's worth adding a test for this case.
4. **The ID regexes and invoice logic are written twice.** `ID_PATTERNS` in `loaders.py` repeats the ID regexes in `models.py`, and the invoice clean-up exists in both files. If one copy changes, the other has to change too.
