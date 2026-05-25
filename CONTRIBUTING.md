# Contributing to Simply Jet repositories

This guide describes the conventions used across [Simply Jet repositories](https://github.com/Simply-Jet). It applies as the default for every repository in the organisation that does not provide its own `CONTRIBUTING.md`.

These conventions are enforced where possible by tooling (`lefthook`, `commitlint`, `biome`, `ruff`, branch protection). This document explains the rules and helps new contributors set up their environment.

## Branching

- The `main` branch is **protected**. All changes land via pull request with at least one approval and clean CI.
- The `staging` branch (where present) is **unprotected**. Direct pushes are permitted for fast iteration on staging environments.
- Feature work happens on short-lived branches.

### Branch name format

```
<type>/<kebab-case-description>
```

Allowed types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`, `perf`, `build`, `style`, `revert`.

Examples:
- `feat/passenger-search-by-tail-number`
- `fix/booking-timeout-on-stripe`
- `chore/bump-drizzle-orm`
- `docs/clarify-deploy-section`

A `lefthook` pre-push hook validates the branch name locally.

## Commits

We follow [Conventional Commits](https://www.conventionalcommits.org/). Every commit message must follow this format:

```
<type>(<optional scope>): <imperative summary>
```

Examples:
- `feat: add passenger search by tail number`
- `fix(booking): retry on Stripe timeout`
- `chore(deps): bump drizzle-orm from 0.30 to 0.31`
- `docs(readme): clarify how to run the staging container`

A `commitlint` hook enforces the format on every commit. Failed commits show which rule broke so you can amend the message.

### Why Conventional Commits

- Git history remains scannable (`git log --oneline` reads like a changelog).
- Changelogs and version bumps can be generated automatically.
- Reviewers quickly identify the nature of a change.

## Pull requests

1. Open a PR from your feature branch into `main` (or `staging` for staging-specific work).
2. Fill in the PR template completely. Reviewers rely on it.
3. Each PR requires **at least 1 approval** and **clean CI** before merging.
4. Default merge strategy is **squash**. The squash commit uses the PR title and body.
5. Force pushes to `main` are blocked. To clean up history, rebase on your feature branch before pushing.
6. The head branch is automatically deleted after merge.

### What makes a reviewable PR

- **One purpose per PR.** Mixing unrelated changes makes review harder.
- **Description matches the change.** The PR template prompts for what, why, how tested, and reviewer notes. Use them.
- **CI is green** before requesting review.
- **Screenshots or recordings** for UI changes.
- **Small enough to review in one sitting.** Aim for under 400 lines changed where possible.

### Reviewing PRs

- Review within 1 business day of being assigned.
- Be specific in feedback; cite the file and line.
- Distinguish blocking comments from nice-to-haves (use the `nit:` prefix for the latter).
- Approve only when you would be comfortable owning the change yourself.

## Code style

Each language has a tool that enforces formatting automatically:

| Language | Tool | Auto-enforced |
|---|---|---|
| TypeScript and JavaScript | [`biome`](https://biomejs.dev/) | Pre-commit hook and CI |
| Python | [`ruff`](https://docs.astral.sh/ruff/) and `ruff format` | Pre-commit hook and CI |
| Terraform | `terraform fmt` and `tflint` | Pre-commit hook and CI |

Run the relevant tool before opening a PR. The `lefthook` pre-commit hook does this automatically for staged files. CI rejects PRs with style failures.

## Local setup

After cloning a repository:

```bash
# Install dependencies (varies by repo)
bun install        # JavaScript and TypeScript repos
uv sync            # Python repos

# Install the git hooks
lefthook install
```

Most repos run `lefthook install` automatically as a post-install step. Verify by attempting a commit; if the commit-msg hook does not fire, run `lefthook install` manually.

## Dependencies

- Dependency updates are proposed automatically by Dependabot every Monday.
- Security updates are auto-PR'd as soon as a CVE is published for a vulnerable dependency.
- Manual dependency updates should go through the normal PR flow.
- Major version bumps for critical libraries (database drivers, framework cores) are blocked by `dependabot.yml` and require a manual migration plan.

## Security

For vulnerability reports, see [SECURITY.md](SECURITY.md). Do not file security issues as public GitHub issues.

## Questions

- **General engineering questions:** ask in the team Slack channel.
- **GitHub, tooling, or organisation questions:** rama-kairi or mondalraj.
- **Security questions:** it@simply-jet.ch.

## Where to find us

- Website: [simply-jet.ch](https://www.simply-jet.ch/)
- LinkedIn: [@simply-jet-sa](https://ch.linkedin.com/company/simply-jet-sa)
