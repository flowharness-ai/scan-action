# FlowHarness Scan Action

Run the published FlowHarness static scanner in a GitHub Actions workflow with
least-privilege repository access.

```yaml
name: FlowHarness Scan
on:
  pull_request:
permissions:
  contents: read
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: flowharness-ai/scan-action@v1
        with:
          directory: "."
          fail-on: "risk"
```

## Inputs

| Input | Required | Default | Meaning |
| --- | --- | --- | --- |
| `directory` | no | `.` | repository-relative directory to scan |
| `fail-on` | no | `risk` | `risk` or `index>N` gate expression |

## Outputs

| Output | Meaning |
| --- | --- |
| `exit-code` | raw scanner code: 0 CLEAR, 1 NEEDS_HUMAN, 2 QUARANTINE |

The action writes the scan summary and then propagates the scanner's raw status.
Exit 1 may also mean a scanner error, and exit 2 may also mean an argparse usage
error (a usage error from argument parsing). If the status handoff is absent,
the action fails closed with exit 2.

The scanner itself is zero-model and zero-network. Installing `uv` and resolving
the published package from PyPI require network access.

Learn more at [flowharness.ai](https://flowharness.ai/).

## Source and releases

The canonical source is
[`suleimanmahmoud/flowharness/.github/actions/scan/action.yml`](https://github.com/suleimanmahmoud/flowharness/blob/main/.github/actions/scan/action.yml).
This repository is a one-way export: change the monorepo source first, re-export
it byte-for-byte, and then move the `v1` tag after review.

Licensed under the Apache License 2.0. See [LICENSE](LICENSE).
