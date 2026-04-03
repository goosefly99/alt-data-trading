# Development Roadmap — Alternative Data Anomaly Detection & Geopolitical Trading System

Converted from spec `bc164a1d-bfb6-4b23-b93b-6ff3175b26ae` (v1.0, 2026-04-03)

---

## Phase 1: Core Data Pipeline & Anomaly Detection

**Description:** Build the data ingestion layer for all 10 sources with error handling, rate limiting, and local storage. Implement the FIRMS anomaly detection algorithm with 14-day rolling baseline and 3x threshold. Set up the DataRecorder to log all raw data. Validate against known historical events (Oct 7, Kursk, Assad fall).

**Estimated Effort:** 2-3 weeks

**Dependencies:** NASA Earthdata account registration, Python environment setup

| # | Deliverable / Task | Status | Target Files | Dependencies |
|---|---|---|---|---|
| 1 | FIRMSCollector with NASA MAP_KEY authentication and region-based queries | Not Started | `src/collectors/firms_collector.py` | NASA Earthdata account |
| 2 | OpenSkyCollector with bounding box aircraft queries | Not Started | `src/collectors/opensky_collector.py` | OpenSky account registration |
| 3 | USGSCollector with GeoJSON parsing and natural/non-natural classification | Not Started | `src/collectors/usgs_collector.py` | None |
| 4 | FinancialIndicatorCollector aggregating 6 financial sources | Not Started | `src/collectors/financial_collector.py` | ExchangeRate API key, CoinGecko API key |
| 5 | PolymarketCollector fetching active geopolitical contracts | Not Started | `src/collectors/polymarket_collector.py` | None |
| 6 | AnomalyDetectionEngine with configurable baseline window and threshold | Not Started | `src/detection/anomaly_engine.py` | Items 1-5 |
| 7 | Continuation bias filter (skip if 7-day baseline tripled) | Not Started | `src/detection/anomaly_engine.py` | Item 6 |
| 8 | DataRecorder saving all raw data to local Parquet/JSON | Not Started | `src/recording/data_recorder.py` | None |
| 9 | Historical validation script testing against 270 days of known data | Not Started | `scripts/historical_validation.py` | Items 6-8 |

---

## Phase 2: Confirmation Scoring & Claude Decision Engine

**Description:** Implement the multi-source ConfirmationScorer with weighted scoring model. Build the ClaudeDecisionEngine with structured prompt template and JSON output parsing. Integrate with PositionSizingEngine for dynamic quarter-Kelly calculations. Test end-to-end signal-to-decision pipeline on historical data.

**Estimated Effort:** 1-2 weeks

**Dependencies:** Phase 1 complete, Anthropic API key

| # | Deliverable / Task | Status | Target Files | Dependencies |
|---|---|---|---|---|
| 10 | ConfirmationScorer with configurable source weights and minimum confirmation threshold | Not Started | `src/confirmation/scorer.py` | Phase 1 |
| 11 | ClaudeDecisionEngine with prompt template, input aggregation, and JSON output parsing | Not Started | `src/decision/claude_engine.py` | Phase 1, Anthropic API key |
| 12 | PositionSizingEngine with EV filter, quarter-Kelly, anomaly scaling, and liquidity cap | Not Started | `src/sizing/kelly_sizer.py` | Item 11 |
| 13 | End-to-end pipeline test: raw data -> anomaly -> score -> Claude decision -> position size | Not Started | `tests/test_pipeline_e2e.py` | Items 10-12 |
| 14 | Dry-run mode producing detailed trade logs without execution | Not Started | `src/execution/executor.py` | Items 10-12 |

---

## Phase 3: Wallet Intelligence Pipeline

**Description:** Build the bulk wallet analysis system. Ingest 17K+ wallets from poly_data, classify strategies with Claude, detect insider-like patterns, and generate copytrade signals. Integrate wallet signals into the ClaudeDecisionEngine as an additional input source.

**Estimated Effort:** 2-3 weeks

**Dependencies:** Phase 2 complete, poly_data dataset downloaded

| # | Deliverable / Task | Status | Target Files | Dependencies |
|---|---|---|---|---|
| 15 | WalletScraper ingesting from poly_data with per-wallet metrics extraction | Not Started | `src/wallets/scraper.py` | poly_data dataset |
| 16 | StrategyClassifier using Claude to map wallets to 11 strategy archetypes | Not Started | `src/wallets/strategy_classifier.py` | Item 15, Anthropic API key |
| 17 | InsiderDetector flagging suspicious patterns (fresh wallets, cluster funding, date-specific entries) | Not Started | `src/wallets/insider_detector.py` | Item 15 |
| 18 | CopytradeSignalGenerator with Sharpe > 2.0 and category win rate filters | Not Started | `src/wallets/copytrade_signals.py` | Items 16-17 |
| 19 | Integration of wallet signals into ClaudeDecisionEngine prompt | Not Started | `src/decision/claude_engine.py` | Items 11, 18 |
| 20 | Daily batch refresh with incremental wallet updates | Not Started | `src/wallets/batch_refresh.py` | Items 15-18 |

