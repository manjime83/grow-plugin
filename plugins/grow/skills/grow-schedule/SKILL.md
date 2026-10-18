---
name: grow-schedule
description: Schedule /grow to run on its own, as a scheduled task in the user's AI client.
---

Set up a scheduled task in the user's AI client that runs `/grow`, so the user gets their next task without asking. Grow stores no schedule: the scheduled task lives in the user's client. Ask one question per message, each with a concrete proposal the user can accept with "yes" ("I'd run it on weekdays at 8:00. Right?").

Safety rules, which no standing decision or request overrides: draft but never send; never buy, move money, enter credentials or create accounts; cancel or merge nothing without the user's ok; one pick at a time.

## 1. Check Grow is ready

Call `list_goals`. With no goals, stop and suggest `/grow-goal` first: Next Task only picks work that serves a goal.

Call `list_standing_decisions`. With none, tell the user to run `/grow` by hand once first: the first run asks for their rules (time budget, language, where tasks come from), and a scheduled run has no one to answer. Offer to continue anyway if they'd rather set the schedule now.

## 2. Choose when

Propose when Next Task should run, from the user's words or their time-budget rule ("weekdays at 8:00", "every day at 7:30"). Each run makes exactly one pick, so "two tasks a day" means two schedules ("8:00 and 14:00"). A schedule can carry steering words that narrow its pick, such as `/grow from the MRR goal`. This step is done when the user accepts the times and the prompt of each schedule.

## 3. Create the scheduled tasks

For each schedule, create a scheduled task whose prompt is `/grow`, followed by its steering words when it has some. If you have a tool that creates scheduled tasks, use it. Otherwise, give the user the exact prompt and times, and the steps to create it in their client.

## 4. Allow unattended runs

Tell the user to set Grow's write tools to "Always allow" in their client's connector settings (in Claude: the Grow connector's tool permissions). A scheduled run has no one to approve a tool call, so a tool that still asks is silently denied and the run can't record its pick or journal. Skip this step when the client doesn't let them set it.

## 5. Report

Tell the user, in a few lines, each schedule with its times and prompt, and that they can still run `/grow` by hand whenever they want. To stop a schedule, they delete or pause the scheduled task in their client.
