# Changelog

All notable changes to worldcup-oracle are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-06-12

### Added
- **First commercial release**, aligned with the 2026 FIFA World Cup opening day (June 11, 2026).
- Multi-model ensemble: Elo + Dixon-Coles Poisson + market consensus + recent form
- Monte Carlo tournament simulator with R32 bracket generation
- 8 R32 "qualified 3rd place" scenario templates
- FIFA tiebreak rules (GD → GF → H2H → fair play → draw)
- Backtest engine: Brier score, Log Loss, hit rate, calibration curve
- Zero-config API key resolution (env → parent config → local yaml → system keyring)
- Time-aware UI: auto-detects "Day X" of the tournament
- Built-in fallback dataset (`wc2026_data.py`) — system runs even with no API access
- Three new CLI commands: `dashboard`, `live_match`, `news`
- Multi-source data layer: API-Football (primary) → football-data.org (fallback) → demo
- Multi-key pool with 429/5xx auto-failover
- Brand identity (banner + model card + version)
- 56-test acceptance suite, all passing

### Changed
- `SKILL.md` rewritten with commercial-grade trigger words
- `predict_match.py` now auto-returns actual result for played matches
- `briefing.py` and `simulate_tournament.py` updated for the new data layer
- `README.md` restructured for production use

### Security
- All API keys resolved at runtime via `secrets.py`; never written into source
- `.gitignore` blocks `config/api_keys.yaml` and other sensitive files

## [0.1.0] - 2026-05-15

### Added
- Initial proof-of-concept
- 4 base models + 1 ensemble
- Single-match prediction CLI
- Basic backtest
- 32 acceptance tests
