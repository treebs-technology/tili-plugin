# Recurring report

A placed campaign gets one recurring task in **your host's scheduler** (scheduled tasks, tasks,
routines — whatever this host offers). Each run analyses the latest report and saves **your read
of it** — headline, story, its warnings in your words, channel notes, suggestions — in tili, next
to the report (its numbers, checks and suggestions are never changed). The report has no story of
its own: your headline, story, warnings and channel notes are its headline band — your whole read
in one place — and your suggestions sit next to its own. The scheduled chat is
only your workspace: it ends with one line and the report link, never the report itself. tili
does not run the task; `get_reporting_schedule` says when it should exist and run.

## The schedule

`get_reporting_schedule` (`campaignId`) returns:

| Field | Use |
| --- | --- |
| `state` | What to do with the task (table below) |
| `startOn` | First run day (the day after delivery starts; today once it has) |
| `stopOn` | Last run day (the final report, after late orders stop arriving); `null` = open-ended |
| `runAt` + `timeZone` | Local time the daily report is ready; run then, never earlier |
| `reportReady` | Yesterday is reported: today's run has new numbers |
| `interpreted` | Your read of the latest report is saved (a repeated run has nothing to do) |
| `reportUrl` | The report's page in the web app — the link you post |
| `task.id` / `task.title` | The task's stable id and name — find, update or delete it by them |

| `state` | Task |
| --- | --- |
| `not_started` | None — nothing is placed yet. Delete one if it exists |
| `scheduled` | Exists; first run on `startOn` |
| `active` | Exists; runs daily |
| `paused` | Exists; runs say it is paused, briefly |
| `wrapping_up` | Exists; delivery is over, late orders still count until `stopOn` |
| `ended` | Delete it |

## Set it up

When: right after a channel is placed (the config card is approved and the channel is `LIVE`,
including a future start), and on any performance read where the task should exist but does not.

1. `get_reporting_schedule`. `not_started` / `ended` → no task; stop.
2. Look for `task.id` in your scheduler. Exactly one task per campaign — never a second.
3. Create it (or update it when `startOn`, `stopOn` or `runAt` changed): `task.id`, `task.title`,
   daily at `runAt` in `timeZone`, first run `startOn`, last run `stopOn` when the scheduler takes
   an end date (otherwise the run deletes itself). The task's prompt is self-contained (a run has
   no memory of this chat):

   > Campaign report for tili campaign {campaignId}. Load the campaign-optimize skill and follow
   > references/recurring-report.md, "Each run", for campaignId {campaignId}. Write to the
   > advertiser in {language}.

