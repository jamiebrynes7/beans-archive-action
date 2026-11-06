# beans-archive-action

A composite GitHub Action that moves completed and scrapped [beans](https://github.com/hmans/beans)
into `.beans/archive/` through a pull request. If there is nothing to archive, it opens no PR.

## Usage

```yaml
name: Archive completed beans

on:
  schedule:
    - cron: "0 5 * * 1"
  workflow_dispatch:

permissions:
  contents: read

jobs:
  archive-beans:
    runs-on: ubuntu-latest
    steps:
      - id: app-token
        uses: actions/create-github-app-token@v3
        with:
          client-id: ${{ vars.APP_CLIENT_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}
      - uses: jamiebrynes7/beans-archive-action@<full-commit-sha>
        with:
          token: ${{ steps.app-token.outputs.token }}
```

## Inputs

| Input          | Required | Default  | Meaning                                                            |
| -------------- | -------- | -------- | ------------------------------------------------------------------ |
| `token`        | yes      |          | Token used for checkout, the PR and the merge. See below.          |
| `auto-merge`   | no       | `true`   | Enable auto-merge on the PR. When `false`, the PR stays open.      |
| `merge-method` | no       | `rebase` | One of `merge`, `squash` or `rebase`. Used only when auto-merging. |

## Token requirement

`token` needs write access to contents and pull requests, and must not be `GITHUB_TOKEN`. PRs opened
with `GITHUB_TOKEN` do not trigger workflows, so required checks never report and the PR never
merges. The action fails if it is given `GITHUB_TOKEN`. A GitHub App token or a personal access token
both work. All writes go through `token`, so the job itself only needs `contents: read`.

## Behaviour

- The action always archives the default branch, even when run manually from another branch.
- The action commits to a fixed `beans/archive` branch. If a PR is already open, for example because
  a required check failed, the next run updates it.
- If a PR can't merge straight away and the repo has auto-merge turned off or disallows
  `merge-method`, the merge step fails and the PR stays open. Set `auto-merge: false` or change
  `merge-method`.
- The repo must keep its beans in `.beans`.

## Beans version

The action builds beans at a fixed commit of
[`jamiebrynes7/dotfiles`](https://github.com/jamiebrynes7/dotfiles). Moving to a newer beans means
bumping that commit in `action.yml`.

## License

[MIT](LICENSE)
