# Vendored skill: security-audit

This directory is a verbatim copy of the `security-audit` Claude Code skill from
[hanskhe/poor-mans-mythos](https://github.com/hanskhe/poor-mans-mythos).

- **Upstream repo:** https://github.com/hanskhe/poor-mans-mythos
- **Upstream path:** `skills/security-audit/`
- **Vendored from commit:** `ca2827974194f1f6d11585026ac9f20ed4e762d4` (branch `main`, release `0.2.0`)
- **License:** MIT — see `LICENSE` in this directory.

## Why vendored?

We ship the skill alongside the shared agentic workflow component so consumer
repositories pick up exactly the version we have validated. To upgrade, refresh
these files from a newer upstream commit, update the SHA above, and bump any
references in `agentic-workflows/.github/workflows/components/security-audit.md`.

## Files

```
SKILL.md
LICENSE
references/finding-format.md
references/owasp-api-top10.md
references/owasp-top10.md
references/recon-checklist.md
references/risk-rating-guide.md
references/scenarios.md
references/severity-rubric.md
```

## Refresh procedure

```bash
cd agentic-workflows/skills/security-audit
BASE=https://raw.githubusercontent.com/hanskhe/poor-mans-mythos/<sha>
curl -sSfLo SKILL.md  "$BASE/skills/security-audit/SKILL.md"
curl -sSfLo LICENSE   "$BASE/LICENSE"
for f in finding-format owasp-api-top10 owasp-top10 recon-checklist \
         risk-rating-guide scenarios severity-rubric; do
  curl -sSfLo "references/$f.md" "$BASE/skills/security-audit/references/$f.md"
done
```

Then update the commit SHA in this file.
