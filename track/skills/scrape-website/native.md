# Scrape Website (Native Tools)

Discover and download relevant pages from a company website using native scraper tools.

## Tools

All tools are prefixed with `mcp__scraper__` and auto-approved.

| Tool | Description |
|------|-------------|
| `mcp__scraper__discover_sitemap` | Discover sitemap URLs and save them |
| `mcp__scraper__list_sitemaps` | List saved sitemaps with entry counts |
| `mcp__scraper__get_sitemap` | Get sitemap entries with filtering and pagination |
| `mcp__scraper__update_sitemap` | Select/deselect entries by URL patterns |
| `mcp__scraper__scrape_sitemap` | Scrape all selected URLs from a sitemap |
| `mcp__scraper__scrape_url` | Scrape a single URL on demand |
| `mcp__scraper__aggregate_links` | Aggregate external links from scraped pages |
| `mcp__scraper__validate_scrape` | Validate scrape output directory |

## Workflow

### Step 1: Discover sitemap

```
mcp__scraper__discover_sitemap({ url: "https://<domain>", company_slug: "<slug>" })
```

### Step 2: Review entries

```
mcp__scraper__get_sitemap({ company_slug: "<slug>", domain: "<domain>" })
```

Use `url_pattern`, `limit`, `offset` to explore large sitemaps.

### Step 3: Select URLs to scrape

Keep: homepage, about, products, solutions, features, pricing, blog, docs, use cases, customer stories.
Exclude: careers, legal, privacy, login, signup, press releases, media kits, localized variants.

```
mcp__scraper__update_sitemap({ company_slug: "<slug>", domain: "<domain>", patterns: ["*/docs/*", "*/blog/*"], selected: true })
mcp__scraper__update_sitemap({ company_slug: "<slug>", domain: "<domain>", patterns: ["*/careers/*", "*/legal/*"], selected: false })
```

Use `["*"]` to select/deselect all entries.

### Step 4: Scrape selected URLs

```
mcp__scraper__scrape_sitemap({ company_slug: "<slug>", domain: "<domain>" })
```

Use `force: true` to re-scrape existing pages. Use `spa_mode: true` for SPA sites.

### Step 5: Aggregate external links

```
mcp__scraper__aggregate_links({ company_slug: "<slug>", domain: "<domain>" })
```

High-count external domains are likely product or documentation sites worth scraping.

### Step 6: Scrape additional domains

For each additional domain, repeat steps 1-4. Use `mcp__scraper__scrape_url` for one-off pages.

### Step 7: Validate

```
mcp__scraper__validate_scrape({ company_slug: "<slug>" })
```

## Output Structure

```
<output-dir>/
  manifest.json
  sitemaps/<domain>.json
  website/
    <domain>/
      index.json
      <path>.md
```

Files are markdown with links preserved. Same-domain links are rewritten to relative `.md` paths.
