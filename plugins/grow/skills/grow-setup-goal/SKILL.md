---
name: grow-setup-goal
description: Set up a SMART goal in Grow through an interview, with its area, measurement, projects and first tasks.
disable-model-invocation: true
---

Run an interview: one question per message, each with a concrete proposal the user can accept with "yes" ("I'd track MRR in USD, from $1,200 today to $5,000 by 31 March. Right?"). Nothing is saved until the user approves the whole plan in step 7.

Safety rules, which nothing the user or a setting says overrides: draft but never send; never buy, move money, enter credentials or create accounts; cancel or merge nothing without the user's ok; one pick at a time.

## 1. Start from their words

Take the goal from the words after `/grow-setup-goal`, or ask what they want to achieve. Call `list_areas` and `list_goals`. If an existing goal already covers it, say so and offer to update that goal instead of creating a second one: load it with `get_goal`, interview only for what changes, and save only the changed fields with `save_goal` and its `id`. This step is done when you know what the user wants to achieve and whether it's a new goal or an update.

## 2. Make it SMART

Push the vague goal until every field below is settled:

- **Title**: the outcome in a few words ("Grow MRR to $5k").
- **Metric**: name, unit, baseline (today's value), target and direction (`up` or `down`). All five go together. When the user doesn't know the baseline, leave it open and measure it in step 3, or take their best estimate.
- **Deadline**: a calendar date. Turn "by summer" into a date and confirm it.
- **Why**: why it matters, in the user's own words. Runs quote it back when motivation dips, so capture their phrasing.
- **Done when**: a plain-language criterion ("Exam passed"). It's the measure when no honest metric exists. With a metric, add one only when something besides the number marks completion.

Ongoing work becomes a goal that measures it, so every task serves a goal:

- "Clear my backlog" → open issues in their Linear backlog, 120 → 50 by 31 December, down.
- "Keep the house running" → overdue home chores, 12 → 2 by 30 November, down.
- "Stay on top of email" → unread inbox threads older than a week, 80 → 0 by 15 November, down.

## 3. Choose how it's measured

Pick the method that fits what's available:

- `connector`: a connector the user's client already has can read the number. Check which connectors you can see.
- `ask`: Grow asks the user for the value.
- `infer`: estimate it from the work logged on the goal's tasks.

Write the **measurement recipe** as plain instructions any agent can follow without this conversation: the source, the query and the arithmetic ("Use the Stripe connector and sum the amounts of active monthly subscriptions, in USD"). Set a **fallback cadence** for asking the user ("weekly"): for `ask` it's how often to ask, and for `connector` and `infer` it's how often to ask when the method can't produce a value. When the baseline is still open, measure it now with the recipe.

## 4. Confirm the area

Infer the area from what the user said: an existing area from `list_areas`, or a new one with a short name (Home, Finance, Business, Health, Career). State it in one line; the step is done when the user accepts it or names another.

## 5. Group the work

Create a project when several tasks form one body of work ("Launch the course", "Kitchen renovation"). Small, independent tasks sit directly under the goal. Give each project a status: `planned` or `in_progress` for work starting now, `backlog` for later. Only tasks in planned or in-progress projects get picked.

## 6. Propose the first tasks

Propose three to seven tasks that move the metric or the done-when, the first one startable today. Each has:

- a summary and a one- or two-line description;
- a parent: a project from step 5, or the goal;
- a status (`todo` for ready now, `backlog` for later);
- a priority (1 urgent, 2 high, 3 medium, 4 low, or 0 none);
- an estimate (1, 2, 3, 5 or 8 points) when the size is clear;
- a due date only when the user names a real one.

## 7. Approve the plan

Show the whole plan in one message: area, goal with every field from steps 2 and 3, projects, and tasks grouped under their parent. Apply the user's changes and show it again until they approve it.

## 8. Save

Save in this order, passing each returned `id` to the next call:

1. `save_area` with `name`, only for a new area.
2. `save_goal` with `areaId` and the fields from steps 2 and 3.
3. `save_project` with `goalId`, `name` and `status`, once per project.
4. `save_task` once per task, with `projectId` or `goalId`, `summary`, `description`, `status`, `priority`, and `estimate` and `dueDate` when set.

When a call is refused, its message names the problem: fix that field and call again, asking the user only when the fix needs their decision. The step is done when every area, goal, project and task in the approved plan has been returned by a save call.

## 9. Report

Tell the user, in a few lines, what was saved: the goal with its metric or done-when and deadline, how it will be measured, and each project and task under its parent. Name anything that couldn't be saved, with the reason.
