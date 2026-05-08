# agentic-workflows

Reusable [GitHub Agentic Workflows](https://github.com/github/gh-aw) for the `awftst` org.

## Workflows

### `security-audit`

Runs a full multi-phase security audit of a repository using the [`security-audit` skill](https://github.com/hanskhe/poor-mans-mythos/tree/main/skills/security-audit) from `hanskhe/poor-mans-mythos`. The audit performs reconnaissance, file classification, per-file review, scenario-driven cross-file review, finding validation, and coverage analysis. The final report is uploaded as a workflow artifact.

**Schedule:** 1st of every month at 06:00 UTC, plus on-demand via `workflow_dispatch`.

**Engine:** Copilot (gh-aw default).

**Pinned skill version:** `hanskhe/poor-mans-mythos@ca2827974194f1f6d11585026ac9f20ed4e762d4`. To upgrade, edit the `SKILL_SHA` in `.github/workflows/security-audit.md`, run `gh aw compile security-audit --strict`, and commit both the `.md` and `.lock.yml` files.

#### Install in a consumer repo

Prerequisites: the `gh aw` extension must be installed locally (see below).

```bash
gh aw add awftst/agentic-workflows/security-audit
git add .gitattributes .github/workflows/security-audit.md .github/workflows/security-audit.lock.yml
git commit -m "Add security-audit agentic workflow"
git push
```

This copies both the source `.md` and the compiled `.lock.yml` into the consumer repo's `.github/workflows/`. The compiled lock file is what GitHub Actions actually executes.

#### Run on demand

```bash
gh workflow run security-audit.lock.yml
```

Or trigger from the GitHub UI: **Actions → security-audit → Run workflow**.

#### Where the report appears

Each run uploads `report-<YYYY-MM-DD>.md` as an artifact named `security-audit-report-<run_id>`. Open the run on the GitHub Actions page and download it from the **Artifacts** section. Retention: 90 days.

#### Cost expectations

The skill is **token-heavy by design**. From the upstream `SKILL.md`:

> A full audit on a ~1,000-file repo with 10 scenarios and 50–100 reviewed files plus validation can easily reach **several million tokens**.

Monthly cadence keeps the bill bounded. Trigger `workflow_dispatch` runs deliberately.

#### Required permissions and secrets

The workflow uses the default `GITHUB_TOKEN` with read-only `contents: read` permission. The Copilot engine credential is provided by gh-aw itself — no extra secrets needed.

## Installing the `gh aw` CLI

```bash
curl -sL https://raw.githubusercontent.com/github/gh-aw/main/install-gh-aw.sh | bash
gh aw version
```

## Repository conventions

- Workflow source lives in `.github/workflows/<name>.md` (markdown with YAML frontmatter).
- The compiled GitHub Actions YAML lives alongside as `.github/workflows/<name>.lock.yml` and is checked in.
- `.gitattributes` marks `*.lock.yml` as `linguist-generated` and `merge=ours` so it doesn't pollute diffs or merge conflicts.
- After editing any `.md` workflow, always run `gh aw compile <name> --strict` and commit both files together.
