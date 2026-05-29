---
name: mcu-bootstrap
description: "After `trellis init -t mcu-stm32-base`, run this once to finish project-root configuration that trellis init cannot write itself: AGENTS.md append, .gitignore merge, doc/, version macros, svn:ignore merge, optional git remote. Recoverable and idempotent. Use when the user just initialized a new MCU firmware project from the mcu-stm32-base template, or says '跑一下 bootstrap' / 'mcu bootstrap' / 'finish init'."
---

# MCU Project Bootstrap

This skill finalizes a new MCU firmware project after `trellis init -t mcu-stm32-base -r <registry>`. `trellis init` copies the spec into `.trellis/spec/`; this skill writes project-root files that the template installer cannot safely create.

## Trigger Conditions

Invoke when:

- User just ran `trellis init -t mcu-stm32-base -r ...` in a new MCU project
- User explicitly mentions this skill: `/mcu-bootstrap`, "跑 mcu-bootstrap", "用 mcu-bootstrap skill", "跑 bootstrap", "mcu 引导", "finish init"
- User asks "把项目初始化收尾" / "配一下新项目"

Do **not** invoke when:

- `AGENTS.md` already contains `<!-- MCU-BOOTSTRAP:DONE` and the user did not ask to audit or repair bootstrap state
- Project is not a Trellis MCU project from this template, for example `.trellis/spec/firmware/version-control.md` is missing

This skill is **agent-agnostic**. Use the current agent's available interaction mechanism. If structured question tools are available, use them; otherwise ask the user in a normal message and wait. If the tool only supports fewer questions per call, split the questions.

## Recovery And Idempotency Rules

- `<!-- MCU-BOOTSTRAP:STARTED ... -->` means bootstrap began and may be partial. It is safe to re-run and repair.
- `<!-- MCU-BOOTSTRAP:DONE ... -->` means all required steps completed or were explicitly marked N/A by this workflow.
- Never write `DONE` until the final summary step.
- If a step fails, keep already completed changes, report the failed step, and leave `STARTED` in place.
- Existing user content must be preserved. Merge missing rules instead of overwriting whole files whenever possible.
- If a required version header does not exist, continue independent steps, report the version macro as pending, and do **not** write `DONE`.

## Workflow

### Step 1 - Preflight And State Check

1. Verify `.trellis/spec/firmware/version-control.md` exists.
2. Read root `AGENTS.md` if it exists.
3. If `AGENTS.md` contains `<!-- MCU-BOOTSTRAP:DONE`, tell the user bootstrap is already complete and stop unless they asked for an audit.
4. If `AGENTS.md` does not exist, create it with a short Trellis note and continue.

### Step 2 - Detect Toolchain

Check project root for indicators:

- IAR: `EWARM/` exists, or any `EWARM/*.ewp` / `EWARM/*.eww` exists
- STM32CubeIDE+GCC: `.cproject`, `.project`, `.mxproject`, or `*.ioc` exists

Decision rules:

- IAR only -> use **IAR project**
- CubeIDE only -> use **STM32CubeIDE+GCC project**
- Both or neither -> ask the user explicitly; do not assume a default toolchain

Record the selected toolchain for `.gitignore` and version macro paths.

### Step 3 - Detect SVN Working Copy

Check current directory and its parent chain for `.svn`:

- If found, record the SVN working-copy directory and the project directory where `svn:ignore` should be set.
- If not found, mark SVN as N/A and skip only the SVN step.

### Step 4 - Collect Project Variables

Ask for:

1. **项目代号** (Project Code), for example `<PROJECT_CODE>`. Used in bootstrap marker, version stamps, SVN log templates, and binary naming guidance.
2. **初始固件版本号**, default `v1.00.00` for IAR-style projects and `v1.0.0` for semantic-version projects.
3. **Git remote URL** (optional). Empty means skip remote binding for now.
4. **是否有独立算法核心需要 ALGO_VERSION**, default no. If yes, ask for initial `ALGO_VERSION`, for example `v1.0`.
5. **AI 协同模式** (`COLLABORATION_MODE`), one of `codex-only` / `dual-track` / `claude-only`. Default `codex-only` (team default). Stored as a marker in the AGENTS.md managed block so future agents can branch on it without guessing.

