<!-- MCU-BOOTSTRAP:START -->
<!-- MCU-BOOTSTRAP:STARTED {{TODAY_YYYY-MM-DD}} 项目代号: {{PROJECT_CODE}} -->
<!-- MCU-BOOTSTRAP:MODE {{COLLABORATION_MODE}} -->

---

## 本项目专用：AI 助手协同规则

**当前协同模式**：`{{COLLABORATION_MODE}}`

本项目按 **任务相位 -> 代码相位** 严格分离工作：

- **任务相位（Trellis `planning` 状态）**：需求收敛、brainstorm、PRD 编写、`doc/` 与 `.trellis/spec/` 文档维护。**不允许修改 C/H 代码**。
- **代码相位（Trellis `in_progress` 状态）**：C/H 代码实现、编译验证（例如 `iarbuild` 或 `make`）、`git commit` 代码改动。
- 相位切换由 `python ./.trellis/scripts/task.py start <task-dir>` 等命令驱动。

不同协同模式下"谁做什么"的对照（**实际生效模式以上方 `MCU-BOOTSTRAP:MODE` 标记为准**）：

- **`codex-only`**（团队默认）：同一个 Codex agent 在两个相位间切换
- **`dual-track`**（Claude + Codex）：Claude 做任务相位，Codex 做代码相位
- **`claude-only`**：同一个 Claude agent 在两个相位间切换

如需变更模式，同时更新上方 `MCU-BOOTSTRAP:MODE` 标记和本行说明。Agent 在读取本块时优先信任 marker，不要靠猜。

完整规则、相位禁止清单、override 例外、Trellis Phase 对照见：
[`.trellis/spec/guides/claude-codex-collaboration.md`](.trellis/spec/guides/claude-codex-collaboration.md)

通用坑点知识库与索引见：
- [`.trellis/spec/firmware/pitfalls-index.md`](.trellis/spec/firmware/pitfalls-index.md)
- [`.trellis/spec/firmware/pitfalls.md`](.trellis/spec/firmware/pitfalls.md)

<!-- MCU-BOOTSTRAP:END -->
