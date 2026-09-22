# ApexInfer 团队日常 Git 工作流程

> 适用成员：陈梓弘、周邦翔、朱健辉
>
> 适用仓库：<https://github.com/ApexForge-cz/apexinfer-minicpm>
>
> 默认主分支：`flagos-2026-s2`
>
> 完整项目计划：[PROJECT_EXECUTION_PLAN.md](./PROJECT_EXECUTION_PLAN.md)

## 1. 先理解这四个概念

```text
官方仓库 upstream/flagos-2026-s2（只拉取，不推送）
                         ↓
团队主分支 origin/flagos-2026-s2（稳定、已审核）
                         ↓
每人自己的任务分支（实际修改代码）
                         ↓
Pull Request 审核后合并回团队主分支
```

| 名称 | 含义 | 能否直接修改 |
|---|---|---|
| `upstream` | FlagOS 官方仓库 | 不能 Push |
| `origin` | 团队 Fork `ApexForge-cz/apexinfer-minicpm` | 可以 Push 自己的任务分支 |
| `flagos-2026-s2` | 团队默认主分支、赛事指定基线 | 不直接开发，通过 PR 合并 |
| 任务分支 | 每个 Issue 对应的开发分支 | 在这里修改和提交 |

核心规则：**一个 Issue、一个任务分支、一个 Pull Request。**

## 2. 第一次加入项目

以下操作每台电脑只做一次。

### 2.1 接受邀请

项目负责人先在 GitHub 仓库中邀请成员：

```text
Settings → Collaborators → Add people
```

组员登录自己的 GitHub 账号并接受邀请。没有接受邀请时，只能读取公开仓库，不能向 `origin` Push。

### 2.2 安装并登录工具

需要安装：

- Git；
- GitHub CLI `gh`，建议使用；
- 开发所需的 Python、编辑器和项目依赖。

登录 GitHub：

```powershell
gh auth login
gh auth status
gh auth setup-git
```

`gh auth status` 必须显示当前使用的是自己的 GitHub 账号。

### 2.3 设置 Git 身份

姓名建议写真实姓名，邮箱使用 GitHub 绑定邮箱或 GitHub noreply 邮箱。

```powershell
git config --global user.name "你的姓名"
git config --global user.email "你的GitHub邮箱"
git config --global pull.ff only
```

检查：

```powershell
git config --global user.name
git config --global user.email
```

不要三个人共用同一个 Git 身份或 GitHub Token。

### 2.4 克隆团队仓库

在自己选择的开发目录执行：

```powershell
git clone https://github.com/ApexForge-cz/apexinfer-minicpm.git
cd apexinfer-minicpm
git switch flagos-2026-s2
```

添加官方只读远程：

```powershell
git remote add upstream https://github.com/flagos-ai/vllm-plugin-FL.git
git remote set-url --push upstream no_push
git remote -v
```

预期结果：

```text
origin    https://github.com/ApexForge-cz/apexinfer-minicpm.git
upstream  https://github.com/flagos-ai/vllm-plugin-FL.git
```

### 2.5 验证权限

组员不要向主分支写测试提交。用一个临时分支验证：

```powershell
git switch -c chore/verify-你的英文名
git push -u origin chore/verify-你的英文名
```

Push 成功后删除远端和本地临时分支：

```powershell
git push origin --delete chore/verify-你的英文名
git switch flagos-2026-s2
git branch -d chore/verify-你的英文名
```

## 3. 每次开始工作前

每次打开项目，必须按顺序执行，不要直接修改文件。

### 3.1 进入项目并检查当前状态

```powershell
cd "你的路径\apexinfer-minicpm"
git status --short --branch
```

如果只显示类似内容，说明工作区干净：

```text
## flagos-2026-s2...origin/flagos-2026-s2
```

如果出现 `M`、`A`、`D` 或 `??`，说明有未提交修改。先完成以下三种处理之一：

1. 修改属于当前任务：完成检查后提交；
2. 暂时不提交：使用 `git stash push -u -m "说明"` 暂存；
3. 不确定是谁的修改：停止操作，联系项目负责人。

禁止为了“清理”工作区执行 `git reset --hard` 或 `git checkout -- .`。

### 3.2 获取远端最新状态

```powershell
git fetch origin --prune
git fetch upstream --prune
```

`fetch` 只下载远端状态，不会覆盖本地文件。

### 3.3 查看自己的 Issue

打开 Issues：

<https://github.com/ApexForge-cz/apexinfer-minicpm/issues>

开始任务前确认：

- 找到 Issue 中分配给自己的任务编号、个人分支和交付物；
- 自己只处理该个人任务，不代替其他成员完成或签署任务；
- 前置阶段已经完成；
- Issue 的阶段候选分支已经由负责人创建并推送；
- 允许修改、禁止修改和验收条件明确；
- 在 Issue 下留言“开始处理 `任务编号`，分支：`个人分支`，预计交付：`文件路径`”。

