# Firmware Directory Structure

> Use this guide when adding files, moving modules, editing CubeMX output, or updating IDE project membership.

## Generated Code Boundaries

CubeMX-managed files and IDE-generated setup should keep durable product logic inside `USER CODE BEGIN/END` blocks unless the task explicitly changes the CubeMX model. Generated-code diffs should be minimal or documented as intentional.

Prefer small project-owned modules over adding business logic directly to generated initialization files.

## Common Module Areas

Use the project's existing layout first. For new STM32-style projects, these areas are typical:

- `Core/`: CubeMX startup, HAL init, interrupts, and generated glue
- `Drivers/`, `Middlewares/`: vendor code; avoid style-only edits
- `BSP/`: board support and hardware abstraction
- `User/App/Task/`: FreeRTOS task entry points
- `User/App/Data/`: public snapshots or shared app data APIs
- `User/App/Command/`: device command APIs and TX ownership
- `User/App/Service/<domain>/`: small app services called from task context
- `User/Protocol/`: protocol parsers/builders that should stay vendor-library free

Do not introduce nested `Inc/` and `Src/` directories inside a domain unless the project already uses that pattern.

## IDE Project Synchronization

When adding source or include paths, update the active IDE/build project:

- IAR: `.ewp`, `.ewd`, `.eww`, linker `.icf`, and startup assembly when relevant
- STM32CubeIDE/GCC: `.project`, `.cproject`, `Makefile`, linker script, and workspace files when relevant

New `.c` files are not done until the active build system includes them.

## Review Checklist

- [ ] Generated-file changes stay in `USER CODE BEGIN/END` or the `.ioc` model change is intentional.
- [ ] New files are in the existing active module area, not an abandoned legacy path.
- [ ] IAR/CubeIDE project membership includes new sources and include paths.
- [ ] Vendor directories were not reformatted or refactored as incidental cleanup.
