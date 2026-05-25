# Incident Response

## Who to contact

| Scenario | Primary | Fallback |
|---|---|---|
| Production outage or critical bug | `@rama-kairi` (Slack DM) | `@mondalraj` |
| Security vulnerability | `it@simply-jet.ch` | `@rama-kairi` |
| Repository or GitHub-org issue | `@rama-kairi` | `@mondalraj` |
| Engineering question | Team Slack channel | n/a |

## Out-of-hours contact

For genuine production incidents outside business hours, page on Slack DM first to rama-kairi or mondalraj. Both are in the `admins` and `devops` teams and can authorise emergency merges.

If both are unreachable for more than 30 minutes, escalate to **Nail Senocak** (Senior Project Manager) via Slack DM.

## Hotfix lifecycle

For genuine production emergencies that need to bypass the normal PR-with-review flow:

1. Open a hotfix branch from `main`
2. Make the minimal fix and push
3. Open a PR
4. As an org admin, self-approve the PR using OrganizationAdmin bypass mode (allowed under our `main-branch-protection` ruleset for emergencies)
5. Merge, deploy, monitor
6. Document the bypass in this file's "Emergency log" within 24 hours

## Force push to `main` (genuine emergencies only)

If history must be rewritten (committed secret, large binary, GDPR compliance), use the documented disable-ruleset workflow:

```bash
# 1. Disable the ruleset
gh api -X PUT orgs/Simply-Jet/rulesets/16833359 -F enforcement=disabled

# 2. Perform the force push
git push --force-with-lease origin main

# 3. Re-enable the ruleset
gh api -X PUT orgs/Simply-Jet/rulesets/16833359 -F enforcement=active
```

Log the action in "Emergency log" below. GitHub's audit log will record both ruleset state changes independently.

## Emergency log

When the normal PR flow is bypassed or the ruleset is temporarily disabled, log the entry here.

| Date | Who | What | Why |
|---|---|---|---|
| 2026-05-25 | rama-kairi | Disabled ruleset briefly to push files to `Simply-Jet/.github` | Bootstrap of org profile and community health files |
