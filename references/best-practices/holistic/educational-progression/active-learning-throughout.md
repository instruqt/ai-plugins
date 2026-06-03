# Active Learning Throughout

Users should be DOING things at every stage, not just reading. Check that concepts that are learning objectives are practiced by users, not pre-configured; users run commands themselves rather than just reading about them; and each challenge has meaningful interactive elements.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Multiple challenges consist entirely of reading with no interactive steps. Key learning objectives are pre-configured rather than practiced by the user. |
| 2 | Below Standard | Most challenges have steps, but some challenges are purely instructional. Some learning objectives are completed by setup scripts instead of by the user. |
| 3 | Adequate | Every challenge has interactive elements, but some steps are trivial (e.g., "click next") or key concepts are demonstrated rather than practiced. |
| 4 | Good | Every challenge has meaningful hands-on steps. Learning objectives are practiced by the user. Only infrastructure and non-taught settings are pre-configured. Occasional missed opportunity for practice. |
| 5 | Excellent | Every challenge has substantive interactive work. All learning objectives require user action. Pre-configuration is limited strictly to infrastructure and settings outside the learning scope. The ratio of doing to reading is high throughout. |

**Pre-configure only what's NOT being taught:**
- Infrastructure (bare repos, network, packages) -> pre-configure in setup scripts
- Settings not being taught (init.defaultBranch, core.editor) -> pre-configure in setup scripts
- Commands users should learn (git config user.name) -> users do this
- Configuration that's the learning objective -> users do this

## What to Watch For

- Active learning absent in certain challenges (all reading, no doing)
- Pre-configured settings in setup scripts that should be learning objectives
