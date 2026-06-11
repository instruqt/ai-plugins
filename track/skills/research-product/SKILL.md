# Research Product

Orchestrates single-product research from scraped content and documentation.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Prerequisites

Run the `scrape-website` skill if product docs haven't been scraped yet.

## Workflow

1. Read scraped HTML files relevant to this product
2. If product URL provided, use `scrape-website` to fetch product-specific docs
3. Extract: overview, key features, use cases, technical details, integrations, documentation links
4. Focus on track-creation-specific info: setup requirements, core workflows, concepts to teach
5. Read template: `templates/product.md`
6. Write output: `${INSTRUQT_DATA_DIR}/products/<company-slug>/<product-slug>/product.md`

## Product Slug

Derive from product name: lowercase, replace spaces with hyphens.
Example: "Acme CLI" -> "acme-cli"

## Quality Checklist

- [ ] All template sections filled with actionable details
- [ ] Technical details are current and accurate
- [ ] Documentation links are included
- [ ] Sources section lists analyzed pages
- [ ] Info is specific enough to inform track content (not marketing fluff)
