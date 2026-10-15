# shared-workflows

Reusable GitHub Actions workflows for ganbitlabs repos. Public on purpose: public repos can only call reusable workflows that live in public repos. Nothing here holds secrets; callers pass them in.

## elek PR review

[elek](https://github.com/selimozten/elek) reviews every pull request with pi on `opencode-go/deepseek-v4.1-flash`. It posts a summary and inline findings, and it cannot approve, merge, or close.

Add `.github/workflows/elek.yml` to a repo:

```yaml
name: elek
on:
  pull_request: { types: [opened, synchronize, reopened] }

permissions:
  contents: read
  pull-requests: write
  issues: write

concurrency:
  group: elek-${{ github.event.pull_request.number || github.event.issue.number || github.ref }}
  cancel-in-progress: true

jobs:
  elek:
    uses: ganbitlabs/shared-workflows/.github/workflows/elek.yml@v1
    secrets:
      OPENCODE_API_KEY: ${{ secrets.OPENCODE_API_KEY }}
```

Then make `OPENCODE_API_KEY` available to the repo, as an org secret or a repo secret. Pass only that secret; avoid `secrets: inherit`, which hands the workflow every secret the repo can see. Optional `with:` inputs: `provider`, `model`, `thinking`, `review_strategy` (default `crosscheck`), `severity_threshold` (default `important`), `pr_prompt` (the review request sent on PR events). Repo-specific review policy goes in the calling repo's `.elek.yml`.

elek reviews when a PR is opened or reopened and on every push to it. There is no comment trigger. PRs from forks run without secrets, as GitHub withholds them, so outside contributors cannot spend the API key.

## Requirements

elek v1.2.3 or later (`opencode_api_key` input). The workflow uses `selimozten/elek@v1`.

## Versioning

Callers pin `@v1` (floating) or an immutable `@v1.0.x` tag or commit SHA for reproducibility. Change workflows on `main`, then move the tag once a change is verified: tag `v1.0.x` on it, then move `v1` (`git tag -fa v1 -m ... && git push -f origin v1`). Breaking input changes get `v2`.
