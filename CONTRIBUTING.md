# Contributing to worldcup-oracle

Thank you for your interest in contributing! worldcup-oracle is a community
project that welcomes contributions from anyone.

## Code of Conduct

This project follows the [Contributor Covenant](https://www.contributor-covenant.org/).
By participating, you are expected to uphold this code.

## How to contribute

### Reporting bugs

Open a [GitHub issue](https://github.com/YOUR_USERNAME/worldcup-oracle/issues)
using the bug report template. Include:
- A minimal reproduction command
- Expected vs actual output
- Output of `python -m scripts.banner` (environment + version)

### Suggesting features

Open a GitHub issue using the feature request template. Describe:
- The use case (who benefits, in what workflow)
- The proposed command/interface
- Any alternatives you've considered

### Submitting code

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-change`)
3. Make your changes
4. Run tests: `python tests/acceptance.py` — all 56 must pass
5. Add new tests for new functionality
6. Update `CHANGELOG.md`
7. Submit a pull request

### Style guide

- **Language**: comments in code should be in Chinese; technical terms in English
- **Type hints**: use Python 3.10+ type hints
- **Pure functions**: prefer pure functions for testability
- **Error handling**: raise specific exceptions, never silently catch
- **Logging**: use the `worldcup-oracle.*` logger namespace
- **No hardcoded data**: anything that varies match-to-match should come from
  the data layer, not the script

## Development setup

```bash
git clone https://github.com/YOUR_USERNAME/worldcup-oracle.git
cd worldcup-oracle
pip install requests pyyaml numpy scipy
python tests/acceptance.py
```

## Architecture

```
scripts/
├── banner.py          # 品牌头 + 时间感知 + 模型卡
├── secrets.py         # 零配置 key 解析
├── data_fetch.py      # 数据接入层(多源 + demo 兜底)
├── wc2026_data.py     # 内置 2026 兜底数据
├── dashboard.py       # 赛事看板
├── live_match.py      # 实时比赛追踪
├── briefing.py        # 今日/明日播报
├── predict_match.py   # 单场预测
├── news.py            # 资讯聚合
├── elo_model.py       # Elo 模型
├── poisson_model.py   # Dixon-Coles 泊松
├── market_model.py    # 赔率去水位
├── form_model.py      # 近期状态
├── ensemble.py        # 集成 + 校准
├── simulate_tournament.py  # 蒙特卡洛
├── evaluate.py        # 回测
└── lib/               # 工具(tiebreak / bracket / team_names / format)
```

## License

By contributing, you agree that your contributions will be licensed under the
[Apache License 2.0](LICENSE).
