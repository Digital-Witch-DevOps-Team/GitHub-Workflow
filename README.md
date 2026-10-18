# Team Git & GitHub Collaboration Workflow

> **Purpose:** A shared, beginner-friendly guide for project managers, documentation teams, senior DevOps engineers, and junior contributors working in the `Digital-Witch-DevOps-Team` organisation repository.

## 1. Workflow at a glance

| Stage | Owner | What happens | Pull request destination |
|---|---|---|---|
| 1. Repository setup | Senior / organisation maintainer | Push the source project into the organisation repository and create `Dev-Team` | Not applicable |
| 2. Task development | Junior developer | Fork the organisation repository, clone the fork, and create a feature branch from `upstream/Dev-Team` | Not applicable yet |
| 3. Submit work | Junior developer | Commit and push the feature branch to their own fork | **Junior fork feature branch → organisation `Dev-Team`** |
| 4. Review and integrate | Senior reviewer | Inspect the changes, approve, and merge the PR | Merges into `Dev-Team` |
| 5. Verify development | Senior team | Update local `Dev-Team`, inspect commits/files, run project tests | Not applicable |
| 6. Release promotion | Senior team | Open a second PR after testing the integrated work | **Organisation `Dev-Team` → organisation `main`** |

### Branch and remote names

| Name | Meaning |
|---|---|
| `origin` in the senior clone | The organisation repository: `Digital-Witch-DevOps-Team/Terraform-Infrastructure-as-code-IaC` |
| `upstream` in the senior clone | The original source repository: `digitalwitchdemo/Final-Project-architecture-EKS-terraform` |
| `origin` in a junior clone | That junior's personal fork, for example `Ose-Jossy/Terraform-Infrastructure-as-code-IaC` |
| `upstream` in a junior clone | The organisation repository, not a single branch |
| `upstream/Dev-Team` | The organisation's remote-tracking branch the junior uses as the starting point for feature work |
| `Dev-Team` | Shared development/integration branch; work enters through reviewed PRs |
| `main` | Stable branch; promote tested changes through a separate PR |

---

## 2. Senior: initialise the organisation repository

**One-time setup.** The organisation repository should exist before these steps. Ideally, create it empty (without an automatically generated README, licence, or `.gitignore`) when importing an existing Git history. If it already contains separate commits, do not force-push; reconcile the histories first.

Run these commands from a parent folder where you want the project directory created. Open that folder in VS Code or use its integrated terminal.

```bash
# Clone the original/source project
git clone https://github.com/digitalwitchdemo/Final-Project-architecture-EKS-terraform.git
cd Final-Project-architecture-EKS-terraform

# Check the working tree and current branch
git status
git branch --show-current

# Set identity for THIS repository only (do not use --global)
git config user.name "YOUR GITHUB USERNAME"
git config user.email "YOUR GITHUB NOREPLY EMAIL"

# Rename the source remote, then make the organisation repo the push remote
git remote rename origin upstream
git remote add origin https://github.com/Digital-Witch-DevOps-Team/Terraform-Infrastructure-as-code-IaC.git

git remote -v
```

Check `git remote -v` before pushing. It should show:

- `origin` → `Digital-Witch-DevOps-Team/Terraform-Infrastructure-as-code-IaC.git`
- `upstream` → `digitalwitchdemo/Final-Project-architecture-EKS-terraform.git`

Then publish `main` and create `Dev-Team` from the current `main` history:

```bash
git push -u origin main
git switch -c Dev-Team
git push -u origin Dev-Team
```

**Important:** These commands assume the cloned source's current branch is `main`. Confirm with `git branch --show-current`. If it is named differently, stop and verify the correct default branch before pushing. Authenticate through Git Credential Manager/browser or your approved GitHub authentication method when prompted. Never paste a password or token into this README.

### Local Git identity

The `git config` commands above apply only to this repository. Use the exact GitHub-provided noreply email shown in **GitHub → Settings → Emails** if you want commits associated with the account. Repeat local identity configuration in each separate clone where needed.

---

## 3. Organisation: protect `Dev-Team` and `main`

In the organisation repository, open **Settings → Rules → Rulesets → New branch ruleset**. The exact labels can change slightly as GitHub updates its interface.

| Setting | `Protect Dev-Team` | `Protect Main` |
|---|---|---|
| Enforcement | Active | Active |
| Target branch | Include by pattern: `Dev-Team` | Include default branch (usually `main`) |
| Require a pull request before merging | On | On |
| Required approvals | 1 | 1 |
| Dismiss stale approvals when new commits are pushed | On | On |
| Block force pushes | On | On |

Keep the bypass list empty if the intention is for administrators and contributors to follow these rules too. Only add a bypass actor when the organisation has deliberately approved that exception. Test the rules using an ordinary contributor account; do not assume admin permissions demonstrate what a junior can or cannot do.

