# S0 双平台环境与版本合同

## 1. 目的与适用范围

本文档定义 MiniCPM5-2B 赛事环境的统一登记字段，供 S1 基线、S2
剖析以及后续优化和回归共同引用。目标平台为天数 BI-V150 与沐曦曦云
C500（64 GB）。本文只定义合同和待采集项，不把仓库默认值当作官方硬件
实测值。

环境记录必须使用以下状态之一：

| 状态 | 含义 |
|---|---|
| `VERIFIED` | 已在目标机器执行采集命令，并保存可追溯证据 |
| `PENDING` | 已知需要采集，但当前没有目标机器或权限 |
| `MISMATCH` | 实测值与赛事、镜像或仓库预期不一致，尚未获准继续 |
| `N/A` | 经负责人确认该字段不适用于此平台 |

禁止用 README、Dockerfile、另一平台或个人电脑的值代替目标机器实测值。
任何 `MISMATCH` 必须在 S1 正式测量前由项目负责人给出处理结论。

## 2. 固定项目合同

| 字段 | 合同值 | 来源/说明 |
|---|---|---|
| 模型 | MiniCPM5-2B | 赛事指定；模型权重不得提交仓库 |
| 插件仓库 | `ApexForge-cz/apexinfer-minicpm` | 团队 Fork |
| 赛事代码分支 | `flagos-2026-s2` | 最终稳定落点 |
| Python 支持范围 | `>=3.10,<3.14` | `pyproject.toml`；正式值仍以目标镜像实测为准 |
| vLLM 项目依赖 | `0.24.0` | `pyproject.toml` 测试依赖；正式值必须实测 |
| FlagGems 赛事预期 | `v5.3.5` | 项目执行计划；与 README 的 `v5.3.4` 差异待负责人确认 |
| 性能场景 | 4k、16k | 不得修改官方输入长度、输出长度、并发和请求数 |
| 精度门槛 | `accuracy >= 0.95` | MATH-500 Level 3 |

## 3. 环境记录索引

每次正式环境锁定新增一行；环境变化必须创建新 `ENV-ID`，不得覆盖旧记录。

| ENV-ID | 平台 | 机器别名 | 采集时间（含时区） | 执行人 | 状态 | 证据目录 | 清单 SHA256 |
|---|---|---|---|---|---|---|---|
| `ENV-TXDA-001` | 天数 BI-V150 | `PENDING` | `PENDING` | 周邦翔 | `PENDING` | `PENDING` | `PENDING` |
| `ENV-METAX-001` | 沐曦曦云 C500（64 GB） | `PENDING` | `PENDING` | 周邦翔 | `PENDING` | `PENDING` | `PENDING` |

机器只使用团队认可的非敏感别名，不登记公网 IP、账号、Token、Cookie、
SSH 私钥或算力券。

## 4. 单个平台环境明细模板

为每个 `ENV-ID` 复制并填写一份本节。所有版本值都必须附采集命令和证据，
无法采集时保持 `PENDING` 并写明原因。

### 4.1 基础设施与硬件

| 字段 | 实测值 | 状态 | 采集命令/来源 | 证据路径或校验值 |
|---|---|---|---|---|
| ENV-ID | `PENDING` | `PENDING` | 本文档编号规则 | `PENDING` |
| 平台/云厂商 | `PENDING` | `PENDING` | 资源审批记录 | `PENDING` |
| 芯片厂商与型号 | `PENDING` | `PENDING` | 厂商认可的设备查询命令 | `PENDING` |
| 芯片数量 | `PENDING` | `PENDING` | 厂商认可的设备查询命令 | `PENDING` |
| 单卡显存 | `PENDING` | `PENDING` | 厂商认可的设备查询命令 | `PENDING` |
| 机器别名 | `PENDING` | `PENDING` | 团队资源登记表 | `PENDING` |
| CPU/内存 | `PENDING` | `PENDING` | `lscpu`、`free -h` | `PENDING` |
| OS/内核 | `PENDING` | `PENDING` | `cat /etc/os-release`、`uname -a` | `PENDING` |
| 容器镜像与 digest | `PENDING` | `PENDING` | 平台镜像详情/容器查询命令 | `PENDING` |
| 挂载与可用空间 | `PENDING` | `PENDING` | `df -h` | `PENDING` |

