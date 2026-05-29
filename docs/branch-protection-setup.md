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

1. 打开 `https://github.com/mantaXray/trellis-templates/settings/branches`
2. 点击 "Add branch protection rule"
3. **Branch name pattern**: `main`
4. 勾选以下规则：
   - ✅ **Require a pull request before merging**
     - ✅ Require approvals → 1（如果只有你 owner，也建议保留，强制自己走 PR 习惯）
     - ✅ Dismiss stale pull request approvals when new commits are pushed
   - ✅ **Require status checks to pass before merging**
     - ✅ Require branches to be up to date before merging
     - 在搜索框输入 `validate`，勾选 GitHub Actions 那个 workflow
   - ✅ **Do not allow bypassing the above settings**（这条很关键，否则 admin 默认可绕过）
   - ✅ **Restrict deletions**
   - ✅ **Allow force pushes**: 取消勾选（默认不允许）
5. 点击 "Create" / "Save changes"

配完后，验证：`git push origin main` 直接推会被服务端拒绝，提示要走 PR。

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
