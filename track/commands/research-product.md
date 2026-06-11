# Research Product Command

You are helping a user research and document a single product for Instruqt track creation.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:research-product <product-name> [url]` — research a specific product
- `/track:research-product` (no argument) — list documented products or ask which to add

## Prerequisites

Resolve `TRACK_RESEARCH_DIR`: if set use it, otherwise default to `~/.instruqt/companies`.

Run `/track:research-company` first to create company context.

## Context Directory

Product context is stored in `${TRACK_RESEARCH_DIR}/<company-slug>/products/<product-slug>.md`.

## Workflow

### Step 1: Check Context

1. Determine which company (from existing context or ask user)
2. List existing products in `${TRACK_RESEARCH_DIR}/<company-slug>/products/`
3. If product already documented, ask: update or add a different one?

### Step 2: Scrape Product Docs (if URL provided)

If a product-specific URL was provided, the command owns scraping. Read `${CLAUDE_PLUGIN_ROOT}/skills/scrape-website/SKILL.md` and follow its mode detection.

1. Discover the sitemap for the product URL (company-slug: `<slug>`, url: `<product-url>`)
2. If sitemap found: filter for product-relevant URLs using the skill's select/deselect, then scrape all selected URLs
3. If no sitemap found: scrape the single product URL directly using the skill's single-URL scrape operation

If no URL provided, skip this step — the researcher will use existing website/ content.

### Step 3: Spawn Product Researcher

All content is local. The agent does analysis only.

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/product-researcher.md for your full instructions.

  Company slug: <slug>
  Product: <product-name>
  Website content: ${TRACK_RESEARCH_DIR}/<slug>/website/

  Read the per-domain index.json files to find pages relevant to this product by title.

  Skills to load:
  - ${CLAUDE_PLUGIN_ROOT}/skills/research-product/SKILL.md

  Templates to use:
  - ${CLAUDE_PLUGIN_ROOT}/templates/product.md

  Analyze the scraped content for this product and return the full product document."
)
```

The agent returns the full product document as its response.

### Step 4: Write Product File

1. Create directory: `mkdir -p ${TRACK_RESEARCH_DIR}/<company-slug>/products`
2. Write the agent's response to `${TRACK_RESEARCH_DIR}/<company-slug>/products/<product-slug>.md`

### Step 5: Present Results

1. Summarize key findings from the written file
2. Ask if anything needs adjustment

## Important Notes

- The command owns all scraping — the agent only analyzes local files
- Company context must exist first (`/track:research-company`)
- Product slug: lowercase, replace spaces with hyphens
- If no URL provided, the researcher uses existing website/ content
- Multiple products can be documented by running this command repeatedly
