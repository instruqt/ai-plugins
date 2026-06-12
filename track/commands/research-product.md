# Research Product Command

You are helping a user research and document a single product for Instruqt track creation.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:research-product <product-name> [url]` — research a specific product
- `/track:research-product` (no argument) — list documented products or ask which to add

## Prerequisites

`CLAUDE_PLUGIN_DATA` is provided by the plugin framework.

## Context Directory

Product context is stored in `${CLAUDE_PLUGIN_DATA}/products/<company-slug>/<product-slug>/`:
```
${CLAUDE_PLUGIN_DATA}/products/
  <company-slug>/
    <product-slug>/
    product.md
    manifest.json
    sitemaps/
    website/
```

## Workflow

### Step 1: Check Context

1. List existing products in `${CLAUDE_PLUGIN_DATA}/products/`
2. If product already documented, ask: update or add a different one?
3. If company context exists in `${CLAUDE_PLUGIN_DATA}/companies/`, check for scraped website content that may be relevant to this product

### Step 2: Scrape Product Docs (if URL provided)

If a product-specific URL was provided, the command owns scraping. Read `${CLAUDE_PLUGIN_ROOT}/skills/scrape-website/SKILL.md` and follow its mode detection.

Set the scraper output directory before running any scraper commands:
```bash
export SCRAPER_DATA_DIR="${CLAUDE_PLUGIN_DATA}/products/<company-slug>"
```

1. Create output directory: `mkdir -p ${CLAUDE_PLUGIN_DATA}/products/<company-slug>/<product-slug>`
2. Discover the sitemap for the product URL (product-slug: `<slug>`, url: `<product-url>`)
3. If sitemap found: filter for product-relevant URLs using the skill's select/deselect, then scrape all selected URLs
4. If no sitemap found: scrape the single product URL directly using the skill's single-URL scrape operation

If no URL provided, skip this step — the researcher will use existing content (from company website scrapes or other sources).

### Step 3: Spawn Product Researcher

All content is local. The agent does analysis only.

```
Agent(
  subagent_type="track:product-researcher",
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/product-researcher.md for your full instructions.

  Product: <product-name>
  Product directory: ${CLAUDE_PLUGIN_DATA}/products/<company-slug>/<product-slug>/

  Primary sources (read these first):
  <list the primary_files if available>

  Additional context:
  <if company context exists, point to relevant website content in ${CLAUDE_PLUGIN_DATA}/companies/<company-slug>/website/>

  Skills to load:
  - ${CLAUDE_PLUGIN_ROOT}/skills/research-product/SKILL.md

  Templates to use:
  - ${CLAUDE_PLUGIN_ROOT}/templates/product.md

  Analyze the scraped content for this product and return the full product document."
)
```

The agent returns the full product document as its response.

### Step 4: Write Product File

1. Create directory: `mkdir -p ${CLAUDE_PLUGIN_DATA}/products/<company-slug>/<product-slug>`
2. Write the agent's response to `${CLAUDE_PLUGIN_DATA}/products/<company-slug>/<product-slug>/product.md`

### Step 5: Present Results

1. Summarize key findings from the written file
2. Ask if anything needs adjustment

## Important Notes

- The command owns all scraping — the agent only analyzes local files
- Products are independent of companies — no company research required
- If company context exists, it can be used as additional source material but is not required
- Product slug: lowercase, replace spaces with hyphens
- If no URL provided, the researcher uses existing content (company website scrapes or previously scraped product content)
- Multiple products can be documented by running this command repeatedly
