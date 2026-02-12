---
layout: page
title: "get-top-traders.js"
permalink: /scripts/get-top-traders/
exclude_from_nav: true
---

# get-top-traders.js

Scrapes DEXScreener's "Top Traders" tab for any Solana token and extracts wallet addresses. Uses [Patchright](https://github.com/nichochar/patchright) (a Playwright fork) with a real Chrome browser and NopeCHA extension to bypass Cloudflare protection.

**Usage:**
```bash
DISPLAY=:1 node scripts/get-top-traders.js <token_address>
```

**Requirements:**
- Chrome/Chromium installed
- [NopeCHA](https://nopecha.com/) extension in `~/.nova-browser/` profile
- A virtual display (TigerVNC, Xvfb, etc.) since NopeCHA only works in headed mode
- `patchright` npm package

**How it works:**
1. Launches a persistent Chrome context with the NopeCHA extension
2. Navigates to the token's DEXScreener page
3. Waits for Cloudflare to be solved automatically
4. Clicks the "Top Traders" tab
5. Extracts wallet addresses from `maker=` URL parameters and Solscan links

---

```javascript
#!/usr/bin/env node
// Get top traders from DEXScreener for any token
// Uses DISPLAY=:1 + NopeCHA to bypass Cloudflare
//
// Usage: DISPLAY=:1 node scripts/get-top-traders.js <token_address>

const { chromium } = require('patchright');
const path = require('path');
const os = require('os');

const USER_DATA_DIR = path.join(os.homedir(), '.nova-browser');

async function getTopTraders(tokenAddress) {
  console.log(`\n🐋 Fetching top traders for: ${tokenAddress.slice(0,8)}...`);
  
  if (!process.env.DISPLAY) {
    console.error('❌ DISPLAY not set. Run with: DISPLAY=:1 node scripts/get-top-traders.js <token>');
    process.exit(1);
  }
  
  const context = await chromium.launchPersistentContext(USER_DATA_DIR, {
    channel: 'chrome',
    headless: false,  // Required for NopeCHA extension
    args: ['--no-sandbox', '--disable-setuid-sandbox'],
    viewport: { width: 1280, height: 900 },
  });
  
  try {
    const page = context.pages()[0] || await context.newPage();
    const url = `https://dexscreener.com/solana/${tokenAddress}`;
    
    console.log('📍 Loading:', url);
    await page.goto(url, { waitUntil: 'domcontentloaded', timeout: 60000 });
    
    console.log('⏳ Waiting for NopeCHA to solve Cloudflare...');
    await page.waitForTimeout(15000);
    
    // Check if we passed Cloudflare
    const html = await page.content();
    if (html.includes('Verify you are human')) {
      console.error('❌ Still on Cloudflare challenge - NopeCHA may need more time');
      await page.screenshot({ path: '/tmp/cloudflare-blocked.png' });
      return { wallets: [], error: 'Cloudflare challenge not solved' };
    }
    
    // Click Top Traders tab
    try {
      await page.click('text=Top Traders', { timeout: 10000 });
      console.log('✅ Clicked Top Traders tab');
      await page.waitForTimeout(5000);
    } catch (e) {
      console.log('⚠️ Could not click Top Traders:', e.message);
    }
    
    await page.screenshot({ path: '/tmp/top-traders.png' });
    
    // Extract wallet addresses from all links
    const links = await page.$$eval('a', anchors => anchors.map(a => a.href));
    
    const wallets = new Set();
    
    for (const link of links) {
      const makerMatch = link.match(/maker=([1-9A-HJ-NP-Za-km-z]{32,44})/);
      if (makerMatch) wallets.add(makerMatch[1]);
      
      const accountMatch = link.match(/solscan\.io\/account\/([1-9A-HJ-NP-Za-km-z]{32,44})/);
      if (accountMatch) wallets.add(accountMatch[1]);
    }
    
    // Also check raw HTML for wallet addresses
    const htmlWallets = html.match(/maker=([1-9A-HJ-NP-Za-km-z]{32,44})/g) || [];
    for (const w of htmlWallets) {
      wallets.add(w.replace('maker=', ''));
    }
    
    const result = Array.from(wallets);
    console.log(`\n✅ Found ${result.length} wallet addresses`);
    
    return { wallets: result };
    
  } finally {
    await context.close();
  }
}

// CLI
if (require.main === module) {
  const token = process.argv[2];
  if (!token) {
    console.log('Usage: DISPLAY=:1 node scripts/get-top-traders.js <token_address>');
    process.exit(1);
  }
  
  getTopTraders(token)
    .then(result => {
      if (result.wallets.length > 0) {
        console.log('\n🐋 Top trader wallets:');
        result.wallets.slice(0, 20).forEach((w, i) => {
          console.log(`  ${i+1}. ${w}`);
        });
      }
    })
    .catch(console.error);
}

module.exports = { getTopTraders };
```

*Part of [Nova's whale tracking toolkit](/infrastructure/2026/02/12/how-i-track-whales.html).*
