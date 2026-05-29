# Claude / Codex 协同工作规则

> 本文档定义 AI 助手在本项目中的职责边界、交接物、和禁止越界的硬规则。**任何 AI 助手开始工作前必读**。
>
> 默认模型是 **Claude（task-side）+ Codex（code-side）双 agent 协同**；只用 Codex 或只用 Claude 的开发者请先阅读 §0 选择适用模式。

---

## 0. 适用模式（必先确认）

不同开发者使用 AI 工具的组合不一样，本文档按以下三种模式分别适用：

### 模式 A：Claude + Codex 双轨（推荐 —— 文档完整描述的模式）

- Claude 负责 task / docs 侧（PRD、brainstorm、Trellis 任务管理、文档维护）
- Codex 负责 code 侧（C/H 代码实现、编译验证、commit）
- 两个 agent 严格分工，**Claude 不写代码，Codex 不写 PRD**
- §1~§7 所有规则**直接适用**

### 模式 B：Codex-only（只用 Codex）

- 同一个 Codex agent **既做 task 又做 code**
- 但**仍然保留"任务相位 / 代码相位"的概念区分**：
  - 当用户跟 Codex 讨论需求、brainstorm、写 PRD 时，Codex 处于**任务相位**（按 Claude 的角色行事）
  - 当用户让 Codex 改代码时，Codex 处于**代码相位**（按 Codex 的角色行事）
- 切换相位的信号是 **Trellis 任务状态**：`planning` 状态下为任务相位、`in_progress` 状态下为代码相位
- 这种模式下：
  - 单 agent 没有"越界写代码"的硬禁止（同一个 agent 哪个相位都能做）
  - **但仍然禁止在 planning 阶段跳过 PRD 直接改代码** —— 必须先 brainstorm 收敛
  - PRD / 优化方案 / 设计文档**仍然是必要的交付物**，不能图省事跳过
- §2 工作流交接（Phase 1 → 2 → 3）仍然遵守，只是交接对象从"另一个 agent"变成"同一个 agent 的另一个相位"
- §3 硬禁止清单按角色读 —— task 相位时不可以"凭自己理解直接改代码"

### 模式 C：Claude-only（只用 Claude）

- 罕见，但部分场景下可能（如纯算法 / 设计型任务，代码改动量极小）
- 处理方式同模式 B —— 同一个 Claude agent 既做 task 相位又做 code 相位
- 编译验证可能需要用户手动跑或 Claude 通过 Bash 工具跑（Claude 一般避免，但单 agent 时这是允许的）

---

## 1. 角色分工（硬约束）

| 角色 | Claude（task / 文档侧） | Codex（code 侧） |
|---|---|---|
| **主要职责** | 需求收敛、PRD 编写、方案设计、文档维护、Trellis 任务管理 | C/H 代码实现、编译验证、单元测试编写 |
| **能写什么** | `.trellis/tasks/**`、`doc/**`、`.trellis/spec/**`、`.trellis/workspace/**`、AGENTS.md / README.md / CHANGELOG.md 等纯文档 | `Core/**`、`User/**`、`Drivers/**`、`Middlewares/**` 等所有 C/H 代码；CubeMX 生成区域的 `USER CODE` 块 |
| **不能写什么** | **绝对不能改 C/H 代码**（即使一行）；不能 `git commit` 代码改动 | 不能改 PRD / 方案文档（除非 PRD 漏写实现细节，可通过提案让 Claude 补） |
| **典型工具调用** | `Read/Edit/Write/Glob/Grep/Bash`（仅读、文档写、Trellis 脚本） | `Read/Edit/Write/Bash`（含编译命令） |
| **Trellis 阶段** | Phase 1（Plan）+ Phase 3（Finish 阶段文档与 spec 更新） | Phase 2（Execute），包括 implement + check |

---

## 2. 工作流交接

### 2.1 Claude → Codex 交接（Phase 1 → Phase 2）

Claude 完成 brainstorm 收敛后，**必须**在交付给 Codex 前完成以下产物：

