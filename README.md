# delete-workflow-runs-action

[![OpenSSF Best Practices](https://www.bestpractices.dev/projects/11003/badge)](https://www.bestpractices.dev/projects/11003)
[![CI](https://github.com/tagdots/delete-workflow-runs/actions/workflows/ci.yaml/badge.svg)](https://github.com/tagdots/delete-workflow-runs/actions/workflows/ci.yaml)
[![marketplace](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/tagdots/delete-workflow-runs/refs/heads/badges/badges/marketplace.json)](https://github.com/marketplace/actions/delete-workflow-runs-action)
[![coverage](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/tagdots/delete-workflow-runs/refs/heads/badges/badges/coverage.json)](https://github.com/tagdots/delete-workflow-runs/actions/workflows/cron-tasks.yaml)

This action runs [delete-workflow-runs](https://github.com/tagdots/delete-workflow-runs) to delete GitHub Action workflows runs.

<br>

## ⭐ Why switch to delete-workflow-runs-action?
Among all the actions on the GitHub marketplace that delete workflow runs,

- we share evidence of "coverage run" tests in action (click Code Coverage badge).
- we reduce your supply chain risks with `openssf best practices` in our SDLC and operations.
- we identify orphan workflow runs that should be deleted when the parent workflow is deleted.
- we produce API rate limit consumption estimate in dry-run, so you can plan your delete task properly.

<br>

## 😎 GitHub Action workflow examples

Use the workflow examples below to create your own workflow inside `.github/workflows/`.

### Example 1 - summary
**delete-workflow-runs-action**:

* runs on a scheduled interval - every day at 5:30 pm UTC  (`- cron: '30 17 * * *'`)
* uses GitHub Token with permissions: `actions: read` and `contents: read`
* performs a **MOCK delete** and provides an estimate of API rate limit consumption (`dry-run: false`)
* keeps the last 10 workflow runs for each workflow and delete the rest (`min-runs: 10`)

### Example 1 - workflow
```
name: delete-workflow-runs

on:
  schedule:
    - cron: '30 17 * * *'

permissions:
  actions: read
  contents: read

jobs:
  delete-workflow-runs:
    runs-on: ubuntu-latest

    permissions:
      actions: read
      contents: read

    - name: Run stale-workflow-runs
      id: stale-workflow-runs
      uses: tagdots/delete-workflow-runs-action@xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx # 1.0.0
      env:
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        repo-url: ${{ github.repository }}
        min-runs: 10
        dry-run: true
```

<br>

### Example 2 - summary
**delete-workflow-runs-action**:

* runs on a scheduled interval - every day at 5:30 pm UTC  (`- cron: '30 17 * * *'`)
* uses GitHub Token with permissions: `actions: write` and `contents: read`
* performs a REAL delete (`dry-run: false`)
* keep the workflow runs in the last 10 days for each workflow and delete the rest (`max-days: 10`)

### Example 2 - workflow
```
name: delete-workflow-runs

on:
  schedule:
    - cron: '30 17 * * *'

permissions:
  actions: read
  contents: read

jobs:
  delete-workflow-runs:
    runs-on: ubuntu-latest

    permissions:
      actions: write
      contents: read

    - id: delete-workflow-runs
      uses: tagdots/delete-workflow-runs-action@main
      env:
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        repo-url: ${{ github.repository }}
        max-days: 10
        dry-run: false
```

<br>

### Example 3 - summary
**delete-workflow-runs-action**:

* runs on a scheduled interval - every day at 5:30 pm UTC  (`- cron: '30 17 * * *'`)
* uses GitHub Token with permissions: `actions: read` and `contents: read`
* performs a REAL delete (`dry-run: true`)
* keeps the workflow runs in the last 10 days for each workflow and delete the rest (`max-days: 10`)
* gets data from workflow run to feed data for another steps/jobs e.g. notification or delete action

### Example 3 - workflow
```
name: delete-workflow-runs

on:
  schedule:
    - cron: '30 17 * * *'

permissions:
  actions: read
  contents: read

jobs:
  delete-workflow-runs:
    runs-on: ubuntu-latest

    permissions:
      actions: read
      contents: read

    - id: delete-workflow-runs
      uses: tagdots/delete-workflow-runs-action@main
      env:
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        repo-url: ${{ github.repository }}
        max-days: 10
        dry-run: true

    # use GITHUB_OUTPUT for integration with other tools
    # e.g.
    # 1. notifications if orphan_ct is not 0 or active_ct is over N threshold
    # 2. use the last step for dry-run, and build another delete-workflow-runs to trigger real delete
    # ....etc.
    - name: Get data and set GITHUB_OUTPUT
      id: get_data
      shell: bash
      run: |
        # get data from the last step to set GITHUB_OUTPUT
        cat data_dict.log | jq .
        echo "dry_run=$(cat data_dict.log | jq '."dry-run"')" >> $GITHUB_OUTPUT
        echo "repo_url=$(cat data_dict.log | jq '."repo-url"')" >> $GITHUB_OUTPUT
        echo "max_days=$(cat data_dict.log | jq '."max-days"')" >> $GITHUB_OUTPUT
        echo "min_runs=$(cat data_dict.log | jq '."min-runs"')" >> $GITHUB_OUTPUT
        echo "limit_usage_est=$(cat data_dict.log | jq '."core-limit-usage-estimate"')" >> $GITHUB_OUTPUT
        echo "limit_rem=$(cat data_dict.log | jq '."core-limit-remaining"')" >> $GITHUB_OUTPUT
        echo "limit_res=$(cat data_dict.log | jq '."core-limit-reset"')" >> $GITHUB_OUTPUT
        echo "active_ct=$(cat data_dict.log | jq '."delete-active-workflow-runs-count"')" >> $GITHUB_OUTPUT
        echo "orphan_ct=$(cat data_dict.log | jq '."delete-orphan-workflow-runs-count"')" >> $GITHUB_OUTPUT
```

<br>

## 🔧 delete-workflow-runs command line options

| Input | Description | Default | Required | Notes |
|-------|-------------|----------|----------|----------|
| `repo-url` | Repository URL | `None` | Yes | e.g. https://github.com/{owner}/{repo} |
| `dry-run` | Dry-Run | `True` | No | - |
| `min-runs` | Min. no. of runs to <br>keep in a workflow | `None` | No | enter either min. runs or max. days |
| `max-days` | Max. no. of days to <br>keep run in a workflow | `None` | No | enter either min. runs or max. days |

<br>

## ⚠️ Summary of GitHub rate limit for standard repository
```
* 1,000 requests per hour per repository.
* No more than 100 concurrent requests are allowed.
* No more than 900 points per minute are allowed for REST API endpoints.
* No more than 90 seconds of CPU time per 60 seconds of real time is allowed.
* Make too many requests that consume excessive compute resources in a short period of time.
```

<br>

## 😕  Troubleshooting

We are here to help - open an [issue](https://github.com/tagdots/delete-workflow-runs-action/issues)

<br>

## 🙌 Appreciation
If you find this project helpful, please ⭐ star it.  **Thank you**.

<br>

## 📖 License

[MIT License](https://github.com/tagdots/delete-workflow-runs-action/blob/main/LICENSE).

<br>

## 📚 References

[GitHub API rate limit](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api?apiVersion=2022-11-28#primary-rate-limit-for-github_token-in-github-actions)
