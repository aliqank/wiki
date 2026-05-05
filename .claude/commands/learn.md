Analyze the current conversation context and save it to a session memory file.

**Steps:**

1. Review the entire conversation: tasks discussed, decisions made, files created or changed, open questions.

2. Define a short session topic (2–5 words in English, snake_case, reflecting the main focus).

   Create a file at `source/working_docs/session_memory/YYYY-MM-DD HH-MM - <topic>.md`, where:
   - Date and time — **Astana timezone (UTC+5)**; use the real current date and time
   - `<topic>` — short snake_case label (e.g. `requirements_registry`, `bpmn_external_fleet`, `ui_filters`, `claude_restructuring`)

3. File structure:

```
# Session context — YYYY-MM-DD

## What we did
- Brief summary of tasks and topics discussed

## Decisions made
- List of decisions, agreements, chosen approaches

## Created / changed files
- File paths and short description of changes

## Open questions and TBD
- Questions left unanswered or requiring follow-up

## Next steps
- What was planned to do next (if discussed)

## Additional context
- Any important information worth remembering in the next session
```

4. If the folder `source/working_docs/session_memory/` does not exist — create it (just create the file at that path, the folder will be created automatically).

5. After creating the file — tell the user the path to the saved file.

**Goal:** after `/clear`, read this file to restore the session context.
