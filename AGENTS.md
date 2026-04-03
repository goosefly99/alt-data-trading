# AGENTS.md — Alt Data Anomaly Detection & Geopolitical Trading

## Overview
Geopolitical trading system using alternative data sources (NASA FIRMS satellite fire data, OpenSky military flights, USGS seismic data) to detect anomalies and trade prediction markets before traditional news breaks.

## Tech Stack
- **Language:** Python 3.11+
- **Data Sources:** NASA FIRMS API, OpenSky Network API, USGS Earthquake API, ExchangeRate API, yfinance, CoinGecko API, Alternative.me API, Gold API
- **AI/LLM:** Claude API (Anthropic) — trade decisions, wallet classification, anomaly interpretation
- **Execution:** py-clob-client (Polymarket official SDK)
- **Wallet Analysis:** poly_data (86M+ historical trades), polyterm (whale tracking), insider-tracker (ML-based detection)
- **Notifications:** python-telegram-bot
- **Data Storage:** pandas + pyarrow (Parquet)
- **Scheduling:** Windows Task Scheduler / cron
- **Security:** python-dotenv
- **HTTP:** httpx / aiohttp (async parallel polling)
- **Monitoring:** Revoke.cash (USDC approval management)

## Architecture
1. **DataIngestionLayer** — Centralized polling of all 10 alternative data sources on a 15-minute cycle with normalization and rate limiting
2. **AnomalyDetectionEngine** — 14-day rolling average baseline per region, 3x threshold detection, continuation bias filter
3. **ConfirmationScorer** — Weighted composite tension score (0-100) requiring 2+ independent confirming sources
4. **WalletIntelligencePipeline** — Bulk 17K+ wallet analysis, strategy classification into 11 archetypes, insider detection
5. **ClaudeDecisionEngine** — Central trade brain synthesizing anomalies, scores, wallet signals, and market data into structured JSON decisions
6. **PositionSizingEngine** — Dynamic quarter-Kelly sizing scaled by anomaly magnitude, confirmation count, and liquidity caps
7. **ExecutionEngine** — Polymarket order placement via py-clob-client with dry-run/live modes and TP/SL/time-stop exit rules
8. **DataRecorder** — Complete market microstructure recording (order books, fills, spreads) for backtesting with state recovery
9. **AlertSystem** — Real-time Telegram notifications for anomalies, signals, trades, exits, wallet activity, and system health
10. **MonitoredRegions** — Configurable geopolitical region definitions (Israel/Gaza, Ukraine/Kursk, Syria, Iran, Korean Peninsula, Taiwan Strait)

## Conventions
- All source code in `src/` with absolute imports
- Tests in `tests/` mirroring src/ structure
- Config via environment variables + .env
- Windows Task Scheduler for 15-minute polling cycles
- Claude API for anomaly classification
- Logging via structlog
- Type hints required on all public functions
- State persistence via DataRecorder + last_run_state.json

## Required Environment Variables
- POLYMARKET_PRIVATE_KEY, POLYMARKET_API_KEY, POLYMARKET_SECRET, POLYMARKET_PASSPHRASE
- ANTHROPIC_API_KEY
- TELEGRAM_BOT_TOKEN, TELEGRAM_CHAT_ID
- OPENSKY_USERNAME, OPENSKY_PASSWORD
- COINGECKO_API_KEY
- EXCHANGERATE_API_KEY

## Directory Structure
```
src/
  collectors/     # FIRMSCollector, OpenSkyCollector, USGSCollector, FinancialIndicatorCollector
  detection/      # AnomalyDetectionEngine, baseline management
  confirmation/   # ConfirmationScorer, multi-source correlation
  wallets/        # WalletAnalyzer, copytrade signals
  decision/       # ClaudeDecisionEngine, trade evaluation
  sizing/         # KellyPositionSizer
  execution/      # PolymarketExecutor, order management
  regions/        # MonitoredRegion configs, geofencing
  recording/      # DataRecorder, pipeline state recovery
  security/       # API key management, rate limiting
tests/
```
