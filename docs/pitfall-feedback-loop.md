# Pitfall Feedback Loop

> 每个项目都可以有自己的 pitfall 文件；模板仓库维护通用知识库。两者不要混成一份。

## 推荐结构

项目内：

```text
doc/pitfalls.md                         # 项目现场事实、排查过程、专有细节
.trellis/spec/firmware/pitfalls.md      # 项目级可执行规则，可含项目特化约定
```

模板内：

```text
specs/mcu-stm32-base/firmware/pitfalls-index.md
specs/mcu-stm32-base/firmware/pitfalls.md
specs/mcu-stm32-base/firmware/pitfalls-<category>.md   # 单分类 >10 条时的拆分文件（按需创建）
```

如果 pitfall 内容很多，不要把所有长篇复盘都塞进 `.trellis/spec`。模板 spec 只放 agent 必须提前知道的短规则、索引和链接；长篇案例、截图、日志、协议附件适合放外部知识库或独立仓库。搭建方式见 [`external-knowledge-base.md`](external-knowledge-base.md)。

## 什么时候写项目本地 pitfall

- 同类问题可能再次出现
- 后续 agent 或同事能通过一条规则提前避免
- 涉及项目现场事实、客户环境、具体寄存器、协议编号、代号或日期
- 还不确定是否适合所有 MCU 项目

## 什么时候反哺模板

满足以下条件再进入模板：

- 跨项目常见，或至少不依赖单一项目结构
- 与 MCU 工具链、RTOS、外设、协议、版本提交流程有关
- 能写成“检查项/禁止项/默认策略”
- 已去掉项目代号、客户信息、具体寄存器编号、现场日期和一次性路径

## 反哺流程

1. 在项目本地记录原始事实和修复过程。
2. 抽象成一句可执行规则，放到项目 `.trellis/spec/firmware/pitfalls.md`。
3. 任务收尾时判断是否通用。
4. 回到模板仓库 `D:\00_Work\00_trellis-templates`，把通用条目加入 `specs/mcu-stm32-base/firmware/pitfalls.md`。
5. 如果某分类超过 10 条，把该分类拆到 `specs/mcu-stm32-base/firmware/pitfalls-<category>.md` 同级文件，并更新核心 `pitfalls.md` 中的章节链接；否则保持在核心 `pitfalls.md`。
6. 更新 `pitfalls-index.md` 和必要 README。
7. 其他已有项目用 `trellis init --append -t mcu-stm32-base -r <registry>` 补新增文件；已有同名 spec 文件时手动 diff/merge。

## 已有 Trellis 项目如何更新

- **只补新文件**：使用 `trellis init --append -t mcu-stm32-base -r <registry>`。
- **同步已有 spec 文件的新版本**：先 diff 模板与项目本地文件，再手动 merge。不要随意 `--overwrite`，避免覆盖项目特化规则。
- **只想引入 pitfall 知识库**：复制或 append `firmware/pitfalls-index.md`、`firmware/pitfalls.md`（以及任何已存在的 `firmware/pitfalls-<category>.md` 拆分文件），再把 `firmware/index.md` 或项目现有入口链接过去。

## 写法要求

- 每条 pitfall 给稳定 ID，例如 `PIT-DMA-001`。
- 标题写症状或风险，不写项目名。
- 正文写默认规则和检查方式，不写长篇复盘。
- 项目专属细节留在 `doc/` 或项目本地知识库。