### Step 5 - Add Or Repair AGENTS.md Managed Block

Append or update a single managed block after `<!-- TRELLIS:END -->` if that marker exists; otherwise append at the end of `AGENTS.md`.

Use exactly one managed block:

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

At the final successful step, replace the `STARTED` line inside this block with:

```markdown
<!-- MCU-BOOTSTRAP:DONE {{TODAY_YYYY-MM-DD}} 项目代号: {{PROJECT_CODE}} -->
```

### Step 6 - Merge `.gitignore`

Create `.gitignore` if missing. If it exists, preserve user content and append a managed block only when missing.

Use:

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

Important:

- Do not ignore shared Trellis content such as `.trellis/spec/` and `.trellis/tasks/`.
- The `*.hex` / `*.bin` / `*.out` ignore rules must appear before the `!Bin/*` release exceptions.

### Step 7 - Create `doc/` And `doc/README.md`

Create `doc/` if missing.

PowerShell:

```powershell
New-Item -ItemType Directory -Path "doc" -Force
```

Bash:

```bash
mkdir -p doc
```

If `doc/README.md` exists, do not overwrite it. If missing, write:

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

### Step 8 - Inject Version Macros

Locate version header by toolchain:

- IAR: `User/App/System/define.h`
- STM32CubeIDE+GCC: `User/App/Inc/define.h`

If the selected file does not exist:

- Report the missing file.
- Continue independent SVN/Git steps.
- Do not write `DONE`.
- Tell the user to add the file and re-run this skill, or manually add:

```c
#define SOFTWARE_VERSION        "{{INITIAL_VERSION}}"   /* 整体固件版本 */
#define SOFTWARE_VERSION_DATE   "{{TODAY_YYYYMMDD}}"    /* 当前版本发布日期 */
```

For STM32CubeIDE/GCC-style projects, use:

```c
#define VERSION                 "{{INITIAL_VERSION}}"   /* 整体固件版本 */
#define VERSION_DATE            "{{TODAY_YYYYMMDD}}"    /* 当前版本发布日期 */
```

Detection rules:

- Test each macro independently with an anchored pattern such as `^\s*#\s*define\s+SOFTWARE_VERSION\b`.
- Do not treat commented lines as existing macros.
- If a macro exists, report its current value and leave it unchanged.
- If only some macros are missing, add only the missing macros.
- If `ALGO_VERSION` was requested and missing, add it even if the firmware version macro already exists.
- Prefer inserting before the final include-guard `#endif`; if unsure, append near the existing version macro block.

This step modifies a `.h` file. `mcu-bootstrap` is an explicit one-time setup exception to the normal task/code phase boundary because it only writes version constants.

### Step 9 - Merge SVN `svn:ignore` If SVN Exists

Do not overwrite existing `svn:ignore`. Read current values, merge the required entries, then write the union.

`svn:ignore` matches basenames in the directory where the property is set. It does not behave like Git's repository-wide path patterns. Set the root project ignore first; if nested IAR output directories such as `EWARM/<target>/Obj` still appear as unversioned, set `svn:ignore` on their parent directories with basename entries such as `Obj`, `Exe`, `List`, and `BrowseInfo`.

Required entries:

```text
.agents
.claude
.codex
.trellis
.git
.gitignore
AGENTS.md
doc
ref_doc
ref_docs
EWARM/Backup*
*.pbi
*.pbw
*.pbd
*.browse
.ninja_deps
.ninja_log
build.ninja
Debug
Release
.qoder
.lingma
.uv-cache
tmp
Thumbs.db
Desktop.ini
.DS_Store
*.swp
*~
*.bak
~$*
```

Windows PowerShell example:

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

Bash example:

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

Do not `svn commit`. Tell the user this property will be included with the next explicit SVN commit.

