---
name: schedule-health-check
description: Review a contractor's schedule or programme and draft review comments to send back. Reads a P6 XER export or a printed Gantt PDF. Use for baseline or update reviews, logic and DCMA checks.
---

# Schedule health check (client-side review)

Review a single Primavera P6 schedule exported as XER and report structural defects that make the schedule unreliable for forecasting or for measuring contractor progress.

This is a review of ONE schedule — a baseline, a tender schedule, or a revision submitted for approval. It is not a comparison against a previous baseline and not a progress review.

## Which input do you have

The review splits into two branches. Decide first, because what can be verified differs sharply.

- **XER export** — the full network is available. Run every check.
- **Printed schedule (PDF, image, screenshot)** — an activity table with a Gantt chart alongside. Only part of the review is possible. Go to "Reviewing a printed schedule" and follow it instead.

If the user supplies both, use the XER and treat the print only as a cross-check on what was shown to management.

If the user has a printed schedule but could obtain the XER, say so early. A print review is a reasonable first pass, not a substitute, and the difference should not be discovered at the end of the report.

## Reading the XER file

An XER file is plain tab-delimited text, not a binary format. Read it directly.

Structure:
- `ERMHDR` — first line: export version, date, source database
- `%T<tab>TABLENAME` — starts a table
- `%F<tab>field1<tab>field2...` — column names for that table
- `%R<tab>value1<tab>value2...` — one data row
- `%E` — end of file

Tables that matter for this review:

| Table | What to take from it |
|---|---|
| `PROJECT` | `last_recalc_date` (the data date), `plan_start_date`, `proj_short_name` |
| `TASK` | one row per activity — see fields below |
| `TASKPRED` | one row per relationship: `task_id` (successor), `pred_task_id`, `pred_type`, `lag_hr_cnt` |
| `CALENDAR` | `clndr_name`, `day_hr_cnt` — needed to convert hours into days |
| `PROJWBS` | WBS structure, for reporting where defects cluster |
| `TASKRSRC` | resource and cost assignments |

Key `TASK` fields: `task_code` (the activity ID the user sees), `task_name`, `task_type`, `status_code`, `target_drtn_hr_cnt`, `remain_drtn_hr_cnt`, `total_float_hr_cnt`, `cstr_type`, `cstr_date`, `act_start_date`, `act_end_date`, `clndr_id`.

Values you need to decode:
- `task_type`: `TT_Task` (normal), `TT_Mile` (start milestone), `TT_FinMile` (finish milestone), `TT_LOE` (level of effort), `TT_Rsrc` (WBS summary)
- `status_code`: `TK_NotStart`, `TK_Active`, `TK_Complete`
- `pred_type`: `PR_FS`, `PR_SS`, `PR_FF`, `PR_SF`
- `cstr_type`: hard constraints are `CS_MANDSTART`, `CS_MANDFIN`, `CS_MSO` (start on), `CS_MEO` (finish on). Soft ones are `CS_MSOA`, `CS_MSOB`, `CS_MEOA`, `CS_MEOB`, `CS_ALAP`.

**Durations and float are stored in HOURS, not days.** Divide by the `day_hr_cnt` of the activity's own calendar before reporting anything in days. Getting this wrong makes every threshold meaningless — a 44-day threshold becomes 44 hours.

Milestones, LOE and WBS-summary activities are excluded from duration and resource checks, because they have no real duration by definition. State this exclusion in the report so the contractor cannot dispute the counts.

## The checks

Count each defect, express it as a share of the relevant population, and always list the activity IDs — a percentage with no IDs is not actionable and will be argued with.

| # | Check | Threshold | Why it matters |
|---|---|---|---|
| 1 | Activities with no predecessor or no successor (open ends) | ≤ 5% | The network cannot calculate a true path; dates move without cause |
| 2 | Negative lags (leads) | 0 | Hides overlap the contractor never justified; distorts float |
| 3 | Positive lags | ≤ 5% | Lag is hidden work or hidden risk with no owner and no progress measure |
| 4 | Finish-to-Start relationships | ≥ 90% of all links | Heavy SS/FF use usually means the sequence was forced to fit a date |
| 5 | Start-to-Finish relationships | 0 | Almost always a modelling error rather than an intent |
| 6 | Hard constraints | ≤ 5% | The schedule stops reacting to delay; float becomes fiction |
| 7 | Total float above a quarter of the remaining project duration | list, do not score | Usually missing logic rather than genuine slack — see below |
| 8 | Negative total float | 0 | The plan is already late against its own logic or constraints |
| 9 | Remaining duration longer than two reporting cycles | list, do not score | Cannot be progressed meaningfully — see below |
| 10 | Activities with duration but no resource or cost | report count | Nothing to measure earned progress against |
| 11 | Actual dates later than the data date, or forecast dates earlier than it | 0 | The schedule was not properly recalculated; all downstream dates are suspect |
| 12 | Number of separate calendars, and activities on an unexpected calendar | report | A single activity on a 7-day calendar can quietly shorten the whole path |
| 13 | Dangling logic: activity whose only predecessor link is SS, or only successor link is FF | report count | Looks connected, but its finish (or start) floats free |

