# tfe-drift-action

[![CI](https://github.com/slok/tfe-drift-action/actions/workflows/ci.yaml/badge.svg?branch=main)](https://github.com/slok/tfe-drift-action/actions/workflows/ci.yaml)
[![Apache 2 licensed](https://img.shields.io/badge/license-Apache2-blue.svg)](https://raw.githubusercontent.com/slok/tfe-drift-action/master/LICENSE)
[![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/slok/tfe-drift-action)](https://github.com/slok/tfe-drift-action/releases/latest)

## Introduction

[tfe-drift] Github action.

## Getting started

A very simple and example, execute every hour drift detection with a limit of plans:

```yaml
name: drift-detection

on:
  schedule:
    - cron:  '0 * * * *' # Every hour.

jobs:
  drift-detection:
    runs-on: ubuntu-latest
    steps:
      - uses: slok/tfe-drift-action@v0.2.0
        id: tfe-drift
        with:
          tfe-token: ${{ secrets.TFE_TOKEN }}
          tfe-org: slok
          limit-max-plans: 3
```

## Input variables

- `tfe-token`: (Required) The Terraform API token.
- `tfe-org`: (Required) The Terraform cloud/enterprise organization to execute the drift detections.
- `not-before`: A time duration (e.g `1h`, `30m`) that will ignore the workspace if in the last specified duration, a drift detections has been already executed
- `limit-max-plans`: An integer that will limit the number of drift detections executed per run.
- `dry-run`: Will not execute any plan.
- `no-error`: Boolean that when enabled, will make the github action not fail when there is any drift or a any drift detection plan fails (used when you want more control using the action output result).
- `debug`: Boolean that will enable debug mode.
- `include-names`: String of comma separated regexes, that will exclude the workspaces that don't match these (e.g: dns,kubernetes-*,production-*).
- `exclude-names`: String of comma separated regexes, that will exclude the workspaces that match these (e.g: dns,kubernetes-*,production-*).
- `include-tags`: String of comma separated tags, that will exclude the workspaces that don't match these tags (e.g: production,infrastructure).
- `exclude-tags`: String of comma separated tags, that will exclude the workspaces that match these (e.g: production,infrastructure).

## Output variables

- `result`: JSON output with a result.

## Drift detection summary

When the action is executed, at the end it will set a drift detection summary with the link to all the plans:

![Drift detection result job summary](docs/img/job-summary.png)

## Usage

Using the JSON result to have more fine grain control.

```yaml
name: drift-detection

on:
  schedule:
    - cron:  '0 3 * * *' # Every day at 3am.

jobs:
  drift-detection:
    runs-on: ubuntu-latest
    steps:
      - uses: slok/tfe-drift-action@v0.2.0
        id: tfe-drift
        with:
          tfe-token: ${{ secrets.TFE_TOKEN }}
          tfe-org: slok
          no-error: true

      - if: fromJSON(steps.tfe-drift.outputs.result).drift
        run: |
          # Do whatever...

          echo "Drift detected"
          exit 1

      - if: fromJSON(steps.tfe-drift.outputs.result).drift_detection_plan_error
        run: |
          # Do whatever...

          echo "Drift detection plan failed"
          exit 1
```

[tfe-drift]: https://github.com/slok/tfe-drift
