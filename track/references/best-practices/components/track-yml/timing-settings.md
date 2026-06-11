# Timing Settings

Evaluates whether timelimit and idle_timeout are appropriately configured for the track's scope, audience, and delivery context.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No timing configured (defaults used), or timelimit so short that average learners cannot complete the track |
| 2 | Below Standard | Timing exists but is clearly mismatched -- a 30-minute timelimit on a 10-challenge workshop, or idle_timeout: 0 on a self-paced track |
| 3 | Adequate | Timing is reasonable but not tuned -- generic values that work but waste cloud resources or pressure learners unnecessarily |
| 4 | Good | Timing appropriate for track scope; idle_timeout prevents runaway costs; show_timer set correctly for the delivery context (production baseline) |
| 5 | Excellent | Timing tested with real learner pacing data; idle_timeout tuned to actual learner break patterns; documentation explains timing choices |

## Guidance

Three fields control track timing: `timelimit` (total track duration in seconds), `idle_timeout` (seconds of inactivity before sandbox teardown), and `show_timer` (whether the learner sees a countdown).

### Common Timing Patterns

These are established conventions used by major Instruqt customers:

| Use Case | timelimit | idle_timeout | show_timer | Notes |
|----------|-----------|--------------|------------|-------|
| Instructional lab (guided) | 5400 (90m) | 1800 (30m) | false | Short focused labs with instructor pacing |
| Short workshop | 7200 (2h) | 3600 (1h) | false | Self-paced but bounded |
| Standard HVD workshop | 28800 (8h) | 7200 (2h) | false | Full-day hands-on workshop |
| Long multi-product workshop | 28800 (8h) | 28800 (8h) | false | Extended sessions, no idle teardown |
| Live-presenter demo | varies | 0 | false | idle_timeout: 0 keeps sandbox alive indefinitely |

Good -- standard workshop timing:

```yaml
timelimit: 28800
idle_timeout: 7200
show_timer: false
```

Good -- focused instructional lab:

```yaml
timelimit: 5400
idle_timeout: 1800
show_timer: false
```

### idle_timeout: 0

Setting idle_timeout to 0 disables the idle teardown. This should only be used for live-presenter tracks where the presenter may pause for extended periods (Q&A, discussion). Never use idle_timeout: 0 on self-paced tracks -- abandoned sandboxes will run until timelimit expires, wasting cloud resources.

Bad -- idle_timeout: 0 on a self-paced track:

```yaml
# Self-paced lab with no idle teardown -- abandoned sandboxes run for 8 hours
timelimit: 28800
idle_timeout: 0
```

### show_timer

Most tracks set `show_timer: false`. Showing the timer creates anxiety and is only appropriate for certification-style assessments or timed challenges where the pressure is intentional.

Good -- timer hidden for learning-focused track:

```yaml
show_timer: false
```

Acceptable -- timer shown for assessment:

```yaml
show_timer: true
timelimit: 3600
```

Bad -- timer shown on a casual introductory lab:

```yaml
# Learner sees a countdown on a "Getting Started" track
show_timer: true
timelimit: 5400
```

### Calculating Appropriate Timelimit

A rough heuristic: estimate the average completion time, then multiply by 2-3x to account for slower learners, reading time, and troubleshooting. For a 5-challenge track where each challenge takes ~10 minutes, the expected completion is ~50 minutes, so a timelimit of 5400 (90m) gives comfortable headroom.

## What to Watch For

- timelimit too short for the number of challenges -- learners hit the wall before finishing
- timelimit excessively long for a simple track -- wastes cloud resources on abandoned sandboxes
- idle_timeout: 0 on anything that is not a live-presenter track
- show_timer: true on learning-focused (non-assessment) tracks
- Timing copied from another track without adjusting for different scope or complexity
- No idle_timeout specified -- relying on platform defaults that may not match the track's needs
