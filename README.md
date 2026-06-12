# worldcup-oracle · 世界杯神谕

> **v1.0.0** · 商用级 2026 世界杯专业预测 Skill · 零配置 · 多模型集成 · 实时追踪
>
> 当前时间:**2026-06-12**,世界杯 **Day 2**,小组赛第 1 轮进行中。

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![Tests](https://img.shields.io/badge/tests-56%2F56-success)](./tests/acceptance.py)

---

## 它是什么

`worldcup-oracle` 是一个**统计预测引擎**——不是猜比分机器人,而是一个有方法论、
可回测、可自我评估的商用级 Skill。它用四种基础模型(Elo、Dixon-Coles 泊松、
市场赔率去水位、近期状态)做集成,蒙特卡洛模拟整届赛事,落地每条预测,
赛后比对真实赛果,滚动计算 Brier Score / Log Loss / 校准度。

## 30 秒上手

```bash
# 1. 安装依赖(OpenClaw/Hermes 已有则跳过)
pip install requests pyyaml numpy scipy --break-system-packages

# 2. 不需要填 key —— OpenClaw/Hermes 已注入。
#    验证一下:
python scripts/data_fetch.py health

# 3. 立刻看 dashboard
python scripts/dashboard.py

# 4. 试一次单场预测
python scripts/predict_match.py --home "美国" --away "巴拉圭"
# (这场已赛,系统会直接给 2-0 真实赛果,不再预测)

# 5. 跑全赛事夺冠概率模拟
python scripts/simulate_tournament.py --scope title --n 5000
```

## 核心能力

| 能力 | 命令 | 说明 |
|---|---|---|
| **赛事总览** | `dashboard.py` | 首屏看板:今日 / 昨日 / 进行中 / 积分榜 / 夺冠 Top-8 |
| **实时比赛** | `live_match.py` | 当前进行的比赛比分 + 实时模型概率重算 |
| **今日播报** | `briefing.py --day today` | 全部今日比赛的预测卡片 |
| **单场预测** | `predict_match.py` | 三分布 + Top-3 比分 + 期望进球 + 关键因子 |
| **小组出线** | `simulate_tournament.py --scope group` | X 组 4 队的出线概率 |
| **夺冠概率** | `simulate_tournament.py --scope title` | 48 队的夺冠 / 决赛 / 4 强概率 |
| **淘汰赛对阵** | `simulate_tournament.py --scope bracket` | 最可能路径 |
| **复盘 / 命中率** | `evaluate.py` | Brier / LogLoss / 校准曲线 |
| **资讯** | `news.py` | 当日要闻(比赛报道 / 伤停 / 赔率变动) |
| **健康检查** | `data_fetch.py health` | 哪个数据源可用 + 错误日志 |
| **模型卡** | `python -m scripts.banner` | 架构 / 训练 / 局限 / 许可证 |

## 零配置 API Key

商用产品的标志:用户不需要填任何 key。系统按以下优先级自动找:

1. **环境变量** — `WORLDCUP_API_FOOTBALL_KEY`, `FOOTBALL_DATA_TOKEN` 等(逗号分隔支持多 key)
2. **父级平台配置** — `~/.openclaw/config/keys.yaml`, `~/.hermes/config/keys.yaml`
3. **本地 `config/api_keys.yaml`(向后兼容老用户)
4. **系统 keyring** — macOS Keychain / Windows Credential Manager

所有数据拉取失败时,自动回退到内置 2026 抽签基准数据(演示模式),**所有输出都会显式标注数据源**:

- 🟢 `api_football` — 真实数据,最强信号
- 🟡 `football_data` — 备源
- 🟠 `demo` — 内置基准,赛事进行中也会持续更新

## 多模型集成

```
P_final = 0.30·Elo + 0.20·Poisson + 0.40·Market + 0.10·Form
```

- **Elo**:实力接近度衰减的平局模型 + 主办国主场加成
- **Dixon-Coles 泊松**:截断到 7:7,带 τ 修正低比分,给比分分布
- **市场赔率去水位**:多家博彩均值 + overround 归一化
- **近期状态**:近 10 场加权 → logistic 映射

**缺源时权重按比例摊给在场模型**(非简单置零)。

## 安装

### OpenClaw

```bash
cp -r worldcup-oracle/ ~/.openclaw/skills/
openclaw skill reload
```

### Hermes

```bash
cp -r worldcup-oracle/ ~/.hermes/skills/
hermes skill register worldcup-oracle
```

## 定时任务

### OpenClaw (`cron.yaml`)

```yaml
tasks:
  - name: worldcup-dashboard-morning
    cron: "0 8 * * *"          # 比赛日早 8 点 UTC
    command: python scripts/dashboard.py
    notify: true

  - name: worldcup-live-poll
    cron: "*/15 18-23 * * *"   # 比赛时段每 15 分钟
    command: python scripts/live_match.py

  - name: worldcup-backtest-nightly
    cron: "30 2 * * *"
    command: python scripts/evaluate.py --backfill

  - name: worldcup-odds-refresh
    cron: "0 */6 * * *"
    command: python scripts/data_fetch.py refresh
```

### Hermes (`schedule.toml`)

```toml
[[task]]
name = "worldcup-dashboard-morning"
cron = "0 8 * * *"
command = "python scripts/dashboard.py"

[[task]]
name = "worldcup-live-poll"
cron = "*/15 18-23 * * *"
command = "python scripts/live_match.py"

[[task]]
name = "worldcup-backtest-nightly"
cron = "30 2 * * *"
command = "python scripts/evaluate.py --backfill"
```

## 目录结构

```
worldcup-oracle/
├── SKILL.md                # 入口 / 触发词
├── README.md               # 本文件
├── LICENSE
├── config/
│   ├── settings.yaml       # 可调参数
│   └── api_keys.example.yaml
├── data/
│   ├── tournament.json     # 赛制基准
│   ├── cache/              # API 响应缓存
│   └── predictions_log.jsonl
├── scripts/
│   ├── banner.py           # 品牌头 + 模型卡 + 时间感知
│   ├── secrets.py          # 零配置 key 解析
│   ├── data_fetch.py       # 多源接入 + demo 兜底
│   ├── wc2026_data.py      # 内置 2026 抽签基准
│   ├── dashboard.py        # 赛事总览看板
│   ├── live_match.py       # 实时比赛追踪
│   ├── briefing.py         # 今日/明日播报
│   ├── predict_match.py    # 单场预测
│   ├── news.py             # 资讯聚合
│   ├── elo_model.py
│   ├── poisson_model.py
│   ├── market_model.py
│   ├── form_model.py
│   ├── ensemble.py
│   ├── simulate_tournament.py
│   ├── evaluate.py
│   └── lib/                # tiebreak / bracket / team_names / format
└── tests/
    └── acceptance.py
```

## 工程取舍

- **零配置 key**:env > 父级 config > 本地 yaml > keyring;多 key 轮询,429/5xx 自愈。
- **多数据源 + demo 兜底**:API 全挂时仍可运行,标注数据源可信度。
- **时间感知**:所有输出顶部带"今天是 Day X"和当前赛事阶段,讲人话。
- **品牌 + 模型卡**:商用产品的"门面"和"透明度"——用户能看到这是谁、怎么算的、局限在哪。
- **已赛比赛不再预测**:系统知道 MEX 2-1 RSA 已经发生,直接给真实赛果。
- **蒙特卡洛**:默认 5000 次,带进度报告;R32 bracket 8 种 scenario 完整实现。
- **缺源权重按比例摊**:非简单置零,保持 ensemble 总权重 = 1。

## 免责声明

本项目所有内容为基于历史数据的统计预测,仅供学习研究使用,**不构成任何投注或博彩建议**。

---

`WorldCup Oracle v1.0.0` · `ensemble-v2` · `build 2026.06.12` · Apache-2.0
