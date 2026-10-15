# shared-workflows

Reusable GitHub Actions workflows for ganbitlabs repos. Public on purpose: public repos can only call reusable workflows that live in public repos. Nothing here holds secrets; callers pass them in.

## elek PR review

[elek](https://github.com/selimozten/elek) reviews every pull request with pi on `opencode-go/deepseek-v4.1-flash`. It posts a summary and inline findings, and it cannot approve, merge, or close.

Add `.github/workflows/elek.yml` to a repo:

```yaml
name: elek
on:
  pull_request: { types: [opened, synchronize, reopened] }
  issue_comment: { types: [created] }

permissions:
  contents: read
  pull-requests: write
  issues: write

concurrency:
  group: elek-${{ github.event.pull_request.number || github.event.issue.number || github.ref }}
  cancel-in-progress: true

jobs:
  elek:
    uses: ganbitlabs/shared-workflows/.github/workflows/elek.yml@main
    secrets: inherit
```

Then make `OPENCODE_API_KEY` available to the repo, as an org secret or a repo secret. Optional `with:` inputs: `provider`, `model`, `thinking`, `review_strategy` (default `crosscheck`), `severity_threshold` (default `important`). Repo-specific review policy goes in the calling repo's `.elek.yml`.

Comment `@pi <request>` on a PR to ask elek for a targeted re-review.

## Requirements

`opencode_api_key` support in elek (selimozten/elek, `feat/opencode-provider`). Until that ships in `v1`, the review step fails to authenticate.
