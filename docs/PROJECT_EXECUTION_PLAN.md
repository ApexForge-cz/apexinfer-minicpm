# ApexInfer MiniCPM5-2B 推理优化项目执行总方案

> 文档用途：团队协作、Issue 拆解、分支管理、阶段验收和最终交付的唯一执行基线  
> 适用仓库：`ApexForge-cz/apexinfer-minicpm`  
> 赛事代码基线：`flagos-2026-s2`  
> 项目负责人：陈梓弘  
> 文档状态：申请算力阶段，可执行初稿；性能结论必须由官方硬件实测后更新

## 1. 项目目标

在赛事规定的 FlagOS `vllm-plugin-FL`、FlagGems `v5.3.5`、MiniCPM5-2B 和固定服务参数下，对天数 BI-V150 与沐曦曦云 C500（64 GB）进行端到端推理优化。

主指标是 `total tokens/s`，同时必须满足以下硬门槛：

- MATH-500 Level 3 精度不低于 `0.95`；
- 4k 与 16k 两个场景均完成评测；
- 1% 以内差异只视为正常波动，不宣称有效提升；
- TTFT 不得超过官方允许范围；当前按“不高于基线 +1%”进行内部预检，最终以官方规则为准；
- 不修改 Benchmark 脚本的测试逻辑；
- 不通过超参数开启量化或投机采样；
- 不识别测试脚本、场景长度或请求数量进行硬编码特判；
- 不以无实质优化的算子切换、删除主要算子选择逻辑或直接合并最新官方代码获取成绩；
- 技术报告中的每项优化必须在源码、提交记录和实验结果中可追溯；
- 最终版本必须能在组委会统一环境中复现。

## 2. 固定评测合同

### 2.1 性能场景

| 场景 | Input length | Output length | Concurrency | Num prompts |
|---|---:|---:|---:|---:|
| 4k | 4096 | 1024 | 64 | 256 |
| 16k | 16384 | 1024 | 64 | 128 |

### 2.2 当前官方材料基线

| 平台 | 场景 | Benchmark duration (s) | Output tok/s | Total tok/s | Mean TTFT (ms) |
|---|---|---:|---:|---:|---:|
| 天数 BI-V150 | 4k | 646.310 | 405.60 | 2028.010 | 11573.530 |
| 天数 BI-V150 | 16k | 2434.820 | 53.83 | 915.150 | 599262.030 |
| 沐曦 C500 | 4k | 257.525 | 1017.93 | 5089.645 | 3199.435 |
| 沐曦 C500 | 16k | 316.975 | 413.51 | 7029.675 | 27197.135 |

上述数值是申请阶段参照，不替代本团队在官方环境中的基线复现。正式对比必须使用同一机器、镜像、代码基线、服务参数和评测脚本。

### 2.3 固定命令边界

服务参数和赛事 Benchmark 命令保持官方口径。允许新增外围的环境登记、日志采集、结果校验和统计脚本，但不得改变请求数据、长度、并发、请求数和指标计算方式。

## 3. 分支与协作规则

### 3.1 主分支定义

本 Fork 虽然存在 GitHub `main` 分支，但比赛指定代码来自 `flagos-2026-s2`。因此：

- `flagos-2026-s2`：本项目的默认主分支，也是用户所说的“提交到 main”的实际落点；
- `upstream/flagos-2026-s2`：官方只读基线；
- 每阶段由负责人从最新 `origin/flagos-2026-s2` 创建 `phaseN/integration`，再创建 `phaseN/release-candidate`；
- 个人任务分支必须从对应 `phaseN/release-candidate` 创建；
- 禁止在 GitHub `main` 上开发比赛功能，避免脱离赛事指定版本；
- 禁止 force push、重写共享分支历史或将个人材料提交到公开仓库。

### 3.2 分支命名

| 类型 | 格式 | 示例 |
|---|---|---|
| 阶段集成 | `phaseN/integration` | `phase4/integration` |
| 阶段候选 | `phaseN/release-candidate` | `phase4/release-candidate` |
| 个人任务 | `task/<成员>-sN-<topic>` | `task/chen-s4-scheduler-kv` |
| 阻断修复 | `fix/<成员>-sN-<topic>` | `fix/chen-s5-metax-attention` |

### 3.3 标准工作流

组员首次加入、每日拉取、任务分支、提交、Push、PR、冲突处理和收工步骤，统一参见 [TEAM_WORKFLOW.md](./TEAM_WORKFLOW.md)。以下为最简流程摘要。

```bash
git fetch origin --prune
git switch <phaseN/release-candidate>
git pull --ff-only origin <phaseN/release-candidate>
git switch -c <Issue 指定的 task/... 个人分支>

# 完成一个可验证的小任务后
git add <明确文件>
git commit -m "<type>: <清晰说明>"
git push -u origin <任务分支>
```

随后创建个人 Pull Request 到该阶段的 `phaseN/release-candidate`。阶段负责人再按 Issue 规定完成候选分支、集成分支和 `flagos-2026-s2` 的逐级收口。每个 PR 必须：

1. 关联一个 Issue；
2. 只处理一个清晰问题；
3. 写明改动前证据、改动机制、风险和回退方式；
4. 附上执行过的测试和结果文件路径；
5. 至少由一名非作者成员检查；
6. 通过该阶段验收条件后再合并。

### 3.4 禁止提交内容

- MiniCPM5-2B 模型权重；
- Evalscope 数据集；
- API Key、访问令牌、`.env`、平台账号或算力券信息；
- 含成员敏感信息的个人简历和算力申请 PDF；
- 无筛选的大型 profiler trace、完整服务日志和重复 Benchmark 输出；
- 无法说明来源、配置和用途的二进制文件。

## 4. 团队职责与负载检查

### 4.1 固定角色

| 成员 | 主角色 | 主要责任 | 不承担的职责 |
|---|---|---|---|
| 陈梓弘 | 项目负责人、核心优化 | 赛题边界、技术路线、代码评审、运行时/调度/KV Cache/算子候选优化、双平台整合、报告主笔 | 不单独批准自己的高风险改动 |
| 周邦翔 | 数据与平台工程 | 环境登记、Benchmark 自动化、指标采集、实验登记、回归测试、稳定性、复现脚本、协助性能剖析 | 未经 profiler 证据不独立决定内核重写 |
| 朱健辉 | 可视化与工具开发 | 结果整理、对比图、辅助脚本、基础接口联调、测试执行、文档与展示材料 | 不承担全栈架构、调度器或底层内核开发 |

### 4.2 RACI 矩阵

说明：`A` 为最终负责，`R` 为直接执行，`C` 为参与评审，`I` 为同步知情。

| 工作项 | 陈梓弘 | 周邦翔 | 朱健辉 |
|---|---|---|---|
| 赛题合规与范围控制 | A/R | C | I |
| 环境固化与基线 | A/C | R | C |
| Benchmark 与实验登记 | A/C | R | C |
| profiler 与瓶颈清单 | A/R | R | I |
| 运行时/调度/KV Cache 优化 | A/R | C/R | I |
| 算子与平台适配 | A/R | C | I |
| 回归、稳定性、复现 | A/C | R | R |
| 图表与结果表达 | A/C | C | R |
| 技术报告与 README | A/R | R | R |

