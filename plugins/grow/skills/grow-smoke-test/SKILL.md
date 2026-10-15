---
name: grow-smoke-test
description: Test the Grow connector by creating a Smoke test area. Run when the user invokes /grow-smoke-test or a scheduled task names it.
---

Call Grow's `save_area` tool once with `{ "name": "Smoke test" }`. Omit the id to create a new area.

Report the returned area name and id. If the tool fails or the connector isn't connected, report that failure and stop.
Only report success when the tool returns the created area. Each invocation creates a new area, including scheduled runs.

Please after list the current areas to verify that the new area has been created successfully.