### Step 10 - Git Remote

If a Git remote URL was provided:

PowerShell:

```powershell
if (-not (Test-Path -LiteralPath ".git")) {
  git init -b main
}
$existing = git remote get-url origin 2>$null
if (-not $existing) {
  git remote add origin <URL>
} elseif ($existing -eq "<URL>") {
  Write-Host "origin already points to <URL>"
} else {
  git remote set-url origin <URL>
}
git remote -v
```

Bash:

```bash
if [ ! -d .git ]; then
  git init -b main
fi
existing="$(git remote get-url origin 2>/dev/null || true)"
if [ -z "$existing" ]; then
  git remote add origin "<URL>"
elif [ "$existing" = "<URL>" ]; then
  printf '%s\n' "origin already points to <URL>"
else
  git remote set-url origin "<URL>"
fi
git remote -v
```

If `origin` exists and points to a different URL, confirm with the user before changing it unless they explicitly provided the URL as a replacement. Do not push.

### Step 11 - Trellis Config And Session Guidance

If `.trellis/config.yaml` exists, review and report these optional settings without forcing changes:

- `session_commit_message`
- `max_journal_lines`, commonly `2000`
- `session_auto_commit`, often `false` when `.trellis/` session notes are manually reviewed
- lifecycle hooks such as `after_create`, `after_start`, `after_finish`, and `after_archive`
- `packages`, `default_package`, and Codex `dispatch_mode`

Generic hooks should receive `TASK_JSON_PATH`. Hook failures should warn and keep core task operations usable.

Do not copy project runtime machinery such as `.trellis/scripts/` or third-party hooks into this template-driven bootstrap. Only document the config choice and local-state ignore policy.

Confirm `.gitignore` keeps local session state ignored:

```gitignore
.trellis/workspace/*/journal-*.md
.trellis/.runtime/
.trellis/.backup-*/
```

Do not ignore shared context:

```gitignore
!.trellis/spec/
!.trellis/tasks/
```

### Step 12 - Finalize Marker And Summary

Only if required steps are complete:

1. Replace `<!-- MCU-BOOTSTRAP:STARTED ... -->` with `<!-- MCU-BOOTSTRAP:DONE {{TODAY_YYYY-MM-DD}} 项目代号: {{PROJECT_CODE}} -->`.
2. Print:

```text
✓ mcu-bootstrap 完成

项目代号: <project_code>
工具链: <IAR / STM32CubeIDE>
SVN: <已合并 svn:ignore / N/A>
Git remote: <已绑定 / 已跳过 / 待用户后续配置>

已完成步骤:
✓ AGENTS.md 协同规则 managed block
✓ .gitignore managed block
✓ doc/ + README.md
✓ 版本宏检查/写入 <path>
[✓ SVN svn:ignore 合并（待下次 SVN 提交带上）]
[✓ Git remote origin 绑定/更新]

下一步:
1. 任务开始前先读 `.trellis/spec/firmware/pitfalls-index.md`
2. 准备好后，用 "提交 git" 触发首次 Git 提交/推送
3. SVN 提交时记得 svn:ignore 属性也要带（一次性的）
4. 开始第一个 Trellis 任务：python ./.trellis/scripts/task.py create "<title>"
```

If required steps are pending, do not write `DONE`. Print the completed steps and the exact pending action.

## Error Handling

- Any step fails -> do not clean up completed state; report the failed step and leave `STARTED`.
- User stops midway -> stop immediately, keep completed changes, and list what remains.
- Existing files with user content -> preserve and merge.
- No Git remote URL -> skip remote binding and still allow `DONE`.
- No SVN working copy -> mark SVN as N/A and still allow `DONE`.

## Not In Scope

- Do not create `.cproject`, `.ioc`, `.ewp`, or IDE project files.
- Do not install toolchains.
- Do not create source code such as `Core/Src/main.c`.
- Do not modify `.trellis/` files already copied from the template.
- Do not run `git push` or `svn commit`.
