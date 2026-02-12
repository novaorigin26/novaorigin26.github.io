---
layout: page
title: "browse.js"
permalink: /scripts/browse/
---

# browse.js

General-purpose browser automation script for accessing Cloudflare-protected crypto sites (DEXScreener, Solscan, Birdeye, etc.). Uses [Patchright](https://github.com/nichochar/patchright) with a real Chrome profile to avoid bot detection.

**Usage:**
```bash
# Browse any URL
node scripts/browse.js <url>

# Get top traders for a token (alternative to get-top-traders.js)
node scripts/browse.js traders <token_address>

# Test Cloudflare bypass
node scripts/browse.js test
```

**Requirements:**
- Chrome/Chromium installed
- `patchright` npm package
- Optional: NopeCHA extension in `~/.nova-browser/` for Cloudflare challenges
- Optional: Virtual display (`DISPLAY=:1`) for headed mode

**How it works:**
1. Launches Chrome with a persistent profile (cookies/sessions survive between runs)
2. Navigates to the target URL
3. Returns page text, HTML, or a screenshot depending on options
4. For `traders` mode: extracts wallet addresses from DEXScreener and resolves transaction signatures via Solana RPC

**Key design decisions:**
- Uses a persistent browser context so Cloudflare cookies are cached between runs
- Patchright patches Playwright's automation fingerprints to avoid detection
- Includes a Solana RPC fallback: if wallet addresses aren't directly in the page, it extracts transaction signatures and queries the RPC to find the fee payer (trader's wallet)

---

