# Firmware Reuse Guide

> Use this guide before adding new helpers, constants, parser logic, storage code, command APIs, or duplicated module patterns.

## Search First

Before creating new code or names, search for:

- existing macros, enums, and error codes
- existing parser helpers, CRC/checksum utilities, endian helpers
- existing command service patterns
- existing task queues and buffer pools
- existing storage/mount wrappers
- existing BSP abstractions for the same peripheral
- existing tests or docs that already define the behavior

Prefer project-local conventions even if they are imperfect. Do not introduce a second style unless the task explicitly includes a migration.

## Reuse Targets

- Shared protocol utilities should stay HAL-free.
- Device command paths should reuse the owning command service.
- Storage users should reuse the shared mount owner.
- UI actions should reuse public command/data APIs.
- New drivers should reuse nearby BSP initialization, timeout, and error patterns.

## Abstraction Criteria

Add an abstraction only when it removes real duplication or clarifies ownership. Avoid generic helpers that hide:

- buffer lifetime
- task/ISR context
- memory placement
- timeout behavior
- protocol validation order

## Batch Change Checklist

- [ ] Search results were checked before adding new symbols.
- [ ] New helper names follow the existing module prefix.
- [ ] Reused code keeps ownership and context boundaries visible.
- [ ] Tests or checks cover at least one reused path and one new path.
- [ ] Docs/specs mention the shared helper if future tasks should use it.

