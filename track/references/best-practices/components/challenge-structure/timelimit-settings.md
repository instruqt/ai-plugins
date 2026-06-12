# Timelimit Settings

Evaluates whether per-challenge timelimit values are appropriate for the challenge's complexity and whether they align with the track-level timing.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Explicit per-challenge limits that are far too short (learners time out mid-task) or absurdly long (hours for a 5-minute exercise) |
| 2 | Below Standard | Some challenges have explicit limits but they are inconsistent or arbitrary -- no relationship to actual task duration |
| 3 | Adequate | Per-challenge limits are reasonable but not tuned; some challenges could benefit from explicit limits but inherit the track default |
| 4 | Good | Appropriate limits that do not pressure or bore; timelimit: 0 used correctly to inherit from track where appropriate (production baseline) |
| 5 | Excellent | Timing tuned from real learner completion data; explicit limits on complex challenges give extra headroom; simple challenges use track inheritance |

## Guidance

Each challenge's `assignment.md` frontmatter can specify its own `timelimit` in seconds. The value `0` means "inherit from the track-level timelimit in track.yml" -- this is the most common setting and is appropriate for most challenges.

### When to Use timelimit: 0 (Inherit)

Use inheritance when all challenges in the track have roughly similar expected durations and the track-level timelimit provides sufficient total time.

Good -- uniform challenges inheriting track timing:

```yaml
# Per-challenge assignment.md timelimit fields, shown together for comparison:
- slug: explore-environment       # 01-explore-environment/assignment.md
  timelimit: 0
- slug: configure-service         # 02-configure-service/assignment.md
  timelimit: 0
- slug: verify-deployment         # 03-verify-deployment/assignment.md
  timelimit: 0
```

### When to Set Explicit Per-Challenge Limits

Set explicit limits when:

- A challenge involves a long-running process (compilation, deployment) that needs extra time
- A challenge is intentionally time-boxed (assessment, timed exercise)
- The track mixes very short and very long challenges

Good -- explicit limit for a complex challenge:

```yaml
# Per-challenge assignment.md timelimit fields:
- slug: explore-environment       # 01-explore-environment/assignment.md
  timelimit: 0
- slug: build-and-deploy          # 02-build-and-deploy/assignment.md
  timelimit: 1800                 # 30 minutes for this challenge specifically
- slug: verify-deployment         # 03-verify-deployment/assignment.md
  timelimit: 0
```

Good -- timed assessment challenge:

```yaml
# Per-challenge assignment.md timelimit fields:
- slug: guided-practice           # 01-guided-practice/assignment.md
  timelimit: 0
- slug: timed-assessment          # 02-timed-assessment/assignment.md
  timelimit: 600                  # 10 minutes to complete independently
```

Bad -- unnecessarily short limit:

```yaml
# 01-deploy-kubernetes-cluster/assignment.md
slug: deploy-kubernetes-cluster
timelimit: 120                    # 2 minutes for a K8s deployment -- impossible
```

Bad -- per-challenge limit longer than track timelimit:

```yaml
# track.yml timelimit: 3600 (1 hour)

# 01-configure-service/assignment.md
slug: configure-service
timelimit: 7200                   # 2 hours -- exceeds track total
```

### Difficulty Progression Reflected in Timing

Later challenges in a track tend to be more complex. If using explicit per-challenge limits, the progression should reflect increasing complexity:

Good -- timing increases with difficulty:

```yaml
# Per-challenge assignment.md timelimit fields, shown together for comparison:
- slug: introduction              # 01-introduction/assignment.md
  timelimit: 300                  # 5 min -- just reading and exploring
- slug: basic-configuration       # 02-basic-configuration/assignment.md
  timelimit: 600                  # 10 min -- simple guided task
- slug: advanced-deployment       # 03-advanced-deployment/assignment.md
  timelimit: 1200                 # 20 min -- complex multi-step task
- slug: troubleshooting           # 04-troubleshooting/assignment.md
  timelimit: 1800                 # 30 min -- independent problem-solving
```

Bad -- flat timing despite varying complexity:

```yaml
# Per-challenge assignment.md timelimit fields:
- slug: introduction              # 01-introduction/assignment.md
  timelimit: 900
- slug: basic-configuration       # 02-basic-configuration/assignment.md
  timelimit: 900
- slug: advanced-deployment       # 03-advanced-deployment/assignment.md
  timelimit: 900                  # Same time for a much harder challenge
- slug: troubleshooting           # 04-troubleshooting/assignment.md
  timelimit: 900
```

## What to Watch For

- Per-challenge timelimit set to a non-zero value without a clear reason -- prefer inheriting from the track
- Limits that do not account for sandbox provisioning time within the challenge (if applicable)
- Assessment challenges without explicit time limits -- timed assessments need deliberate limits
- Per-challenge limits that sum to more than the track timelimit -- the track limit is the hard ceiling
- All challenges set to the same explicit value instead of using timelimit: 0 -- adds maintenance burden with no benefit