---

## 4. Junior developer: fork and clone the organisation repository

### Step 1 — Fork the repository on GitHub

1. Open [the organisation repository](https://github.com/Digital-Witch-DevOps-Team/Terraform-Infrastructure-as-code-IaC).
2. Click **Fork**.
3. Choose your personal GitHub account as the owner (example: `Ose-Jossy`).
4. Keep the repository name `Terraform-Infrastructure-as-code-IaC`, unless your team asks otherwise.
5. If you want all branches copied into the fork, leave **Copy the default branch only** unchecked. Even if `Dev-Team` is not copied into the fork, you can still fetch it from the organisation `upstream` remote in the next steps.
6. Click **Create fork**.

### Step 2 — Clone your own fork

Replace `JUNIOR-USERNAME` with your GitHub username:

```bash
git clone https://github.com/JUNIOR-USERNAME/Terraform-Infrastructure-as-code-IaC.git
cd Terraform-Infrastructure-as-code-IaC

git status
git branch --show-current
```

### Step 3 — Add the organisation repository as `upstream`

```bash
git remote add upstream https://github.com/Digital-Witch-DevOps-Team/Terraform-Infrastructure-as-code-IaC.git
git remote -v
git fetch upstream
git branch -r
```

Expected remote meaning:

| Remote | Must point to |
|---|---|
| `origin` | Your personal fork (`JUNIOR-USERNAME/Terraform-Infrastructure-as-code-IaC`) |
| `upstream` | The organisation repository (`Digital-Witch-DevOps-Team/Terraform-Infrastructure-as-code-IaC`) |

In `git branch -r`, look for **`upstream/Dev-Team`**. The list of `origin/*` branches can vary depending on whether the fork copied all branches; that is okay as long as `git fetch upstream` shows `upstream/Dev-Team`.

### Step 4 — Set your Git identity in this clone only

```bash
git config user.name "YOUR GITHUB USERNAME"
git config user.email "YOUR GITHUB NOREPLY EMAIL"
git config --local --list
```

Do not use `git config --global` on a shared/work computer unless the user explicitly intends to change the identity for every repository.

### Step 5 — Create a feature branch from the organisation's `Dev-Team`

```bash
git switch -c feature/update-terraform upstream/Dev-Team
git branch
git status
```

The current branch should be `feature/update-terraform` (marked with `*`). Use a new, task-specific branch name for each assignment, such as `feature/add-vpc-module` or `docs/update-deployment-guide`.

### Step 6 — Edit, review, commit, and push your changes

Edit only the files related to the assigned task in VS Code. Then review and stage **specific files** rather than blindly staging everything:

```bash
git status --short
git add path/to/changed-file1 path/to/changed-file2
git diff --cached
git commit -m "Add short description of change"
git push -u origin feature/update-terraform
```

Replace the example paths, branch name, and commit message with the actual task. If only one file changed, stage that one file. The `git diff --cached` command lets you inspect exactly what is staged before committing. After the first push with `-u`, future pushes from that branch can normally use `git push`.

> **Security rule — never commit secrets.** Do not add real `.env`, `atlas.credentials.env`, cloud credentials, access keys, private keys, Terraform state files, passwords, or tokens. Add sensitive local files to `.gitignore`; commit a safe `.env.example` containing placeholder values instead. If a secret is ever committed or pushed, revoke/rotate it immediately. Deleting the file in a later commit does not make the exposed secret safe.

For example, review and adapt these patterns in the repository's `.gitignore` (do not blindly replace existing project rules):

```gitignore
# Local environment and credential files
.env
.env.*
!.env.example
atlas*.env
*.pem
*.key

# Terraform local state/cache (follow your team's state-management policy)
.terraform/
*.tfstate
*.tfstate.*
```

### Step 7 — Open the junior pull request on GitHub

The junior opens the PR **on GitHub in the browser**, after pushing the feature branch. The local clone is used to make, commit, and push the work; the PR itself is created on the GitHub website.

On the new PR page, check both the repository and branch selectors:

| PR field | Correct value |
|---|---|
| **Base repository** | `Digital-Witch-DevOps-Team/Terraform-Infrastructure-as-code-IaC` |
| **Base branch (destination)** | `Dev-Team` |
| **Head repository (source)** | `JUNIOR-USERNAME/Terraform-Infrastructure-as-code-IaC` |
| **Compare branch (source branch)** | `feature/update-terraform` |

The direction must be:

**`JUNIOR-USERNAME:feature/update-terraform` → `Digital-Witch-DevOps-Team:Dev-Team`**

Add a clear title and summary, mention the task or issue, and click **Create pull request**. Do not create the PR in the opposite direction. The junior can select **Allow edits from maintainers** only when that is appropriate for the project; review the implications for repositories containing GitHub Actions workflows.

---

## 5. Senior: review and merge the junior PR into `Dev-Team`

### Review on GitHub

1. Sign in using a senior/reviewer account that has write access to the organisation repository.
2. Open **Pull requests** and select the junior's PR.
3. Review the description and click **Files changed**.
4. Check correctness, security, Terraform/configuration changes, naming, documentation, and test evidence.
5. Click **Review changes → Approve → Submit review** when the changes are acceptable.
6. Resolve requested changes and required checks, then merge only when GitHub permits it.

The author and approving reviewer should be different accounts for this team workflow. For example, `Ose-Jossy` opens the PR and `whoamiiamubuntu-ops` reviews it. The approval must be a formal **Approve** review, not only a conversation comment.

### Update the senior's local `Dev-Team` branch

Before switching branches, make sure you have no uncommitted work to lose. Then run these commands inside the senior's local clone, where `origin` points to the organisation repository:

```bash
git status
git fetch origin
git switch Dev-Team
git pull --ff-only origin Dev-Team
git log -5 --oneline
git status
```

Inspect the actual files changed by the PR. For the earlier training example only, you can run `cat junior-work.txt` if that file exists. In a real project, inspect the assigned Terraform/configuration/documentation files and run the tests or validation required by the project.

**No `git push` is needed to receive the junior's already-merged PR.** The changes were merged on GitHub; these commands fetch and update the senior's local branch.

---

## 6. Senior: promote tested work from `Dev-Team` into `main`

### Refresh the senior's local branches

```bash
git fetch origin
git switch main
git pull --ff-only origin main
git switch Dev-Team
git pull --ff-only origin Dev-Team
```

These commands update local branches. **They do not merge `Dev-Team` into `main`.** Test the integrated development branch before promoting it. You may inspect the difference using:

```bash
git diff --stat origin/main...origin/Dev-Team
```

### Create the promotion PR on GitHub

1. Open the organisation repository and click **Pull requests → New pull request**.
2. Set **Base (destination)** to `main`.
3. Set **Compare (source)** to `Dev-Team`.
4. Use a title such as `Promote tested Dev-Team changes to main`.
5. Summarise the changes and the tests performed, then click **Create pull request**.
6. Review the changed files and checks. Obtain the required approval from an eligible reviewer.
7. Click **Merge pull request** and confirm only when all required reviews/checks pass.
8. After the merge, update the senior's local `main` branch:

```bash
git switch main
git pull --ff-only origin main
```

The release direction is **organisation `Dev-Team` → organisation `main`**. Do not use `git merge Dev-Team` followed by `git push origin main` as a shortcut while the `main` branch protection is active; use the PR so reviews and branch rules are enforced.

---

## 7. If the senior needs to make changes

When branch protection is active, the senior should also work on a separate branch and submit a PR rather than committing directly to `Dev-Team` or `main`.

```bash
git fetch origin
git switch Dev-Team
git pull --ff-only origin Dev-Team
git switch -c senior/update-practice-file

# Edit the required files in VS Code, then stage only those files
git status --short
git add path/to/changed-file
git diff --cached
git commit -m "Update development configuration"
git push -u origin senior/update-practice-file
```

Then create a PR with **Base: `Dev-Team`** and **Compare: `senior/update-practice-file`**, review it, obtain approval, and merge it. Use another branch name for each new task.

---

## 8. Important rules for every team member

- **Never commit directly to `main`.** Promote work through an approved PR.
- When protection is active, send changes to `Dev-Team` through PRs as well. Avoid direct pushes to protected branches.
- Do not force-push to shared branches; do not use `--force` to work around a rejected push.
- Do not commit secrets, credentials, `.env` files, private keys, or Terraform state files.
- Before `git add`, inspect `git status --short`; before `git commit`, inspect `git diff --cached`.
- `origin` and `upstream` have different meanings in the senior and junior clones. Check with `git remote -v` before pushing.
- If `git pull --ff-only` fails, stop and inspect `git status` and branch history. Do not immediately reset or force-push.
- Use one feature branch per task and write clear commit messages and PR descriptions.

## Official references

- [GitHub Docs: Creating a pull request from a fork](https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/creating-a-pull-request-from-a-fork)
- [GitHub Docs: Fork a repository](https://docs.github.com/en/pull-requests/how-tos/work-with-forks/fork-a-repo)
- [GitHub Docs: Available rules for rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
- [GitHub Docs: Approving a pull request with required reviews](https://docs.github.com/en/pull-requests/how-tos/review-pull-requests/approving-a-pull-request-with-required-reviews)
- [GitHub Docs: Removing sensitive data from a repository](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
