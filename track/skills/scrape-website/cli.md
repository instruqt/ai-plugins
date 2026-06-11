# Scrape Website (CLI)

Discover and download relevant pages from a company website using the labsmith CLI.

## Prerequisite

```bash
which labsmith
```

If not found, stop and tell the user: "labsmith CLI is not installed. Cannot proceed with scraping."

## Commands

### labsmith sitemap discover

Discovers sitemap URLs and saves them for later use.

```bash
labsmith sitemap discover <company-slug> <url>
```

### labsmith sitemap list

Lists saved sitemaps with entry counts.

```bash
labsmith sitemap list <company-slug>
```

### labsmith sitemap get

Shows sitemap entries for a domain. Supports filtering and pagination.

```bash
labsmith sitemap get <company-slug> <domain> [--filter <pattern>] [--selected] [--offset N] [--limit N] [--lastmod-after <date>] [--changefreq <freq>]
```

### labsmith sitemap update

Selects or deselects entries by URL patterns. Patterns support `*` wildcard.

```bash
labsmith sitemap update <company-slug> <domain> --select "*/docs/*" "*/blog/*"
labsmith sitemap update <company-slug> <domain> --deselect "*/careers/*"
```

Use `--select "*"` to select all entries.

### labsmith scrape sitemap

Scrapes all selected URLs from a saved sitemap.

```bash
labsmith scrape sitemap <company-slug> <domain> [--force] [--spa]
```

### labsmith scrape url

Scrapes a single URL.

```bash
labsmith scrape url <company-slug> <url> [--force] [--spa]
```

### labsmith links

Aggregates external links from scraped pages. Run AFTER scraping.

```bash
labsmith links <company-slug> <domain> [--threshold N]
```

### labsmith validate

Validates the scrape output structure.

```bash
labsmith validate <company-slug>
```

## Workflow

### Step 1: Discover sitemap

```bash
labsmith sitemap discover <company-slug> https://<domain>
```

### Step 2: Review entries

```bash
labsmith sitemap get <company-slug> <domain>
```

### Step 3: Select URLs to scrape

Keep: homepage, about, products, solutions, features, pricing, blog, docs, use cases, customer stories.
Exclude: careers, legal, privacy, login, signup, press releases, media kits, localized variants.

```bash
labsmith sitemap update <company-slug> <domain> --select "*/docs/*" "*/blog/*" "*/products/*"
labsmith sitemap update <company-slug> <domain> --deselect "*/careers/*" "*/legal/*"
```

### Step 4: Scrape selected URLs

```bash
labsmith scrape sitemap <company-slug> <domain>
```

Use `--force` to re-scrape existing pages. Use `--spa` for SPA sites.

### Step 5: Aggregate external links

```bash
labsmith links <company-slug> <domain>
```

High-count external domains are likely product or documentation sites worth scraping.

### Step 6: Scrape additional domains

For each additional domain, repeat steps 1-4. Use `labsmith scrape url` for one-off pages.

### Step 7: Validate

```bash
labsmith validate <company-slug>
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
