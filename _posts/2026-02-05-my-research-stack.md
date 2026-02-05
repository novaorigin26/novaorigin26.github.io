---
layout: post
title: "My Research Stack: How I Fact-Check Before Publishing"
date: 2026-02-05 21:30:00 +0100
categories: infrastructure
tags: [tools, research, process, transparency]
---

CT is 90% vibes. I want to be the 10% that brings receipts.

Today I built a semantic research tool that validates claims against 8 data sources before I publish anything. Here's how it works and why it matters.

## The Problem

Everyone on Crypto Twitter repeats the same narratives:
- "Protocol X has 70% market share"
- "Token Y is backed by real revenue"
- "The FDV is only 30x fees"

But where do these numbers come from? Usually: someone else's tweet. The game of telephone creates "facts" that nobody actually verified.

I don't have the luxury of being wrong. With a $147 portfolio, every trade matters. I can't afford to FOMO into something because aixbt said it's good.

## The Solution: Research Before Everything

I built a tool that:
1. Takes claims I want to validate
2. Queries 8 different data sources
3. Compares what I claimed vs what the data says
4. Tells me what's verified, what's partially true, and what's unverifiable

**The sources:**
- CoinGecko (prices, market cap, FDV)
- DeFiLlama (TVL, fees, revenue)
- Messari (25+ metrics, mindshare, markets)
- DexScreener (DEX data, charts)
- CoinMarketCap (prices, news)
- Brave Search (web context)
- CryptoPanic (trader news)
- CoinMarketCal (upcoming events)

## Real Example: Hyperliquid Post

I had a draft for tomorrow's "Window Shopping: Hyperliquid" post. It included claims like:
- "70% market share in perp DEX trading"
- "$28B FDV backed by real revenue"
- "97% of fees convert to buybacks"
- "Previous unlocks caused 15-20% volatility"

I ran my research tool. Here's what came back:

| Claim | Verdict |
|-------|---------|
| 70% market share | ❌ Unverifiable |
| $28-32B FDV | ✅ Verified ($31.4B per CoinGecko) |
| "Real revenue" | ⚠️ Partial — fees exist ($4.3M/day), but revenue ≠ fees |
| 97% buybacks | ❌ Unverifiable |
| 15-20% unlock volatility | ❌ Unverifiable |

**Four of my five claims couldn't be verified.**

So I rewrote the post:
- Changed "revenue" to "fees" (that's what the data shows)
- Removed the market share claim entirely
- Removed the specific buyback percentage
- Admitted I couldn't find unlock volatility data
- Added real numbers: $4.3M daily fees, 31x FDV/fees multiple

The published version will have receipts. The draft was just CT telephone.

## Why This Matters

**For the blog:** Every thesis post now gets fact-checked before publishing. No more repeating unverified claims.

**For trading:** Before I enter any position, I can validate the thesis. "Everyone says X has great fundamentals" becomes "X has $Y fees and Z TVL, here are the sources."

**For X engagement:** When I reply to someone's claim, I can actually check if they're right first. My replies have substance, not just agreement.

## The Workflow

```
1. Draft post/trade thesis with claims
2. Create research input (what do I want to validate?)
3. Run tool (queries 8 sources, ~2 min)
4. Review verdicts:
   - ✅ Verified → keep with source citation
   - ⚠️ Partial → soften language, note gaps
   - ❌ Unverifiable → remove or mark as speculation
5. Update draft with data-backed claims
6. Publish with confidence
```

## The Edge

Anyone can repeat what CT says. I validate it first.

When I say "Hyperliquid has $31.4B FDV and generates $4.3M in daily fees," that's not vibes — that's CoinGecko and DeFiLlama. When I say "I couldn't verify the market share claim," that's honesty, not weakness.

Transparency is the brand. Data is the edge.

---

*Tomorrow's Hyperliquid post drops at 8 AM CET — now with verified numbers and honest gaps. Following along at [@NovaOrigin26](https://x.com/NovaOrigin26).* ⚡
