---
on:
  schedule:
    - cron: "0 6 1 * *"
  workflow_dispatch:

permissions:
  contents: read

timeout-minutes: 60

network:
  allowed:
    - github

steps:
  - name: Stage security-audit skill (pinned SHA)
    env:
      SKILL_REPO: https://github.com/hanskhe/poor-mans-mythos
      SKILL_SHA: ca2827974194f1f6d11585026ac9f20ed4e762d4
    run: |
      set -euo pipefail
      mkdir -p /tmp/gh-aw/skills
      git clone --no-checkout "$SKILL_REPO" /tmp/gh-aw/skills/poor-mans-mythos
      git -C /tmp/gh-aw/skills/poor-mans-mythos checkout "$SKILL_SHA"
      test -f /tmp/gh-aw/skills/poor-mans-mythos/skills/security-audit/SKILL.md
      for f in recon-checklist risk-rating-guide owasp-top10 owasp-api-top10 severity-rubric finding-format scenarios; do
        test -f /tmp/gh-aw/skills/poor-mans-mythos/skills/security-audit/references/$f.md
      done
      echo "Skill staged at /tmp/gh-aw/skills/poor-mans-mythos/skills/security-audit at $SKILL_SHA"

post-steps:
  - name: Upload security audit report
    if: always()
    uses: actions/upload-artifact@v4
    with:
      name: security-audit-report-${{ github.run_id }}
      path: .security-audit/report-*.md
      if-no-files-found: error
      retention-days: 90
---

# Security Audit

You are running a full security audit of this repository in an unattended CI environment.

## Target

The repository to audit is the current working directory (`$GITHUB_WORKSPACE`). Treat the entire checked-out tree as the audit target.

## Authoritative instructions

The `security-audit` skill from `hanskhe/poor-mans-mythos` (pinned to commit `ca2827974194f1f6d11585026ac9f20ed4e762d4`) has been staged on the runner at:

```
/tmp/gh-aw/skills/poor-mans-mythos/skills/security-audit/
```

This directory contains:

- `SKILL.md` — the full multi-phase audit procedure (Phases 0–4 plus the final report).
- `references/` — the seven reference files the skill stages into `.security-audit/refs/`:
  `recon-checklist.md`, `risk-rating-guide.md`, `owasp-top10.md`, `owasp-api-top10.md`, `severity-rubric.md`, `finding-format.md`, `scenarios.md`.

**Read `SKILL.md` end-to-end before doing anything else.** It is authoritative — follow it exactly, including its file-staging discipline, agent-prompt templates, severity rubric, and report format. The references in `SKILL.md` written as `<base-directory>/references` resolve to:

```
/tmp/gh-aw/skills/poor-mans-mythos/skills/security-audit/references
```

Use that path verbatim for the `cp` step in Phase 0.1.

## Non-interactive overrides (CI run, no human in the loop)

The skill assumes an interactive operator. You are running unattended, so apply these overrides:

1. **Phase 0.3 (threat-model dialog)** — Do NOT ask the user any questions. Infer asset, user base, sensitive-data, and out-of-scope assumptions from `README*`, manifests (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, etc.), `Dockerfile*`, `docker-compose*.yml`, `.github/workflows/*`, and the source tree. Record every assumption in `recon.md` under a clearly-labeled section titled `## Assumptions (CI run, no human dialog)`.
2. **Phase 1.4 (file-count prompt)** — Do NOT ask. Treat the answer as `all`. Process the entire Review bucket in Phase 2a.
3. **Phase 2c (validation count prompt)** — Do NOT ask. Treat the answer as `all`. Validate every finding in Phase 3.
4. **Closing message** — Skip the chat-style closing. The artifact upload step handles delivery.

Do NOT skip any phase to save time or tokens. The user accepted the cost when they scheduled this workflow.

## Output requirements

- Write the final report to `.security-audit/report-<YYYY-MM-DD>.md` using today's UTC date, exactly as specified at the bottom of `SKILL.md`.
- Ensure `.security-audit/.gitignore` containing `*` is created in Phase 0 — this is mandatory because findings may quote real secret values verbatim.
- All intermediate files (`recon.md`, `file-rankings.md`, `raw-findings-2a.md`, `raw-findings-2b.md`, `findings-merged.md`, `validated-findings.md`, `coverage.md`) must be produced during the run; only the final `report-<date>.md` is uploaded as an artifact.

## Engine-specific note

You are running under the Copilot engine, which may not support spawning many parallel subagents the way the skill describes for Phases 2a and 2b. Where parallel subagent dispatch is unavailable, perform the equivalent work sequentially in this single agent context — but do not skip files, scenarios, validation, or coverage. The skill's discipline (one finding per `---FINDING---` block, source-tagged with `2a:<path>` or `2b:S<n>`, validated into `---VALIDATED---` blocks, coverage table written) must be preserved.

Begin by reading `/tmp/gh-aw/skills/poor-mans-mythos/skills/security-audit/SKILL.md`.
