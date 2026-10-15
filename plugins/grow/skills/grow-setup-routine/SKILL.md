---
name: grow-setup-routine
description: Set up a Grow routine through an interview, with the goals it pursues and its rules, and schedule it.
disable-model-invocation: true
---

Run an interview: one question per message, each with a concrete proposal the user can accept with "yes" ("I'd call it Daily and have it pursue Clear the backlog first, then Grow MRR. Right?"). Nothing is saved until the user approves the whole plan in step 4.

A routine is a named way of choosing what to work on next. It pursues an ordered list of the user's goals, and its rules are its standing decisions. Every run of it is `/grow-next-task`, which settles the previous pick and then picks, breaks down and prepares exactly one task.

Safety rules, which no standing decision or request overrides: draft but never send; never buy, move money, enter credentials or create accounts; cancel or merge nothing without the user's ok; one pick at a time.

## 1. Start from their words

Take the routine's name and purpose from the words after `/grow-setup-routine`, or ask what the routine should help them move forward. Call `list_routines`. If an existing routine already covers it, say so and offer to update that routine instead of creating a second one: load it with `get_routine` and `list_standing_decisions`, interview only for what changes, and save only the changes in step 5. This step is done when you have a short name ("Daily", "Revenue") and know whether it's a new routine or an update.

## 2. Choose the goals

Call `list_goals` and propose the active goals the routine should pursue, most important first. The order is the routine's priority between its goals. A goal can sit in more than one routine, but two routines that pursue separate goals never compete for a pick. When the work the user describes has no goal yet, stop and suggest `/grow-setup-goal` first: everything a routine picks serves a goal. This step is done when the user accepts the goals and their order.

## 3. Capture the rules

Ask for the rules every run must follow. Each rule becomes one standing decision, written in the user's words. Cover, one question at a time and proposing a typical answer:

- **Time budget**: how long a run's task should take, per kind of day ("Weekdays about 30 minutes, weekends 1 to 2 hours").
- **Language and report style**: the language of the run's report, and its shape ("Spanish, three short lines: the task, the first step, one question").
- **Where tasks come from**: when the user's work lives in an external tracker, which part of it to pick from, reached through the user's own connector ("Linear team OMA, excluding project Road to 10K MRR"). Picked issues become shadow tasks in Grow.
- **Focus and day rules**: anything that narrows a pick ("Only personal tasks on weekends", "Favour the MRR goal this month", "Always tell me what else I could do").

A rule can't break a safety rule. "Give me two tasks a day" becomes two runs a day (two schedules in step 6), never two picks in one run. This step is done when the user has nothing to add.

## 4. Approve the plan

Show the whole plan in one message: the routine's name, its goals in order, and its rules as a numbered list. Apply the user's changes and show it again until they approve it.

## 5. Save

1. `save_routine` with `name` and `goalIds` in order. For an update, pass the routine's `id` and only what changed; `goalIds` replaces the whole list.
2. `save_standing_decision` once per rule, with the saved routine's `id` as `routineId`, the rule as `text`, and `createdBy: "user"`. For an update, change a rule by passing its `id` and the new `text`, and retire a dropped rule by passing its `id` and `active: false`.

When a call is refused, its message names the problem: fix that field and call again, asking the user only when the fix needs their decision. The step is done when the routine and every rule in the approved plan have been returned by a save call.

## 6. Offer a schedule

Offer to run the routine on a schedule, with a Claude scheduled task whose prompt is `/grow-next-task <routine name>`, for example `/grow-next-task Daily`. Propose times that fit its rules ("weekdays at 8:00"); a rule like "two tasks a day" means two schedules. If you have a tool that creates scheduled tasks, create it once the user agrees. Otherwise, give the user the prompt and times to set up themselves. The user can also run `/grow-next-task` by hand whenever they want, scheduled or not.

## 7. Allow unattended runs

When the user accepts a schedule, tell them to set Grow's write tools to "Always allow" in their client's connector settings (in Claude: the Grow connector's tool permissions). A scheduled run has no one to approve a tool call, so a tool that still asks is silently denied and the run can't record its pick, journal or handoff. Skip this step when the client doesn't let them set it.

## 8. Report

Tell the user, in a few lines, what was saved: the routine with its goals in order, its rules, and its schedule, or how to run it by hand. Name anything that couldn't be saved, with the reason.
