# Challenge Design

Evaluates the pedagogical design of generated challenges — whether each challenge has a clear goal, appropriate scope, descriptive title, and well-structured assignment that guides the learner through a focused learning experience.

## Goal clarity

Evaluate whether each challenge has a clear, achievable goal that the learner understands before starting.

Score 4: Clear goal stated or evident from the assignment. The learner knows what they are working toward.
Score 5: Goal is explicit in the first paragraph, connects to the track's overall learning objectives, and the learner can self-assess progress toward it.
Score 3: Goal is implied but not stated. The learner can figure out the objective from the steps.
Score 2: Goal is vague or buried. The learner must complete the challenge to understand what it was about.
Score 1: No discernible goal. Steps are disconnected from a coherent learning objective.

## Challenge scoping

Evaluate whether each challenge covers a single concept and is achievable in a focused session.

Score 4: Single concept per challenge, 5-15 minutes estimated completion, no extraneous steps.
Score 5: Tightly scoped to one learning objective. Every step contributes to the goal. The challenge is achievable in one focused effort without fatigue.
Score 3: Mostly one concept but slightly overloaded — combines two loosely related ideas or would take 20+ minutes.
Score 2: Multiple unrelated concepts crammed into one challenge, or the challenge would take 30+ minutes.
Score 1: The challenge tries to cover an entire topic area. The learner would need multiple sessions to complete it.

## Step structure quality

Evaluate whether the assignment body guides the learner through a logical sequence of steps with clear instructions.

Score 4: Steps are numbered or clearly delineated, each describes one action, and the sequence is logical. The learner can follow without guessing.
Score 5: Steps build incrementally — each one produces a visible result that confirms progress. The learner is never uncertain about what to do next or whether they did it correctly.
Score 3: Steps exist but some are vague ("configure the application") or combine multiple actions into one step.
Score 2: Instructions are a wall of text with no clear step boundaries. The learner must parse the entire assignment to extract action items.
Score 1: No discernible step structure. The assignment reads as prose with no actionable guidance.

## Check script alignment

Evaluate whether what the check script validates matches what the assignment asks the learner to do.

Score 4: Check script validates the outcomes described in the assignment. No surprise checks for things not mentioned in the instructions.
Score 5: Perfect alignment — every check assertion maps to a specific assignment step, and the fail-messages reference the relevant step. The learner is never surprised by what is validated.
Score 3: Most checks align with the assignment, but one check validates something not explicitly mentioned in the instructions.
Score 2: Significant gaps between what the assignment describes and what the check validates. The learner may complete all steps correctly and still fail.
Score 1: Check script validates entirely different things than what the assignment asks for.

## Title and teaser quality

Evaluate whether challenge titles and teasers are descriptive and help learners build a mental model of the track from the sidebar navigation alone.

Score 4: Titles are descriptive, action-oriented, and use consistent naming conventions across the track.
Score 5: A learner could build a complete mental model of the track's progression from the challenge titles alone. Each title describes the outcome, not just the topic.
Score 3: Titles are descriptive but inconsistent in style — some are action-oriented, others are topic labels.
Score 2: Titles are vague ("Configuration", "Setup") or use generic numbering ("Step 1", "Part A").
Score 1: Titles are missing, misleading, or use placeholder text.
