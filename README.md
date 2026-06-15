# ETH Gas Live

![ETH Gas Live dashboard](assets/live-dashboard.png)

**Live Ethereum gas intelligence** on Logic Encoder — see what a send costs right now, compare three practical send tiers, and decide whether to submit or wait. Open [logicencoder.com/ethereum-gas-tracker/](https://logicencoder.com/ethereum-gas-tracker/) in the browser. Built for senders, swappers, and NFT minters who want more than a wallet’s single high/medium/low guess — especially when timing a transaction around congestion spikes, listing mints, or bridge hops where a few gwei difference is real money at scale.

The header shows **connection status**, **last update time**, and **live ETH/USD** so you know the numbers are fresh. The page title and favicon react to current Standard gwei so a pinned tab doubles as an at-a-glance fee indicator without opening the full dashboard.

**Typical flows:** before a wallet send, open Live App → compare your wallet’s “medium” quote to **Standard Way** → if **Send Recommendation** says wait and **IPI** is high, defer or drop to **Base Route**. Planning a DEX swap → **Fee calculator** preset **Swap (DEX)** → cross-check **Common transaction costs** for approve + swap totals. Scheduling a treasury payout → **Gas heatmap** + **Best Time (Last 24h)** → set an **UNDER** alert and go do something else until the browser notification fires. Fees jumped on Twitter → **Network statistics** (tx/min + utilization) → **Why fees high** in the topic nav without losing the live socket. Site feels stale → wp-admin **Mission Control** → **Fetch + Push** card and last push timestamp on the WP mirror footer.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| WordPress plugin | PHP, WordPress REST API, shortcodes, wp-admin Mission Control, LiteSpeed cache bypass |
| Public SPA | HTML, CSS, vanilla JavaScript (`gas_tracker.html`), Tailwind CSS, Chart.js |
| Live backend | Python 3, FastAPI, uvicorn, web3.py, aiohttp, websockets, pydantic, orjson, SQLite |
| SEO data | WordPress local HTML templates with `{{PLACEHOLDER}}` keys filled from backend `GET /api/gas/ssr-data`; Node.js Express SSR on the backend for proxied crawler routes |
| Data | WordPress MySQL (options/transients), SQLite gas history (~30 days) on the backend |
| Realtime | WebSocket (`/ws/gas`), REST bootstrap and analytics endpoints, backend push to WordPress transient mirror |
| Networking | Cloudflare tunnel, Ethereum JSON-RPC / mempool feeds |
| Hosting | WordPress on shared hosting; Python and Node on self-hosted Linux servers |

## Live dashboard

The main app is a full-screen **gas tracker** with the **Live App** tab as home base. Data refreshes over **WebSocket** on every new block when the browser can connect; if a corporate network blocks WS, REST polling plus the WordPress transient mirror still updates headline tiers every few seconds. On first load the SPA bootstraps from REST so tier cards paint before the socket handshake completes.

Backend gas math (three tiers, IPI, heatmap buckets, featured-action costs) never runs in PHP — WordPress serves the shell, injects API and WebSocket URLs, and mirrors the latest payload for fallback readers.

### Topic navigation and in-app guides

Below the header, a horizontal **topic nav** keeps you inside one app shell instead of cold-loading eleven separate articles.

- **Live App** — the realtime dashboard (network grid, tiers, charts, sidebar tools).
- **Topic tabs** — fees today, best time to send, mempool, calculator, transaction costs, swap fees, network status, price history, percentiles, and related guides. Labels come from the backend SEO hub manifest; clicking a tab loads that page’s HTML into an **in-app panel** via embed mode (`?gt_embed=1`) while live gas fields stay wired to the same WebSocket stream.
- **Back to Live App** — one click hides the guide panel and restores the full dashboard.

Each embedded guide is cached briefly in the browser so switching tabs feels instant; headline numbers inside the panel still sync from the live gas object. Crawlers and direct URL visitors get the same content as full WordPress pages with theme chrome — the in-app panel is the reader-friendly shortcut for people already on the tracker.

**Example:** fees spike on social media — stay on **Live App**, click **Why fees high** or **Mempool** in the topic nav, read the explainer in the in-panel mount while tier cards keep updating; click **Live App** again to send when **Send Recommendation** flips favorable.

### Three send tiers

Ethereum post-EIP-1559 pricing is not one number — senders trade off cost vs inclusion speed. The product surfaces **three named tiers** instead of opaque wallet labels:

| Tier | Role |
|------|------|
| **Base Route** | Lowest practical fee — targets economical sends when the network is calm and you can wait an extra block or two. |
| **Standard Way** | Balanced default — the tier most users should compare against wallet “medium” estimates. |
| **Faster Inclusion** | Priority-heavy path when mempools are busy, NFT mints are competitive, or you need the next block. |

Each card shows **gwei** (base + priority breakdown), **ETH and USD estimates** for a reference transfer, and a plain-language **confirmation hint** (“~1 block”, “may take several blocks”, etc.). Cards pulse-update on every WebSocket tick so you watch fees move during a congestion spike without refreshing.

When Standard is elevated vs its rolling average, the **Gas Intelligence Hub** (sidebar) nudges you toward wait-or-send guidance rather than leaving you to guess from a single red number.

**Example:** your wallet shows three opaque speeds — map **low / medium / high** to **Base Route / Standard Way / Faster Inclusion** here first. Competitive NFT mint with a short window → watch **Faster Inclusion** and **fee competition**; routine ERC-20 send when **network status** is Normal → **Base Route** is usually enough if the confirmation hint allows an extra block.

![Three send tiers — Base Route, Standard Way, Faster Inclusion](assets/send-tiers.png)

### Network statistics

Above the tier cards, a **network statistics grid** answers *why* fees moved — not only *what* they are. Metrics include:

- **Tx / minute** — estimated from recent blocks; primary activity signal when mempool depth is noisy.
- **Network status** — Normal / Elevated / High / Spike labels derived from backend stress scoring.
- **Trend (Standard)** — short-term direction vs the last hour so you see fees climbing before they peak.
- **Last block** — anchor for staleness checks.
- **Avg / current block size and utilization** — how full blocks are; high utilization usually means higher competition for space.
- **IPI (Inclusion Pressure Index) 0–100** — single stress score aligned with the Intelligence Hub.
- **SPIKE score** and **fee competition** — how aggressive other senders are right now (priority spread between high and median tips).
- **Block speed pressure** — share of recent blocks above 90% full, affecting queue drain rate.

Use this panel when Standard jumped but you are unsure if it is a blip (one fat block) or sustained load (climbing tx/min + high utilization).

**Example:** Standard spiked in the last ten minutes — if **tx/min** and **current utilization** climbed together and **SPIKE** is elevated, treat it as real congestion; if only **last block** is fat while averages stay calm, wait one block before overpaying on **Faster Inclusion**.

![Network statistics grid on the Live App tab](assets/network-statistics.png)

### Gas Intelligence Hub

The sidebar **Gas Intelligence Hub** compresses backend insight into one scrollable panel — for mobile readers who will not expand four charts before sending.

**Insight** — one-sentence network readout tied to current IPI and spike state (plain language, not raw numbers only).

**Smart Timing Insights** — **Best Time (Last 24h)** and **Worst Time (Last 24h)** cards with hour labels and average Standard gwei for each window. Use them to defer non-urgent sends to historically cheaper hours.

**Send Recommendation** — actionable badge (send now vs consider waiting) with supporting copy, plus a comparison strip: **current Standard gwei and USD** vs **24h average**, and a **trend** line showing how far today sits from the mean. This is the same logic behind the one-line “send / wait” guidance, expanded for readers who want numbers behind the verdict.

**Network Health Score** — **Current Network Stress** as a 0–100 score with a gradient bar (green → yellow → red) and a short advice line below. Pairs with IPI in the network grid — the hub version is optimized for quick glances on narrow screens.

The hub recomputes on every payload tick using the same backend scoring as charts and SEO pages, so guidance stays consistent across panels.

**Example:** **Send Recommendation** shows “consider waiting”, **current Standard** sits above **24h average**, and **Worst Time (Last 24h)** is still hours away — batch your non-urgent transfers for the **Best Time** window instead of submitting now on **Standard Way**.

![Gas Intelligence Hub — send recommendation, timing, and network health](assets/gas-intelligence-hub.png)

### Fee calculator

The **fee calculator** answers “how much will *my* transaction cost?” — not just the reference transfer on the tier cards.

- **Gas limit presets** — ETH transfer, ERC-20 transfer, DEX swap, token approve, NFT mint, NFT sale; or type a custom limit if you know units from a contract simulation.
- **Tier selector** — price from Base Route, Standard Way, Faster Inclusion, or **custom gwei** for what-if scenarios; “Auto (Standard)” tracks live Standard when you do not want to babysit the field.
- **Live output** — instant ETH and USD fee estimate updating with each WebSocket tick when a tier is selected.

Power users cross-check wallet simulation quotes; newcomers use presets so they do not have to know that a swap is ~180k gas.

**Example:** MetaMask quotes a swap — pick **Swap (DEX)** preset, leave tier on **Auto (Standard)**, read **Estimated Cost** in USD; switch tier to **Faster Inclusion** to see the cost of jumping the queue if the swap is time-sensitive.

![Fee calculator with tier presets and live Standard tracking](assets/fee-calculator.png)

### Common transaction costs

The **featured transaction costs** list shows real-time estimates for **nineteen action types** at **all three tiers** side by side — ETH transfer, token transfer, approve, DEX swap, NFT mint/sale, bridging, borrowing, lending, staking, unstaking, claim rewards, compound, liquidity add/remove, governance vote, multi-send, contract creation, and contract deployment.

Each row displays the action name, **gas limit** used for the estimate, and **Base / Standard / Faster** columns in both USD and gwei. Useful when planning multi-step workflows (approve + swap + bridge) where you only care about total dollars, not manual gwei math. The scrollable list keeps priority actions (transfer, swap, approve) at the top.

**Example:** three-step DeFi exit — note **Token Approve**, **Swap (DEX)**, and **Bridging** rows at **Standard Way** USD, add the three numbers for a rough total before you open the wallet; if the sum hurts, check whether **Base Route** on the approve step is acceptable while keeping **Faster** only on the swap step.

![Common transaction costs for transfer and swap actions](assets/transaction-costs.png)

### Gas price history

The **history chart** plots **Base Route**, **Standard Way**, and **Faster Inclusion** over selectable ranges — **1h, 3h, 6h, 12h, 24h, 3 days, 7 days, 30 days**. A secondary axis overlays **average block utilization** so you correlate fee spikes with full blocks.

**Smoothing** — Raw, EMA (5), Median (5), or Clamp 99% for noisy priority markets. Short ranges (≤6h) stream live over WebSocket; longer ranges load from SQLite history on demand.

**Faster-tier percentile band** — **FIP** readouts (P50, P70, and related chips in the chart footer) show how volatile the priority market is — wide spread means tips swing block to block. Ties back to the Percentiles SEO page for search landings.

**Example:** during a spike, set range to **1 Hour**, smoothing **Raw**, watch **Standard** vs **utilization** — if both spike together it is block pressure; switch to **7 Days** afterward to see whether today is an outlier vs the weekly band.

![Gas price history chart with tier lines and utilization](assets/gas-price-history.png)

### Gas heatmap

The **heatmap** colors **hourly average Standard Way gas** over the last **seven days** (thirty days when the history range selector is on the 30-day window). Darker cells = cheaper hours; bright cells = expensive windows.

Hours shift to **your browser timezone** — a European evening cheap window displays in local time, not UTC. Use it to schedule non-urgent sends (treasury payouts, batch claims) without setting a manual alarm. Hover cells show the exact hour and average gwei behind each color.

**Example:** you pay payroll every Tuesday — find the consistently dark (cheap) cells on the same weekday row, line them up with **Best Time (Last 24h)** in the Intelligence Hub, and submit in that window using **Base Route** unless **IPI** says otherwise.

![Gas heatmap — hourly Standard Way averages by day](assets/gas-heatmap.png)

### Advanced statistics

Three in-app sub-tabs for researchers who want tables, not only charts:

**Overview** — current, min, max, average, and median gwei per tier for the selected history window. Quick answer to “how bad was today vs the week?”

**Detailed** — percentile breakdowns (P10/P50/P90), base fee stats, live tx throughput, sample quality indicators, and **IPI** history snippets. Use when writing up network conditions or verifying a spike was real in the data.

**Time Analysis** — best and worst hour plus a full **hourly averages** table for the period. Pairs with the heatmap for exact numbers behind the colors.

**Example:** writing a “gas post-mortem” after a busy day — **Detailed** tab for P90 vs P50 spread, **Time Analysis** for the exact worst hour to cite, **Overview** for min/max/median per tier in one glance.

![Advanced statistics — Overview tab](assets/advanced-stats-overview.png)

![Advanced statistics — Detailed tab](assets/advanced-stats-detailed.png)

![Advanced statistics — Time Analysis tab](assets/advanced-stats-time-analysis.png)

### Custom alerts

The **Gas Alerts** panel lets you set a **gwei threshold** and choose **Alert when gas goes OVER threshold** or **Alert when gas goes UNDER threshold** — useful for “notify me when fees drop enough to batch payouts” or “warn me before fees spike past my budget.”

Alerts are stored per **browser session** on the backend; the list shows active rules with delete actions. When a rule fires, the UI shows an on-screen toast and, if you granted permission, a **browser notification** with current vs threshold gwei and USD equivalents. The WebSocket stream delivers alert events in realtime so you do not need to keep the tab focused.

**Example:** you will not babysit the tab — allow browser notifications, set **UNDER** at your target Standard gwei, keep the tab open in the background (or pinned); when fees drop, submit the batch payout from the wallet using **Base Route** or **Standard Way** as **Send Recommendation** suggests.

## SEO topic pages

Eleven indexable URLs on logicencoder.com are filled with **live data**, not static marketing copy. WordPress serves local HTML templates; placeholders are replaced from the backend SSR data bundle (refreshed on a short transient cache). Each page targets a search intent and links back to the main tracker:

| Page | URL | Intent |
|------|-----|--------|
| Gas hub | [ethereum-gas](https://logicencoder.com/ethereum-gas/) | Overview landing |
| Fees today | [ethereum-gas-fees-today](https://logicencoder.com/ethereum-gas-fees-today/) | “What do fees look like right now?” |
| Why fees high | [why-are-ethereum-gas-fees-high](https://logicencoder.com/why-are-ethereum-gas-fees-high/) | Explainers during spikes |
| Best time | [best-time-to-send-ethereum](https://logicencoder.com/best-time-to-send-ethereum/) | Scheduling sends |
| Calculator | [ethereum-gas-calculator](https://logicencoder.com/ethereum-gas-calculator/) | Landings for “gas calculator” queries |
| Tx costs | [ethereum-transaction-costs](https://logicencoder.com/ethereum-transaction-costs/) | Per-action comparisons |
| Swap gas | [ethereum-swap-gas-fees](https://logicencoder.com/ethereum-swap-gas-fees/) | DeFi-specific |
| Mempool | [ethereum-mempool-tracker](https://logicencoder.com/ethereum-mempool-tracker/) | Queue / load context |
| Network status | [ethereum-network-status](https://logicencoder.com/ethereum-network-status/) | Health dashboard for search |
| History | [ethereum-gas-price-history](https://logicencoder.com/ethereum-gas-price-history/) | Long-range charts |
| Percentiles | [ethereum-gas-percentiles](https://logicencoder.com/ethereum-gas-percentiles/) | Statistical deep dive |

**Embed mode** (`?gt_embed=1`) serves a chromeless view for in-app panels and third-party iframes without WordPress theme chrome. The retired `/ethereum-gas-tracker-live/` path returns gone — the canonical live URL is `/ethereum-gas-tracker/`.

## WordPress embedding

Shortcode **`[eth_gas_dashboard]`** drops the same dashboard shell into any WordPress page or post. The plugin handles dedicated routing for `/ethereum-gas-tracker/`, **LiteSpeed / full-page cache bypass** on all gas routes (stale gwei on cached HTML is unacceptable), and a REST mirror of the latest payload that the backend can push every block.

Editors embed the live tool inside articles (“check fees below”) without maintaining a separate iframe host. Activation creates the SEO page records and stores their IDs so template routing survives theme changes.

## Site operator tools

wp-admin **ETH Gas Live** includes **Mission Control** — production observability without SSH.

**Runtime card** — backend uptime, host/port, ingest loop timing (last/avg/max ms), last error string. Loop spikes often precede stale public data.

**WebSocket card** — current and max connected clients, total connect/disconnect lifetime counts, messages per minute, broadcast failure counter. If failures climb while clients are high, check network or payload size.

**Fetch + Push card** — RPC fetch success rate, fetch latency (last/avg/max), WordPress push HTTP status and latency. Confirms the Python → WordPress mirror path that keeps REST fallback fresh.

**Database + Cache card** — gas history row count, alert row count, SQLite file sizes (DB/WAL/SHM), history and heatmap cache hit ratios. Low hit ratio after deploy may mean cold cache, not broken ingest.

Three rolling **charts** plot loop cycle ms, fetch latency ms, and WS client count. Expandable **raw monitoring JSON** for copying into tickets. **WordPress cache mirror** footer shows last push timestamp and cached payload key count.

Settings above Mission Control configure API base URL, WebSocket URL, public asset base for CSS rewrite, push API key, and client refresh interval — change endpoints after infra moves without editing code.

**Example:** users report stale tiers but WS shows connected — check **Fetch + Push** for failed WordPress push or rising fetch latency; if **last push timestamp** on the WP mirror footer is old, fix push key or API base URL in settings before touching PHP.

![Mission Control — runtime, WebSocket, fetch/push, and database health](assets/mission-control.png)

## Shared hosting headroom (corroboration)

Gas math and WebSocket fan-out run off shared hosting; WordPress only serves the SPA shell, SEO templates, and the REST mirror. One CPU/memory graph below shows headroom on the live install — corroboration for recruiters, not part of the product UI.

![Shared hosting — CPU and memory usage vs plan limits](assets/hostinger-cpu-memory.jpg)

Private code: [eth-gas-live-plugin](https://github.com/logicencoder/eth-gas-live-plugin) · live data [eth-gas-live-backend](https://github.com/logicencoder/eth-gas-live-backend)

Backend overview: [eth-gas-live-backend-overview](https://github.com/logicencoder/eth-gas-live-backend-overview)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
