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
3. List `products/` directory
4. Read product files relevant to the current track (check `${TRACK_OUTPUT_DIR}/track.yml` for which products)

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