### 4.3 分配合理性结论

- 陈梓弘贯穿所有阶段，满足项目负责人需要掌握完整证据链的要求；
- 周邦翔的任务集中在可测量、可复现和平台工程，和现有 Python 后端、数据管线及测试能力匹配；
- 朱健辉的任务限定在已证明具备的 Flask/EChartsGL、数据展示和辅助开发能力范围内；
- 调度、KV Cache 和内核优化由陈梓弘主责，周邦翔提供测量与回归支持，避免把高风险核心模块交给经验不匹配的成员；
- S3、S4、S5 不同时全面展开，只允许 profiler 支持的候选进入开发，控制项目负责人负载；
- 每阶段都有至少一名主责和一名复核者，不存在单人开发、单人验收同一高风险改动的情况。

## 5. 全局完成标准

任何阶段或 Issue 只有同时满足以下条件才能关闭：

- 任务范围内的代码、脚本或文档已经提交；
- 输出文件有固定路径和命名规则；
- 验收命令能够被另一名成员执行；
- 结果包含成功状态和失败原因，不能只保留“最佳一次”；
- 涉及性能时至少完成 3 次有效重复并保存原始结果；
- 涉及模型行为时通过正确性或精度门禁；
- PR 描述包含证据、风险和回退方式；
- README、实验登记表和代码状态保持一致；
- 阶段产物能够直接成为下一阶段输入，形成闭环。

## 6. 阶段依赖与里程碑

```text
S0 项目治理与环境准备
 └─ S1 双平台基线与精度复现
     └─ S2 分层剖析与瓶颈地图
         ├─ S3 低风险运行时优化
         ├─ S4 调度 / KV Cache / 图执行优化
         └─ S5 热点算子与平台适配
             └─ S6 集成、消融、回归与稳定性
                 └─ S7 最终报告、复现与提交包
```

阶段推进原则：

- S0、S1、S2 必须顺序完成；
- S3、S4、S5 只在 S2 给出证据后选择性启动，不要求全部实施；
- S6 只组合已经通过单项门禁的改动；
- S7 不再引入新的性能机制，只做修复、复现和交付。

## 7. Issue 与阶段执行卡

### S0：项目治理、仓库安全与开发环境合同