---

## Phase 4: Execution, Alerts & Live Deployment

**Description:** Build the ExecutionEngine with py-clob-client integration, exit rule monitoring, and security hardening. Deploy the AlertSystem via Telegram. Set up the full pipeline on Windows Task Scheduler (15-minute cycle). Begin dry-run testing with real-time data.

**Estimated Effort:** 1-2 weeks

**Dependencies:** Phase 3 complete, Polygon wallet with USDC, Polymarket API keys, Telegram bot created

| # | Deliverable / Task | Status | Target Files | Dependencies |
|---|---|---|---|---|
| 21 | ExecutionEngine with py-clob-client order placement, dry-run and live modes | Not Started | `src/execution/executor.py` | Phase 3, Polymarket API keys |
| 22 | Exit rule monitor (take-profit +50%, stop-loss -30%, time-stop 14 days) | Not Started | `src/execution/exit_monitor.py` | Item 21 |
| 23 | Telegram AlertSystem with all 6 alert types and rate limiting | Not Started | `src/alerts/telegram_alerts.py` | Telegram bot token |
| 24 | Windows Task Scheduler configuration for 15-minute cycle | Not Started | `scripts/setup_scheduler.ps1` | Items 21-23 |
| 25 | Security setup: dedicated wallet, USDC approval caps, .env protection | Not Started | `src/security/wallet_security.py` | Polygon wallet |
| 26 | 2-week dry-run validation before live capital deployment | Not Started | `scripts/dry_run.py` | Items 21-25 |
| 27 | Performance dashboard showing signals, decisions, and simulated PnL | Not Started | `src/dashboard/performance.py` | Items 21-26 |

---

## Phase 5: Backtesting Infrastructure & Optimization

**Description:** Build statistically valid backtesting framework using recorded data. Implement fill simulation with realistic slippage modeling. Run minimum 1000-window backtests per strategy variant. Optimize anomaly thresholds, confirmation weights, and Kelly fractions based on results. Address the core lesson: data completeness > strategy quality.

**Estimated Effort:** 2-3 weeks

**Dependencies:** Phase 4 complete, Minimum 30 days of recorded live data

**Note:** DataRecorder must run from Phase 1 day 1, not Phase 4 completion. The 30-day data window starts accumulating at Phase 1 start.

| # | Deliverable / Task | Status | Target Files | Dependencies |
|---|---|---|---|---|
| 28 | Backtesting engine with realistic fill simulation (not perfect execution assumption) | Not Started | `src/backtesting/backtest_engine.py` | Phase 4, 30 days recorded data |
| 29 | Slippage model calibrated from recorded order book depth data | Not Started | `src/backtesting/slippage_model.py` | Item 28 |
| 30 | Pre-window and post-window data analysis for timing optimization | Not Started | `src/backtesting/timing_analysis.py` | Item 28 |
| 31 | Parameter optimization for anomaly thresholds per region | Not Started | `src/backtesting/param_optimizer.py` | Items 28-30 |
| 32 | Confirmation weight tuning based on historical signal quality | Not Started | `src/backtesting/weight_tuner.py` | Items 28-30 |
| 33 | Performance report with Sharpe ratio, max drawdown, win rate by signal type | Not Started | `src/backtesting/performance_report.py` | Items 28-32 |
| 34 | Comparison of anomaly-only vs anomaly+wallet vs full-pipeline performance | Not Started | `src/backtesting/strategy_comparison.py` | Items 28-33 |

---

## Summary

| Phase | Name | Items | Effort | Status |
|---|---|---|---|---|
| 1 | Core Data Pipeline & Anomaly Detection | 1-9 | 2-3 weeks | Not Started |
| 2 | Confirmation Scoring & Claude Decision Engine | 10-14 | 1-2 weeks | Not Started |
| 3 | Wallet Intelligence Pipeline | 15-20 | 2-3 weeks | Not Started |
| 4 | Execution, Alerts & Live Deployment | 21-27 | 1-2 weeks | Not Started |
| 5 | Backtesting Infrastructure & Optimization | 28-34 | 2-3 weeks | Not Started |
| **Total** | | **34 items** | **8-13 weeks** | |