### Checks 7 and 9 — why they are relative, and why they do not fail

Fixed thresholds (the usual 44 days) break on real project schedules. On a four-year offshore programme, 44 days of float is nothing; on a six-month campaign it is the whole job. The same applies to duration: a procurement or fabrication activity is legitimately long, while a construction activity of the same length almost always hides an unbroken sequence.

So:

- **Float** is measured against the remaining duration from the data date to project completion. Pair it with the number of successors: high float plus a single weak successor is a missing relationship, not spare time.
- **Duration** is measured against the reporting cycle — ask the user for it if it is not stated, and assume monthly if they do not answer. Apply it separately by discipline or phase where the activity coding allows, and exclude procurement, fabrication and transport. If the schedule has no coding that makes this separation possible, report that absence as a finding in itself.

Both produce a list of activities and a question for the contractor, never a pass/fail percentage. The value is the list, not the score — these checks find things worth asking about, not things that are automatically wrong.

Three DCMA checks do NOT apply to a single un-progressed schedule and should be named as out of scope rather than silently skipped: Missed Tasks, Baseline Execution Index, and Critical Path Length Index. All three need a baseline plus status data.

## Scope completeness

This applies to every review, not only to prints. The numeric checks all ask whether the activities present are modelled correctly; none of them asks whether the right activities are there at all, and a missing scope is invisible to every count in the table above.

Read the activity names and compare them against what the work obviously requires. Look for commissioning, pre-commissioning, statutory and authority inspections, energisation, punch list clearance, as-built documentation, O&M manuals, training, demobilisation and handover. Check also that each discipline named in a summary or activity title actually appears below it — an activity called "HVAC, Plumbing and Fire Protection" with nothing anywhere covering fire protection is a gap worth raising.

State these as questions about where the scope is covered rather than as assertions that it is absent, because the contract may place it elsewhere. One question naming three or four missing items is enough; do not list every activity a project of this type might contain.

## Critical path sanity

After the counts, examine the longest path, not just the float values:

- Does the critical path run continuously from the data date to project completion, or does it break?
- Does it pass through activities that make engineering sense, or through administrative filler (permits, reviews, "hold" activities)?
- Is any part of it driven by a constraint rather than by logic? Say which constraint.

A schedule can pass every numeric check and still have a critical path that no one would defend in a meeting. Say so plainly when that happens.

## Reviewing a printed schedule

A printed schedule is a filtered, formatted picture of a network, chosen by the contractor. Treat it that way: what is missing from the page is itself evidence.

### Dates come from the table, never from the bars

The activity table is text and is authoritative. The bar chart is a rendering, and what reaches you may be clipped, scaled, or missing part of its header band.

So: every date, duration and date comparison in the review is taken from the Start, Finish and Duration columns. Never state or infer a date by reading a bar against the timescale, and never describe the chart's time extent — that it "stops in July", that months are "cut off", that a bar "runs past the page" — because a truncated render looks identical to a truncated schedule from the inside, and the resulting comment sends the reviewer to check something that was never wrong.

If the printed timescale appears not to cover the full date range in the table, that is a fact about the copy being read, not a finding. Leave it out.

The bars support exactly two things: colour, and visibly drawn relationship lines. Both carry their caveat in the finding. Everything else comes from the columns.

### On a print, prefer the question to the statement

A print review earns its place by asking well, not by concluding. Reserve the required-corrections section for things readable as text in the table and not open to another explanation: an activity dated before the approval or predecessor it depends on, an actual date beyond the data date, a missing data date or revision, scope the contract requires that is nowhere in the table.

Everything else goes to the clarification section as a question — in particular anything derived from the float column, from bar colour, or from inferring what a row is. Float is the recurring trap: without an activity-type column there is no way to tell a level-of-effort or summary row from real work, so "the plan is already late against its own logic" can be true on one schedule and nonsense on the next, with the page looking identical. Asked as a question, it is useful either way; asserted, it is a coin flip with the reviewer's credibility on it.

