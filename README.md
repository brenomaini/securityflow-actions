# SecurityFlow scan

Runs open-source scanners in your own CI and sends the raw results to your SecurityFlow instance:

- **SAST** (OpenGrep)
- **SCA and IaC** (Trivy)
- **Secrets** (Gitleaks, including git history)

Findings are deduplicated, scored and routed in SecurityFlow like any other. Nothing is scanned on SecurityFlow's side.

## Usage

```yaml
name: SecurityFlow
on:
  push:
    branches: [main]
  schedule:
    - cron: "17 3 * * 1"

jobs:
  securityflow:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0   # full history, so secrets removed from the tree are still found
      - uses: brenomaini/securityflow-actions@v1
        with:
          securityflow-url: ${{ vars.SF_URL }}
          api-key: ${{ secrets.SF_API_KEY }}
```

Create the key in SecurityFlow under **Settings → Ingestion API Keys** (type *SARIF pipeline*), then add it as the repository secret `SF_API_KEY` and your instance URL as the variable `SF_URL`.

## Inputs

| Input | Required | Default | |
|---|---|---|---|
| `securityflow-url` | yes | | Base URL of your SecurityFlow instance |
| `api-key` | yes | | Ingestion API key (`ingest:sarif` scope) |
| `categories` | no | `sast,sca,secret` | Scans to run |
| `scan-mode` | no | `full` | `full` closes findings that disappeared (default branch only); `partial` only adds |
| `default-branch` | no | the repository's default branch | Needed by SecurityFlow the first time a repository is ingested |
| `wait` | no | `true` | Fail the step if SecurityFlow fails to process the results |

Pull request runs are accepted and recorded but do not change the official findings.

## Requirements

- Linux runners (this is a container action).
- Outbound access to your SecurityFlow URL and to `ghcr.io`. Trivy downloads its vulnerability database on each run.

## License

Apache-2.0
