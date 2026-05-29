# Bootstrap Checklist

> `trellis init -t mcu-stm32-base` 只能写入 `.trellis/spec/` 下的内容。本文档列出模板安装后需要完成的项目级配置。优先使用 `mcu-bootstrap` skill 自动执行；本 checklist 是手动等价流程和审计清单。
>
> 按项目实际工具链使用。IAR Embedded Workbench for Arm、STM32CubeIDE+GCC 或其他构建方式的差异在条目里单独说明。

---

## 0. 推荐执行方式

新项目建议直接运行 `mcu-bootstrap` skill。它会按文件状态补齐缺失项，并用以下 marker 记录状态：

- `<!-- MCU-BOOTSTRAP:STARTED ... -->`：bootstrap 已开始，可安全重跑修复
- `<!-- MCU-BOOTSTRAP:DONE ... -->`：所有必需步骤已完成

如果手动执行本文档，不要在流程未完成前写 `DONE` 标记。

---

## 1. AGENTS.md 末尾追加协同规则提示

打开项目根 `AGENTS.md`，在 `<!-- TRELLIS:END -->` 之后追加一个 managed block。若没有 `TRELLIS:END`，追加到文件末尾。

```markdown
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
```

全部步骤完成后，把 `STARTED` 行替换为：

```markdown
<!-- MCU-BOOTSTRAP:DONE {{TODAY_YYYY-MM-DD}} 项目代号: {{PROJECT_CODE}} -->
```

---

## 2. 合并项目根 `.gitignore`

如果 `.gitignore` 已存在，保留原内容，只补下面 managed block。不要整文件覆盖。

```gitignore
# BEGIN MCU-BOOTSTRAP GITIGNORE
# IAR Embedded Workbench temporary/intermediate files
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
EWARM/settings/

Debug/
Release/
*.o
*.d
*.su
*.cyclo

*.elf
*.map
*.list
*.hex
*.bin
*.out
!Bin/*.hex
!Bin/*.bin
!Bin/*.out

.metadata/
.vscode/BROWSE.VC.DB*
.vscode/ipch/

.qoder/
.lingma/
.uv-cache/
tmp/

ref_doc/
ref_docs/

.trellis/workspace/*/journal-*.md
.trellis/.runtime/
.trellis/.backup-*/

*.swp
*~
*.bak
*.orig
~$*

Thumbs.db
Desktop.ini
.DS_Store
# END MCU-BOOTSTRAP GITIGNORE
```

注意：

- `*.hex` / `*.bin` / `*.out` 必须放在 `!Bin/*` 发布产物例外之前。
- 不要 ignore `.trellis/spec/`、`.trellis/tasks/` 等共享 Trellis 内容。
- 只 ignore `.trellis/workspace/*/journal-*.md`、`.trellis/.runtime/`、`.trellis/.backup-*/` 等本地状态。

---

## 3. 合并 SVN `svn:ignore`

SVN 工作副本下需要在项目目录上设置 `svn:ignore`。不要覆盖已有属性；先读取现有值，再合并缺失项。

`svn:ignore` 对设置它的目录按 basename 生效，不是 Git 风格的全仓库路径通配。根目录先设置 AI/Trellis、Git、文档、顶层构建目录等规则；如果 `EWARM/<target>/Obj`、`EWARM/<target>/Exe` 等嵌套产物仍显示为 `?`，需要在对应父目录继续设置 basename ignore。

### Windows PowerShell

```powershell
$projectDir = (Get-Location).Path
$required = @(
  ".agents", ".claude", ".codex", ".trellis", ".git", ".gitignore", "AGENTS.md",
  "doc", "ref_doc", "ref_docs",
  "EWARM/Backup*", "*.pbi", "*.pbw", "*.pbd", "*.browse",
  ".ninja_deps", ".ninja_log", "build.ninja",
  "Debug", "Release",
  ".qoder", ".lingma", ".uv-cache", "tmp",
  "Thumbs.db", "Desktop.ini", ".DS_Store", "*.swp", "*~", "*.bak", "~$*"
)
$current = svn.exe propget svn:ignore -- "$projectDir" 2>$null
$merged = @($current -split "`r?`n") + $required |
  Where-Object { $_ -and $_.Trim() } |
  Sort-Object -Unique
$tmp = New-TemporaryFile
Set-Content -LiteralPath $tmp.FullName -Value $merged -Encoding UTF8
svn.exe propset svn:ignore -F $tmp.FullName -- "$projectDir"
svn.exe propget svn:ignore -- "$projectDir"
Remove-Item -LiteralPath $tmp.FullName -Force
```

