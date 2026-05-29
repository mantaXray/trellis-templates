# MCU Trellis Guides Index

> Use this index for process and reasoning guides. Firmware implementation rules live under `../firmware/`.

## When To Read

| Situation | Read |
|---|---|
| Starting any firmware task | [`../firmware/index.md`](../firmware/index.md) |
| Mapping ISR, queue, task, parser, display, storage, or command paths | [`firmware-data-flow-guide.md`](firmware-data-flow-guide.md) |
| Adding helpers, constants, drivers, parsers, or repeated logic | [`firmware-reuse-guide.md`](firmware-reuse-guide.md) |
| Creating PRD, research notes, implement/check context, or finishing a task | [`task-process.md`](task-process.md) |
| Coordinating Claude/Codex or single-agent task/code phases | [`claude-codex-collaboration.md`](claude-codex-collaboration.md) |
| Debugging by observed symptoms | [`debug-symptom-index.md`](debug-symptom-index.md) |

> Bootstrap 流程、坑点反哺、外部知识库等模板维护文档不再随 spec 发到下游项目，移到了模板仓库 `docs/` 目录（仅维护者参考）。

## Default Task Start

1. Read `../firmware/index.md`.
2. Search `../firmware/pitfalls-index.md` for relevant pitfall IDs.
3. If the task crosses modules or contexts, write a short data-flow map before editing.
4. Search existing code/specs before adding new helpers or constants.

