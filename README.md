# agentic-workflows

Reusable [GitHub Agentic Workflows](https://github.github.com/gh-aw/) for the
`awftst` organization.

## Components

### `components/security-audit.md`

Runs the vendored [Poor Man's Mythos](https://github.com/hanskhe/poor-mans-mythos)
`security-audit` skill against the consuming repository and uploads the
resulting report as a workflow artifact.

- **Engine:** GitHub Copilot
- **Schedule:** monthly + `workflow_dispatch` (defined by the consumer workflow)
- **Permissions:** read-only (`contents: read`)
- **Skill source:** vendored at [`skills/security-audit/`](./skills/security-audit/)
  (MIT — see [`SOURCE.md`](./skills/security-audit/SOURCE.md) for the upstream commit)
- **Artifact:** `security-audit-report-<run_id>` (120-day retention)
- **Workspace output:** `./.security-audit/` containing `recon.md`,
  `file-rankings.md`, raw + validated findings, `coverage.md`, and the final
  `report.md`.

## Consuming the security-audit workflow from another repo

1. **Create a PAT** with `Contents: read` permission on `awftst/agentic-workflows`
   (the repo is private, so the consumer needs a token to pull the vendored
   skill files at runtime). A fine-grained personal access token or a GitHub
   App installation token both work.

2. **Add the secret** to the consuming repository:

   - Settings → Secrets and variables → Actions → New repository secret
   - Name: `AGENTIC_WORKFLOWS_TOKEN`
   - Value: the PAT from step 1

3. **Create** `.github/workflows/security-audit.md` in the consuming repo:

   ```aw
   ---
   on:
     schedule:
       - cron: "0 9 1 * *"   # 09:00 UTC on the 1st of every month
     workflow_dispatch:
   permissions:
     contents: read
   imports:
     - awftst/agentic-workflows/.github/workflows/components/security-audit.md@main
   ---

   Run the imported security audit and upload the report as an artifact.
   ```

   `gh aw` does not provide a fuzzy `monthly` schedule, so use a standard cron
   expression. Pick any day 1–28 to avoid skipping short months. The
   `workflow_dispatch:` trigger enables on-demand runs from the Actions tab.

4. **Compile** the workflow to generate the lock file:

   ```bash
   gh aw compile security-audit
   ```

5. **Ensure** `.github/workflows/*.lock.yml linguist-generated=true merge=ours`
   is in the consuming repo's `.gitattributes`.

6. **Commit** both files:

   ```bash
   git add .github/workflows/security-audit.md \
           .github/workflows/security-audit.lock.yml \
           .gitattributes
   git commit -m "Add monthly security audit workflow"
   git push
   ```

7. The first run will appear in the Actions tab. Download the
   `security-audit-report-<run_id>` artifact when it finishes — `report.md`
   inside is the consolidated report.

## Pinning to a version

Replace `@main` in the import with a tag or commit SHA for reproducibility:

```yaml
imports:
  - awftst/agentic-workflows/.github/workflows/components/security-audit.md@v1.0.0
```

The component checks out `awftst/agentic-workflows@main` to fetch the
skill files. If you pin the import, also pin the `ref:` inside
[`components/security-audit.md`](./.github/workflows/components/security-audit.md)
(or fork the component) so the skill version matches the workflow version.

## Refreshing the vendored skill

Follow the procedure in
[`skills/security-audit/SOURCE.md`](./skills/security-audit/SOURCE.md). Update
the recorded upstream commit SHA after refreshing and bump the version of any
release tag you cut from this repository.

## Cost warning

The Poor Man's Mythos `security-audit` skill is **intentionally token-heavy**.
A full audit on a ~1,000-file repository with all ten scenario reviewers can
reach **several million tokens** of Copilot usage. Stick with the monthly
schedule (and on-demand) recommended above — do not move to weekly or daily
without monitoring spend.

## License

The shared workflow component and this repo's tooling are governed by the
license of this repository. The vendored `security-audit` skill is MIT-licensed
by the upstream author; see [`skills/security-audit/LICENSE`](./skills/security-audit/LICENSE).
