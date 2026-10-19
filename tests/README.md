# Testing

Unit tests run offline (no exchange, no LLM network calls) and need only the
project requirements installed (`nautilus_trader`, `anthropic`, `openai`).

```bash
# All offline tests
python -m pytest tests -q --deselect tests/manual_order_test.py

# Individual files also run as plain scripts
python tests/test_llm_client.py
python tests/test_bracket_order.py
python tests/test_strategy_components.py
python tests/test_rounding_fix.py
python tests/test_integration_mock.py

# Manual Binance connectivity / order test (needs real keys; use --dry-run)
python tests/manual_order_test.py --dry-run --signal BUY
```

## Files

| File | Covers |
|---|---|
| `helpers.py` | `make_strategy_stub()` - binds the strategy's Python methods onto a plain object so Cython read-only attributes (`cache`, `log`, ...) can be stubbed |
| `test_llm_client.py` | Claude (Anthropic) request shape incl. `output_config` structured outputs + `effort`, graceful fallback when structured outputs are rejected, response validation (unknown enum values -> HOLD), retry -> fallback, OpenAI-compatible path, JSON extraction |
| `test_bracket_order.py` | Bracket SL/TP pricing (S/R and ATR), no unprotected entries, partial-TP state without trailing (regression), trailing stop **modifies** the SL in place (regression: cancelling it cancelled the OUO-linked TP), native-stop fallback after restart, SL never moves backwards |
| `test_strategy_components.py` | Risk-based position sizing (leverage cap, min-notional skip, confidence/RSI modifiers), SL/TP maths, daily-loss guard (live-vs-live only), pause expiry, anti-churn ignores HOLD runs, close-only reversals, pyramiding block |
| `test_rounding_fix.py` | Quantity flooring to 0.001 and the Phase 0 "skip, never upsize" rule |
| `test_integration_mock.py` | Mocked Binance -> indicators -> AI -> order-decision flow |

## Live smoke test (no orders)

```bash
TEST_MODE=true AUTO_CONFIRM=true python main_live.py
```

Runs the full node with real market data and real Claude calls in signal-only
mode; check `logs/signal_log.jsonl` for the recorded signals.
