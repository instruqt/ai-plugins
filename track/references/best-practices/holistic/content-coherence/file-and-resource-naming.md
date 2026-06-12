# File and Resource Naming

Consistent naming across the track for YAML configuration, scripts, assignments, and challenge directories.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Naming conventions vary widely. Mixed casing styles, mismatched directory and slug names, and inconsistent resource name patterns across the track. |
| 2 | Below Standard | A naming pattern exists but is not followed consistently. Several challenge directories, scripts, or filenames deviate from the pattern without reason. |
| 3 | Adequate | Most names follow a consistent pattern. A few deviations exist (e.g., one directory uses a different casing style or one script name does not match its challenge). |
| 4 | Good | Naming is consistent across YAML configuration, scripts, assignments, and challenge directories. One or two minor inconsistencies that do not cause confusion. |
| 5 | Excellent | All naming is systematic and predictable. Challenge directory names match challenge slugs exactly. Script filenames (setup, check, solve, cleanup) follow a uniform pattern. Assignment files are descriptive and follow a single convention throughout. |

- Challenge directory names match slugs defined in track.yml
- Script filenames follow the pattern (setup-<host>, check-<host>, solve-<host>, cleanup-<host>)
- Assignment filenames are consistent across challenges
- YAML keys and values follow a uniform style
- Intentional deviations are acceptable when they are part of the learning scenario (e.g., a track teaching naming conventions may deliberately use inconsistent names for the learner to fix)

## What to Watch For

- Inconsistent naming conventions across challenge directories, scripts, and configuration files
- Missing cleanup scripts when challenges create temporary state that should be cleaned up
- **Image filenames** should use `kebab-case-descriptive-name.png` -- avoid spaces, `@` characters, and screenshot tool output names like `CleanShot 2024-05-21 at 14.34.52@2x.png` or `Feb-04-2026_at_11.36.08-image.png`, which break URLs and are impossible to reference reliably in markdown
