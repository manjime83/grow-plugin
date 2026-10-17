---
name: grow
description: Settle the previous pick, then pick, break down and prepare the next task across your ranked goals. Words after /grow steer the pick.
---

Run Next Task for the user. The words after `/grow` are the user's **steering words** ("from project Launch", "something easy", "I have 20 minutes"): they narrow this one pick and leave the standing decisions as they are.

A scheduled task can start this run with nobody there to answer. So, after the first-run interview, the run keeps moving: wherever a step needs the user's answer, take the safe default that step names and make it the report's question. The user answers after reading the report (step 14).

Safety rules, which no standing decision or steering words override: draft but never send; never buy, move money, enter credentials or create accounts; nothing cancelled or merged without the user's ok; one pick at a time.

## 1. Read the standing decisions

Call `list_standing_decisions`. The user's standing decisions take priority over this script's defaults: wherever one speaks, follow it. Only the safety rules rank above them.

## 2. Start the run

Call `start_run` with the steering words as `request`, or with no `request` when there are none. When it's refused because a run is still open, the refusal names that run: close it with `finish_run`, a summary saying it was left open and an empty `outcomes` list, then call `start_run` again. This step is done when you hold an open run's id.

## 3. First run: interview for the standing decisions

Skip this step when step 1 returned any standing decisions.

Interview the user, one question per message, each with a concrete proposal they can accept with "yes" ("I'd plan about 30 minutes on weekdays and 1–2 hours at weekends. Right?"):

- **Time budget** per day type.
- **Language and report style**.
- **Where tasks come from**: only Grow, or also an external tracker their Claude has a connector for ("Linear team OMA, excluding project Road to 10K MRR"), and which goal its issues serve.
- **Focus and day rules** ("only personal tasks on weekends", "favour MRR this month").

Save each accepted answer with `save_standing_decision`, `text` in the user's words and `createdBy: "user"`. This step is done when every topic has a saved decision or the user declined it. Then tell them `/grow-schedule` can run `/grow` on its own, and that two tasks a day means two schedules. Carry on: they ran `/grow` for a task.

When a scheduled task started this run, skip the interview and carry on with this script's defaults; the report's question is "Run `/grow` by hand once so I can learn your standing decisions."

## 4. Settle the previous pick

Call `list_recent_runs` with `limit: 10`. The **previous pick** is the task the newest run that recorded a `continued` outcome recorded it for. With none, go to step 5.

Load it with `get_task`. For a shadow task, also read its issue through the tracker's connector. Settle it on the evidence (its status, the issue's status, journal entries newer than that run, and anything the user has said in this chat), taking the first outcome that fits:

1. **done**: it's done, or the user says they finished it. Set its Grow status to `done` with `save_task`, passing the user's date as `today`, when it isn't yet. A recurring task (one with a `recurrence`) comes back as `todo` on its next occurrence when it's marked done, and its journal records each occurrence as "Done for <date>": it's already done when that entry is newer than the run.
2. **rescheduled**: the user moved it, in a journal entry or in their words.
3. **continued**: the user reported progress since that run. It's today's pick. When the steering words point elsewhere, record it `rescheduled` instead.
4. **skipped**: the safe default, when the evidence shows none of these. Its question is "Did you get to it? I recorded it as skipped."

Hold the outcome for step 13. This step is done when the previous pick has an outcome.

## 5. Update the scoreboard

Call `list_active_goals`. Each metric comes with its pace, which holds its latest reading. A metric is **due** when its latest reading is older than its fallback cadence, weekly when it has none. Measure each metric the way its method says:

- `connector`: when you have the connector its recipe names and its latest reading is from an earlier day, follow the recipe and record the value with `record_reading`, `source: "connector"`, the connector's name as `sourceDetail`, and the link or query as `evidence`.
- `infer`: when it's due, estimate it from the work logged on the goal's tasks and record it with `source: "inferred"`, citing those journal entries as `evidence`.
- `ask`, or a `connector` metric you can't read: when it's due, it goes in the **weekly check-in**.

The weekly check-in is one question: the value of each due metric, and a confirm-or-correct for each unconfirmed estimate (call `get_goal` for each active goal with an `infer` metric, and take the readings with `confirmed: false`). Ask it at most once per cadence: skip it when a recent run's summary says it was asked within the cadence.

After recording, call `list_active_goals` again when you recorded anything. When every metric of a goal has a confirmed latest reading whose pace is `met`, propose confirming the goal's done-when: "Your readings meet every target for X. Is it done: <done-when>?" Readings are evidence; only the user's yes achieves the goal.

This step is done when every metric you can read through a connector has a reading from today, every due `infer` metric has a new estimate, and you hold the check-in and the done-when proposal, or know there's none.

## 6. Choose the cleanup question

A run asks at most one **cleanup question**: keep, cancel, or merge one task into another. Call `list_cleanup_questions`. When any are pending, the oldest is this run's cleanup question, still waiting for the user's answer.

Otherwise look in the recent runs for a task that deserves one: recorded `skipped` in three or more of them, the most skipped first. Propose a verdict with a one-line reason from its history: keep it, cancel it, or merge it into another task that covers the same work. Record it with `ask_cleanup_question`, passing `mergeIntoTaskId` for a merge.