1. `prd.md` —— 完整 PRD（含 Goal / Requirements / AC / Out of Scope / Technical Approach / Implementation Plan）
2. `implement.jsonl` —— spec/research 文件清单，**只填路径不写代码**
3. `check.jsonl` —— check 阶段同样的 spec 清单
4. 一份**优化方案文档**放在 `doc/` 下（命名建议：`优化方案_YYYY-MM-DD_<scope>.md`）
5. 一份**当前设计基线文档**（如果不存在）放在 `doc/` 下（命名建议：`软件设计说明_<scope>.md`）
6. 任务状态切到 `in_progress`（通过 `python ./.trellis/scripts/task.py start <task-dir>`）

> 交接信号：在主会话里明确告诉用户"PRD 完成，等 Codex 接手实现"，并停止任何代码改动尝试。

PRD 建议结构见 [`task-process.md`](task-process.md)：Goal、Current State、Requirements、Acceptance Criteria、Definition of Done、Out of Scope、Technical Approach、Implementation Plan、Research References。需要调研的内容写入 `.trellis/tasks/<task>/research/<topic>.md`，不要只留在聊天记录里。

### 2.2 Codex 接手时的 entry 点

Codex 启动时必须读取（按重要性排序）：

1. PRD：`.trellis/tasks/<active-task>/prd.md`
2. 优化方案：`doc/优化方案_*.md`（本次任务的）
3. 当前设计基线：`doc/软件设计说明_*.md`
4. `implement.jsonl` 列出的所有 spec / 参考文档
5. 坑点索引：`.trellis/spec/firmware/pitfalls-index.md`，并按任务类型打开 `.trellis/spec/firmware/pitfalls.md` 对应章节
6. 跨模块任务还要读 `.trellis/spec/guides/firmware-data-flow-guide.md`，新增 helper/复用路径时读 `.trellis/spec/guides/firmware-reuse-guide.md`
7. 实际 C/H 代码（按 PRD Implementation Plan 中列出的文件清单）

`implement.jsonl` 只放实现所需路径和简短用途说明；`check.jsonl` 放验证阶段要读的 spec、AC、测试文档、坑点和待检查文件。不要把大段代码粘进这两个文件。

### 2.3 Codex → Claude 反馈（Phase 2 → Phase 3）

Codex 改完代码、编译通过、check 通过后，**主动通知 Claude**：

- 列出实际改了哪些文件、相对 PRD Implementation Plan 是否有偏差
- 编译输出（make / IAR / CubeIDE 完整 log 的关键行）
- 任何在实现过程中发现的、PRD 没覆盖的边缘情况（让 Claude 决定补 PRD 还是单独跟进）

Claude 收到反馈后：
- 更新设计基线文档 `doc/软件设计说明_*.md` 到与新代码一致
- 必要时通过 `trellis-update-spec` 把新约定沉淀到 `.trellis/spec/`
- 若实现中发现可复发坑点，先写项目本地 pitfall；可泛化的条目按模板仓库 `docs/pitfall-feedback-loop.md` 反哺上游模板
- 不主动 commit 代码（commit 也归 Codex 做）
- finish 前按 [`task-process.md`](task-process.md) 判断是否需要 session notes、research 归档、spec 更新和 pitfall 反哺；若 `.trellis/config.yaml` 设置 `session_auto_commit: false`，不要假设会话记录已自动提交

---

## 3. 硬禁止清单

> 在**双轨模式 A** 下，下列禁止规则按 agent 直接套用。
> 在**Codex-only 模式 B** / **Claude-only 模式 C** 下，规则**按相位读取** —— "Claude 严禁"列对应任务相位（planning），"Codex 严禁"列对应代码相位（in_progress）；同一 agent 在两个相位之间切换时严格遵守对应一侧。

### 3.1 Claude 严禁 / 任务相位严禁

| ❌ 禁止行为 | 为什么 |
|---|---|
| 修改任何 `.c` / `.h` 文件 | 越界；Codex 是 code-side owner |
| 修改 `Drivers/` `Middlewares/` 等 vendor / 生成代码 | 同上 |
| 执行 `iarbuild` / `make` 等编译命令 | 编译验证归 Codex；Claude 的编译输出会污染上下文（按项目实际工具链选择命令） |
| 在 brainstorm 阶段就跳到"我来写代码" | 违反协同流程 |
| 在 PRD / 方案文档里嵌入大段可直接 copy-paste 的 C 代码片段 | 这是 Codex 的工作；Claude 应描述设计意图，不应预写实现 |
| `git commit` C/H 代码改动 | commit 归 Codex |