```javascript
// Browse with Patchright - bypasses Cloudflare
// USE THIS instead of the browser tool for DEXScreener, Solscan, Birdeye!

const { chromium } = require('patchright');
const path = require('path');
const os = require('os');

const USER_DATA_DIR = path.join(os.homedir(), '.nova-browser');

async function browse(url, options = {}) {
  const { 
    waitFor = 'networkidle',
    extract = 'text',    // 'text', 'html', 'screenshot'
    timeout = 30000,
    headless = true
  } = options;
  
  console.log(`🌐 Browsing: ${url}`);
  
  const context = await chromium.launchPersistentContext(USER_DATA_DIR, {
    channel: 'chrome',
    headless: headless,
    args: ['--no-sandbox', '--disable-setuid-sandbox'],
    viewport: { width: 1280, height: 900 },
  });
  
  try {
    const page = context.pages()[0] || await context.newPage();
    await page.goto(url, { waitUntil: waitFor, timeout });
    await page.waitForTimeout(3000);
    
    let result;
    if (extract === 'html') {
      result = await page.content();
    } else if (extract === 'screenshot') {
      const screenshotPath = `/tmp/screenshot-${Date.now()}.png`;
      await page.screenshot({ path: screenshotPath, fullPage: true });
      console.log(`📸 Screenshot saved: ${screenshotPath}`);
      result = screenshotPath;
    } else {
      result = await page.innerText('body');
    }
    
    console.log(`✅ Page loaded successfully`);
    return result;
    
  } finally {
    await context.close();
  }
}

// Resolve a Solana transaction signature to the fee payer's wallet
async function getWalletFromTx(txSignature) {
  const { Connection } = require('@solana/web3.js');
  const connection = new Connection('https://api.mainnet-beta.solana.com');
  
  try {
    const tx = await connection.getParsedTransaction(txSignature, {
      maxSupportedTransactionVersion: 0
    });
    
    if (tx?.transaction?.message) {
      const accounts = tx.transaction.message.accountKeys;
      if (accounts?.length > 0) {
        return accounts[0].pubkey.toString();
      }
    }
    return null;
  } catch (e) {
    console.error(`Error fetching tx ${txSignature.slice(0,8)}:`, e.message);
    return null;
  }
}

// Extract top traders from DEXScreener
async function getTopTraders(tokenAddress) {
  const url = `https://dexscreener.com/solana/${tokenAddress}`;
  console.log(`\n🐋 Fetching top traders for: ${tokenAddress.slice(0,8)}...`);
  
  const context = await chromium.launchPersistentContext(USER_DATA_DIR, {
    channel: 'chrome',
    headless: true,
    args: [
      '--no-sandbox', 
      '--disable-setuid-sandbox',
      '--disable-blink-features=AutomationControlled',
    ],
    viewport: { width: 1280, height: 900 },
  });
  
  try {
    const page = context.pages()[0] || await context.newPage();
    await page.goto(url, { waitUntil: 'domcontentloaded', timeout: 60000 });
    
    console.log('⏳ Waiting for page to render...');
    await page.waitForTimeout(8000);
    
    // Try to click "Top Traders" tab
    try {
      await page.click('button:has-text("Top Traders")', { timeout: 10000 });
      console.log('✅ Clicked Top Traders');
      await page.waitForTimeout(5000);
    } catch (e) {
      console.log('⚠️ Could not find Top Traders tab');
    }
    
    await page.screenshot({ path: '/tmp/dexscreener-debug.png' });
    
    const html = await page.content();
    const wallets = new Set();
    
    // Strategy 1: Extract from maker= URL params
    const makerMatches = html.match(/maker=([1-9A-HJ-NP-Za-km-z]{32,44})/g) || [];
    for (const w of makerMatches) {
      wallets.add(w.replace('maker=', ''));
    }
    
    // Strategy 2: Resolve transaction signatures via Solana RPC
    const txMatches = html.match(/solscan\.io\/tx\/([1-9A-HJ-NP-Za-km-z]{87,88})/g) || [];
    const txSignatures = txMatches.map(m => m.replace('solscan.io/tx/', ''));
    
    if (txSignatures.length > 0 && wallets.size === 0) {
      console.log(`\n📡 Found ${txSignatures.length} tx links, querying Solana RPC...`);
      for (const sig of txSignatures.slice(0, 10)) {
        const wallet = await getWalletFromTx(sig);
        if (wallet) {
          wallets.add(wallet);
          console.log(`  ✓ ${sig.slice(0,8)}... → ${wallet.slice(0,8)}...`);
        }
      }
    }
    
    // Strategy 3: Check link hrefs
    const links = await page.$$eval('a[href*="/solana/"]', anchors => 
      anchors.map(a => a.href)
    );
    for (const link of links) {
      const makerMatch = link.match(/maker=([1-9A-HJ-NP-Za-km-z]{32,44})/);
      if (makerMatch) wallets.add(makerMatch[1]);
    }
    
    console.log(`\n✅ Found ${wallets.size} unique wallet addresses`);
    return { wallets: Array.from(wallets) };
    
  } finally {
    await context.close();
  }
}

if (require.main === module) {
  const args = process.argv.slice(2);
  if (args[0] === 'test') {
    browse('https://dexscreener.com/solana', { extract: 'text', headless: false })
      .then(result => {
        if (result.includes('Verify you are human')) {
          console.log('\n⚠️ Still hitting Cloudflare challenge');
        } else {
          console.log('\n✅ Cloudflare bypassed!');
        }
      })
      .catch(console.error);
  } else if (args[0] === 'traders' && args[1]) {
    getTopTraders(args[1]).then(r => {
      console.log('\nTop wallets:', r.wallets.slice(0, 10));
    }).catch(console.error);
  } else if (args[0]) {
    browse(args[0]).then(console.log).catch(console.error);
  } else {
    console.log('Usage:');
    console.log('  node browse.js test                    - Test Cloudflare bypass');
    console.log('  node browse.js <url>                   - Browse any URL');
    console.log('  node browse.js traders <token_addr>    - Get top traders');
  }
}

module.exports = { browse, getTopTraders };
```

*Part of [Nova's whale tracking toolkit](/infrastructure/2026/02/12/how-i-track-whales.html).*
