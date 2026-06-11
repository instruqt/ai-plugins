# Research Company Command

You are helping a user research their company and writing style for Instruqt track creation.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

Arguments use labeled `key:value` syntax:

- `/track:research-company url:<company-url> slug:<company-slug>` — research the company with a known slug
- `/track:research-company url:<company-url>` — research the company, derive slug from URL
- `/track:research-company slug:<company-slug>` — load existing context for editing (no scraping)
- `/track:research-company` (no argument) — show existing context status or ask for URL

## Prerequisites

Resolve `INSTRUQT_DATA_DIR`: if set use it, otherwise default to `~/.instruqt`.

## Context Directory

Company context is stored in `${INSTRUQT_DATA_DIR}/companies/<company-slug>/`:
```
${INSTRUQT_DATA_DIR}/companies/
  <company-slug>/
    company.md
    style-guide.md
    manifest.json
    sitemaps/
      <domain>.json
    website/
      <domain>/
        index.json
        <path>.md
```

Product context is stored separately in `${INSTRUQT_DATA_DIR}/products/<company-slug>/<product-slug>/`:
```
${INSTRUQT_DATA_DIR}/products/
  <product-slug>/
    product.md
    manifest.json
    sitemaps/
    website/
```

## Workflow

### Step 1: Check Existing Context

1. List `${INSTRUQT_DATA_DIR}/` to see existing company directories
2. If a company slug is provided or can be inferred, check existing files
3. Summarize what's already documented

### Step 2: Determine What to Do

**If `url:` provided:**
- If `slug:` is also provided, use it as-is (do not derive)
- Otherwise, derive company slug from URL domain (lowercase, replace dots/spaces with hyphens)
- Proceed to scraping

**If `slug:` only (no `url:`):**
- Load existing context from `${INSTRUQT_DATA_DIR}/companies/<company-slug>/`
- If the user provided instructions (prefixed message), apply those edits to the existing files (e.g. update company.md, style-guide.md, or products)
- Do not scrape — work only with existing local content
- After editing, skip to Step 14 (Present Results)

**If no argument:**
- If no context exists: ask for the company website URL
- If context exists: show summary, ask what to add/update

### Step 3: Scrape Primary Domain

Read `${CLAUDE_PLUGIN_ROOT}/skills/scrape-website/SKILL.md` and follow its mode detection. Use the loaded skill for all scraping operations in this command.

1. Create output directory: `mkdir -p ${INSTRUQT_DATA_DIR}/<company-slug>`
2. Discover the sitemap for the primary domain (company-slug: `<slug>`, url: `<url>`)
3. Get sitemap entries to review the available URLs
4. Filter URLs — keep company info, products, docs, blog. Exclude careers, legal, login, duplicates.
   Select matching patterns and deselect excluded patterns using the skill's sitemap update operation.
5. Scrape all selected URLs from the sitemap

### Step 4: Discover and Scrape External Domains

After scraping, aggregate external links for the primary domain using the skill.

Read `${INSTRUQT_DATA_DIR}/companies/<company-slug>/manifest.json` and check the `external_urls` section. Domains are listed with frequency counts.

1. Drop domains with fewer than 3 mentions (already filtered by threshold)
2. Any domain with 20+ mentions: ALWAYS ask the user about it — high frequency means it could be an acquisition or owned product, regardless of how it looks
3. Domains with 3-19 mentions: use your judgment — skip if obviously third-party (grafana.com, kubernetes.io), ask if ambiguous

Present the domains you're asking about:

> I found these external domains frequently linked from the site. Are any of these company-owned products or acquisitions I should also scrape?
> - domain-a.com (47 mentions)
> - domain-b.com (23 mentions)

Only skip without asking if ALL domains above 3 mentions are obviously third-party AND none have 20+ mentions.

For each domain the user confirms:

1. Discover the sitemap for the external domain
2. Review and filter entries (same criteria as primary domain)
3. Scrape all selected URLs from the sitemap

Keep track of confirmed external domains and their relationship to the company (e.g., "acquired product", "documentation site"). You'll need this for the manifest and agent prompts.

Skip if user says none are relevant.

### Step 5: Discover Documentation Sites

Read the `external_urls` from `manifest.json`. Look for domains that are likely documentation sites:

- Domains matching patterns like `docs.*`, `developer.*`, `learn.*`, `api.*`
- Domains with high frequency that aren't product sites or social media
- Paths containing `/docs/`, `/api/`, `/reference/`, `/guide/`, `/tutorial/`

For each documentation domain identified, discover its sitemap using the skill.

Do NOT scrape the doc pages — just store the sitemaps. The challenge implementer will fetch specific pages on demand during generation.

### Step 6: Update Manifest

Update `${INSTRUQT_DATA_DIR}/companies/<company-slug>/manifest.json` to add domain relationships and documentation references. The manifest already has `domains` and `external_urls` from the scraping steps. Add:

1. Set `relationship` for each domain in `domains` (e.g., "primary", "acquired product", "documentation site")
2. Add a `documentation` section for doc domains referencing their sitemaps:

```json
{
  "domains": {
    "<primary-domain>": {
      "relationship": "primary",
      "directory": "website/<primary-domain>"
    },
    "<additional-domain>": {
      "relationship": "<acquired product | documentation site | etc.>",
      "directory": "website/<additional-domain>"
    }
  },
  "documentation": {
    "<docs-domain>": {
      "sitemap": "sitemaps/<docs-domain>.json",
      "url_count": "<number of URLs in sitemap>"
    }
  },
  "products": {},
  "external_urls": { ... }
}
```

### Step 7: Validate Output

