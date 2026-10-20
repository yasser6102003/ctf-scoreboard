# TP1 - Source Control Management

Team: Yasser Chiheb (yasser6102003, owner of the repo) and Abdel Moumene Senoune (asenouneinf-rgb).
We are only 2 in the team, so each of us reviewed the other's pull request.

Repo: https://github.com/yasser6102003/ctf-scoreboard

---

## Exercise 1

**a)** The two files that start with a dot are `.gitignore` and `.env.example`.

**b)** With `*` the shell replaces the star with the file names before `cp` runs,
and `*` does not match hidden files (names starting with a dot). So `.gitignore`
and `.env.example` would not be copied. The bad part is that without `.gitignore`,
`git add .` would also add things like `.env`, the database `data/scoreboard.db`
or `__pycache__/`. With `ctf-scoreboard/.` we copy everything inside the folder,
hidden files too.

**c)** `.gitignore` only works for files that Git is not tracking yet. If `.env` is
already in a commit, Git keeps tracking it even after we add it to `.gitignore`.
And the old commit still has the file inside, so anyone who cloned the repo can
read it. We would need `git rm --cached .env` and a new commit, but the file is
still in the history. So if there was a secret in it, we have to consider it
leaked and change it.

---

## Exercise 2

On `main` we changed the-long-game to 600, and on `tune/lower` to 450. The merge
gave a conflict in `data/challenges.yaml`. We kept 600, removed the markers and
committed the merge (`8681f47`).

**a)** Git compares the two branches with their common ancestor, where the value
was 500. Here both branches changed the same line, but to different values (600
and 450). Git can't know which one is right, it's a decision for a person, so it
stops and shows the conflict.

**b)** If we change two different challenges, the changes are on different lines
of the file. For each part only one branch changed something, so Git just takes
that change and the merge works alone. Git merges line by line, not file by file.
(If the two changed lines are right next to each other, it can still conflict.)

**c)** It can run in two places:
- on our own machine before each commit, with a pre-commit hook (for example the
  `check-merge-conflict` hook, or `git diff --cached --check`);
- on GitHub in the CI pipeline (GitHub Actions) on every push and pull request.
  If we make it a required check on `main`, a file with conflict markers can't be
  merged even if someone skips the local hook.

---

## Exercise 3

`git bisect run` found the bad commit:
`e1ab973 refactor(scoring): simplify the point total`.
We reverted it (`e5cee52`) and `/tmp/check.sh && echo repaired` printed `repaired`.

**a)** Bisect ran the test 5 times (on changelog entry 15, entry 22, entry 19,
`e1ab973` and entry 20), so 5 steps.

**b)** log2(31) is about 4.95, so the ceiling is 5. It's the same as what we got.
Every test cuts the remaining commits in half (31, 15, 7, 3, 1), like a binary
search. Testing commits one by one could take up to 31 tests.

**c)** If `check.sh` is in the repo, it changes every time bisect checks out
another commit. In the old commits the script doesn't exist yet, so the command
fails and bisect thinks the commit is bad (or it stops). In other commits it could
be a different version of the script. So we wouldn't be running the same test on
every commit and the result would be wrong. That's why we put it in `/tmp`,
outside the repo.

**d)** Bisect would just say that the one big commit is the first bad commit. It's
true, but it doesn't help much: we know the bug is somewhere in the 31 changes but
not which one. Bisect can't go smaller than one commit, so small commits that do
one thing each make it much more useful.

---

## Exercise 4

On the branch `scratch/secret` we made two commits: `2ec3ac7` adds a fake AWS key
to `config.py`, and `83b7607` puts the file back like before. We never pushed this
branch and deleted it after the scan.

**a)**
- Scan of the working tree: exit code **0**, no leaks found (the file has no key now).
- Scan of the history: exit code **1**, 1 leak found in 37 commits (in commit `2ec3ac7`).

**b)** The rule is `aws-access-token`. It matched because the value starts with
`AKIA`, which is how AWS access keys start.

**c)** A commit is a full snapshot of the files plus the hash of its parent. The
second commit doesn't change the first one, it just adds a new snapshot on top
and keeps `2ec3ac7` as its parent. So the old snapshot with the key is still there
and still reachable. And we can't edit it: if we change `2ec3ac7` its hash changes,
then the parent hash saved in `83b7607` is wrong, so that one changes too, and so
on for all the commits after. The only way to really remove the key is to rewrite
the history and force push.

**d)**
1. First, revoke the key at the provider (in AWS IAM) and make a new one. Also
   check the logs to see if someone used it.
2. Then clean the repo: remove the key from all the history (`git filter-repo` or
   BFG), force push, and tell everyone to clone again. The new key goes in an
   environment variable or a secret manager, not in the code.

Why this order: once a key is pushed to a public repo we have to assume someone
already copied it. Bots scan GitHub all the time and find keys in minutes, and it
can also be in forks, clones or CI logs. Cleaning the history doesn't delete those
copies and takes time. Revoking the key makes all the copies useless right away,
so it has to come first.

---

## Exercise 5

We protected `main` with a rule: a pull request is required, with 1 approval,
and force pushes and deletion are blocked. Then each of us made a change on a
branch and the other one reviewed it:
- PR #1 (`docs/scoring-docstring`, Yasser): approved and merged by Abdel Moumene (`f7094a9`).
- PR #2 (`docs/contributing`, Abdel Moumene): approved and merged by Yasser (`21044d6`).

**a)** Without the rule, the attacker can log in as that developer and push
whatever they want directly to `main`, with no review. For example, they can add
a backdoor or code that steals secrets. Nobody has to look at it before it is in
`main`, and from lab sheet 2 the pipeline would build it and later deploy it.

**b)** They can also force push to `main`. That rewrites the history: they can
hide their own commit inside an old one, or delete the work of the team. They can
even delete the `main` branch. With the rule, they can only open a pull request,
and a second person must approve it. So one stolen password is not enough, the
attacker needs two accounts.

**c)** The box says to use a public repository. On a free GitHub account, three
features only work on public repositories: branch protection (lab sheet 1),
pulling images from `ghcr.io` without a password (lab sheets 3 and 4) and build
attestations (lab sheet 6). Nothing in this repository is secret (the flags in
`docs/FLAGS.md` are public on purpose), so making it public costs nothing.
