# Git Workflow

How code moves from an issue to `main` in this repo. It applies to every change — features, fixes, chores, docs.

## Branches

| Branch | Purpose | Rules |
| --- | --- | --- |
| `main` | Releases | Protected. Never commit or branch feature work here. Changes arrive only by PR from `develop`. |
| `develop` | Integration (default branch) | Never commit directly. All work arrives by PR from a feature branch. |
| `<type>/<issue>-<description>` | Your work | Branched off the latest `develop`. Short-lived: one issue, one branch, one PR. |

## 1. Pick up an issue

- Make sure the work has an issue and it's assigned to you.
- Move its card to **In Progress** on the project board.

## 2. Branch off `develop`

```bash
git checkout develop
git pull origin develop
git checkout -b feat/12-implement-login-page
```

Branch names follow `<type>/<issue-number>-<short-kebab-description>`:

| Prefix | Use for | Example |
| --- | --- | --- |
| `feat/` | New functionality | `feat/12-implement-login-page` |
| `bug/` | Fixing broken behaviour | `bug/27-header-contrast-dark-mode` |
| `chore/` | Tooling, deps, config, housekeeping | `chore/31-upgrade-eslint` |
| `docs/` | Documentation only | `docs/9-readme-setup-steps` |
| `refactor/` | Behaviour-preserving restructure | `refactor/18-extract-match-service` |

The issue number is required — it ties the branch, PR, and board card together.

## 3. Commit incrementally, in Conventional Commits format

Commit early and often while you work — small checkpoints are encouraged. Every message follows the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) spec:

```
<type>(optional scope): <imperative, lower-case subject>

optional body: the why, not the what
```

Examples:

```
feat(auth): add login form validation
bug(header): fix contrast of nav links in dark mode
chore: pin node version in engines
```

Types match the branch prefixes above, plus `test`, `style`, `perf`, `ci` where they fit. A breaking change adds `!` after the type and a `BREAKING CHANGE:` footer.

## 4. Stay current by rebasing — never merge `develop` in

If `develop` moves while you work:

```bash
git fetch origin
git rebase origin/develop
# resolve conflicts, then:
git push --force-with-lease
```

- Rebase your own feature branch only. Never rebase `develop` or `main`.
- Always `--force-with-lease`, never `--force` — it refuses to overwrite work you haven't seen.

## 5. Squash to a single commit before review

Work in as many checkpoint commits as you like, but the PR is reviewed and merged as **one commit** that tells one story. Before marking the PR ready:

```bash
git rebase -i origin/develop   # mark the first commit `pick`, the rest `squash`
git push --force-with-lease
```

(Equivalent shortcut: `git reset --soft $(git merge-base HEAD origin/develop) && git commit`.)

The final commit message must be a Conventional Commit whose subject describes the whole change, e.g. `feat(auth): implement login page (#12)`.

## 6. Open the PR

- Base branch: `develop`. Head: your feature branch.
- Fill in the PR template — every section.
- Link the issue in the description: `Closes #12`.
- Address review feedback in new commits so reviewers can see what changed, then squash again to one commit before merge.
- Move the board card as the PR progresses (**Review in progress** → **Reviewer approved** → **Done**).

## 7. Releases

`develop` → `main` by PR only. `main` requires an approving review and blocks force-pushes; what lands there is what's deployed.

## Quick reference

```bash
git checkout develop && git pull origin develop      # start fresh
git checkout -b feat/<issue>-<description>           # branch
# ...work, committing incrementally (conventional commits)...
git fetch origin && git rebase origin/develop        # stay current
git rebase -i origin/develop                         # squash to one commit
git push --force-with-lease                          # publish
# open PR -> develop, "Closes #<issue>", template filled
```

## Monorepo note

This repo holds both apps: work in `frontend/` or `backend/` per your issue's area label. Branch naming stays the same; use the **commit scope** to convey the area — `feat(frontend): …`, `bug(backend): …` — and keep each PR to a single area whenever possible.
