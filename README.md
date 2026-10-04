# Schedule health check

A skill for reviewing a contractor's project schedule from the client side and
drafting review comments to send back.

It works from either:

- a **Primavera P6 XER export**, where the full network is available and every
  check can be run; or
- a **printed schedule** — a PDF or image of a Gantt chart with an activity
  table — where only part of the review is possible.

The output is a numbered list of required corrections, recommendations and
questions, written so it can be pasted into correspondence with the
contractor's scheduler. It does not issue an overall pass or fail.

## What it checks

From an XER: open ends, leads and lags, relationship type mix, hard
constraints, negative and high float, dangling logic, duration coarseness,
calendars, resource and cost loading, date integrity against the data date,
and the continuity of the critical path.

From a print: whatever the page actually supports — the layout itself, date
contradictions readable in the table, how criticality is distributed, repeated
float values, clusters of identical dates and durations, scope completeness,
and milestone structure.

## Design decisions worth knowing

**It will not state a number it cannot see.** On a printed schedule, counts and
percentages are not produced at all. A plausible-looking statistic derived from
a picture is worse than no review, because it gets quoted back in
correspondence and cannot be defended.

**On a print, it prefers the question to the assertion.** A print carries no
activity-type column, so level-of-effort and summary rows are indistinguishable
from real work — which makes float findings unreliable. Those go to a
clarification section as questions rather than being asserted as defects.

**Defensibility over coverage.** A finding that is probably right but hard to
substantiate is dropped or demoted. One successfully rebutted comment weakens
every other comment in the document.

**No overall verdict.** Schedules are rarely rejected outright, and a summary
judgement invites an argument about the judgement instead of work on the
findings.

## Limitations

- A print review is a first pass, not a substitute for the file. Every print
  review closes by asking for the XER, a tabular export, or a reprint with the
  missing columns.
- Baseline-comparison checks (Missed Tasks, BEI, CPLI) need a baseline plus
  status data and are out of scope.
- It does not judge whether durations are realistic. That needs scope knowledge
  the file does not contain.
- Findings derived from bar colour carry a caveat and should be confirmed in
  the scheduling tool before being relied on.

## Installation

Install from the Claude directory, or download this repository and upload the
`skills/schedule-health-check` folder as a ZIP under
Settings → Capabilities → Skills.

## Feedback

Issues and pull requests welcome, particularly reports of findings that turned
out to be wrong on a real schedule. Those are more useful than feature
requests.

## License

MIT. See LICENSE.
