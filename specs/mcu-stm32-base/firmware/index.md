# Firmware Spec Index

> MCU 固件任务开始前先读本入口。**默认只读 Tier 1 的 2 个文件**，其他全部按变更类型触发，不要一次性灌完整个 spec。

## Tier 1 — 任务开始必读（轻量，~70 行）

| # | 文件 | 作用 |
|---|---|---|
| 1 | [`pitfalls-index.md`](pitfalls-index.md) | 按任务类型/症状快速定位高风险坑点 ID |
| 2 | [`project-context.md`](project-context.md) | 项目 MCU、工具链、中间件、文档归属 |

读完 Tier 1 后，agent 决定接下来要不要读 Tier 2 的哪些文件。**不要默认全读**。

## Tier 2 — 按变更类型触发

| 如果任务涉及 | 读取 |
|---|---|
| `pitfalls-index.md` 命中了具体条目 | [`pitfalls.md`](pitfalls.md) |
| 新增/重命名/移动源文件、IDE 工程成员同步 | [`directory-structure.md`](directory-structure.md) |
| 写或修改 C/C++ 代码、注释、API、外设逻辑 | [`coding-standard.md`](coding-standard.md) |
| 完成任务、选择验证命令、Markdown 同步检查 | [`verification.md`](verification.md) |
| Git/SVN 提交、版本号 bump、提交信息规范 | [`version-control.md`](version-control.md) |
| 大 buffer、linker、map、cache、DMA buffer、显示/存储内存 | [`memory-placement.md`](memory-placement.md) |
| UART DMA、idle callback、协议解析、TX complete、baud 切换 | [`uart-dma-protocol.md`](uart-dma-protocol.md) |
| FreeRTOS task、queue、ISR、watchdog、共享状态、文件系统上下文 | [`freertos-task-ownership.md`](freertos-task-ownership.md) |
| UI/显示、显示快照、生成 UI 文件、显示触发设备命令（不用 UI 的项目跳过） | [`ui-display.md`](ui-display.md) |
| 文件系统、Flash/EEPROM/SD/eMMC/SPI flash、日志、append-only 记录、持久化（无持久化的项目跳过） | [`storage-persistence.md`](storage-persistence.md) |
| 公开 command API、主机/UI/debug 触发设备命令、可选硬件 feature gate | [`command-contracts.md`](command-contracts.md) |

## Pre-Development Checklist

- [ ] 当前任务涉及的模块已在 `pitfalls-index.md` 中查过
- [ ] PRD / Implementation Plan 已把相关 pitfall 转成检查项
- [ ] 跨 ISR / queue / task / parser / display / storage / command 的路径已写清楚
- [ ] 新增 `.c` 文件时同步 IAR `.ewp` 或 CubeIDE 工程文件
- [ ] 涉及 DMA / cache / RTOS / 协议解析时，明确 buffer 所有权和边界条件
- [ ] 涉及固件行为变化时，计划同步版本宏、设计文档和验证命令

## Quality Check

- [ ] 实现后重新对照相关 pitfall 条目
- [ ] 编译/检查命令已记录到 Git commit message 或任务 check 记录
- [ ] 发现可复发的新坑时，先写入项目本地 pitfall；可泛化时反哺模板
- [ ] 行为、架构、构建命令或版本规则变化已同步到 `doc/` 或 `.trellis/spec/`
