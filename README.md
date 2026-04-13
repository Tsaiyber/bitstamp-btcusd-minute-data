# Bitstamp BTC/USD 1分钟 OHLC 数据

BTC/USD 历史与实时 1 分钟 K 线数据，个人数据基础设施

![Version](https://img.shields.io/badge/Version-0.1.0-green.svg)
![Python](https://img.shields.io/badge/Python-3.13+-blue.svg?logo=python&logoColor=white)
![uv](https://img.shields.io/badge/uv-latest-purple.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

---

## 📖 项目概述

来自 Bitstamp 交易所的 BTC/USD **1 分钟 OHLC** 数据，覆盖 2012 年至今。历史数据以压缩 CSV 形式存储，`uv run btcdata-sync` 从 Bitstamp API 拉取增量数据并自动合并为单一 Parquet 文件。

- 📊 **完整历史** — 2012-01-01 至今，750 万条记录，无缺口无空值
- 🔄 **增量同步 + 自动合并** — `uv run btcdata-sync` 拉取最新分钟数据，合并输出 `btcusd_bitstamp_1min_all.parquet`
- 🔍 **数据校验** — `uv run btcdata-inspect` 检查缺口、重复、空值
- ⚡ **兼容 pandas 3.x** — 修复原版 `fill_missing_minutes` 在 pandas 3 下的时间戳溢出 bug

---

## 📊 数据集概览

| 文件 | 时间范围 | 记录数 | 说明 |
|------|----------|--------|------|
| `data/historical/btcusd_bitstamp_1min_2012-2025.csv.gz` | 2012-01-01 ~ 2025-01-07 | 6,846,600 | 历史压缩包，一次性下载 |
| `data/updates/btcusd_bitstamp_1min_latest.csv` | 2025-01-07 ~ 今日 | ~65万+ | 增量更新（btcdata-sync 写入） |
| `data/btcusd_bitstamp_1min_all.parquet` | 2012-01-01 ~ 今日 | ~750万 | 合并全量文件（btcdata-sync 自动生成） |

字段：`timestamp` · `open` · `high` · `low` · `close` · `volume`

---

## 🚀 快速开始

### 环境要求

- [uv](https://github.com/astral-sh/uv) 已安装

### 安装

```bash
git clone https://github.com/ff137/bitstamp-btcusd-minute-data
cd bitstamp-btcusd-minute-data
uv sync
```

### 日常同步

```bash
uv run btcdata-sync
```

输出示例：
```
Loaded daily dataset with 653294 records
No missing minutes detected
No null values detected
Successfully saved 653412 records to data/updates/btcusd_bitstamp_1min_latest.csv
Merged 7499906 records -> data/btcusd_bitstamp_1min_all.parquet
```

### 检查数据

```bash
uv run btcdata-inspect updated   # 检查增量更新文件
uv run btcdata-inspect bulk      # 检查历史压缩包
uv run btcdata-inspect merged    # 检查合并后全量数据
```

---

## 📚 工作流程

```mermaid
flowchart TD
    subgraph S1["  历史数据（一次性）  "]
        GZ[("🗜️ CSV.gz\n历史文件\n2012~2025")]
    end

    subgraph S2["  增量同步（日常）  "]
        API["🌐 Bitstamp API\nOHLC v2"]
        SYNC["⚡ btcdata-sync"]
        CSV[("📄 latest.csv\n增量更新")]
        PQ[("📦 all.parquet\n全量合并")]
        API -->|"fetch missing"| SYNC --> CSV
        SYNC -->|"auto merge"| PQ
        GZ -->|"concat"| PQ
    end

    subgraph S3["  使用数据  "]
        INSPECT["🔍 btcdata-inspect\n校验完整性"]
        ML["🤖 ML 模型\n5min/15min 预测"]
        PQ --> INSPECT
        PQ --> ML
    end
```

---

## 📁 项目结构

```
bitstamp-btcusd-minute-data/
├── data/
│   ├── historical/
│   │   └── btcusd_bitstamp_1min_2012-2025.csv.gz     # 历史数据（一次性下载）
│   ├── updates/
│   │   └── btcusd_bitstamp_1min_latest.csv            # 增量更新
│   └── btcusd_bitstamp_1min_all.parquet               # 全量合并（自动生成）
├── scripts/
│   ├── update_data.py       # btcdata-sync 入口（同步 + 合并）
│   ├── inspect_data.py      # btcdata-inspect 入口
│   └── preprocess_bulk_data.py  # 一次性历史数据预处理
├── pyproject.toml
└── uv.lock
```

---



## 🔗 技术栈

| 库 | 版本 | 用途 |
|----|------|------|
| pandas | >=2.2.3 | 数据处理 |
| pyarrow | >=18.0 | Parquet 读写 |
| requests | >=2.32.3 | Bitstamp API 请求 |
| ruff | >=0.9.7 | 代码格式化（dev） |

---

## 📄 许可证

MIT — Fork 自 [ff137/bitstamp-btcusd-minute-data](https://github.com/ff137/bitstamp-btcusd-minute-data)
