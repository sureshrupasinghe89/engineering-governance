# FOSSA SCA Scan Action

Run FOSSA scans from GitHub Actions with support for:

- `fossa analyze`
- optional `fossa test`
- optional diff-based test gating for pull requests
- optional attribution report generation
- optional debug mode
- optional high/critical vulnerability gating through the FOSSA issues API

## Prerequisites

- A FOSSA API key stored as a GitHub secret such as `FOSSA_API_KEY`
- A Linux or macOS runner with network access to your FOSSA endpoint
- For the optional vulnerability gate, a FOSSA project scope convention that can be resolved from `scope-id-prefix`, `github-org`, and `repo-name`

## Usage

Use a pinned release tag or commit SHA when consuming this action from another repository.

### Basic scan

```yaml
jobs:
  fossa-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: wso2/engineering-governance/.github/security/fossa-sca-scan@main
        with:
          api-key: ${{ secrets.FOSSA_API_KEY }}
```

### Scan and test

```yaml
jobs:
  fossa-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: wso2/engineering-governance/.github/security/fossa-sca-scan@main
        with:
          api-key: ${{ secrets.FOSSA_API_KEY }}
          run-tests: "true"
```

### Scan a specific folder in a repository

```yaml
jobs:
  fossa-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: wso2/engineering-governance/.github/security/fossa-sca-scan@main
        with:
          api-key: ${{ secrets.FOSSA_API_KEY }}
          working-directory: apps/web
          run-tests: "true"
```

### Pull request diff gate

```yaml
jobs:
  fossa-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: wso2/engineering-governance/.github/security/fossa-sca-scan@main
        with:
          api-key: ${{ secrets.FOSSA_API_KEY }}
          run-tests: ${{ github.event_name == 'pull_request' }}
          test-diff-revision: ${{ github.event.pull_request.base.sha }}
```

### Generate an attribution report

```yaml
jobs:
  fossa-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - id: fossa
        uses: wso2/engineering-governance/.github/security/fossa-sca-scan@main
        with:
          api-key: ${{ secrets.FOSSA_API_KEY }}
          generate-report: html

      - run: echo '${{ steps.fossa.outputs.report }}' > report.html
```

### Enable the vulnerability API gate

```yaml
jobs:
  fossa-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: wso2/engineering-governance/.github/security/fossa-sca-scan@main
        with:
          api-key: ${{ secrets.FOSSA_API_KEY }}
          run-tests: "true"
          fail-on-vulnerabilities: "true"
          scope-id-prefix: custom+28356/github.com
```

## Inputs

- `api-key`: Required FOSSA API key.
- `run-tests`: Optional. Runs `fossa test` after analysis when set to `"true"`. Default: `"false"`.
- `generate-report`: Optional. Runs `fossa report attribution --format <value>` and exposes the report content as an output.
- `test-diff-revision`: Optional. Passed to `fossa test --diff`. Effective only when `run-tests` is enabled.
- `branch`: Optional. Branch passed to FOSSA CLI. Defaults to the current GitHub ref.
- `project`: Optional. Project name passed to FOSSA CLI.
- `endpoint`: Optional. FOSSA endpoint URL. Default: `https://app.fossa.com`.
- `debug`: Optional. Runs FOSSA commands in debug mode when set to `"true"`. Default: `"false"`.
- `working-directory`: Optional. Directory to scan. Default: `.`.
- `fail-on-vulnerabilities`: Optional. Enables the extra FOSSA issues API gate for active `high` and `critical` vulnerabilities. Default: `"false"`.
- `repo-name`: Optional. Repository name used by the vulnerability API gate. Defaults to the current repository name.
- `github-org`: Optional. Repository owner used by the vulnerability API gate. Defaults to the current repository owner.
- `scope-id-prefix`: Optional. FOSSA scope prefix used by the vulnerability API gate. Required for that gate to run.

## Outputs

- `revision`: Revision extracted from `fossa analyze` output. Falls back to `GITHUB_SHA` if the revision cannot be parsed.
- `active-count`: Active high/critical vulnerability count returned by the FOSSA issues API when the vulnerability gate is enabled.
- `report`: Generated attribution report content when `generate-report` is used.

## Notes

- The vulnerability gate is a WSO2-specific extension on top of the standard FOSSA analyze/test flow.
- The action currently installs a repo-managed FOSSA CLI version: `v3.17.10`. Update the action itself when you want to roll forward the CLI version across consumers.
- The examples above still use `@main` for readability. For production use, pin to a release tag or commit SHA.
