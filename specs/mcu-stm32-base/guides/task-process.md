# Trellis Task Process

> Keep tasks small enough that PRD, implementation, verification, and finish notes stay readable.

## Plan Phase

Create the task before deep implementation work. Start implementation only after the PRD and context files are ready.

A useful PRD usually includes:

- Goal
- Current State / What We Know
- Requirements
- Acceptance Criteria
- Definition of Done
- Out of Scope
- Technical Approach
- Implementation Plan
- Research References

## Research Artifacts

Put durable research under the task directory, not only in chat:

```text
.trellis/tasks/<task>/research/<topic>.md
```

Use one file per topic. Keep raw private/vendor material out of Git; summarize and link to local reference paths when needed.

## Context Files

Use `implement.jsonl` for implementation context and `check.jsonl` for verification context.

- `implement.jsonl`: PRD, relevant specs, research notes, design docs, and source files needed to implement
- `check.jsonl`: verification specs, acceptance criteria, test docs, pitfalls, and files to inspect after implementation

Do not paste large code bodies into these files. List paths and short purpose notes.

## Finish Phase

Before finishing:

- Run the narrowest reliable verification and record output.
- Check whether docs, source comments, version macros, or `.trellis/spec/` need updates.
- Record new repeatable pitfalls locally first; feed back only generalized rules.
- Separate AI-touched files from unrelated dirty files before commit.
- If `.trellis/config.yaml` uses `session_auto_commit: false`, add session notes manually or through the project workflow before closing.

## Trellis Config Notes

Projects may document:

- `session_commit_message`
- `max_journal_lines`
- `session_auto_commit`
- lifecycle hooks such as `after_create`, `after_start`, `after_finish`, and `after_archive`
- package scopes and `default_package`
- Codex `dispatch_mode`

Generic hooks should receive `TASK_JSON_PATH`. Hook failures should warn and leave the core task operation usable.

