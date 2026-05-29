# MCU Pitfalls Index

> 这是坑点知识库的快速入口。任务开始前先按关键词扫本页，再跳到 [`pitfalls.md`](pitfalls.md) 的对应章节。
>
> 当某分类规则数量超过 10 条时，拆到 `firmware/pitfalls-<category>.md` 同级文件；本索引仍保持短小。

## 快速索引

| 如果任务涉及 | 先看 |
|---|---|
| 新项目初始化、Git/SVN、生成物、参考资料 | `PIT-REPO-*` |
| CubeMX、IAR 工程、`.ioc`、`.ewp`、active path | `PIT-PROJ-*` |
| UART、DMA、cache、协议解析 | `PIT-DMA-*`、`PIT-PROTO-*` |
| FreeRTOS、ISR、看门狗、共享状态 | `PIT-RTOS-*` |
| 大数组、SRAM/DTCM/AXI/SDRAM、map 文件 | `PIT-MEM-*` |
| I2C/SPI 传感器、Flash、RTC/backup | `PIT-PERIPH-*` |
| UI/display、日志、文件系统、持久化 | `PIT-UIFS-*` |
| 文档同步、spec 更新、项目坑点反哺 | `PIT-DOC-*` |

## 症状入口

| 症状 | 优先排查 |
|---|---|
| 本地编译通过但同事缺文件 | `.ewp` / `.cproject` 工程成员未同步，见 `PIT-PROJ-001` |
| 串口偶发丢包、粘包、解析错帧 | DMA 分片/多帧/跨帧处理，见 `PIT-DMA-001` |
| H7 类 MCU DMA 数据偶发旧值 | cache clean/invalidate 与内存区域，见 `PIT-DMA-002`、`PIT-MEM-001` |
| 新功能运行一段时间后 watchdog reset | 长循环、任务心跳、阻塞 API，见 `PIT-RTOS-001` |
| SVN 提交混入 AI/Trellis 或构建产物 | Git/SVN 边界和 `svn:ignore`，见 `PIT-REPO-001` |
| 文档和代码越做越不一致 | 任务收尾缺少 spec/doc sync，见 `PIT-DOC-001` |

## 使用规则

- 任务开始：把命中的 pitfall ID 写进 PRD 或实现检查清单。
- 实现阶段：不要只读标题，必须打开 `pitfalls.md` 看完整规则。
- 收尾阶段：如果出现新坑，先写项目本地记录；能泛化的再反哺模板。