4. Tell the advertiser in plain words: when the first report comes, how often, until when, and
   where they read it ("From 2 October, every morning at 9, your campaign report in tili gets my
   reading of the numbers and next steps, until a week after the ads stop; I'll post the link
   here"). Daily is the default; they may ask for weekly (same `runAt`, their weekday) or for
   none — then delete it.
5. No scheduler in this host → say updates come when they ask; do not promise a schedule.

## Keep it in step

After `pause_channel`, `resume_channel`, `end_channel`, `cancel_campaign`, a schedule or end-date
change, or a go back that unplaces the channel: `get_reporting_schedule` again and create, update
or delete the task to match. Each run also re-checks, so a change made elsewhere (web app, Ads
Manager) still lands at the next run.

## Each run

1. `get_reporting_schedule` (`campaignId`).
   - `ended` → if anything was delivered and the last report is not `interpreted`, write the final
     briefing (steps 3–5, the whole flight); then delete this task (`task.id`). Cancelled before
     delivery → delete; one line.
   - `not_started` → delete this task; one line that nothing is running.
   - Otherwise, if `startOn`, `stopOn` or `runAt` changed, update this task.
2. `reportReady: false` or `interpreted: true` → nothing new; no message. Stop.
3. **Analyse** (in this chat, not shown to anyone):
   - `get_campaign_performance` (`campaignId`) — the latest report: goal, budget, confidence,
     checks, the report's own suggestions. With `date` = the previous report day: your read of it
     (`interpretation`) and whether the advertiser accepted your suggestions.
   - `get_campaign_history` (`campaignId`, last 14 days) — the last 7 days against the 7 before,
     yesterday against the days before it, and the days around any change.
   - `get_channel_performance` / `get_ad_set_performance` when a finding needs the ad set or ad
     behind it.
   - Answer three questions: **what happened** (against the target and the previous period),
     **why** (what changed, which ad set or creative drove it, what the data can't tell),
     **what now** (a concrete step, or "nothing to do" when that is the honest answer).
4. **Write your read** — a sharp colleague, not a dashboard, in plain words
   (`advertiser-language.md`), about the channels you run (the report's own story covers the
   rest):
   - **Lead with the answer:** "You're slightly behind target, mostly because of one group of
     people", not "This report covers…".
   - **Explain a metric the first time**, in the same sentence ("what you get back per €1 spent").
   - **Say why carefully:** "probably", "most likely" when the data suggests but does not prove.
   - **Every figure is a number from the report**, written as the tools return it (rounded is
     fine). Do not calculate new figures (no own sums, averages or "1 in 75"); the 7-day `delta`
     and `recent` / `previous` values are the report's comparison. The save refuses any other
     figure and says where it is.
   - **Say when nothing is wrong**, and when it is too early: "Too early to tell: 6 sales so far."
   - A tracking problem (checks) comes first.
   - Headline ≤ 120 characters; story ≤ 5 sentences; each channel note ≤ 2 sentences.
   - **Warnings:** one for every `WARN` check in `get_campaign_performance` `checks` (`checkId` =
     its `id`): what it means for them and what to do, e.g. "Sales from the last 36 hours are not
     in yet — reconnect your shop if this lasts." No warnings → `[]`. Warnings live only there:
     the story does not repeat them (the save refuses a story sentence that does).
5. **Suggestions** — from the data you analysed, up to 3 of your own:
   - Do not repeat one of the report's suggestions; if you agree with one, say so in the story.
   - Each: `title`, `reason` (≤ 200 characters, with its numbers), `risk` (SAFE small and
     reversible / BOLD bigger move / ADVICE nothing on Meta), `evidence` (label → report figure),
     and `action` when Meta can carry it out (the same changes as `propose_channel_adjustment`,
     Meta ids from `get_channel_performance`).
   - None worth making → none; "nothing to do" in the story.
6. `save_report_interpretation` (`campaignId`, `reportDate` from the report you read, `headline`,
   `story`, `warnings`, `channelNotes`, `suggestions`). Saving again replaces your read. Refused →
   fix exactly what it names and save again (a newer report → back to step 3).
7. **Post in this chat** one line and the link, nothing more: "Your report for 12 October is
   ready: {reportUrl}". Not the briefing, not the numbers.
8. By state:
   - `paused` → the briefing says since when it is paused and that nothing is spending; suggest
     resuming only if the data supports it.
   - `wrapping_up` → the briefing says the ads stopped on `deliveryEndsOn` and late orders still
     count until `stopOn`. On the last run, the whole flight.

## After the advertiser decides

They accept or reject your suggestions on the report page, next to the report's own. An accepted
suggestion with an `action` is carried out like one of the report's, in a normal chat with them:
`propose_channel_adjustment` with its `suggestionId` → they approve the card.

## Rules

- DO NOT create a second task for a campaign — find it by `task.id` first
- DO NOT run before `runAt`, before `startOn`, or after `ended`
- DO NOT post the briefing or numbers in the scheduled chat — only the line and `reportUrl`
- DO NOT put a figure in your read that is not a number from the report
- DO NOT propose cards, change budgets or pause anything in a scheduled run — suggestions go on
  the report; changes happen only after the advertiser accepts them
- ONLY delete the task when the schedule says so (`ended` / `not_started`) or the advertiser asks
