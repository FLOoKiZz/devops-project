# Lab sheet 2: Continuous Integration

## Exercise 1

**(a)** A `pull_request` run checks a change before it is merged into `main`.
That gives the team a chance to catch a problem before it becomes part of the
main branch. A `push` run checks a change after it has been pushed to the
branch.

**(b)** Linting is quick, so it is useful to run it before the tests. If the
style check already fails, the pipeline can stop early instead of spending
time on tests that take longer.

**(c)** The order matters even more when tests take 20 minutes. A quick lint
failure can be reported right away, rather than making everyone wait for the
full test suite to finish.

## Exercise 2

**(a)** The `--no-verify` option to `git commit` skips local commit hooks.
That is why local hooks are useful but cannot be the only check.

**(b)** A user who is allowed to bypass the branch protection rule could still
merge a change that the pipeline says contains a secret. The status check
helps prevent that when the rule is enforced and cannot be bypassed.

## Exercise 3

**(a)** The CI error was:
`ModuleNotFoundError: No module named 'scoreboard'`.

**(b)** The application uses a `src` layout: its package is in `src/scoreboard`,
not at the repository root. The tests import `scoreboard`, so Python needs the
package to be installed before it can find it.

**(c)** The workflow's `pip install -e .` command installed the project for the
tests. Removing that command caused the import error.

## Exercise 4

The pipeline log showed:

| Defect | Gate | Finding |
|---|---|---|
| Test 1 | Tests | `test_submitting_for_an_unknown_team_is_not_found` failed with an `AttributeError`: `'NoneType' object has no attribute 'id'`. This was an exception. |
| Test 2 | Tests | `test_last_solve_time_is_the_most_recent_not_the_last_added` failed because the returned time was 12:05 instead of the expected 12:30. This was an assertion failure. |
| Credential | Secret scan | Gitleaks reported 2 findings. The captured output did not include the rule IDs or file locations. |
| Dependency | Dependency audit | `idna` 3.6 was reported with `PYSEC-2024-60` (fixed in 3.7) and `PYSEC-2026-215` (fixed in 3.15). |

The full diagnosis was committed in `docs/TP2_diagnosis.md` before we inspected
the code changes.

## Exercise 5

**(a)** The vulnerable package was a transitive dependency, not one of the
direct dependencies listed in `requirements.txt` on `main`. The seeded branch
added `idna==3.6`, and the audit reported vulnerabilities in that version.

**(b)** I checked with `.venv/bin/pip show idna`. Its `Required-by` line listed
packages including `requests`, which showed that other packages depend on
`idna`. It is not listed as a direct dependency in `requirements.txt` on
`main`.
