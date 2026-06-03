# Challenge Roadmap

Evaluates whether the challenge roadmap tells a coherent learning story. The roadmap should read as a narrative arc from novice to competent -- each challenge's goal requiring skills from prior challenges, complexity increasing naturally, and no concept appearing without foundation.

## Challenge roadmap progression

Evaluate whether challenges follow a logical learning sequence where each builds on the previous.

Score 4: Each challenge builds on the previous one. Complexity increases naturally. No orphaned concepts appear without foundation from earlier challenges.
Score 5: Deliberate progression where each challenge's goal requires skills from prior challenges. The roadmap reads as a narrative arc. A reader could predict what the next challenge covers based on the learning trajectory.
Score 3: Clear progression but gaps between challenges -- some concepts appear without foundation, or the difficulty jump between adjacent challenges is uneven.
Score 2: Some logical order but challenges could be rearranged without impact on learning. The sequence is arbitrary rather than intentional.
Score 1: Challenges are unrelated topics with no logical order. Prerequisites are violated (e.g., advanced configuration before basic setup).

## Challenge dependencies

Evaluate whether the roadmap's dependency chain flows forward -- challenge N never requires knowledge only taught in challenge N+k.

Score 4: All challenges follow the prerequisite chain. Knowledge flows forward. Skipping a challenge degrades the experience but does not break comprehension of later challenges.
Score 5: Airtight prerequisite chain. Every concept is fully planned before it is referenced. Dependencies are explicit in the roadmap, not implicit.
Score 3: Logical sequence overall, but one concept is referenced before its formal introduction. A careful reader would notice the gap.
Score 2: One or two challenges assume knowledge from later challenges. The dependency chain has backward references.
Score 1: Multiple challenges require knowledge from later challenges. The roadmap cannot be followed in order.

## Progressive complexity

Evaluate whether the roadmap plans for increasing independence -- early challenges providing more guidance, later challenges granting more autonomy.

Score 4: Clear progression from guided to independent. The roadmap indicates that scaffolding will decrease as skills are established.
Score 5: Deliberate scaffolding plan visible in the roadmap. Early challenges plan for explicit instruction; later challenges plan for goal-setting with less hand-holding. The progression is smooth and intentional.
Score 3: Some variation in planned guidance level, but the progression is inconsistent -- some later challenges still plan for the same detail level as early ones.
Score 2: Minor variation in guidance level. The roadmap treats all challenges as needing the same amount of scaffolding.
Score 1: All challenges plan for the same level of guidance throughout. No progression from novice to competent is visible.

## Challenge scope balance

Evaluate whether challenges are balanced in scope and estimated time, driven by topic coherence rather than arbitrary step counts.

Score 4: Challenges are 5-15 minutes each with no more than 2x variation between shortest and longest. Variation is justified by topic scope.
Score 5: Consistent challenge scope where each challenge has a clear, achievable goal completable in a focused session. Balance is driven by learning coherence, not symmetry.
Score 3: Mostly balanced with minor variations. One challenge may be noticeably longer or shorter without clear justification.
Score 2: Noticeable imbalance -- one or two challenges are significantly longer than others, suggesting they should be split.
Score 1: Wildly unbalanced -- some challenges are 2 minutes, others 30 minutes. Organization is by convenience, not learning.

## Timelimit settings

Evaluate whether planned timelimit values per challenge are appropriate for the expected work.

Score 4: Each challenge has a timelimit that allows comfortable completion with buffer. Timelimits increase with challenge complexity. No challenge has a timelimit so short it pressures the learner or so long it suggests poor scoping.
Score 5: Timelimits are precisely calibrated per challenge based on expected steps. Simple challenges get shorter limits, complex challenges get proportionally longer ones. The values reflect actual task duration estimates, not uniform defaults.
Score 3: Timelimits are present but uniform across all challenges regardless of complexity, or one challenge has a noticeably wrong timelimit.
Score 2: Timelimits are present but several are clearly wrong -- trivial challenges get 30 minutes, complex challenges get 5 minutes.
Score 1: No timelimits specified, or all challenges use a single default value with no consideration of actual scope.

## Difficulty progression labeling

Evaluate whether challenges are labeled with appropriate difficulty indicators that match the actual complexity of the work.

Score 4: Difficulty labels (beginner, intermediate, advanced) are present and accurately reflect the cognitive and technical demands of each challenge. Labels progress logically through the track.
Score 5: Difficulty labels are precise, consistent with challenge content, and form a visible gradient. Labels help learners set expectations and help authors calibrate explanation depth.
Score 3: Difficulty labels are present but one or two are mismatched with the actual challenge complexity.
Score 2: Difficulty labels exist but are inconsistent or arbitrary -- no clear correlation with challenge demands.
Score 1: No difficulty labeling, or labels are uniformly the same across all challenges.

## Slug naming conventions

Evaluate whether challenge slugs follow a consistent, descriptive naming pattern.

Score 4: Slugs follow the NN-descriptive-name pattern (e.g., 01-install-the-cli, 02-configure-the-cluster). Slugs are concise, descriptive, and ordered.
Score 5: Slugs are concise, descriptive, and form a readable table of contents on their own. The naming pattern is uniform and the slug alone conveys the challenge's purpose.
Score 3: Slugs are descriptive but naming is inconsistent -- some use verbs, others use nouns, numbering is present but irregular.
Score 2: Slugs exist but are vague or generic (e.g., 01-step-one, 02-next-part).
Score 1: No slugs defined, or slugs are meaningless identifiers with no descriptive value.
