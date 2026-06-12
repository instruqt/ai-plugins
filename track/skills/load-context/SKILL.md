# Load Context

Discover available context and selectively read what's needed for the current task.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Paths

These should already be resolved by the calling command:
- `${CLAUDE_PLUGIN_DATA}`: base data directory (provided by the plugin framework)
- `${TRACK_OUTPUT_DIR}`: local track state directory (absolute path)

Derived paths:
- Companies: `${CLAUDE_PLUGIN_DATA}/companies/<company-slug>/`
- Products: `${CLAUDE_PLUGIN_DATA}/products/<company-slug>/<product-slug>/`

## Two-phase approach: discover, then read

### Phase 1: Discovery (always run, cheap)

Check which context exists without reading file contents. This is just `ls` / file-exists checks:

- **Company**: does `${CLAUDE_PLUGIN_DATA}/companies/<company-slug>/company.md` exist? `style-guide.md`?
- **Products**: which product directories exist under `${CLAUDE_PLUGIN_DATA}/products/<company-slug>/`?
- **Track**: does `track.yml`, `config.yml`, `.instruqt/plan.md` exist in the working directory?
- **Challenge plans**: which `.instruqt/<NN-slug>/plan.md` files exist?
- **Generated content**: which `<NN-slug>/assignment.md` files exist?
- **README**: does `README.md` exist?

Report the discovery results to the calling command. Every command runs discovery so it knows what's available.

### Phase 2: Selective reading (only what the task needs)

Each command type needs different context. Only read what's relevant:

| Command | Read company | Read style guide | Read products | Read track plan | Read challenge plans | Read generated content | Read README |
|---------|-------------|-----------------|--------------|----------------|---------------------|----------------------|-------------|
| `research-company` | existing (for update) | existing (for update) | - | - | - | - | - |
| `research-product` | for additional sources | - | existing (for update) | - | - | - | - |
| `plan-track` | yes | yes | relevant only | existing (for extend) | - | existing (for extend) | existing |
| `plan-challenge` | - | - | relevant only | yes | prior challenges | prior challenges | - |
| `generate-challenge` | yes | yes | relevant only | yes | this + prior | prior challenges | - |
| `review-track-plan` | yes | yes | relevant only | yes | - | - | - |
| `review-challenge-plan` | - | - | relevant only | yes | this challenge | - | - |
| `review-challenge` | yes | yes | relevant only | yes | this challenge | this challenge | - |
| `review-track` | yes | yes | relevant only | yes | all | all | yes |
| `generate-all-challenges` | yes | yes | relevant only | yes | all | all | - |
| `generate-readme` | - | - | - | yes | - | all | existing |

**`-`** means don't read it even if it exists — it's not useful for this task.

## Product filtering

Only load product files that the track actually covers:

| Stage | How to determine relevant products |
|-------|-------------------------------------|
| `plan-track` | User's command arguments or topic. Match against available product directories. If ambiguous, list available products and ask. |
| `plan-challenge` | Track plan "Products Covered" section. Load only those. |
| `generate-challenge` | Same as plan-challenge. |
| `review-*` | Same as plan-challenge. |

**Hard rule:** Never load all product files. A user may have 15+ products — loading all wastes context and introduces irrelevant terminology.

## Context locations

### Company context (`${CLAUDE_PLUGIN_DATA}/companies/<company-slug>/`)

```
company.md            # Company info (mission, values, industry)
style-guide.md        # Writing style (tone, voice, terminology)
manifest.json         # Scraping metadata, domain relationships
website/              # Scraped website content
```

### Product context (`${CLAUDE_PLUGIN_DATA}/products/<company-slug>/<product-slug>/`)

```
product.md            # Product details (features, architecture, use cases)
manifest.json         # Product-specific scraping metadata
website/              # Product-specific scraped content
```

### Track context (current working directory or `${TRACK_OUTPUT_DIR}/`)

```
track.yml             # Track metadata
config.yml            # Sandbox configuration
.instruqt/plan.md     # Track plan (objectives, audience, challenge outline)
```

### Challenge context

```
.instruqt/<NN-slug>/plan.md     # Challenge plan
<NN-slug>/assignment.md         # Generated assignment content
<NN-slug>/setup-*               # Lifecycle scripts
<NN-slug>/check-*
<NN-slug>/solve-*
<NN-slug>/cleanup-*
```

## Using context

- Company name: use exact capitalization from company.md
- Terminology: substitute generic terms with company-preferred terms from style-guide.md
- Product references: use accurate feature names and descriptions from product files
- Tone: match the formality/casualness documented in style-guide.md
- Existing content: maintain consistency with what's already in the track

## Fallback behavior

Every context type is optional. When context is missing:
- No company.md: use neutral professional tone, no company-specific branding
- No style-guide.md: use neutral professional tone
- No product files: rely on topic research and user input for technical accuracy
- No track plan: the calling command handles this (may start planning interactively)
- No existing content: generate from scratch

If company or product context would significantly improve quality, suggest the user run the relevant research command — but do not block.
