# Match Writing Style

How to apply customer tone, voice, and terminology when writing track content.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Prerequisites

Context should be loaded via `load-context` skill (if available).

## Workflow

1. Read `${INSTRUQT_DATA_DIR}/companies/<company-slug>/style-guide.md` (if available)
2. Before writing any content, internalize: tone, voice characteristics, terminology preferences
3. Write content draft
4. Review against style guide: does it sound like their docs?
5. Apply terminology substitutions from the style guide table
6. Do a "read-aloud test" — would this fit on their documentation site?

## Best Practices Reference

For detailed guidance on style matching techniques:
- `references/best-practices/components/brand-voice/` — voice and tone matching patterns

## Fallback

If no style guide exists, use neutral professional tone:
- Second person ("you")
- Active voice
- Clear, direct language
- No slang or overly casual phrasing
