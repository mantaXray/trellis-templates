# Project Context

> Fill this file after installing the template. Keep it short; it is the first place an agent checks before touching firmware.

## Project Facts

- Project code:
- MCU family and exact part:
- Board or hardware revision:
- Toolchain: IAR Embedded Workbench for Arm, STM32CubeIDE/GCC, or other project-specific build system
- CubeMX model file:
- Middleware in use: HAL, FreeRTOS, filesystem, UI/display, USB, TCP/IP, or none
- Main firmware entry points:
- Build command:

## Documentation Ownership

- Project-level design docs live under `doc/`.
- Trellis shared rules live under `.trellis/spec/`.
- Local reference inputs live under ignored folders such as `ref_docs/` or `ref_doc/`.
- When local reference material becomes implementation guidance, distill it into design docs, Trellis specs, source comments, or tests instead of committing raw private/vendor dumps.

## Before Coding

- Confirm the active toolchain and generated-code boundary.
- Confirm which task owns each UART, storage path, display path, and shared buffer.
- Confirm whether the change affects version macros, protocols, memory placement, or project docs.
