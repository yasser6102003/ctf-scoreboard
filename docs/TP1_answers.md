Lab 1: Source Control Management
Module: DevOps
Academic Year: 2026–2027
Instructor: Dr. Ahlem Belabbaci
Team: -Yasser Chiheb 
-Abdel Moumene Senoune
Repo: https://github.com/yasser6102003/ctf-scoreboard
Exercise 1
a) The two hidden files are .gitignore and .env.example.
b) The shell expands * into filenames before running cp, but it does not include 
hidden files, .gitignore and .env.example would be left out. Without
.gitignore, git add . could also add files such as .env, data/scoreboard.db, or
 pycache /. Using ctf-scoreboard/. copies everything inside the folder, including 
hidden files.
c) .gitignore only affects files Git is not already tracking. If .env is already 
committed, adding it to .gitignore will not remove it from Git or from previous 
commits. We would need git rm --cached .env and a new commit to stop tracking 
it. If it contained a secret, we should treat it as leaked and change it.
Exercise 2
a) Git compares both branches with their common ancestor, where the value was 
500. Since both branches changed the same line to different values, Git cannot 
decide which value to keep, so we must resolve the conflict ourselves.
b) If we change two different challenges on separate lines, Git can usually 
merge the changes automatically because each branch changed a different
part of the file. However, changes to nearby lines can sometimes still cause a
conflict.
c) We can check for conflicts locally using a pre-commit hook, such as check￾merge-conflict, or git diff --cached --check. We can also run checks in GitHub 
Actions for every push and pull request. Making the CI check required helps 
prevent changes containing conflict markers from being merged.
Exercise 3
a) Bisect ran the test five times, checking changelog entries 15, 22, and
19, followed by e1ab973 and entry 20.
b) ceil(log2(31)) = 5. This matches our result. Each test roughly halves the 
remaining commits, while checking them one by one could require up to 31 tests.
c) If check.sh were stored in the repository, it could be missing or different in older 
commits. Bisect might then run a different test or fail, making the result 
unreliable. Keeping the script in /tmp means the same test is available 
throughout
the search.
d) Bisect would identify the large commit as the first bad commit, but we would 
not know which of its 31 changes caused the bug. Smaller commits that each 
make one focused change make debugging much easier.
Exercise 4
a) The working-tree scan returned exit code 0, with no leaks found. The history 
scan returned exit code 1 and found one leak in 37 commits, in commit 2ec3ac7.
b) The rule was aws-access-token. It matched because the value starts with AKIA, 
a common prefix for AWS access key IDs.
c) Each commit records a snapshot of the files and references its parent. Restoring 
a file in a later commit does not remove the earlier snapshot containing the key. 
Changing an old commit also changes its hash, which affects the hashes of later 
commits. Removing the key from the repository's history therefore requires 
rewriting that history.
d) First, revoke the exposed key through AWS IAM, create a replacement, and 
check the logs for suspicious use. Then remove the key from the repository's 
history using a tool such as git filter-repo or BFG, force-push the cleaned history, 
and ask collaborators to clone the repository again. Store the replacement key in 
an environment variable or secret manager instead of the source code.
This order matters because public repositories can be scanned quickly, and copies 
may remain in forks, clones, or logs. Revoking the key first makes the exposed 
credential unusable even if those copies still exist.
Exercise 5
a) Without branch protection, an attacker using a developers account could push 
changes directly to main without a review. They could introduce a backdoor or 
code that exposes secrets, and the pipeline from Lab Sheet 2 could then build and 
potentially deploy those changes.
b) attacker could also force-push to rewrite history or delete the main branch. 
Branch protection blocks these direct actions and requires a pull request with 
another person's approval. This makes compromising one account less powerful, 
although the protection is not a complete guarantee if other accounts or 
permissions are compromised.
c) The lab requires a public repository. On a free GitHub account, the lab uses 
public repositories for branch protection (Lab Sheet 1), pulling images from GHCR 
without a password (Lab Sheets 3 and 4), and build attestations (Lab Sheet 6). 
Since this repository contains no private information and docs/FLAGS.md 
intentionally contains public flags, making it public is suitable for the lab.