### 4.2 驱动、运行时与框架

| 字段 | 实测值 | 状态 | 采集命令/来源 | 证据路径或校验值 |
|---|---|---|---|---|
| 设备驱动版本 | `PENDING` | `PENDING` | 厂商认可的驱动查询命令 | `PENDING` |
| 厂商运行时版本 | `PENDING` | `PENDING` | 厂商认可的运行时查询命令 | `PENDING` |
| Python | `PENDING` | `PENDING` | `python --version` | `PENDING` |
| PyTorch | `PENDING` | `PENDING` | `python -c "import torch; print(torch.__version__)"` | `PENDING` |
| vLLM | `PENDING` | `PENDING` | `python -c "import importlib.metadata as m; print(m.version('vllm'))"` | `PENDING` |
| vllm-plugin-FL 包版本 | `PENDING` | `PENDING` | `python -c "import importlib.metadata as m; print(m.version('vllm-plugin-fl'))"` | `PENDING` |
| vllm-plugin-FL Git 提交 | `PENDING` | `PENDING` | `git rev-parse HEAD` | `PENDING` |
| FlagGems 包版本 | `PENDING` | `PENDING` | `python -c "import importlib.metadata as m; print(m.version('flag-gems'))"` | `PENDING` |
| FlagGems Git 提交 | `PENDING` | `PENDING` | FlagGems 工作树内执行 `git rev-parse HEAD` | `PENDING` |
| FlagCX/通信库 | `PENDING` | `PENDING` | 平台认可的版本查询命令 | `PENDING` |
| 编译器/CMake | `PENDING` | `PENDING` | `cc --version`、`cmake --version` | `PENDING` |

### 4.3 模型、数据与运行配置

| 字段 | 实测值 | 状态 | 采集命令/来源 | 证据路径或校验值 |
|---|---|---|---|---|
| MiniCPM5-2B 可访问性 | `PENDING` | `PENDING` | 只记录“可访问/不可访问”，不提交权重和真实私有路径 | `PENDING` |
| MATH-500 数据可访问性 | `PENDING` | `PENDING` | 只记录“可访问/不可访问”，不提交数据集 | `PENDING` |
| `VLLM_VENDOR` | `PENDING` | `PENDING` | `printenv VLLM_VENDOR` | `PENDING` |
| `VLLM_PLUGINS` | `PENDING` | `PENDING` | `printenv VLLM_PLUGINS` | `PENDING` |
| 其他非敏感环境变量 | `PENDING` | `PENDING` | 使用环境变量白名单采集 | `PENDING` |
| 构建命令 | `PENDING` | `PENDING` | S1 `commands.md` | `PENDING` |
| 启动命令 | `PENDING` | `PENDING` | S1 `commands.md` | `PENDING` |
| Benchmark 命令 | `PENDING` | `PENDING` | 官方命令，不修改测量口径 | `PENDING` |

禁止执行或保存无筛选的 `env`/`printenv` 输出，以免把密钥和凭据写入证据。

## 5. 采集与校验流程

1. 在目标机器创建本次 `ENV-ID`，确认执行人和采集时间。
2. 逐项执行表中的命令，将原始输出放在仓库外受控目录。
3. 仓库只提交去敏后的摘要、证据相对引用和 SHA256。
4. 对照固定项目合同；差异标记为 `MISMATCH`，不得静默改成预期值。
5. 由另一名成员复核至少一个版本命令、插件提交号和清单校验值。
6. 全部必填项为 `VERIFIED` 后，才允许在 S1 的 `environment-lock.md` 引用该
   `ENV-ID`。

## 6. 变更与复核记录

| 日期 | ENV-ID | 变更原因 | 复核人 | 结论 |
|---|---|---|---|---|
| `PENDING` | `PENDING` | 初始环境待官方算力实测 | `PENDING` | `PENDING` |
