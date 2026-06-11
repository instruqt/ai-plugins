# Scoring Guide

Global score-level definitions used across all evaluation rubrics. Every scorer agent receives this guide as context to ensure consistent calibration.

Checklist rubrics use levels 0-1. Analytic and holistic rubrics use levels 1-5. Score 4 is the production baseline -- the minimum for a track that ships.

| Score | Label | Meaning |
|-------|-------|---------|
| 0 | Fail | (Checklist only) Required element is missing or check fails |
| 1 | Blocking | Blocks the learner or is fundamentally broken |
| 2 | Significant Gaps | Degrades the learning experience with major omissions or errors |
| 3 | Functional | Works but has notable quality issues that affect the experience |
| 4 | Production Baseline | Meets all stated requirements; ready to ship |
| 5 | Deliberate Craft | Demonstrates intentional quality visible in the output itself |

## How to Use These Levels

**Score 4 is the anchor.** Write or evaluate Score 4 first -- it describes what "done" looks like. Then ask: what would make this noticeably worse (3, 2, 1) or noticeably better (5)?

**Score 5 describes output, not process.** "Evidence of testing" is not observable in the artifact. "Error messages guide the learner toward the fix" is. Score 5 must be verifiable by reading the track, not by watching the author work.

**Impact over counting.** "3 shellcheck warnings" is a count. "Shellcheck warnings indicate correctness issues that could cause scripts to fail" is impact. Describe what the learner experiences, not how many defects exist.

**Parallel structure.** Each score level for a criterion should address the same dimensions. The difference between levels is quality, not topic. If Score 4 mentions error handling, Score 2 should also mention error handling (just worse), not introduce an unrelated concern.
