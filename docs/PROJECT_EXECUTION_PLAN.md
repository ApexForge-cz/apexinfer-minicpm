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
- 功能分支：从最新 `origin/flagos-2026-s2` 创建；
- 禁止在 GitHub `main` 上开发比赛功能，避免脱离赛事指定版本；
- 禁止 force push、重写共享分支历史或将个人材料提交到公开仓库。

### 3.2 分支命名

| 类型 | 格式 | 示例 |
|---|---|---|
| 性能优化 | `perf/<stage>-<topic>` | `perf/s4-scheduler-kvcache` |
| Benchmark/实验 | `bench/<stage>-<topic>` | `bench/s1-baseline-harness` |
| 测试 | `test/<stage>-<topic>` | `test/s6-regression-matrix` |
| 文档 | `docs/<stage>-<topic>` | `docs/s7-final-delivery` |
| 修复 | `fix/<stage>-<topic>` | `fix/s5-metax-attention` |

### 3.3 标准工作流

组员首次加入、每日拉取、任务分支、提交、Push、PR、冲突处理和收工步骤，统一参见 [TEAM_WORKFLOW.md](./TEAM_WORKFLOW.md)。以下为最简流程摘要。

```bash
git switch flagos-2026-s2
git pull --ff-only origin flagos-2026-s2
git switch -c <任务分支>

# 完成一个可验证的小任务后
git add <明确文件>
git commit -m "<type>: <清晰说明>"
git push -u origin <任务分支>
```

随后创建 Pull Request 到 `flagos-2026-s2`。每个 PR 必须：

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
- **主负责人**：陈梓弘
- **协作者**：周邦翔、朱健辉
- **建议分支**：`chore/s0-project-bootstrap`
- **前置依赖**：无
- **目标**：让三名成员使用同一赛事基线、分支规则、结果格式和安全边界开展工作。

具体任务：

- [ ] 邀请两名成员加入 Fork，并验证各自可创建分支和 PR；
- [ ] 记录 `origin`、`upstream`、默认分支和官方基线提交；
- [ ] 建立 `docs/experiments/`、`docs/results/`、`scripts/competition/` 的用途约定；
- [ ] 补充 `.gitignore`，排除模型、数据集、密钥、原始日志、trace 和本地结果；
- [ ] 定义实验编号：`EXP-平台-阶段-序号`，例如 `EXP-METAX-S1-001`；
- [ ] 定义 PR 模板字段：Issue、平台、场景、正确性、性能、风险、回退；
- [ ] 确认天数与沐曦算力申请、账号、可用时段和负责人；
- [ ] 建立版本清单模板和实验登记模板。

交付物：

- 本总方案；
- 环境与版本清单模板；
- 实验登记模板；
- 安全的 `.gitignore`；
- 可执行的团队 Git 工作流。

验收条件：

- 三名成员都能从 `origin/flagos-2026-s2` 创建分支并发起 PR；
- `git status` 不显示模型、数据、密钥或大型结果目录；
- 任意成员能根据文档解释代码应提交到哪里、结果保存到哪里；
- 仓库默认分支与官方赛事分支关系已写清。

闭环：S0 的版本、目录和记录模板直接作为 S1 的输入；缺少任一项不得开始正式基线测量。

### S1：双平台基线、精度与测量稳定性