没有对应 Issue，不开始正式开发。临时发现的问题先新建 Issue 或在原 Issue 中确认范围。

## 4. 开始一个新任务

### 4.1 更新 Issue 指定的基准分支

```powershell
git fetch origin --prune
git switch <阶段候选分支>
git pull --ff-only origin <阶段候选分支>
```

例如 S4 的个人任务统一从 `phase4/release-candidate` 创建，不从 `flagos-2026-s2` 或其他成员分支创建。如果候选分支尚不存在，联系阶段负责人创建，不得自行换一个基准。

如果 `pull --ff-only` 失败，说明本地分支出现了额外提交或分叉。不要强行处理，联系项目负责人。

### 4.2 从最新阶段候选分支创建个人任务分支

分支名必须与 Issue 对应：

```powershell
git switch -c <Issue 指定的个人分支>
```

示例：

```powershell
# S1 周邦翔基线任务
git switch phase1/release-candidate
git switch -c task/zhou-s1-benchmark

# S4 陈梓弘调度与 KV Cache 任务
git switch phase4/release-candidate
git switch -c task/chen-s4-scheduler-kv

# S7 朱健辉答辩证据任务
git switch phase7/release-candidate
git switch -c task/zhu-s7-presentation-evidence
```

创建后检查：

```powershell
git branch --show-current
git status --short --branch
```

必须确认当前分支就是 Issue 分配给自己的 `task/...` 分支，再开始修改。

### 4.3 首次 Push 任务分支

可以在开始工作后立即建立远端分支：

```powershell
git push -u origin <分支名>
```

设置 `-u` 后，后续只需执行 `git push`。

## 5. 修改代码时

### 5.1 保持范围小

- 一个分支只解决一个 Issue；
- 不顺手重构无关代码；
- 不修改官方 Benchmark 的场景、请求、并发和指标口径；
- 不提交模型、数据集、Token、密码、个人简历或大型原始日志；
- 性能实验必须记录平台、提交号、命令和原始结果位置；
- 代码改动必须有对应测试或明确说明为什么无法本地测试。

### 5.2 随时查看修改

```powershell
git status --short
git diff
```

只查看某个文件：

```powershell
git diff -- path/to/file.py
```

### 5.3 小步提交

建议每完成一个独立、可检查的小步骤就提交一次。不要把几天工作压成一个巨大提交，也不要每改一行就提交。

## 6. 提交前检查

### 6.1 通用检查

```powershell
git diff --check
git status --short
```

`git diff --check` 不应输出空白错误。

### 6.2 文档任务

至少检查：

- Markdown 标题层级和链接；
- 命令、分支名、Issue 编号是否正确；
- 是否包含密码、Token、个人敏感信息；
- 文档是否与实际代码、结果和当前阶段一致。

### 6.3 Python 或框架代码

根据改动范围运行最小相关测试：

```powershell
# 格式与静态检查
pre-commit run --all-files

# 无需专用硬件的单元测试
pytest tests/unit_tests/ -v

# 或只运行相关测试文件
pytest tests/unit_tests/ops/test_layernorm.py -v
```

如果本地环境不能运行完整测试，在 PR 中必须写明：

- 实际运行了什么；
- 哪些没有运行；
- 没运行的原因；
- 将在哪个官方硬件阶段补测。

### 6.4 性能任务

性能结果不能只写一个“最好值”。至少包含：

- 平台和机器；
- 代码提交号；
- 构建、启动和 Benchmark 命令；
- 4k/16k 场景；
- 至少 3 次有效重复；
- `total tokens/s`、TTFT、失败请求和显存；
- 基线与优化中位数；
- 精度结果或明确的待补测状态。

## 7. 创建本地提交

### 7.1 只暂存本次任务文件

不要习惯性使用 `git add .`。优先明确列出文件：

```powershell
git add path/to/file1.py path/to/file2.py
```

查看即将提交的内容：

```powershell
git diff --cached
git status --short
```

发现无关文件时，从暂存区移出但保留本地修改：

```powershell
git restore --staged path/to/unrelated-file
```

### 7.2 提交信息格式

```text
<类型>: <做了什么>
```

常用类型：

| 类型 | 用途 | 示例 |
|---|---|---|
| `perf` | 性能优化 | `perf: reuse attention metadata buffers` |
| `bench` | Benchmark 与实验工具 | `bench: add baseline result validator` |
| `test` | 测试 | `test: cover metax graph fallback` |
| `fix` | 缺陷修复 | `fix: avoid stale kv block mapping` |
| `docs` | 文档 | `docs: add team workflow` |
| `chore` | 仓库和工具维护 | `chore: ignore profiler traces` |

