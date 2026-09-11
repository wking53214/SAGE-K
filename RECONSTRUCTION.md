# Reconstruction record

The original `wking53214/SAGE-K` no longer exists. This repository was rebuilt
in September 2026 from the account's conversation archives. This file documents
how, what was verified, and every point where this copy knowingly differs from
the original. Read it before treating anything here as historically exact.

## Target state

The reconstruction targets the original repository's most developed state:
commit **`baf5ec8`** — *"Rebuild as installable package; fix adapter sync/async
bug and chain-verification bug"* (2026-08-14), which replaced three
non-executing archived artifacts with an installable package.

That state is independently documented in `docs/SEAM_INVENTORY.md`, a 39-seam
boundary analysis produced against a clone of the live repository at that
commit. It records 20 tracked files, roughly 4,470 lines, and 30 components.
Because it was written from the real repository rather than from memory, it
served as the specification this rebuild was checked against.

## Sources

| Part | Recovered from |
|---|---|
| `kernel.py`, `gsa_adapter.py`, `graph_extractor.py`, `tests/test_sanity.py`, all three `examples/`, `pyproject.toml`, `requirements.txt`, `.gitignore` | Claude archive, conversation `dd8404f9` (2026-08-14) — the rebuild session itself, captured with full file contents |
| `scenarios.py`, `generator.py`, `harness.py`, `drift.py`, `realignment.py`, `report.py`, `__init__.py`, `demo.py`, `tests/test_interpretation.py` | Claude archive, conversation `39aecf97` (2026-08-06) — the authoring session for the `interpretation` package |
| `docs/SEAM_INVENTORY.md` | Claude archive, conversation `5e8b9b1a` (2026-08-14) |
| `docs/GEMINI_SOURCE_RECORD.md` | Gemini Takeout export, activity record `2026-07-04T20:27:43.223Z` |

Module sources were recovered as complete file contents, not paraphrased or
regenerated. The two post-creation edits to `gsa_adapter.py` from the rebuild
session were recovered as exact text replacements and re-applied in order.

## Structural corroboration

Line counts were checked against the seam inventory's independent measurements
of the live repository:

| File | Seam inventory | This copy | |
|---|---|---|---|
| `gsa_adapter.py` | 296 | 296 | exact |
| `kernel.py` | 443 | 443 | exact |
| `graph_extractor.py` | 167 | 167 | exact |
| `__init__.py` | 90 | 90 | exact |
| `scenarios.py` | 266 | 266 | exact |
| `drift.py` | 303 | 303 | exact |
| `realignment.py` | 370 | 370 | exact |
| `report.py` | 302 | 302 | exact |
| `harness.py` | 263 | 257 | **−6** |
| `generator.py` | 211 | 225 | **+14** |

`gsa_adapter.py` reaching exactly 296 only after the two recovered edits were
applied is the strongest single check here: the edits were recovered
independently of the line count they had to produce.

## Behavioural verification

`examples/run_kernel_stress_test.py` is seeded, so its output is a fingerprint.
The archive records the original repository producing `GREEN: 20, YELLOW: 0,
RED: 0`. This copy produces:

```
=== DUAL PHASE RESULTS ===
GREEN : 20
YELLOW: 0
RED   : 0
```

Byte-identical. Also verified: the package imports cleanly, all three examples
run to completion, the five-year interpretation demo runs end to end, and the
full suite passes — 43 tests (4 from `test_sanity.py`, 39 from
`test_interpretation.py`).

## Divergences from the original

### 1. `harness.py` and `generator.py` differ by a few lines

These two were recovered from the 2026-08-06 authoring session, not from the
SAGE-K repository itself. Between being authored and being measured by the seam
inventory, each picked up small changes that exist in no surviving archive
record. The recovered versions are functionally complete and fully exercised by
the test suite, but they are not line-for-line the SAGE-K copies. Unrecoverable
without the original repository.

### 2. `requirements.txt` — two conflicting archive records

