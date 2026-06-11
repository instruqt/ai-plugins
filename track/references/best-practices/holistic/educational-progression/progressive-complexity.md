# Progressive Complexity

Early challenges should provide more guidance. Later challenges should expect more independence.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | All challenges provide the same level of detail and hand-holding from start to finish. No visible progression in difficulty or independence. |
| 2 | Below Standard | Minor variation exists between challenges, but later challenges still spell out commands and steps that were already taught. Check script feedback remains equally specific throughout. |
| 3 | Adequate | There is a noticeable shift from guided to independent work, but it is inconsistent -- some later challenges still over-explain while some early challenges under-explain. |
| 4 | Good | Clear progression from detailed step-by-step early challenges to higher-level guidance in later challenges. Check script feedback for previously covered skills evolves from guiding questions to direct statements. New concepts still get full guidance regardless of challenge position. Minor inconsistencies remain. |
| 5 | Excellent | Deliberate, smooth gradient from full hand-holding to independent work. Early challenges give explicit commands and check script messages that guide with questions. Later challenges give goals, expect recall, and check script messages for previously covered skills state what is wrong directly. New concepts introduced in any challenge still receive guiding questions and detailed support. No regressions. |

**Early challenges (more hand-holding):**
- Detailed step-by-step instructions in assignment.md
- Explicit commands shown
- Check script failure messages guide with questions ("Is your user.name configured?")
- More assignment steps per challenge (smaller steps)

**Later challenges (more independence on previously taught skills):**
- Higher-level guidance ("push your changes to the remote")
- Users expected to remember commands from earlier challenges
- Check script failure messages for *previously covered* skills state the problem directly ("Changes are not staged.")
- Fewer assignment steps per challenge (larger steps)

Note: new concepts introduced in later challenges still deserve full guidance. The reduction applies to skills already taught, not to everything in later challenges.

**Example -- failure message progression for a previously taught skill:**

Challenge 2 -- staging introduced:
```
"No commits found. Have you staged and committed your changes? Use 'git add' to stage and 'git commit -m \"message\"' to commit."
```

Challenge 7 -- staging reappears:
```
"Your changes aren't staged yet."
```

## What to Watch For

- All challenges at the same difficulty level (no progression)
- Later challenges still providing the same level of hand-holding as challenge 1 for previously covered skills
- Large complexity jumps between challenges (too steep a curve)
- Check script failure messages at the same specificity level throughout the entire track
