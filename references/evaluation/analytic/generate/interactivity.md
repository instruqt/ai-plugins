# Interactivity

Evaluates the quality and effectiveness of interactive elements in the generated track. In tracks, every challenge is inherently interactive (each has a check script), so interactivity is measured by the quality of hands-on engagement, not the presence of activities.

## Concepts reinforced through practice

Evaluate whether every key concept introduced in the track is reinforced through a hands-on task that the learner performs and the check script validates.

Score 4: All key concepts are reinforced through hands-on tasks. The learner does not just read about a concept — they apply it and get feedback via the check.
Score 5: Concepts build on each other through practice. Later challenges require applying earlier concepts in new contexts, creating compounding reinforcement.
Score 3: Most concepts are reinforced, but one or two important concepts are only explained in the assignment without a corresponding hands-on task or check.
Score 2: Several concepts are explained but never practiced. The learner reads about them but the check script does not validate understanding.
Score 1: The track is primarily reading with minimal hands-on practice. Check scripts validate trivial actions rather than concept understanding.

## Check script as learning feedback

Evaluate whether check scripts provide meaningful learning feedback, not just pass/fail gates.

Score 4: Check scripts validate the learning outcome with specific, actionable fail-messages. The learner learns from failure — the message tells them what to investigate, not just what is wrong.
Score 5: Checks form a diagnostic sequence — ordered from prerequisites to specifics. The first failure is always the most actionable. Success messages (set-status) reinforce what the learner accomplished.
Score 3: Check scripts have fail-messages but they are generic ("not configured correctly") rather than diagnostic.
Score 2: Check scripts exist but fail-messages are missing or vague. The learner gets "check failed" with no guidance.
Score 1: Check scripts are missing for some challenges, or validate things unrelated to the learning objective.

## Tab layout supports the task

Evaluate whether each challenge's tab configuration gives the learner the right tools for the task without clutter.

Score 4: Each challenge has the tabs needed for its task — terminal for CLI work, editor for code editing, service tab for web UIs. No unnecessary tabs.
Score 5: Tab configuration is precisely tailored per challenge. Challenges that need multiple panes have them. Service tabs deep-link to the relevant page. Editor tabs open the right file or directory.
Score 3: Tabs are functional but not optimized — every challenge has the same set of tabs regardless of whether they are all needed.
Score 2: Missing a required tab (e.g., no service tab for a challenge that asks the learner to check a web UI).
Score 1: Tab configuration is wrong — wrong hostnames, wrong ports, or missing critical tabs.

## Progressive independence

Evaluate whether the track progressively reduces hand-holding as the learner builds competence.

Score 4: Early challenges provide step-by-step commands. Later challenges describe the goal and let the learner figure out the approach. The level of guidance decreases as the learner's skill increases.
Score 5: Deliberate scaffolding — early challenges model the pattern, middle challenges guide with hints, late challenges set the goal and step back. The learner feels their growing competence.
Score 3: Some progression exists but inconsistent — a late challenge suddenly provides step-by-step instructions for something the learner should know by now.
Score 2: All challenges have the same level of guidance regardless of position in the track.
Score 1: Later challenges are actually more hand-holding than earlier ones, or all challenges are copy-paste exercises with no independent thinking required.

## Assignment engagement

Evaluate whether assignments are engaging — do they provide context, motivation, and a sense of purpose, or are they dry instruction manuals?

Score 4: Assignments set context before asking the learner to act. The learner understands why they are doing each step, not just what to do.
Score 5: Assignments tell a story. Each challenge has a scenario or motivation that makes the task feel purposeful. Technical content is woven into a narrative rather than presented as a list of commands.
Score 3: Assignments are clear and correct but dry — they read as instruction manuals without context or motivation.
Score 2: Assignments are lists of commands to run with minimal explanation of why.
Score 1: Assignments are incomprehensible, incomplete, or provide no context for the actions requested.
