# agentic-workflows

Reusable [GitHub Agentic Workflows](https://github.github.com/gh-aw/) for the
`awftst` organization.

## Components

### `shared/security-audit.md`

Runs the [Poor Man's Mythos](https://github.com/hanskhe/poor-mans-mythos)
`security-audit` Claude Code skill against the consuming repository and uploads
the resulting report as a workflow artifact.

- **Engine:** `claude`
- **Permissions:** read-only (`contents: read`)
- **Artifact:** `security-audit-report-<run_id>` (120-day retention)
- **Output location in workspace:** `.security-audit/report-<date>.md`

## Consuming the security-audit workflow from another repo

1. **Add the secret** `ANTHROPIC_API_KEY` to the consuming repository
   (Settings → Secrets and variables → Actions → New repository secret).

2. **Create** `.github/workflows/security-audit.md` in the consuming repo:

   ```yaml
   ---
   on:
     schedule: monthly
     workflow_dispatch:
   permissions:
     contents: read
   imports:
     - awftst/agentic-workflows/.github/workflows/shared/security-audit.md@main
   ---

   See the imported shared component for full instructions.
   ```

3. **Compile** the workflow to generate the lock file:

   ```bash
   gh aw compile security-audit
   ```

4. **Commit** both files:

   ```bash
   git add .github/workflows/security-audit.md .github/workflows/security-audit.lock.yml
   git commit -m "Add monthly security audit workflow"
   ```

5. **Make sure** `.github/workflows/*.lock.yml linguist-generated=true merge=ours`
   is in `.gitattributes`.

## Pinning to a version

Replace `@main` in the import with a tag or commit SHA for reproducibility:

```yaml
imports:
  - awftst/agentic-workflows/.github/workflows/shared/security-audit.md@v1.0.0
```

## Cost warning

The security-audit skill is intentionally token-heavy. A full audit on a
~1,000-file repo can reach **several million tokens** of Anthropic API usage.
A monthly schedule is the recommended default — do not run more frequently
without monitoring spend.
