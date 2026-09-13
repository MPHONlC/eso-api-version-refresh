# ESO Addon API Version Refresh

Checks the [esoui/esoui](https://github.com/esoui/esoui) repo's own live README - which states the Elder Scrolls Online client's current live API version in plain text - and bumps your addon manifest's `## APIVersion:` pair (`current`, `current+1`) if it's fallen behind, committing and pushing the change.

The "current+1" convention is a common ESO addon-dev practice: declaring the next API version alongside the current one is a preemptive "probably still works on the next patch too" statement. A stale-but-lower declared `APIVersion` shows as "Out of Date" in the in-game Add-On Manager but doesn't hard-block loading, so pre-declaring compatibility ahead of manual testing is a widely accepted, low-risk convention - though you should still verify your addon actually works after each real ESO update.

## Usage

Your workflow must check out the repo first, with `permissions: contents: write` on the job (this action commits and pushes).

```yaml
name: API Version Refresh

on:
  schedule:
    - cron: '0 12 * * *'   # once a day
  workflow_dispatch:
  workflow_call:

jobs:
  refresh-api-version:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v7

      - uses: MPHONlC/eso-api-version-refresh@Version-0.0.1
        with:
          manifest_file: 'MyAddon.addon'
          git_name: 'YourGitName'
          git_email: 'your-email@example.com'
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `manifest_file` | Yes | - | Path to your addon's `.addon` manifest file. |
| `git_name` | Yes | - | Git name to attribute the refresh commit to. |
| `git_email` | Yes | - | Git email to attribute the refresh commit to. |

## Outputs

| Output | Description |
|---|---|
| `result` | One of `updated`, `up_to_date`, or `extraction_failed`. |
| `old_api` | The declared `APIVersion` pair before this run. |
| `new_api` | The declared `APIVersion` pair after this run (only set when `result` is `updated`). |

## Requirements

- Your manifest's `## APIVersion:` line must already declare exactly two numbers (`current` and `current+1`) - this action only ever rewrites both together.
- The calling job needs `permissions: contents: write`, since this action pushes directly.
- No secrets required - the source it reads (`raw.githubusercontent.com/esoui/esoui`) is public.

> [!IMPORTANT]
> Without `permissions: contents: write` on the job, the commit/push step fails - this is the most common setup mistake with this action.

> [!WARNING]
> The `result: extraction_failed` output means the regex that scrapes `esoui/esoui`'s own README for its current live API version didn't match anything - most likely because esoui/esoui changed the wording or formatting of that line. This doesn't fail the job or touch your manifest, but it does mean the check silently did nothing that run; worth watching for if esoui/esoui ever restructures their README.

## License

MIT - see [LICENSE](LICENSE).
