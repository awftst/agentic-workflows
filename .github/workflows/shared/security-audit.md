---
description: Reusable security audit shared component. Runs the Poor Man's Mythos security-audit skill against the consuming repository and uploads the resulting report as a workflow artifact.
engine:
  id: claude
permissions:
  contents: read
network:
  allowed:
    - github.com
    - raw.githubusercontent.com
    - codeload.github.com
    - objects.githubusercontent.com
    - api.anthropic.com
    - statsig.anthropic.com
tools:
  cache-memory: true
steps:
  - name: Install Poor Man's Mythos security-audit skill
    shell: bash
    run: |
      set -euo pipefail
      echo "Cloning hanskhe/poor-mans-mythos..."
      git clone --depth 1 https://github.com/hanskhe/poor-mans-mythos.git /tmp/pmm
      mkdir -p .claude/skills
      cp -r /tmp/pmm/skills/security-audit .claude/skills/security-audit
      echo "Skill staged at .claude/skills/security-audit"
      ls -la .claude/skills/security-audit
post-steps:
  - name: Upload security audit report
    if: always()
    uses: actions/upload-artifact@v4
    with:
      name: security-audit-report-${{ github.run_id }}
      path: .security-audit/
      if-no-files-found: warn
      retention-days: 120
---

<!--
Poor Man's Mythos Security Audit — Shared Component

Source skill: https://github.com/hanskhe/poor-mans-mythos
Reference docs: https://github.github.com/gh-aw/reference/imports/

What this component does:
  - Stages the security-audit Claude Code skill into .claude/skills/ in the workspace
  - Runs the skill non-interactively against the checked-out repository
  - Uploads the full .security-audit/ directory as a workflow artifact
    (retention: 120 days; name: security-audit-report-<run_id>)

Required secrets in the consumer repository:
  - ANTHROPIC_API_KEY  (consumed by the claude engine)

Cost warning:
  The security-audit skill is intentionally token-heavy. A full audit on a
  ~1,000-file repo can reach several million tokens. Schedule sparingly.

Usage in a consumer workflow:
  ---
  on:
    schedule: monthly
    workflow_dispatch:
  permissions:
    contents: read
  imports:
    - awftst/agentic-workflows/.github/workflows/shared/security-audit.md@main
  ---
-->

# Security Audit

Run a complete security audit of the checked-out repository using the
**security-audit** Claude Code skill that was staged at
`.claude/skills/security-audit/` in the pre-step.

## How to invoke the skill

The skill's `SKILL.md` lives at `.claude/skills/security-audit/SKILL.md`.
Read it first, then execute its full five-phase workflow exactly as documented:
Phase 0 (Recon), Phase 1 (Classify), Phase 2a (Per-file review), Phase 2b
(Scenario-driven review), Phase 2c (Deduplication), Phase 3 (Validation),
Phase 4 (Coverage), and the Final Report.

When staging reference files in Phase 0.1, the skill's base directory is
`.claude/skills/security-audit`, so the references live at
`.claude/skills/security-audit/references/`.

## Non-interactive execution — REQUIRED

This workflow runs unattended on a schedule. There is no human to answer
questions. You **must not** prompt the user. Whenever the skill asks an
interactive question, apply these answers automatically and continue:

- **Asset question (Phase 0.3)**: "Treat all source code, configuration,
  credentials, and customer data as the highest-impact assets. Assume an
  internet-facing service with no out-of-scope areas."
- **User-base / out-of-scope question**: "Internet-facing service. Nothing
  is out of scope."
- **Sensitive-data question**: "Assume the codebase may handle PII,
  authentication tokens, and secrets."
- **Phase 1.4 file-count question** ("How many would you like me to cover
  in the per-file pass?"): answer **`all`** — review every file in the
  Review bucket.
- **Phase 2c dedup / validation count question** ("How many would you
  like me to validate?"): answer **`all`** — validate every finding.
- **Any other clarifying question**: pick the most thorough,
  security-conservative default and continue without asking.

Do not pause, do not summarize and wait for confirmation between phases —
run the entire pipeline end-to-end in a single pass.

## Output

The skill writes everything to `.security-audit/` in the repo root,
including the final `report-<YYYY-MM-DD>.md`. After the agent finishes,
the post-step uploads this directory as a workflow artifact named
`security-audit-report-<run_id>` with 120-day retention.

You do **not** need to upload artifacts yourself or commit anything.
Just complete the audit and leave the files in `.security-audit/`.

## If something goes wrong

If the skill fails to stage references, abort with a clear error message
written to `.security-audit/ERROR.md` so it is captured in the artifact.
If the audit fails partway through, write whatever findings exist to
`.security-audit/` before exiting — partial output is uploaded by the
`if: always()` post-step.
