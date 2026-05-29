# UART DMA And Protocol Boundaries

> Use this guide for UART RX/TX DMA, idle callbacks, TX-complete callbacks, protocol parsers, runtime baud changes, and command services.

## Receive Path Rules

- Never assume a UART idle/Rx event callback contains one complete protocol frame.
- Parsers must tolerate partial frames, multiple frames in one callback, and payloads split across callbacks.
- ISR paths may copy bounded fragments, update lightweight diagnostics, maintain cache, and enqueue descriptors.
- Protocol parsing, logging, UART reconfiguration, storage work, and blocking waits belong in task context.
- Queues should carry descriptors such as source, length, command, buffer ID, and backing pointer; do not queue large protocol arrays by value.
- Do not pass a reusable DMA RX buffer pointer to a task unless ownership prevents reuse until the task releases it.

## Transmit Path Rules

- A HAL-style TX-complete callback is often global; dispatch affected UARTs by instance or link record.
- TX-complete callbacks may release a slot, start the next frame, or set a pending flag.
- TX-complete callbacks must not block, print logs, wait, or reinitialize UARTs.
- Runtime baud-rate changes complete in task context. ISR code records only a pending switch.
- Public command APIs that may queue or start DMA TX are thread-context APIs.
- Blocking chunked sends that need exclusive bus ownership must suppress normal FIFO continuation and track ownership generation.

## Protocol Contracts

- Validate delimiter/header, declared length, maximum payload, CRC/checksum, command ID, and endian conversion before reading payload fields.
- Parser output pointers are valid only for their documented lifetime. Copy data immediately when callers need to retain it.
- Protocol modules should expose clear good/base/bad cases and regression tests for malformed frames.

## Link Separation

- Keep debug/log links, host links, module links, and internal command links separate in documentation.
- Do not route debug text commands through business data queues.
- Floating or optional UART RX lines need a safe idle level when hardware allows it.

## Vendor Library Portability Boundary

- Protocol parsers/builders should stay HAL-free and vendor-library free.
- UI/display, protocol, and business-state code should not call raw UART HAL/DMA functions directly.
- Route TX through command services, RX through generated/Core callback dispatch plus task-owned parsers, and board operations through BSP or narrow wrappers.

## Required Checks

- [ ] Fragmented input, multi-frame input, and partial-frame carry-over are tested or explicitly covered by existing tests.
- [ ] New command APIs submit through an owning command service instead of direct HAL transmit calls.
- [ ] TX-complete dispatch covers the affected link.
- [ ] DMA cache maintenance and buffer ownership are documented.
- [ ] Protocol changes update host tests, protocol docs, or `.trellis/spec/firmware/*` contracts.
