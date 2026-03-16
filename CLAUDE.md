# SCF Review Boilerplate

Turnkey review kit for Stellar Community Fund (SCF) rounds. You review all submissions from a CSV export and produce structured evaluation reports.

## Prerequisites — Install Before Reviewing

You MUST have these external skills installed. They provide Stellar and smart contract domain knowledge critical for accurate technical evaluation. **Without them, Technical Depth and Stellar Integration scores will lack domain-specific depth.**

| Skill | Repository | Why It's Needed |
|---|---|---|
| **Stellar Dev Skill** | [stellar/stellar-dev-skill](https://github.com/stellar/stellar-dev-skill) | Soroban SDK, Stellar RPC vs Horizon, SEPs, Smart Accounts, ecosystem context — used when scoring Technical Depth and Stellar Integration |
| **OpenZeppelin Skills** | [OpenZeppelin/openzeppelin-skills](https://github.com/OpenZeppelin/openzeppelin-skills) | Smart contract security patterns, Stellar contract setup/upgrades — essential for RFP Track (C-Address Tooling requires OpenZeppelin Smart Account standard) and any Soroban contract evaluation |

Follow each repository's README for installation. If either is missing when you start a review, **stop and tell the user to install them first**.

## How to Run a Review

1. Find the CSV in `data/` (ask if multiple)
2. Follow `.claude/skills/scf-round-reviewer.md` — it orchestrates the full pipeline

That skill handles everything: column auto-detection, track detection, batching, parallel review agents, calibration, and final ranking. Read it before starting.

## Key Rules

- **CSV is the single source of truth.** All submission data comes from the CSV.
- **Auto-detect columns.** Never hardcode column names — Airtable names change between rounds. Use semantic matching (see column mapping table in `.claude/skills/scf-round-reviewer.md`).
- **Fetch architecture docs with `curl`, NOT WebFetch.** Google Docs/Drive URLs must be fetched via `curl -sL` — WebFetch cannot follow Google's redirect chain. See `.claude/skills/fetch-external-doc.md`.
- **Deliverable rigor matters.** Tranches must have concrete, verifiable completion criteria — not just business outcomes like "$1M volume" or "2 clients live". Budget must have per-deliverable breakdowns. FLAG completeness and cap Budget at 2-2.5/5 if missing. A low budget total does NOT equal a well-justified budget.
- **Slug filenames.** Use kebab-case project names for output files.
- **Score inflation.** 5/5 is rare. Use 1-2 for weak submissions. Verify claims independently via WebSearch.
- **Track-aware scoring.** Open, Integration, and RFP tracks each have different dimensions and weights. Using the wrong track's dimensions produces garbage.
- **Handle missing data gracefully.** Note gaps in the review but don't fail.
- **No PDF generation.** Output markdown and CSV only.

## Output

All output goes to `reviews/`:
- `00-column-mapping.md` — auto-detected CSV → review field mapping
- `00-master-index.md` — summary stats, categories, batch assignments
- `00-rfp-spec.md` — (RFP Track) extracted spec and compliance checklists
- `00-external-docs.md` — pre-fetched architecture docs (Google Docs, Drive, GitHub)
- `01-ranking.md` — final ranked list with recommendations
- `results.csv` — machine-readable results (importable to Airtable)
- `prescreen-results.csv` — prescreen check details per submission
- `{project-slug}.md` — individual review per submission
