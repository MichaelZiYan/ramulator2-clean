User: 翻译，保持技术名词准确
Kimi: # Ramulator 2.1 用户指南

- [1. 概述](#1-概述)
- [2. 使用 Ramulator 2.1](#2-使用-ramulator-21)
- [3. 首次运行](#3-首次运行)
- [4. 编写配置](#4-编写配置)
- [5. 验证与回归测试](#5-验证与回归测试)
- [6. gem5 集成](#6-gem5-集成)
- [7. 将 Ramulator 用作纯 C++ 库](#7-将-ramulator-用作纯-c-库)
- [8. 扩展 Ramulator](#8-扩展-ramulator)
- [9. Ramulator 内部工作原理](#9-ramulator-内部工作原理)
- [10. 轨迹可视化工具](#10-轨迹可视化工具)

## 1. 概述

### 1.1 简介

Ramulator 2.1 是一款现代、模块化且可扩展的周期级 DRAM 模拟器。它是 Ramulator 1.0 [Kim+, CAL'16] 的继任者，也是对 Ramulator 2.0 [Luo+, CAL'23] 的重大重构。Ramulator 2.1 的目标是支持在内存控制器和 DRAM 中快速、敏捷地实现和评估设计变更，以满足日益增长的提升内存系统性能、安全性和可靠性的研究需求。Ramulator 2.1 具有简洁、模块化的 C++ 代码库，并配备自动生成的 Python 封装，可实现轻松且可脚本化的配置与扩展。用户只需专注于实现建模和模拟逻辑的 C++ 代码，无需担心手动维护样板代码。

Ramulator 2.1 既可用作接收内存轨迹的独立模拟器，也可轻松集成到其他模拟器中作为 DRAM 和内存控制器模拟库。我们目前提供 gem5 封装，可作为即插即用组件用于 SE 和 FS 模式（已在 gem5 v25.1 上测试，FS 模式已在启动后检查点上测试）。

本 GitHub 仓库包含 Ramulator 2.1 的公开版本。我们将不定期从内部版本同步代码框架改进、新增功能、错误修复等。Ramulator 2.1 欢迎您的贡献，以及内存系统领域的新想法和实现。

目前，Ramulator 2.1 为以下标准提供 DRAM 器件和内存控制器模型：
- DDR3、DDR4、DDR5
- GDDR6、GDDR7
- LPDDR5、LPDDR6
- HBM1、HBM2、HBM3、HBM4

Ramulator 2.0 的变更内容：
- 汇总的错误修复
- 对新型 DRAM 和控制器功能更全面的支持
- 更全面的测试和验证工作流
- 显著提升了易用性、配置和扩展的便捷性
- 整体代码质量改进

如果您在研究中使用了 Ramulator 2.1/2.0，请使用以下引用：
```
@misc{luo2026ramulator2.1,
	title        = {{Ramulator 2.1: A Composable Memory System Simulator for Modern DRAM Systems}},
	author       = {Luo, Haocong and Bostanc{\i}, F. Nisa and Olgun, Ataberk and Makeenkova, Maria and Malik, Ziad and Akdeniz, Ipek and Mutlu, Onur},
	year         = 2026,
	howpublished = {The 3rd Tutorial on Ramulator and DRAM Bender colocated with ICS}
}
@article{luo2024ramulator2.0,
	title   = {{Ramulator 2.0: A Modern, Modular, and Extensible DRAM Simulator}},
	author  = {Luo, Haocong and Tu{\u{g}}rul, Yahya Can and Bostanc{\i}, F. Nisa and Olgun, Ataberk and Ya{\u{g}}l{\i}k{\c{c}}{\i}, A. Giray and Mutlu, Onur},
	year    = 2024,
	journal = {IEEE Computer Architecture Letters}
}
```
### 1.2 仓库结构

- `src/`
  主模拟器实现的 C++ 代码。
- `python/`
  用于轻松、可脚本化配置 Ramulator 的 Python 封装。
- `examples/`
  可立即运行的示例配置和轨迹。
- `tests/`
  测试和验证工作流。
- `resources/gem5_wrappers/`
  用于 gem5 集成的参考封装代码。
- `visualizer/`
  基于浏览器的轨迹可视化工具（Nuxt/WebGL）。参见[第 10 节](#10-轨迹可视化工具)。

## 2. 使用 Ramulator 2.1

我们强烈建议使用我们的容器（Dockerfile 位于 `.devcontainer/Dockerfile`）以避免任何依赖问题。该仓库也配置为可一键作为开发容器打开，所有依赖项均已预装。开始使用和开发 Ramulator 2.1 的最简单方式是直接在 Ramulator 2.1 仓库的 GitHub 页面上创建 Codespace。如果您使用 Visual Studio Code，在首次克隆并打开仓库后，它应自动提示您在开发容器中重新打开仓库。

如果您想在本地设置容器，可以执行以下步骤：
```bash
docker compose up -d --build
docker compose exec ramulator2 bash
```

这样做会创建一个包含所有依赖项的容器，将 Ramulator 2.1 仓库挂载到 `/workspace`，并在容器 bash 中自动激活 `ramulator2-venv`。

如果您需要配置自己的环境，请参阅第 2.3 节了解详细说明。

### 2.2 快速入门

从仓库根目录执行：

```bash
mkdir -p build
cd build
cmake ..
make -j
cd ..
```

此默认构建会完成三件有用的事情：

- 在仓库根目录构建 `libramulator.so`
- 在 `python/ramulator/` 下构建 Python 扩展模块
- 运行代码生成器，使所有自动生成的代码与源代码同步

然后使用示例配置以独立模式运行 Ramulator 2.1：

```bash
python3 examples/example_config.py
```

您应该会看到一些示例统计信息被打印出来。如果您不需要在自定义环境中构建 Ramulator 2.1，可以直接前往第 3 节，了解如何使用和配置 Ramulator 2.1 的详细说明和指令。

### 2.3 构建要求

**必需：**

- C++20 编译器，如 `g++-12` 或 `clang++-15`
- CMake 3.14 或更新版本
- Python 3.10 或更新版本（如需 Python 绑定、CLI 或测试）

**由 CMake 自动获取（无需手动安装）：**

- `yaml-cpp` — YAML 配置解析
- `fmt` — C++ 打印格式化
- `nanobind` — Python-C++ 绑定（仅当 `RAMULATOR_PYTHON_BINDINGS=ON` 时）
- `argparse` — 命令行参数解析（vendored 于 `ext/`）

**Python 开发依赖**（使用 `pip install -r requirements-dev.txt` 安装）：

- `pytest` — 测试框架
- `matplotlib` — 延迟-吞吐量绘图
- `ruff` — Python 代码检查器和格式化工具

`setuptools >= 64` 作为构建后端是必需的，会通过 `pip install -e .` 自动拉取。

**可选：** `clang-format`。

以独立模式构建 Ramulator 2.1：
```bash
mkdir -p build
cd build
cmake ..
make -j
cd ..
```

如果您只需要纯 C++ 库而不需要 Python 绑定，用于您自己的模拟器：

```bash
mkdir -p build
cd build
cmake .. -DRAMULATOR_PYTHON_BINDINGS=OFF
make -j
cd ..
```

### 2.4 安装 Python 包

构建完成后，以可编辑模式安装 Python 包，以便在任何目录下 `python -m ramulator` 和 `import ramulator` 都能正常工作：

```bash
pip install -e .
```

然后直接运行示例：

```bash
python3 examples/example_config.py
```

如果您不想安装 Python 包，也可以将 `PYTHONPATH=python` 作为一次性替代方案：

```bash
PYTHONPATH=python python3 examples/example_config.py
```

## 3. 首次运行

### 3.1 理解 Ramulator 2.1 配置

`examples/example_config.py` 如下所示：

```python
"""Ramulator2 示例配置和模拟脚本"""

import ramulator

# 配置发送内存请求的前端
frontend = ramulator.frontend.SimpleO3(
    clock_ratio=8,
    traces=["./examples/traces/example_inst.trace"],
    num_expected_insts=500000,
    translation=ramulator.translation.NoTranslation(max_addr=2147483648),
)

# 创建 DRAM 配置
ddr4 = ramulator.dram.DDR4(org_preset="DDR4_8Gb_x8", timing_preset="DDR4_2400R", rank=1)
# 使用 DRAM 配置实例化内存控制器
ctrl = ramulator.controller.GenericDDR(
    dram=ddr4,
    scheduler=ramulator.scheduler.FRFCFS(),
    refresh_manager=ramulator.refresh_manager.AllBank(),
    row_policy=ramulator.row_policy.Open(),
    addr_mapper=ramulator.addr_mapper.RoBaRaCoCh(),
)
# 使用控制器创建内存系统
mem = ramulator.memory_system.GenericDRAM(
    clock_ratio=3,
    controllers=[ctrl],
    channel_mapper=ramulator.channel_mapper.CacheLineInterleave(),
)

# 运行模拟
sim = ramulator.Simulation(frontend, mem)
sim.run()

# sim.stats 返回所有模拟统计信息的嵌套 Python 字典
stats = sim.stats

# 此处为 `ramulator export` 的保护，它会捕获配置而不运行模拟
if stats:
    # 控制器统计信息位于 memory_system → controller 下
    ctrl_stats = stats["memory_system"]["controller"]

    print(f"Controller cycles:     {ctrl_stats['cycles']}")
    print(f"Avg read latency:      {ctrl_stats['avg_read_latency']:.1f} cycles")
    print(f"Read requests:         {ctrl_stats['num_read_reqs']}")
    print(f"Write requests:        {ctrl_stats['num_write_reqs']}")
    print(f"Row hits:              {ctrl_stats['row_hits']}")
    print(f"Row misses:            {ctrl_stats['row_misses']}")
    print(f"Row conflicts:         {ctrl_stats['row_conflicts']}")
```

Ramulator 2.1 中有两个顶层组件：

- `frontend`
  生成内存流量。在此示例中，它是一个由内存指令轨迹驱动的简单乱序核心模型。
- `memory_system`
  封装一个或多个内存控制器（通道）。每个控制器拥有一个 DRAM 器件模型。

任何 Ramulator 2.1 模拟都必须包含这两个组件。它们用于创建主模拟对象（`ramulator.Simulation(frontend, mem)`），作为模拟的入口点。

### 3.2 各部分功能说明

#### 前端

```python
frontend = ramulator.frontend.SimpleO3(
    clock_ratio=8,
    traces=["./examples/traces/example_inst.trace"],
    num_expected_insts=500000,
    translation=ramulator.translation.NoTranslation(max_addr=2147483648),
)
```

前端生成内存请求并将其发送到内存系统。在此示例中，`SimpleO3` 模拟一个具有 LLC 的简单乱序处理器。它读取一个或多个指令轨迹文件，每个文件对应一个处理器核心。模拟将运行直到退役指令数达到 `500000`。配置的前端不会对轨迹中的内存地址应用地址转换（即 `ramulator.translation.NoTranslation`）。`clock_ratio=8` 表示每 `x` 个内存时钟周期（即内存系统侧的 `clock_ratio=x`），前端将被触发 `8` 次。

#### DRAM 器件

```python
ddr4 = ramulator.dram.DDR4(
    org_preset="DDR4_8Gb_x8",
    timing_preset="DDR4_2400R",
    rank=2,
)
```

这包括：

- 一个组织预设，如裸片密度、DQ 宽度、Bank 数量等。
- 一个时序约束预设，如 tRCD、tRAS、tRP 等。
- 对两个预设的可选覆盖。在此示例中，我们设置 `rank=2`。您可以添加任意数量的覆盖。

如果您想了解此对象在运行时变成什么，第 9.3 节将详细介绍完整的 DRAM 器件模型及其背后的分层状态机。

#### 控制器

```python
ctrl = ramulator.controller.GenericDDR(
    dram=ddr4,
    scheduler=ramulator.scheduler.FRFCFS(),
    refresh_manager=ramulator.refresh_manager.AllBank(),
    row_policy=ramulator.row_policy.Open(),
    addr_mapper=ramulator.addr_mapper.RoBaRaCoCh(),
)
```

这为刚刚配置的 `ddr4` DRAM 配置了一个 GenericDDR 内存控制器。它具有 `FRFCFS`（First-Ready First-Come-First-Served，先就绪先服务）调度器、全 Bank 刷新、`Open` 行策略和 `RoBaRaCoCh` 地址映射器。

#### 内存系统

```python
mem = ramulator.memory_system.GenericDRAM(
    clock_ratio=3,
    controllers=[ctrl],
    channel_mapper=ramulator.channel_mapper.CacheLineInterleave(),
)
```

`GenericDRAM` 是一个围绕一个或多个控制器的薄顶层封装。它包含一个设置内存侧触发率的 `clock_ratio`、一个控制器列表（`controllers=[...]`），以及一个决定内存请求发送到哪个控制器的通道映射器（`channel_mapper=...`）。`clock_ratio=3` 表示每 `y` 个前端时钟周期（即前端侧的 `clock_ratio=y`），内存系统将被触发 `3` 次。目前，`GenericDRAM` 要求其所有内存控制器具有相同的频率。

### 3.4 预期输出

示例在模拟完成后从 `sim.stats` 打印几个关键数字：

```text
Controller cycles:     81302
Avg read latency:      45.2 cycles
Read requests:         6
Write requests:        0
Row hits:              3
Row misses:            3
Row conflicts:         0
```

具体数字取决于工作负载和配置。

`sim.stats` 是一个具有两个顶层键的嵌套 Python 字典：`"frontend"` 和 `"memory_system"`。最有用的计数器位于 `stats["memory_system"]["controller"]` 下：

| 统计项 | 含义 |
|--------|---------|
| `cycles` | 控制器周期 |
| `avg_read_latency` | 以控制器周期为单位的平均读取延迟 |
| `num_read_reqs` | 已接受的读取请求 |
| `num_write_reqs` | 已接受的写入请求 |
| `row_hits` | 总行命中次数 |
| `row_misses` | 总行缺失次数 |
| `row_conflicts` | 总行冲突次数 |
| `read_queue_len_avg` | 平均读取队列占用 |
| `write_queue_len_avg` | 平均写入队列占用 |
| `total_num_read_requests` | 内存系统接受的总读取数（上一级，位于 `stats["memory_system"]` 下） |
| `total_num_write_requests` | 内存系统接受的总写入数 |

对于单通道，`stats["memory_system"]["controller"]` 是一个字典。对于多通道，它变为一个字典列表（每个通道一个），每个字典都有一个 `"id"` 字段（例如 `"Channel 0"`）。

## 4. 编写配置

### 4.1 主要组件

Ramulator Python 包将主要组件公开为一组命名空间：

- `ramulator.frontend`
- `ramulator.dram`
- `ramulator.controller`
- `ramulator.scheduler`
- `ramulator.refresh_manager`
- `ramulator.row_policy`
- `ramulator.addr_mapper`
- `ramulator.channel_mapper`
- `ramulator.translation`
- `ramulator.controller_plugin`
- `ramulator.memory_system`

### 4.2 常见首次变更

#### 切换到另一种 DRAM 标准

您可以通过替换 DRAM 对象以及在需要时替换控制器类来将 DDR4 替换为另一种标准。

示例：

```python
dram = ramulator.dram.DDR5(org_preset="DDR5_8Gb_x8", timing_preset="DDR5_4800AN")
ctrl = ramulator.controller.GenericDDR(dram=dram, ...)
```

```python
dram = ramulator.dram.LPDDR5(org_preset="LPDDR5_8Gb_x16", timing_preset="LPDDR5_5500")
ctrl = ramulator.controller.LPDDR5(dram=dram,...)
```

```python
dram = ramulator.dram.HBM2(org_preset="HBM2_2Gb", timing_preset="HBM2_2000Mbps")
ctrl = ramulator.controller.HBM12(dram=dram,...)
```

使用与您要建模的标准匹配的控制器。DDR3、DDR4、DDR5 和 GDDR6 使用 `GenericDDR`。LPDDR5 使用 `LPDDR5`。HBM1 和 HBM2 使用 `HBM12`；HBM3 和 HBM4 使用 `HBM34`。

#### 更改 Rank 数量或其他 DRAM 覆盖

DRAM 对象接受预设名称加覆盖：

```python
dram = ramulator.dram.DDR4(
    org_preset="DDR4_8Gb_x8",
    timing_preset="DDR4_2400R",
    rank=2, # 这将 org_preset 中的 1 rank 设置覆盖为 2 ranks
    # 在此处添加更多组织和时序覆盖
)
```

覆盖会根据 DRAM 规范进行验证。覆盖不存在的参数会引发错误。

#### 添加更多通道

一个控制器对应一个通道。

```python
ctrl = ramulator.controller.GenericDDR(
    dram=ramulator.dram.DDR4(org_preset="DDR4_8Gb_x8", timing_preset="DDR4_2400R", rank=2),
    scheduler=ramulator.scheduler.FRFCFS(),
    refresh_manager=ramulator.refresh_manager.AllBank(),
    row_policy=ramulator.row_policy.Open(),
    addr_mapper=ramulator.addr_mapper.RoBaRaCoCh(),
)

num_channels = 2
mem = ramulator.memory_system.GenericDRAM(
    clock_ratio=3,
    controllers=[ctrl] * num_channels, 
    channel_mapper=ramulator.channel_mapper.CacheLineInterleave(),
)
```

此处有一个重要细节：`CacheLineInterleave` 目前要求通道数为 2 的幂。

#### 选择不同的前端

内置前端服务于不同目的：

- `SimpleO3`
  使用简单核心模型和 LLC 进行内存轨迹驱动研究的首选。内存轨迹包括 1) 内存请求，以及 2) 连续内存请求之间的间隔（即非内存指令的数量）。请查看 `src/ramulator/frontend/impl/processor/simpleO3/simpleO3.cpp` 了解轨迹格式。
- `LoadStoreTrace`
  重放具有 `LD` 和 `ST` 记录的平面地址轨迹。不建模内存请求之间的间隔（即每个周期都将内存请求发送到内存系统）。
- `ReadWriteTrace`
  重放具有 `R` 和 `W` 记录的轨迹。类似于 `LoadStoreTrace`，但期望地址向量而非平面地址。适用于调试/测试。
- `LatencyThroughputTrace`
  验证工作流使用的合成负载生成器，生成两种内存请求：1) 用于探测内存访问延迟的随机访问指针追踪类请求，以及 2) 用于对内存系统产生负载的流式访问请求（可通过连续流式请求之间的间隔进行配置）。

### 4.5 `sim.stats` 和 `sim.stats_yaml`

`sim.stats` 将所有模拟统计信息作为嵌套 Python 字典快照返回。这是访问结果的最简单方式，使您能够在单个 Python 脚本中简化实验工作流（配置、参数扫描、结果分析）。`sim.stats_yaml` 以 YAML 格式快照返回相同数据，以便您将结果保存到磁盘。

读取 `sim.stats` 或 `sim.stats_yaml` 会更新派生统计信息（如平均延迟和吞吐量），但不会终结模拟或刷新最终插件输出。当您需要终结副作用（如命令计数器 CSV 或轨迹记录器输出）时，调用 `sim.finalize()`。`sim.finalize()` 是幂等的。

## 5. 验证与回归测试

Ramulator 在 `tests/` 下包含四个实用的测试层。

### 5.1 四个测试套件的区别

- `smoke`
  最快的端到端健全性检查。验证每个支持的标准能否构建真实配置并运行而不崩溃。
- `latency_throughput`
  围绕生成的前端流量构建的建模保真度套件。检查空载延迟和持续带宽是否与 DRAM 定义所隐含的理论行为一致。
- `device_timings`
  短序列器件级正确性套件。逐个命令检查 DRAM 命令合法性、前提条件和时序门控行为。
- `controller_scheduling`
  短序列控制器级正确性套件。检查发出的命令序列、行命中/缺失/冲突行为以及优先级缓冲区调度约定。

这些套件是互补的：

- `smoke` 回答"它能运行吗？"
- `latency_throughput` 回答"高层性能形态看起来对吗？"
- `device_timings` 回答"DRAM 时序和前提条件行为看起来对吗？"
- `controller_scheduling` 回答"控制器以正确的方式调度和发出命令吗？"

### 5.2 Smoke 测试

Smoke 测试确保每个支持的标准能够端到端运行而不崩溃。

```bash
PYTHONPATH=python pytest tests/smoke -q
```

这是构建或本地代码更改后最快的信心检查。

### 5.3 快速延迟-吞吐量

快速延迟-吞吐量是常规开发中使用的主要建模保真度检查。应在几分钟内完成。

```bash
PYTHONPATH=python pytest tests/latency_throughput/test_fast.py -v -s
```

它对每个 DRAM 标准做三件事：

- 运行无刷新延迟-吞吐量扫描
- 根据时序公式检查空载延迟
- 检查测量的流式吞吐量与理论峰值

它还将带注释的绘图写入：

```text
tests/latency_throughput/plots/fast/
```

如果您只想要一个标准：

```bash
PYTHONPATH=python pytest tests/latency_throughput/test_fast.py -v -s -k DDR4
```

### 5.4 完整延迟-吞吐量

完整延迟-吞吐量是更长的启用刷新扫描。它使用与快速套件相同的延迟-吞吐量曲线形状，但使用完整配置文件的请求数运行每个点：

```bash
PYTHONPATH=python pytest tests/latency_throughput/test_full.py -v -s
```

它将绘图写入：

```text
tests/latency_throughput/plots/full/
```

### 5.5 器件时序

器件时序是短 DRAM 命令序列的器件级正确性套件。

```bash
PYTHONPATH=python pytest tests/device_timings -q
```

它专注于对于吞吐量扫描来说太小、太具体的案例：

- DRAM 前提条件，如 `RD` 需要 `ACT`
- 时序门控，如 `nRCD`、`nRTP` 和 `nRP`

当您更改命令语义或器件时序强制时，使用它。

`DeviceUnderTest` 提供三种互补操作：

- `probe(command, addr_vec, clk)`
  只读查询。它询问"如果我想在这个周期发出这个命令，器件会说什么？"它不改变器件状态。
- `issue(command, addr_vec, clk)`
  状态变更操作。它实际将命令发出到器件并更新时序/状态。如果命令在该周期不完全合法，则抛出异常。
- `assert_earliest_ready_at(command, addr_vec, clk)`
  只读断言。它在 `clk - 1` 探测并断言命令*未*就绪，然后在 `clk` 探测并断言它*是*就绪的。使用此功能验证时序门控是紧的：尽可能早地合法，不更早。失败消息包含 `preq` 和 `timing_OK`，因此您可以一目了然地判断回归是与状态相关还是与时序相关。

`probe()` 返回五条信息：

- `preq`
  从器件角度看应该接下来发生的命令。如果您在关闭的 Bank 上探测 `RD`，这将是 `ACT`。
- `timing_OK`
  时序本身是否允许在该周期探测命令。
- `ready`
  命令现在是否完全可发出。这是功能前提条件和时序的组合：
  `ready = (preq == command) and timing_OK`
- `row_hit`
  目标 Bank 是否已经打开到请求的行。
- `row_open`
  目标 Bank 中是否已经有某行被打开。

这种区分很重要，因为命令可能因两种不同的原因失败：

1. Bank 状态错误，因此必须先发生不同的前提命令。
2. Bank 状态正确，但时序仍然阻塞命令。

例如，关闭 Bank 的读取未就绪，但仍可以是时序 OK 的：

```python
# 关闭 Bank 的读取在打开行之前被功能阻塞。
closed = dut.probe("RD", a, clk=0)
# 未发出 ACT，前提条件是 ACT。
assert closed.preq == "ACT"
# 这里我们只检查时序约束。时序在这里是 OK 的，因为尚未发出 ACT！
assert closed.timing_OK is True
# ready 表示命令现在完全可发出：正确的前提条件和 timing_OK。
assert closed.ready is False
```

这里，时序不是问题。缺少的前提条件才是。

发出 `ACT` 后，前提条件变为 `RD`，但访问仍然被阻塞直到 `nRCD` 到期：

```python
dut.issue("ACT", a, clk=0)

early = dut.probe("RD", a, clk=dut.timings["nRCD"] - 1)
assert early.preq == "RD"
assert early.timing_OK is False
assert early.ready is False
```

现在状态正确，但时序不对。

恰好在 `nRCD` 时，相同的探测变为完全就绪：

```python
ontime = dut.probe("RD", a, clk=dut.timings["nRCD"])
assert ontime.preq == "RD"
assert ontime.timing_OK is True
assert ontime.ready is True
```

这就是 `issue()` 变为有效的点：

```python
dut.issue("RD", a, clk=dut.timings["nRCD"])
```

`issue()` 是故意严格的。它不会尝试为您修复序列。如果您在前提条件不同的命令上调用它，或在仍然被时序阻塞的命令上调用它，它会引发错误。一个好的测试模式是：

1. 使用 `probe()` 了解器件接下来期望什么。
2. 对 `preq`、`timing_OK` 和 `ready` 进行断言。
3. 仅在您期望 `ready` 为 `True` 时调用 `issue()`。

一个最小的 `DeviceUnderTest` 示例如下：

```python
import ramulator
import tests.device_timings.harness as device_timings

dram = ramulator.dram.DDR4(org_preset="DDR4_8Gb_x8", timing_preset="DDR4_2400R", rank=1)
dut = device_timings.DeviceUnderTest(dram)
a = dut.addr_vec(Rank=0, BankGroup=0, Bank=0, Row=12, Column=0)

assert dut.probe("RD", a, clk=0).preq == "ACT"
dut.issue("ACT", a, clk=0)
assert dut.probe("RD", a, clk=dut.timings["nRCD"]).ready is True
```

规范的完整器件示例位于：

- `tests/device_timings/example.py`

### 5.6 控制器调度

控制器调度是短请求和维护序列的控制器级正确性套件。

```bash
PYTHONPATH=python pytest tests/controller_scheduling -q
```

它专注于对于吞吐量扫描来说太小、太具体的案例：

- 行命中、缺失和冲突的控制器发出命令序列
- 调度偏好，如 FRFCFS 选择就绪请求
- 优先级/内部命令周围的控制器约定

当您更改行策略行为、控制器调度或命令发出逻辑时，使用它。

一个最小的 `ControllerUnderTest` 示例如下：

```python
import ramulator
import tests.controller_scheduling.harness as cs

dram = ramulator.dram.DDR4(org_preset="DDR4_8Gb_x8", timing_preset="DDR4_2400R", rank=1)
dut = cs.ControllerUnderTest.make_generic_ddr(dram)

row0 = dut.addr_vec(Rank=0, BankGroup=0, Bank=0, Row=0, Column=0)
row1 = dut.addr_vec(Rank=0, BankGroup=0, Bank=0, Row=1, Column=0)

dut.send_request("Read", row0)
dut.send_request("Read", row1)
history = dut.run_until_idle(max_ticks=128)
dut.assert_commands(["ACT", "RD", "PREpb", "ACT", "RD"], history=history)
```

规范的完整控制器示例位于：

- `tests/controller_scheduling/examples/test_controller_example.py`

### 5.8 何时使用哪种测试

- 在干净构建或小的代码更改后使用 smoke 测试
- 当您更改时序行为、控制器逻辑或 DRAM 定义时使用快速延迟-吞吐量
- 当您更改命令合法性或时序强制时使用器件时序
- 当您更改控制器命令序列或行策略行为时使用控制器调度
- 当您想要更全面的健全性检查时使用完整延迟-吞吐量

## 6. gem5 集成

Ramulator2 作为即插即用内存系统集成到 gem5 中。您可以在相同的 gem5 Python 配置脚本中配置 Ramulator2。集成使用 gem5 的 stdlib Board API，因此它适用于 `SimpleBoard` 和其他 stdlib 板。

已在 gem5 v25.1 上测试。

### 6.1 先决条件

- 从源代码构建的 gem5（推荐稳定分支）
- 使用 `libramulator.so` 构建的 Ramulator2（默认构建）
- 可用的 Ramulator2 Python 包（`pip install -e .` 或 `PYTHONPATH`）

### 6.2 安装封装

将封装文件复制到您的 gem5 源代码树中：

```bash
cp -r resources/gem5_wrappers/ <gem5>/src/mem/ramulator2/
```

这会创建 gem5 SimObject 声明、C++ 封装源文件和构建文件：

| 文件 | 用途 |
|------|---------|
| `Ramulator2.py` | 单端口 gem5 SimObject 声明 |
| `Ramulator2VectorPorts.py` | 向量端口 gem5 SimObject 声明 |
| `ramulator2_base.hh`, `ramulator2_base.cc` | 共享的 Ramulator 生命周期、触发、队列和统计实现 |
| `ramulator2.hh`, `ramulator2.cc` | 单端口封装实现 |
| `ramulator2_vector_ports.hh`, `ramulator2_vector_ports.cc` | 向量端口封装实现 |
| `SConscript` | 构建配置 |

编辑 `SConscript` 并将 `RAMULATOR2_HOME` 设置为您的 ramulator2 目录：

```python
# 在 <gem5>/src/mem/ramulator2/SConscript 中
RAMULATOR2_HOME = '/path/to/ramulator2'  # ← 更改此项
```

然后重新构建 gem5：

```bash
cd <gem5>
scons build/X86/gem5.opt -j$(nproc) --ignore-style
```

### 6.3 编写 gem5 配置脚本

完整的工作示例位于 `examples/gem5_se_ramulator_hello_world.py`：

```python
import sys

# 替换为实际路径
sys.path.insert(0, "/path/to/ramulator2/python")

import ramulator
from gem5.components.boards.simple_board import SimpleBoard
from gem5.components.cachehierarchies.classic.no_cache import NoCache
from gem5.components.processors.cpu_types import CPUTypes
from gem5.components.processors.simple_processor import SimpleProcessor
from gem5.isas import ISA
from gem5.resources.resource import BinaryResource
from gem5.simulate.simulator import Simulator

# ── Ramulator2 内存配置 ──

ddr4 = ramulator.dram.DDR4(org_preset="DDR4_8Gb_x8", timing_preset="DDR4_2400R", rank=1)
ctrl = ramulator.controller.GenericDDR(
    dram=ddr4,
    scheduler=ramulator.scheduler.FRFCFS(),
    refresh_manager=ramulator.refresh_manager.AllBank(),
    row_policy=ramulator.row_policy.Open(),
    addr_mapper=ramulator.addr_mapper.RoBaRaCoCh(),
)
mem_sys = ramulator.memory_system.GenericDRAM(
    clock_ratio=3,
    controllers=[ctrl],
    channel_mapper=ramulator.channel_mapper.CacheLineInterleave(),
)

memory = ramulator.gem5.Memory(mem_sys, size="4GiB")

# ── gem5 系统设置 ──

processor = SimpleProcessor(cpu_type=CPUTypes.TIMING, isa=ISA.X86, num_cores=1)
board = SimpleBoard(
    clk_freq="3GHz",
    processor=processor,
    memory=memory,
    cache_hierarchy=NoCache(),
)

board.set_se_binary_workload(binary=BinaryResource(local_path="/path/to/binary"))

simulator = Simulator(board=board)
simulator.run()
```

### 6.4 运行

```bash
cd <gem5>
build/X86/gem5.opt configs/your_config.py
```

`SConscript` 设置 `RPATH`，以便自动找到 `libramulator.so`。如果您在构建后移动了库，则改为设置 `LD_LIBRARY_PATH`：

```bash
LD_LIBRARY_PATH=/path/to/ramulator2 build/X86/gem5.opt configs/your_config.py
```

对于请求级调试追踪，添加 `--debug-flags=Ramulator2`。

Ramulator 的内部统计信息（行命中、队列长度、读取延迟等）与 gem5 自己的 `stats.txt` 一起写入 `m5out/ramulator_stats.yaml`。

### 6.5 PyTrafficGen 延迟-吞吐量测试

`tests/latency_throughput/gem5_pytrafficgen/` 测量由 gem5 的 PyTrafficGen 驱动的多通道 Ramulator2 内存系统的延迟-吞吐量曲线。

运行完整的 16 通道 DDR5 延迟-吞吐量扫描：

```bash
GEM5_BIN=<gem5>/build/X86/gem5.opt \
python3 tests/latency_throughput/gem5_pytrafficgen/run.py --dram DDR5 --channels 16
```

或使用特定配置点：

```bash
GEM5_BIN=<gem5>/build/X86/gem5.opt \
python3 tests/latency_throughput/gem5_pytrafficgen/run.py \
  --dram DDR5 --channels 16 \
  --traffic stream --read-ratios 100 --intensities 0.5 0.9 1.0
```

结果和日志默认存储在 `/tmp/gem5-pytrafficgen/<DRAM>_<MAPPER>_ch<N>/`（可用 `--out-dir` 覆盖）

使用以下方式绘制结果，例如：

```bash
python3 tests/latency_throughput/gem5_pytrafficgen/plot.py \
  --csv /tmp/gem5-pytrafficgen/DDR5_MOP4CLXOR_ch16/results.csv
```

## 7. 将 Ramulator 用作纯 C++ 库

如果您想在模拟器中使用 Ramulator 而不引入 Python 绑定，您仍然可以将 Ramulator 用作纯 C++ 库。

### 7.1 典型工作流

将 Ramulator 嵌入另一个模拟器时，您使用 `External` 前端。您的模拟器向 Ramulator 发送内存请求并触发内存系统；Ramulator 处理调度、时序和状态跟踪。

典型流程是：

1. 构建 Ramulator（`libramulator.so`）
2. 编写使用 `ramulator.frontend.External` 的 Python 配置并将其导出为 YAML
3. 从 C++ 加载该 YAML
4. 通过 `receive_external_requests()` 发送内存请求
5. 以 DRAM 时钟频率触发内存系统

创建一个使用 `External` 前端的 Python 配置脚本，然后导出它：

```python
import ramulator

frontend = ramulator.frontend.External(clock_ratio=1)

dram = ramulator.dram.DDR4(org_preset="DDR4_8Gb_x8", timing_preset="DDR4_2400R", rank=1)
ctrl = ramulator.controller.GenericDDR(
    dram=dram,
    scheduler=ramulator.scheduler.FRFCFS(),
    refresh_manager=ramulator.refresh_manager.AllBank(),
    row_policy=ramulator.row_policy.Open(),
    addr_mapper=ramulator.addr_mapper.RoBaRaCoCh(),
)
mem = ramulator.memory_system.GenericDRAM(
    clock_ratio=3,
    controllers=[ctrl],
    channel_mapper=ramulator.channel_mapper.CacheLineInterleave(),
)

sim = ramulator.Simulation(frontend, mem)
```

然后运行：

```bash
python3 -m ramulator export external_config.py -o config.yaml
```

生成的 YAML 从 Ramulator 的角度来看与导出它的 Python 配置完全等效。它的可读性不如 Python 配置，因为它旨在供 Ramulator 解析。不期望您手动编辑 YAML 文件。

### 7.2 最小 C++ 集成骨架

```cpp
#include <ramulator/base/config.h>
#include <ramulator/base/factory.h>
#include <ramulator/base/request.h>
#include <ramulator/frontend/i_frontend.h>
#include <ramulator/memory_system/i_memory_system.h>

// 1. 加载配置并创建组件
auto config = Ramulator::Config::parse_config_file("config.yaml");

auto* frontend = Ramulator::Factory::create_frontend(config);
auto* memory_system = Ramulator::Factory::create_memory_system(config);

frontend->connect_memory_system(memory_system);
memory_system->connect_frontend(frontend);

// 2. 发送读取请求
//    req_type_id: Request::Type::Read (0) 或 Request::Type::Write (1)
//    addr:        字节地址
//    source_id:   标识哪个核心或源（单核为 0）
//    callback:    请求完成时调用
//    size_bytes:  请求大小（字节）
bool accepted = frontend->receive_external_requests(
    Ramulator::Request::Type::Read,   // 读取请求
    0x1000,                           // 地址
    0,                                // 源 ID
    [](Ramulator::Request& req) {
      // 请求完成 — req.depart 包含完成周期
    },
    64                                // size_bytes
);

// 如果 accepted 为 false，则内存系统队列已满。
// 在触发后的下一个周期重试。

// 3. 每个 DRAM 周期触发内存系统
memory_system->tick();

// 4. 完成时，终结以刷新统计信息
frontend->finalize();
memory_system->finalize();
```

`External` 前端的 `tick()` 是无操作 — 您的模拟器控制何时以及如何注入请求。您只需要触发内存系统。

### 7.3 集成注意事项

- 导出的配置完全展开。C++ 端期望解析后的值，而非符号化 Python 预设。
- `libramulator.so` 是您链接的库。
- 当控制器的请求队列已满时，`receive_external_requests()` 返回 `false`。调用方必须在后续周期重试。
- 必须为每个外部请求设置 `size_bytes`，且不得超过 `memory_system->get_tx_bytes()` 返回的 DRAM 事务大小。
- 请求类型 ID 为 `0`（读取）和 `1`（写入），对应 `Request::Type::Read` 和 `Request::Type::Write`。
- 对于完整的工作集成，请参见 `resources/gem5_wrappers/` 中的 gem5 封装或上面的 gem5 集成部分。

## 8. 扩展 Ramulator

本节涵盖大多数贡献者实际接触的扩展点。

### 8.1 添加新实现

最常见的变更是添加现有接口的新实现，例如调度器。

创建单个 `.cpp` 文件：

```cpp
#include "controller/controller_base.h"
#include "controller/scheduler/i_scheduler.h"

namespace Ramulator {

// 同时继承其接口和公共 Implementation 基类
class FooBarScheduler : public IScheduler, public Implementation {
  // 向接口注册实现类并指定名称 ("FooBar")
  // Python 封装是*自动*生成的
  RAMULATOR_REGISTER_IMPLEMENTATION(IScheduler, FooBarScheduler, "FooBar")

  ControllerBase* m_ctrl = nullptr;
  int m_weight = 0;
  size_t s_decisions = 0;

  void init() override {
    // 使用配置中的参数初始化组件
    RAMULATOR_PARSE_PARAM(m_weight, int, "weight").default_val(4);

    // 如果需要访问父组件
    m_ctrl = cast_parent<ControllerBase>();

    // 将变量注册为统计信息以自动打印
    m_stats.add("foobar_decisions", s_decisions);
  }

  void setup(IFrontEnd* frontend, IMemorySystem* memory_system) override {
    // setup() 在所有组件初始化后调用
    // 您可以在这里解析依赖于其他组件的配置
    // ...
  }

  ReqBuffer::iterator get_best_request(
      ReqBuffer& buffer,
      RequestFilterRef filter) override {
        // 通过覆盖接口虚函数实现组件的实际行为和逻辑
  }
};

}  // namespace Ramulator
```

然后将该文件添加到相关的 `CMakeLists.txt`，重新构建，并从 Python 中使用它：

```python
ctrl = ramulator.controller.GenericDDR(
    dram=dram,
    scheduler=ramulator.scheduler.FooBar(weight=8), # 新调度器！
    refresh_manager=ramulator.refresh_manager.AllBank(),
    row_policy=ramulator.row_policy.Open(),
    addr_mapper=ramulator.addr_mapper.RoBaRaCoCh(),
)
```

### 8.2 控制器插件

控制器插件是附加到控制器的可选观察者组件。

目前，插件生命周期包括：

- `pre_schedule()`
  在候选选择之前运行
- `on_issue(const Request&)`
  在命令发出后运行
- `post_schedule()`
  在控制器 tick 结束时运行

内置插件包括：

- `CommandCounter`
  统计选中的 DRAM 命令并将 `command, count` 行写入 CSV 文件
- `CmdTraceRecorder`
  将每个发出的命令记录到每通道轨迹文件，如 `trace.csv.ch0`
- `BinTraceRecorder`
  为[轨迹可视化工具](#10-轨迹可视化工具)记录每通道的二进制 `.ram2bin` 轨迹
- `LiveTraceStreamer`
  通过 HTTP 实时将命令流式传输到[轨迹可视化工具](#10-轨迹可视化工具)
- `IssuedCommandValidationHook`
  将每个发出的 DRAM 命令转发到 Python 中的控制器调度测试套件

示例：

```python
ctrl = ramulator.controller.GenericDDR(
    dram=dram,
    scheduler=ramulator.scheduler.FRFCFS(),
    refresh_manager=ramulator.refresh_manager.AllBank(),
    row_policy=ramulator.row_policy.Open(),
    addr_mapper=ramulator.addr_mapper.RoBaRaCoCh(),
    controller_plugins=[
        ramulator.controller_plugin.CommandCounter(
            commands_to_count=["ACT", "PREpb", "RD", "WR", "REFab"],
            path="cmd_counts.csv",
        ),
    ],
)
```

### 8.3 扩展 DRAM 标准

DRAM 标准在 Python 脚本中定义，描述 DRAM 标准的规范。Ramulator 在构建过程中自动生成适合建模方法的 C++ 代码。用户可以享受 Python 的可读性、灵活性和可扩展性，避免可读性低得多的底层代码。

例如，如果您想通过添加新 DRAM 命令（例如 `FOO`）来创建现有 DRAM 标准的变体，请遵循以下两个简单步骤：

1. 在 `src/ramulator/dram/commands/` 下添加 `FOO` 命令的实现。

2. 通过简单地继承基础 DRAM 标准并仅指定变更来创建变体 DRAM 标准：

```python
import math

from ramulator.dram.ddr3 import DDR3
from ramulator.dram.spec import TimingConstraint


class DDR3Foo(DDR3):
    name = "DDR3Foo"

    # 添加新命令
    commands = DDR3.commands + ["FOO"]

    # 从新命令添加新时序约束
    timing_params = DDR3.timing_params + ["nFOO"]
    timing_constraints = DDR3.timing_constraints + [
        TimingConstraint(level="Bank", preceding=["FOO"], following=["ACT"], latency="nFOO"),
        TimingConstraint(level="Bank", preceding=["ACT"], following=["FOO"], latency="nRC"),
    ]
```

### 8.4 创建新 DRAM 标准

新 DRAM 标准是 `python/ramulator/dram/` 下的 `DRAMStandard` 子类。

您定义：

- `name`
- `levels`
- `commands`
- `states`
- `timing_params`
- `supported_requests`
- `timing_constraints`
- `org_presets`
- `timing_presets`
- `resolve_secondary_timings()`

代码生成导入 `python/ramulator/dram/` 下的模块，发现这些类，并在 `src/ramulator/dram/impl/` 中生成相应的 C++ 实现。

### 8.5 将新标准添加到延迟-吞吐量测试

DRAM 定义存在后，在 `tests/latency_throughput/testcases/` 中添加测试用例文件。

当前的延迟-吞吐量流程期望如下配置形状：

```python
config = {
    "name": "MyStandard",
    "dram_class": "MyStandard",
    "org_preset": "MyOrgPreset",
    "timing_preset": "MyTimingPreset",
    "controller_class": "GenericDDR",
    "stream_cls": 8,
    "nop_counters": (1, 10, 100, 1000),
}
```

确切的控制器类取决于标准。例如，DDR3/DDR4/DDR5/GDDR6 使用 `GenericDDR`，LPDDR5 使用 `LPDDR5`，HBM1/HBM2 使用 `HBM12`，HBM3/HBM4 使用 `HBM34`。

然后运行：

```bash
PYTHONPATH=python pytest tests/smoke -v -k MyStandard
PYTHONPATH=python pytest tests/latency_throughput/test_fast.py -v -s -k MyStandard
PYTHONPATH=python pytest tests/latency_throughput/test_full.py -v -s -k MyStandard
```

## 9. Ramulator 内部工作原理

本节描述公开配置和扩展 API 背后的实现约定。如果您只想运行实验，可以更早停止，需要更深入模型时再回来。

### 9.1 接口与组件框架

Ramulator 使用接口和实现模式。接口对模拟系统中的组件类型进行建模。接口类定义该组件向系统中其他组件公开的高级协议和约定（即其他组件应如何与此组件交互），通过虚函数实现。接口类以 `I` 开头，头文件以 `i_` 为前缀。

示例：

- `IFrontEnd`
- `IMemorySystem`
- `IController`
- `IScheduler`
- `IRowPolicy`
- `IRefreshManager`
- `IAddrMapper`
- `IChannelMapper`
- `IControllerPlugin`
- `ITranslation`

实现是组件类型接口建模的具体实例。实现类通过覆盖接口类的虚函数来定义其所建模组件的具体行为。在 Ramulator2 中，所有实现类必须同时继承其接口类和一个提供基本共享工具和样板的公共 `Implementation` 类。

Ramulator2 实现了一个自注册组件工厂，以便可以从配置自动创建组件层次结构。只要用户通过以下单行宏使其自定义接口和实现可发现，就无需担心处理工厂样板：

```cpp
RAMULATOR_REGISTER_INTERFACE(IfceClassName, "ifce_name")
RAMULATOR_REGISTER_IMPLEMENTATION(IfceClassName, ImplClassName, "ImplName")
```

工厂使用这些注册从配置数据自动发现和创建组件（例如，上述示例在 Python 侧将变为 `ramulator.ifce_name.ImplName`）。没有手动维护的注册文件。

类似地，通过使用提供的宏来解析参数和创建子组件，Python 发现它们的样板代码也会自动生成。无需手动维护 C++ 到 Python 的绑定。
```cpp
RAMULATOR_PARSE_PARAM(parsed_variable, type_t, "param_name")
RAMULATOR_CREATE_CHILD(IfceClassName, "ifce_name")
```

### 9.2 配置流程

在 Python 模式下，配置路径是：

1. 您创建 Python 组件对象。
2. 每个对象使用 `to_config()` 序列化自身。
3. Python 绑定将嵌套字典转换为 `ConfigNode`。
4. C++ 工厂创建顶层前端和内存系统对象。
5. 子组件在 `init()` 期间递归创建。
6. 模拟循环根据它们的时钟比率推进前端和内存系统。

在 C++ 库模式下，唯一的区别是您从导出的 YAML 文件加载 `ConfigNode` 树，而不是从 Python 对象开始。

### 9.3 DRAM 器件模型与分层状态机

控制器拥有一个 `DRAMDevice`，该器件是 Ramulator 将 DRAM 标准描述转换为实时协议模型的地方。最简单的理解方式是，器件同时保持同一通道的两个视图：

- 用于时序的节点层次结构
- 用于命令语义的平面 Bank 导向视图

DRAM 时序规则在不同作用域编写。一些存在于 Bank 级别，一些存在于 Bank Group 或 Rank 级别，一些存在于通道或伪通道级别。功能状态通常从特定 Bank 的角度回答。Ramulator 使用层次结构处理作用域时序规则，使用平面 Bank 视图处理直接 Bank 本地命令语义。

器件不保持自己的自由运行时钟。控制器拥有 `m_clk` 并将当前周期传递给每个器件查询和每个发出的命令。这就是 `DRAMDevice::check_timing`、`DRAMDevice::get_preq_command` 和 `DRAMDevice::issue_command` 都将 `clk` 作为参数的原因。

#### 9.3.1 模型来源

模型始于 Python，而非 C++。每个 DRAM 标准是 `python/ramulator/dram/` 下的 `DRAMStandard` 子类。该类定义：

- 通过 `levels` 定义的层次结构
- 通过 `commands` 定义的合法命令集
- 通过 `states` 定义的状态名称
- 通过 `timing_params` 定义的时序词汇
- 通过 `supported_requests` 定义的外部请求到 DRAM 命令映射
- 通过 `TimingConstraint` 定义的时序规则
- 通过 `command_cycles`、`tick_multiplier`、`row_commands` 和 `column_commands` 定义的可选总线时序行为

`to_config()` 是 Python 定义变为运行时数据的地方。它所做的工作比其名称暗示的更多：

- 解析选定的组织和时序预设
- 应用用户覆盖，如 `rank=2`
- 在 `resolve_secondary_timings()` 中计算派生时序
- 评估时序表达式，如 `nCL + nBL + 2 - nCWL`
- 当 `tick_multiplier` 大于 1 时将所有内容缩放为模拟周期
- 将每个 `TimingConstraint` 展开为整数索引条目
- 使用 `command_cycles` 为多周期命令调整延迟
- 为需要它的标准自动生成总线占用约束

当配置到达 C++ 时，符号化 DRAM 描述已经完全解析。C++ `DRAMSpec` 不会即时重新推导 JEDEC 表。它存储解析后的名称、时序值、时序约束、命令元数据、Bank 目标模式和命令行为函数指针。

运行时代码使用解析后的整数 ID、时序值和命令元数据，而非在每个周期重新评估符号化 Python 定义。

#### 9.3.2 运行时实例化内容

当控制器初始化其器件时，`DRAMDevice::init()` 做三件事：

1. 获取解析后 `DRAMSpec` 的所有权
2. 构建根 `DRAMNode`
3. 收集所有 Bank 节点的平面列表

节点树表示一个通道的结构层次。对于 DDR4，该层次结构实际上是：

```text
Channel -> Rank -> BankGroup -> Bank
```

对于 HBM3 是：

```text
Channel -> PseudoChannel -> BankGroup -> Bank
```

树在 `Row` 级别之前停止。Ramulator 不会为每个物理行实例化一个节点。相反，它在拥有这些行的类 Bank 节点内部延迟跟踪行状态。

每个 `DRAMNode` 存储四种状态：

- `m_state`
  该节点的粗略协议状态，如 `Closed`、`Opened` 或 LPDDR5 的 `Activating`
- `m_cmd_ready_clk`
  对于每个命令，该命令下次可能在此节点发出的最早周期
- `m_cmd_history`
  每个命令的最近发出时间，大小足以建模该级别看到的最大滚动窗口
- `m_row_state`
  该类 Bank 节点当前打开行的映射

最后一个字段是 Ramulator 能够建模大型器件而无需创建数百万行对象的原因。行仅在被打开时才会出现在状态映射中。关闭的 Bank 具有空的 `m_row_state`。

平面 Bank 数组 `m_bank_nodes` 只是同一树的不同视图。它让控制器无需每次遍历层次结构即可询问 Bank 本地问题。

#### 9.3.3 查询 DRAMSpec

组件查询 `DRAMSpec`（定义于 `src/ramulator/dram/dram_spec.h`）以获取级别 ID、命令 ID 和时序值。API 有两个系列：

**存在性检查**返回 `bool` — 在操作之前使用这些来测试可选功能是否存在：

| 方法 | 返回 |
|--------|---------|
| `has_level("X")` / `has_command("X")` / `has_state("X")` / `has_timing("X")` | `bool` |

**值获取器**返回整数 ID 或解析后的值。如果名称在 DRAM 标准中不存在，它们会立即抛出 `std::runtime_error` — 没有静默的 `-1` 返回：

| 方法 | 返回 |
|--------|---------|
| `get_level_id("X")` / `get_command_id("X")` / `get_state_id("X")` | `int` — 给定名称的整数 ID |
| `get_timing_value("X")` | `int` — 解析后的时序值（非索引） |
| `get_level_size("X")` | `int` — 命名级别的实例数量（来自 `organization.level_sizes`） |

**最佳实践：在初始化时缓存查找。** API 不阻止在运行时调用这些，但字符串键映射查找在热路径上是不必要的开销。推荐模式是在 `init()` 中调用它们一次并将结果存储在成员变量中：

```cpp
void init() override {
  const auto& spec = *m_ctrl->m_device.m_spec;

  // 必需查找 — 如果 DRAM 标准缺少这些则立即抛出
  m_cmd_act  = spec.get_command_id("ACT");
  m_cmd_rd   = spec.get_command_id("RD");
  m_nCL      = spec.get_timing_value("nCL");
  m_bank_lvl = spec.get_level_id("Bank");

  // 可选功能 — 先检查，显式分支
  m_cmd_rda = spec.has_command("RDA") ? spec.get_command_id("RDA") : -1;
  m_has_ap  = (m_cmd_rda != -1);
}
```

运行时，使用缓存的成员变量（`m_cmd_act`、`m_nCL` 等）和直接数组访问（`spec.command_meta[cmd]`、`spec.bank_targets[cmd]`）。

#### 9.3.4 时序侧：分层合法性检查

时序模型由 `TimingConstraint` 驱动。每个约束说明：

- 它适用的层次级别
- 哪些前置命令创建约束
- 哪些后续命令被它阻塞
- 延迟多长
- 是否需要滚动历史窗口
- 效果是否适用于同级节点而非仅确切寻址路径

在配置时，这些对象成为 `DRAMSpec::timing_cons` 中的 `TimingConsEntry` 记录。从那时起，运行时只处理整数命令 ID、级别 ID 和解析后的周期计数。

`DRAMNode` 上的两个方法实现时序算法：

- `DRAMNode::check_timing`
- `DRAMNode::update_timing`

`check_timing()` 是只读时序侧。它从根向寻址作用域遍历，在每个节点询问候选命令是否仍被时序阻塞。如果当前周期早于 `m_cmd_ready_clk[command]`，答案立即为 false。否则遍历继续。

如果地址向量命名了特定子节点，`check_timing()` 跟随该单一路径。如果地址向量在下一级别包含 `-1`，则命令作用域较宽，时序必须对该作用域的每个后代成立。这就是 `PREab` 和 `REFab` 等命令如何自然成为多 Bank 检查而无需控制器中的特殊遍历逻辑。

`update_timing()` 是写入侧。它在命令实际发出时运行，并更新目标路径和任何相关同级节点。

在两种情况下更容易理解该逻辑。

首先，考虑位于寻址路径上的节点。`update_timing()`：

- 在 `m_cmd_history[command]` 中记录发出时间
- 查找该级别由该命令触发的所有非同级时序约束
- 使用记录的历史计算每个被阻塞命令何时再次合法
- 更新这些被阻塞命令的 `m_cmd_ready_clk`
- 递归进入子节点

现在考虑同一级别的同级节点。如果地址向量命名了不同的子节点且约束条目标记为 `sibling=true`，Ramulator 更新该同级的 `m_cmd_ready_clk` 而不再深入下降。这就是影响同级 Rank 或同级 Bank Group 的规则如何被清晰建模。

`window` 字段使滚动约束生效。一个很好的例子是 `nFAW`，它限制最近间隔内可以发生多少次激活。如果约束的 `window=4`，节点保留该前置命令的四个最近发出时间。当新命令到达时，Ramulator 回溯到第四个最近的时间戳，并使用它来决定下一个被阻塞命令何时可以发出。

因为递归访问每个相关级别，时序规则自然组合：

- 通道级规则建模共享总线或顶层串行化
- Rank 级规则建模 Rank 范围交互，如刷新和激活窗口
- Bank Group 级规则建模同组限制
- Bank 级规则建模每 Bank 打开、关闭和访问时序
- 伪通道级规则建模 HBM 系列器件使用的每 PC 时序域

重要的要点是 Ramulator 不会将所有时序扁平化为一个巨大的表。它将时序保留在规则实际存在的作用域，然后让递归在运行时组合这些作用域。

#### 9.3.5 状态机侧：接下来应该发生什么命令

时序告诉您命令现在是否可以发出。它不会告诉您请求接下来应该发出哪个命令。这部分来自 `populate_commands()` 中注册的命令处理器。

每个命令最多可提供四个 Bank 级处理器：

- `preq`
  返回应该接下来发出的前提命令
- `action`
  在命令实际发出时变更状态
- `rowhit`
  回答请求是否为行命中
- `rowopen`
  回答目标 Bank 中是否已经有某行被打开

对于正常的 DDR 风格访问，命令链由 `preq` 驱动：

- 如果 Bank 关闭，访问命令的前提是 `ACT`
- 如果 Bank 已打开到请求的行，前提就是访问本身
- 如果 Bank 打开到错误的行，前提变为 `PREpb`

该逻辑直接存在于命令处理器中。`ACT::preq()` 是一个很好的例子。它检查 Bank 是关闭的、已打开到同一行，还是打开到冲突行，然后分别返回 `ACT`、原始命令或 `PREpb`。

`RD::preq()` 和 `WR::preq()` 重用该开行逻辑而不是重复它。`PREpb::action()` 关闭 Bank 并清除 `m_row_state`。`RDA` 和 `WRA` 被建模为也关闭 Bank 的访问命令。`REFab::preq()` 检查所有目标 Bank 是否关闭，如果不是，则首先要求 `PREab`。

`BankTarget` 决定该 Bank 本地分派的宽度：

- `Single`
  一个特定 Bank，用于 `ACT`、`RD`、`WR` 和 `PREpb` 等命令
- `All`
  寻址作用域中的每个 Bank，用于 `PREab` 和 `REFab` 等命令
- `SameBank`
  更宽作用域中的相同 Bank ID，用于需要该模式的标准，如 DDR5

`DRAMDevice::get_target_banks()` 将地址向量和 `BankTarget` 转换为具体的 Bank 节点索引。这就是刷新处理程序如何仍被编写为 Bank 本地逻辑同时影响许多 Bank。

#### 9.3.6 具体请求遍历示例

遍历关闭的 DDR4 Bank 上的普通读取请求有助于理解。

1. 前端向控制器发送读取请求。
2. 控制器将物理地址映射到 `addr_vec`。
3. 控制器从 DRAM 标准的 `supported_requests` 设置 `final_command`，因此读取请求目标为 `RD`。
4. 调度器或控制器调用 `get_preq_command(final_command, addr_vec)`。
5. 器件将该问题分派到相关 Bank 节点。因为 Bank 关闭，答案是 `ACT`。
6. 控制器调用 `check_timing(ACT, addr_vec)`。层次结构检查通道、Rank、Bank Group 和 Bank 时序状态。
7. 如果时序允许，`issue_command(ACT, addr_vec, clk)` 运行。首先通过节点树更新时序，然后应用变更 Bank 状态为打开并记录打开行的功能操作。
8. 因为 `ACT` 被标记为打开命令，请求移动到活动缓冲区而非退役。
9. 在后续 tick 上，控制器再次询问前提命令。现在 Bank 已打开到正确的行，因此答案是 `RD`。
10. `check_timing(RD, addr_vec)` 针对所有相关时序作用域验证访问。
11. `issue_command(RD, addr_vec, clk)` 再次更新时序。`RD` 不打开新行，因此从 DRAM 命令角度看请求现在完成。
12. 控制器退役请求并使用 DRAM 读取延迟分配其离开时间。

如果 Bank 已打开到错误的行，相同的流程会在 `ACT` 之前插入 `PREpb`，然后才到达 `RD`。控制器不会硬编码该序列。它来自对当前 Bank 状态的重复前提检查。

请求不会提前展开为固定命令脚本。每个周期，控制器询问器件："给定现在的状态，该请求的下一个合法命令是什么？"

### 9.4 控制器 Tick 流程

我们以 `GenericDDR` 控制器流程为例进行说明：

1. `tick_prologue()`

   推进控制器时钟，累积队列长度统计信息，并服务完成的读取（即，在应将它们返回前端时调用其回调）。

2. `m_refresh->tick()`

   给刷新管理器注入维护工作的机会。

3. `m_rowpolicy->pre_schedule()` 和插件 `pre_schedule()`

   让策略在候选选择之前做出反应。

4. 通过 3 个请求缓冲区（活动、优先级和普通读/写）进行候选选择

   控制器首先尝试调度活动请求，然后是优先级请求，然后是普通读取或写入流量。活动请求是已经打开其 DRAM 行的请求。优先处理它们可减少浪费周期的过早预充电。

5. `m_rowpolicy->try_upgrade_command(req)`

   行策略可能就地更改命令，例如 `RD` 更改为 `RDA`（即，关闭行策略），如果该升级命令有效且就绪。

6. 发出调度请求需要推进的命令

   `m_device.issue_command(...)` 更新时序状态和任何命令驱动的状态变更。

7. 更新统计信息并通知观察者

   控制器更新行命中和行缺失统计信息，然后在行策略和所有插件上调用 `on_issue(...)`。

8. 推进请求生命周期

   如果发出的命令是最终命令，则请求退役。如果是打开命令如 `ACT`，则请求提升到活动缓冲区。

9. `post_schedule()` 钩子

   行策略和插件可以在 tick 结束时执行操作。

### 9.5 源代码的良好阅读顺序

如果您想在不迷失的情况下理解代码库，这个顺序很有效：

1. `examples/example_config.py`
2. `python/ramulator/__init__.py`
3. `src/ramulator/python/bindings.cpp`
4. `src/ramulator/controller/impl/generic_ddr_controller.cpp`
5. `src/ramulator/controller/controller_base.cpp`
6. `src/ramulator/dram/device.h` 和 `src/ramulator/dram/node.cpp`
7. `python/ramulator/dram/` 中的一个 DRAM 定义，如 `ddr4.py`

该路径从公共 API 开始，然后进入执行路径，最后进入更深入的建模机制。

## 10. 轨迹可视化工具

Ramulator 2.1 包含基于浏览器的轨迹可视化工具，可渲染命令时间线、请求泳道和吞吐量图表。它支持两种模式：

- **离线** — 记录 `.ram2bin` 轨迹文件，然后在浏览器中打开。
- **实时** — 从运行中的模拟实时将事件流式传输到浏览器。

### 10.1 Docker

可视化工具是可选的 — `docker compose up` 仍然只启动 Ramulator 构建容器。使用 Compose 配置文件来包含它：

```bash
docker compose --profile visualizer up --build
```

这会构建自包含的生产镜像并在 `http://localhost:3000` 提供可视化工具。

用于热重载的开发：

```bash
docker compose --profile visualizer-dev up
```

使用 `RAMULATOR_VISUALIZER_PORT` 覆盖端口：

```bash
RAMULATOR_VISUALIZER_PORT=4000 docker compose --profile visualizer up --build
```

### 10.2 不使用 Docker

**要求：** Node.js 20+（含 npm）。

可视化工具必须在首次使用前构建一次：

```bash
cd visualizer
npm install
npm run build
cd ..
```

然后使用以下命令启动：

```bash
python -m ramulator visualize
```

这会启动生产服务器并打印 URL（默认 `http://localhost:3000`）。选项：

```bash
python -m ramulator visualize --port 4000   # 自定义端口
python -m ramulator visualize --dev         # 带热重载的开发服务器
```

### 10.3 离线模式

向控制器添加 `BinTraceRecorder` 插件以记录二进制轨迹：

```python
controller_plugins=[
    ramulator.controller_plugin.BinTraceRecorder(path="my_trace"),
]
```

这会为每个通道生成一个 `.ram2bin` 文件（例如 `my_trace.ch0.ram2bin`）。在浏览器中打开可视化工具并将文件拖放到上传区域。

### 10.4 实时流式传输

改为添加 `LiveTraceStreamer` 插件。首先启动可视化工具，然后运行模拟：

```bash
python -m ramulator visualize          # 终端 1
python my_config.py                    # 终端 2
```

```python
controller_plugins=[
    ramulator.controller_plugin.LiveTraceStreamer(
        port=3000,              # 必须与可视化工具端口匹配
        tick_interval=10000,    # 每 N 个模拟周期刷新
        update_interval_s=0.5,  # 或每 N 个 wall-clock 秒
    ),
]
```

浏览器通过 WebSocket 连接并在命令到达时显示。
