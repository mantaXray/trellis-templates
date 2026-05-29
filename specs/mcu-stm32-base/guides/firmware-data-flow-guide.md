# Firmware Data Flow Guide

> Use this guide before changing behavior that crosses ISR, queue, task, parser, command, display, storage, or host boundaries.

## Write The Path First

Before editing code, name the complete path in the PRD or implementation notes:

```text
event/source -> ISR or callback -> queue/descriptor -> owning task
    -> parser/service -> shared state or storage -> command/display/readback
```

For each hop, record:

- owner: ISR, task, service, parser, or UI
- data shape: bytes, descriptor, parsed struct, snapshot, file record
- lifetime: copied, borrowed, static, DMA-owned, task-owned
- failure path: drop, retry, resync, timeout, error code
- verification: test, build, map check, hardware smoke, or skipped reason

## Boundary Rules

- ISR code should only do bounded work and hand off descriptors or flags.
- Queues should not move large buffers by value.
- Parsers should not expose pointers beyond their documented lifetime.
- UI/display code should read snapshots and send commands; it should not reach into parser internals.
- Storage code should run in task context and own mount/file lifetime.
- Command APIs should define whether success means queued, transmitted, acknowledged, or executed.

## Review Checklist

- [ ] The full data path is written down before implementation.
- [ ] Every shared buffer has producer, consumer, and release rules.
- [ ] Every context switch has a bounded payload and failure path.
- [ ] Tests or checks cover good/base/bad paths, not only the happy path.
- [ ] The matching firmware spec is updated when the boundary becomes a stable project rule.

