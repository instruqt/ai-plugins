# Difficulty Progression

Evaluates whether the difficulty field on each challenge accurately reflects the content and whether difficulty ramps progressively across the track.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Difficulty fields missing entirely, or all challenges marked the same level regardless of actual content complexity |
| 2 | Below Standard | Difficulty fields present but inaccurate -- a complex multi-step challenge marked "basic" or a simple read-only challenge marked "advanced" |
| 3 | Adequate | Difficulty generally matches content but the progression is flat or inconsistent -- no deliberate ramp across the track |
| 4 | Good | Difficulty accurately reflects content; progressive ramp across challenges visible (production baseline) |
| 5 | Excellent | Difficulty labels help learners calibrate expectations; ramp is deliberate and tested; labels match both content complexity and cognitive load |

## Guidance

The `difficulty` field on each challenge accepts four values: `basic`, `intermediate`, `advanced`, or an empty string (unset). This field is visible to learners and helps them gauge effort before starting each challenge.

### What Each Level Means

- **basic** -- The learner follows explicit step-by-step instructions. Commands are provided. The challenge introduces a concept with minimal decision-making required.
- **intermediate** -- The learner must apply knowledge from prior challenges. Some steps require understanding rather than just copying commands. Minor troubleshooting may be needed.
- **advanced** -- The learner must synthesize multiple concepts, make decisions, or solve problems with minimal guidance. The assignment describes the goal, not the exact steps.
- **empty string** -- Difficulty not specified. Use sparingly; prefer explicit labeling.

### Progressive Ramp

A well-structured track starts with basic challenges and ramps to intermediate or advanced by the end:

Good -- progressive difficulty:

```yaml
challenges:
- slug: explore-environment
  difficulty: basic
- slug: configure-service
  difficulty: basic
- slug: deploy-application
  difficulty: intermediate
- slug: add-monitoring
  difficulty: intermediate
- slug: troubleshoot-issue
  difficulty: advanced
```

Good -- short track with gentle ramp:

```yaml
challenges:
- slug: introduction
  difficulty: basic
- slug: hands-on-exercise
  difficulty: intermediate
- slug: apply-independently
  difficulty: intermediate
```

Bad -- all the same difficulty:

```yaml
challenges:
- slug: explore-environment
  difficulty: intermediate
- slug: configure-service
  difficulty: intermediate
- slug: deploy-application
  difficulty: intermediate
- slug: troubleshoot-issue
  difficulty: intermediate
```

Bad -- difficulty decreases:

```yaml
challenges:
- slug: complex-setup
  difficulty: advanced
- slug: simple-verification
  difficulty: basic       # Easier than the previous -- ramp goes backward
```

Bad -- labels do not match content:

```yaml
# Challenge has step-by-step instructions with every command provided
- slug: deploy-application
  difficulty: advanced    # Should be basic or intermediate
```

### Matching Labels to Content Signals

| Signal | Likely Difficulty |
|--------|-------------------|
| Commands provided verbatim with run modifier | basic |
| Commands provided with copy modifier, some editing needed | basic to intermediate |
| Goal described, learner must figure out commands | intermediate to advanced |
| Troubleshooting or debugging required | advanced |
| Multiple concepts combined in a single task | advanced |
| Read-only exploration (no actions) | basic |

### Empty String vs Basic

Use an empty string only when the challenge does not fit the basic/intermediate/advanced spectrum (e.g., a pure information slide with no task). For any challenge that has a task, prefer an explicit label.

## What to Watch For

- All challenges set to the same difficulty -- this provides no useful signal to learners
- Difficulty labels that contradict the assignment content (advanced label on a copy-paste exercise)
- Difficulty that decreases across the track -- learners expect increasing challenge, not decreasing
- Empty difficulty fields on challenges that clearly have a task component
- Advanced difficulty on early challenges -- learners drop off if the first challenge feels too hard
- No basic challenges at all -- even experienced learners benefit from a gentle start to orient themselves
