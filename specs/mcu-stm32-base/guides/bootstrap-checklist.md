# Bootstrap Checklist

> `trellis init -t mcu-stm32-base` 只能写入 `.trellis/spec/` 下的内容。**本文档列出装完模板后需要手动完成的项目级配置**。新项目按下面 checklist 逐项落地。
>
> **本 checklist 默认按 IAR Embedded Workbench for Arm 主流项目场景**编写（约公司 90% MCU 项目）。少数 legacy STM32CubeIDE+GCC 项目的差异在条目里单独说明。

---

## 1. AGENTS.md 末尾追加协同规则提示

打开项目根的 `AGENTS.md`（trellis init 已生成），**在 `<!-- TRELLIS:END -->` 之后**追加以下内容：

```markdown
---

## 本项目专用：Claude / Codex 协同硬规则

本项目采用 **Claude（task / 文档侧）+ Codex（code 侧）双 agent 协同**，职责边界严格：

- **Claude** 只负责：任务管理（PRD、brainstorm、Trellis 任务状态切换）、文档维护（`doc/`、`.trellis/spec/`、`.trellis/tasks/`）
- **Codex** 只负责：C/H 代码实现、编译验证（`iarbuild`）、`git commit` 代码改动

**Claude 严禁修改任何 `.c` / `.h` 文件**（即使一行）。即使 PRD 已经写完，Claude 也不应自行越界改代码 —— 代码部分等 Codex 接手。

唯一例外：用户在**当前消息**里明确说出 "你直接改" / "别派 sub-agent" / "main session 写就行" / "do it inline" / "no sub-agent" 等短语时，Claude 可以临时改一次。

完整规则与流程见：[`.trellis/spec/guides/claude-codex-collaboration.md`](.trellis/spec/guides/claude-codex-collaboration.md)
```

> 放在 `TRELLIS:END` 之后是为了避免被未来的 `trellis update` 覆盖。

---

## 2. 写项目根 `.gitignore`

### 2.1 IAR 主流项目模板（默认用这个）

```gitignore
# =============================================================================
# IAR Embedded Workbench temporary/intermediate files
# =============================================================================

EWARM/*/Obj/
EWARM/*/BrowseInfo/
EWARM/*/Exe/
EWARM/*/List/
EWARM/*/.ninja_deps
EWARM/*/.ninja_log
EWARM/*/*.dep
EWARM/*/*.rsp
EWARM/*/*.pbi
EWARM/*/*.pbw
EWARM/*/*.pbd
EWARM/*/*.browse
EWARM/*/*build_cache*
EWARM/*/build.ninja
EWARM/Backup*
.ninja_deps
.ninja_log
build.ninja

# IAR local workspace/debug state
EWARM/settings/

# =============================================================================
# Released binaries are tracked in Bin/, but raw build artifacts ignored
# =============================================================================

# Released versioned binaries are kept
!Bin/*.hex
!Bin/*.bin
!Bin/*.out
# Other top-level build leakage
*.hex
*.bin
*.out
*.elf

# =============================================================================
# IDE local state
# =============================================================================

# VS Code cache (workspace file *.code-workspace itself is tracked)
.vscode/BROWSE.VC.DB*
.vscode/ipch/

# =============================================================================
# Local assistant / tooling state
# =============================================================================

.qoder/
.lingma/
.uv-cache/
tmp/

# =============================================================================
# Reference documents: local-only, do not upload to Git or SVN
# =============================================================================

ref_doc/
ref_docs/

# =============================================================================
# Trellis per-developer journals (local only)
# =============================================================================

.trellis/workspace/*/journal-*.md

# =============================================================================
# Editor / OS junk
# =============================================================================

*.swp
*~
*.bak
*.orig

# Microsoft Office lock files
~$*

Thumbs.db
Desktop.ini
.DS_Store
```

### 2.2 STM32CubeIDE+GCC 例外项目模板（仅 legacy 项目）

如果项目使用 STM32CubeIDE 而不是 IAR（公司内属少数 legacy 情况），把 §2.1 的 IAR 段落整体替换成：

```gitignore
# STM32CubeIDE / GCC build outputs
Debug/
Release/
*.o
*.d
*.su
*.cyclo
*.map
*.list

# Released binaries in Bin/ are tracked (versioned releases)
*.hex
*.bin
!Bin/*.hex
!Bin/*.bin

# CubeIDE workspace metadata
.metadata/
```

其他段落（assistant state / ref_doc / journals / editor / OS junk）与 IAR 模板相同。

---

## 3. 设置 SVN `svn:ignore`（公司 SVN 工作副本下都需要）

在项目目录上设 `svn:ignore` 属性。先把下面内容写到临时文件 `/tmp/svn_ignore.txt`：