The rebuild session wrote a stdlib-only `requirements.txt`. The seam inventory,
analysing the same commit, recorded instead eight pinned packages — `fastapi`,
`uvicorn`, `psycopg2-binary`, `anthropic`, `pytest`, `numpy`, `pydantic`,
`python-dotenv` — of which only `numpy` (soft) and `pytest` (tests only) appear
anywhere in the code. The seam analysis flagged this as seam **S35**: the file
implies a web service, a Postgres store and an API client that do not exist in
the tree, and installation plus the full test run succeeded with none of the
eight installed.

**This copy ships the stdlib-only version.** It is accurate, it agrees with
`pyproject.toml`'s `dependencies = []`, and reproducing the eight-package file
would be reproducing a defect the original analysis had already identified as
misleading. The conflict is recorded here rather than hidden.

### 3. `TRANSCRIPT.md` could not be fully recovered

The original retained a 1,352-line verbatim chat log. The Gemini Takeout export
preserves rendered activity records, not full conversation threading, so only
two SAGE-K-notebook records survive in the archive. The substantive one — the
consolidated module the conversation produced — is preserved in
`docs/GEMINI_SOURCE_RECORD.md` under that filename rather than as `TRANSCRIPT.md`,
so the name does not promise completeness it cannot deliver.

### 4. Restored: `demo.py` and `tests/test_interpretation.py`

Neither appears in the seam inventory's component list, so neither was in the
original repository at `baf5ec8`. Both were authored in the same 2026-08-06
session as the interpretation modules and were simply not carried across in the
merge. They are restored here because they are genuine artifacts of this code,
and their absence was the single largest capability loss in the original: the
repository shipped six governance modules with **zero** tests covering them,
while its only test file covered the other subsystem. Restoring the suite takes
test count from 4 to 43.

This is the one place where this copy is deliberately *more* than the original
state it targets. It adds no new code — only code that already existed for these
exact modules.

### 5. Fixed: `demo.py` handed answers to the wrong scenarios

A real defect in the restored demo, found by running it. `ScenarioLibrary.all()`
sorts by `scenario_id`, which is a UUID, so iteration order is deliberately not
authoring order. The demo's year-by-year answer patterns were handed out
positionally, so each answer reached an arbitrary scenario. Most then fell
outside that scenario's own options and scored as `ERROR`, which made every zone
report "no data" — erasing exactly the multi-year drift the demo exists to show.

Fixed in `demo.py` only, by binding each year's answers to scenario IDs up front
instead of dealing them out positionally. No library code was touched; the
library was behaving correctly, including when it rejected out-of-option answers.
The demo now reports zero errors and shows the intended degradation: geographic
scope holding at 100%, proxy correlation slipping to 50%, thin-file applicants
breaching at 0%, overall risk HIGH.

### 6. Left as-found: the package facade hides subsystem B

`import sage_k` exposes only the six interpretation modules. `kernel`,
`gsa_adapter` and `graph_extractor` export nothing at package level and must be
imported as submodules. The seam inventory catalogued this as seam **S1** and
found no comment, doc, or commit message explaining it — so whether it was
intentional is genuinely unknown.

It was left exactly as the original had it. Changing it would alter the public
API of the artifact being reconstructed on a guess about intent, and the failure
mode the seam analysis named — a consumer of the package-level API concluding
those three modules do not exist — is better addressed by documenting it, which
`README.md` now does.

### 7. `__all__` length

The seam inventory describes the facade as exporting "30 named symbols from six
modules". The recovered facade is 90 lines, matching exactly, but its `__all__`
lists 40 entries. Unresolved; most likely a counting difference in the original
analysis rather than a different file, since every other measurement of this
file agrees.

## What is not here

- The original git history. This is a fresh reconstruction; commit `baf5ec8` and
  the initial artifact commit exist only as archive references.
- `artifact_1.py`, `artifact_2.py`, `artifact_3.py`. The rebuild deleted them
  deliberately and none executed. Their substance survives in
  `docs/GEMINI_SOURCE_RECORD.md`.
- Any `Resolver` or `ModelClient` implementation. Both are protocols with no
  production implementation, in the original and here. The `Resolver` seam is
  where Sentinel was intended to attach.
