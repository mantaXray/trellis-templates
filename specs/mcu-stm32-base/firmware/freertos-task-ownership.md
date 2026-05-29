# FreeRTOS Task And Ownership Rules

> Use this guide when changing tasks, queues, shared buffers, watchdog heartbeats, blocking calls, app services, storage calls, or ISR/task handoff.

## Task Boundaries

- Name the owning task for each UART parser, command service, storage path, display snapshot, and shared state object.
- Bootstrap/default tasks should not become long-lived hidden owners unless documented.
- Application services called from task context should not silently become task entry points.
- Public APIs should state whether they are task-context only, ISR-safe, or forbidden from ISR.

## Queue And Buffer Ownership

- Queues should move bounded descriptors, not large arrays.
- A descriptor should identify source, command, length, buffer ID, and backing pointer when lifetime matters.
- The consumer releases or recycles owned buffers after processing.
- Do not store parser output pointers or queue buffer pointers beyond their documented lifetime.
- Avoid copying large display snapshots, spectra, logs, or file buffers onto task stacks.

## Blocking And Context Rules

- File-system calls, protocol parsing, logging, UART reconfiguration, and retry loops run in task context.
- ISR code must not wait, lock, print, parse large frames, call storage APIs, or call blocking RTOS APIs.
- Blocking waits need bounded timeouts and must not stall unrelated UART, UI, watchdog, or storage paths.
- Shared state needs a single writer, lock, critical section, or snapshot boundary.

## Watchdog Rules

- Do not mask or disable hardware watchdog long term to hide bring-up problems.
- Adding a required heartbeat source requires updating source IDs, config, heartbeat calls, diagnostics, and documentation together.
- Hardware watchdog refresh should confirm the handle/instance is initialized before use.
- Before diagnosing a missing response as RX loss, check whether watchdog reset or heartbeat masking explains the symptom.

## Storage Ownership

- File-system work stays out of ISR context.
- Multiple services on the same logical drive should share the project-owned mount object.
- Services should not mount private objects for a shared drive or unmount shared volumes from service deinit.
- File rotation must close/sync the current file before opening the next one when filesystem lock limits are tight.

## Review Checklist

- [ ] Every blocking call has an owning task and timeout.
- [ ] Every queue payload has producer, consumer, and buffer lifetime rules.
- [ ] Required watchdog-source changes update code and docs together.
- [ ] Storage calls run in task context and use shared mount ownership.
- [ ] ISR paths remain lightweight and only hand off descriptors or flags.