This costs little. A contractor's scheduler answers a precise question and a stated defect with the same piece of work, and the question does not have to be withdrawn when the answer is "that row is level of effort".

### The overriding rule

Never report a count or a percentage that cannot be seen on the page. A print does not support the numeric checks above, and a plausible-looking "7% of activities have open ends" derived from a picture is worse than no review at all, because it will be quoted back in correspondence and cannot be defended. State what was observed, where, and say plainly when something cannot be determined.

Where something is visible but not countable, say so in those terms: "several activities in the fabrication section appear to have no successor" is honest; "12% open ends" is not.

### Read the header first

Before the bars, check what the page declares: data date, revision number and date, project and contract reference, applied filter, sort and grouping, and whether the chart is the full schedule or an extract.

A chart with no data date cannot be reviewed — the position of every bar relative to today is unverifiable. That is the first finding, not a footnote. A visible filter is equally important: a schedule shown "Level 2 summary" or filtered to one discipline cannot be used to judge completeness, and any comment about missing scope must be qualified accordingly.

### Audit the layout

Do this before anything else and report it near the top. Which columns the contractor chose to print is readable with certainty, needs no inference, and determines what the rest of the review can say. It is also the cheapest thing for them to fix.

Work through the list and record each as present or absent. Group the absent ones into three tiers, because a flat wish list gets ignored:

**Required — without these the schedule cannot be reviewed by the recipient at all**

| Item | What its absence costs |
|---|---|
| Data date | Every date is unanchored; nothing can be judged as early or late |
| Revision number and issue date | No way to tell which submission is being commented on |
| Activity type | Level-of-effort, summary and milestone rows cannot be separated from real work, so float and cost counts are unreliable |
| Total float | No independent view of criticality; negative float invisible |
| Constraint type and date | Cannot tell whether a date is driven by logic or imposed |
| Predecessors and successors with type and lag | No logic review at all: open ends, dangling logic, leads and lags, relationship mix |

**Useful — each unlocks specific checks**

| Item | What it unlocks |
|---|---|
| Remaining duration | Progress measurability; the printed original duration says nothing about what is left |
| Percent complete, and its type | Whether reported progress agrees with the dates |
| Actual dates marked distinctly from forecast | Date integrity against the data date |
| Calendar name per activity | Whether a quiet calendar change is shortening the path |
| Resource or cost loading | The basis on which progress will be earned |
| Activity codes or WBS path | Grouping by area and discipline, so findings can be targeted |
| Baseline start and finish | Variance against the approved plan |

**Desirable — presentation that makes the page usable**

Relationship lines drawn on the bars; critical bars coloured with a legend; a timescale covering the whole project window; the filter, sort and grouping stated; untruncated project and activity names; page numbering.

Report the absent items as one numbered finding, tiered as above, naming the checks each tier blocks. Then say plainly that the review's reach is limited accordingly — and keep the scope note at the top consistent with it.

### What the picture supports

**Critical path, where it is coloured.** This is the most informative feature of a typical print. Trace it end to end and ask:

- Is it continuous from the data date to the final milestone, or does it break and resume? A broken red line means either a constraint or missing logic is interrupting the path.
- Does it terminate at a contract milestone, or somewhere inside the works?
- Does it run through work that would be defended in a meeting — engineering, procurement, construction — or through permits, reviews and holds?
- Are two or more parallel red paths shown? If near-critical work is not visible at all, criticality may have been filtered out of the view.

**How criticality is distributed.** This is often the most revealing reading available on a weak print, and it works even when no logic is drawn. Look at where the critical work sits relative to where the detail sits.

A large, finely detailed section containing no critical activity at all — typically procurement, submittals, approvals or external works — rarely means that section is genuinely comfortable. It usually means it is not tied into the work it feeds. Those bars are positioned by dates rather than driven by logic, so they will not move when the work they supply slips, and the schedule will not show the consequence. Name the section, state that no activity within it appears critical despite feeding downstream construction, and ask for the driving relationships.

The reverse is equally informative: if almost everything is critical, the network is probably over-constrained or the float has been squeezed out by a contractual end date.

Where critical bars are coloured, this analysis is required, not optional. On a print with no float column it is the only criticality evidence that exists, so omitting it discards the most useful thing the page offers. Do not move criticality to the "not covered" section when colour is visible.

