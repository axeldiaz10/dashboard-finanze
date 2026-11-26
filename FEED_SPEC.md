# feed.json specification (schema 1)

`feed.json` is the analysis data the dashboard (`index.html`) reads. It is written by the scheduled Claude tasks. The portfolio itself (positions, prices, orders, fills, cash, history) is NOT stored here: the page reads it live from Alpaca.

## Hard rules

- **Never** put API keys, tokens, passwords, account numbers or any credential in this file. The repository is public.
- Keep it valid JSON (check with `python3 -m json.tool feed.json`). Keep the schema; unknown fields are ignored by the page.
- Read the current file first and change only the fields your task owns; keep the rest as they are.
- Dates: `YYYY-MM-DD`. `updated_at`: ISO 8601 UTC. Text in Spanish (rioplatense), short.
- Crypto symbols use Alpaca's slash form: `BTC/USD`, `ETH/USD`, `SOL/USD`.

## Fields

| Field | Type | Content |
|---|---|---|
| `schema` | number | Always `1`. |
| `updated_at` | string | When this write happened (UTC). |
| `updated_by` | string | Task name, e.g. `Pre-apertura 25/11`. |
| `start` | object | `{date, capital, spy, btc}` — portfolio start and benchmark prices. Do not change. |
| `profiles` | object | `{Conservador|Moderado|Agresivo: {allocated, cash, stop_pct, realized}}`. `cash` = the profile's cash from the logbook (Resumen D9:D11). `realized` = realized profit/loss in USD of the profile's closed trades since the start (0 if none). The page shows each profile as `allocated + realized + unrealized P/L of its open positions`; the gap with the account total is shown as fees and adjustments. |
| `assets` | object | One entry per open or pending position: `{symbol: {profile, market, name, conviction, thesis}}`. `thesis` is one line. Remove assets that are fully closed and have no pending order. |
| `headline` | string | One line: the most important thing right now for the portfolio. |
| `news` | array | Up to 3 items, most important first: `{title, summary, why, url, source, assets}`. `summary` 2-3 lines; `why` = why it matters for our portfolio; `url` must be an `https://` link to the original article; `assets` = related symbols. |
| `news_date` | string | Date of the news selection. |
| `catalysts` | array | Upcoming (and recently resolved) events: `{id, date, time, event, impact (Alto/Medio/Bajo), assets, scenario, scenarios, plan, status (Próximo/Resuelto)}`. `time` in Argentina time as `HH:MM` when known (the page counts down to it). `scenario` = most likely scenario with its probability, as text. `scenarios` = the full set for Alto/Medio events: `[{label, prob (0-1), tone (bueno/malo/neutro, from the portfolio's point of view)}]`, probabilities summing to 1, labels short and plain (e.g. "Más alta (malo)"). `plan` = the pre-written action plan. Keep at most ~15, drop resolved ones older than 7 days. |
| `planned_orders` | array | Orders scheduled in the logbook but not yet sent to Alpaca (status Pendiente): `{date, asset, profile, side (Compra/Venta), type, qty, limit}`. Empty array when there are none. |
| `watchlist` | array | Radar candidates in Seguimiento with status Observando: `{asset, name, trigger, status, state, levels, conditions, note}`. `trigger` = the entry trigger as one line of text. `state` = `esperando`, `activado` (all conditions met; the order is being prepared) or `descartada`; the page itself shows "Cerca del gatillo" when the live price is within 5% of `levels.trigger`. `levels` = `{trigger, zone: [low, high] or omitted, target}` in USD: the breakout price, the buy-on-pullback zone if any, and the prediction target. `conditions` = `[{text, met}]`, each a short checkable condition (e.g. "Cierre > 17,30", "con volumen alto", "Brent > 85"). `note` = optional one line for news that changes the picture. |
| `predictions` | object | `{open: [{id, asset, direction (Sube/Baja), entry, target, due, probability (0-1)}], closed, hits, brier}`. `brier` = mean of (probability − outcome)² over closed predictions, or `null` if none closed. |
| `sentiment` | object | Market mood, 0 to 100 (0 extreme fear, 100 extreme greed). `{stocks: {value, label, source, as_of, history: [{date, value}]}, crypto: {same}, note}`. `stocks` = CNN Fear & Greed (read it from https://production.dataviz.cnn.io/index/fearandgreed/graphdata: `fear_and_greed.score` and the last 30 daily points of `fear_and_greed_historical.data`). `crypto` = alternative.me (https://api.alternative.me/fng/?limit=30); the page also reads it live, so this is the fallback. `note` = one line linking the mood to our portfolio (what it means for our positions or stops), or empty. Labels: <25 Miedo extremo, 25-44 Miedo, 45-55 Neutral, 56-75 Codicia, >75 Codicia extrema. |
| `phase1` | string | Phase 1 summary. Do not change. |
| `sheet_url` | string | Link to the Claude logbook sheet. Do not change. |

## How to publish

1. Attach the repository with push access (`add_repo`: owner `axeldiaz10`, repo `dashboard-finanze`, access `push`) and clone it as instructed.
2. Edit `feed.json`, validate it, then commit with message `Update feed (<task> <date>)` and push to `main` with `git -c push.negotiate=true push origin HEAD:main`.
3. If the push is rejected because the remote moved, `git pull --rebase origin main` and push once more. If it still fails, say so in one line of the task's final message. Never force-push.

## Who updates the dashboard v2 fields

Every task that publishes the feed must also keep these fields current, in addition to what its own prompt lists:

- **Pre-apertura and Cierre**: `sentiment` (both indexes, with their last 30 points, and `note`); `watchlist[].state`, `conditions[].met` and `note` with the latest prices and news; `catalysts[].scenarios` and `time` for any Alto/Medio event in the next 5 business days.
- **Planificación (Sunday) and Revisión semanal (Friday)**: `watchlist` complete for every candidate (`levels`, `conditions`, `state`, `note`), including `state: "descartada"` when a thesis breaks (remove it from the list the following week); `catalysts[].scenarios` for every Alto/Medio event; `profiles[].realized`.
- A candidate whose conditions are all met gets `state: "activado"`, and the task that detects it schedules the entry order in the logbook following the Reglas.
