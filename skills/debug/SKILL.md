---
schema_version: "1.0.0"
name: debug
description: Investigate conversation and automation failures
tools:
  - mcp__auriga-mcp__conversation_debug
  - mcp__auriga-mcp__automation_list
  - mcp__auriga-mcp__ion_read
  - mcp__auriga-mcp__ion_list
---

You help users investigate failures in Auriga conversations and
automation runs.

## Debugging a conversation

Use `conversation_debug(conversation_id="...")` to inspect any
conversation. This returns:

- Status and flow type
- Error messages
- Event timeline
- Debug artifacts (stderr, summary.json)

## Debugging an automation

Use `conversation_debug(automation_id="...")` to inspect the
most recent run. Use `automation_list` to find automation IDs.

## Browsing Ion turn artifacts

Each conversation turn produces artifacts in Ion at:
```
/chats/{conversation_id}/files/turns/{turn_number}/
```

Key files:
- `stderr` -- sandbox process stderr (Python tracebacks, etc.)
- `stdout` -- sandbox process stdout
- `summary.json` -- structured execution summary
- `output.json` -- skill script output

Use `ion_list` to browse and `ion_read` to inspect these files.

## Common error patterns

### Missing OAuth scopes
The skill requires Google OAuth scopes that the user hasn't
authorized. Run the skill interactively with `run_skill` to
trigger the authorization card.

### Missing skills
"Skill not found" means the automation's requires weren't
resolved. Recreate the automation so requires are populated
from the skill manifest.

### Sandbox errors
Python tracebacks appear in stderr. Common causes:
- Missing package in sandbox venv
- Missing API keys (secrets not configured)
- Import errors in skill scripts

### Flow processing failed
Check sandbox stderr for the real error. Use
`conversation_debug` to surface it.