### Bash

```bash
project_dir="$(pwd)"
tmp_file="$(mktemp)"
{
  svn propget svn:ignore "$project_dir" 2>/dev/null || true
  printf '%s\n' \
    .agents .claude .codex .trellis .git .gitignore AGENTS.md \
    doc ref_doc ref_docs \
    'EWARM/Backup*' '*.pbi' '*.pbw' '*.pbd' '*.browse' \
    .ninja_deps .ninja_log build.ninja \
    Debug Release \
    .qoder .lingma .uv-cache tmp \
    Thumbs.db Desktop.ini .DS_Store '*.swp' '*~' '*.bak' '~$*'
} | awk 'NF && !seen[$0]++' > "$tmp_file"
svn propset svn:ignore -F "$tmp_file" "$project_dir"
svn propget svn:ignore "$project_dir"
rm -f "$tmp_file"
```

常见嵌套目录补充规则示例：

```powershell
# 在 EWARM/<target>/ 上设置 Obj、Exe、List、BrowseInfo 等 basename
svn.exe propset svn:ignore "Obj`nExe`nList`nBrowseInfo" -- "EWARM/<target>"
```

不要在 bootstrap 时 `svn commit`。这个属性等下一次用户明确要求 SVN 提交时一起带上。

---

## 4. 创建 `doc/` 目录与 README

### Windows PowerShell

```powershell
New-Item -ItemType Directory -Path "doc" -Force
```

### Bash

```bash
mkdir -p doc
```

如果 `doc/README.md` 已存在，不覆盖。缺失时写：

```markdown
# doc/ 目录约定

本目录存放项目相关的设计文档、优化方案、阶段性总结。**只上 Git，不上 SVN**。

## 命名约定

| 类型 | 命名 | 示例 |
|---|---|---|
| 当前设计基线 | `软件设计说明_<scope>.md` | `软件设计说明_采集流程与算法.md` |
| 优化方案 | `优化方案_<YYYY-MM-DD>_<scope>.md` | `优化方案_2026-05-27_采集流程修复.md` |
| 阶段性总结 | `阶段总结_<YYYY-MM-DD>_<phase>.md` | `阶段总结_2026-06-15_alpha-release.md` |
| 项目坑点 | `pitfalls.md` 或 `坑点记录.md` | `pitfalls.md` |

## 同步规则

