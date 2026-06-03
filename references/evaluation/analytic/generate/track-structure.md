# Track Structure

Evaluates the organization and progression of challenges within the generated track — whether they follow a logical learning arc, are balanced in scope, and form a coherent whole.

## Challenge ordering

Evaluate whether challenges follow a natural learning progression where foundations come first and each challenge builds on what was covered before.

Score 4: Clear progression where each challenge builds on the previous one. Foundational concepts precede application. No prerequisite violations.
Score 5: Deliberate pedagogical arc. The sequence feels inevitable — each challenge arrives at exactly the right moment. A learner would struggle to suggest a better order.
Score 3: Logical progression with one minor ordering issue that does not break comprehension.
Score 2: General progression is apparent but two or more challenges are out of sequence — prerequisites are introduced after they are needed.
Score 1: Challenge order is arbitrary. Prerequisites are violated. The track reads as a collection of unrelated exercises.

## Challenge balance

Evaluate whether challenges are balanced in scope and duration, driven by topic coherence rather than arbitrary sizing.

Score 4: Challenge durations reflect the natural scope of their content. Variation exists but is clearly content-driven (a setup challenge is shorter, a complex debugging challenge is longer).
Score 5: Well-justified variation. The track has a natural pace — no challenge feels rushed or bloated. Early challenges are shorter and more guided; later challenges allow more independence.
Score 3: Roughly balanced with minor variation. One challenge is noticeably longer or shorter than the others without clear justification.
Score 2: Noticeable imbalance — one challenge would take 30 minutes while others take 5. Content should be split or merged.
Score 1: Extreme imbalance resulting from poor content distribution across challenges.

## Difficulty progression

Evaluate whether the track has a deliberate difficulty ramp that matches the stated difficulty labels on each challenge.

Score 4: Difficulty increases progressively. Labels (basic/intermediate/advanced) match the actual content complexity. Early challenges provide more guidance; later ones expect more independence.
Score 5: Smooth difficulty curve with no sharp jumps. Each challenge is slightly harder than the last. The learner never feels overwhelmed or bored. Difficulty labels are precise.
Score 3: Generally progressive but one challenge is noticeably harder or easier than its position suggests. Labels are mostly accurate.
Score 2: Difficulty is inconsistent — hard challenges appear early or easy ones appear late. Labels do not match content complexity.
Score 1: No difficulty progression. All challenges are the same difficulty, or hard challenges appear before foundational concepts are covered.

## Directory structure and naming

Evaluate whether the track's file structure follows conventions — numbered directories, consistent slug naming, all required files present.

Score 4: Challenge directories use sequential numbering (01-, 02-, ...) matching track.yml ordering. Slugs are descriptive and consistent. Every directory has assignment.md and required scripts.
Score 5: Perfect convention adherence. Slugs read as a table of contents. Script naming is consistent across all challenges (same hostnames, same script types).
Score 3: Directories are numbered correctly but slugs are inconsistent in style or not descriptive.
Score 2: Numbering gaps or mismatches with track.yml ordering. Some slugs are placeholder-quality.
Score 1: No numbering convention. Slugs are random or use generic names. Files are missing from challenge directories.

## Config.yml coherence

Evaluate whether config.yml accurately reflects the infrastructure the track actually needs — no unused resources, no missing resources, ports and hostnames align with tab definitions.

Score 4: Every resource in config.yml is used by at least one challenge. Every service tab has a matching port. No orphaned containers or VMs.
Score 5: Config.yml is minimal and precise. Resources are ordered logically. Comments or environment variables clarify the purpose of each resource. No waste.
Score 3: Config.yml has one minor issue — an unused port exposure or a resource that could be sized differently.
Score 2: Config.yml has unused resources (containers/VMs that no challenge references) or is missing resources that challenges need.
Score 1: Config.yml is significantly out of sync with the track content — multiple unused resources or missing dependencies.
