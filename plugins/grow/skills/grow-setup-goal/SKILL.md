---
name: grow-setup-goal
description: Set up a SMART goal in Grow through an interview, with its done-when, scoreboard, projects and first tasks.
disable-model-invocation: true
---

Run an interview: one question per message, each with a concrete proposal the user can accept with "yes" ("I'd track MRR in USD, from $1,200 today to $5,000 by 31 March. Right?"). Nothing is saved until the user approves the whole plan in step 7.

Safety rules, which nothing the user or a setting says overrides: draft but never send; never buy, move money, enter credentials or create accounts; cancel or merge nothing without the user's ok; one pick at a time.

## 1. Start from their words

Take the goal from the words after `/grow-setup-goal`, or ask what they want to achieve. Call `list_areas` and `list_goals`. If an existing goal already covers it, say so and offer to update that goal instead of creating a second one: load it with `get_goal`, interview only for what changes, and save only the changed fields with `save_goal` or `save_metric` and the `id`. A new metric on an existing goal takes its baseline like any other. This step is done when you know what the user wants to achieve and whether it's a new goal or an update.

## 2. Make it SMART

Push the vague goal until every field below is settled:

- **Title**: the outcome in a few words ("Grow MRR to $5k").
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
- `infer`: estimate it from the work logged on the goal's tasks.

Write the **measurement recipe** as plain instructions any agent can follow without this conversation: the source, the query and the arithmetic ("Use the Stripe connector and sum the amounts of active monthly subscriptions, in USD"). Set a **fallback cadence** for asking the user ("weekly"): for `ask` it's how often to ask, and for `connector` and `infer` it's how often to ask when the method can't produce a value.

Then take the **baseline**, today's value: measure it now with the recipe, or take the user's best estimate. It becomes the metric's first reading, so it has to agree with the done-when: today's values show the goal isn't done yet, each target lies beyond its baseline in its direction, and reaching the targets together describes the done-when. When they don't agree, settle the target or the done-when with the user. This step is done when every metric has a recipe, a cadence and a baseline that agrees with the done-when.

## 4. Confirm the area

Infer the area from what the user said: an existing area from `list_areas`, or a new one with a short name (Home, Finance, Business, Health, Career). State it in one line; the step is done when the user accepts it or names another.

## 5. Group the work

Create a project when several tasks form one body of work ("Launch the course", "Kitchen renovation"). Small, independent tasks sit directly under the goal. Give each project a status: `planned` or `in_progress` for work starting now, `backlog` for later. Only tasks in planned or in-progress projects get picked.

## 6. Propose the first tasks

Propose three to seven tasks that move the scoreboard toward the done-when, the first one startable today. Each has:

- a summary and a one- or two-line description;
- a parent: a project from step 5, or the goal;
- a status (`todo` for ready now, `backlog` for later);
- a priority (1 urgent, 2 high, 3 medium, 4 low, or 0 none);
- an estimate (1, 2, 3, 5 or 8 points) when the size is clear;
- a due date only when the user names a real one.

## 7. Approve the plan

Show the whole plan in one message: area; goal with its title, why, deadline and done-when; the scoreboard, each metric with its baseline, target, direction, method, recipe and cadence; projects; and tasks grouped under their parent. Apply the user's changes and show it again until they approve it.

## 8. Save

Save in this order, passing each returned `id` to the next call:

1. `save_area` with `name`, only for a new area.
2. `save_goal` with `areaId`, `title`, `why`, `deadline` and `doneWhen`.
3. `save_metric` once per metric, in scoreboard order, with `goalId`, `name`, `unit`, `direction`, `target`, `measurementMethod`, `measurementRecipe`, `fallbackCadence` and `baseline`.
4. `save_project` with `goalId`, `name` and `status`, once per project.
5. `save_task` once per task, with `projectId` or `goalId`, `summary`, `description`, `status`, `priority`, and `estimate` and `dueDate` when set.

When a call is refused, its message names the problem: fix that field and call again, asking the user only when the fix needs their decision. The step is done when every area, goal, metric, project and task in the approved plan has been returned by a save call.

## 9. Report

Tell the user, in a few lines, what was saved: the goal with its done-when and deadline, the scoreboard with each metric's baseline, target and how it's measured, and each project and task under its parent. Name anything that couldn't be saved, with the reason.
