# FlowHarness Scan Action

Run FlowHarness Scan in a GitHub Actions workflow to inspect a repository's
agent-context surfaces. Scan is deterministic, read-only, zero-model, and
account-free. It emits GitHub annotations, adds a verdict to the job summary,
and propagates the scanner's exit code.

The agent governance and evaluation control plane for organizations.

**Available today:** Scan and Vibe Check. Local tools and GitHub Actions are
free and Apache-2.0; the organization-wide control plane is commercial.

## Add Scan to a pull request

Save this as `.github/workflows/flowharness-scan.yml`:

```yaml
name: FlowHarness Scan
"on":
  pull_request:
permissions:
  contents: read
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: flowharness-ai/scan-action@fac3d8f7f5b176ccc1deaa0d43de0937b042b464 # v1.1.0
        with:
          directory: "."
          fail-on: "risk"
```

The workflow uses the `pull_request` trigger and exactly `contents: read`; it
does not need a FlowHarness secret. This works for fork pull requests with the
same read-only posture. Do not replace the trigger with `pull_request_target`
to obtain secrets or broader permissions while handling untrusted pull-request
content.

The Scan Action is pinned above to an immutable commit SHA. Treat the
`actions/checkout` reference the same way in security-sensitive workflows:
pin it to a reviewed immutable SHA through your dependency-update process, then
review and deliberately update both pins for releases. The `v1` tag remains the
supported major-version line for users who deliberately accept a moving tag.

## Inputs

| Input | Required | Default | Meaning |
| --- | --- | --- | --- |
| `directory` | no | `.` | Repository-relative directory to scan. |
| `fail-on` | no | `risk` | Gate expression: `risk` or `index>N`. |

## Output and verdicts

| Output | Meaning |
| --- | --- |
| `exit-code` | Raw scanner exit code. |

With the default `fail-on: risk` gate, `0` means `CLEAR`, `1` means
`NEEDS_HUMAN`, and `2` means `QUARANTINE`. A `NEEDS_HUMAN` result requires
review; do not auto-approve it. `1` can also mean a scanner or local-output
error, and `2` can also mean command-usage error. Inspect the log and report
before classifying an exit code as a verdict. If the action cannot hand off a
status, it fails closed with `2`.

Use `fail-on: "index>N"` when an index threshold, rather than the default risk
gate, is the desired policy. See the public [Scan guide](https://github.com/flowharness-ai/flowharness/blob/main/docs/scan.md)
for output formats, baselines, SARIF, and the full exit-code behavior.

## Network, source, and licensing

The scanner itself makes no network calls once its released package is
available. Workflow setup can fetch declared Actions, and `uvx` can contact
PyPI and package hosts to acquire the released `flowharness` package and its
dependencies before a scan starts. The action invokes `uvx` with
`--no-config --no-sources` so package resolution is isolated from persistent
user or system uv configuration and configured sources.

This public repository contains the browsable composite Action source and is
the reviewed release and export surface for the Scan composite Action.
Upstream/internal development may export reviewed Action bytes here one-way
for public release; this repository is not the canonical source for the
commercial platform or Python package. Public users should rely on this
repository's reviewed tag and immutable commit (including the exact Action pin
above), plus the PyPI source distribution (sdist) documented in the public
[source and provenance documentation](https://github.com/flowharness-ai/flowharness/blob/main/docs/source-and-provenance.md),
to evaluate and use the released Action. They do not need access to non-public
development sources. The local tool and this GitHub Action are Apache-2.0.

## Troubleshooting

- If package acquisition fails, distinguish that setup traffic from the
  scanner's default network posture, then retry with the published package
  version shown in the [Scan guide](https://github.com/flowharness-ai/flowharness/blob/main/docs/scan.md).
- If the workflow rejects an input, use only `directory` and `fail-on` from
  the table above.
- If a scan returns `1` or `2`, review the logs and generated report for a
  scanner/output or usage error before treating it as a policy result.
- Keep repository tokens and sensitive finding data out of links, logs, and
  external uploads unless you have independently reviewed their destination.

## Organization-wide governance

Scan helps a repository identify agent-context drift. Teams that need shared
policy, accountable approvals, governed distribution, fleet-wide visibility,
or durable evidence across repositories can discuss the commercial
organization-wide control plane. It is a separate product, not an additional
mode unlocked in this free action.

[Talk to FlowHarness about organization-wide governance](https://flowharness.ai/go/scan)
uses a fixed acquisition link and carries no repository, path, finding, report,
token, or secret data.

Licensed under the Apache License 2.0. See [LICENSE](LICENSE).
