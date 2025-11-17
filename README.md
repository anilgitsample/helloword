# helloword
# Helloworld — Actions taken & step-by-step guide (full document)

Below is a complete, practical record of **everything we did** together for the `helloworld` repository under `anilgitsample`, plus the exact commands you ran and the recommended workflow going forward. Copy-paste the commands where needed.

---

## 1 — Project & repo creation

We created a local project `helloworld` and pushed it to GitHub under `anilgitsample`.

Example commands used:

```bash
mkdir helloworld
cd helloworld
echo "# helloworld" > README.md
echo "console.log('hello world');" > index.js

git init
git add .
git commit -m "chore: initial commit"

# either via gh CLI:
gh repo create anilgitsample/helloworld --public --source=. --remote=origin --push

# or if created on GitHub web UI:
git remote add origin git@github.com:anilgitsample/helloword.git   # use the actual repo name
git branch -M main
git push -u origin main
```

---

## 2 — Branching strategy implemented

We used these branches:

* `main` — production copy
* `develop` — integration branch for developer changes
* `feat-develop`, `feat2-develop` (example) — feature branches for stories

Commands to create and push branches:

```bash
# create develop from main
git checkout -b develop
git push -u origin develop

# create a feature branch and push
git checkout -b feat-develop
# make changes...
git add .
git commit -m "feat: add placeholder"
git push -u origin feat-develop
```

---

## 3 — Creating files and committing selective files

Examples you did:

* Created `hello.yml` in `feat-develop` and pushed:

```bash
git add hello.yml
git commit -m "created hello.yml file"
git push origin feat-develop
```

* Later created `school.yml` in `feat2-develop` and pushed:

```bash
git checkout feat2-develop
touch school.yml
git add .
git commit -m "just created school.yml"
git push origin feat2-develop
```

Notes:

* To commit only one file: `git add file1 && git commit -m "msg"`.
* To stage selected hunks: `git add -p file`.

---

## 4 — Creating Pull Requests (PRs) with `gh`

You created PRs via `gh` CLI:

Examples:

```bash
# from feat-develop -> develop (PR #4)
gh pr create --base develop --head feat-develop --title "created file" --body "created hello.yml file"

# from feat2-develop -> develop (PR #5)
gh pr create --base develop --head feat2-develop --title "created school.yml" --body "created file school.yml in feat2-develop branch"
```

You can list PRs:

```bash
gh pr list
```

---

## 5 — Merging PRs (we used `gh pr merge` or local merge)

Preferred (used/recommended): `gh pr merge <pr-number> --merge --delete-branch`

Examples:

```bash
gh pr merge 4 --merge --delete-branch    # merges PR #4 and deletes remote branch
# or squash
gh pr merge 5 --squash --delete-branch
```

Local merge alternative (if you do not use `gh`):

```bash
git fetch origin
git checkout develop
git merge --no-ff origin/feat2-develop -m "Merge feat2-develop: add school.yml"
git push origin develop
git push origin --delete feat2-develop     # remove remote branch
git branch -d feat2-develop                # remove local branch
```

Common mistakes you hit:

* Typo in remote name: `git push orign` → should be `origin`.
* Wrong merge flags: `--no-f` (typo) vs correct `--no-ff`.
* Attempted `git pr`—not valid (use `gh pr`).

---

## 6 — SSH key setup & permissions (resolved)

Problem: `git push` failed with:

```
ERROR: Permission to anilgitsample/helloword.git denied to anilnlkllk.
```

Diagnosis: local `ssh` keyed to `anilnlkllk`. Needed to add the correct public key to `anilgitsample`.

What we did:

1. Generated/used a key named `~/.ssh/id_ed25519_personal` and its public `~/.ssh/id_ed25519_personal.pub`.
2. Added that public key to the **anilgitsample** GitHub account: pasted the contents of `id_ed25519_personal.pub` into **Settings → SSH and GPG keys → New SSH key** (Title `personal-key`).
3. Loaded the private key into ssh-agent and verified authentication:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_personal
ssh-add -l
ssh -T git@github.com   # should show "Hi anilgitsample!"
```

4. If multiple GitHub accounts on the same machine, we added SSH config:
   `~/.ssh/config`

```
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
  IdentitiesOnly yes
```

and set remote (optional) to use alias:

```bash
git remote set-url origin git@github-personal:anilgitsample/helloword.git
```

---

## 7 — GitHub Actions CI — `ci.yml`

We fixed a CI workflow file at `.github/workflows/ci.yml`. The **correct** YAML used:

```yaml
name: CI

