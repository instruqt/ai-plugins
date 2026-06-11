# Generate Track Command

You are helping a user generate an Instruqt track. This command detects the current state of the track, presents it, and picks up from wherever the user left off — or starts fresh if they prefer.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:generate-track` — generate a complete track, resuming from current state
- `/track:generate-track --fresh` — ignore existing state and start from scratch

## Paths

Resolve paths:
- `TRACK_RESEARCH_DIR`: if set use it, otherwise default to `~/.instruqt/companies`
- `TRACK_OUTPUT_DIR`: if set use it, otherwise default to `~/.instruqt/tracks`

## Workflow

### Step 1: Detect Current State

Scan the filesystem to determine what exists:

**Research context** (`${TRACK_RESEARCH_DIR}/<company-slug>/`):
- [ ] `company.md` exists
- [ ] `style-guide.md` exists
- [ ] `products/` has at least one file

**Track plan** (`${TRACK_OUTPUT_DIR}/`):
- [ ] `track.yml` exists
- [ ] `config.yml` exists
- [ ] `.instruqt/plan.md` exists

**Scores** (`${TRACK_OUTPUT_DIR}/.instruqt/`):
- [ ] `scores.json` exists (scoring checkpoint)

**Challenge plans and content** (for each challenge in .instruqt/plan.md, or each `NN-*/` directory):
- [ ] `.instruqt/<NN-slug>/plan.md` exists (planned)
- [ ] `<NN-slug>/assignment.md` exists (generated)
- [ ] `<NN-slug>/check-*` exists (scripts generated)
- [ ] `scores.json` has entry with `status: "passed"` or `"escalated"` (scored)

### Step 2: Present State and Prompt

Present the detected state to the user as a clear summary:

```
Track state for [company] / [track title]:

Research:
  ✓ Company context (company.md, style-guide.md)
  ✓ Product context (terraform.md)

Track plan:
  ✓ track.yml
  ✓ config.yml
  ✓ .instruqt/plan.md (5 challenges)

Challenges:
  ✓ 01-getting-started     — planned, generated, scored (passed)
  ✓ 02-configure-provider   — planned, generated, scored (passed)
  ✓ 03-create-resources     — planned, generated, scored (escalated — 1 finding)
  ✗ 04-manage-state         — planned, not generated
  ✗ 05-collaborate          — planned, not generated
```

Then prompt:

- **Continue** — validate existing content, then generate from challenge 04 onward
- **Start over** — regenerate everything from scratch (deletes existing generated content, keeps research context)

If there is no existing state at all, skip the prompt and start from the beginning.

### Step 3: Validate Existing Content

For all content marked as existing, run validation:

1. `instruqt track validate` (if CLI available)
2. `shellcheck` on all existing scripts (if available)
3. Internal validation (platform limits, YAML syntax, script conventions, structural checks)

Report any issues found in existing content. If there are blocking issues in existing challenges, ask the user whether to fix them first or continue generating new challenges.

### Step 4: Fill Gaps

Work through each gap in order:

**If research context is missing:**
- Direct the user to run `/track:research-company` and `/track:research-product`
- Cannot proceed without at least `company.md`

**If track plan is missing:**
- Direct the user to run `/track:plan-track`
- Cannot proceed without `.instruqt/plan.md`, `track.yml`, and `config.yml`

**If challenge plans are missing:**
- Direct the user to run `/track:plan-challenge` for each missing plan
- Cannot generate a challenge without its plan

**For each challenge that needs generating** (in order):

#### 4a. Generate

Spawn the challenge-implementer agent with validation and scoring loops. Identical to what `/track:generate-challenge` does for a single challenge.

#### 4b. Validate (automatic)

After generation completes, automatically run:
1. Internal validation (platform limits, YAML syntax, script conventions, structural checks)
2. `shellcheck` on all bash scripts (if available)
3. `instruqt track validate` (if CLI available)

If there are blocking issues, fix them before proceeding.

#### 4c. Prompt for Testing

After validation passes, prompt the user:

- **Test automatically** — run `instruqt track test --path ${TRACK_OUTPUT_DIR}` to execute all lifecycle scripts in a sandbox. This tests the entire track up to and including the current challenge.
- **Test manually** — the user will run `instruqt track test` themselves. Pause and wait for confirmation.
- **Skip testing** — continue to the next challenge.

If auto-test fails, report the failures and ask: fix and re-test, or continue.

### Step 5: Final Summary

After all challenges are generated (or already existed):

1. Run a final `instruqt track validate` on the complete track
2. Present a summary:
   - Challenges: [existing] already present, [generated] newly generated
   - Validation status: [pass/fail]
   - Test status per challenge: [tested/skipped/pre-existing]
3. Suggest next steps: `/track:review-track` for a full quality scorecard, or `instruqt track push` to publish

## Error Handling

| Error | Message |
|-------|---------|
| No research context | Run `/track:research-company` first |
| No track plan | Run `/track:plan-track` first |
| Missing challenge plans | Run `/track:plan-challenge` for: [list missing] |
| `instruqt` CLI not found | Validation will use internal checks only. Install the Instruqt CLI for full validation. |
| `shellcheck` not found | Script linting skipped. Install shellcheck for script validation. |
| Test failure | Report which scripts failed and on which challenge. Ask: fix and re-test, or continue. |
| Validation failure in existing content | Report issues. Ask: fix first, or continue generating new challenges. |

## Important Notes

- This command is **idempotent** — run it as many times as needed, it always picks up from the current state
- Existing content is always re-validated before generating new content
- Challenges are generated strictly in order — later challenges may depend on earlier ones
- Validation runs automatically after every new challenge — no way to skip it
- Testing is always the user's choice — auto-test, manual, or skip
- The `id` field in track.yml and assignment.md frontmatter is intentionally omitted — Instruqt assigns UUIDs on first `instruqt track push`
- "Start over" preserves research context (company.md, products) — only regenerates track plan and challenge content
