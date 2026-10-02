# S0 正式实验登记模板

## 1. 使用规则

每次正式实验复制“单次实验记录”部分，保存为独立记录。实验编号格式为：

```text
EXP-<PLATFORM>-S<STAGE>-<NNN>
```

- `PLATFORM`：`TXDA`（天数 BI-V150）或 `METAX`（沐曦 C500）。
- `STAGE`：`0` 至 `7`。
- `NNN`：同平台、同阶段从 `001` 开始递增，不复用、不覆盖。

正式性能结论至少需要三次有效重复；失败运行必须保留并说明原因。原始模型、
数据集、大型日志和 profiler trace 不进入 Git，只在记录中保存受控位置和校验值。

## 2. 单次实验记录

### 2.1 身份与追溯

| 字段 | 值 |
|---|---|
| 实验编号 | `EXP-<PLATFORM>-S<STAGE>-<NNN>` |
| 实验标题 | `PENDING` |
| 日期、开始/结束时间及时区 | `PENDING` |
| 执行人 | `PENDING` |
| 复核人 | `PENDING` |
| 平台与机器别名 | `PENDING` |
| 环境锁定编号（ENV-ID） | `PENDING` |
| 关联 Issue/任务编号 | `PENDING` |
| 运行状态 | `PENDING / PASSED / FAILED / BLOCKED` |

### 2.2 版本锁定

| 组件 | 版本/提交 | 获取命令 | 证据路径或 SHA256 |
|---|---|---|---|
| vllm-plugin-FL | `PENDING` | `git rev-parse HEAD` | `PENDING` |
| 工作分支 | `PENDING` | `git branch --show-current` | `PENDING` |
| 工作区状态 | `PENDING` | `git status --short` | `PENDING` |
| FlagGems | `PENDING` | 包版本及源码 `git rev-parse HEAD` | `PENDING` |
| Python | `PENDING` | `python --version` | `PENDING` |
| PyTorch | `PENDING` | Python 包版本查询 | `PENDING` |
| vLLM | `PENDING` | Python 包版本查询 | `PENDING` |
| 驱动/运行时 | `PENDING` | 厂商认可的版本查询命令 | `PENDING` |
| 容器镜像 digest | `PENDING` | 平台镜像/容器查询命令 | `PENDING` |

若工作区不是干净状态，必须记录 `git diff` 的受控证据或停止正式实验。

### 2.3 假设与唯一变量

- 改动假设：`PENDING`
- 与对照实验的唯一差异：`PENDING`
- 对照实验编号：`PENDING`
- 预期影响指标：`PENDING`
- 正确性风险与回退方式：`PENDING`

一次实验原则上只改变一个因素。无法保持单一变量时，必须明确列出所有差异，
且不得把结果用于单项优化收益声明。

### 2.4 命令

```text
# 构建命令
PENDING

# 启动命令（凭据必须使用占位符，不写真实 Token/Cookie）
PENDING

# 精度命令
PENDING

# Benchmark 命令
PENDING

# 结果解析与校验命令
PENDING
```

### 2.5 场景合同

| 场景 | Input length | Output length | Concurrency | Num prompts | 重复要求 |
|---|---:|---:|---:|---:|---:|
| 4k | 4096 | 1024 | 64 | 256 | 至少 3 次有效运行 |
| 16k | 16384 | 1024 | 64 | 128 | 至少 3 次有效运行 |

不得改变官方请求数据、长度、并发、请求数、采样参数和指标算法。

### 2.6 运行结果

| 场景 | Run | 有效 | Duration (s) | Output tok/s | Total tok/s | Mean TTFT (ms) | P50/P99 TTFT | TPOT/ITL | 峰值显存 | 成功/失败请求 | 原始结果 SHA256 |
|---|---:|---|---:|---:|---:|---:|---|---|---|---|---|
| `PENDING` | 1 | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` |
| `PENDING` | 2 | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` |
| `PENDING` | 3 | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` | `PENDING` |

| 汇总字段 | 值 |
|---|---|
| Total tok/s 中位数 | `PENDING` |
| Total tok/s 极差 | `PENDING` |
| Total tok/s 变异系数 | `PENDING` |
| TTFT 与基线差异 | `PENDING` |
| 相对基线收益 | `PENDING` |
| 失败运行及原因 | `PENDING` |

小于或等于 1% 的性能差异视为正常波动，不宣称有效提升。

### 2.7 精度与正确性

| 字段 | 值 |
|---|---|
| 数据集/子集 | MATH-500 Level 3 |
| Accuracy | `PENDING` |
| 门槛 | `>= 0.95` |
| 通过 | `PENDING` |
| 正确性/回归命令 | `PENDING` |
| 结果路径与 SHA256 | `PENDING` |

### 2.8 原始证据索引

| 证据 | 仓库外受控路径/对象 ID | SHA256 | 保留期限 | 可访问角色 |
|---|---|---|---|---|
| 服务日志 | `PENDING` | `PENDING` | `PENDING` | `PENDING` |
| Benchmark 原始输出 | `PENDING` | `PENDING` | `PENDING` | `PENDING` |
| 精度结果 | `PENDING` | `PENDING` | `PENDING` | `PENDING` |
| profiler trace（如有） | `PENDING` | `PENDING` | `PENDING` | `PENDING` |

仓库中的摘要不得包含凭据、私有 IP、个人材料或未经筛选的大型日志。

### 2.9 结论

- 决策：`保留 / 回退 / 继续诊断 / 阻塞`
- 是否满足精度门槛：`PENDING`
- 是否满足性能声明条件：`PENDING`
- 已知限制：`PENDING`
- 风险：`PENDING`
- 下一步及负责人：`PENDING`
- 复核意见：`PENDING`

## 3. 提交前检查

- [ ] 实验编号唯一，平台、阶段和环境编号可以互相追溯。
- [ ] 插件、FlagGems、镜像、驱动、运行时和命令均已登记。
- [ ] 正式性能结果至少有三次有效重复，失败运行没有被删除。
- [ ] 4k/16k 参数与官方合同一致。
- [ ] 精度结果已记录；没有精度结果时明确标记阻塞，不宣称收益。
- [ ] 原始结果有路径和 SHA256，且大型文件未提交 Git。
- [ ] 摘要中没有密钥、Cookie、账号、私有 IP 或个人材料。
- [ ] 结论包含保留/回退条件、风险和下一步。
