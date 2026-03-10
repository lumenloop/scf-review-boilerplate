# SCF Review Boilerplate

An open-source, turnkey review kit for [Stellar Community Fund](https://communityfund.stellar.org/) rounds. Drop a CSV export from Airtable, point Claude at this repo, and it reviews every submission using a team of parallel agents.

## Try It

Drop your CSV in `data/`, then tell Claude:

```
Use https://github.com/lumenloop/scf-review-boilerplate and review RFP #41
```

That's it. Claude clones the repo, reads the CSV, fetches RFP specs, spawns parallel review agents, and produces ranked reviews for every submission.

## Quick Start

```bash
git clone https://github.com/lumenloop/scf-review-boilerplate
cd scf-review-boilerplate
```

1. Export your round's submissions from Airtable as CSV
2. Place the CSV in `data/` (e.g. `data/submissions.csv`)
3. Open Claude Code in this directory:
   ```bash
   claude
   ```
4. Tell Claude:
   ```
   Review the submissions in data/submissions.csv — this is an RFP Track round
   ```
   (or "Open Track" / "Integration Track" — Claude will ask if you don't specify)

Claude reads `CLAUDE.md`, auto-detects CSV columns, loads the skills from `.claude/skills/`, and starts the multi-phase review process automatically.

## Supported Tracks

| Track | What It Reviews | Key Scoring Dimension |
|---|---|---|
| **Open Track** | New projects/protocols for Stellar | Ecosystem Impact |
| **Integration Track** | Stellar building block integrations into existing products | Integration Partner Fit |
| **RFP Track** | Proposals responding to specific SCF requests (listed in handbook) | Spec Compliance (fetches the RFP spec and checks against it) |

## What It Does

| Phase | What Happens |
|---|---|
| **0. Detect Track** | Identifies Open/Integration/RFP, fetches RFP spec if applicable |
| **1. Categorize** | Auto-detects CSV columns, categorizes submissions, builds batches |
| **2. Parallel Review** | Spawns agents to review batches in parallel (prescreen, track-specific scoring, link verification, competitive analysis) |
| **3. Calibration** | Cross-checks scores across batches for consistency |
| **4. Final Ranking** | Produces ranked list with referral adjustments and constructive feedback |

## CSV Columns

**Column names are auto-detected** — Claude maps CSV headers to review fields using semantic matching, so it handles column name changes between rounds. It needs columns for:

- Project name and description
- Budget
- Products/services and technical architecture
- Traction evidence
- Tranche deliverables and dates (T1, T2, T3)
- Team description
- Referral info (if any)
- Track/category info
- Status and round identifier

Claude will write the detected column mapping to `reviews/00-column-mapping.md` so you can verify it.

## Output

All review output goes to `reviews/`:

```
reviews/
  00-column-mapping.md  # Auto-detected CSV column mapping
  00-master-index.md    # Summary stats, categories, batches
  00-rfp-spec.md        # (RFP Track only) Extracted RFP requirements
  01-ranking.md         # Final ranked list with recommendations
  results.csv           # Machine-readable results (importable to Airtable)
  calibration-top-bottom.md
  calibration-middle.md
  {project-slug}.md     # Individual review per submission
```

## Skills

The review process uses these Claude skills (included in `.claude/skills/`):

| Skill | Purpose |
|---|---|
| `scf-round-reviewer` | Orchestrates the full review pipeline |
| `scf-reviewer` | Core evaluation framework (track-specific weightings) |
| `scf-prescreen-checker` | 5-area prescreen simulation |
| `scf-budget-builder` | Budget validation with benchmarks |
| `scf-competitor-analyst` | Competitive landscape analysis |
| `fetch-external-doc` | URL resolution for Google Docs, Drive PDFs, GitHub, Notion, IPFS |

Skills are sourced from [awesome-stellar-community-fund](https://github.com/lumenloop/awesome-stellar-community-fund/tree/main/skills).

## Adapting for Other Rounds

- Works with any SCF round — just export the CSV from Airtable
- Budget benchmarks and category definitions can be adjusted in the skills
- The round number is auto-detected from the CSV `Round` column

## License

MIT
