# Optional UI And Display Rules

> Use this guide only when the project has a UI/display layer, generated UI files, display snapshots, touch/display BSP work, or display-facing memory. Projects without UI/display can skip it.

## Display Snapshot Ownership

- Name the writer of each display snapshot or UI-facing data block.
- UI code should read through public snapshot APIs, not parser internals or task-private headers.
- UI code should not write snapshots to simulate a device response.
- Device writes should go through command APIs owned by the relevant task or service.

## Generated UI Code

- Generated UI files may be overwritten by GUI tooling or vendor display builders.
- Keep durable product logic in command, data, task, or service modules when practical.
- If generated event code must call a command API, keep the generated call minimal and keep validation in the command layer.
- Generated-code diffs should be minimal or documented as intentional.

## Display Memory And Timing

- Display buffers, draw buffers, and UI snapshots need explicit memory placement.
- Do not put unrelated storage, protocol, or log state into display-reserved memory simply because it is large.
- Flush callbacks, touch input, tick timing, GPU/LTDC-style resources, and cache/MPU settings need owner notes when changed.
- Avoid copying large snapshots onto UI task stacks; use read APIs, bounded buffers, or summary fields.

## Display-Backed Command Rules

Typical path:

```text
UI event -> command API -> device TX -> parser/task update -> display snapshot -> UI read API
```

- Successful command submission means TX accepted or queued; it does not prove the device accepted or executed the command.
- Readbacks come from snapshots only after parser/task code updates them.
- UI behavior changes that affect device commands should update command contracts and tests.

## Review Checklist

- [ ] UI code reads only public snapshot APIs.
- [ ] Device writes go through the owning command service.
- [ ] Generated UI code remains thin.
- [ ] Display buffers have memory/cache ownership notes.
- [ ] Command contracts and docs are updated when a page adds device behavior.
