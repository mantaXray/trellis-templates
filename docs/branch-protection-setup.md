# Branch Protection 配置

> 给希望"接手维护本仓库"的人保留一道审核闸门。Owner（你自己）也建议走 PR 流程，避免直接推 main 时绕过 CI。
>
> 三个 remote（origin/gitee/github）需要分别配。最重要的是 GitHub（如果未来开源），其次是公司 Gitea（多人协作场景）。Gitee 通常作为镜像，强制规则可选。

## 推荐的保护规则

| 规则 | 必要性 |
|---|---|
| 直接 push 到 `main` 被拦截（包括 owner） | 强烈推荐 |
| 必须经过 PR 才能合并 | 强烈推荐 |
| Validator CI 必须绿才能合并 | 强烈推荐 |
| 至少 1 个 reviewer approve 才能合并 | 推荐（仅你 owner 时可选） |
| 禁止 force push 到 `main` | 必须 |
| 禁止删除 `main` | 必须 |

---

## GitHub

GitHub 有两套机制，但**在 free personal account 的 private 仓库里能用的只有一套**：

| 机制 | 推出时间 | private + free personal | private + Pro/Team Org | public |
|---|---|---|---|---|
| **Branch protection rules**（老版） | 老 | ✅ 可用 | ✅ 可用 | ✅ 可用 |
| **Repository Rulesets**（新版） | 2023 | ❌ **不可用** | ✅ 可用 | ✅ 可用 |

实测 free personal + private 仓库打开 Rulesets 页面会被提示："To access rulesets in a private repository, you'll need to move it to a GitHub Team organization account."

**因此本仓库（private + free personal）当前推荐路径是 Branch protection rules**。未来仓库变 public 或升 Pro / 转 Team Org，再迁移到 Rulesets。

### 当前推荐路径：Branch protection rules（free 可用）

1. 打开 `https://github.com/mantaXray/trellis-templates/settings/branches`
2. 点击 **Add branch protection rule**
3. **Branch name pattern**: `main`
4. 勾选：
   - ✅ **Require a pull request before merging**
     - Required approvals: `1`（仅 owner 时也建议保留，强制自己走 PR 习惯）
     - ✅ Dismiss stale pull request approvals when new commits are pushed
   - ✅ **Require status checks to pass before merging**
     - ✅ Require branches to be up to date before merging
     - 在搜索框输入 `validate`，勾选 GitHub Actions 那个 workflow
   - ✅ **Do not allow bypassing the above settings**（**这条很关键**，否则 admin 默认可绕过）
   - ✅ **Restrict deletions**
   - 取消勾选 **Allow force pushes**（默认就是不勾）
5. 点击 **Create** / **Save changes**

### 未来升级路径：Repository Rulesets（仓库 public 或账户升级后再做）

满足以下任一条件时可以迁移到 Rulesets：

- 仓库改为 public
- 升级到 GitHub Pro（个人付费版，private 仓库支持 Rulesets）
- 仓库移到 GitHub Team Organization 账户

迁移步骤：

1. 打开 `https://github.com/mantaXray/trellis-templates/settings/rules`
2. 点击 **New ruleset** → **New branch ruleset**
3. **Ruleset name**: `main protection`
4. **Enforcement status**: `Active`
5. **Bypass list**: 留空
6. **Target branches** → **Add target** → **Include default branch**
7. **Branch rules** 勾选：
   - ✅ Restrict deletions
   - ✅ Block force pushes
   - ✅ Require a pull request before merging → Required approvals: 1
   - ✅ Require status checks to pass → Add checks: `validate`，勾选 Require branches to be up to date
   - 可选：Require linear history / Require signed commits
8. **Create**
9. 迁移完成后，到 `settings/branches` 把老的 Branch protection rule 删掉，避免两套规则叠加冲突

---

## Gitee

1. 打开仓库 → **管理** → **分支管理**
2. 找到 `main` 分支，点击 **保护**
3. 设置：
   - ✅ **仅 Owner / Admin 可推送**
   - ✅ **不允许强推（force push）**
   - ✅ **不允许删除**
4. 如果团队加了协作者，再去「PR 设置」要求 N 个 approve

> ⚠️ Gitee Actions 的状态检查可能需要单独配置；如果项目不依赖 Gitee CI（本仓库 CI 主要在 GitHub），可以跳过 status check 要求，只靠人工 review。

---

## 公司 Gitea (origin)

1. 打开仓库 → **Settings** → **Branches**
2. **Protected Branches** 部分，点击 **Add Rule**
3. **Branch Name**: `main`
4. 勾选：
   - ✅ **Enable Push Whitelist** → 只允许 owner / specific users
   - ✅ **Enable Merge Whitelist** → 同上
   - ✅ **Require approval from N reviewers**（按团队规模填，1~2 较常见）
   - ✅ **Block on official review requests**
   - ✅ **Block on outdated branch**
5. 保存

> Gitea 1.19+ 也支持 Gitea Actions 做 status check；如果公司 Gitea 开了 Actions 并跑了 validator，可在 **Status Check Patterns** 里加 `validate`。

---

## 验证配完没

最简单的方式：

```bash
# 1. 在本地 main 上做个无意义的小改动
echo "" >> README.md
git add README.md && git commit -m "test: verify branch protection"

# 2. 试着直接推
git push origin main
# 期望：服务端拒绝，提示需要 PR

# 3. 撤销刚才的 test commit
git reset --hard HEAD~1
```

如果三个 remote 中任何一个能直接推上去，说明那个 remote 的保护没生效。

---

## 紧急绕过

如果出现真正的紧急情况（CI 跑不通且必须立刻 push），可以临时关掉 branch protection：
- GitHub: Settings → Branches → 该规则旁边 "Disable"，处理完立刻 "Enable"
- Gitee/Gitea: 类似操作

**不要养成 `--no-verify` 习惯**——本地 pre-push hook 可以跳过，但服务端 branch protection 是最后一道防线。
