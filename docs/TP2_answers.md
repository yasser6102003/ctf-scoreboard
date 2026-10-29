Lab 2: Continuous Integration

Module: DevOps

Academic Year: 2026–2027

Instructor: Dr. Ahlem Belabbaci

Team: Yasser Chiheb, Abdel Moumene Senoune

Repo: https://github.com/yasser6102003/ctf-scoreboard

Exercise 1 answers:

a) A push run only happens after the change is already on main, so a problem is found too late. A pull_request run happens before the merge: it tests the proposed change, shows the result on the pull request, and with a required check it can block the merge. So bad code never reaches main.

b) Lint is the cheapest and fastest gate (0 s in our run, tests took 5 s). It runs first so that simple problems are reported in seconds. This is "fail fast": the cheap check gives feedback before the expensive one.

c) With a 20-minute test suite, a lint error would otherwise be found only after waiting 20 minutes. Running the cheap gate first gives the answer in seconds, so the developer can fix it at once, and the team does not waste 20 minutes of runner time on a change that is already wrong.

Exercise 2 answers:

a)`git commit --no-verify` (short form: `git commit -n`). It skips the pre-commit hook.

b)Only someone with admin rights on the repository, for example the owner . An admin can edit or disable the ruleset protect-main, or remove verify from the required checks, and then merge or push without a green pipeline. A normal collaborator cannot: the merge button stays grey until verify is green and a teammate approves.

Exercise 3 answers:

a) E   ModuleNotFoundError: No module named 'scoreboard'

b) The project uses a src layout: the package is in src/scoreboard/, not at the top of the repository. When pytest runs, Python looks for packages in the current folder and in the installed packages, so it does not find src/scoreboard. The line pip install -e . installs the package in the environment (in editable mode), so import scoreboard works. Without it, tests/conftest.py fails on from scoreboard.app import create_app.

c) make install. In the Makefile, the install target runs .venv/bin/pip install -q -e . for us.

Exercise 4 answers:

Pull request #12 (chaos/tp2), team name chiheb-senoune. The table is also in docs/TP2_diagnosis.md, committed on chaos/tp2 before the fixes (commit feef52f).

| Defect | Gate that failed | Name in the log | Assertion or exception | One-line cause |
|---|---|---|---|---|
| Test 1 | tests | tests/test_challenges.py::test_defaults_are_applied | Assertion (AssertionError: assert 100 == 10) | The default value of decay_solves for a challenge is 100 instead of 10. |
| Test 2 | tests | tests/test_challenges.py::test_a_missing_file_is_reported_clearly | Exception (FileNotFoundError) | A missing challenges file is not reported clearly anymore: the raw FileNotFoundError comes out of the loader. |
| Credential | secret scan | generic-api-key in src/scoreboard/db.py, line 31 (STAGING_DB_CREDENTIAL) | — | A hard-coded credential was added to db.py. |
| Dependency | dependency audit | urllib3 2.0.5: 18 known vulnerabilities (for example PYSEC-2023-192) | — | requirements.txt pins an old urllib3 version with known vulnerabilities. |

Exercise 5 answers:

We fixed the four defects with one commit each, without changing any test: 34027aa (decay_solves back to 10), 08af82e (the missing-file check is back in load()), 9fe54dd (the credential is removed), 0e7942b (the urllib3 pin is removed). After that, make test gives 73 passed and the pipeline on pull request #12 is green.

a) urllib3 is a transitive dependency. It is not in our requirements.txt; it is installed because requests needs it.

b) grep urllib3 requirements.txt prints nothing, and .venv/bin/pip show urllib3 shows the line "Required-by: requests".

Screenshots:

All the commands we ran on our computer are in docs/screenshots/ (command and full output):

01-before-make-test-lint.png: before you start, make test (73 passed) and make lint (nothing printed). We took this one again on main at the end, because we had closed the terminal of the first run.

02-partA-ci-yml.png: Part A, the pipeline file .github/workflows/ci.yml.

03-partC-no-install-commit.png: Part C, the commit db76980 that removes the line pip install -e . from the pipeline.

04-partC-cleanup.png: Part C, deleting the branch scratch/no-install on our computer and on GitHub.

05-partD-git-diff-defects.png: Part D, git diff main -- src requirements.txt after the diagnosis, showing the four defects.

06-partD-gitleaks-verbose.png: Part D, gitleaks dir -v --redact . to get the RuleID, the file and the line of the credential.

07-fix1-decay-solves.png: fix 1, commit 34027aa (decay_solves back to 10).

08-fix2-missing-file.png: fix 2, commit 08af82e (the missing-file check is back in load()).

09-fix3-credential.png: fix 3, commit 9fe54dd (the credential is removed).

10-fix4-dependency.png: fix 4, commit 0e7942b (the urllib3 pin is removed).

11-partD-commit-order.png: the commits of chaos/tp2 in order: the four defects, then the diagnosis feef52f, then the four fixes.

12-after-fixes-install-test-lint-push.png: after the fixes, make install, make test (73 passed), make lint and git push.

13-ex5-pip-show-urllib3.png: Exercise 5, grep urllib3 requirements.txt and .venv/bin/pip show urllib3.
