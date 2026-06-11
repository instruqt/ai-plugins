# Engagement Validation

Deciding when and how to add engagement-validation patterns that detect whether learners are genuinely working through content versus clicking through rapidly or gaming the system.

## Decision Criteria

### Add engagement validation when:

- The track is part of a **certification or graded program** where progress must be attributable.
- The publisher needs to **detect cheating** (rapid clicking, scripted solves, idle sessions).
- **Compliance requirements** mandate proof of learner engagement (regulated training, continuing education).
- The track awards **credits, badges, or completion certificates** with real-world value.

### Skip engagement validation when:

- The track is a **demo or workshop** where the goal is product exploration.
- The audience is **self-motivated** (internal engineering teams, conference attendees).
- Adding friction would **frustrate genuine learners** more than it deters bad actors.
- The track is a **prototype or draft** still under iteration.

## Validation Patterns

### Shell history forensics

Check that specific commands were actually executed by examining shell history:

```bash
# Check mongosh history for a specific query
grep -q "db.collection.find" /root/.mongosh/mongosh_history || fail-message "Run the find query before continuing."
```

Best for: CLI-based tracks where the learner must execute specific commands.

### Anti-rapid-click counters

Track time spent per challenge and flag suspiciously fast completions:

```bash
# Record challenge start time
date +%s > /tmp/.challenge_start_time

# In check script: verify minimum time elapsed
START=$(cat /tmp/.challenge_start_time 2>/dev/null || echo 0)
NOW=$(date +%s)
ELAPSED=$((NOW - START))
if [ "$ELAPSED" -lt 30 ]; then
  fail-message "Please take time to read through the material before continuing."
fi
```

Best for: content-heavy challenges where meaningful engagement requires reading time.

### Marker files

Require specific actions that produce verifiable artifacts:

```bash
# Check that a config file was actually modified (not just default)
if diff -q /workspace/config.yaml /workspace/.config.yaml.orig > /dev/null 2>&1; then
  fail-message "Edit the configuration file before continuing."
fi
```

Best for: tracks where the learner must create or modify specific files.

### Cron-based pause tracker

Monitor idle time to detect sessions left open without interaction:

```bash
# In setup: start an activity monitor
cat > /etc/cron.d/activity-monitor <<'EOF'
* * * * * root if [ -f /tmp/.last_activity ]; then echo "$(date +\%s)" >> /var/log/activity.log; fi
EOF

# Learner activity updates the timestamp (via shell PROMPT_COMMAND or similar)
export PROMPT_COMMAND='touch /tmp/.last_activity'
```

Best for: timed assessments where idle time should not count toward completion.

## Trade-offs

| Factor | With Engagement Validation | Without |
|--------|---------------------------|---------|
| Learner friction | Higher -- false positives frustrate genuine learners | None |
| Cheating deterrence | Effective against casual gaming | No deterrence |
| Implementation complexity | Moderate to high | None |
| Maintenance burden | Validation logic must stay in sync with content | None |
| False positive risk | Real -- slow readers, accessibility needs, network latency | N/A |
| Check script complexity | Significantly increased | Standard checks only |

## Recommendation

**Avoid engagement validation on demo and workshop tracks.** The friction it adds is not worth the benefit when learners are self-motivated and no credential is at stake.

**Use it judiciously on certification/graded tracks.** Start with the lightest-touch pattern that meets the requirement (usually shell history forensics or marker files). Add heavier patterns (time tracking, idle detection) only when lighter approaches prove insufficient.

**Always provide clear fail messages.** When validation blocks a learner, the message should explain exactly what action is expected -- never a generic "try again" that leaves the learner guessing.

**Test with real users before publishing.** Engagement validation that makes sense to the author can be baffling to learners. False positives on legitimate slow readers, learners with accessibility needs, or learners on high-latency connections will generate support tickets.
