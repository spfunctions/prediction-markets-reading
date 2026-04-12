# Kalshi API Quick Start: JavaScript and Python in 5 Minutes

> From zero to your first API call in both languages. Authentication, market data, placing orders — then how SimpleFunctions collapses it all into one command.

**Category:** guide | **Author:** SimpleFunctions | **Reading time:** 7 min

---
The Kalshi API is well-documented but has one sharp edge that trips up every new developer: authentication. Unlike most APIs that hand you a bearer token, Kalshi uses RSA-PSS signature-based auth. You sign every request with your private key. Once you get past that, everything else is straightforward REST.

This guide takes you from zero to making your first API call — fetching live market data, reading an orderbook, and placing an order — in both JavaScript (Node.js) and Python. Then we'll show you how SimpleFunctions does the same thing in one command.

## Step 0: Get Your API Keys

1. Log into [kalshi.com](https://kalshi.com)
2. Go to **Settings → API Keys**
3. Click **Generate New Key Pair**
4. Kalshi generates an RSA key pair. **Download both the private key (.pem) and note your API key ID.** The private key is shown only once.

Store the private key somewhere safe. You'll reference it in your code:

```
~/.kalshi/private_key.pem
```

Your API key ID looks something like `a1b2c3d4-e5f6-7890-abcd-ef1234567890`.

## JavaScript (Node.js): Authentication

The core challenge is building the RSA-PSS signature. Here's a complete working example:

```javascript
import crypto from 'crypto';
import fs from 'fs';

const KALSHI_API_BASE = 'https://api.elections.kalshi.com/trade-api/v2';
const API_KEY_ID = process.env.KALSHI_API_KEY_ID;
const PRIVATE_KEY = fs.readFileSync(
  process.env.KALSHI_PRIVATE_KEY_PATH || '~/.kalshi/private_key.pem',
  'utf-8'
);

function signRequest(method, path, timestamp) {
  // Kalshi expects: METHOD + timestamp_ms + path
  const message = timestamp + method + '/trade-api/v2' + path;

  const signature = crypto.sign('sha256', Buffer.from(message), {
    key: PRIVATE_KEY,
    padding: crypto.constants.RSA_PKCS1_PSS_PADDING,
    saltLength: crypto.constants.RSA_PSS_SALTLEN_DIGEST,
  });

  return signature.toString('base64');
}

async function kalshiFetch(method, path, body = null) {
  const timestamp = Date.now().toString();
  const signature = signRequest(method, path, timestamp);

  const response = await fetch(`${KALSHI_API_BASE}${path}`, {
    method,
    headers: {
      'KALSHI-ACCESS-KEY': API_KEY_ID,
      'KALSHI-ACCESS-SIGNATURE': signature,
      'KALSHI-ACCESS-TIMESTAMP': timestamp,
      'Content-Type': 'application/json',
    },
    body: body ? JSON.stringify(body) : undefined,
  });

  if (!response.ok) {
    const text = await response.text();
    throw new Error(`Kalshi API ${response.status}: ${text}`);
  }

  return response.json();
}
```

**The gotcha:** The timestamp must be in milliseconds, and the signature message format is `timestamp + METHOD + full_path`. Get the concatenation order wrong and you'll get 401s that tell you nothing useful.

## JavaScript: Fetching Market Data

With the auth wrapper in place, getting market data is one call:

```javascript
// Get all events (grouped markets)
const events = await kalshiFetch('GET', '/events?status=open&limit=50');
console.log(`Found ${events.events.length} open events`);

// Get a specific market by ticker
// Example: KXRECESSION-26 (US Recession 2026)
const market = await kalshiFetch('GET', '/markets/KXRECESSION-26');
console.log(`${market.market.title}: YES @ ${market.market.yes_ask}¢`);

// Get the orderbook for that market
const orderbook = await kalshiFetch('GET', '/markets/KXRECESSION-26/orderbook');
console.log('Best bid:', orderbook.orderbook.yes[0]);
console.log('Best ask:', orderbook.orderbook.no[0]);
```

Real tickers you can try right now:
- `KXRECESSION-26` — US Recession 2026
- `KXFEDRATE-26JUN` — Fed Rate Decision June 2026
- `KXCPI-26MAR` — CPI March 2026

## JavaScript: Placing an Order

```javascript
const order = await kalshiFetch('POST', '/portfolio/orders', {
  ticker: 'KXRECESSION-26',
  action: 'buy',
  side: 'yes',
  type: 'limit',
  count: 10,          // 10 contracts
  yes_price: 35,      // limit price in cents
});

console.log('Order placed:', order.order.order_id);
console.log('Status:', order.order.status);
// status will be 'resting' (on book) or 'executed' (filled)
```

**Important:** Prices are in cents (integers 1-99). Don't pass decimals. `yes_price: 35` means 35 cents.

## Python: The Same Thing

Python's `cryptography` library handles RSA-PSS cleanly:

```python
import time
import json
import requests
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import padding
import base64
import os

KALSHI_API_BASE = "https://api.elections.kalshi.com/trade-api/v2"
API_KEY_ID = os.environ["KALSHI_API_KEY_ID"]

with open(os.environ.get("KALSHI_PRIVATE_KEY_PATH", "~/.kalshi/private_key.pem"), "rb") as f:
    PRIVATE_KEY = serialization.load_pem_private_key(f.read(), password=None)


def sign_request(method: str, path: str, timestamp: str) -> str:
    message = f"{timestamp}{method}/trade-api/v2{path}"
    signature = PRIVATE_KEY.sign(
        message.encode(),
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH,
        ),
        hashes.SHA256(),
    )
    return base64.b64encode(signature).decode()


def kalshi_fetch(method: str, path: str, body: dict = None):
    timestamp = str(int(time.time() * 1000))
    signature = sign_request(method, path, timestamp)

    headers = {
        "KALSHI-ACCESS-KEY": API_KEY_ID,
        "KALSHI-ACCESS-SIGNATURE": signature,
        "KALSHI-ACCESS-TIMESTAMP": timestamp,
        "Content-Type": "application/json",
    }

    response = requests.request(
        method,
        f"{KALSHI_API_BASE}{path}",
        headers=headers,
        json=body,
    )
    response.raise_for_status()
    return response.json()
```

## Python: Market Data and Orders

```python
# Fetch a market
market = kalshi_fetch("GET", "/markets/KXRECESSION-26")
print(f"{market['market']['title']}: YES @ {market['market']['yes_ask']}¢")

# Read the orderbook
book = kalshi_fetch("GET", "/markets/KXRECESSION-26/orderbook")
print(f"Bid: {book['orderbook']['yes'][0]}, Ask: {book['orderbook']['no'][0]}")

# Place a limit order
order = kalshi_fetch("POST", "/portfolio/orders", {
    "ticker": "KXRECESSION-26",
    "action": "buy",
    "side": "yes",
    "type": "limit",
    "count": 10,
    "yes_price": 35,
})
print(f"Order {order['order']['order_id']}: {order['order']['status']}")
```

## Common Mistakes

**1. Signature format.** The message to sign is `timestamp + METHOD + /trade-api/v2 + path`. Not just `path`. Not `/v2/path`. The full prefix matters.

**2. Timestamp drift.** Kalshi rejects requests where the timestamp is more than 5 seconds from server time. If you're running on a VM or serverless function with clock drift, you'll get intermittent 401s. Fix: check your system clock or add retry logic with a fresh timestamp.

**3. Price vs probability.** Kalshi prices are in cents (integers). A market at "35%" means `yes_price: 35`. Don't divide by 100.

**4. Rate limits.** Kalshi allows roughly 10 requests/second for market data and 1 request/second for orders. Burst above that and you'll get 429s.

**5. Null prices.** Low-liquidity markets sometimes return `null` for `yes_ask` or `yes_bid`. Always null-check before doing math.

## Now Do It in One Command

Everything above — authentication, market scanning, orderbook analysis — is what SimpleFunctions wraps into the CLI:

```bash
# Scan all markets for edge opportunities
sf scan

# Get full context on a specific market
sf context KXRECESSION-26
# Returns: price, orderbook depth, spread, volume, historical movement,
# related markets, and a preliminary edge assessment

# Compare a market against your thesis
sf edge --thesis iran-war --ticker KXRECESSION-26
```

`sf scan` does what 50+ lines of auth code + API calls do, but across *every active market* on Kalshi and Polymarket simultaneously. It handles authentication, pagination, rate limiting, null-price edge cases, and cross-venue deduplication.

`sf context` pulls everything you'd want to know about a specific ticker into one view — no more chaining 4 API calls to get price + orderbook + history + related markets.

## MCP Server Setup for Claude Code / Cursor

If you use Claude Code or Cursor as your development environment, you can connect SimpleFunctions as an MCP (Model Context Protocol) server. This lets the AI assistant call SimpleFunctions tools directly:

```json
{
  "mcpServers": {
    "simplefunctions": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-sf@latest"],
      "env": {
        "SF_API_KEY": "your-simplefunctions-api-key"
      }
    }
  }
}
```

Save this in `.claude/mcp.json` (Claude Code) or `.cursor/mcp.json` (Cursor).

Now you can ask your AI assistant:

> "Scan Kalshi for markets related to US recession and show me the ones with the widest bid-ask spread"

And it will call `sf scan`, filter results, and present the analysis — without you writing a single line of API code.

The MCP bridge gives you the power of the full SimpleFunctions toolkit inside your conversational coding workflow. Ask questions in natural language, get structured market data back.

## What You Should Do Next

1. **Copy the auth code** above and make one API call. Just one. Seeing live market data in your terminal changes how you think about prediction markets.
2. **Install the CLI** (`npm i -g @spfunctions/cli && sf setup`) and run `sf scan`. Compare how long it takes vs. writing the API wrapper yourself.
3. **Set up the MCP server** if you use Claude Code or Cursor. Having market data available inside your development conversation is a workflow multiplier.

The Kalshi API is good. But you shouldn't be spending time on authentication plumbing when the real work is building a thesis and finding edge.