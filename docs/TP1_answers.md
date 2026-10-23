# Lab sheet 1: Source Control Management

**Team members**
- DJERADI Fares Abdelouahab
- RECIOUI Mohammed
- SACI Abdelhak
- ROUIGHI Sid Ahmed Yacine

**Repository:** https://github.com/FLOoKiZz/devops-project

---

## Exercise 1: Identity and import

**(a)** The two files whose names start with a dot are `.env.example` and `.gitignore`.

**(b)** With `cp -r .../ctf-scoreboard/*`, the shell does not match names that start with a dot. So `.env.example` and `.gitignore` would not be copied. Without `.gitignore`, files such as `.env`, the virtual environment or the database could be added to Git by accident.

**(c)** `.gitignore` only works for files that Git does not track yet. If `.env` is already in a commit, Git keeps tracking it, and the file stays in the history. Adding it to `.gitignore` later changes nothing. We would need `git rm --cached .env`, and we would have to change (rotate) any secret that was inside the file.

---

## Exercise 2: A conflict on purpose

**(a)** Both branches started from the same commit and changed the same line (`points:` of `the-long-game`) to different values (450 and 600). Git cannot know which value is correct, so it stops and asks a human to choose.

**(b)** If the two changes were on two different challenges, they would be in different places of the file (different hunks). Git can merge them by itself, with no conflict.

**(c)** The check can run in two places:
- locally, in a `pre-commit` hook, before each commit;
- on the server, in a CI job that runs on every pull request.

The CI check is stronger, because a local hook can be skipped with `git commit --no-verify`.

---

## Exercise 3: Find a bad commit with bisect

Result from the run in our shared repository:

- First bad commit: `8d97bbffc03e23e8974d73decfecee1788bed840` ("refactor(scoring): simplify the point total")
- Revert commit: `60d1d34`
- Number of `running '/tmp/check.sh'` lines: 5

**(a)** `git bisect run` needed **5 steps** to search the 31 commits.

**(b)** ⌈log₂ 31⌉ = 5, because 2⁴ = 16 is too small and 2⁵ = 32 is enough. The two numbers are equal. Each step tests the middle commit and removes half of the remaining commits.

**(c)** If `check.sh` is inside the repository, bisect checks out old commits, and in those commits the script may not exist or may be a different version. Then the test result is not reliable and bisect can point to the wrong commit. This is why the script is in `/tmp`.

**(d)** If the 31 commits are squashed into one commit, bisect can only return that one commit. It does not tell us which change caused the bug.

---

## Exercise 4: A secret that stays in the history

**(a)** Exit codes:
- `gitleaks dir .` (working tree): `exit=0`, no leak found.
- `gitleaks git -v .` (history): `exit=1`, 1 leak found.

**(b)** The rule that matched is `aws-access-token` (an AWS access key). It was found in `src/scoreboard/config.py` in the first scratch commit.

**(c)** Every commit stores the hash of its parent, so the history is a chain. The second commit does not edit the first one. It only creates a new snapshot of the files, without the key. The first commit is still in the chain, so anyone can read the key there.

**(d)** The first two steps, in order:
1. Revoke or rotate the key (make the old key useless).
2. Clean the history (for example with `git filter-repo`), force-push, and tell the team to re-clone.

The order matters: the key must be treated as already stolen once it was pushed. Cleaning the history cannot take back a copy that someone already has, so the key must be disabled first.

---

## Exercise 5: One shared, protected repository

**(a)** Without the rule, one stolen developer password may be enough for an attacker to push code directly to `main`, or rewrite or delete its history. With the rule enforced and not bypassed, changes must go through a pull request and receive another person's approval; force pushes and deletions are blocked. This makes it harder for one compromised account or a malicious insider to alter `main` unnoticed, so branch protection is also a security control, not only a quality step.

**(b)** It makes a supply-chain attack harder, for example when an attacker uses a compromised developer account to inject malicious code into the project.

**(c)** On a free GitHub account, branch protection is only available for public repositories. Private repositories need a paid plan.
