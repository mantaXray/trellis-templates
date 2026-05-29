# External Knowledge Base

> 外部知识库用于保存大量 pitfall 复盘、长案例、日志摘要、截图说明和跨项目经验原文。`.trellis/spec` 只保存 agent 必须快速读取的索引和短规则。

## 推荐方案

优先使用一个独立 Git 仓库，例如：

```text
mcu-knowledge-base/
├── README.md
├── index.md
├── pitfalls/
│   ├── repository-and-tooling.md
│   ├── cubemx-iar-active-paths.md
│   ├── uart-dma-cache-protocols.md
│   ├── rtos-watchdog-shared-state.md
│   ├── memory-placement.md
│   └── sensors-display-storage.md
├── cases/
│   └── YYYY-MM/<sanitized-case-title>.md
├── attachments/
│   └── README.md
└── templates/
    ├── pitfall-entry.md
    └── debug-case.md
```

为什么用 Git：

- 能 review、diff、回滚
- agent 可以用全文搜索读取
- Markdown 对 Trellis/Claude/Codex 都友好
- 不依赖额外服务，适合内网

## 与模板仓库的关系

```text
trellis-mcu-templates
  specs/mcu-stm32-base/firmware/pitfalls-index.md         # 短索引，随模板发给每个项目
  specs/mcu-stm32-base/firmware/pitfalls.md               # 高频通用规则，随模板发给每个项目
  specs/mcu-stm32-base/firmware/pitfalls-<category>.md    # 单分类 >10 条时的拆分文件（按需创建）

mcu-knowledge-base
  pitfalls/*.md                                           # 长规则、扩展示例
  cases/*.md                                        # 去敏后的案例复盘
  attachments/                                      # 日志/截图/波形等附件说明
```

模板里只链接外部知识库的入口，不复制长内容。项目需要深入排查时，agent 再按关键词打开外部库。

## 条目模板

### Pitfall Entry

```markdown
# PIT-<AREA>-<NNN> - <短标题>

## Rule

<一句默认规则或禁止项>

## When To Check

- <触发条件 1>
- <触发条件 2>

## Failure Mode

<典型症状，去掉项目代号和客户信息>

## Checks

- <检查命令或文件>
- <人工确认点>

## Links

- Template summary: `trellis-mcu-templates/specs/mcu-stm32-base/firmware/pitfalls.md`
- Related cases: `cases/YYYY-MM/<case>.md`
```

### Debug Case

```markdown
# <去敏后的案例标题>

## Context

- MCU/toolchain:
- Module:
- Symptom:

## Timeline

- <关键观察>
- <误判路径>
- <最终根因>

## Root Cause

<根因，去掉项目专属编号和客户信息>

## Fix

<修复策略，不贴大段项目代码>

## Generalized Lesson

<可反哺模板的规则；如果不可泛化，写 N/A>

## Follow-up

- [ ] 是否更新项目 `.trellis/spec/firmware/pitfalls.md`
- [ ] 是否反哺模板 `firmware/pitfalls.md`
```

## 去敏规则

不要放入外部知识库：

- 客户名、项目代号、真实 SVN/Git URL
- 具体寄存器编号、私有协议字段、设备序列号
- 原始大日志、完整截图、密钥、账号、内网地址

可以保留：

- MCU 系列、工具链、外设类型
- 现象、根因类别、检查方法
- 去敏后的代码片段或伪代码
- 抽象后的 pitfall 规则

## 接入 Trellis

每个项目可以在 `AGENTS.md` 或项目本地 spec 里加一行：

```markdown
长篇 MCU 知识库：<mcu-knowledge-base 路径或仓库 URL>。只有在 `.trellis/spec/firmware/pitfalls-index.md` 指向外部案例，或用户要求深挖历史案例时才读取。
```

不要让每次任务默认读取整个外部知识库。默认只读模板内短索引；需要时再按 pitfall ID 或关键词搜索外部库。