The question's task stays as it is until the user answers, and today's pick comes from the other tasks. When another question wins the report (step 12), the cleanup question stays pending for a later run. This step is done when you hold this run's cleanup question, or know there's none.

## 7. Pick one task

Call `list_eligible_tasks` with the user's date as `today`. It lists a recurring task only once its occurrence is due. When a standing decision names an external tracker, also read its open issues through Claude's connector for that tracker, within the decision's limits. When the previous pick continues, it's today's pick: compare it with the candidates only to write the reason.

Otherwise choose exactly one task, weighing these together, the first counting most:

1. the goal's rank, and its pace from step 5: within the goals' rank, favour work that moves a goal whose metrics are `behind`;
2. due date and priority;
3. a size that fits today's time budget;
4. variety: a different project from the previous pick;
5. age: older tasks first.

Pick a `backlog` task only when no `todo` or `in_progress` task fits.

Apply the steering words to this pick. When they contradict a standing decision, ask the user which to follow before picking. The safe default is the standing decision, with the question "Your words said X, but your standing decision says Y. Which should win next time?"

An external issue you pick becomes a shadow task: `save_task` with `externalSystem`, `externalId`, `externalUrl`, a `summary`, status `todo`, and the `goalId` of the goal it serves (or a `projectId` under it).

Write the **reason**: one line on why this task and not the next two candidates, naming them. This step is done when you hold one task id and its reason. With nothing eligible, go to step 13 with no pick and suggest `/grow-goal` or new tasks in the report.

## 8. Break down today's slice

Size today's slice to the time budget for today's day type. Split it into three to seven steps the user can start without thinking: each names the first concrete action and the file, link or person it touches. Start from the task's handoff when it has one.

Call `get_playbook` with the pick's goal id as `goal`. When it returns a `mentor`, that mentor coaches this work: build the steps on its playbook items, the ones more videos support first. When a step needs details the items don't give, such as a script, a template or a number to aim for, call `search_knowledge` with the goal's id as `goal` and what the step needs as `query`. Cite the mentor idea each such step comes from, with the mentor and the video link ("price on the value you create, not your costs: @founder, <url>"). A goal without a mentor runs on your general knowledge; don't mention mentors then. This step is done when the steps fit the budget.

## 9. Prepare

Do as much of the slice as you safely can before the user sits down: research with sources, drafts, scripts, checklists, comparison tables. When the goal has a mentor, search what it teaches for the details a draft or checklist needs, and cite it as in step 8. Keep each draft where the user will find it: a draft in their email or docs through a connector, or in the report. This step is done when every step you could advance without the user has been advanced.

## 10. Write the journal and handoff

Record what this run did with `log_task_entry`, passing the run's `id` as `runId`: `progress` for what you prepared, `decision` for a choice made, `blocker` for what stops the work. Then rewrite the task's handoff with `set_task_handoff`: where the task stands now and the first step of today's slice. This step is done when the pick has this run's entries and a new handoff.

## 11. Propose a standing decision

Look back over the recent runs for a run where the lack of a standing decision cost a pick or wasted effort: a pick the user rescheduled for the same reason twice, a slice too big for the time they had, a task source they had to point out. When you can name that run, propose the standing decision that would have prevented it, in a sentence the user can accept with "yes", with that run as its evidence. When another question wins the report, the proposal waits for a later run that still has the evidence. Next Task improves only through standing decisions; this script stays as written. This step is done when you hold one proposal with its run, or know there's none.

## 12. Choose the report's question

Ask at most one question, the first that applies:

1. "Run `/grow` by hand once so I can learn your standing decisions." (step 3);
2. the contradiction between steering words and a standing decision (step 7);
3. the done-when proposal (step 5);
4. the weekly check-in (step 5);
5. the cleanup question (step 6);
6. the settle question about the previous pick (step 4);
7. the proposed standing decision (step 11).

## 13. Finish the run

Call `finish_run` with the run's `id`, a summary of a few plain sentences that names the question the report asks, and one outcome per task: the previous pick's outcome from step 4, and today's pick as `continued` (or `done` if the run finished it). When today's pick is the previous pick, it gets the one outcome `continued`. This is the last Grow call of the run.

## 14. Report

Write a short report in the user's language and report style, readable in under a minute:

- today's task, its goal, and the reason;
- the steps of today's slice;
- what you prepared, and where it is;
- the question from step 12, if any.

When you have a notification tool, send the report through it. Your final message is the full report.

When the user answers the question in this chat, act on it:

- the done-when proposal: on yes, `close_goal` with `status: "achieved"`;
- the weekly check-in: `record_reading` with `source: "asked"` for each value they give, and `confirm_reading` for each estimate they confirm (with `value` when they correct it);
- a goal they say they missed or gave up: `close_goal` with `status: "missed"` or `"dropped"`;
- a cleanup question: `resolve_cleanup_question` with the question's `id` and `resolution: "ok"` when they accept the verdict, or `"changed"` with their corrected `verdict` (and `mergeIntoTaskId` for a merge), and a `note` when their answer adds anything;
- a corrected settle: log it with `log_task_entry`;
- a proposed standing decision they accept: `save_standing_decision` with `text`, `createdBy: "agent"` and `evidence` naming the run;
- a standing decision they want changed: update it with `save_standing_decision`.
