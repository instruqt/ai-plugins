# Research Product Command

You are helping a user research and document a single product for Instruqt track creation. The command owns all scraping; the agent only analyzes local files.

## Arguments

- `/track:research-product <product-name> [url]` — research a specific product
- `/track:research-product` (no argument) — list documented products or ask which to add

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

Products are independent of companies — no company research is required, though existing company context can serve as additional source material.

## Workflow

### Step 1: Check Context

1. List existing products in `${CLAUDE_PLUGIN_DATA}/products/`. If the product is already documented, ask: update, or add a different one?
2. If company context exists in `${CLAUDE_PLUGIN_DATA}/companies/`, check for scraped website content relevant to this product.

### Step 2: Scrape Product Docs (if URL provided)

If a product URL was provided, read `${CLAUDE_PLUGIN_ROOT}/skills/scrape-website/SKILL.md` and follow its mode detection. Set the output directory first:

```bash
export SCRAPER_DATA_DIR="${CLAUDE_PLUGIN_DATA}/products/<company-slug>"
```

1. `mkdir -p ${CLAUDE_PLUGIN_DATA}/products/<company-slug>/<product-slug>`
2. Discover the sitemap for the product URL.
3. If a sitemap is found: filter for product-relevant URLs, then scrape the selected URLs. If not: scrape the single product URL directly.

If no URL was provided, skip this step — the researcher uses existing content (company website scrapes or previously scraped product content).

### Step 3: Spawn Product Researcher

```
Agent(
  subagent_type="track:product-researcher",
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/product-researcher.md for your full instructions.
  Product: <product-name>
  Product directory: ${CLAUDE_PLUGIN_DATA}/products/<company-slug>/<product-slug>/
  Primary sources (read first): <list primary_files if available>
  Additional context: <if company context exists, point to relevant website content under ${CLAUDE_PLUGIN_DATA}/companies/<company-slug>/website/>
  Analyze the scraped content for this product and return the full product document."
)
```

### Step 4: Write Product File

1. `mkdir -p ${CLAUDE_PLUGIN_DATA}/products/<company-slug>/<product-slug>`
2. Write the agent's response to `.../product.md`.

### Step 5: Present Results

Summarize key findings and ask if anything needs adjustment.

## Important Notes

- Product slug: lowercase, replace spaces with hyphens.
- Run repeatedly to document multiple products.
