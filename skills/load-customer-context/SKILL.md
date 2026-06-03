# Load Customer Context

How to find and load customer context files for track generation.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Paths

These should already be resolved by the calling command:
- `${TRACK_RESEARCH_DIR}`: global research data. Default `~/.instruqt/companies`
- `${TRACK_OUTPUT_DIR}`: local track state directory (absolute path). Default `~/.instruqt/tracks`

## Context Location

Customer context is stored globally in `${TRACK_RESEARCH_DIR}/<company-slug>/`:

```
${TRACK_RESEARCH_DIR}/
  <company-slug>/
    company.md            # Company info (mission, values, industry)
    style-guide.md        # Writing style (tone, voice, terminology)
    products/
      <product-slug>.md   # Product details (one per product)
    website/              # Raw scraped HTML (not used directly in generation)
```

## Loading Order

1. Read `company.md` — required. If missing, direct user to run `/track:research-company`
2. Read `style-guide.md` — recommended. Generation works without it but quality suffers
3. Determine which products are relevant (see Product Filtering below)
4. Read ONLY the relevant product files. **Do not load all product files.**

## Product Filtering

Only load product files that the track actually covers. The filter source depends on the pipeline stage:

| Stage | Filter source | How to determine relevant products |
|-------|--------------|-------------------------------------|
| `plan-track` | User's command arguments | The user specifies a topic or product name. Match against `products/` filenames. If ambiguous, list available products and ask. |
| `plan-challenge` | Track plan | Read `${TRACK_OUTPUT_DIR}/.instruqt/plan.md`, extract the "Products Covered" section. Load only those product files. |
| `generate-challenge` | Track plan | Same as plan-challenge. |
| `review-*` | Track plan | Same as plan-challenge. |

**Hard rule:** Never load product files not listed in the track plan's "Products Covered" section (except during `plan-track` where the track plan does not yet exist). A customer may have 15+ product files — loading all of them wastes context and introduces irrelevant terminology.

## Using Context

- Company name: use exact capitalization from company.md
- Terminology: substitute generic terms with customer-preferred terms from style-guide.md
- Product references: use accurate feature names and descriptions from product files
- Tone: match the formality/casualness documented in style-guide.md

## Fallback Behavior

If context files are missing:
- No company.md: Cannot proceed — customer context is required
- No style-guide.md: Use neutral professional tone, flag to user
- No product files: Can generate generic content, flag to user