代码改动完成后，任务相位负责把"软件设计说明"文档同步到与新代码一致的状态。
"优化方案"文档作为单次任务的产物，归档后不再修改。
项目专属坑点先记录在本目录；可泛化的条目反哺到 `.trellis/spec/firmware/pitfalls.md`。
```

---

## 5. 加入版本宏

IAR-style 项目常见路径：`User/App/System/define.h`。

```c
#define SOFTWARE_VERSION        "v1.00.00"   /* 整体固件版本 */
#define SOFTWARE_VERSION_DATE   "{{TODAY_YYYYMMDD}}"    /* 当前版本发布日期 */
```

STM32CubeIDE/GCC-style 项目常见路径：`User/App/Inc/define.h`。

```c
#define VERSION                 "v1.0.0"     /* 整体固件版本 */
#define VERSION_DATE            "{{TODAY_YYYYMMDD}}"    /* 当前版本发布日期 */
```

可选：若项目有独立算法核心，加：

```c
#define ALGO_VERSION            "v1.0"       /* 算法核心版本，仅算法变更才 bump */
```

执行规则：

- 每个宏独立检测，已有则不改，缺哪个补哪个。
- 日期用执行当天替换 `{{TODAY_YYYYMMDD}}`。
- 详细 bump 规则见 `.trellis/spec/firmware/version-control.md` §2。

---

## 6. 检查项目代号一致性

如果项目从模板或其他项目克隆而来，检查以下位置：

- `EWARM/<项目代号>.ewp`、`.ewd`、`.eww` 文件名和内部引用
- `.cproject`、`.project`、`*.code-workspace` 里的项目名
- `<项目代号>.ioc` 文件名和内部引用
- `Bin/<项目代号>_v*.bin`、`Bin/<项目代号>_v*.hex` 发布命名
- SVN log 模板里的 `版本：<项目代号>:...` 前缀

---

## 7. 初始化 Git 远端

Bootstrap 只绑定或修正 remote，不 push。

### Windows PowerShell

```powershell
if (-not (Test-Path -LiteralPath ".git")) {
  git init -b main
}
$existing = git remote get-url origin 2>$null
if (-not $existing) {
  git remote add origin <git remote URL>
} elseif ($existing -eq "<git remote URL>") {
  Write-Host "origin already configured"
} else {
  git remote set-url origin <git remote URL>
}
git remote -v
```

### Bash

```bash
if [ ! -d .git ]; then
  git init -b main
fi
existing="$(git remote get-url origin 2>/dev/null || true)"
if [ -z "$existing" ]; then
  git remote add origin "<git remote URL>"
elif [ "$existing" = "<git remote URL>" ]; then
  printf '%s\n' "origin already configured"
else
  git remote set-url origin "<git remote URL>"
fi
git remote -v
```

首次 push 等用户明确说“提交 git”或“提交远程”后再做。

---

## 8. Trellis config 与会话策略

如果项目使用 `.trellis/config.yaml`，建议检查或补充以下约定。没有该文件时，本节可作为后续人工配置参考，不阻塞 bootstrap。

```yaml
session_commit_message: "chore(trellis): update session notes"
max_journal_lines: 2000
session_auto_commit: false

hooks:
  after_create: []
  after_start: []
  after_finish: []
  after_archive: []

packages: []
default_package: null

codex:
  dispatch_mode: default
```

规则：

- `session_auto_commit: false` 适合希望人工 review `.trellis/` 会话记录的项目。
- `.trellis/workspace/*/journal-*.md`、`.trellis/.runtime/`、`.trellis/.backup-*/` 是本地状态，应被 Git ignore。
- `.trellis/spec/` 和 `.trellis/tasks/` 是共享上下文，不要被 Git ignore。
- 通用生命周期 hook 可通过环境变量 `TASK_JSON_PATH` 获取任务元数据。
- hook 失败应打印 warning，不应阻塞 create/start/finish/archive 主流程。
- 不要把某个项目的 `.trellis/scripts/` runtime 或外部系统 hook 原样复制进模板。

---

## 9. 验证清单

- [ ] `AGENTS.md` 有 `MCU-BOOTSTRAP:STARTED` 或最终 `MCU-BOOTSTRAP:DONE`
- [ ] `.gitignore` 已包含 managed block，且 `Bin/*.hex` / `Bin/*.bin` / `Bin/*.out` 仍可跟踪
- [ ] SVN 工作副本 `svn status --no-ignore` 显示 AI/Trellis / 构建产物为 ignored
- [ ] `doc/README.md` 已存在
- [ ] `User/App/System/define.h` 或 `User/App/Inc/define.h` 有版本宏
- [ ] 项目代号在工程文件、`.ioc`、`Bin/`、SVN log 模板中一致
- [ ] `git remote -v` 显示正确远端，且 bootstrap 没有 push
- [ ] `.trellis/config.yaml` 的 `session_auto_commit`、journal、hook、package 策略已确认或明确跳过
- [ ] 任务开始前已读 `.trellis/spec/firmware/pitfalls-index.md`
- [ ] IAR 项目：能在命令行用 `iarbuild <项目>.ewp -build Debug` 跑通编译
- [ ] STM32CubeIDE/GCC-style 项目：能在命令行用 `make -j8 all` 跑通编译
