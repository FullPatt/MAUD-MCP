# MAUD-MCP 🧪🤖

**AI-Agent-Friendly MCP Server for MAUD — Combined Analysis of Diffraction Data**

[![License](https://img.shields.io/badge/License-BSD--3--Clause-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10+-blue.svg)]()
[![MCP Protocol](https://img.shields.io/badge/MCP-1.0+-green.svg)](https://modelcontextprotocol.io)
[![Java 21+](https://img.shields.io/badge/Java-21+-orange.svg)]()

MAUD-MCP 将衍射组合分析软件 [MAUD](https://github.com/luttero/maud)（Materials Analysis Using Diffraction）封装为标准 **MCP (Model Context Protocol)** 服务器，使 AI 智能体（Claude、Copilot、Cursor 等）与自动化工作流可直接调用 Rietveld 精修、物相分析、织构分析、应力分析等全部功能。

MAUD-MCP wraps [MAUD](https://github.com/luttero/maud) as a standard **MCP (Model Context Protocol)** server, enabling AI agents and automated workflows to directly invoke Rietveld refinement, phase analysis, texture analysis, stress analysis, and more.

---

## 项目目的 / Purpose

本仓库是 [luttero/maud](https://github.com/luttero/maud) 的 **fork + MCP 包装**。MAUD Java 源码保持上游原样，MCP 层以独立 Python 包形式通过 subprocess 调用 `Maud.jar`，不修改任何 Java 代码。

This repository is a **MAUD fork + MCP wrapper**. The Java sources stay as upstream; the MCP layer is a self-contained Python package that drives `Maud.jar` via subprocess.

| 特性 | 说明 |
|------|------|
| 🧩 **MCP 协议服务** | 通过 stdio 暴露 **12 个** MCP 工具 |
| 🤖 **AI 智能体接口** | AI 可直接加载数据、创建参数、执行计算、解析结果 |
| ☕ **Java 桥接** | subprocess 调用 `Maud.jar`，Java 源码零改动 |
| ⚙️ **Headless 运行** | 无需 MAUD GUI，纯命令行 |
| 📄 **PAR 文件编辑** | .par（CIF 格式）读写、环路解析、约束编辑 |
| 🧪 **测试覆盖** | 6 个测试文件，64 个用例 |
| 🔍 **智能诊断** | 精修质量评估 + 参数推荐 + 报告生成 |

### 分支说明 / Branch

| | 分支 |
|---|---|
| 本仓库默认分支 | `version2` |
| 上游 `luttero/maud` 默认分支 | `version3` |

MCP 层基于 `version2` 维护；与上游的同步状态以提交历史为准。

### 架构 / Architecture

```
AI Agent (Claude/Copilot)
        │ MCP stdio
        ▼
┌───────────────────────┐     subprocess     ┌───────────────────────┐
│   src/maud_mcp/       │ ─────────────────→  │  Maud.jar (Java)      │
│   (MCP 协议 + 桥接)    │                     │  (Rietveld 引擎)       │
└───────────────────────┘                     └───────────────────────┘
```

---

## 代码结构 / Code Structure

```
MAUD-MCP/
├── src/                                    # MAUD Java 源码 + MCP 服务器
│   ├── com/                                #   MAUD 核心 Java 包（76 个 .java）
│   ├── ij/  gov/  org/  net/  fr/  it/     #   ImageJ 等第三方 Java 依赖源码
│   ├── HTTPClient/  Jama/  gl4java/ ...    #   其他上游依赖源码
│   └── maud_mcp/                           # 💚 MCP 服务器（本 fork 新增）
│       ├── __init__.py                     #    版本声明
│       ├── __main__.py                     #    CLI: status / validate / test-ins / server
│       ├── config.py                       #    引擎自动检测（MAUD_HOME / JAVA_HOME）
│       ├── exceptions.py                   #    自定义异常
│       ├── core/                           #    核心引擎
│       │   ├── java_bridge.py              #    ☕ Java subprocess 桥接
│       │   ├── par_editor.py               #    📄 .par（CIF 格式）读写器
│       │   ├── data_manager.py             #    🔄 数据格式转换 + CIF 解析
│       │   └── result_parser.py            #    📊 结果解析（Rwp / GOF / 晶胞）
│       ├── server/mcp_server.py            #    MCP 协议层（12 个工具）
│       ├── ai/                             #    🤖 参数推荐 / 诊断 / 报告
│       ├── tests/                          #    🧪 6 个测试文件（64 用例）
│       └── _legacy/                        #    旧版实现备份
├── libs/                                   # 上游 Java 依赖 JAR（编译期）
├── ImageJ/  media/  docs/                  # 上游资源
├── build.xml                               # 🏗️ Ant 构建配置
├── Maud.iml / Maud.ipr / Maud.iws          # IntelliJ IDEA 工程文件
└── ant_maud_v2.properties                  # Ant 构建属性
```

### 模块依赖关系 / Module Dependencies

```
maud_mcp (Python MCP Server)
    │
    ├── core/java_bridge.py    — subprocess 调用 Maud.jar
    ├── core/par_editor.py     — 解析/编辑 .par（CIF 格式）参数文件
    ├── core/data_manager.py   — 衍射数据格式转换 + CIF 解析
    ├── core/result_parser.py  — 解析 MAUD stdout 与 .par 结果
    ├── server/mcp_server.py   — FastMCP 服务器（12 个工具）
    └── ai/                    — 参数推荐、诊断、报告生成
```

---

## 安装 / Installation

### ⚠️ 关于 MAUD 运行时

**本仓库不包含 `Maud.jar`。** `libs/` 里是上游 MAUD 编译所需的第三方依赖 JAR，不含 MAUD 本体。运行 MCP 服务器前需自行准备运行时：

```bash
# 方式 A：下载上游预编译包
wget https://github.com/luttero/maud/releases/download/v2.99993/maud.zip
unzip maud.zip -d maud_runtime/          # 期望结构：maud_runtime/lib/Maud.jar

# 方式 B：自行编译（需 Apache Ant + JDK 21）
ant -buildfile build.xml jar
cp build/Maud.jar maud_runtime/lib/
```

MCP 服务器按以下优先级定位 MAUD（见 `src/maud_mcp/config.py`）：

| 优先级 | 来源 |
|---|---|
| 1 | 环境变量 `MAUD_HOME` |
| 2 | `<repo>/maud_runtime/lib/Maud.jar` |
| 3 | `~/maud_runtime/lib/Maud.jar` |
| 4 | `<cwd>/maud_runtime/lib/Maud.jar` |

### 前置条件 / Prerequisites

| 依赖 | 用途 | 安装方式 |
|------|------|----------|
| **Java 21+** | 运行 MAUD 引擎 | Adoptium Temurin |
| **Python 3.10+** | MCP 服务器 | `apt install python3.11` |
| Python 包 | MCP / numpy / scipy | `pip install mcp>=1.0 numpy scipy PyCifRW` |
| **Maud.jar** | Rietveld 精修引擎 | 见上（需自行获取） |

### 1️⃣ 获取代码与 Python 依赖

```bash
git clone https://github.com/FullPatt/MAUD-MCP.git
cd MAUD-MCP
pip install mcp>=1.0 numpy scipy PyCifRW
```

### 2️⃣ 安装 Java 21

```bash
wget -qO- https://github.com/adoptium/temurin21-binaries/releases/latest/download/OpenJDK21U-jdk_x64_linux_hotspot_21.0.6_7.tar.gz | tar xz
export JAVA_HOME=$(pwd)/jdk-21.0.6+7
export PATH=$JAVA_HOME/bin:$PATH
java -version
```

### 3️⃣ 部署 Maud.jar

见上文「关于 MAUD 运行时」。

### 4️⃣ 验证

```bash
python -m src.maud_mcp
```

正常输出示例：

```
=== MAUD 引擎状态 ===
  状态:      ✅ ready
  Maud.jar:  /path/to/MAUD-MCP/maud_runtime/lib/Maud.jar
  存在:      ✅
  Java:      /path/to/jdk-21.0.6+7/bin/java
  存在:      ✅
  版本:      21.0.6
  平台:      Linux
```

### 5️⃣ 最小运行测试

```bash
python -m src.maud_mcp --test-ins
# → ✅ INS 执行成功
```

### 环境变量 / Environment Variables

| 变量 | 说明 | 默认值 |
|---|---|---|
| `MAUD_HOME` | MAUD 安装目录（含 `lib/Maud.jar`） | 自动检测 |
| `MAUD_JAVA_HOME` | JDK 安装目录 | 自动检测 |
| `MAUD_WORK_DIR` | 工作目录 | `/tmp/maud_work` |
| `MAUD_MAX_MEM` | Java 最大堆内存 | `2g` |
| `MAUD_TIMEOUT` | 精修超时（秒） | `180` |
| `MAUD_LOG_LEVEL` | 日志级别 | `INFO` |

---

## MCP 服务器使用 / MCP Server Usage

### 启动

```bash
python -m src.maud_mcp --server
```

### MCP 宿主配置 / Claude Desktop Config

```json
{
  "mcpServers": {
    "maud-mcp": {
      "command": "python3",
      "args": ["-m", "src.maud_mcp", "--server"],
      "cwd": "/path/to/MAUD-MCP"
    }
  }
}
```

### MCP 工具清单（12 个）

| 类别 | 工具 | 功能 |
|------|------|------|
| **系统** | `get_status` | MAUD 引擎状态、Java 版本、JAR 路径 |
| **数据** | `load_data` | 加载衍射数据并返回摘要（点数、范围、格式） |
| | `convert_data` | 转换数据格式（→ .xye / .dat / .raw） |
| | `data_stats` | 数据统计直方图 |
| **PAR 编辑** | `read_par` | 读取 .par 文件，返回 CIF 结构化 JSON |
| | `edit_par` | 修改 .par 参数（晶胞、原子、迭代次数、标题） |
| | `import_cif` | 从 CIF 生成 .par 文件 |
| | `generate_ins` | 从数据 + CIF 自动生成 INS 控制文件 |
| **精修** | `refine` | 执行 MAUD Rietveld 精修 |
| | `compute` | 模拟计算（0 次迭代） |
| | `get_results` | 解析精修结果（Rwp、GOF、晶胞参数） |
| **诊断** | `diagnose` | 分析精修日志，给出诊断与修复建议 |

> 上表与 `server/mcp_server.py` 中 `@mcp.tool()` 注册的 **12 个** 工具逐一对齐。`ai/` 目录下的参数推荐（`params.py`）与报告生成（`reporter.py`）目前为内部模块，尚未注册为独立 MCP 工具。

### 快速功能测试

```python
import sys
sys.path.insert(0, "/path/to/MAUD-MCP/src")

from maud_mcp.core.java_bridge import JavaBridge
bridge = JavaBridge()
status = bridge.get_status()
print(f"MAUD ready: {status['status']}")

result = bridge.run_ins("""
_riet_analysis_iteration_number  0
""")
print(f"SUCCESS: {result.success}")
```

---

## MAUD 能力概述 / MAUD Capabilities

MAUD（Materials Analysis Using Diffraction）是开源的 Java 衍射组合分析软件，由 **Luca Lutterotti**（特伦托大学）开发。

| 类别 | 可确定参数 |
|------|-----------|
| **晶体结构** | 晶格参数、原子坐标、占位率、温度因子 |
| **微观结构** | 晶粒尺寸、微应变分布、层错、位错密度 |
| **织构 (ODF)** | WIMV/EWIMV/谐波/标准函数法，MTEX 集成 |
| **残余应力** | 宏观应力张量、三轴应力、EPSC 模型 |
| **物相定量** | 晶相 + 非晶相质量/体积分数 |
| **化学组成** | XRF / EDXRF / TXRF 元素分析 |
| **反射率** | 薄膜厚度、密度、粗糙度（Parrat / 矩阵法） |
| **结构解析** | 遗传算法、模拟退火、反蒙特卡洛、Charge Flipping |
| **PDF** | 对分布函数导出 |
| **电子密度图** | 3D Fourier / MEM 重构 |

**辐射源**：X 射线（Cu/Co/Cr/Mo/Fe/Ag/Ga…）、同步辐射、中子（恒定波长 ILL / TOF LANSCE·HIPPO·ISIS GEM）、电子衍射

**几何**：Bragg-Brentano、Debye-Scherrer、平板 IP、CPS 探测器、TOF 多 bank、Laue 透射、反射率

**数据格式**：60+（Bruker/Siemens UXD·RAW、Philips XRDML、Rigaku、GSAS、FullProf、CIF、TIFF、HDF5、ILL D1B/D20/D19、HIPPO、INEL、MDI 等）

---

## 本 Fork 的变更 / Changes vs Upstream

| 变更 | 路径 | 说明 |
|------|------|------|
| maud_mcp Python 包 | `src/maud_mcp/` | 完整 MCP 服务器（core / server / ai / tests） |
| Java 桥接 | `core/java_bridge.py` | subprocess 调用 `Maud.jar` |
| PAR 编辑器 | `core/par_editor.py` | CIF 格式 .par 的解析与编辑 |
| 数据管理器 | `core/data_manager.py` | 格式转换 + CIF 解析 |
| 结果解析器 | `core/result_parser.py` | Rwp / GOF / 晶胞参数提取 |
| AI 模块 | `ai/` | 参数推荐、诊断、报告生成 |
| MCP 服务器 | `server/mcp_server.py` | 12 个 MCP 工具 |
| 测试套件 | `tests/` | 6 个文件、64 个用例 |
| 旧版实现 | `_legacy/` | 早期 `maud_api.py` / `maud_mcp_server.py` 备份 |

> Java 源码、`libs/`、`ImageJ/`、`build.xml` 等均保持上游原样，**未作修改**。

---

## 测试 / Testing

```bash
cd /path/to/MAUD-MCP
python -m pytest src/maud_mcp/tests/ -v
```

| 测试文件 | 用例数 |
|---|---|
| `test_java_bridge.py` | 7 |
| `test_par_editor.py` | 13 |
| `test_result_parser.py` | 10 |
| `test_data_manager.py` | 8 |
| `test_mcp_server.py` | 11 |
| `test_ai_modules.py` | 15 |
| **合计** | **64** |

---

## 相关项目 / Related Projects

| 项目 | 说明 | 仓库 |
|------|------|------|
| **Profex-MCP** | Profex / BGMN XRD 分析与精修 MCP 服务器 | [FullPatt/Profex-MCP](https://github.com/FullPatt/Profex-MCP) |
| **FullProf-App-MCP** | FullProf Rietveld 精修 MCP 服务器 | [FullPatt/FullProf-App-MCP](https://github.com/FullPatt/FullProf-App-MCP) |
| **GSAS2-MCP** | GSAS-II 引擎打包（Rietveld 精修运行时底座） | [FullPatt/GSAS2-MCP](https://github.com/FullPatt/GSAS2-MCP) |

---

## 许可证 / License

**BSD 3-Clause License**（与上游 MAUD 一致）

本仓库是 [luttero/maud](https://github.com/luttero/maud) 的 fork，所有上游贡献者的版权均保留。

---

## 致谢 / Acknowledgements

- **Luca Lutterotti** — MAUD 作者，University of Trento
- **Advanced Photon Source / Argonne National Lab** — GSAS-II 开发团队
- **Bruker AXS / Rigaku / PANalytical** — XRD 数据格式标准

---

*MAUD 主页: https://maud.radiographema.com*
*MAUD 上游仓库: https://github.com/luttero/maud*
