# Completeness

Binary checks for structural completeness of generated track artifacts. Each item verifies that a required element exists and is correctly wired. These are mechanical checks -- if the element is present and correct, it passes.

## Every challenge has an assignment.md

Verify that each challenge directory contains an assignment.md file with non-empty content.

- **1 (pass):** Every challenge directory contains an assignment.md file with content.
- **0 (fail):** One or more challenge directories are missing assignment.md or the file is empty.

## Every challenge has expected scripts

Verify that each challenge has the scripts it needs: at minimum a check script (if the challenge has check assertions) and a solve script. Setup and cleanup scripts are present where the challenge plan specifies them.

- **1 (pass):** Every challenge has all scripts specified in its plan, and all scripts are executable.
- **0 (fail):** One or more challenges are missing planned scripts, or scripts exist but lack executable permissions.

## Config.yml is valid

Verify that config.yml exists, parses as valid YAML, and contains the required top-level structure for an Instruqt track.

- **1 (pass):** config.yml is present, parses cleanly, and contains required keys.
- **0 (fail):** config.yml is missing, fails to parse, or is missing required top-level keys.

## Track.yml is valid

Verify that track.yml exists, parses as valid YAML, and contains required track metadata (title, slug, description).

- **1 (pass):** track.yml is present with valid metadata.
- **0 (fail):** track.yml is missing, fails to parse, or is missing required metadata fields.

## Challenge directories match assignment.md slugs

Verify that every challenge directory follows the NN-slug naming convention and that each directory contains an assignment.md whose `slug:` field matches the directory name (minus the number prefix).

- **1 (pass):** Every challenge directory uses NN-slug format, and the slug in each assignment.md matches its directory name.
- **0 (fail):** One or more challenge directories are missing the numbered prefix, or the slug in assignment.md does not match the directory name.
