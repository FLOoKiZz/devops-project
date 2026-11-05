# Lab sheet 2: diagnosis from the pipeline log

| Defect | Gate that failed | Name in the log | Assertion or exception | One-line cause |
|---|---|---|---|---|
| Test 1 | tests | `test_submitting_for_an_unknown_team_is_not_found` | Exception: `AttributeError: 'NoneType' object has no attribute 'id'` | Submitting for a team that does not exist raised an exception instead of returning the expected 404. |
| Test 2 | tests | `test_last_solve_time_is_the_most_recent_not_the_last_added` | Assertion: actual time was 12:05; expected time was 12:30. | The reported last-solve time did not match the most recent solve time. |
| Credential | secret scan | Gitleaks reported 2 findings; the captured log did not show RuleID, file, or line. | — | The secret-scanning gate detected secret-like content in the working tree. |
| Dependency | dependency audit | `idna` 3.6; `PYSEC-2024-60` (fixed in 3.7) and `PYSEC-2026-215` (fixed in 3.15). | — | The audit reported known advisories for the installed `idna` version. |
