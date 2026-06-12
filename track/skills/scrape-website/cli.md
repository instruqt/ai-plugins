# Scrape Website (CLI)

Discover and download relevant pages from a company website using the scraper CLI.

## Prerequisite

```bash
which scraper
```

If not found, install it automatically:

1. Detect the platform:
```bash
OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m)
case "$ARCH" in
  x86_64)  ARCH="amd64" ;;
  aarch64|arm64) ARCH="arm64" ;;
esac
BINARY="scraper-${OS}-${ARCH}"
```

2. Download the latest release to the plugin's bin directory:
```bash
INSTALL_DIR="${CLAUDE_PLUGIN_ROOT}/bin"
TAG=$(gh release list --repo instruqt/ai-plugins --limit 1 --json tagName --jq '.[0].tagName')
gh release download "$TAG" --repo instruqt/ai-plugins --pattern "${BINARY}" --dir "$INSTALL_DIR" --clobber
mv "${INSTALL_DIR}/${BINARY}" "${INSTALL_DIR}/scraper"
chmod +x "${INSTALL_DIR}/scraper"
```

3. Verify it works:
```bash
scraper --version
```

If the download fails (e.g. no release exists for this platform), stop and tell the user: "Could not auto-install scraper CLI — no release found for ${OS}/${ARCH}. Download the binary for your platform from https://github.com/instruqt/ai-plugins/releases, place it at `${CLAUDE_PLUGIN_ROOT}/bin/scraper`, and make it executable (`chmod +x`)."

## Commands

### scraper sitemap discover

Discovers sitemap URLs and saves them for later use.

```bash
scraper sitemap discover <company-slug> <url>
```

### scraper sitemap list

Lists saved sitemaps with entry counts.

```bash
scraper sitemap list <company-slug>
```

### scraper sitemap get

Shows sitemap entries for a domain. Supports filtering and pagination.

```bash
scraper sitemap get <company-slug> <domain> [--filter <pattern>] [--selected] [--offset N] [--limit N] [--lastmod-after <date>] [--changefreq <freq>]
```

### scraper sitemap update

Selects or deselects entries by URL patterns. Patterns support `*` wildcard.

```bash
scraper sitemap update <company-slug> <domain> --select "*/docs/*" --select "*/blog/*"
scraper sitemap update <company-slug> <domain> --deselect "*/careers/*"
```

Use `--select "*"` to select all entries.

### scraper scrape sitemap

Scrapes all selected URLs from a saved sitemap.

```bash
scraper scrape sitemap <company-slug> <domain> [--force] [--spa]
```

### scraper scrape url

Scrapes a single URL.

```bash
scraper scrape url <company-slug> <url> [--force] [--spa]
```

### scraper links

Aggregates external links from scraped pages. Run AFTER scraping.

```bash
scraper links <company-slug> <domain> [--threshold N]
```

### scraper validate

Validates the scrape output structure.

```bash
scraper validate <company-slug>
```

## Workflow

### Step 1: Discover sitemap

```bash
scraper sitemap discover <company-slug> https://<domain>
```

### Check for llms.txt

After discovering the sitemap, check if llms.txt was detected:

```bash
scraper sitemap list <company-slug>
```

If the sitemap has `llmsTxt` or `llmsFullTxt` set, skip Steps 2-3 (review and select) and go directly to Step 4 (scrape). The scraper uses llms.txt automatically and ignores URL selection.

### Step 2: Review entries

```bash
scraper sitemap get <company-slug> <domain>
```

### Step 3: Select URLs to scrape

Keep: homepage, about, products, solutions, features, pricing, blog, docs, use cases, customer stories.
Exclude: careers, legal, privacy, login, signup, press releases, media kits, localized variants.

```bash
scraper sitemap update <company-slug> <domain> --select "*/docs/*" --select "*/blog/*" --select "*/products/*"
scraper sitemap update <company-slug> <domain> --deselect "*/careers/*" --deselect "*/legal/*"
```

### Step 4: Scrape selected URLs

```bash
scraper scrape sitemap <company-slug> <domain>
```

Use `--force` to re-scrape existing pages. Use `--spa` for SPA sites.

### Step 5: Aggregate external links

```bash
scraper links <company-slug> <domain>
```

High-count external domains are likely product or documentation sites worth scraping.

### Step 6: Scrape additional domains

For each additional domain, repeat steps 1-4. Use `scraper scrape url` for one-off pages.

### Step 7: Validate

```bash
scraper validate <company-slug>
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
