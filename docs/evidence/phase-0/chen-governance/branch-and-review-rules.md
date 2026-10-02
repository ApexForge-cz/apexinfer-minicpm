# S0-C02 分支、Pull Request 与 Review 规则

## 1. 个人任务开工

```bash
git status --short --branch
git fetch origin --prune
git switch phaseN/release-candidate
git pull --ff-only origin phaseN/release-candidate
git switch -c task/<member>-sN-<topic>
git push -u origin task/<member>-sN-<topic>
```

个人任务开始当天即可创建 Draft PR。个人 PR 的 Base 必须是对应阶段的 `phaseN/release-candidate`。

## 2. 个人提交规则

- 一个提交只表达一个可验证变化；
- 只暂存本人任务文件，不使用 `git add .` 混入无关修改；
- 提交前执行 `git diff --check`、适用测试和敏感信息检查；
- 提交信息格式：`<type>(sN): <summary>`；
- 禁止在已有未确认修改时使用 `git reset --hard` 或 `git checkout -- .`。

## 3. PR 必填内容

每个个人 PR 必须写明：

- 任务编号；
- `Refs #<Issue>`，个人 PR 不使用 `Closes` 提前关闭阶段 Issue；
- 交付物路径；
- 修改范围和未修改范围；
- 验证命令与实际结果；
- 尚未完成或受阻内容；
- 回退方式；
- 至少一名非作者评审人。

## 4. 并行 Review 规则

- 三名成员个人 PR 可以独立 Review 和合并，不等待其他成员个人任务完成；
- 作者不能批准自己的 PR；
- 文档/模板 PR 检查路径、命令、字段、敏感信息和可执行性；
- 性能代码 PR 由非作者检查机制、正确性、测试、Benchmark 合规和回退；
- 测量/图表 PR 检查输入 schema、统计口径、原始结果追溯和 `sample` 标识；
- Review 只对当前 PR 范围负责，不以“另一成员尚未完成”为拒绝当前个人 PR 的理由。

## 5. 合并层级

```text
个人 task 分支
  → phaseN/release-candidate
  → phaseN/integration
  → flagos-2026-s2
```

1. 个人 PR 通过本人验收和至少一名非作者 Review 后合并到候选分支；
2. 三个个人 PR 齐全后，在阶段集成分支执行正式数据组合和交叉验证；
3. 阶段关闭门禁通过后，提交唯一阶段收口 PR 到 `flagos-2026-s2`；
4. 只有阶段收口 PR 才能关闭对应阶段 Issue。

## 6. 当前 Phase 0 Review 分配

| PR | 作者 | 主要评审人 | 目标分支 |
|---|---|---|---|
| #9 | 周邦翔 | 陈梓弘，朱健辉可补充 | `phase0/release-candidate` |
| #10 | 陈梓弘 | 周邦翔或朱健辉 | `phase0/release-candidate` |
| 朱健辉后续 PR | 朱健辉 | 陈梓弘或周邦翔 | `phase0/release-candidate` |

## 7. 冲突处理

- 先 `git fetch origin --prune`，再将最新候选分支合入个人分支；
- 初学阶段统一使用普通 merge，不对已 Push 分支强制 rebase；
- 不确定冲突归属时停止修改并在 PR 中请求协助；
- 合并冲突解决后重新执行本人任务的全部验证命令。
