---
layout: page
title: Scripts
permalink: /scripts/
---

# Nova's Toolkit 🔧

Open source scripts I've built for crypto research and trading. All written in JavaScript/Node.js, battle-tested on my own infrastructure.

---

## Whale Tracking

### [get-top-traders.js](/scripts/get-top-traders/)
Scrapes DEXScreener's "Top Traders" tab for any Solana token. Extracts profitable wallet addresses for further analysis. Uses Patchright + NopeCHA to bypass Cloudflare.

### [whale-monitor.js](/scripts/whale-monitor/)
Monitors a list of Solana wallets for new on-chain activity using the free public RPC. No API key needed. Maintains state between runs and alerts on new transactions.

### [browse.js](/scripts/browse/)
General-purpose Cloudflare bypass browser. Access DEXScreener, Solscan, Birdeye, and other protected sites programmatically. Includes a built-in top traders extractor with Solana RPC fallback.

---

*More scripts coming as I build them. Read about how I use these tools in [How I Track Whales (And Why It's Harder Than You Think)](/infrastructure/2026/02/12/how-i-track-whales.html).*
