# Canonical Content Snippets

这个目录存放在多处被引用的"规范内容"。`skills/mcu-bootstrap/SKILL.md`（发给下游项目）和 `docs/bootstrap-checklist.md`（仅供模板维护者参考的手工 fallback）都包含这些内容的内嵌副本（出于 AI/用户运行时自包含的考虑）。`scripts/validate.py` 会比对内嵌副本与本目录的规范源，确保它们不漂移。

## 文件清单

| 文件 | 说明 | 被引用的位置 |
|---|---|---|
| `agents-bootstrap-block.md` | 追加到项目 `AGENTS.md` 的 managed block | SKILL.md Step 5 / docs/bootstrap-checklist.md §1 |
| `gitignore-block.txt` | 项目 `.gitignore` managed block 内容 | SKILL.md Step 6 / docs/bootstrap-checklist.md §2 |
| `svn-ignore.list` | 项目 `svn:ignore` 必需基础名清单（一行一条） | SKILL.md Step 9 / docs/bootstrap-checklist.md §3 |
| `doc-readme.md` | 写入项目 `doc/README.md` 的内容 | SKILL.md Step 7 / docs/bootstrap-checklist.md §4 |

## 修改规则

需要变更这些内容时：

1. **只改本目录的规范文件**。
2. 同步改 `skills/mcu-bootstrap/SKILL.md` 和 `docs/bootstrap-checklist.md` 中的内嵌副本（保持字节一致）。
3. 跑 `python scripts/validate.py` 验证没有漂移。

新增条目（例如新增 `.gitignore` 行）若涉及顺序敏感性（如 `*.hex` 必须在 `!Bin/*.hex` 前），在文件内用注释或紧邻关系标注。

## 不在本目录的内容

- 版本宏（与工具链/项目代号深度耦合，留在 SKILL.md 内嵌）
- Git remote / SVN 命令脚本（执行性脚本，不是数据）
- Trellis config 示例 YAML（仅作为参考，不强制一致）
