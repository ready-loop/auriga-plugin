---
schema_version: "1.0.0"
name: report
description: File a bug report against Auriga
tools:
  - mcp__auriga-mcp__file_bug_report
---

You help users file bug reports against Auriga without leaving
Claude Code.

## Workflow

1. If the user didn't provide a bug description, ask them to
   describe the issue they encountered.

2. Summarize relevant conversation context -- what the user was
   trying to do, what went wrong, any error messages or unexpected
   behavior observed in the current session.

3. Gather session metadata:
   - Read ~/.claude/sessions/ to find the current session ID by
     matching the current PID or most recently modified file.
   - Note the current working directory.
   - Note the timestamp and basic environment info (OS, shell).

4. Build a condensed transcript of the current session:
   - Summarize the last ~50 messages or key exchanges that show
     the error context.
   - Include user messages, assistant actions, tool calls/results,
     and error messages. Omit large code blocks or binary content.
   - Keep the transcript concise but preserve the debugging context.

5. Call `file_bug_report` with:
   - `summary`: a concise bug title (under 80 chars)
   - `description`: rich description including:
     - What the user was doing
     - Steps to reproduce (if identifiable)
     - Expected vs actual behavior
     - Relevant error messages or conversation context
   - `session_id`: from step 3
   - `cwd`: current working directory
   - `environment`: OS, shell, relevant versions
   - `transcript`: the condensed session transcript from step 4

6. Show the user the Jira issue URL and let them know they can
   open it to attach screenshots or additional files.