Carry the caveat on the finding itself rather than only in a note at the top of the report: say that criticality was read from the printed bars and should be confirmed in the scheduling tool before the comment is relied on. State the pattern — which sections carry critical work and which carry none — rather than listing every activity's colour, because a pattern survives one misread bar and a list does not.

**Clusters of identical dates and durations.** Scan for blocks of activities sharing the same start, the same finish, and the same duration. A row of approvals all at fourteen days, or a dozen submittals all running the same week, is a template applied to the whole class rather than a plan — and it points to either one shared predecessor or none. Quote two or three examples with activity IDs and ask what drives each block.

Uniform durations across an entire activity type are worth raising on their own: they tell you the durations were assigned by category, not estimated by scope.

**Cost or resource columns, where shown.** Check whether loading is complete or partial. Activities at zero cost sitting between costed neighbours matter more than whole sections at zero: if cost or hours are the basis of progress measurement, those activities can never earn, so reported progress will diverge from physical progress regardless of how the work goes. Also note any single activity carrying a disproportionate share of the total value — it deserves its own sequencing question.

**Relationship lines.** Where drawn, they support observation rather than counting:

- Bars with no incoming or outgoing line visible — candidate open ends, named individually.
- Lines running backwards on the page, right to left — out-of-sequence logic or a constraint overriding the network.
- A visible gap between the end of one bar and the start of its successor — a positive lag, or a missing driving relationship. Name the pair and ask which it is.
- Successors starting before their predecessor ends on a finish-to-start link — a lead, or a link type that is not what it appears.

**Dates in the table.** Activities whose dates run past the project completion milestone, activities dated across the data date without being shown as in progress, and activities whose duration spans several reporting cycles. Take all of this from the date columns, not from where the bars sit.

**The total float column, where shown.** Many prints carry it, and where present it upgrades the review substantially — it is the only genuinely numeric evidence on the page, so use it, but read it against the bars rather than on its own:

- **Negative float** means the plan is late against its own logic or constraints — but only on rows that are ordinary activities, which a print cannot confirm. List the activities and the values and ask what drives each, in the clarification section. It becomes a required correction once the activity types are known.
- **Cross-check the column against the red bars.** They should agree. Red bars carrying positive float mean criticality is being driven by something other than float — a constraint, a different longest-path setting, or a stale print. Activities at zero or negative float that are *not* coloured mean the view is filtered or the driving-path logic excludes them. Either mismatch is worth a direct question, and neither is visible if you look at only one of the two.
- **Repeated identical float values** are the strongest single pattern a float column offers. Within a run of activities they mean the chain hangs off one constraint or open end upstream. Across *unrelated* parts of the schedule — different trades, different areas, different years — the same value appearing again and again points to one constraint driving everything, or to a float calculation hitting a limit. Count roughly how much of the schedule carries the value, name two or three places it appears that have nothing to do with each other, and ask what sets it. One question here can explain a third of a schedule, so look for it before writing anything else about float.

Rule out the dull explanation first. A level-of-effort or hammock row takes its dates from the activities it spans, so its float simply mirrors theirs. If one of the rows sharing the value also spans the others in date range, or carries a large share of the cost, it is probably that row — and the repeat is arithmetic rather than a finding. Check that before asking what constraint drives the block, and leave such a row out of the float list rather than presenting it alongside real activities.
- **High float** relative to the remaining duration to completion — the check 7 logic applies here, phrased as a question rather than a score.
- **Float on completed or in-progress work** that still shows a value is normally a sign the schedule was not recalculated at the stated data date.

The column's weakness: it is only as current as the last recalculation. If the data date is absent or the print appears stale, say that the float values cannot be relied on and treat them as indicative.

**Rows that may not be ordinary activities.** A print rarely shows an activity-type column, so level-of-effort, hammock and summary rows look exactly like real work. Float on those rows is an artefact of what they span, and cost on them is usually the budget for everything underneath. Treating either as a defect produces a confident finding about a row that was never meant to behave like an activity.

Suspect a row is not ordinary work when it carries a large share of the project cost, spans a period covering many other activities, or shows float that contradicts the detail activities finishing on its own dates. Where that is the case, ask what activity type it is — as a question, in the clarification section — and do not report its float or its cost as a finding until the answer comes back. Say in the scope note that activity types were not shown, so milestones, level-of-effort and summary rows could not be excluded from any count.

**Milestone structure.** List the milestones and check they are unambiguous. Two milestones on the same date — a project finish and a handover, say — raise the question of which one the contract measures and which one carries liquidated damages. Milestones with no visible predecessor, or a completion milestone that is not the end of the critical path, are worth the same question.

