# ETH Gas Live

![ETH Gas Live dashboard](assets/live-dashboard.png)

**Live Ethereum gas intelligence** on Logic Encoder — see what a send costs right now, compare three practical send tiers, and decide whether to submit or wait. Open [logicencoder.com/ethereum-gas-tracker/](https://logicencoder.com/ethereum-gas-tracker/) in the browser. Built for senders, swappers, and NFT minters who want more than a wallet’s single high/medium/low guess — especially when timing a transaction around congestion spikes, listing mints, or bridge hops where a few gwei difference is real money at scale.

The header shows **connection status**, **last update time**, and **live ETH/USD** so you know the numbers are fresh. The page title and favicon react to current Standard gwei so a pinned tab doubles as an at-a-glance fee indicator without opening the full dashboard.

**Typical flows:** scheduling a non-urgent payout → **Gas heatmap** for cheap weekday/hour pattern → confirm with **Gas price history** on **7 Days** → set **UNDER** alert. Live spike on Twitter → **Network statistics** + **1 Hour** chart (**Raw**) → **Why fees high** in topic nav. Before a wallet send → compare wallet “medium” to **Standard Way** → **Send Recommendation** in the Intelligence Hub. DEX swap → **Fee calculator** + **Common transaction costs** for approve + swap totals. Site stale → wp-admin **Mission Control** → **Fetch + Push** card.

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
- **Topic tabs** — **Today**, **Why High?**, **Best Time**, **Calculator**, **Tx Costs**, **Swap Fees**, **Network**, **History**, **Percentiles** (plus related guides from the SEO hub manifest). Labels come from the backend; clicking a tab loads that page’s HTML into an **in-app panel** via embed mode (`?gt_embed=1`) while live gas fields stay wired to the same WebSocket stream.
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

Each card shows **gwei** (base + priority breakdown), **ETH and USD estimates** for a reference transfer, and a plain-language **confirmation hint** in the top-right corner — typically **~12s (1 block)** for Standard and Faster, **~24s (2 blocks)** for Base Route when the network is calm. Below the headline gwei, **Prio**, **ETH**, and **USD** rows update on every WebSocket tick so you watch fees move during a congestion spike without refreshing.

When Standard is elevated vs its rolling average, the **Gas Intelligence Hub** (sidebar) nudges you toward wait-or-send guidance rather than leaving you to guess from a single red number.

**Example:** your wallet shows three opaque speeds — map **low / medium / high** to **Base Route / Standard Way / Faster Inclusion** here first. Competitive NFT mint with a short window → watch **Faster Inclusion** and **fee competition**; routine ERC-20 send when **network status** is **LOW** or **NORMAL** → **Base Route** is usually enough if the **~24s (2 blocks)** hint is acceptable.

![Three send tiers — Base Route, Standard Way, Faster Inclusion](assets/send-tiers.png)

### Network statistics

Above the tier cards, a **network statistics grid** sits under the topic nav bar — twelve KPI cells in a responsive grid. Metrics include:

- **Tx / minute** — estimated from recent blocks; primary activity signal when mempool depth is noisy.
- **Network status** — **LOW**, **NORMAL**, **HIGH**, or **SPIKE** labels derived from backend stress scoring (green **LOW** when conditions are quiet).
- **Trend (Standard)** — short-term direction with arrow and gwei delta vs the last hour.
- **Last block** — anchor for staleness checks.
- **Avg / current block size and utilization** — gas used vs block limit, including **% of limit** sublabels on current block.
- **IPI (Inclusion Pressure Index) 0–100** — single stress score aligned with the Intelligence Hub.
- **SPIKE score** and **fee competition** — short-term regime vs baseline; **GWEI (P90−P50 tip)** spread for how aggressive other senders are.
- **Block speed pressure** — share of recent blocks above 90% full, affecting queue drain rate.

Use this panel when Standard jumped but you are unsure if it is a blip (one fat block) or sustained load (climbing tx/min + high utilization).

**Example:** Standard spiked in the last ten minutes — if **tx/min** and **current utilization** climbed together and **SPIKE** is elevated, treat it as real congestion; if only **last block** is fat while averages stay calm, wait one block before overpaying on **Faster Inclusion**.

![Network statistics grid on the Live App tab](assets/network-statistics.png)

### Gas price history

The **history chart** sits in the main column directly under the tier cards — the first place to look when today’s fees feel wrong compared to the last hour or the last week. It plots **Base Route**, **Standard Way**, and **Faster Inclusion** over selectable ranges — **1h, 3h, 6h, 12h, 24h, 3 days, 7 days, 30 days**. A secondary axis overlays **average block utilization** so you correlate fee spikes with full blocks.

**Smoothing** — **Raw**, **EMA (5)**, **Median (5)**, or **Clamp 99%** for noisy priority markets. Short ranges (≤6h) stream live over WebSocket; longer ranges load from SQLite history on demand.

**Faster-tier percentile band** — **FIP** readouts (P50, P70, and related chips in the chart footer) show how volatile the priority market is — wide spread means tips swing block to block. Ties back to the **Percentiles** topic tab and SEO page for statistical depth.

**Example — is this spike normal?** Fees jumped in the last half hour → set **1 Hour** + **Raw**, watch **Standard** climb alongside **Avg Utilization** → real congestion. Switch to **7 Days** → if today’s band sits above the weekly cloud, wait or use **Base Route**; if only the last hour is hot, one block of patience often enough.

**Example — mint or bridge deadline:** Open **3 Hours** + **Raw**, watch **Faster Inclusion** vs **Standard** spread widen → mempool is competitive; if the spread is flat, **Standard Way** usually lands without **Faster** pricing. Cross-check **fee competition** in the network grid before you overpay in the wallet.

**Example — batch treasury send:** Set **24 Hours** or **7 Days**, find where **Standard** spent most of its time below your comfort line → line that window up with the **heatmap** below and submit the batch in that band using **Base Route** unless **IPI** is elevated.

![Gas price history chart with tier lines and utilization](assets/gas-price-history.png)

### Gas heatmap

Directly under the history chart, the **heatmap** answers *when* to send — not only *how much* right now. It colors **hourly average Standard Way gas** over the last **seven days** (thirty days when the history range selector is on the **30 Days** window). **Darker cells** = cheaper hours; **brighter cells** = expensive windows.

Hours shift to **your browser timezone** — a cheap European evening shows in local time, not UTC. Hover a cell for the exact hour and average gwei behind the color.

**Example — weekly payroll:** Find the weekday row you usually pay on → note the consistently dark column (same local hour each week) → open the wallet in that window with **Base Route** → skip bright columns that match US/EU business-hour peaks.

**Example — “I'll send when it's cheap” without staring at the tab:** Pick your target hour band from the heatmap → set an **UNDER** alert near the top of that band → allow browser notifications → when the alert fires, confirm on the **1 Hour** chart that fees are still in the cheap band, then submit.

**Example — heatmap + Intelligence Hub:** Dark cell on the heatmap but **Send Recommendation** still says wait → check **IPI** and **Worst Time (Last 24h)** — historical cheap hour does not override an active spike; defer until the hub flips or **SPIKE** drops in the network grid.

![Gas heatmap — hourly Standard Way averages by day](assets/gas-heatmap.png)

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

### Gas fees today (SEO page)

The **[ethereum-gas-fees-today](https://logicencoder.com/ethereum-gas-fees-today/)** landing is a dated article-style page filled from live backend data — not static copy. Three tables anchor the page:

- **Today’s Gas Statistics** — today’s average, low/high with UTC hour, yesterday comparison, P50/P80, transaction count, avg block utilization.
- **Multi-Period Comparison** — today vs yesterday vs **7-day** and **30-day** averages in one row.
- **Network Intelligence Today** — IPI, spike score, fee competition, block speed pressure, tx/min, network health score, ETH/USD and last-updated footer.

Prose sections below explain how daily patterns work (US/EU business-hour peaks vs overnight lulls). FAQ blocks answer “what are fees today?” in plain language for search snippets.

**Example:** you landed from Google on “gas fees today” — read **Multi-Period Comparison** first: if today sits far above **7-Day Average**, open the live tracker’s **7 Days** chart before sending; click **Live App** in the topic nav to watch tiers update in realtime.

![Gas fees today — statistics tables and network intelligence](assets/seo-gas-fees-today.png)

### Swap gas fees (SEO page)

**[ethereum-swap-gas-fees](https://logicencoder.com/ethereum-swap-gas-fees/)** targets DeFi swap queries. **Swap Costs by Speed** lists Base / Standard / Faster with gwei, swap-only USD, and **total with approve** (first-time DEX users often miss the one-time approval). **Network Intelligence for DeFi** explains IPI, spike score, fee competition, block speed pressure, DeFi activity level, and tx/min with an **Impact** column. **Historical Swap Cost Context** compares 1h / 24h / 7d average swap cost.

Educational sections cover why swaps cost more than transfers (~180k gas), the hidden approve step (~46k gas), and when waiting beats swapping (spike score and IPI thresholds in the copy).

**Example:** planning your first Uniswap trade — read **total with approve** on **Standard Way** before you set slippage; if **Spike Score** impact says consider waiting above 60, defer and set an **UNDER** alert on the live dashboard instead of forcing the swap.

![Swap gas fees — tier table, DeFi intelligence, historical context](assets/seo-swap-gas-fees.png)

### Network status (SEO page)

**[ethereum-network-status](https://logicencoder.com/ethereum-network-status/)** is the health dashboard for “is Ethereum congested?” searches. A **Full Network Intelligence Dashboard** table lists network status, health score, IPI, spike score, fee competition, block speed pressure, block utilization, tx/min, Standard gwei, 24h/7d averages, and timestamp — each with an **Interpretation** column. Long-form copy defines **LOW / NORMAL / HIGH / SPIKE** regimes and what each means for send timing (same scale as the live grid’s **Network status** cell).

**Example:** non-urgent transfer while status reads **HIGH** — read the interpretation for **Block utilization** and **IPI**; if both elevated, use **Best Time** or the heatmap on the live app rather than submitting on **Faster Inclusion** immediately.

![Network status — intelligence dashboard and status-level guide](assets/seo-network-status.png)

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

## Shared hosting headroom

Gas math and WebSocket fan-out run off shared hosting; WordPress only serves the SPA shell, SEO templates, and the REST mirror. After offloading ingest and fan-out, shared-hosting CPU and memory stay well below plan limits while the live tracker runs.

![Shared hosting — CPU and memory usage vs plan limits](assets/hostinger-cpu-memory.jpg)

Private code: [eth-gas-live-plugin](https://github.com/logicencoder/eth-gas-live-plugin) · live data [eth-gas-live-backend](https://github.com/logicencoder/eth-gas-live-backend)

Backend overview: [eth-gas-live-backend-overview](https://github.com/logicencoder/eth-gas-live-backend-overview)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
