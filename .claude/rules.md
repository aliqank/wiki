# Working rules

## Language and style
- Communicate with the user in **English**. The user may write in Russian — always reply in English.
- Preserve technical and domain terms from the BRD as-is: `Fleet Owner`, `Service Work Request`, `Justification`, `Usage Rate`, `On-demand BP (Showcase)`, etc.
- Keep responses concise and structured; use tables and code blocks where they improve readability.

## File and folder naming
- All file and folder names must be in **English**, in **snake_case** format.
- For dated files: `YYYY-MM-DD - topic_name.md` (e.g. `2026-05-06 - db_schema_v3.md`).
- For session memory files: `YYYY-MM-DD HH-MM - topic_name.md` (e.g. `2026-05-06 02-32 - files_and_folders_renaming.md`).

## Working with materials
- Always open `materials.md` in the project root first.
- Use the descriptions to identify the needed files and read them selectively.
- Do a broad repository scan only if `materials.md` lacks sufficient information.
- When adding new meaningful files, update `materials.md`.

## Artifact workflow
- Raw notes and intermediate working materials go into `working_docs/`, inside the relevant month/day folders.
- After collaborative analytical work, the final formatted result is created as a **`.md` file** in `claude_results/`.
- Do not move or publish anything to `wiki/` without an explicit user command.
- `wiki/` is reserved only for final, approved, **ready-for-development** artifacts.

## Saving results to `claude_results/`
- Save all formatted analytical results in `claude_results/` as `.md` files.
- File name: `YYYY-MM-DD - <topic_name>.md` — English, snake_case (e.g. `2026-05-06 - db_schema_v3.md`).
- At the top of each new analytical document include:
  - **Created:** `YYYY-MM-DD HH:MM`
  - **Last updated:** `YYYY-MM-DD HH:MM`
  - **Author:** `Telman Nurzhanov (SA)`
- Before creating files in other formats (`.bpmn`, `.xml`, code, export files, etc.) — ask the user first.

## Working with requirements
- When analyzing requirements, consider both final artifacts and AS-IS context from transcriptions and working notes.
- Explicitly record open questions, assumptions, and TBD items.
- Preferred structure for analytical documents: `context → AS-IS → TO-BE → entities → roles → integrations → open questions`.
- Avoid hardcoded solutions: business parameters, timeouts, limits, and reference data should be configurable via Admin Panel where possible.

## Working with `wiki/`
- `wiki/` is not a draft workspace.
- Before moving anything to `wiki/`, a prepared artifact must exist in `claude_results/` or another explicitly agreed final source.
- Any update to `wiki/` is only performed on a separate explicit user command.
