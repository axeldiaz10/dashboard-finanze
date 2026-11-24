# dashboard-finanze

Personal dashboard for a paper-trading portfolio (Alpaca practice account).

- `index.html` — the dashboard page. Live prices come from Alpaca's WebSocket stream; positions, orders, fills and account history from Alpaca's REST API. API keys are entered in the page and stored only in the browser's `localStorage` — never commit them.
- `feed.json` — analysis data (profiles, cash per profile, news, catalysts, predictions, watchlist) written by scheduled Claude tasks. No credentials.

Practice account only. Nothing here is investment advice.