提交：

```powershell
git commit -m "bench: add baseline result validator"
```

不要使用 `update`、`修改一下`、`test`、`final` 等无法判断内容的提交信息。

## 8. Push 到团队仓库

```powershell
git push
```

第一次 Push 没有设置上游时：

```powershell
git push -u origin <分支名>
```

确认远端分支：

```powershell
git status --short --branch
```

Push 的是自己的任务分支，不是官方 `upstream`，也不直接 Push `flagos-2026-s2`。

## 9. 创建 Pull Request

### 9.1 GitHub 页面创建

Push 后打开仓库：

<https://github.com/ApexForge-cz/apexinfer-minicpm>

点击 `Compare & pull request`，确认：

```text
base: Issue 中的 phaseN/release-candidate
compare: Issue 中你的 task/... 个人分支
```

不要把 Base 选成 `main`、`flagos-2026-s2`、阶段集成分支或其他成员分支。

### 9.2 命令行创建

```powershell
gh pr create `
  --base <阶段候选分支> `
  --head <任务分支> `
  --title "<类型>: <清晰标题>" `
  --fill
```

任务未完成时创建 Draft PR：

```powershell
gh pr create --draft --base <阶段候选分支> --head <任务分支> --fill
```

### 9.3 PR 必填内容

按照仓库模板填写：

- PR Category；
- PR Type；
- Description：为什么改、改了什么；
- Related Issues：个人任务 PR 使用 `Refs #编号`，不要提前关闭阶段 Issue；
- Changes：主要文件和机制；
- Testing：命令、平台、结果；
- 性能 PR 额外写基线、优化结果、TTFT、精度、风险和回退方式。

示例：

```markdown
### Related Issues
Refs #2

### Task
- Task ID: S1-Z03
- Deliverables: `docs/evidence/phase-1/zhou-benchmark/baseline-results.csv`

### Testing
- `pytest tests/unit_tests/... -v`: passed
- Metax C500, 4k: 3 runs, median ... tok/s
- MATH-500 Level 3: accuracy ...
- Not tested on BI-V150 yet: waiting for allocated hardware
```

## 10. PR 审核与合并

### 10.1 作者要做什么

- 在群里发送 PR 链接和一句话目标；
- 指定至少一名非作者审核；
- 回答评论并继续在原分支提交修改；
- 每次 Push 后确认 PR 已更新；
- 所有验收项完成后，将 Draft 改为 Ready for review。

### 10.2 审核者检查什么

- 改动是否符合 Issue 范围；
- 是否改动了禁止修改的 Benchmark 口径；
- 实现机制是否有证据；
- 测试能否支持结论；
- 性能是否超过 1% 波动且没有 TTFT/精度退化；
- 是否存在模型、数据、密钥、个人信息或大型日志；
- 是否有清晰回退方式；
- 文档、代码和实验记录是否一致。

### 10.3 CI 注意事项

当前上游 `.github/workflows/ci.yml` 的自动 PR 触发目标是 `main`，而团队比赛主分支是 `flagos-2026-s2`。因此，PR 页面没有自动 CI 结果时，**不代表测试通过**。

现阶段必须以 PR 中记录的本地测试和官方硬件测试为准。需要时由负责人通过 GitHub Actions 手动触发适用工作流。未经测试记录，不合并。

### 10.4 合并权限

- 普通组员不直接向阶段候选、阶段集成或赛事主分支 Push；
- 项目负责人在审核通过后合并；
- 高风险性能改动必须由非作者复核；
- S6 之后进入代码冻结，只接受阻断性修复和交付文档。

## 11. PR 合并后

每位成员在自己的电脑执行：

```powershell
git switch flagos-2026-s2
git pull --ff-only origin flagos-2026-s2
```

删除已经合并的本地任务分支：

```powershell
git branch -d <已合并分支>
```

如果 GitHub 没有自动删除远端分支，由分支作者执行：

```powershell
git push origin --delete <已合并分支>
```

然后在对应 Issue 中记录个人任务 PR、合并提交、交付物和验收人。个人 PR 不关闭阶段 Issue；只有阶段收口 PR 达到关闭门禁后，才由负责人关闭对应 Issue。

## 12. 同一个任务第二天继续做

如果任务分支还没有合并：

```powershell
cd "你的路径\apexinfer-minicpm"
git status --short --branch
git fetch origin --prune
git switch <你的任务分支>
git pull --ff-only origin <你的任务分支>
```

如果阶段候选分支在此期间有新提交，把它合入任务分支：

```powershell
git merge origin/<阶段候选分支>
```

解决可能的冲突、运行测试，然后：

```powershell
git push
```

