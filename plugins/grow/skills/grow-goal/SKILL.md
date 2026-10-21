---
name: grow-goal
description: Set up a SMART goal in Grow through an interview, with its done-when, scoreboard, projects and first tasks.
---

First call `get_user_settings` and use its saved timezone and date today for all dates and times.

Run an interview: one question per message, each with a concrete proposal the user can accept with "yes" ("I'd track MRR in USD, from $1,200 today to $5,000 by 31 March. Right?"). Nothing is saved until the user approves the whole plan in step 7.

Safety rules, which nothing the user or a setting says overrides: draft but never send; never buy, move money, enter credentials or create accounts; cancel or merge nothing without the user's ok; one pick at a time.

## 1. Start from their words

Take the goal from the words after `/grow-goal`, or ask what they want to achieve. Call `list_goals`. If an existing goal already covers it, say so and offer to update that goal instead of creating a second one: load it with `get_goal`, interview only for what changes, and save only the changed fields with `save_goal` or `save_metric` and the `id`. A new metric on an existing goal takes its baseline like any other. This step is done when you know what the user wants to achieve and whether it's a new goal or an update.

## 2. Make it SMART

Push the vague goal until every field below is settled:

- **Title**: the outcome in a few words ("Grow MRR to $5k").
- **Label**: Life for a personal goal or Work for a professional goal. Ask the user to choose; every goal needs one. Both labels share the same ranking and three active slots.
- **Deadline**: a calendar date. Turn "by summer" into a date and confirm it.
- **Why**: why it matters, in the user's own words. Runs quote it back when motivation dips, so capture their phrasing.
- **Done when**: the plain-language definition of done that every goal has ("MRR at $5k for two months in a row", "Exam passed"). The goal is achieved only when the user confirms it.
- **Scoreboard**: one or more metrics that show progress toward the done-when, each with a name, a unit, a direction (`up` or `down`) and a target. Propose the number the done-when hinges on first, then the few that lead it ("trials per week" ahead of MRR). An exam goal gets a number too, such as a practice-test score.

Ongoing work becomes a goal with a done-when and a metric that measures it, so every task serves a goal:

- "Clear my backlog" → done when 50 or fewer issues are open in their Linear backlog; open issues, down to 50 by 31 December.
- "Keep the house running" → done when no more than 2 home chores are overdue; overdue chores, down to 2 by 30 November.
- "Stay on top of email" → done when no inbox thread is unread for more than a week; such threads, down to 0 by 15 November.

## 3. Measure each metric

For each metric, pick the method that fits what's available:

- `connector`: a connector the user's client already has can read the number. Check which connectors you can see.
- `ask`: Grow asks the user for the value.

Write the **measurement recipe** as plain instructions any agent can follow without this conversation: the source, the query and the arithmetic ("Use the Stripe connector and sum the amounts of active monthly subscriptions, in USD"). Set a **fallback cadence** for asking the user ("weekly"): for `ask` it's how often to ask, and for `connector` it's how often to ask when the connector can't produce a value.

Then take the **baseline**, today's value: measure it now with the recipe, or take the user's best estimate. It becomes the metric's first reading, so it has to agree with the done-when: today's values show the goal isn't done yet, each target lies beyond its baseline in its direction, and reaching the targets together describes the done-when. When they don't agree, settle the target or the done-when with the user. This step is done when every metric has a recipe, a cadence and a baseline that agrees with the done-when.

## 4. Rank it

The user's goals are ranked 1, 2, 3…, and Next Task favours higher-ranked goals. Only the top three open goals are active; the rest wait until they move up. From `list_goals`, show where the goal would sit among their goals and propose a rank ("I'd put it 2nd, after Grow MRR. Right?"). With no other goals it's 1st. When the rank leaves the new goal waiting, or pushes an active goal out of the top three, say which goal waits. The step is done when the user accepts a rank.

## 5. Group the work

Create a project when several tasks form one body of work ("Launch the course", "Kitchen renovation"). Small, independent tasks sit directly under the goal. Give each project a status: `planned` or `in_progress` for work starting now, `backlog` for later. Only tasks in planned or in-progress projects get picked.

## 6. Propose the first tasks

Propose three to seven tasks that move the scoreboard toward the done-when, the first one startable today.

First, the goal's **mentor**: at most one YouTube channel that coaches it. Call `list_mentors`. When one fits the goal, propose it from its title and description ("I'd have @founder coach this goal: they teach pricing for service businesses. Right?"); when none fits, say the goal runs on general knowledge. For an existing goal, `get_goal` shows its current `mentor`: offer to keep it, change it or remove it.

With a mentor, call `get_playbook` with the goal's id as `goal` (for a new goal, `search_knowledge` with the mentor's handle as `mentor`) for what the mentor teaches, and `search_knowledge` again when a task needs details. Base the tasks on what it teaches, and cite the mentor idea each task comes from in its description, with the mentor and the video link ("From @founder: price on the value you create, <url>"). A goal without a mentor runs on your general knowledge; don't mention mentors then.

Each task has:

- a summary and a one- or two-line description;
- a parent: a project from step 5, or the goal;
- a status (`todo` for ready now, `backlog` for later);
- a priority (1 urgent, 2 high, 3 medium, 4 low, or 0 none);
- an estimate (1, 2, 3, 5 or 8 points) when the size is clear;
- a due date only when the user names a real one;
- a recurrence when the work repeats ("publish every Monday", "review ad spend on the 1st"): every N days, every N weeks on chosen weekdays, or every N months. A recurring task is one task that comes back after each occurrence; its due date is its first occurrence.

## 7. Approve the plan

Show the whole plan in one message: the goal with its title, label, why, deadline, done-when, rank and mentor; the scoreboard, each metric with its baseline, target, direction, method, recipe and cadence; projects; and tasks grouped under their parent. Apply the user's changes and show it again until they approve it.

## 8. Save

Save in this order, passing each returned `id` to the next call:

1. `save_goal` with `title`, `label` (`life` or `work`), `why`, `deadline`, `doneWhen` and `rank`, and `mentor` (its handle) when the plan has one. On an update, pass `label` only to change it; pass `mentor` only to change it, or `null` to remove it.
2. `save_metric` once per metric, in scoreboard order, with `goalId`, `name`, `unit`, `direction`, `target`, `measurementMethod`, `measurementRecipe`, `fallbackCadence` and `baseline`.
3. `save_project` with `goalId`, `name` and `status`, once per project.
4. `save_task` once per task, with `projectId` or `goalId`, `summary`, `description`, `status`, `priority`, and `estimate`, `dueDate` and `recurrence` when set (omit `today` to use the saved timezone).

When a call is refused, its message names the problem: fix that field and call again, asking the user only when the fix needs their decision. The step is done when every goal, metric, project and task in the approved plan has been returned by a save call.

## 9. Report

Tell the user, in a few lines, what was saved: the goal with its label, done-when, deadline, rank and mentor, the scoreboard with each metric's baseline, target and how it's measured, and each project and task under its parent. Name anything that couldn't be saved, with the reason.