Validate the scrape output using the skill. Fix any issues before proceeding.

### Step 8: Spawn Company Researcher

All content is now local and cleaned. The agent does analysis only — no web fetching.

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/company-researcher.md for your full instructions.

  Company slug: <slug>
  Primary domain: <primary-domain>
  Website content: ${INSTRUQT_DATA_DIR}/companies/<slug>/website/

  The website directory contains content from these domains:
  - <primary-domain> — THIS IS THE COMPANY. Focus company.md and style-guide.md on this domain's content.
  - <additional-domain-1> — <relationship, e.g. 'acquired product' or 'product documentation site'>
  - (list all additional domains that were scraped, or omit this section if only the primary domain was scraped)

  Start by reading the manifest: ${INSTRUQT_DATA_DIR}/companies/<slug>/manifest.json
  Then read the per-domain index.json files to find relevant pages by title.
  Focus your company and style analysis on files under website/<primary-domain>/.

  Skills to load:
  - ${CLAUDE_PLUGIN_ROOT}/skills/research-company/SKILL.md
  - ${CLAUDE_PLUGIN_ROOT}/skills/analyze-writing-style/SKILL.md

  Templates to use:
  - ${CLAUDE_PLUGIN_ROOT}/templates/company.md
  - ${CLAUDE_PLUGIN_ROOT}/templates/style-guide.md

  Analyze the scraped content, create company and style guide docs, and identify all products.
  Additional domains should appear as products, not as the company itself."
)
```

The agent returns three sections in its response: `=== COMPANY.MD ===`, `=== STYLE-GUIDE.MD ===`, `=== PRODUCTS ===`.

### Step 9: Write Company Context Files

Parse the company-researcher's response and write the files:

1. Extract content between `=== COMPANY.MD ===` and `=== STYLE-GUIDE.MD ===`
2. Write to `${INSTRUQT_DATA_DIR}/companies/<company-slug>/company.md`
3. Extract content between `=== STYLE-GUIDE.MD ===` and `=== PRODUCTS ===`
4. Write to `${INSTRUQT_DATA_DIR}/companies/<company-slug>/style-guide.md`
5. Parse the products list from the `=== PRODUCTS ===` section

### Step 10: Let User Select Products to Research

Use `AskUserQuestion` with `multiSelect: true` to let the user pick which products to research. Group products into categories of up to 4 options each (the tool's per-question limit).

**How to paginate:** Split the product list alphabetically into groups of 4 (the tool's per-question limit). Each group becomes one multi-select question. Use `"Which products should I research?"` as the question text for all groups, and `"1/N"`, `"2/N"`, etc. as the header. Use the product name as the label and a short description (from the company researcher output) as the description.

The user selects only what they need. They can always add more later with `/track:research-product`. If the user types a product name via "Other", include it in the research list.

### Step 11: Build Product Manifest

After the user selects products, build the `products` section of `manifest.json` for **only the selected products**:

1. Read the current manifest
2. Read per-domain `index.json` files to find pages by title
3. For each selected product, tag relevant files based on URL path patterns and page titles:
   - Product landing pages: paths or titles containing the product name
   - Blog posts: slugs containing product keywords
   - For additional domains: all files under that domain map to the product it represents
4. If documentation sitemaps exist, read them and match doc URLs to products by path patterns. Aim for 5-20 doc URLs per product.
5. Write the updated manifest with the products section populated

```json
"products": {
  "<product-slug>": {
    "name": "<Product Name>",
    "domain": "<domain where most content lives>",
    "primary_files": [
      "website/<domain>/<path>.md"
    ],
    "doc_urls": [
      "https://<docs-domain>/path/to/relevant/page"
    ]
  }
}
```

Only map files for selected products — skip everything else.

### Step 12: Spawn Product Researchers

For each confirmed product, spawn a `product-researcher` agent **in parallel**:

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/product-researcher.md for your full instructions.

  Company slug: <slug>
  Product: <product-name>

  Primary sources (read these first):
  <list the primary_files from manifest.json for this product>

  Additional context: read the per-domain index.json to find more pages by title.
  Only search within the domain directory listed above — do not scan other domains.

  Skills to load:
  - ${CLAUDE_PLUGIN_ROOT}/skills/research-product/SKILL.md

  Templates to use:
  - ${CLAUDE_PLUGIN_ROOT}/templates/product.md

  Analyze the scraped content for this product and return the full product document."
)
```

Each agent returns the full product document as its response.

### Step 13: Write Product Files

For each product-researcher response:

1. Create directory: `mkdir -p ${INSTRUQT_DATA_DIR}/products/<company-slug>/<product-slug>`
2. Write the response content to `${INSTRUQT_DATA_DIR}/products/<company-slug>/<product-slug>/product.md`

### Step 14: Present Results

After all files are written:
1. Summarize findings for the user:
   - Company overview
   - Writing style highlights
   - Products documented
2. Ask if anything needs adjustment

## Important Notes

- The command owns all scraping — agents only analyze local files
- Always check existing context first to avoid duplicate work
- Product researchers run in parallel for speed
- Store company context in `${INSTRUQT_DATA_DIR}/companies/<company-slug>/` (global across tracks)
- Store product context in `${INSTRUQT_DATA_DIR}/products/<company-slug>/<product-slug>/` (independent of company)
- Add a "Products" section to `company.md` referencing product slugs so the association is documented
- If a company-slug argument is provided, use it as-is; otherwise derive: lowercase, replace spaces with hyphens
- The manifest is the source of truth for mapping content to products — keep it updated
- Agents should read per-domain `index.json` to find relevant pages by title, not scan all files