团队初学阶段统一使用 `merge` 同步主分支，不要求成员对已经 Push 的共享分支执行 `rebase` 或 force push。

## 13. 发生冲突怎么办

### 13.1 查看冲突

```powershell
git status
```

冲突文件中会出现：

冲突块由“当前分支内容、分隔线、主分支内容”三部分组成。编辑器通常会用 `Current Change` 和 `Incoming Change` 标记它们；不要把这些标记保留在最终文件中。

与相关成员确认正确内容后，删除标记并保留最终版本。

### 13.2 完成冲突合并

```powershell
git add <已解决文件>
git commit
git push
```

### 13.3 不确定时取消本次合并

```powershell
git merge --abort
```

然后联系项目负责人。不要用 `reset --hard`、force push 或删除他人分支来“解决”冲突。

## 14. 临时切换任务

原则上先完成当前小任务再切换。确需临时切换时：

```powershell
git status --short
git stash push -u -m "WIP: #Issue编号 简短说明"
git switch flagos-2026-s2
```

返回原任务：

```powershell
git switch <原任务分支>
git stash list
git stash pop
```

`stash` 只是短期保管，不是备份。超过一天的有效工作应形成清晰提交并 Push 到自己的任务分支。

## 15. 负责人同步官方更新

只有项目负责人执行官方同步。普通组员不要自行把 `upstream` 合并进团队主分支。

```powershell
git switch flagos-2026-s2
git status --short --branch
git fetch upstream
git merge upstream/flagos-2026-s2
```

同步前必须确认官方更新是否符合“禁止直接合并最新代码获得性能提升”的赛事规则。任何官方同步都要单独记录来源提交、原因和影响，并经过团队审核后 Push。

## 16. 三名成员的日常入口

### 陈梓弘

1. 查看 S0-S7 依赖和阻塞项；
2. 确认当天允许启动的 Issue；
3. 处理运行时、调度、KV Cache 或算子候选；
4. 审核其他成员 PR，检查赛题合规和证据链；
5. 每天结束前更新 Issue 状态和下一步。

### 周邦翔

1. 同步主分支并查看 Benchmark/测试 Issue；
2. 维护环境、脚本、实验登记和结果校验；
3. 对性能改动运行重复实验和回归；
4. 在 PR 中给出原始结果路径、中位数、波动和失败情况；
5. 不在没有 profiler 证据时独立启动内核重写。

### 朱健辉

1. 同步主分支并查看可视化、辅助脚本或测试执行任务；
2. 只从锁定 CSV/JSON 生成图表，不手工修改数值；
3. 协助运行测试、整理结果和展示材料；
4. 不修改调度器、KV Cache 或底层内核，除非 Issue 明确调整且有人协作审核；
5. 在 PR 中说明数据来源和生成命令。

## 17. 每日开工清单

```text
[ ] git status，确认没有来历不明的修改
[ ] git fetch origin --prune
[ ] 查看并领取 Issue
[ ] 更新 flagos-2026-s2
[ ] 切换或创建自己的任务分支
[ ] 确认当前分支不是主分支
[ ] 开始修改
```

## 18. 每日收工清单

```text
[ ] git diff，检查改动范围
[ ] 运行适用测试并记录结果
[ ] git add 明确文件，不盲目 git add .
[ ] git diff --cached，复核提交内容
[ ] 使用清晰 commit message
[ ] git push 到自己的任务分支
[ ] 更新 PR 或创建 Draft PR
[ ] 更新 Issue：完成内容、证据、阻塞和下一步
[ ] 确认没有模型、数据、密钥、个人材料或大型日志被提交
```

## 19. 最短命令速查

### 开始新任务

```powershell
git status --short --branch
git fetch origin --prune
git switch <阶段候选分支>
git pull --ff-only origin <阶段候选分支>
git switch -c <Issue 指定的个人分支>
git push -u origin <任务分支>
```

### 提交工作

```powershell
git diff --check
git status --short
git add <本次任务文件>
git diff --cached
git commit -m "<类型>: <清晰说明>"
git push
```

### 创建 PR

```powershell
gh pr create --base <阶段候选分支> --head <任务分支> --fill
```

### 合并后同步

```powershell
git switch <阶段候选分支>
git pull --ff-only origin <阶段候选分支>
git branch -d <已合并分支>
```

## 20. 遇到问题时先提供这些信息

在群里求助时，不要只发“Git 报错了”。发送：

```powershell
git status --short --branch
git branch --show-current
git remote -v
git log -5 --oneline --decorate
```

再附上：

- 正在处理的 Issue 链接；
- 刚执行的完整命令；
- 完整错误文本；
- 是否有未提交修改；
- 期望达到的结果。

不要发送 GitHub Token、平台密码或包含密钥的配置文件。
