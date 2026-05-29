# Optional Storage And Persistence Contracts

> Use this guide only when the project has filesystem, flash, EEPROM, SD/eMMC/SPI flash, file logs, append-only records, timestamps, or other persistence services. Projects without persistent storage can skip it.

## Mount Ownership

- Define one owner for each logical drive.
- Application services on the same logical drive should share the project-owned filesystem or mount object.
- Do not create service-private mounts for the same logical drive unless the architecture explicitly allows it.
- Service deinit should not unmount a shared volume that other services may still use.

## Disk I/O Contracts

- Validate physical drive, buffer pointer, sector/count range, and command before touching hardware.
- Chunk transfers when the lower driver has count or size limits.
- Initialization failure should return a clear not-ready/error state without hardfault, hang, or hidden retry loop.
- Timestamp code should define its RTC/fallback behavior.
- Storage APIs should return explicit error codes instead of blocking indefinitely.

## Memory And Context

- Storage state is usually non-DMA application state unless the driver says otherwise.
- Keep file buffers, filesystem objects, and log state out of scarce stack/default RAM when large.
- File-system calls run in task context, not ISR context.
- Multiple writers need serialization, a single owner, or a documented queue.

## Append-Only Records

For event history, logs, and fault records:

- Use fixed-size records or an explicit header with length and checksum.
- On boot, recover from damaged tails by truncating partial records and scanning valid IDs.
- Do not reuse old monotonic IDs after damaged-tail recovery.
- Skip bad records during query instead of copying corrupt data to callers.
- Pair optional blob files with record IDs and remove orphans during cleanup.

## Tests Required

- Mount failure and missing-media paths return errors without watchdog reset.
- Read/write smoke checks compare data after close/reopen.
- Append-only recovery handles partial tail, bad checksum, and orphan blobs.
- New `0:` or default-drive users share the mount owner.
- Map-file or memory report is checked when large storage objects are added.
