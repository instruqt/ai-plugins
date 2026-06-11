# Infrastructure Changes

Evaluates whether planned infrastructure changes for the challenge are specific and implementable. Infrastructure changes include ports, services, environment variables, and configuration that this challenge requires beyond what prior challenges established.

## Infrastructure changes clarity

Evaluate whether planned infrastructure changes are specific enough to implement directly.

Score 4: Complete specification of new services, ports, environment variables, and how they connect to existing infrastructure state. An implementer could begin writing config.yml additions and setup scripts from this section.
Score 5: Infrastructure changes are precise enough to translate directly to config.yml and setup scripts. Includes dependency ordering (what must exist before this challenge's resources). No ambiguity about what is new versus what was established by prior challenges.
Score 3: Specific enough to start implementation but missing some details -- port numbers, environment variable values, or service dependencies need to be inferred.
Score 2: New services or configuration mentioned but without ports, images, or environment details. An implementer would need to research and guess.
Score 1: Infrastructure changes mentioned vaguely or not at all when the challenge clearly requires new services or configuration.

## Tab configuration varies by need

Evaluate whether the plan specifies appropriate tabs for the challenge's content and tasks.

Score 4: Plan specifies tabs that match the challenge's needs -- terminal tabs for CLI work, service tabs for web UIs, editor tabs for file editing. Tab choices are consistent with the type of work planned.
Score 5: Tabs are deliberately chosen per challenge with explicit rationale. The plan specifies which services need tabs, what ports they use, and how the learner interacts with each. Tab layout supports the assignment flow.
Score 3: Tabs are mentioned and mostly appropriate, but some tasks would benefit from tabs that are not planned.
Score 2: Only terminal tabs planned despite the challenge involving web UIs or file editing that would benefit from additional tab types.
Score 1: No tab planning. The plan does not address how the learner will interact with the infrastructure.
