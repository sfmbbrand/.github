# .github

Organization-wide GitHub Actions workflows and community health files for [sfmbbrand](https://github.com/sfmbbrand).

## Reusable Workflows

### PR Review (`pr-review.yml`)

Automated code review using Claude. After CI passes on a PR, Claude reviews the changes and either approves + merges or requests changes.

**Used by:** `erosguia_br`, `erosguia_br_admin_ng`, `eg-int-private-area-nextjs`, `eg-int-private-backend`, `eg_admin_backend`, `eg-int-privada-php`

**Not used by:** `eg-int-common` (manual review only)

#### How it works

1. A PR is opened and CI passes
2. Each repo's `claude-review.yml` triggers `pr-review.yml` via `workflow_run`
3. Claude reviews the diff for blocking issues (security, bugs, data loss)
4. If clean → approve + squash merge. If issues → request changes with details.

#### Required secrets (org-level)

| Secret | Description |
|--------|-------------|
| `ANTHROPIC_API_KEY` | Anthropic API key for Claude |
| `GH_REVIEW_TOKEN` | Fine-grained PAT for PR approval and merge |

#### Caller workflow

Each repo needs `.github/workflows/claude-review.yml`:

```yaml
name: Claude PR Review

on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]

jobs:
  review:
    if: >
      github.event.workflow_run.conclusion == 'success' &&
      github.event.workflow_run.event == 'pull_request'
    uses: sfmbbrand/.github/.github/workflows/pr-review.yml@main
    secrets: inherit
```
