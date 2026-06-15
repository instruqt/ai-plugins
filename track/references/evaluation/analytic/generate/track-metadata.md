# Track Metadata

Evaluates the quality of track-level metadata in track.yml — title, description, tags, timing settings, and lab_config. Good metadata makes the track discoverable in the catalog, sets accurate expectations, and configures the player experience appropriately.

## Title and description quality

Evaluate whether the track title and description accurately represent the content and are optimized for catalog discovery.

Score 4: Title is descriptive and specific. Description accurately summarizes what the learner will do and learn. Both match the actual track content.
Score 5: Title is compelling and specific. Description includes the learning outcome, target audience hint, and key technologies. The learner can decide whether this track is for them from the catalog card alone.
Score 3: Title and description are accurate but generic — they could apply to many tracks on the same topic.
Score 2: Title or description is vague, aspirational, or does not match the actual content.
Score 1: Title or description is missing, uses placeholder text, or is misleading.

## Tags accuracy

Evaluate whether tags are accurate to the actual track content and follow the structured prefix convention.

Score 4: Tags include relevant product associations, persona targeting, and flag tags. All are accurate to the content. Prefix conventions are followed.
Score 5: Tags are precise, complete, and optimized for discoverability. Every tag reflects actual content. The set covers product, persona, and category without bloat or aspirational tags.
Score 3: Tags include the primary technology but miss persona or category, or contain minor inaccuracies.
Score 2: Tags are aspirational rather than accurate, use inconsistent prefixes, or contain redundant entries.
Score 1: Tags are missing or entirely unrelated to the track content.

## Timing settings

Evaluate whether timelimit, idle_timeout, and per-challenge timelimits are calibrated to the actual content.

Score 4: Track timelimit matches the sum of estimated challenge durations with a reasonable buffer (20-30%). idle_timeout is appropriate for the delivery context (300-600s for self-paced, higher for instructor-led).
Score 5: Timing is precisely calibrated. Track timelimit accounts for varying challenge difficulty. idle_timeout reflects the expected pace. Per-challenge timelimits (if set) are individually tuned.
Score 3: Timelimit is in the right ballpark but not tuned — either too generous (2x actual) or too tight (no buffer).
Score 2: Timelimit is within a reasonable range but idle_timeout is left at default regardless of context.
Score 1: Timing settings are wildly miscalibrated or left entirely at defaults.

## lab_config appropriateness

Evaluate whether lab_config settings (layout, sidebar, theme, loading experience) are configured for the track's content type.

Score 4: Layout matches content — code-heavy tracks use smaller sidebar (30), reading-heavy use larger (40-45). Theme is set. `loadingMessages` is `false` (no custom message lists); tracks with long setup times keep learners engaged via notes slides instead.
Score 5: Every lab_config setting is deliberately chosen and justified by the content. `loadingMessages` is disabled and loading engagement is handled by notes slides. Layout sidebar size is tuned per challenge where needed.
Score 3: Lab_config is partially configured — layout is set but theme is left at default.
Score 2: Lab_config is mostly defaults, layout does not match the content type, or custom `loadingMessages` arrays are used instead of notes slides.
Score 1: Lab_config is missing or entirely default with no consideration of the track's needs.

## Pausable and behavioral settings

Evaluate whether pausable, skipping_enabled, and other behavioral settings match the track's delivery context and content.

Score 4: `pausable: false` (the default) unless the user explicitly requested pause/resume for a multi-session track. skipping_enabled reflects whether challenges are independent or sequential. Settings are coherent with each other.
Score 5: All behavioral settings are deliberately configured with the delivery context in mind. `pausable` stays `false` unless explicitly requested, and when enabled it has an appropriate TTL and suspend-safe state. `show_timer` is `true` (the default, so learners can pace themselves) except on presenter-paced or open-ended tracks where it is deliberately hidden.
Score 3: Key settings are configured but some are left at defaults that don't match the context (e.g., skipping enabled on a strongly sequential track).
Score 2: Settings are inconsistent — e.g., `pausable: true` without an explicit request or with a short/missing TTL, or skipping enabled when challenges have hard dependencies.
Score 1: All behavioral settings are defaults with no consideration of the track's needs.