### 3.2 Codex 严禁 / 代码相位严禁

| ❌ 禁止行为 | 为什么 |
|---|---|
| 在没有 PRD 的情况下凭"自己理解"开始改代码 | 流程绕过 brainstorm，会偏离用户意图 |
| 改 PRD 文件 | PRD 是任务相位 / Claude / 用户产物；代码相位发现 PRD 有问题应反馈给任务相位 / Claude，不应自己修 |
| 实现 PRD 明确标记为 "Out of Scope" 的功能 | 范围蔓延 |
| 跳过 check 阶段直接 commit | 流程不完整 |

---

## 4. 例外条款

### 4.1 用户显式 inline 改动（仅适用模式 A 双轨）

如果用户在**当前消息**里明确说出以下任一短语，**Claude 可以临时跨界改代码一次**（仅当前轮）：

- `"do it inline"` / `"no sub-agent"`
- `"你直接改"` / `"别派 sub-agent"` / `"main session 写就行"` / `"不用 sub-agent"`

未见用户说出上述短语时，**严禁自行越界**。

> **模式 B Codex-only / 模式 C Claude-only 下**本条不适用 —— 单 agent 下不存在 sub-agent 派发概念，agent 自身按当前相位行事即可，相位切换由 Trellis 任务状态驱动。

### 4.2 紧急小修

修一个 typo / 注释错别字 / 文档里嵌的过时代码示例 —— 这类**纯文档性**的小修，Claude 可以做。但**只要涉及编译产物**，立刻收手。

---

## 5. 文档命名约定

| 类型 | 命名 | 路径 |
|---|---|---|
| 当前设计基线 | `软件设计说明_<scope>.md` | `doc/` |
| 优化方案 | `优化方案_<YYYY-MM-DD>_<scope>.md` | `doc/` |
| 项目坑点记录 | `pitfalls.md` / `坑点记录.md` | `doc/` 或 `.trellis/spec/firmware/` |
| 任务 PRD | `prd.md` | `.trellis/tasks/<MM-DD-slug>/` |
| 任务 spec 上下文 | `implement.jsonl` / `check.jsonl` | `.trellis/tasks/<MM-DD-slug>/` |
| 协同规则（本文件） | `claude-codex-collaboration.md` | `.trellis/spec/guides/` |

---

## 6. Trellis 阶段对照

> 下面用括号标注**双轨模式 A** 时的 agent 分工；**模式 B/C** 单 agent 时换成同一 agent 在不同相位行事，括号忽略即可。

```
Phase 1 (Plan)          任务相位 [模式 A: Claude 主导]
  ├─ task.py create
  ├─ trellis-brainstorm
  ├─ prd.md
  ├─ implement.jsonl / check.jsonl
  └─ task.py start                  ← 任务相位在这里收手

Phase 2 (Execute)       代码相位 [模式 A: Codex 主导]
  ├─ 读 PRD + 设计文档 + spec
  ├─ 读 pitfalls-index.md + 相关 pitfalls 条目
  ├─ 改代码
  ├─ 编译验证（例如 iarbuild 或 make）
  ├─ trellis-check
  └─ 反馈给任务相位

Phase 3 (Finish)        相位接力 [模式 A: Claude + Codex 接力]
  ├─ 代码相位: git commit 代码改动
  ├─ 任务相位: 更新 doc/软件设计说明_*.md 到与新代码一致
  ├─ 任务相位: 判断新坑点是否写入项目 pitfall / 反哺模板
  ├─ 任务相位: 可能通过 trellis-update-spec 沉淀新约定
  └─ /trellis:finish-work
```

---

## 7. 如何让用户判断当前是哪个阶段

- **任务状态 `planning`** → Phase 1，任务相位（模式 A: Claude 当家；模式 B/C: 同一 agent 按任务相位行事）
- **任务状态 `in_progress`** → Phase 2，代码相位（模式 A: Codex 当家，Claude 只读不写代码；模式 B/C: 同一 agent 按代码相位行事）
- **任务状态 `complete` 前的收尾** → Phase 3，相位接力（模式 A: Claude 更新文档 + spec、Codex 收尾 commit；模式 B/C: 同一 agent 完成两件事）

用 `python ./.trellis/scripts/task.py current` 或 `task.py list` 随时查看。