```
# AI / Trellis / Git config — git-only
.agents
.claude
.codex
.trellis
.git
.gitignore
AGENTS.md

# Documents — git-only
doc

# Reference docs — local only
ref_doc
ref_docs

# IAR build artifacts (主流项目)
EWARM/Backup*
*.pbi
*.pbw
*.pbd
*.browse
.ninja_deps
.ninja_log
build.ninja

# STM32CubeIDE+GCC build artifacts (少数 legacy 项目)
Debug
Release

# Local tooling state
.qoder
.lingma
.uv-cache
tmp

# OS / editor junk
Thumbs.db
Desktop.ini
.DS_Store
*.swp
*~
*.bak
~$*
```

然后：

```bash
svn propset svn:ignore -F /tmp/svn_ignore.txt <项目目录>
svn propget svn:ignore <项目目录>   # 验证
```

⚠️ **不要立刻 `svn commit` 这个属性改动**，等下一次 SVN 提交时一起带。

---

## 4. 创建 `doc/` 目录与命名约定 README

```bash
mkdir -p doc
```

写 `doc/README.md`：

```markdown
# doc/ 目录约定

本目录存放项目相关的设计文档、优化方案、阶段性总结。**只上 Git，不上 SVN**。

## 命名约定

| 类型 | 命名 | 示例 |
|---|---|---|
| 当前设计基线 | `软件设计说明_<scope>.md` | `软件设计说明_采集流程与算法.md` |
| 优化方案 | `优化方案_<YYYY-MM-DD>_<scope>.md` | `优化方案_2026-05-27_采集流程修复.md` |
| 阶段性总结 | `阶段总结_<YYYY-MM-DD>_<phase>.md` | `阶段总结_2026-06-15_alpha-release.md` |

## 同步规则

代码改动完成后，**Claude 负责**把"软件设计说明" 文档同步到与新代码一致的状态。
"优化方案" 文档作为单次任务的产物，归档后不再修改。
```

---

## 5. 加入版本宏（按项目惯例）

如果项目还没有版本宏，新增到 **`User/App/System/define.h`**（IAR 主流项目惯例）：

```c
#define SOFTWARE_VERSION        "v1.00.00"   /* 整体固件版本 */
#define SOFTWARE_VERSION_DATE   "20260527"   /* 当前版本发布日期 */
```

少数 legacy STM32CubeIDE 项目用 `User/App/Inc/define.h` + `VERSION` / `VERSION_DATE` 命名。

可选：若项目有独立算法核心，加 `ALGO_VERSION`：

```c
#define ALGO_VERSION            "v1.0"       /* 算法核心版本，仅算法变更才 bump */
```

详细 bump 规则见 `.trellis/spec/firmware/version-control.md` §2。

---

## 6. 检查项目代号在所有位置一致

更新以下文件里的项目代号（如果是从模板 / 其他项目克隆出来的）：

- `EWARM/<项目代号>.ewp` `EWARM/<项目代号>.ewd` `EWARM/<项目代号>.eww` 文件名 + 内部引用（IAR 项目）
- `.cproject` `.project` `*.code-workspace` 里的项目名（少数 legacy CubeIDE 项目）
- `<项目代号>.ioc` 文件名 + 内部引用
- `Bin/<项目代号>_v*.bin` `Bin/<项目代号>_v*.hex` 发布命名
- SVN log 模板里的 `版本：<项目代号>:...` 前缀（见 version-control.md §4.3）

---

## 7. 初始化 git 远端（如果还没绑定）

```bash
# 公司内网 Gitea
git remote add origin <git remote URL>
git push -u origin main
```

> 公司内网 Gitea 服务器地址按现行配置；如有迁移以最新通知为准。

---

## 8. 验证清单

完成上述步骤后，自检：

- [ ] `AGENTS.md` 末尾有 Claude/Codex 协同规则提示
- [ ] `.gitignore` 已写好，`git status` 不显示 `EWARM/*/Obj/` 等 IAR 构建产物
- [ ] SVN 工作副本 `svn status --no-ignore` 显示 AI/Trellis / 构建产物为 `I`
- [ ] `doc/README.md` 已写
- [ ] `User/App/System/define.h`（或等价位置）有版本宏
- [ ] 项目代号在所有位置一致（`.ewp` / `.ioc` / `Bin/` / SVN log 模板）
- [ ] `git remote -v` 显示正确远端
- [ ] `trellis-meta` skill 跑过一遍，确认 spec 正确加载
- [ ] **IAR 项目**：能在命令行用 `iarbuild <项目>.ewp -build Debug` 跑通编译
- [ ] **少数 CubeIDE 项目**：能在命令行用 `make -j8 all` 跑通编译

完成后，把当前 commit 用 `chore: bootstrap project from mcu-stm32-base template` 类似 message commit 一下。
