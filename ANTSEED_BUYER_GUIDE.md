# RiskRangesOS AntSeed Buyer Guide

RiskRangesOS exposes deterministic Binance spot USDT trading-signal services through an OpenAI-compatible chat-completions API. The services return compact JSON for agent workflows, dashboards, ranking bots, and execution-review systems.

Disclaimer: systematic signal output, not financial advice.

## Buyer Summary

| Field | Value |
| --- | --- |
| Seller / peer handle | `@riskrangesOS` |
| Public service ID | `riskranges-binance-fractal-signals-v1` |
| Rankings model ID | `riskranges-binance-rankings-v1` |
| Upstream signal model | `crypto-signal-v3-3` |
| Protocol | OpenAI chat completions |
| Supported universe | Binance spot USDT pairs |
| Output format | Compact JSON in `choices[0].message.content` |
| Contact | `@riskrangesOS` |
| Pricing | Use the active AntSeed marketplace listing or buyer agreement. Pricing is not encoded in the RiskRangesOS API. |

Keywords/categories:

`crypto`, `trading-signals`, `risk-ranges`, `fractal-signals`, `signal-ranking`, `systematic-trading`, `market-structure`, `binance-spot`, `usdt-pairs`, `agent-api`, `json-output`

## Service 1: Single-Symbol Signal

Model ID:

```text
riskranges-binance-fractal-signals-v1
```

Use this service when an agent needs the current structured signal for one Binance spot USDT ticker.

What it does:

- Accepts one Binance spot USDT symbol.
- Generates the current v3_3 fractal/risk-range signal.
- Returns machine-readable JSON with decision, execution plan, risk levels, targets, market state, quality metadata, and disclaimer.

Input examples:

```text
BTCUSDT
LINKUSDT
Give me the signal for SOLUSDT
```

Expected output fields include:

- `decision.final_action`
- `decision.status`
- `decision.executable_now`
- `execution_plan.instruction`
- `execution_plan.trigger`
- `risk.invalidation_level`
- `risk.recommended_risk_pct`
- `targets.primary_target`
- `targets.expansion.probability`
- `market_state.price`
- `market_state.risk_range`
- `quality.data_quality`
- `quality.history_robustness`
- `venue`
- `market_type`
- `quote_asset`
- `disclaimer`

AntSeed buyer curl:

```bash
curl -X POST "$ANTSEED_OPENAI_BASE_URL/v1/chat/completions" \
  -H "Authorization: Bearer $ANTSEED_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "riskranges-binance-fractal-signals-v1",
    "messages": [
      {
        "role": "user",
        "content": "LINKUSDT"
      }
    ],
    "temperature": 0,
    "stream": false
  }'
```

## Service 2: Multi-Symbol Rankings

Model ID:

```text
riskranges-binance-rankings-v1
```

Use this service when an agent needs a ranked table of current signals across multiple Binance spot USDT symbols.

What it does:

- Accepts a ranking request, a plain-text list of symbols, or a generic ranking prompt.
- Scans Binance spot USDT symbols with the existing RiskRangesOS ranking engine.
- Returns compact JSON with ranked symbols, actions, instructions, trigger/invalidation/target levels, expansion probabilities, ranking scores, latency diagnostics, per-symbol errors, and disclaimer.

Default ranking universe:

```text
BTCUSDT, ETHUSDT, SOLUSDT, LINKUSDT, BNBUSDT, XRPUSDT, DOGEUSDT, AVAXUSDT, SUIUSDT, ENAUSDT
```

Input examples:

```text
top crypto signals
rankings
BTCUSDT ETHUSDT SOLUSDT
```

JSON input example:

```json
{"symbols":["BTCUSDT","ETHUSDT","SOLUSDT"],"limit":3}
```

Optional request fields:

- `symbols`: list of Binance spot USDT symbols.
- `limit`: number of ranked rows to return.
- `mode`: currently `full`.
- `use_partial_current_day`: boolean.

Expected output fields include:

- `ok`
- `service_id`
- `generated_at_utc`
- `universe.symbols_requested`
- `universe.symbols_scanned`
- `universe.symbols_failed`
- `rankings[].rank`
- `rankings[].symbol`
- `rankings[].action`
- `rankings[].status`
- `rankings[].instruction`
- `rankings[].trigger`
- `rankings[].invalidation_level`
- `rankings[].primary_target`
- `rankings[].expansion_probability`
- `rankings[].ranking_score`
- `rankings[].venue`
- `rankings[].market_type`
- `rankings[].quote_asset`
- `latency.total_ms`
- `latency.per_symbol_stages`
- `errors`
- `disclaimer`

AntSeed buyer curl, default universe:

```bash
curl -X POST "$ANTSEED_OPENAI_BASE_URL/v1/chat/completions" \
  -H "Authorization: Bearer $ANTSEED_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "riskranges-binance-rankings-v1",
    "messages": [
      {
        "role": "user",
        "content": "top crypto signals"
      }
    ],
    "temperature": 0,
    "stream": false
  }'
```

AntSeed buyer curl, custom universe:

```bash
curl -X POST "$ANTSEED_OPENAI_BASE_URL/v1/chat/completions" \
  -H "Authorization: Bearer $ANTSEED_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "riskranges-binance-rankings-v1",
    "messages": [
      {
        "role": "user",
        "content": "{\"symbols\":[\"BTCUSDT\",\"ETHUSDT\",\"SOLUSDT\"],\"limit\":3}"
      }
    ],
    "temperature": 0,
    "stream": false
  }'
```

## Developer Notes

Both services use the OpenAI-compatible response envelope:

```json
{
  "object": "chat.completion",
  "model": "riskranges-binance-rankings-v1",
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "{...compact JSON...}"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 1,
    "completion_tokens": 1,
    "total_tokens": 2
  }
}
```

Parse `choices[0].message.content` as JSON. The API does not stream responses.

Public discovery endpoints:

- `GET /metadata`
- `GET /metadata/docs`
- `GET /symbols`
- `GET /v1/models`

Protected execution endpoints require `Authorization: Bearer <API key>`.

