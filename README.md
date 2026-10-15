# shared-workflows

Reusable GitHub Actions workflows for ganbitlabs repos. Public on purpose: public repos can only call reusable workflows that live in public repos. Nothing here holds secrets; callers pass them in.

## elek PR review

[elek](https://github.com/selimozten/elek) reviews every pull request with pi on `opencode-go/deepseek-v4.1-flash`. It posts a summary and inline findings, and it cannot approve, merge, or close.

Add `.github/workflows/elek.yml` to a repo:

```yaml
name: elek
on:
  pull_request: { types: [opened, synchronize, reopened, ready_for_review] }

permissions:
  contents: read
  pull-requests: write
  issues: write

concurrency:
  group: elek-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

jobs:
  elek:
    uses: ganbitlabs/shared-workflows/.github/workflows/elek.yml@v1
    secrets:
      OPENCODE_API_KEY: ${{ secrets.OPENCODE_API_KEY }}
```

Then make `OPENCODE_API_KEY` available to the repo, as an org secret or a repo secret. Pass only that secret; avoid `secrets: inherit`, which hands the workflow every secret the repo can see. Optional `with:` inputs: `provider`, `model`, `thinking`, `severity_threshold` (default `important`). Repo-specific review policy goes in the calling repo's `.elek.yml`.

elek reviews when a PR is opened, reopened, or marked ready for review, and on every push to it. Draft PRs are skipped until they are marked ready. There is no comment trigger. Only `pull_request` events from branches in the same repo are reviewed. Fork PRs are skipped, since GitHub withholds secrets from them, so outside contributors cannot spend the API key.

## Requirements

The workflow pins elek v1.3.0 and `actions/checkout` by commit SHA. Bump those pins here, then cut a new tag.

## Versioning

Callers pin `@v1` (floating) or an immutable `@v1.0.x` tag or commit SHA for reproducibility. Change workflows on `main`, then move the tag once a change is verified: tag `v1.0.x` on it, then move `v1` (`git tag -fa v1 -m ... && git push -f origin v1`). Breaking input changes get `v2`.
