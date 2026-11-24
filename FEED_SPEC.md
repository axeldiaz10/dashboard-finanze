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
| `profiles` | object | `{Conservador|Moderado|Agresivo: {allocated, cash, stop_pct}}`. `cash` = the profile's cash from the logbook (Resumen D9:D11). |
| `assets` | object | One entry per open or pending position: `{symbol: {profile, market, name, conviction, thesis}}`. `thesis` is one line. Remove assets that are fully closed and have no pending order. |
| `headline` | string | One line: the most important thing right now for the portfolio. |
| `news` | array | Up to 3 items, most important first: `{title, summary, why, url, source, assets}`. `summary` 2-3 lines; `why` = why it matters for our portfolio; `url` must be an `https://` link to the original article; `assets` = related symbols. |
| `news_date` | string | Date of the news selection. |
| `catalysts` | array | Upcoming (and recently resolved) events: `{id, date, time, event, impact (Alto/Medio/Bajo), assets, scenario, plan, status (Próximo/Resuelto)}`. `scenario` = most likely scenario with its probability; `plan` = the pre-written action plan. Keep at most ~15, drop resolved ones older than 7 days. |
| `planned_orders` | array | Orders scheduled in the logbook but not yet sent to Alpaca (status Pendiente): `{date, asset, profile, side (Compra/Venta), type, qty, limit}`. Empty array when there are none. |
| `watchlist` | array | Candidates in Seguimiento with status Observando: `{asset, name, trigger, status}`. |
| `predictions` | object | `{open: [{id, asset, direction (Sube/Baja), entry, target, due, probability (0-1)}], closed, hits, brier}`. `brier` = mean of (probability − outcome)² over closed predictions, or `null` if none closed. |
| `phase1` | string | Phase 1 summary. Do not change. |
| `sheet_url` | string | Link to the Claude logbook sheet. Do not change. |

## How to publish

1. Attach the repository with push access (`add_repo`: owner `axeldiaz10`, repo `dashboard-finanze`, access `push`) and clone it as instructed.
2. Edit `feed.json`, validate it, then commit with message `Update feed (<task> <date>)` and push to `main` with `git -c push.negotiate=true push origin HEAD:main`.
3. If the push is rejected because the remote moved, `git pull --rebase origin main` and push once more. If it still fails, say so in one line of the task's final message. Never force-push.