- **Issue**：`[S1] 复现 MiniCPM5-2B 双平台性能与精度基线`
- **GitHub Issue**：[#2](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/2)
- **主负责人**：周邦翔
- **最终负责**：陈梓弘
- **协作者**：朱健辉
- **建议分支**：`bench/s1-baseline-harness`
- **前置依赖**：S0 完成；至少获得一种官方算力
- **目标**：建立可信的本团队基线，证明测量链路稳定且与官方口径一致。

具体任务：

- [ ] 登记芯片、显存、驱动、运行时、Python、PyTorch、vLLM、插件和 FlagGems 版本；
- [ ] 按官方命令启动服务，保存完整启动日志；
- [ ] 执行简单 API 请求，检查模型名、返回结构和非空输出；
- [ ] 运行 MATH-500 Level 3，保存 Evalscope 命令、工作目录和汇总结果；
- [ ] 分别运行 4k/16k 场景，每个平台每场景至少 3 次有效重复；
- [ ] 记录 `total tokens/s`、Output tok/s、duration、Mean/P50/P99 TTFT、TPOT、ITL 和失败请求；
- [ ] 计算中位数、极差和变异系数；
- [ ] 由朱健辉生成不改变数据含义的基线对比表和图；
- [ ] 若与官方基线偏差超过 1%，先排查环境、预热、后台负载和失败请求。

建议代码与产物位置：

- `scripts/competition/`：外围启动、采集和校验脚本；
- `docs/experiments/EXP-*-S1-*.md`：实验记录；
- `docs/results/baseline-summary.md`：汇总结论；
- 原始大型日志保存在算力环境，不直接提交仓库，只登记路径和校验值。

验收条件：

- 精度 `accuracy >= 0.95`；
- 四个平台-场景组合均有 3 次有效结果；
- 每组结果变异系数不高于 1%，否则形成波动原因说明；
- 中位数低于官方基线 1% 以上时不得宣称复现成功；
- TTFT 超过基线 +1% 时标记阻断并排查；
- 另一名成员能用同一脚本复现结果文件结构。

闭环：S1 输出稳定基线、命令和日志索引，作为 S2 profiler 对照及后续所有优化的唯一分母。

### S2：分层性能剖析与瓶颈地图

- **Issue**：`[S2] 建立 Prefill/Decode 分层瓶颈地图与优化候选清单`
- **GitHub Issue**：[#3](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/3)
- **主负责人**：陈梓弘
- **协作者**：周邦翔
- **支持**：朱健辉
- **建议分支**：`perf/s2-profile-bottlenecks`
- **前置依赖**：S1 至少完成一个平台的稳定基线
- **目标**：用 profiler 和框架日志回答“时间花在哪里、显存卡在哪里、哪个候选值得开发”。

具体任务：

- [ ] 对 4k/16k 分别采集 Prefill 与 Decode 时间分解；
- [ ] 记录请求等待、每轮运行请求数、批次 token 数和调度空隙；
- [ ] 检查 `vllm_fl/worker/model_runner.py` 的输入准备、执行、采样和同步路径；
- [ ] 检查 `vllm_fl/platform.py` 与 `vllm_fl/compilation/graph.py` 的图捕获、回退和形状覆盖；
- [ ] 记录 KV Cache 使用、块分配/回收、碎片、临时 workspace 和峰值显存；
- [ ] 统计累计耗时最高的 10 个算子及调用次数；
- [ ] 分别检查 `vendor/txda/` 和 `vendor/metax/` 的注册、补丁和回退路径；
- [ ] 将候选按“预期收益、实现成本、正确性风险、平台范围、验证成本”排序；
- [ ] 明确哪些候选进入 S3、S4、S5，哪些候选拒绝及原因。

交付物：

- `docs/results/bottleneck-map.md`；
- 4k/16k、Prefill/Decode、天数/沐曦四维瓶颈表；
- 热点算子清单和候选优化决策表；
- profiler 原始文件的外部路径与校验值。

验收条件：

- 每个候选都能指向日志、trace 或代码路径，禁止纯经验猜测；
- 至少给出一个框架级候选和一个算子级候选，或以证据说明某类候选不存在；
- profiler 运行与正式 Benchmark 分开，不能把 profiler 结果当正式吞吐成绩；
- 陈梓弘和周邦翔共同签字确认候选排序。

闭环：S2 的候选决策表是 S3-S5 的启动门；未进入清单的优化不得占用官方算力开发。

### S3：低风险运行时与执行路径优化

- **Issue**：`[S3] 优化运行时热路径与可复用缓冲区`
- **GitHub Issue**：[#4](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/4)
- **主负责人**：陈梓弘
- **测量负责人**：周邦翔
- **建议分支**：`perf/s3-runtime-hotpath`
- **前置依赖**：S2 证明运行时或执行路径存在可观测开销
- **目标**：优先获得不改变模型数值语义、容易回退的性能增益。

候选任务，仅执行被 S2 证据支持的条目：

- [ ] 缓存可复用的元数据、索引、形状或调度辅助结构；
- [ ] 减少热路径 Python 对象创建、重复检查和同步；
- [ ] 复用临时张量或 workspace，减少高频申请和释放；
- [ ] 消除可证明冗余的数据布局或设备转换；
- [ ] 为改动补充 CPU 单测或最小 GPU 功能测试；
- [ ] 对每个改动做单项 A/B，而不是一次合并多个机制；
- [ ] 记录回退开关或恢复方式。

重点代码范围：

- `vllm_fl/worker/model_runner.py`；
- `vllm_fl/worker/worker.py`；
- `vllm_fl/platform.py`；
- S2 实际定位到的相关模块。

验收条件：

- 单项优化至少在一个目标场景超过 1% 波动区间，或有明确 CPU/同步/显存证据支持保留；
- 另一场景不得出现超过 1% 的无解释回退；
- 精度仍不低于 `0.95`，输出和请求成功率正常；
- 不改变官方服务参数和 Benchmark；
- 单项提交可独立回退，实验登记完整。

闭环：通过门禁的提交进入 S6 候选组合；无收益或不稳定的提交关闭并保留负结果记录。

### S4：调度、KV Cache 与图执行协同优化

- **Issue**：`[S4] 优化调度、KV Cache 与图执行协同路径`
- **GitHub Issue**：[#5](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/5)
- **主负责人**：陈梓弘
- **协作者**：周邦翔
- **建议分支**：`perf/s4-scheduler-kvcache`
- **前置依赖**：S2 证明存在调度空隙、KV Cache 开销或图回退问题
- **目标**：提高设备持续忙碌时间和有效并发，同时守住 TTFT 与显存边界。

候选任务：

- [ ] 分析 Prefill/Decode 混合批次、形状抖动和调度空隙；
- [ ] 分析 `model_runner.py` 的 batch reorder、输入准备与 attention metadata 构建；
- [ ] 检查 KV Cache 块映射、申请、回收和元数据更新路径；
- [ ] 检查长上下文下的碎片、无效搬运和峰值显存；
- [ ] 检查天数 `FULL_DECODE_ONLY` 图模式的捕获命中与回退原因；
- [ ] 检查沐曦平台图执行能力和 `platform.py` 中的兼容性分支；
- [ ] 仅使用运行时状态设计通用策略，禁止读取固定场景常量做特判；
- [ ] 补充不同长度、批次变化、空队列和显存压力测试。

重点代码范围：

- `vllm_fl/worker/model_runner.py`；
- `vllm_fl/worker/scheduler_fl.py`；
- `vllm_fl/compilation/graph.py`；
- `vllm_fl/platform.py`。

验收条件：

- 4k 或 16k 的 `total tokens/s` 中位数提升超过 1%；
- 另一目标场景无超过 1% 的无解释退化；
- TTFT 满足官方门槛，峰值显存不引入 OOM；
- 在非目标长度的最小回归用例中不出现硬编码行为；
- 精度、服务稳定性和图执行回退日志通过检查。

闭环：通过的调度/KV/图改动形成可独立启用的组合候选，交给 S6 消融；失败候选回退并记录瓶颈是否已被证伪。

### S5：热点算子、FlagGems 与双平台适配

- **Issue**：`[S5] 基于 profiler 优化热点算子并完成双平台适配`
- **GitHub Issue**：[#6](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/6)
- **主负责人**：陈梓弘
- **协作者**：周邦翔
- **支持**：朱健辉负责结果对比图，不参与内核实现
- **建议分支**：`perf/s5-hot-operators`
- **前置依赖**：S2 给出明确热点算子、调用形状和平台证据
- **目标**：对累计耗时显著的真实热点实施融合、内核或编译优化。

候选任务：

- [ ] 确认热点属于 Attention、RMSNorm、RoPE、激活、GEMM、采样或数据转换中的哪一类；
- [ ] 检查 FlagGems `v5.3.5` 已有实现、调用路径和回退路径；
- [ ] 评估相邻逐元素计算融合、减少中间写回或 kernel launch 的可行性；
- [ ] 分析访存、并行划分、shape specialization 和编译缓存；
- [ ] 在 `vendor/txda/` 或 `vendor/metax/` 中隔离平台差异；
- [ ] 如需修改 FlagGems，单独 Fork 并记录插件提交与 FlagGems 提交的对应关系；
- [ ] 增加参考实现对照、容差、边界 shape 和回退测试；
- [ ] 验证优化不是单纯切换算子或删掉选择逻辑。

重点代码范围由 S2 决定，可能包括：

- `vllm_fl/dispatch/backends/vendor/txda/`；
- `vllm_fl/dispatch/backends/vendor/metax/`；
- `vllm_fl/dispatch/backends/flaggems/impl/`；
- `vllm_fl/ops/`；
- 独立 FlagGems Fork 中的对应算子。

验收条件：

- 数值结果与参考实现处于明确容差内；
- 目标场景中位数提升超过 1%，或热点累计时间有可重复的显著下降且端到端无退化；
- 不支持的平台走清晰回退路径，不破坏另一平台；
- 新增算子测试和端到端精度均通过；
- 技术报告能够说明机制、适用 shape、平台差异和代价。

闭环：通过的算子改动及其依赖版本进入 S6；未达端到端收益的实验只作为诊断记录，不进入最终组合。

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
- **主负责人**：陈梓弘
- **协作者**：周邦翔、朱健辉
- **建议分支**：`docs/s7-final-delivery`
- **前置依赖**：S6 完成并锁定最终提交
- **目标**：形成评委能够独立理解、编译、运行和验证的完整作品。

具体任务：

- [ ] 清理源码、调试开关、临时打印和无关文件；
- [ ] 编写 README：环境、版本、编译、启动、评测、结果校验、故障排查；
- [ ] 编写技术报告：问题、瓶颈、方法、实现、实验、消融、平台差异、创新与限制；
- [ ] 对照代码检查报告中的每项策略确实存在；
- [ ] 对照结果文件检查摘要、正文、表格和图表数值一致；
- [ ] 在干净环境完成一次从安装到结果校验的复现演练；
- [ ] 整理 `vllm-plugin-FL/`，涉及算子时整理 `FlagGems/`；
- [ ] 检查无模型、数据集、密钥、个人材料和不必要大文件；
- [ ] 准备最终 GitHub PR 标题和变更说明；
- [ ] 在截止时间前完成平台提交和获奖后的官方 PR 要求。

验收条件：

- `report.pdf`、`readme.md`、源码和结果完全一致；
- 全新环境严格按 README 能完成安装、启动和评测；
- 关键命令可以直接执行，无隐含手工步骤；
- 提交包结构符合赛事要求；
- 团队三人完成交叉检查并签字确认。

闭环：S7 产物直接组成赛事提交包；组委会复现结果是项目最终验收。

## 8. Issue 总表

| 阶段 | Issue 标题 | 主负责人 | 分支 | 优先级 | 状态 |
|---|---|---|---|---|---|
| S0 | [#1](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/1) 建立项目治理、仓库安全与环境合同 | 陈梓弘 | `chore/s0-project-bootstrap` | P0 | 已创建 |
| S1 | [#2](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/2) 复现 MiniCPM5-2B 双平台性能与精度基线 | 周邦翔 | `bench/s1-baseline-harness` | P0 | 已创建/等待算力 |
| S2 | [#3](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/3) 建立 Prefill/Decode 分层瓶颈地图与优化候选清单 | 陈梓弘 | `perf/s2-profile-bottlenecks` | P0 | 已创建/阻塞于 S1 |
| S3 | [#4](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/4) 优化运行时热路径与可复用缓冲区 | 陈梓弘 | `perf/s3-runtime-hotpath` | P1 | 已创建/等待 S2 |
| S4 | [#5](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/5) 优化调度、KV Cache 与图执行协同路径 | 陈梓弘 | `perf/s4-scheduler-kvcache` | P1 | 已创建/等待 S2 |
| S5 | [#6](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/6) 基于 profiler 优化热点算子并完成双平台适配 | 陈梓弘 | `perf/s5-hot-operators` | P1 | 已创建/等待 S2 |
| S6 | [#7](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/7) 完成双平台组合优化、消融与回归验收 | 周邦翔 | `test/s6-integration-ablation` | P0 | 已创建/等待 S3-S5 |
| S7 | [#8](https://github.com/ApexForge-cz/apexinfer-minicpm/issues/8) 完成技术报告、复现说明与最终提交包 | 陈梓弘 | `docs/s7-final-delivery` | P0 | 已创建/等待 S6 |

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