- **Issue**：`[S0] 建立项目治理、仓库安全与环境合同`
- **GitHub Issue**：[#1](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/1)
- **阶段信息**：项目启动与协作规则冻结阶段；主责：陈梓弘；环境复核：周邦翔；文档可用性复核：朱健辉。
- **输入门禁**：团队 Fork 已建立，三名成员 GitHub 账号可用，赛事指定分支和版本信息已确认。
- **输出去向**：S1-S7 共用的仓库规则、环境合同、证据模板和安全边界。

#### 小组目标

让三名成员能在同一赛事基线和同一证据口径下独立开工、提交 PR 和交叉验收。规则必须写进仓库，不能依赖聊天记录或口头约定。

#### 分支与合并要求

- 阶段集成分支：`phase0/integration`；小组候选分支：`phase0/release-candidate`；
- 陈梓弘：`task/chen-s0-governance`；周邦翔：`task/zhou-s0-environment`；朱健辉：`task/zhu-s0-templates`；
- 个人分支从 `phase0/release-candidate` 创建，个人 PR 合并回该分支；
- 陈梓弘汇总后提交 `phase0/release-candidate` → `phase0/integration`，验收后提交 `phase0/integration` → `flagos-2026-s2`；
- 禁止直接提交共享分支，禁止 force-push 或变基改写共享历史；
- PR 必须包含任务编号、交付物、检查命令、实际结果、遗留问题和评审人。

#### 陈梓弘个人任务：项目治理与仓库规则

- **任务编号**：`S0-C01` 建立远程与分支合同；`S0-C02` 固化 PR/Review/合并规则；`S0-C03` 配置仓库安全边界。
- **个人分支**：`task/chen-s0-governance`。
- **交付物**：`docs/evidence/phase-0/chen-governance/repository-contract.md`、`branch-and-review-rules.md`、`.gitignore-audit.md`。
- **允许修改**：协作文档、PR 模板、`.gitignore` 和不影响框架行为的仓库配置。
- **禁止修改**：模型行为、官方 Benchmark、赛事分支历史和上游远程地址；不得提交账号、令牌或个人材料。
- **验收**：远程、默认分支、命名、Review 和回退规则均有可执行命令，另两名成员完成权限验证。

#### 周邦翔个人任务：环境与版本合同

- **任务编号**：`S0-Z01` 建立双平台环境字段；`S0-Z02` 建立实验编号与结果字段；`S0-Z03` 登记算力申请和使用边界。
- **个人分支**：`task/zhou-s0-environment`。
- **交付物**：`docs/evidence/phase-0/zhou-environment/environment-matrix.md`、`experiment-template.md`、`compute-resource-register.md`。
- **允许修改**：环境、实验和算力登记文档以及外围采集模板。
- **禁止修改**：不得填写未实测版本或未获批算力；不得把真实密钥、Cookie、机器凭据写入仓库。
- **验收**：模板覆盖芯片、驱动、运行时、Python、PyTorch、vLLM、插件、FlagGems、提交号、命令、结果和校验值。

#### 朱健辉个人任务：协作文档与证据模板

- **任务编号**：`S0-H01` 验证新成员工作流；`S0-H02` 建立证据索引模板；`S0-H03` 检查文档路径和命令可读性。
- **个人分支**：`task/zhu-s0-templates`。
- **交付物**：`docs/evidence/phase-0/zhu-templates/onboarding-check.md`、`evidence-index-template.md`、`documentation-usability-report.md`。
- **允许修改**：Markdown 模板、示例和文档链接。
- **禁止修改**：不得修改调度器、KV Cache、算子、模型代码；不得用虚构运行结果填充模板。
- **验收**：朱健辉能只按文档完成拉取、建分支、提交、推送和创建 PR，并记录发现的问题。

#### 小组共同任务与关闭门禁

- `S0-G01`：三人各完成一次测试分支和测试 PR；交付 `docs/evidence/phase-0/group/access-verification.md`。
- `S0-G02`：登记本阶段交接；交付 `docs/evidence/phase-0/group/handoff.md`。
- 三个个人 PR 均经至少一名其他成员 Review，候选分支已汇总到阶段集成分支；
- `git status` 不显示模型、数据、密钥或大型结果，任意成员能说明代码、结果和原始日志的存放位置；
- 未达到门禁不得开始 S1 正式测量。

#### 本阶段不做

- 不进行性能优化或宣称性能收益；不上传模型、数据集和大型日志；不创建脱离 Issue 的长期个人分支。

闭环：S0 的版本、目录、分支和证据合同直接作为 S1 输入。

### S1：双平台基线、精度与测量稳定性

- **Issue**：`[S1] 复现 MiniCPM5-2B 双平台性能与精度基线`
- **GitHub Issue**：[#2](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/2)
- **阶段信息**：双平台基线冻结阶段；主责：周邦翔；运行链路复核：陈梓弘；结果展示复核：朱健辉。
- **输入门禁**：S0 已合并；至少一种官方算力可用；模型、框架、FlagGems 和数据集路径可访问。
- **输出去向**：S2 profiler 对照和 S3-S6 所有收益计算的唯一基线。

#### 小组目标

建立天数 BI-V150 与沐曦 C500 上可复现的精度、吞吐、时延和稳定性基线。正式数据必须绑定环境、命令、提交号和原始结果，不允许只保留截图或最好一次。

#### 分支与合并要求

- 阶段集成分支：`phase1/integration`；候选分支：`phase1/release-candidate`；
- 周邦翔：`task/zhou-s1-benchmark`；陈梓弘：`task/chen-s1-runtime-check`；朱健辉：`task/zhu-s1-baseline-evidence`；
- 个人 PR → `phase1/release-candidate` → `phase1/integration` → `flagos-2026-s2`；
- 禁止直接提交共享分支、修改正式 Benchmark 口径或只提交筛选后的成功结果。

#### 周邦翔个人任务：正式基线测量

- **任务编号**：`S1-Z01` 登记双平台环境；`S1-Z02` 执行服务与精度基线；`S1-Z03` 执行 4k/16k 性能基线；`S1-Z04` 统计稳定性。
- **个人分支**：`task/zhou-s1-benchmark`。
- **交付物**：`docs/evidence/phase-1/zhou-benchmark/environment-lock.md`、`accuracy-baseline.md`、`baseline-results.csv`、`stability-analysis.md`、`commands.md`。
- **允许修改**：`scripts/competition/` 的启动、采集、解析和校验脚本，以及个人证据目录。
- **禁止修改**：官方输入/输出长度、并发、请求数、采样参数、模型行为和 Benchmark 指标算法；不得删除失败请求。
- **验收**：两平台 × 4k/16k 均至少 3 次有效重复；`accuracy >= 0.95`；结果含中位数、极差、变异系数、TTFT 和失败数。

#### 陈梓弘个人任务：运行链路与版本复核

- **任务编号**：`S1-C01` 核对加载与平台路径；`S1-C02` 核对服务参数和图模式；`S1-C03` 诊断超过 1% 的基线偏差。
- **个人分支**：`task/chen-s1-runtime-check`。
- **交付物**：`docs/evidence/phase-1/chen-runtime/runtime-path-audit.md`、`serve-parameter-check.md`、`baseline-deviation-diagnosis.md`。
- **允许修改**：诊断脚本、日志开关和文档；必要修复必须单独提交并说明不改变计算语义。
- **禁止修改**：不得在基线阶段加入性能优化，不得改变官方启动参数或隐藏平台回退。
- **验收**：两平台实际走到的插件、vendor、图执行和 FlagGems 路径可由日志证明，偏差均有结论或阻塞登记。

#### 朱健辉个人任务：基线图表与证据索引

- **任务编号**：`S1-H01` 校验结果字段；`S1-H02` 生成基线对比图；`S1-H03` 建立实验与原始日志索引。
- **个人分支**：`task/zhu-s1-baseline-evidence`。
- **交付物**：`docs/evidence/phase-1/zhu-evidence/figure-source.csv`、`baseline-comparison.png`、`baseline-evidence-index.md`。
- **允许修改**：结果可视化脚本、Markdown 和个人证据目录。
- **禁止修改**：不得手工改数、挑选最好一次、修改核心框架/调度/KV Cache/算子代码或把未测平台写成完成。
- **验收**：图可从锁定 CSV 重生成，数值与周邦翔的结果逐项一致，每个图表数据能回到实验编号和原始路径。

#### 小组共同任务与关闭门禁

- `S1-G01`：另一成员按同一命令复跑一个平台场景；交付 `docs/evidence/phase-1/group/cross-reproduction.md`。
- `S1-G02`：形成基线锁定和 S2 交接；交付 `baseline-lock.md`、`handoff.md`。
- 四个平台-场景组合结果齐全；TTFT 超过官方基线 +1% 或吞吐低于基线 1% 以上时不得标记完成，除非形成明确阻塞说明；
- 三个个人 PR 已评审并汇总，正式结论不依赖未合并分支。

#### 本阶段不做

- 不做性能机制开发，不量化、不投机采样，不调整官方测试场景，不用 profiler 数据冒充正式成绩。

闭环：S1 的锁定 CSV、环境和命令作为 S2-S6 唯一对照分母。

### S2：分层性能剖析与瓶颈地图

- **Issue**：`[S2] 建立 Prefill/Decode 分层瓶颈地图与优化候选清单`
- **GitHub Issue**：[#3](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/3)
- **阶段信息**：性能剖析与候选决策阶段；主责：陈梓弘；采集质量负责人：周邦翔；可视化与证据负责人：朱健辉。
- **输入门禁**：S1 至少一个平台的正式基线已锁定，采集环境与基线提交一致。
- **输出去向**：S3 运行时、S4 调度/KV/图执行、S5 算子优化的启动清单。

#### 小组目标

用 profiler、框架日志和代码路径回答时间、显存和同步开销的位置，并给出可验证、可拒绝、可排序的候选，禁止凭经验直接开发。

#### 分支与合并要求

- 阶段集成分支：`phase2/integration`；候选分支：`phase2/release-candidate`；
- 陈梓弘：`task/chen-s2-bottleneck-map`；周邦翔：`task/zhou-s2-profile-capture`；朱健辉：`task/zhu-s2-profile-evidence`；
- 个人 PR → 候选分支 → 阶段集成分支 → `flagos-2026-s2`；禁止直接向共享分支提交。

#### 陈梓弘个人任务：框架瓶颈定位与候选决策

- **任务编号**：`S2-C01` 分解 Prefill/Decode 路径；`S2-C02` 审计运行时、调度、KV 和图执行；`S2-C03` 建立热点算子与候选优先级。
- **个人分支**：`task/chen-s2-bottleneck-map`。
- **交付物**：`docs/evidence/phase-2/chen-analysis/bottleneck-map.md`、`code-path-audit.md`、`optimization-candidates.md`。
- **允许修改**：诊断埋点、可关闭日志、分析脚本和个人证据；诊断代码不得进入最终热路径。
- **禁止修改**：不得把候选直接实现成正式优化，不得修改 Benchmark 或根据固定场景硬编码。
- **验收**：每个候选都有 trace/日志、代码路径、预期收益、风险、平台范围、验证成本和进入/拒绝结论。

#### 周邦翔个人任务：Profiler 采集与测量控制

- **任务编号**：`S2-Z01` 设计采集矩阵；`S2-Z02` 采集双场景 trace 与显存；`S2-Z03` 校验采集开销和重复性。
- **个人分支**：`task/zhou-s2-profile-capture`。
- **交付物**：`docs/evidence/phase-2/zhou-capture/profile-plan.md`、`profile-index.csv`、`measurement-quality.md`、`commands.md`。
- **允许修改**：外围采集、解析和校验脚本；个人证据目录。
- **禁止修改**：不得将 profiler 运行当成正式吞吐结果，不得只采对结论有利的区间或删除异常 trace。
- **验收**：4k/16k 至少覆盖 Prefill、Decode、峰值显存、调度空隙和前十热点，原始文件有路径和 SHA256。

#### 朱健辉个人任务：瓶颈图与证据索引

- **任务编号**：`S2-H01` 整理结构化 profiler 数据；`S2-H02` 生成分层瓶颈图；`S2-H03` 建立候选证据索引。
- **个人分支**：`task/zhu-s2-profile-evidence`。
- **交付物**：`docs/evidence/phase-2/zhu-evidence/profile-source.csv`、`prefill-decode-breakdown.png`、`top-operators.png`、`candidate-evidence-index.md`。
- **允许修改**：解析辅助脚本、可视化脚本、图表和证据索引。
- **禁止修改**：不得修改核心执行路径、调度、KV Cache 或算子；不得用图形美化改变数值或夸大候选收益。
- **验收**：所有图可重生成，单位、平台、场景和采集条件完整，候选能反查原始 trace 和陈梓弘的代码结论。

#### 小组共同任务与关闭门禁

- `S2-G01`：三人评审候选并登记进入 S3/S4/S5 或拒绝原因；交付 `candidate-review.md`。
- `S2-G02`：形成阶段交接；交付 `docs/evidence/phase-2/group/handoff.md`。
- 至少给出一个框架级和一个算子级候选，或用证据说明不存在；三个个人 PR 完成 Review 并汇总；
- 未进入候选清单的优化不得占用官方算力开发。

#### 本阶段不做

- 不宣称正式性能提升，不把相关性当因果，不实施无法独立验证或回退的“大改动”。

闭环：候选决策表是 S3-S5 的启动门。

### S3：低风险运行时与执行路径优化

- **Issue**：`[S3] 优化运行时热路径与可复用缓冲区`
- **GitHub Issue**：[#4](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/4)
- **阶段信息**：低风险框架热路径优化阶段；开发主责：陈梓弘；独立测量：周邦翔；测试证据：朱健辉。
- **输入门禁**：S2 明确指出运行时/执行路径开销及对应 trace、代码位置和候选优先级。
- **输出去向**：通过单项门禁的提交进入 S6 候选组合。

#### 小组目标

只实现 S2 有证据支持、数值语义不变且可独立回退的运行时优化。每个机制单独提交、单独测试、单独 A/B，负结果同样留档。

#### 分支与合并要求

- 阶段集成分支：`phase3/integration`；候选分支：`phase3/release-candidate`；
- 陈梓弘：`task/chen-s3-runtime`；周邦翔：`task/zhou-s3-validation`；朱健辉：`task/zhu-s3-test-evidence`；
- 个人 PR → 候选分支 → 阶段集成分支 → `flagos-2026-s2`；核心代码 PR 必须由周邦翔复核性能、朱健辉复核证据完整性。

#### 陈梓弘个人任务：运行时优化实现

- **任务编号**：`S3-C01` 锁定候选与回退点；`S3-C02` 实现元数据/对象/同步优化；`S3-C03` 实现缓冲复用或冗余转换消除；`S3-C04` 补充正确性测试。
- **个人分支**：`task/chen-s3-runtime`。
- **交付物**：代码与测试；`docs/evidence/phase-3/chen-runtime/change-design.md`、`correctness-report.md`、`rollback-plan.md`。
- **允许修改**：S2 指定的 `vllm_fl/worker/`、`vllm_fl/platform.py` 等运行时模块及对应测试。
- **禁止修改**：不得扩大到调度/KV/算子机制，不得改 Benchmark、模型行为或按固定场景写特判。
- **验收**：每项改动独立提交、可关闭/回退；单测通过；代码说明与 S2 候选一一对应。

#### 周邦翔个人任务：独立 A/B 与回归

- **任务编号**：`S3-Z01` 建立单项 A/B 矩阵；`S3-Z02` 执行目标与非目标场景；`S3-Z03` 核对精度、TTFT、显存和稳定性。
- **个人分支**：`task/zhou-s3-validation`。
- **交付物**：`docs/evidence/phase-3/zhou-validation/ab-matrix.md`、`performance-results.csv`、`regression-report.md`、`commands.md`。
- **允许修改**：外围测量/解析脚本和个人证据目录。
- **禁止修改**：不得修改陈梓弘的核心实现来追分，不得过滤负结果或改变基线与候选的运行口径。
- **验收**：至少一个目标场景超过 1% 或有明确资源证据；另一场景无超过 1% 的无解释退化；`accuracy >= 0.95`。

#### 朱健辉个人任务：测试矩阵与证据整理

- **任务编号**：`S3-H01` 建立边界测试清单；`S3-H02` 运行文档/测试复核；`S3-H03` 建立提交-测试-结果索引。
- **个人分支**：`task/zhu-s3-test-evidence`。
- **交付物**：`docs/evidence/phase-3/zhu-evidence/test-matrix.md`、`test-review.md`、`evidence-index.md`。
- **允许修改**：非核心测试辅助、Markdown 和证据索引。
- **禁止修改**：不得修改调度器、KV Cache、算子或核心运行时实现，不得自行判定性能通过。
- **验收**：每个优化提交均能链接到设计、测试、A/B、回退方法和评审结论。

#### 小组共同任务与关闭门禁

- `S3-G01`：逐项作出保留/回退/继续诊断决定；交付 `candidate-decision.md`。
- `S3-G02`：向 S6 交接通过门禁的提交；交付 `docs/evidence/phase-3/group/handoff.md`。
- 三个个人 PR 完成 Review；精度、服务参数、请求成功率和稳定性通过；负结果未被删除。

#### 本阶段不做

- 不实施 S2 未批准机制，不混入调度/KV/内核重写，不一次合并多个无法消融的机制。

闭环：通过门禁的提交进入 S6，失败候选回退并保留记录。

### S4：调度、KV Cache 与图执行协同优化

- **Issue**：`[S4] 优化调度、KV Cache 与图执行协同路径`
- **GitHub Issue**：[#5](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/5)
- **阶段信息**：调度、KV Cache 与图执行优化阶段；开发主责：陈梓弘；压力与性能验收：周邦翔；测试证据：朱健辉。
- **输入门禁**：S2 已证明存在调度空隙、KV 开销、显存碎片或图捕获/回退问题。
- **输出去向**：可独立启用的 S6 调度/KV/图执行候选。

#### 小组目标

提高设备忙碌时间和有效并发，同时守住 TTFT、峰值显存、正确性与非目标长度行为。策略只读取真实运行时状态，禁止识别固定 Benchmark 场景。

#### 分支与合并要求

- 阶段集成分支：`phase4/integration`；候选分支：`phase4/release-candidate`；
- 陈梓弘：`task/chen-s4-scheduler-kv`；周邦翔：`task/zhou-s4-stress-validation`；朱健辉：`task/zhu-s4-test-evidence`；
- 个人 PR → 候选分支 → 阶段集成分支 → `flagos-2026-s2`；核心实现须通过另外两人交叉验收。

#### 陈梓弘个人任务：调度/KV/图执行实现

- **任务编号**：`S4-C01` 锁定调度与 KV 瓶颈；`S4-C02` 实现通用运行时策略；`S4-C03` 处理双平台图执行与回退；`S4-C04` 增加边界测试和回退开关。
- **个人分支**：`task/chen-s4-scheduler-kv`。
- **交付物**：代码与测试；`docs/evidence/phase-4/chen-implementation/design.md`、`platform-behavior.md`、`rollback-plan.md`。
- **允许修改**：经 S2 批准的 `model_runner.py`、`scheduler_fl.py`、`compilation/graph.py`、`platform.py` 和相应测试。
- **禁止修改**：不得读取固定 input/output/concurrency 常量做特判，不得改模型、采样、量化或 Benchmark 口径。
- **验收**：策略适用于非目标长度；天数与沐曦路径和回退清晰；单项提交可独立撤销。

#### 周邦翔个人任务：压力、性能与稳定性验收

- **任务编号**：`S4-Z01` 建立目标/非目标矩阵；`S4-Z02` 执行吞吐与 TTFT A/B；`S4-Z03` 执行 KV/显存/空队列/批次变化压力测试。
- **个人分支**：`task/zhou-s4-stress-validation`。
- **交付物**：`docs/evidence/phase-4/zhou-validation/performance-results.csv`、`memory-stress-report.md`、`graph-fallback-report.md`、`commands.md`。
- **允许修改**：测试和采集脚本、个人证据目录。
- **禁止修改**：不得调整核心算法、删除 OOM/回退/失败请求，或使用与基线不同的服务参数。
- **验收**：4k 或 16k 中位吞吐提升超过 1%；另一目标场景无未解释退化；TTFT 合规且无新增 OOM。

#### 朱健辉个人任务：边界测试与证据索引

- **任务编号**：`S4-H01` 整理长度/批次/空队列测试清单；`S4-H02` 核查平台回退日志；`S4-H03` 建立代码-测试-结果证据链。
- **个人分支**：`task/zhu-s4-test-evidence`。
- **交付物**：`docs/evidence/phase-4/zhu-evidence/boundary-test-matrix.md`、`fallback-log-index.md`、`evidence-index.md`。
- **允许修改**：测试辅助、日志解析、Markdown 证据。
- **禁止修改**：不得开发或修改调度器、KV Cache、图编译与算子核心代码；不得将未测试状态标成通过。
- **验收**：每个边界用例有输入、预期、实际、提交号和日志；能识别硬编码或平台错误回退。

#### 小组共同任务与关闭门禁

- `S4-G01`：完成候选保留/回退评审；交付 `candidate-decision.md`。
- `S4-G02`：向 S6 交接候选、开关、限制和提交；交付 `docs/evidence/phase-4/group/handoff.md`。
- 三个个人 PR 已 Review；精度、服务稳定性、非目标长度和图回退检查全部通过。

#### 本阶段不做

- 不重写与瓶颈无关模块，不做 Benchmark 特判，不用增大显存风险换取无法稳定复现的单次吞吐。

闭环：通过门禁的候选进入 S6，失败候选回退并登记。

### S5：热点算子、FlagGems 与双平台适配

- **Issue**：`[S5] 基于 profiler 优化热点算子并完成双平台适配`
- **GitHub Issue**：[#6](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/6)
- **阶段信息**：热点算子与平台适配阶段；实现主责：陈梓弘；数值/端到端验收：周邦翔；结果图与证据：朱健辉。
- **输入门禁**：S2 给出热点算子、真实调用 shape、累计耗时、平台路径和预期验证方法。
- **输出去向**：通过数值和端到端门禁的插件/FlagGems 对应提交进入 S6。

#### 小组目标

只优化真实热点，通过融合、访存、并行划分或编译缓存取得可复现收益，并为不支持的平台保留正确回退。禁止没有实质优化的算子切换。

#### 分支与合并要求

- 阶段集成分支：`phase5/integration`；候选分支：`phase5/release-candidate`；
- 陈梓弘：`task/chen-s5-operators`；周邦翔：`task/zhou-s5-op-validation`；朱健辉：`task/zhu-s5-op-evidence`；
- 个人 PR → 候选分支 → 阶段集成分支 → `flagos-2026-s2`；涉及 FlagGems 时必须记录两个仓库提交对应关系。

#### 陈梓弘个人任务：热点算子实现与双平台回退

- **任务编号**：`S5-C01` 锁定热点与参考实现；`S5-C02` 设计并实现融合/内核/编译优化；`S5-C03` 隔离 txda/metax 差异；`S5-C04` 增加参考、边界 shape 与回退测试。
- **个人分支**：`task/chen-s5-operators`。
- **交付物**：代码与测试；`docs/evidence/phase-5/chen-operators/operator-design.md`、`platform-dispatch.md`、`commit-mapping.md`、`rollback-plan.md`。
- **允许修改**：S2 指定的 vendor、FlagGems dispatch、`vllm_fl/ops/` 及独立 FlagGems Fork 对应算子。
- **禁止修改**：不得删主要选择逻辑、无优化切换算子、修改模型语义、量化/投机采样或按 Benchmark shape 硬编码结果。
- **验收**：机制、适用 shape、容差、平台差异和代价明确；不支持平台有显式正确回退。

#### 周邦翔个人任务：数值与端到端独立验收

- **任务编号**：`S5-Z01` 建立参考实现和 shape 矩阵；`S5-Z02` 执行数值/边界/回退测试；`S5-Z03` 执行热点与端到端 A/B。
- **个人分支**：`task/zhou-s5-op-validation`。
- **交付物**：`docs/evidence/phase-5/zhou-validation/numerical-results.csv`、`shape-and-fallback-report.md`、`performance-results.csv`、`commands.md`。
- **允许修改**：测试、基准采集和结果解析脚本。
- **禁止修改**：不得改算子实现以适配测试，不得放宽未说明的容差或忽略另一平台失败。
- **验收**：数值在明确容差内；目标场景中位数提升超过 1%，或热点显著下降且端到端无退化；`accuracy >= 0.95`。

#### 朱健辉个人任务：算子结果图与证据索引

- **任务编号**：`S5-H01` 核查结果输入；`S5-H02` 生成热点/端到端对比图；`S5-H03` 建立插件-FlagGems-测试证据索引。
- **个人分支**：`task/zhu-s5-op-evidence`。
- **交付物**：`docs/evidence/phase-5/zhu-evidence/figure-source.csv`、`operator-performance.png`、`end-to-end-comparison.png`、`evidence-index.md`。
- **允许修改**：可视化与证据辅助脚本、图表、Markdown。
- **禁止修改**：不得参与或修改底层算子实现、调度器和 KV Cache；不得手工改变实验数值或掩盖负结果。
- **验收**：图表可重生成且与锁定 CSV 一致，能追溯到两个仓库提交、测试命令和原始结果。

#### 小组共同任务与关闭门禁

- `S5-G01`：完成候选保留/回退评审；交付 `candidate-decision.md`。
- `S5-G02`：向 S6 交接实现、依赖、回退和限制；交付 `docs/evidence/phase-5/group/handoff.md`。
- 数值、边界、回退、端到端精度和性能均完成；三个个人 PR 经交叉 Review 并汇总。

#### 本阶段不做

- 不优化非热点，不以算子切换冒充优化，不牺牲正确性或另一平台，不合入无法说明机制与回退的代码。

闭环：通过门禁的算子及对应版本进入 S6；其他实验只保留为诊断记录。

### S6：组合优化、消融、回归与稳定性

- **Issue**：`[S6] 完成双平台组合优化、消融与回归验收`
- **GitHub Issue**：[#7](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/7)
- **阶段信息**：最终组合与发布前验收阶段；截止时间为进入 S7 前；本阶段不再新增未经 S2-S5 证据支持的优化机制。
- **成员**：陈梓弘、周邦翔、朱健辉；主责：周邦翔；最终范围与提交负责人：陈梓弘；结果证据负责人：朱健辉。
- **输入门禁**：S3-S5 至少一项通过单项门禁；每项候选已有提交号、测试记录、精度结果、性能结果和回退方式。
- **输出去向**：最终候选提交、消融结果、双平台回归报告、结果图表、S7 技术报告与复现说明。

#### 小组目标

从已通过单项门禁的候选中形成唯一最终组合，证明收益来自可解释改动，并在天数 BI-V150、沐曦 C500 的 4k/16k 场景中稳定复现。所有性能、精度、TTFT、显存和失败请求结论必须来自锁定提交和可追溯原始结果；不能用口头说明、截图或“计划测试”代替验收证据。

#### 分支与合并要求

- 阶段集成分支：`phase6/integration`，由陈梓弘从最新 `origin/flagos-2026-s2` 创建并推送；
- 小组候选分支：`phase6/release-candidate`，由陈梓弘从 `phase6/integration` 创建并推送；
- 周邦翔个人分支：`task/zhou-s6-validation`；
- 陈梓弘个人分支：`task/chen-s6-integration`；
- 朱健辉个人分支：`task/zhu-s6-evidence`；
- 所有个人分支均从最新 `phase6/release-candidate` 创建；
- 个人任务 PR 合并目标统一为 `phase6/release-candidate`；
- 个人任务完成并交叉评审后，由陈梓弘从 `phase6/release-candidate` 向 `phase6/integration` 提交小组汇总 PR；
- S6 全部验收后，由陈梓弘从 `phase6/integration` 向 `flagos-2026-s2` 提交唯一阶段收口 PR；
- 禁止直接向 `flagos-2026-s2`、`phase6/integration` 或 `phase6/release-candidate` 提交；
- 禁止共享分支 force-push、变基改写历史或把未验收提交标成最终版本；
- 每个 PR 必须写明任务编号、交付物路径、验证命令、实际结果、遗留问题和评审人。

#### 个人任务一：周邦翔——最终 Benchmark 与质量回归

- **任务分支**：`task/zhou-s6-validation`
- **个人职责**：数据与平台工程、正式测量、回归和结果登记；不负责决定是否新增核心优化机制。

**个人任务**

- **S6-Z01：建立最终测量矩阵。** 从 `phase6/release-candidate` 检出候选，按 Baseline、单项 A、单项 B、组合 A+B 建立矩阵；每次记录平台、提交号、FlagGems 版本、命令、运行时间和原始结果路径。
- **S6-Z02：执行四个平台-场景组合。** 在天数 BI-V150 和沐曦 C500 上分别执行 4k `[4096,1024,64,256]` 与 16k `[16384,1024,64,128]`，每个组合至少 3 次有效重复，计算中位数、极差、变异系数。
- **S6-Z03：执行正确性与服务回归。** 运行 API 冒烟、MATH-500 Level 3、请求完成率、超时、异常日志、OOM 和连续多轮稳定性检查。
- **S6-Z04：锁定性能结论。** 只允许使用锁定 CSV/JSON 计算 `total tokens/s`、Output tok/s、duration、Mean/P50/P99 TTFT、TPOT、ITL、峰值显存和失败请求；不得只提交最好一次结果。

**个人交付物**

- `docs/evidence/phase-6/zhou-validation/measurement-matrix.md`；
- `docs/evidence/phase-6/zhou-validation/performance-results.csv`；
- `docs/evidence/phase-6/zhou-validation/final-regression-report.md`；
- `docs/evidence/phase-6/zhou-validation/commands-and-environment.md`；
- 原始日志、CSV 和评测目录的外部路径与 SHA256 校验值。

**允许修改**

- `scripts/competition/` 下的环境登记、结果解析和校验脚本；
- `docs/evidence/phase-6/zhou-validation/` 下的记录；
- 与测试结果格式相关的非核心测试工具。

**禁止修改**

- 禁止修改官方 Benchmark 的输入长度、输出长度、并发、请求数和指标算法；
- 禁止为“跑出更高分”修改核心调度、算子或模型行为；
- 禁止删除失败请求、OOM、超时或异常结果；
- 禁止把未完成的硬件测试写成已通过。

**验收标准**

- 四个平台-场景组合均有至少 3 次有效结果；
- `accuracy >= 0.95`；
- 最终宣称的提升超过 1% 正常波动；
- TTFT 在官方允许范围内，无失败请求和 OOM；
- 另一名成员能按记录中的命令复核 CSV 与结论。

#### 个人任务二：陈梓弘——候选集成与消融收口

- **任务分支**：`task/chen-s6-integration`
- **个人职责**：项目负责人、候选组合与范围收口；负责决定哪些已验收候选进入最终版本，不单独批准自己的高风险改动。

**个人任务**

- **S6-C01：创建阶段分支。** 从最新 `flagos-2026-s2` 创建 `phase6/integration`，再创建 `phase6/release-candidate`，在本 Issue 记录远程分支链接和起始提交。
- **S6-C02：逐项合并候选。** 只合并 S3-S5 已通过单项门禁的提交；每次合并记录候选名称、提交号、依赖、回退方式和预期影响。
- **S6-C03：组织消融矩阵。** 至少形成 Baseline、A、B、A+B 或等价组合，明确每个组合启用的改动，禁止一次合入无法拆解的“大杂烩”提交。
- **S6-C04：完成最终范围锁定。** 对每个候选标记保留、回退或诊断-only，形成最终提交号、改动清单、平台差异和已知限制。
- **S6-C05：提交小组汇总 PR。** 将 `phase6/release-candidate` 合并到 `phase6/integration`，附上个人任务 PR、周邦翔结果报告、朱健辉证据索引和遗留问题。

**个人交付物**

- `docs/evidence/phase-6/chen-integration/candidate-lock.md`；
- `docs/evidence/phase-6/chen-integration/ablation-matrix.md`；
- `docs/evidence/phase-6/chen-integration/final-commit-lock.md`；
- `docs/evidence/phase-6/chen-integration/known-limitations.md`；
- 小组汇总 PR 和阶段收口 PR 链接。

**允许修改**

- 已在 S3-S5 通过门禁的 `vllm_fl/` 核心代码；
- 候选组合的配置、回退开关和集成测试；
- `docs/evidence/phase-6/chen-integration/` 下的收口记录。

**禁止修改**

- 禁止在 S6 新增未经过 S2 profiler 和单项门禁的性能机制；
- 禁止修改模型权重、采样行为、量化/投机采样设置或官方 Benchmark 口径；
- 禁止为了让消融“好看”删除负结果；
- 禁止跳过第二成员 Review 直接向阶段集成分支合并。

**验收标准**

- 每个最终改动均可追溯到单项 PR、测试和结果；
- 消融矩阵能解释组合收益或相互抵消；
- 最终组合在两平台、两场景无未解释退化；
- 最终提交不包含未验收功能和无关临时文件；
- 阶段汇总 PR 通过周邦翔和朱健辉交叉检查。

#### 个人任务三：朱健辉——结果可视化与证据索引

- **任务分支**：`task/zhu-s6-evidence`
- **个人职责**：锁定结果整理、对比图、证据索引和展示材料；不承担调度器、KV Cache 或底层算子开发。

**个人任务**

- **S6-H01：建立结果输入清单。** 只接收周邦翔确认过的 CSV/JSON 和陈梓弘确认过的提交号，记录数据源、生成命令和文件校验值。
- **S6-H02：生成双平台对比图。** 生成 4k/16k 的基线-单项-组合 `total tokens/s`、TTFT 和显存对比图；图中保留单位、场景、平台、重复次数和统计口径。
- **S6-H03：建立证据索引。** 把每张图、每个表、每个结论链接到提交号、实验编号、原始 CSV/JSON、日志路径和验证人。
- **S6-H04：准备答辩结果页。** 只从锁定数据生成 1 页结果摘要和 1 页失败/限制说明，不把计划中能力写成已完成。

**个人交付物**

- `docs/evidence/phase-6/zhu-evidence/figure-source.csv`；
- `docs/evidence/phase-6/zhu-evidence/throughput-comparison.png`；
- `docs/evidence/phase-6/zhu-evidence/latency-and-memory-comparison.png`；
- `docs/evidence/phase-6/zhu-evidence/evidence-index.md`；
- `docs/evidence/phase-6/zhu-evidence/presentation-result-summary.md`。

**允许修改**

- `scripts/competition/visualize_results.py` 或同等结果可视化脚本；
- `docs/evidence/phase-6/zhu-evidence/` 下的图表和索引；
- 与图表展示相关的 Markdown 文档。

**禁止修改**

- 禁止手工改写 CSV、JSON、数值、坐标、误差线或图例；
- 禁止使用未锁定实验、单次最好结果或截图作为正式结论；
- 禁止修改 `vllm_fl/` 调度、KV Cache、算子和平台核心代码；
- 禁止将个人材料、模型、数据集和大型原始日志提交仓库。

**验收标准**

- 每张图可由仓库脚本从锁定数据重新生成；
- 图表数值与周邦翔的最终 CSV 一致；
- 每个结论都能沿证据索引回到提交号和原始结果；
- 图中明确区分基线、单项优化、组合优化和未测项；
- 陈梓弘和周邦翔完成图表数值交叉检查。

#### 小组共同任务

- **S6-G01：完整彩排。** 三人从 `phase6/integration` 检出，完成服务启动、简单请求、正式 Benchmark、精度评测、结果生成和证据索引；交付 `docs/evidence/phase-6/group/final-rehearsal-report.md`。
- **S6-G02：发布候选检查。** 陈梓弘确认范围，周邦翔确认测试，朱健辉确认图表和证据；交付 `docs/evidence/phase-6/group/release-candidate-checklist.md`。
- **S6-G03：阶段交接。** 记录最终提交号、结果文件、未关闭问题、S7 接收人和下一步；交付 `docs/evidence/phase-6/group/handoff.md`。

#### 本阶段不做

- 不新增未经证据支持的性能机制；
- 不修改模型行为、量化、投机采样或 Benchmark 口径；
- 不把单次最佳结果当作最终成绩；
- 不删除失败测试、OOM、超时、精度下降或不支持平台记录；
- 不把计划、截图、口头说明冒充代码、测试或复现证据。

#### S6 单周关闭门禁

- 三名成员均完成个人任务 PR，且每个 PR 有交付物、验证命令、实际结果和交叉 Review；
- `phase6/release-candidate` 已汇总到 `phase6/integration`；
- 两平台四场景的最终结果、精度、TTFT、显存和失败请求已锁定；
- 结果图表可由脚本重生成，证据索引完整；
- 所有遗留项均登记 Owner、影响、依赖、下一步和截止时间；
- 陈梓弘确认范围与最终提交，周邦翔确认质量与测试，朱健辉确认结果证据；
- 未达到以上条件不得关闭 Issue #7，不得进入 S7 最终交付。

#### S6 交付包

```text
docs/evidence/phase-6/
├── zhou-validation/
│   ├── measurement-matrix.md
│   ├── performance-results.csv
│   ├── final-regression-report.md
│   └── commands-and-environment.md
├── chen-integration/
│   ├── candidate-lock.md
│   ├── ablation-matrix.md
│   ├── final-commit-lock.md
│   └── known-limitations.md
├── zhu-evidence/
│   ├── figure-source.csv
│   ├── throughput-comparison.png
│   ├── latency-and-memory-comparison.png
│   ├── evidence-index.md
│   └── presentation-result-summary.md
└── group/
    ├── final-rehearsal-report.md
    ├── release-candidate-checklist.md
    └── handoff.md
```

闭环：个人任务 PR → `phase6/release-candidate` → `phase6/integration` → `flagos-2026-s2`；S6 的最终提交、结果表和证据索引直接作为 S7 技术报告、README 和提交包的输入。

### S7：技术报告、README、提交包与复现演练

- **Issue**：`[S7] 完成技术报告、复现说明与最终提交包`
- **GitHub Issue**：[#8](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/8)
- **阶段信息**：冻结、发布、答辩与最终提交阶段；主责：陈梓弘；复现质量负责人：周邦翔；图表与提交证据负责人：朱健辉。
- **输入门禁**：S6 最终提交、双平台结果、消融、精度、已知限制和证据索引全部锁定。
- **输出去向**：赛事提交 ZIP、`report.pdf`、`readme.md`、最终标签、演示材料和获奖后官方 PR。

#### 小组目标

从唯一冻结提交形成评委可在新环境独立理解、安装、启动、评测和核验的提交包。代码、报告、README、图表与演示使用同一组锁定结果，任何未完成能力明确披露。

#### 分支与合并要求

- 阶段集成分支：`phase7/integration`；发布候选分支：`phase7/release-candidate`；
- 陈梓弘：`task/chen-s7-release`；周邦翔：`task/zhou-s7-reproduction`；朱健辉：`task/zhu-s7-presentation-evidence`；
- 个人 PR → `phase7/release-candidate` → `phase7/integration` → `flagos-2026-s2`；
- 最终标签只允许指向通过全部门禁的唯一提交；共享分支禁止 force-push 和变基改写历史。

#### 陈梓弘个人任务：报告、范围与发布包

- **任务编号**：`S7-C01` 锁定最终范围与提交；`S7-C02` 编写技术报告；`S7-C03` 整理源码与提交清单；`S7-C04` 创建最终标签和发布记录。
- **个人分支**：`task/chen-s7-release`。
- **交付物**：`report.pdf`、`docs/evidence/phase-7/chen-release/final-scope.md`、`submission-manifest.md`、`final-release-record.md`、最终汇总 PR。
- **允许修改**：报告、发布文档、打包清单及清除已确认的临时调试输出；代码变化仅限阻断复现的已评审修复。
- **禁止修改**：不得在冻结后新增性能机制、改模型行为或改写锁定结果；不得把未完成项写成已实现。
- **验收**：报告每项策略都能追溯到代码、提交、实验和图表；提交清单中文件存在且版本一致。

#### 周邦翔个人任务：README 与干净环境复现

- **任务编号**：`S7-Z01` 编写环境/安装/启动/评测 README；`S7-Z02` 在新目录或新机器从零复现；`S7-Z03` 执行最终精度、性能和稳定性回归；`S7-Z04` 登记故障排查与限制。
- **个人分支**：`task/zhou-s7-reproduction`。
- **交付物**：`readme.md`、`docs/evidence/phase-7/zhou-reproduction/clean-install-report.md`、`final-regression-report.md`、`troubleshooting.md`。
- **允许修改**：复现说明、外围安装/校验脚本和证据目录。
- **禁止修改**：不得使用本机隐含路径、缓存或未记录手工步骤；不得改变正式参数来使复现通过。
- **验收**：非作者严格按 README 可完成安装、启动、API、精度和 Benchmark；最终测试提交与发布提交一致。

#### 朱健辉个人任务：最终图表、演示与证据索引

- **任务编号**：`S7-H01` 核对报告图表数据；`S7-H02` 整理演示结果页和讲解顺序；`S7-H03` 建立最终提交证据索引；`S7-H04` 检查提交包隐私与无关文件。
- **个人分支**：`task/zhu-s7-presentation-evidence`。
- **交付物**：`docs/evidence/phase-7/zhu-evidence/final-figure-source.csv`、`presentation-outline.md`、`final-evidence-index.md`、`package-content-audit.md`。
- **允许修改**：锁定数据生成的图表、答辩文档、Markdown 索引和打包审计清单。
- **禁止修改**：不得修改核心代码、调度器、KV Cache、算子或实验数据；不得将个人简历、学号、密钥、模型和数据集打入提交包。
- **验收**：报告和演示中的数值均来自锁定 CSV，可回到提交号和原始结果；提交包无敏感或无关文件。

#### 小组共同任务

- `S7-G01`：完成两次全流程复现与十分钟答辩彩排；交付 `docs/evidence/phase-7/group/final-rehearsal-report.md`。
- `S7-G02`：三人交叉核对代码、报告、README、图表、限制和贡献；交付 `release-checklist.md`。
- `S7-G03`：形成赛事提交和获奖后 PR 交接；交付 `handoff.md`。

#### 最终提交包

- `vllm-plugin-FL/` 全部源码和编译脚本；涉及算子改动时包含 `FlagGems/`；
- `report.pdf`、`readme.md`、锁定结果、复现证据和许可证/版本说明；
- 不包含模型、数据集、密钥、个人材料、缓存、构建垃圾和大型原始日志。

#### S7 关闭门禁

- 三个个人 PR 完成 Review 并合并，发布候选汇总到阶段集成分支；
- `report.pdf`、`readme.md`、源码、结果与图表完全一致；
- 干净环境复现通过，`accuracy >= 0.95`，正式结果绑定最终提交；
- 最终标签指向唯一通过门禁提交，提交包完成 SHA256 登记；
- 陈梓弘签署范围与提交完整性，周邦翔签署复现和测试结论，朱健辉签署图表与证据一致性；
- 未达到以上条件不得创建最终标签或提交赛事平台。

#### 本阶段不做

- 不新增功能或优化机制，不为演示删除检查或失败测试，不伪造结果，不隐瞒已知限制。

闭环：S7 产物直接组成赛事提交包；组委会复现结果是最终验收。

## 8. Issue 总表

| 阶段 | Issue 标题 | 主负责人 | 分支 | 优先级 | 状态 |
|---|---|---|---|---|---|
| S0 | [#1](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/1) 建立项目治理、仓库安全与环境合同 | 陈梓弘 | `phase0/release-candidate` | P0 | 已创建 |
| S1 | [#2](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/2) 复现 MiniCPM5-2B 双平台性能与精度基线 | 周邦翔 | `phase1/release-candidate` | P0 | 已创建/等待算力 |
| S2 | [#3](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/3) 建立 Prefill/Decode 分层瓶颈地图与优化候选清单 | 陈梓弘 | `phase2/release-candidate` | P0 | 已创建/阻塞于 S1 |
| S3 | [#4](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/4) 优化运行时热路径与可复用缓冲区 | 陈梓弘 | `phase3/release-candidate` | P1 | 已创建/等待 S2 |
| S4 | [#5](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/5) 优化调度、KV Cache 与图执行协同路径 | 陈梓弘 | `phase4/release-candidate` | P1 | 已创建/等待 S2 |
| S5 | [#6](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/6) 基于 profiler 优化热点算子并完成双平台适配 | 陈梓弘 | `phase5/release-candidate` | P1 | 已创建/等待 S2 |
| S6 | [#7](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/7) 完成双平台组合优化、消融与回归验收 | 周邦翔 | `phase6/release-candidate` | P0 | 已创建/等待 S3-S5 |
| S7 | [#8](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/8) 完成技术报告、复现说明与最终提交包 | 陈梓弘 | `phase7/release-candidate` | P0 | 已创建/等待 S6 |

## 9. 实验记录模板

每次正式实验新增一条记录，至少包含：

```markdown
# EXP-<PLATFORM>-<STAGE>-<NNN>

- 日期与执行人：
- 平台/机器编号：
- 代码提交：
- FlagGems 提交：
- 镜像、驱动、运行时：
- 改动假设：
- 与上一实验唯一差异：
- 构建命令：
- 启动命令：
- Benchmark 命令：
- 场景与重复次数：
- Accuracy：
- Total tok/s（每次、中位数、变异系数）：
- Mean/P50/P99 TTFT：
- TPOT/ITL：
- 峰值显存：
- 请求成功/失败：
- 原始日志路径和校验值：
- 结论：保留 / 回退 / 继续诊断
- 风险与下一步：
```

## 10. 推荐仓库产物结构

```text
docs/
├── PROJECT_EXECUTION_PLAN.md
├── experiments/              # 小型实验登记，不放大型原始日志
└── results/                  # 锁定后的汇总表、结论和图表索引
scripts/
└── competition/              # 环境登记、外围执行和结果校验脚本
benchmarks/                   # 官方/上游 Benchmark，禁止为成绩修改口径
tests/                        # 单元、功能、E2E 和回归测试
vllm_fl/                      # 主要框架优化代码
```

## 11. 立即执行顺序

1. 开始 S0：邀请成员、确认 GitHub 用户名、验证 Push/PR 权限；
2. 提交本总方案到默认主分支 `flagos-2026-s2`；
3. 启用并创建 S0-S7 GitHub Issues；
4. 建立实验登记和版本清单模板；
5. 获得算力后启动 S1，不在无官方硬件证据时提前宣称性能收益；
6. S1 稳定后启动 S2；
7. 根据 S2 证据决定 S3-S5 的实际投入顺序。

## 12. 项目成功判定

项目成功不是“写了很多优化代码”，而是同时满足：

- 提升超过正常波动并在官方环境复现；
- 精度、TTFT、稳定性和合规性全部通过；
- 每项结论可从题目要求追溯到代码、实验、结果和报告；
- 三名成员的任务边界与实际能力匹配；
- 最终提交材料让评委无需口头补充即可完成复现。