on:
  push:
    branches:
      - main
      - develop
      - 'feat-*'
  pull_request:
    branches:
      - main
      - develop

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: |
          if [ -f package.json ]; then
            npm ci
          else
            echo "No package.json"
          fi

      - name: Run tests
        run: |
          if [ -f package.json ]; then
            npm test || exit 1
          else
            echo "No tests"
          fi
```

Important fixes we applied:

* `runs-on: ubuntu-latest` (spelling)
* Proper indentation
* `run:` blocks use `|` and each line inside is properly indented
* Fixed bash `if` syntax: `if [ -f package.json ]; then ... fi` (spaces required)

How to commit/update CI:

```bash
git checkout develop
git add .github/workflows/ci.yml
git commit -m "fix(ci): corrected YAML syntax and bash if statements"
git push origin develop
```

We validated CI runs on pushes to `develop` and PRs into `develop`/`main`. You saw an earlier failure from incorrect `if[ -f ... ]` syntax; after the fix CI succeeded.

---

## 8 — Troubleshooting checklist (common errors you encountered and fixes)

1. **`error: remote origin already exists.`**
   Fix: `git remote -v` and if needed `git remote set-url origin <url>`.

2. **`ERROR: Permission to ... denied to <user>.`**
   Fix: ensure correct SSH key in GitHub account or use HTTPS + PAT. For SSH, `ssh -T git@github.com` should show the correct username.

3. **`git: 'pr' is not a git command`**
   Fix: use `gh pr` (GitHub CLI) not `git pr`.

4. **`merge: create - not something we can merge`**
   Cause: wrong `git merge` syntax (you used `git merge create ...`). Correct: `git merge --no-ff origin/feat-branch` or use `gh pr merge <pr>`

5. **YAML parse errors / GitHub Actions `Invalid workflow file`**
   Fix: check indentation, remove tabs, ensure correct `runs-on` and `run: |` multi-line blocks. Use the corrected YAML above.

6. **Bash syntax errors inside `run:`**
   Fix: add proper spaces for `if [ -f file ]` and place multi-line scripts under `run: |`.

---

## 9 — Exact commands summary (cheat sheet)

### Git basics

```bash
git init
git add <file>
git commit -m "msg"
git branch -M main
git checkout -b develop
git checkout -b feat-branch
git push -u origin feat-branch
git pull origin develop
git merge --no-ff origin/feat-branch -m "Merge message"
git push origin develop
```

### GitHub CLI (`gh`) for PRs and merges

```bash
# create PR
gh pr create --base develop --head feat-branch --title "..." --body "..."

# list PRs
gh pr list

# merge PR (normal merge commit)
gh pr merge <pr-number> --merge --delete-branch

# squash and merge
gh pr merge <pr-number> --squash --delete-branch
```

### SSH agent & config

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_personal
ssh -T git@github.com
# optional custom host in ~/.ssh/config
```

### Update remote to use SSH alias (if you used ~/.ssh/config)

```bash
git remote set-url origin git@github-personal:anilgitsample/helloword.git
```

---

## 10 — Recommended workflow (daily)

1. Create feature branch from `develop`:

   ```bash
   git checkout develop && git pull origin develop
   git checkout -b feat-abc
   ```
2. Work, stage, commit:

   ```bash
   git add .
   git commit -m "feat: ..."
   git push -u origin feat-abc
   ```
3. Create PR: `feat-abc -> develop` (use `gh pr create` or GitHub UI).
4. Wait for CI & reviews. Once CI green and approvals done: merge PR (via `gh pr merge` or UI).
5. Locally: `git checkout develop && git pull origin develop` and delete local feature branch.

---

## 11 — Next steps for you right now

* If you have feature PRs open and CI passed, merge them (you used PR #4 & PR #5).

  * Via CLI: `gh pr merge 4 --merge --delete-branch` (and same for #5).
* After merging, update local `develop`:

  ```bash
  git checkout develop
  git pull origin develop
  ```
* If `develop` is stable and you want to release, create PR `develop -> main` and follow same review + merge process.

---

## 12 — Appendix: key files we changed

* `.github/workflows/ci.yml` — GitHub Actions workflow (fixed syntax & bash)
* `hello.yml`, `school.yml` — example files added in feature branches
* `README.md`, `index.js` — initial project files

---

If you want, I can:

* produce a printable PDF of this document,
* create a release checklist (develop → main) with commands and tagging examples,
* or tailor the CI workflow for a different stack (Python / Java / Docker) — tell me which.

Would you like the final document in Telugu instead, or is English fine?
