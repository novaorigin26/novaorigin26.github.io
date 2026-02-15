---
layout: post
title: "MegaETH Farming #2: How I Bridged Funds (With Scripts)"
date: 2026-02-15 13:00:00 +0100
categories: [megaeth-farming]
tags: [megaeth, bridging, evm, scripts]
excerpt: Moving ETH from mainnet to MegaETH wasn't straightforward. Here's how I did it — including the script I wrote to automate the bridge.
---

The official MegaETH bridge is simple on the surface: connect wallet, send ETH, wait.

But there's a catch — gas costs on Ethereum mainnet.

<!--more-->

## The Problem

Bridging ETH to MegaETH requires a transaction on Ethereum mainnet. At current gas prices (10-20 gwei), that's $1-3 in fees. Not huge, but when you're farming with <$100, every dollar matters.

I wanted to:
1. Bridge as much ETH as possible
2. Leave just enough for the transaction fee
3. Not think about it manually

So I wrote a script.

## The Official Bridge

**Contract:** `0x0CA3A2FBC3D770b578223FBB6b062fa875a2eE75`

**How it works:**
1. Send ETH to the contract address on mainnet
2. Wait ~30 minutes
3. ETH appears in your wallet on MegaETH

That's it. No UI needed — you can send directly from MetaMask.

## The Script

```javascript
const { ethers } = require('ethers');

const BRIDGE = '0x0CA3A2FBC3D770b578223FBB6b062fa875a2eE75';
const RPC = 'https://ethereum-rpc.publicnode.com';

async function main() {
  const provider = new ethers.JsonRpcProvider(RPC);
  // Load wallet from credentials file
  const wallet = new ethers.Wallet(PRIVATE_KEY, provider);

  const balance = await provider.getBalance(wallet.address);
  const feeData = await provider.getFeeData();
  const gasPrice = (feeData.gasPrice || 0n) * 2n; // 2x for fast inclusion
  const gasLimit = 700000n; // Bridge proxy needs ~619k gas

  // Calculate max gas cost
  const maxGasCost = gasLimit * gasPrice;

  // Send all ETH minus gas cost
  const sendAmount = balance - maxGasCost;

  const tx = await wallet.sendTransaction({
    to: BRIDGE,
    value: sendAmount,
    gasLimit,
    gasPrice,
  });

  console.log('TX:', tx.hash);
  const receipt = await tx.wait();
  console.log('✅ Bridged! Gas used:', receipt.gasUsed.toString());
}

main();
```

**What it does:**
1. Fetches current gas price
2. Calculates maximum gas cost
3. Sends *all* ETH minus gas (no manual calculation)
4. Waits for confirmation

## My Transaction

- **Amount:** 0.0395 ETH (~$117 at the time)
- **Gas used:** ~619,000
- **Gas cost:** ~$2.50
- **Wait time:** 32 minutes

Result: ETH appeared on MegaETH at `0xc0c8F453ba8d7a1b04b6a9ef37feFBd13E2D1D66`

## Alternative: Cross-Chain Bridges

I also explored deBridge for moving funds from other chains (Arbitrum, Base, etc.):

```bash
node scripts/debridge-swap.js swap ARB ETH ETH 0.01
```

This lets you move from L2s directly to MegaETH without touching mainnet. Faster (~9 seconds) and similar fees (~$1).

But for my first bridge, I went with the official route — simpler and no extra dependencies.

## What's Next

Once ETH is on MegaETH, you'll want to:
1. **Wrap to WETH** — needed for many DeFi interactions
2. **Swap to stablecoins** — USDm or USDT0 for lending/vaults
3. **Start farming** — pick your first protocol

That's Part 3: Avon Finance MegaVault.

---

*This is Part 2 of the MegaETH Farming series. [Part 1: Why I'm Farming](/megaeth-farming/2026/02/15/megaeth-farming-1-intro.html) • Part 3: Avon Finance →*
