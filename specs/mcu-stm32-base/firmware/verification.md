# Firmware Verification

> Run the narrowest reliable check that proves the changed behavior. Do not claim completion without fresh output.

## General Rule

Use the project's normal build first when available. If IAR is not callable from the shell, use focused host tests, parser tests, static searches, map-file checks, `git diff --check`, and clear skipped-build notes.

## Required Checks By Change Type

| Change type | Verification |
|---|---|
| Parser or protocol logic | Focused parser regression covering good, bad, boundary, and resync cases |
| UART DMA handling | Fragmented input, multiple frames in one callback, and partial-frame carry-over |
| New source files | Active IAR `.ewp` or CubeIDE `.cproject` membership check |
| CubeMX generated files | User logic remains in `USER CODE BEGIN/END`, or `.ioc` change is intentional |
| Memory/linker/buffer changes | Map-file or build-memory output review |
| FreeRTOS/ISR changes | Context, timeout, queue lifetime, and watchdog impact review |
| Storage/persistence changes | Mount or ownership boundary, failure path, and persistence smoke checks |
| Docs-only Trellis changes | `git diff --check`, `git status`, and relevant reference search |
| Git/SVN policy changes | `git status --porcelain=v1`, `git check-ignore`, and `svn status` if SVN exists |

## Markdown Sync Check

Every source-code change needs a documentation sync decision. Update Markdown in the same change set when paths, behavior, architecture, style rules, build commands, version rules, or repeatable pitfalls change.

Common targets:

- `doc/软件设计说明_*.md`
- `doc/优化方案_*.md`
- `.trellis/spec/firmware/`
- `.trellis/spec/guides/`
- project protocol documents
- source comments

## Cleanup

- Run cleanup scripts in dry-run mode first.
- Keep generated build output out of Git unless the project explicitly versions release artifacts.
- Read and write Markdown and source files as UTF-8 when Chinese text is involved.
