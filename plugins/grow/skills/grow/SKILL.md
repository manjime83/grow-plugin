---
name: grow
description: Settle the previous pick, then pick, break down and prepare the next task across your ranked goals. Words after /grow steer the pick.
---

Run Next Task for the user. The words after `/grow` are the user's **steering words** ("from project Launch", "something easy", "I have 20 minutes"): they narrow this one pick and leave the standing decisions as they are.

A scheduled task can start this run with nobody there to answer. So, after the first-run interview, the run keeps moving: wherever a step needs the user's answer, take the safe default that step names and make it the report's question. The user answers after reading the report (step 12).

Safety rules, which no standing decision or steering words override: draft but never send; never buy, move money, enter credentials or create accounts; nothing cancelled or merged without the user's ok; one pick at a time.

## 1. Read the standing decisions

Call `list_standing_decisions`. Wherever a standing decision speaks, it overrides this script's defaults.

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

1. **done**: it's done, or the user says they finished it. Set its Grow status to `done` with `save_task` when it isn't yet.
2. **rescheduled**: the user moved it, in a journal entry or in their words.
3. **continued**: the user reported progress since that run. It's today's pick. When the steering words point elsewhere, record it `rescheduled` instead.
4. **skipped**: the safe default, when the evidence shows none of these. Its question is "Did you get to it? I recorded it as skipped."

Hold the outcome for step 11. This step is done when the previous pick has an outcome.

## 5. Spot a task runs keep passing over

Look for a task recorded `skipped` in three or more of the recent runs, and take the most skipped. Propose a verdict, keep or cancel, with a one-line reason from its history: that's its keep-or-cancel question. The task stays as it is until the user answers, and today's pick comes from the other tasks. This step is done when you hold one such task with its verdict, or know there's none.

## 6. Pick one task

Call `list_eligible_tasks`. When a standing decision names an external tracker, also read its open issues through Claude's connector for that tracker, within the decision's limits. When the previous pick continues, it's today's pick: compare it with the candidates only to write the reason.

Otherwise choose exactly one task, weighing these together, the first counting most:

1. the goal's rank, and its pace: how far its deadline is against the work left (call `get_goal` for the goals among the candidates, at most three);
2. due date and priority;
3. a size that fits today's time budget;
4. variety: a different project from the previous pick;
5. age: older tasks first.

Pick a `backlog` task only when no `todo` or `in_progress` task fits.

Apply the steering words to this pick. When they contradict a standing decision, ask the user which to follow before picking. The safe default is the standing decision, with the question "Your words said X, but your standing decision says Y. Which should win next time?"

An external issue you pick becomes a shadow task: `save_task` with `externalSystem`, `externalId`, `externalUrl`, a `summary`, status `todo`, and the `goalId` of the goal it serves (or a `projectId` under it).

Write the **reason**: one line on why this task and not the next two candidates, naming them. This step is done when you hold one task id and its reason. With nothing eligible, go to step 11 with no pick and suggest `/grow-goal` or new tasks in the report.

## 7. Break down today's slice

Size today's slice to the time budget for today's day type. Split it into three to seven steps the user can start without thinking: each names the first concrete action and the file, link or person it touches. Start from the task's handoff when it has one. This step is done when the steps fit the budget.

## 8. Prepare

Do as much of the slice as you safely can before the user sits down: research with sources, drafts, scripts, checklists, comparison tables. Keep each draft where the user will find it: a draft in their email or docs through a connector, or in the report. This step is done when every step you could advance without the user has been advanced.

## 9. Write the journal and handoff

Record what this run did with `log_task_entry`, passing the run's `id` as `runId`: `progress` for what you prepared, `decision` for a choice made, `blocker` for what stops the work. Then rewrite the task's handoff with `set_task_handoff`: where the task stands now and the first step of today's slice. This step is done when the pick has this run's entries and a new handoff.

## 10. Choose the report's question

Ask at most one question, the first that applies:

1. "Run `/grow` by hand once so I can learn your standing decisions." (step 3);
2. the contradiction between steering words and a standing decision (step 6);
3. the keep-or-cancel question (step 5);
4. the settle question about the previous pick (step 4).

## 11. Finish the run

Call `finish_run` with the run's `id`, a summary of a few plain sentences, and one outcome per task: the previous pick's outcome from step 4, and today's pick as `continued` (or `done` if the run finished it). When today's pick is the previous pick, it gets the one outcome `continued`. This is the last Grow call of the run.

## 12. Report

Write a short report in the user's language and report style, readable in under a minute:

- today's task, its goal, and the reason;
- the steps of today's slice;
- what you prepared, and where it is;
- the question from step 10, if any.

When you have a notification tool, send the report through it. Your final message is the full report.

When the user answers the question in this chat, act on it: on "cancel", set the task's status to `canceled` with `save_task`; on a corrected settle, log it with `log_task_entry`; on a standing decision they want changed, update it with `save_standing_decision`.
