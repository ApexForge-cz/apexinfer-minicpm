# S0-C01 仓库与远程合同

## 1. 合同范围

本文固定团队 Fork、官方上游、赛事主分支、阶段分支和个人任务分支的用途。适用于陈梓弘、周邦翔、朱健辉在本项目中的全部提交。

## 2. 仓库与远程

| 名称 | 地址 | 权限 | 用途 |
|---|---|---|---|
| `origin` | `https://github.com/ApexForge-cz/apexinfer-minicpm.git` | 团队成员可 Push | 团队 Fork、Issue、个人分支和 PR |
| `upstream` | `https://github.com/flagos-ai/vllm-plugin-FL.git` | 本地 Push 地址固定为 `no_push` | 只读获取官方更新 |

本 Fork 当前为公开仓库，GitHub 默认分支为 `flagos-2026-s2`。比赛功能不得在 GitHub `main` 上开发。

## 3. 分支层级

```text
flagos-2026-s2
└── phaseN/integration
    └── phaseN/release-candidate
        ├── task/chen-sN-<topic>
        ├── task/zhou-sN-<topic>
        └── task/zhu-sN-<topic>
```

- `flagos-2026-s2`：团队赛事主分支，只接收阶段收口 PR；
- `phaseN/integration`：阶段集成和最终交叉验证分支；
- `phaseN/release-candidate`：三名成员共同的个人任务 PR 目标分支；
- `task/...`：成员独立任务分支，只处理 Issue 分配的任务；
- `fix/...`：阶段冻结后的阻断性修复，必须单独关联 Issue。

## 4. Phase 0 当前基准

| 分支 | 起始提交 | 状态 |
|---|---|---|
| `phase0/integration` | `aacc090` | 已创建 |
| `phase0/release-candidate` | `aacc090` | 已创建 |
| `task/chen-s0-governance` | `aacc090` | 已创建并提交治理文档 |
| `task/zhou-s0-environment` | `aacc090` | 已创建，PR #9 |
| `task/zhu-s0-templates` | `aacc090` | 已创建，等待朱健辉个人交付 |

## 5. 获取全部远程分支

部分初始克隆只抓取赛事主分支。每名成员首次配置时执行：

```bash
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
git fetch origin --prune
git branch -r
```

此操作只更新远程引用，不覆盖工作区文件。

## 6. 同阶段并行合同

- 阶段负责人先冻结公共输入快照和 `phaseN/release-candidate`；
- 三名成员从同一提交同时创建个人分支；
- 个人任务不得依赖本阶段其他成员未合并的分支、文件或结论；
- 正式数据组合、交叉验证和阶段收口只在小组集成任务执行；
- 硬件被单人占用时，其他成员继续完成脚本、测试矩阵、图表生成器或文档，不得停工等待。

## 7. 禁止事项

- 禁止直接 Push `flagos-2026-s2`、`phaseN/integration` 或 `phaseN/release-candidate`；
- 禁止向 `upstream` Push；
- 禁止共享分支 force-push、变基改写历史或删除他人提交；
- 禁止将模型、数据集、密钥、个人材料和大型原始日志提交仓库；
- 禁止跨阶段绕过输入门禁提交正式结果。

## 8. 自检命令

```bash
git remote -v
git config --get-all remote.origin.fetch
git status --short --branch
git branch -a
git ls-remote --heads origin
```
