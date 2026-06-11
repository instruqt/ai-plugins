# No Accidental Future-Challenge Spoilers

Examples and demonstrations in earlier challenges must not *accidentally* complete steps in later challenges. This is one of the most common and damaging mistakes.

Note: intentional reinforcement is different from accidental spoilers. A track may deliberately have learners repeat a skill (e.g., staging files) in a more complex context later. That is fine when clearly framed as deliberate practice. The problem is when an offhand example pre-completes a future challenge step without the learner realizing it.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Multiple examples in earlier challenges accidentally complete steps from later challenges. Learners who follow instructions will skip learning objectives without realizing it. |
| 2 | Below Standard | One or two earlier examples create files or state that partially or fully satisfy later check script conditions. The overlap is not systematic but causes real problems. |
| 3 | Adequate | No examples accidentally complete future challenge steps, but some examples use files or concepts closely associated with future learning objectives, creating potential confusion. |
| 4 | Good | Examples use neutral files and state that do not accidentally overlap with future challenge steps. Intentional reinforcement of earlier skills is clearly framed. One minor edge case may exist. |
| 5 | Excellent | Every example is crafted with awareness of the full track sequence. No accidental spoilers exist. Intentional skill reinforcement is clearly signposted so learners understand they are practicing a known skill in a new context. |

**The problem:**

In Challenge 3 "Staging Changes":
```bash
echo "def add(a, b): return a + b" > math_utils.py
echo "*.pyc" > .gitignore      # Creates .gitignore!
git add math_utils.py .gitignore
```

Later in Challenge 5 "Ignoring Files":
```
Step: Create a .gitignore file...
```

The user already did this if they copy-pasted the earlier example. The learning opportunity is lost.

**The fix:**

Use files that aren't learning objectives in examples:
```bash
echo "def add(a, b): return a + b" > math_utils.py
echo "DEBUG = True" > config.py     # Neutral file, not a future challenge step
git add math_utils.py config.py
```

**Common pitfalls:**
- Using `.gitignore` in staging examples (when gitignore is taught later)
- Creating branches in earlier examples (when branches are taught later)
- Showing merge commands before the merge challenge
- Including remote operations before teaching remotes

### Review Checklist for Accidental Spoilers

Before including an example:
1. Does this example *accidentally* do something a later challenge step asks for?
2. Does it use files that are learning objectives in future challenges?
3. Does it demonstrate concepts before they're formally taught?
4. If the overlap is intentional reinforcement, is it clearly framed as deliberate practice?

## What to Watch For

- Examples in challenge N that create files/state needed by steps in challenge N+1
