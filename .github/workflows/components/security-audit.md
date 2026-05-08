---
engine: copilot
permissions:
  contents: read
network: defaults
steps:
  - name: Checkout vendored security-audit skill
    uses: actions/checkout@v4
    with:
      repository: awftst/agentic-workflows
      ref: main
      token: ${{ secrets.AGENTIC_WORKFLOWS_TOKEN }}
      path: .agentic-workflows-src
      sparse-checkout: skills/security-audit
      sparse-checkout-cone-mode: false
      persist-credentials: false
  - name: Stage skill files into audited repo
    shell: bash
    run: |
      set -euo pipefail
      mkdir -p .security-audit-skill/references .security-audit
      cp .agentic-workflows-src/skills/security-audit/SKILL.md .security-audit-skill/SKILL.md
      cp .agentic-workflows-src/skills/security-audit/references/*.md .security-audit-skill/references/
      printf '*\n' > .security-audit/.gitignore
      ls -la .security-audit-skill .security-audit-skill/references
safe-outputs:
  upload-artifact:
    max-uploads: 1
    default-retention-days: 120
    max-retention-days: 120
    max-size-bytes: 524288000
    allowed-paths:
      - ".security-audit/**"
---

<!--
Shared agentic workflow: security-audit
Source repo: awftst/agentic-workflows (private)
Skill: vendored from hanskhe/poor-mans-mythos (MIT). See
       skills/security-audit/SOURCE.md for the upstream commit.

Required secret in the consuming repository:
  - AGENTIC_WORKFLOWS_TOKEN: PAT (or GitHub App token) with
    Contents: read on awftst/agentic-workflows. Used by actions/checkout
    to pull the vendored skill files.

Outputs:
  Workflow artifact named `security-audit-report-<run_id>` containing the
  contents of `.security-audit/` (intermediate analysis files plus the
  final report.md). Retention: 120 days.

Engine: GitHub Copilot. The audit is intentionally token-heavy; monthly is
the recommended maximum cadence.
-->

# Security audit of this repository

You are running the **Poor Man's Mythos `security-audit`** skill against this
repository. The skill files have already been staged for you on disk:

- Skill instructions: `./.security-audit-skill/SKILL.md`
- Reference material: `./.security-audit-skill/references/*.md`
  - `recon-checklist.md`
  - `risk-rating-guide.md`
  - `owasp-top10.md`
  - `owasp-api-top10.md`
  - `severity-rubric.md`
  - `finding-format.md`
  - `scenarios.md`

The audited repository is the current working directory (`${{ github.workspace }}`).

## Your task

1. **Read the entire skill specification** at `./.security-audit-skill/SKILL.md`
   before doing anything else. Treat it as authoritative — follow its five
   phases (Recon → Classify → Review → Validate → Coverage) exactly.
2. **Treat `./.security-audit-skill/references/` as the skill's `references/`
   directory.** Whenever the SKILL.md tells you to copy or read files from
   `<base-directory>/references/X.md`, read them from
   `./.security-audit-skill/references/X.md` instead. The `<base-directory>`
   value to use is `./.security-audit-skill`.
3. **Write all output to `./.security-audit/`** in the workspace root. The
   directory and its `.gitignore` (`*`) have already been created — do not
   delete them.
4. **Phases to produce on disk** (per the skill spec):
   - `recon.md`
   - `file-rankings.md`
   - per-file and per-scenario raw findings
   - validated findings
   - `coverage.md`
   - **`report.md`** — the final consolidated security report
5. The final `report.md` MUST include: an executive summary, every validated
   finding (with severity, CWE, OWASP category, code excerpt, data-flow
   trace, and fix recommendation), and the explicit coverage statement.

## Execution constraints

- You are running non-interactively in CI. The skill expects to ask the user
  one or two threat-model questions in Phase 0; **do not block on input** —
  make conservative assumptions, document them in `recon.md`, and proceed.
- Do not attempt to commit, push, open issues, or open pull requests. The
  workflow has read-only `contents` permission.
- Do not exfiltrate findings to any external service.
- Stay within the runner's filesystem; the only network access is what the
  Agent Workflow Firewall already permits.
- The audit may legitimately run for a long time. Pace yourself, but make
  steady progress through every phase — do not stop after Phase 0 or
  Phase 1.

## Publishing the report

After all five phases finish, call the `upload_artifact` safe-output tool
exactly once:

```
upload_artifact(
  name = "security-audit-report-${{ github.run_id }}",
  path = ".security-audit",
  retention_days = 120
)
```

Then print a short summary to the run log:

- Number of validated findings, broken down by severity (Critical / High /
  Medium / Low).
- Path to each top-level artifact file under `.security-audit/`.
- Any phase you had to truncate or skip, and why.
