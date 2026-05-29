# MCU Firmware Pitfalls

> 本文件记录可跨 MCU 项目复用的坑点规则。只写可预防、可检查、可泛化的内容；不要写项目代号、客户信息、具体寄存器编号、现场日期或专有协议字段。

## How To Use This File

- 任务开始前先读 [`pitfalls-index.md`](pitfalls-index.md)，再跳到相关条目。
- PRD / Implementation Plan 中引用 pitfall ID，例如 `PIT-DMA-001`。
- 如果某条规则与项目实际冲突，以项目本地 spec 为准，并在任务收尾时更新项目 spec。
- 新 lesson 先进入项目本地 pitfall；跨项目可复用后再反哺本模板。
- 本文件保持精简；当单个章节规则数量超过 10 条时，把该章节拆到 `firmware/pitfalls-<category>.md` 同级文件，并更新本文件章节链接和 [`pitfalls-index.md`](pitfalls-index.md)。

---

## Repository And Tooling

### PIT-REPO-001 - Git/SVN 边界必须先定清楚

AI/Trellis 配置、`doc/`、`.gitignore` 只上 Git，不上 SVN；源码、工程文件、发布产物按项目约定双轨保留。提交 SVN 前必须显式检查 `svn status`，不要把 `.trellis/`、`.claude/`、`.codex/`、`doc/`、构建产物混进去。

### PIT-REPO-002 - Ignore 规则只忽略本地状态，不忽略共享 spec

`.trellis/spec/`、`.trellis/tasks/` 是团队共享上下文，不应被 Git ignore。可以忽略 `.trellis/workspace/*/journal-*.md`、`.trellis/.runtime/`、`.trellis/.backup-*/` 等本地状态。

### PIT-REPO-003 - 发布产物例外要放在 ignore 之后

如果用 `*.hex`、`*.bin`、`*.out` 忽略构建泄漏，必须先写 ignore，再写 `!Bin/*.hex`、`!Bin/*.bin`、`!Bin/*.out`。顺序反了会导致发布产物继续被忽略。

---

## CubeMX, IAR, And Active Paths

### PIT-PROJ-001 - 新增源文件必须同步工程成员

新增 `.c` 文件后，除了 Git/SVN add，还要确认 IAR `.ewp` 或 CubeIDE `.cproject` 已加入该文件。否则本机命令行可能没覆盖到，或同事打开 IDE 后缺模块。

### PIT-PROJ-002 - CubeMX 生成区只改 USER CODE 区块

CubeMX 管理的文件只能在 `USER CODE` 区块内写项目代码。生成区外的手工改动会在重新生成时丢失；需要持久化的架构约定应写入 `.ioc`、工程配置或项目 spec。

### PIT-PROJ-003 - 克隆项目后先清 active path 和项目代号

从旧项目复制时，必须检查 `.ewp/.eww`、`.ioc`、`.project/.cproject`、`*.code-workspace`、`Bin/` 发布命名和 SVN log 前缀。不要让旧项目路径或代号残留到新项目。

---

## UART, DMA, Cache, And Protocols

### PIT-DMA-001 - DMA 接收必须覆盖分片、多帧、跨帧

UART/DMA 回调可能一次只到半帧，也可能一次到多帧，还可能一帧跨多次回调。解析器必须有累积 buffer、长度上限和重同步策略，不能假设“一次回调一帧”。

### PIT-DMA-002 - Cache MCU 上 DMA buffer 要明确内存区和维护规则

STM32H7 等带 cache 的 MCU 中，DMA buffer 必须记录所在内存区、对齐、clean/invalidate 时机和 producer/consumer 所有权。低端无 cache MCU 可以简化，但不要把无 cache 假设写进通用驱动。

### PIT-PROTO-001 - 协议解析先验证再使用 payload

使用 payload 字段前必须先验证 delimiter、声明长度、最大 payload、CRC/checksum、字节序和命令合法性。任何失败路径都要丢弃或重同步，不能半解析半执行业务。

### PIT-PROTO-002 - 寄存器映射变更默认新增不破坏

Modbus 或主机协议寄存器变更时，默认保留旧编号和语义；新增功能使用未占用区域。破坏性变更必须有 PRD 明确授权，并同步主机端协议文档。

---

## RTOS, Watchdog, And Shared State

### PIT-RTOS-001 - 长循环和等待路径必须让出调度或喂狗

FreeRTOS 任务中的长循环、轮询等待、批处理和重试逻辑要有 `osDelay()`、超时退出或喂狗策略。不能用忙等拖死同优先级任务。

### PIT-RTOS-002 - ISR 只做短路径，跨上下文用 FromISR API

ISR 内不要做阻塞、长计算、文件系统操作或复杂协议解析。需要通知任务时使用符合中断优先级限制的 `*FromISR` API，并检查 queue/semaphore handle 非 `NULL`。

### PIT-RTOS-003 - 共享状态要有单写者或快照边界

传感器数据、协议状态、UI 展示状态等共享结构要明确单写者、锁、临界区或快照复制。不要让多个任务直接读写同一结构再靠“访问很快”碰运气。

---

## Memory Placement

### PIT-MEM-001 - 大 buffer 必须说明放置区域

大数组、DMA buffer、帧缓存、日志缓存要说明放在 DTCM、SRAM、AXI、SDRAM 或其他区域。涉及 DMA 时确认该区域可被外设访问；涉及 cache 时确认维护策略。

### PIT-MEM-002 - 内存变化要看 map 文件或等价输出

新增大对象、UI/display buffer、文件系统 cache、协议缓存后，要检查 map 文件或编译输出中的 RAM/flash 占用。不要只看 C 文件大小判断风险。

---

## Sensors, Display, Storage

### PIT-PERIPH-001 - I2C 地址统一按 7-bit 输入

传感器驱动 API 默认接收 7-bit I2C 地址，在 HAL 调用前由驱动内部左移。不要在调用层混用 7-bit 和 8-bit 地址。

### PIT-PERIPH-002 - Flash 写入必须有页边界和失败策略

Flash 持久化要明确页范围、对齐、擦写粒度、掉电风险、失败重试或回滚策略。不要把校准参数和运行日志写进未文档化地址。

### PIT-UIFS-001 - UI/display 和 storage 要串行化访问边界

UI 对象更新应集中在 UI 线程或受控回调中；文件系统、Flash、日志轮转或其他持久化访问要明确串行化策略。不要让多个任务同时操作同一文件、同一持久化区域或同一 UI 对象。

---

## Documentation Sync

### PIT-DOC-001 - 行为变化必须同步设计基线或 spec

如果代码改变了状态机、协议、寄存器映射、版本规则、构建命令或目录约定，任务收尾时必须更新 `doc/软件设计说明_*.md` 或 `.trellis/spec/`。不能只靠 commit message 记录。

### PIT-DOC-002 - 本地 pitfall 和模板 pitfall 分层维护

项目本地 pitfall 记录现场事实和项目特化经验；模板 pitfall 只保留跨项目可复用规则。反哺前必须去掉项目代号、客户信息、具体寄存器编号和一次性现象。
