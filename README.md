# ETH Gas Live

![ETH Gas Live dashboard](assets/live-dashboard.png)

**Live Ethereum gas intelligence** on Logic Encoder — see what a send costs right now, compare three practical send tiers, and decide whether to submit or wait. Open [logicencoder.com/ethereum-gas-tracker/](https://logicencoder.com/ethereum-gas-tracker/) in the browser. Built for senders, swappers, and NFT minters who want more than a wallet’s single high/medium/low guess — especially when timing a transaction around congestion spikes.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| WordPress plugin | PHP, WordPress REST API, shortcodes, wp-admin settings, LiteSpeed cache bypass |
| Public SPA | HTML, CSS, vanilla JavaScript (`gas_tracker.html`), Tailwind CSS, Chart.js |
| Live backend | Python 3, FastAPI, uvicorn, web3.py, aiohttp, websockets, pydantic, orjson |
| SEO SSR | Node.js, Express (`ssr-server.js`) |
| Data | WordPress MySQL (options/transients), SQLite (gas history on backend) |
| Realtime | WebSocket (`/ws/gas`), REST push ingest with API key, REST mirror on WordPress |
| Networking | Cloudflare tunnel, Ethereum JSON-RPC / mempool feeds |
| Hosting | WordPress on shared hosting; Python and Node on self-hosted Linux servers |

## Live dashboard

The main app is a full-screen **gas tracker** with tabbed views (Live App, Today, Why High?, Best Time, Calculator, Tx Costs, Swap Fees, Network, History, Percentiles). Data refreshes over **WebSocket** when available, with REST fallback so the page stays usable on strict networks.

### Three send tiers

**Base Route**, **Standard Way**, and **Faster Inclusion** — each card shows gwei, priority component, ETH and USD estimates, and a plain-language confirmation hint.

![Three send tiers — Base Route, Standard Way, Faster Inclusion](assets/send-tiers.png)

### Network statistics

Transactions per minute, congestion status, block utilization, fee competition, IPI/SPIKE scores, and trend indicators — so you see *why* fees moved, not only the number.

![Network statistics grid on the Live App tab](assets/network-statistics.png)

### Gas Intelligence Hub

Short **send / wait guidance** when fees are elevated vs the rolling average, plus **smart timing** (best and worst hours) and a **network health score** in one panel.

![Gas Intelligence Hub — send recommendation, timing, and network health](assets/gas-intelligence-hub.png)

### Fee calculator

Enter a gas limit (presets or manual), pick a tier or custom gwei, and get an instant ETH and USD estimate.

![Fee calculator with tier presets and live Standard tracking](assets/fee-calculator.png)

### Common transaction costs

Real-time estimates for **ETH transfer**, **token transfer**, and **DEX swap** at all three send tiers — useful when you know the action but not the gas units.

![Common transaction costs for transfer and swap actions](assets/transaction-costs.png)

### Gas price history

Selectable ranges (1h through 30d) chart **Base Route**, **Standard**, and **Faster Inclusion** alongside average block utilization, with FIP percentile readouts.

![Gas price history chart with tier lines and utilization](assets/gas-price-history.png)

### Gas heatmap

Hourly average **Standard Way** gas over the last eight days — color-coded so you spot cheap windows at a glance (timezone follows the browser).

![Gas heatmap — hourly Standard Way averages by day](assets/gas-heatmap.png)

### Advanced statistics

Three in-app tabs — **Overview** (current/min/max/avg/median per tier), **Detailed** (percentiles, base fee, live tx throughput, sample quality), and **Time Analysis** (best/worst hour plus hourly averages).

![Advanced statistics — Overview tab](assets/advanced-stats-overview.png)

![Advanced statistics — Detailed tab](assets/advanced-stats-detailed.png)

![Advanced statistics — Time Analysis tab](assets/advanced-stats-time-analysis.png)

## SEO topic pages

Eleven indexable URLs on logicencoder.com are filled with **real-time SSR data**, not static marketing copy. Examples:

- [Ethereum gas fees today](https://logicencoder.com/ethereum-gas-fees-today/)
- [Best time to send Ethereum](https://logicencoder.com/best-time-to-send-ethereum/)
- [Ethereum gas calculator](https://logicencoder.com/ethereum-gas-calculator/)
- [Ethereum mempool tracker](https://logicencoder.com/ethereum-mempool-tracker/)
- [Ethereum network status](https://logicencoder.com/ethereum-network-status/)

Each page targets a specific search intent while linking back to the main tracker. Embed mode (`?gt_embed=1`) serves a chromeless view for in-app panels.

## WordPress embedding

Shortcode **`[eth_gas_dashboard]`** drops the same dashboard shell into any WordPress page. The plugin handles routing, cache-friendly delivery, and a REST mirror of the latest payload — **gas math runs on the backend**, not in PHP.

## Site operator tools

wp-admin **ETH Gas Live** includes Mission Control: backend health (WebSocket clients, fetch/push stats, database/cache indicators), configurable API base URL, WebSocket URL, SSR base, push key, and refresh interval — so operators can confirm the live site is fed without touching code.

![Mission Control — runtime, WebSocket, fetch/push, and database health](assets/mission-control.png)

Private code: [eth-gas-live-plugin](https://github.com/logicencoder/eth-gas-live-plugin) · live data [eth-gas-live-backend](https://github.com/logicencoder/eth-gas-live-backend)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