**Completeness.** Compare the scope on the page against what the contract requires: commissioning, pre-commissioning, handover, punch list clearance, documentation, demobilisation. These are the activities most often absent from a print, and their absence is a finding even without the XER.

### What cannot be determined from a print

Say this explicitly, as its own section, naming the checks: relationship type mix, lag and lead counts, hard constraints, dangling logic, calendar assignments, resource and cost loading, and date integrity against the data date. Each needs the underlying file.

Where no total float column is shown, add float and criticality to that list — and note that its absence from a submitted schedule is itself worth raising, since it is the column that would let the client judge criticality independently.

### When the print shows neither logic nor float

A print with no relationship lines and no total float column — an activity table of ID, name, duration, dates and cost beside a bare bar chart — is the common case, not the exception. It still supports a useful review, but say at the outset which of the two is missing, because that is a comment on the submission and not only a limit on the review: a schedule issued for approval without float, constraints or logic cannot be verified by the party being asked to approve it.

What still works: criticality distribution where critical bars are coloured, date and duration clustering, completeness against the contract scope, cost loading, milestone structure, and whatever the page furniture declares about the data date, revision and filter. Build the review from those and be explicit that the rest is unverifiable.

Close the report with a request for the underlying file, offered as a ladder so that a refusal at the top does not end the conversation. In most print-only reviews this is the single most valuable line in the document.

1. The native XER export. Everything in this review becomes verifiable.
2. Failing that, a tabular export carrying activity ID, durations, total float, constraint type and date, calendar, and predecessor and successor links with their types and lags.
3. Failing that — and this is the cheapest of the three for the contractor — a reprint of the same schedule with the columns identified in the layout audit added. Refer to that finding by its number rather than restating the list, and ask for the required tier as a standing layout for future submissions, not a one-off.

Say which of the findings above each option would settle. A request that explains what it unlocks is answered far more often than one that does not, and option three is a layout change rather than a data release, so it rarely needs anyone's approval.

## Report structure

Use this structure:

```
# Schedule review — [project name]
Data date: [date] | Activities: [n] | Relationships: [n] | Reviewed: [date]

## What the print shows
[Print reviews only. Two or three lines: which columns are present, which are
absent by tier, and one sentence on what that limits. The detail goes in the
numbered layout finding below; this is the orientation for a reader who will
not reach it.]

## Required corrections
[Numbered. Each one: what was found, how many, which activity IDs, what it
means, and the correction requested. These are defects that make the schedule
unreliable — they are not matters of preference.]

## Recommendations
[Numbered. Improvements that would make the schedule more useful but do not
by themselves invalidate it.]

## Items to clarify
[Findings from checks 7 and 9, and anything that cannot be judged from the
file alone. Phrased as questions to the contractor, not as defects.]

## Not covered by this review
[Checks that need a baseline or status data, and anything requiring scope
knowledge. For a print review this section is mandatory and names every
check that the format prevented, followed by the request for the export.]
```

Do not issue an overall verdict and do not grade the schedule as acceptable or unacceptable. A schedule is almost never rejected outright, and a summary judgement invites an argument about the judgement instead of work on the findings. The output of a review is a list of things to fix, ordered so the contractor knows what must change and what is advice.

When one finding undermines the basis for the others, say so where it is found and again where they are listed. A schedule with no data date, or one whose latest actual date is far behind its print date, has not been recalculated — so its float values, its criticality and its date relationships are all products of a stale calculation and may change entirely on the next run. State plainly that the remaining findings are provisional on recalculation. Without that sentence a reader will act on a finding that recalculation would erase, and the review loses credibility for a reason that was visible on page one.

Quantify staleness only against a date that belongs to the schedule. A stated data date, or a print timestamp generated by the party who issued the schedule, supports the arithmetic: "the latest actual is 13-Feb-15 against a print dated 24-Feb-16, so it appears not to have been updated for about a year" is a finding.

A print timestamp on its own does not. It records when somebody pressed print, which may be years after the schedule was current and may be the reviewer's own copy. Where no data date is shown, the finding is that no data date is shown — ask for it, and stop there. Do not compute an elapsed period from the print stamp and present it as neglect; a long gap there is as likely to mean an old file was reprinted as that work stopped.

