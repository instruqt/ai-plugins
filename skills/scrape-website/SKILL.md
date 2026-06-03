# Scrape Website

Discover and download relevant pages from a company website for agent consumption.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

This skill supports two modes depending on the environment.

## Mode Detection

Check the environment variable `LABSMITH_SCRAPER_NATIVE_TOOLS`:

- If `LABSMITH_SCRAPER_NATIVE_TOOLS` is `true`: read `${CLAUDE_PLUGIN_ROOT}/skills/scrape-website/native.md` and follow its instructions.
- Otherwise: read `${CLAUDE_PLUGIN_ROOT}/skills/scrape-website/cli.md` and follow its instructions.
