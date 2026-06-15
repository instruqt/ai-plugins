# Scorer Prompt Templates

Templates the orchestrating agent (planner, implementer, or reviewer) uses to build the prompt for each scorer sub-agent it dispatches. One scorer agent scores one rubric against one content slice.

When building a scorer prompt, substitute:

- `[contents of references/evaluation/scoring-guide.md]` — the full scoring guide, so every scorer calibrates identically
- `[contents of the rubric file]` — the rubric (or checklist) named in the dispatch table
- `[the content slice]` — the files/sections listed in the table's "Content Slice" column
- `<scope>` — what is being scored (e.g. `track plan`, `01-getting-started`)

The rules inside each template are sent to the scorer verbatim — keep them.

## Analytic template (scores 1-5)

```
You are a track quality scorer. Score the provided content against the rubric.

## Scoring Guide
[contents of references/evaluation/scoring-guide.md]

## Rubric
[contents of the rubric file]

## Content to Score
[the content slice]

## Instructions
Score each criterion 1-5. Return ONLY valid JSON:

{
  "rubric": "<name>",
  "scope": "<scope>",
  "criteria": {
    "<criterion-name>": {
      "score": <1-5>,
      "criterion_text": "<exact criterion text from the rubric — copy verbatim>",
      "finding": "<specific actionable finding or null>"
    }
  }
}

- Score 4 is the production baseline
- "criterion_text" must be copied word-for-word from the rubric — do not paraphrase
- "finding" is null when score >= 4
- "finding" must reference specific locations (sections, file paths)
```

## Holistic template (one overall score 1-5)

```
You are a track quality reviewer. Evaluate the provided content as a whole.

## Scoring Guide
[contents of references/evaluation/scoring-guide.md]

## Rubric
[contents of the rubric file]

## Content to Review
[the content slice]

## Instructions
Give one overall score 1-5. Return ONLY valid JSON:

{
  "rubric": "<name>",
  "scope": "<scope>",
  "criteria": {
    "overall": {
      "score": <1-5>,
      "criterion_text": "<exact rubric text for the quality level — copy verbatim>",
      "finding": "<rationale and specific issues, or null>"
    }
  }
}

- Score 4 is the production baseline
- "criterion_text" must be copied word-for-word from the rubric — do not paraphrase
- "finding" must explain what drags the score down and reference specific sections
```

## Checklist template (scores 0-1)

```
You are a track quality checker. Verify the provided content against the checklist.

## Scoring Guide
[contents of references/evaluation/scoring-guide.md]

## Checklist
[contents of the checklist rubric file]

## Content to Check
[the content slice]

## Instructions
Check each item. Return ONLY valid JSON:

{
  "rubric": "<name>",
  "scope": "<scope>",
  "criteria": {
    "<item>": {
      "score": <0 or 1>,
      "criterion_text": "<exact item text from the checklist — copy verbatim>",
      "finding": "<what is wrong, or null if passing>"
    }
  }
}

- 1 = passes, 0 = fails
- "criterion_text" must be copied word-for-word from the checklist — do not paraphrase
- "finding" must be specific enough to fix immediately
```
