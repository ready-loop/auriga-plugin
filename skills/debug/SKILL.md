---
schema_version: "1.0.0"
name: debug
description: Investigate conversation and automation failures
tools:
  - mcp__auriga-mcp__conversation_debug
  - mcp__auriga-mcp__turn_artifacts
  - mcp__auriga-mcp__automation_list
  - mcp__auriga-mcp__ion_read
  - mcp__auriga-mcp__ion_list
---

Investigate failures in Auriga conversations and automation runs.

## Debugging workflow

### Step 1: Get the overview

Call `conversation_debug(conversation_id="...")` to get:

- Conversation metadata (personality, debug mode, user skills,
  status, flow, result_summary)
- `turn_summaries`: per-turn exit_code, elapsed_ms, cost_usd,
  error_snippet, flow name
- `turns`: event timeline grouped by turn (request_id), with
  model, cost, token usage, and errors
- `events`: flat event timeline (same data, ungrouped)

For automations, pass `automation_id` instead. Use
`automation_list` to find automation IDs.

### Step 2: Identify the failing turn

Look at `turn_summaries` for non-zero `exit_code` or non-empty
`error_snippet`. For multi-turn conversations, the failure is
often in the last turn.

### Step 3: Drill into the failing turn

Call `turn_artifacts(conversation_id="...", turn=N)` to read
the full log files. Focus on:

- `stderr.log` -- Python tracebacks, import errors, crashes
- `server.log` -- platform-level errors during execution
- `stdout.log` -- NDJSON events emitted by the flow
- `summary.json` -- structured metadata (exit_code, elapsed_ms,
  flow, cost_usd, cost_breakdown, flow_params, error_snippet)

You can select specific files:
`turn_artifacts(..., artifacts="stderr.log,server.log")`

Also available (when debug mode was on):
- `trace.jsonl` -- OTel trace spans

## Event types

- `prompt` -- user message (data: text)
- `status` -- status update (data: message)
- `done` -- turn completed (data: text, model, elapsed_ms,
  cost_usd, usage)
- `error` -- error occurred (data: message, error_code, detail)

## Common error patterns

### Exit code non-zero (sandbox crash)
Python tracebacks appear in stderr.log. Common causes:
- Missing package in sandbox venv
- Missing API keys (secrets not configured)
- Import errors in skill scripts

### Exit code -9 (SIGKILL)
Sandbox was OOM-killed. The skill loads too much data or
a large model into memory.

### Flow timed out
Sandbox exceeded the execution timeout. Look for infinite
loops, blocking I/O, or overly large LLM calls.

### Flow processing failed
Check stderr.log for the real error. The top-level error
message is generic; the details are in the artifacts.

### Missing OAuth scopes
The skill requires Google OAuth scopes the user hasn't
authorized. Run the skill interactively with `run_skill`
to trigger the authorization card.

### Missing skills
"Skill not found" means the automation's requires weren't
resolved. Recreate the automation so requires are populated
from the skill manifest.

### Credit limit reached
User exhausted their credit quota. Check the error event
for a billing URL.
