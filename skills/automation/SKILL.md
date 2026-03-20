---
schema_version: "1.0.0"
name: automation
description: Create, manage, and trigger skill automations
tools:
  - mcp__auriga-mcp__automation_create
  - mcp__auriga-mcp__automation_list
  - mcp__auriga-mcp__automation_update
  - mcp__auriga-mcp__automation_delete
  - mcp__auriga-mcp__automation_fire
  - mcp__auriga-mcp__conversation_debug
---

You help users create, manage, and trigger Auriga skill automations.
Automations run skills on a schedule or on demand via webhooks.

## Creating an automation

Every automation requires a `name` and a `skill` (the skill name
in kebab-case). The skill must exist in Ion before creating the
automation. Requires (network, secrets, OAuth scopes, VFS) are
automatically resolved from the skill's SKILL.md manifest.

A webhook is always enabled for on-demand triggering.

### Scheduled automations

For recurring execution, provide `initial_fire_at` (ISO 8601
datetime with timezone) and `cadence` (ISO 8601 duration):

- `PT1H` -- every hour (minimum cadence)
- `P1D` -- daily
- `P1W` -- weekly

Example:
```
automation_create(
    name="Morning Digest",
    skill="daily-digest",
    initial_fire_at="2025-01-15T08:00:00-05:00",
    cadence="P1D",
    timezone="America/New_York"
)
```

### Timezone handling

Read the user's timezone from sysfs before scheduling:

```
tz = ion_read("/sys/user/timezone")
```

If the result is empty or "unknown", ask the user for their
timezone. Otherwise use the stored IANA timezone for the
`timezone` parameter and `initial_fire_at` offset. Confirm
the detected timezone with the user before scheduling.

Always pass `timezone` as an IANA timezone string (e.g.
`America/New_York`, `Europe/London`). The `initial_fire_at`
should include a UTC offset matching the timezone.

When displaying automation times, reference the user's
local timezone.

### Budget controls

Use `daily_budget_usd` to cap spending. The automation is
paused for the day once the budget is reached.

## On-demand triggering

Use `automation_fire(name="...")` to trigger any automation
immediately via its webhook. This creates a new conversation
and returns its ID.

## Checking run history

Use `conversation_debug(automation_id="...")` to inspect the
most recent run of an automation. This shows status, errors,
event timeline, and debug artifacts.

## Managing automations

- `automation_list` -- see all automations with status
- `automation_update` -- modify settings (PUT semantics,
  pass the full desired state)
- `automation_delete` -- remove an automation (past
  conversations are kept)

## Common failure patterns

- "Skill not found": the skill name doesn't match any SKILL.md
  in Ion. Check with `ion_list("/skills/")`.
- Missing OAuth scopes: the skill's `requires.oauth_scopes`
  need user authorization first. Run the skill interactively
  with `run_skill` to trigger the authorization card.
- Sandbox errors: use `conversation_debug` to read stderr
  from the failed run.

## Further reading

Read `auriga://docs/automations` for the full guide.
