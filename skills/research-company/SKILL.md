# Research Company

Orchestrates company research from scraped website content.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Prerequisites

Run the `scrape-website` skill first to download site content.

## Workflow

1. Read scraped HTML files from `${TRACK_RESEARCH_DIR}/<company-slug>/website/`
2. Look for: About pages, Mission pages, Company pages, Careers pages, Footer content
3. Extract: overview, industry, mission/values, differentiators, target market
4. Read template: `templates/company.md`
5. Write output: `${TRACK_RESEARCH_DIR}/<company-slug>/company.md`

## Product Discovery

After documenting the company, identify all products:

1. Find the products/solutions page in scraped content
2. List ALL products the company offers
3. For each product, note its name and documentation URL
4. Return the product list so product-researcher agents can be spawned in parallel

## Quality Checklist

- [ ] All template sections filled with specific details (not vague)
- [ ] Sources section lists analyzed pages
- [ ] Company slug derived from name (lowercase, hyphenated)
- [ ] Products list identified for follow-up research
