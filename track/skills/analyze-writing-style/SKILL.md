# Analyze Writing Style

Orchestrates writing style extraction from scraped documentation.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Workflow

1. Read scraped HTML files — prefer documentation and tutorial pages
2. Select 2-3 pages that best represent the company's writing style
3. Analyze across five dimensions (see knowledge base for detailed guidance):
   - Tone (formality, technical depth, personality)
   - Voice (pronouns, sentence structure, paragraph length)
   - Terminology (preferred terms, product-specific language)
   - Patterns (common phrases, formatting conventions)
   - Anti-patterns (terms and patterns they avoid)
4. Extract 2-3 example excerpts that exemplify their style
5. Read template: `templates/style-guide.md`
6. Write output: `${CLAUDE_PLUGIN_DATA}/companies/<company-slug>/style-guide.md`

## Best Practices Reference

For detailed guidance on style analysis dimensions and techniques, read:
- `references/best-practices/components/brand-voice/` — what to look for in voice and tone

## Quality Checklist

- [ ] All template sections completed
- [ ] Concrete examples included (not just descriptions)
- [ ] Terminology table has specific substitutions
- [ ] Anti-patterns documented with alternatives
- [ ] Sources list the analyzed pages
