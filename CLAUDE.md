# SCF Review Boilerplate

This repo is a turnkey review kit for Stellar Community Fund (SCF) rounds. When pointed at this repo, your job is to review all submissions from a CSV export and produce structured evaluation reports.

## How It Works

1. The user places a CSV export from Airtable in `data/`
2. You read the CSV — it contains all submission data
3. You follow the `scf-round-reviewer` skill in `.claude/skills/` to orchestrate the full review pipeline
4. All output goes to `reviews/` as markdown files

## Getting Started

When the user asks you to review submissions:

1. Find the CSV file in `data/` (any `.csv` file — ask the user if there are multiple)
2. Read the CSV headers and auto-map columns (see below)
3. Detect the track (Open, Integration, or RFP) — ask the user if ambiguous
4. Invoke the review pipeline by following `.claude/skills/scf-round-reviewer.md`

## CSV Column Auto-Detection

**Do NOT hardcode column names.** Airtable column names change between rounds (e.g., "SCF #41 Assigned Reviewers" vs "SCF #42 Assigned Reviewers"). Instead, read the CSV headers and map them to review fields by semantic matching.

### Required Review Fields

You need to find columns that map to these fields. Use fuzzy matching — the column name won't be exact every time.

| Review Field | Look For (patterns) | Example Column Names |
|---|---|---|
| **project_name** | "project", "submission" | `Submission / Project`, `Project`, `Project Name` |
| **description** | "description" | `Description (from Project)`, `Description`, `Project Description` |
| **budget** | "budget" (parse as number, strip `$` and commas) | `Budget`, `Requested Budget`, `Budget (USD)` |
| **products_services** | "products", "services", "what" | `Products & Services`, `Products and Services` |
| **technical_architecture** | "technical", "architecture" | `Technical Architecture`, `Architecture`, `Tech Architecture` |
| **traction** | "traction" | `Traction Evidence`, `Traction`, `Traction & Evidence` |
| **tranche_1** | "tranche 1", "deliverables", "mvp" | `Tranche 1 - Deliverables`, `T1 Deliverables`, `MVP Deliverables` |
| **tranche_1_date** | "tranche 1" + "date" | `Tranche 1 Date`, `T1 Date`, `MVP Date` |
| **tranche_2** | "tranche 2", "testnet" | `Tranche 2 - Testnet`, `T2 Deliverables`, `Testnet Deliverables` |
| **tranche_2_date** | "tranche 2" + "date" | `Tranche 2 Date`, `T2 Date` |
| **tranche_3** | "tranche 3", "mainnet" | `Tranche 3 - Mainnet`, `T3 Deliverables`, `Mainnet Deliverables` |
| **tranche_3_date** | "tranche 3" + "date" | `Tranche 3 Date`, `T3 Date` |
| **team** | "team" | `Team Description (from Project)`, `Team`, `Team Description` |
| **referral** | "referral", "referrer" (not "how did" or "readiness") | `Referral`, `Referrer`, `Referred By` |
| **referral_context** | "how did referrer", "referral context" | `How did referrer meet team?` |
| **referral_build_readiness** | "build readiness" | `Build Readiness from Referral` |
| **referral_team_assessment** | "team assessment" | `Team Assessment From Referral` |
| **track_specifics** | "track", "rfp", "specifics" | `RFP Track Specifics`, `Track`, `Track Specifics` |
| **status** | "status" | `Status`, `Submission Status` |
| **video** | "video" | `Video`, `Demo Video`, `Video URL` |
| **round** | "round", "scf" | `Round`, `SCF Round` |

### Auto-Detection Process

1. Read all CSV headers
2. For each required review field, find the best-matching column using the patterns above
3. Log the mapping you chose (write it to `reviews/00-column-mapping.md` so the user can verify)
4. If a required field can't be mapped, warn the user and ask for clarification
5. Ignore any columns that don't map to a review field (internal Airtable fields like "Assigned Reviewers", "Completed Reviews", "Tentative Panel Decision", "Thumbnail", etc.)

## Track Detection

The review process differs significantly by track. Detect the track from:
1. The CSV column names (e.g., "RFP Track Specifics" suggests RFP track)
2. The content of a track/round column
3. Ask the user if ambiguous

See the Track-Specific Scoring section in `scf-round-reviewer.md` for how each track changes the review.

## Key Rules

- **CSV is the single source of truth.** Everything comes from the CSV.
- **No PDF generation.** Output markdown only.
- **Auto-detect columns.** Never hardcode column names. Map them on each run.
- **Slug filenames.** Use kebab-case project names for output files.
- **Fetch architecture docs.** The technical architecture column often contains URLs. Fetch these during review.
- **Handle missing data gracefully.** Note gaps in the review but don't fail.
- **Round-agnostic.** Detect the round from the CSV. Don't hardcode round numbers.
- **Track-aware.** Use the correct scoring dimensions and weights for the detected track.
