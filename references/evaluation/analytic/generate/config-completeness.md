# Config Completeness

Evaluates whether the generated config.yml is fully configured to support the entire track. Binary checks (all scripts present, executable permissions) have moved to the completeness checklist. This rubric evaluates the quality and appropriateness of config.yml configuration decisions.

## Resource images appropriate

Evaluate whether each resource uses an appropriate image that matches the technology stack, with a version that aligns with the track content.

Score 4: Images match the technology stack and versions are appropriate for the track content. No deprecated or end-of-life images.
Score 5: Smallest appropriate image selected for each resource. Specific versions chosen to match the track's content exactly. Image choices minimize startup time and resource consumption.
Score 3: Reasonable image choice but not optimal for the workload.
Score 2: Image works but is oversized or uses a generic image where a purpose-built one exists.
Score 1: Unrelated or deprecated image used.

## Resource sizing

Evaluate whether machine type and memory allocations are appropriate for the workload.

Score 4: Appropriate sizing that considers simultaneously running services and tools. Resources are neither over-provisioned nor constrained.
Score 5: Right-sized with explicit justification for the chosen values. Machine types balance cost and performance for the specific workload.
Score 3: Functional but not tuned for the specific workload.
Score 2: Noticeable performance issues or approximately 2x over-provisioned.
Score 1: Drastically wrong sizing causing OOM kills or extreme resource waste.

## Environment supports all challenges

Evaluate whether the config.yml environment supports the entire track, not just the early challenges.

Score 4: All challenges are supported with infrastructure provisioned correctly. Services needed by later challenges are accounted for.
Score 5: All challenges supported, provisioning happens at the right timing via setup scripts, and the environment is structured to support the full challenge progression.
Score 3: All challenges can run but the environment is fragile or requires unplanned manual intervention.
Score 2: Most challenges work, but at least one later challenge would fail due to missing infrastructure.
Score 1: Only the first few challenges can run successfully in this environment.

## YAML structure valid

Evaluate whether config.yml follows correct YAML structure with proper indentation, valid keys, and correct value types.

Score 4: Valid YAML with correct structure. All required keys are present with appropriate values. Indentation is consistent.
Score 5: Clean, well-organized YAML. Keys are logically grouped. Comments explain non-obvious configuration choices. Structure follows Instruqt config.yml conventions exactly.
Score 3: Valid YAML but with minor structural issues -- inconsistent indentation or keys in an unusual order.
Score 2: YAML parses but contains incorrect key names or value types that would cause runtime errors.
Score 1: Invalid YAML that does not parse, or missing required top-level keys.