A contradiction visible in the table's own dates is a required correction, not a question. If an activity starts before the approval, drawing or predecessor it depends on finishes, that is demonstrable from the printed columns alone — nothing is inferred, nothing can be rebutted, and the contractor must either correct the dates or explain the dependency. Findings of that kind outrank anything that rests on assuming what the contract covers, what a section ought to contain, or what a bar colour means. Put them first.

Sort the required corrections by consequence, not by check number. An open end on the critical path matters more than twenty open ends in a completed section.

Keep a required correction out of the recommendations list and vice versa. The distinction is the whole point of the split: if everything is required, nothing is.

## Who reads the output

The person running this review is usually a scheduler or reviewer acting on a request from a project manager. The output is not for them alone — it travels to the contractor's scheduler, who must act on it, and to managers on both sides, who have no scheduling background. Write for that whole chain.

In practice:

- Explain the consequence in project terms, not scheduling terms. Not "dangling logic on the fabrication chain" but "the finish of this activity is not linked to anything, so delay to it will not show anywhere in the completion date".
- Use the technical term and then say what it means, once. Do not assume DCMA, free float, out-of-sequence or finish-to-start are shared vocabulary.
- Keep each finding self-contained. It will be quoted on its own in an email, separated from the rest of the report.
- Number findings and keep the numbering and wording stable across revisions, so a comment raised last month can be referenced, answered and closed rather than re-argued from scratch.
- Anything the contractor is expected to answer carries a number. No unnumbered tail of "minor points" or "also worth a question": an item worth asking is worth tracking, and an item not worth tracking should be cut rather than demoted. If the count is the reason for the tail, drop the weakest items instead — the limit exists to force that choice, not to be worked around.

**Keep the document short enough to be acted on.** Aim for no more than eight numbered findings. Beyond that, readers stop ranking and start skimming, and the strong findings are diluted by the weak ones. If the review produces more, merge related ones, demote the minor ones into a single closing paragraph, or drop them.

Where two parts of the schedule contradict each other, decide which one is the anomaly and say so. Do not write a finding in branches — "if A is correct then B is wrong, and if B is correct then A is wrong" — because it reads as the reviewer declining to look, and it hands the contractor two things to answer where there is one problem. Weigh the evidence: a handful of dates that disagree with the bulk of the schedule are the suspect ones, and progress reported on work whose predecessors have not started is suspect regardless of what date it carries. Name that reading, give the reason, and ask for confirmation. If the evidence genuinely does not favour either side, the finding is the contradiction itself, stated once.

The same applies across findings: once a defect is identified, do not write a separate finding that reasons from the data it invalidates. Fold it into the one finding and say what follows from it.

The cap is a reason to prioritise, never a reason to bundle. One numbered finding covers one root cause: if its sub-points would each need a different answer from the contractor, it is several findings pretending to be one, and the contractor will answer the easiest sub-point and ignore the rest. Split them and drop the weakest, or group them only where a single explanation would resolve all of them.

Two specific habits to avoid: raising the same underlying problem twice under different headings, and ending the document with low-stakes presentation advice. A review that closes on print layout reads as a review that ran out of substance.

Where a finding cannot be stated without speculating about how the contractor organised their scope, leave it out. "This may sit under another subcontractor" invites a one-line dismissal that costs more than the finding was worth.

**Defensibility over coverage.** The reviewer's name is on these comments and the contractor's scheduler will challenge anything weak. A finding that is probably right but hard to substantiate from the file costs more than it is worth: one successfully rebutted comment weakens every other comment in the document. Drop it, or demote it to a question in the clarification section. Never include a finding to make the review look thorough.

## Writing the findings

Write each finding so it can be pasted into a letter to the contractor. That means:

- State the fact, then the consequence, then the request. Not an opinion about the contractor.
- Name the activity IDs. If there are more than ten, give ten and the total count.
- Distinguish "this breaks the schedule" from "this is not best practice". A reviewer who calls everything critical gets ignored on the things that are.

**Example:**

Weak: "Too many constraints are used, which is bad practice."

Better: "Seven activities carry Mandatory Finish constraints (A1020, A1340, A2150, and four others). A mandatory constraint overrides network logic, so delay to any predecessor will not show as slip on these dates — the schedule will continue to report on time while the work is late. Replace with Finish On or Before, or provide the contractual basis for each date."

## What not to do

- Do not estimate or round counts. Every number in the report comes from the file.
- Do not report a threshold as failed without listing the activities behind it.
- Do not comment on whether durations are realistic. That needs scope knowledge the file does not contain — say it is outside a structural review.
- If the file is missing a table the check needs, say the check could not be run. Do not infer.
