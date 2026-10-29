# Lab sheet 2: diagnosis from the pipeline log

Pull request #12 (`chaos/tp2`), team name `chiheb-senoune`. This diagnosis comes from the pipeline log only, before looking at the code changes.

| Defect | Gate that failed | Name in the log | Assertion or exception | One-line cause |
|---|---|---|---|---|
| Test 1 | tests | `tests/test_challenges.py::test_defaults_are_applied` | Assertion (`AssertionError: assert 100 == 10`) | The default value of `decay_solves` for a challenge is 100 instead of 10. |
| Test 2 | tests | `tests/test_challenges.py::test_a_missing_file_is_reported_clearly` | Exception (`FileNotFoundError: [Errno 2] No such file or directory: '/nonexistent/challenges.yaml'`) | A missing challenges file is not reported clearly anymore: the raw `FileNotFoundError` comes out of the loader. |
| Credential | secret scan | `generic-api-key` in `src/scoreboard/db.py`, line 31 (`STAGING_DB_CREDENTIAL`) | — | A hard-coded credential was added to `db.py`. |
| Dependency | dependency audit | `urllib3` 2.0.5: 18 known vulnerabilities (for example `PYSEC-2023-192`, `PYSEC-2026-1995`) | — | `requirements.txt` pins an old `urllib3` version with known vulnerabilities (fixed in newer versions, up to 2.8.0). |

Note: the pipeline log of the secret scan only shows `leaks found: 1`, because the pipeline runs gitleaks without `-v`. We ran the same scan on our computer with `gitleaks dir -v --redact .` to get the RuleID, the file and the line.
