<!-- BANNER IMAGE HERE -->

<div align="center">

# PUMPNUKE

**Volume bot for pump.fun** — generate organic-looking trade volume through wallet pool cycling with buy/sell rotations.

[![Node.js](https://img.shields.io/badge/Node.js-22%2B-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![Solana](https://img.shields.io/badge/Solana-Web3.js-9945FF?logo=solana&logoColor=white)](https://solana.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

[Quick Start](#quick-start) · [How It Works](#how-it-works) · [CLI Commands](#cli-commands) · [Dashboard](#dashboard) · [Configuration](#configuration)

</div>

---

## What It Does

PUMPNUKE creates realistic trading volume on pump.fun tokens. It manages a pool of wallets that execute buy/sell cycles with randomized timing, amounts, and rotation — making the volume appear organic on chart scanners and trackers.

```
                         PUMPNUKE Flow
                         
  ┌──────────┐     ┌──────────────────┐     ┌──────────────────┐
  │  Master   │────▶│   Fund Wallets   │────▶│   Initial Buy    │
  │  Wallet   │     │  (SOL transfer)  │     │  (30-50% each)   │
  └──────────┘     └──────────────────┘     └────────┬─────────┘
                                                     │
                                                     ▼
                                          ┌──────────────────────┐
                                          │   Volume Cycling     │
                                          │                      │
                                          │  ┌─────┐   ┌──────┐ │
                                          │  │ BUY │◀─▶│ SELL │ │
                                          │  └─────┘   └──────┘ │
                                          │                      │
                                          │  • Wallet rotation   │
                                          │  • Random amounts    │
                                          │  • Timed delays      │
                                          └──────────┬───────────┘
                                                     │
                                                     ▼
                                          ┌──────────────────────┐
                                          │     Cleanup          │
                                          │  Sell All → Drain    │
                                          │  SOL back to master  │
                                          └──────────────────────┘
```

## Features

- **Two Strategies** — Basic (fast cycling, 1-3s delays) and Staggered (organic timing, 2-8s delays)
- **Paper Mode** — Full simulation with no real transactions — test strategies risk-free
- **Wallet Pool** — Auto-generates keypairs, funds from master, drains back on completion
- **Smart Rotation** — Least-recently-used wallet selection, never repeats same wallet twice
- **Partial Sells** — Sells 40-70% of holdings per cycle (not 100%), keeping positions alive
- **Panic Sell** — Emergency button: instantly sells all tokens across all wallets and drains SOL
- **Web Dashboard** — Dark-themed live monitoring with real-time trade log, stats, and controls
- **Auto-Cleanup** — On completion or SIGINT: sells remaining tokens, drains wallets, prints session stats
- **Jito Integration** — Bundle submission for faster execution when needed
- **Curve + PumpSwap** — Automatically detects if token is on bonding curve or PumpSwap AMM
- **5-Error Auto-Stop** — Automatically stops job after 5 consecutive failures (token might be dead)

## Quick Start

### Prerequisites

- Node.js 22+
- Solana RPC endpoint (Helius, Quicknode, etc.)
- SOL in your master wallet

### Install

```bash
git clone https://github.com/zostaff/PUMPNUKE.git
cd PUMPNUKE
npm install
cp .env.example .env
```

### Configure

Edit `.env` with your RPC and wallet:

```env
RPC_URL=https://mainnet.helius-rpc.com/?api-key=YOUR_KEY
WALLET_PRIVATE_KEY=your_base58_private_key
DRY_RUN=true     # Start with paper mode!
```

### Run (Paper Mode)

```bash
node cli/nuke.mjs --mint <TOKEN_ADDRESS> --budget 0.5 --wallets 8 --strategy staggered
```

### Run (Live Mode)

```bash
# Set DRY_RUN=false in .env, then:
node cli/nuke.mjs --mint <TOKEN_ADDRESS> --budget 0.5 --wallets 8 --strategy staggered
```

## How It Works

### Phase 1: Fund Pool
Master wallet sends SOL directly to each generated wallet. Equal distribution based on budget.

### Phase 2: Initial Buy
Each wallet buys tokens using 30-50% of its balance. Timing depends on strategy:
- **Basic**: All wallets buy immediately
- **Staggered**: 2-6 second delays between each wallet's buy

### Phase 3: Volume Cycling
Continuous buy/sell loop until duration expires:
1. **Rotate** — Pick wallet with oldest `lastUsedAt`, never same as previous
2. **Decide** — Buy if wallet has no tokens, otherwise roll against `buyBias` (default 60% buy)
3. **Execute** — Buy with ~30% of SOL balance, or sell 40-70% of token holdings
4. **Delay** — Wait per strategy timing before next trade

### Phase 4: Cleanup
Sell all remaining tokens across all wallets, then drain SOL back to master wallet.

## CLI Commands

### `nuke.mjs` — Start Volume Job

```bash
node cli/nuke.mjs --mint <CA> [options]
```

| Flag | Description | Default |
|------|-------------|---------|
| `-m, --mint` | Token contract address | *required* |
| `-b, --budget` | Total SOL budget | `0.5` |
| `-w, --wallets` | Number of wallets | `8` |
| `-s, --strategy` | `basic` or `staggered` | `staggered` |
| `-d, --duration` | Duration in minutes | `15` |
| `--bias` | Buy probability (0-1) | `0.6` |
| `--slippage` | Slippage in bps | `1000` |
| `-y, --yes` | Skip confirmation | `false` |

### `status.mjs` — View Jobs

```bash
node cli/status.mjs            # List all jobs
node cli/status.mjs <jobId>    # Job details + recent trades
```

### `panic.mjs` — Emergency Sell + Drain

```bash
node cli/panic.mjs <jobId>
```

Immediately sells all tokens from all wallets and drains SOL back to master. Use when you need to exit instantly.

### `drain.mjs` — Drain SOL Only

```bash
node cli/drain.mjs <jobId>
```

Collects remaining SOL from all job wallets back to master (without selling tokens first).

## Dashboard

```bash
node dashboard/server.mjs
# Open http://localhost:3002
```

<!-- DASHBOARD SCREENSHOT HERE -->

The web dashboard provides:
- **Live job monitoring** — Status, buy/sell counts, cycles, SOL spent
- **Trade log** — Real-time feed of all trades with color-coded types
- **Job controls** — Start new jobs, stop running ones, panic sell
- **Create jobs** — Form to launch new volume jobs from the browser

API endpoints:
| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/jobs` | List all jobs |
| `GET` | `/api/jobs/:id` | Job details + trades |
| `POST` | `/api/jobs` | Create and start job |
| `POST` | `/api/jobs/:id/stop` | Stop running job |
| `POST` | `/api/jobs/:id/panic` | Panic sell + drain |

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `RPC_URL` | Solana RPC endpoint | `https://api.mainnet-beta.solana.com` |
| `WALLET_PRIVATE_KEY` | Master wallet private key (base58) | — |
| `DRY_RUN` | Paper mode (`true`/`false`) | `true` |
| `DEFAULT_STRATEGY` | Default strategy | `staggered` |
| `DEFAULT_WALLETS` | Default wallet count | `8` |
| `DEFAULT_BUDGET` | Default SOL budget | `0.5` |
| `DEFAULT_DURATION_MIN` | Default duration (minutes) | `15` |
| `DEFAULT_BUY_BIAS` | Buy probability (0-1) | `0.6` |
| `DEFAULT_SLIPPAGE_BPS` | Slippage basis points | `1000` |
| `PRIORITY_FEE` | Compute unit price (microLamports) | `300000` |
| `JITO_TIP` | Jito tip amount (SOL) | `0.0001` |
| `DASHBOARD_PORT` | Dashboard HTTP port | `3002` |
| `SOL_RESERVE` | Min SOL to keep per wallet | `0.01` |

### Strategies

| Strategy | Initial Buy | Cycle Delay | Best For |
|----------|------------|-------------|----------|
| `basic` | Simultaneous | 1-3s | Fast volume, less organic |
| `staggered` | 2-6s gaps | 2-8s | Organic-looking volume |

## Project Structure

```
pumpnuke/
├── package.json
├── .env.example
├── README.md
├── lib/
│   ├── config.mjs           # Environment and config loading
│   ├── constants.mjs         # Program IDs, Jito tip accounts
│   ├── wallet-pool.mjs       # Wallet generation, funding, drain, sellAll
│   ├── trade-builder.mjs     # Buy/sell via pump-sdk (curve + pumpswap)
│   ├── volume-engine.mjs     # Main volume cycling loop
│   ├── job-manager.mjs       # Job lifecycle: prepare → confirm → run → stop
│   ├── jito.mjs              # Jito bundle submission
│   └── logger.mjs            # Colored ANSI terminal output
├── cli/
│   ├── nuke.mjs              # Start volume job
│   ├── status.mjs            # View job status
│   ├── panic.mjs             # Emergency sell + drain
│   └── drain.mjs             # Drain SOL back to master
└── dashboard/
    ├── server.mjs             # HTTP server on localhost:3002
    └── public/
        └── index.html         # Dark theme dashboard
```

## What PUMPNUKE Can Do

- Generate organic-looking buy/sell volume on any pump.fun token
- Run in paper mode for risk-free strategy testing
- Manage pools of 2-20 wallets with automatic rotation
- Panic sell everything in seconds when needed
- Monitor everything through a web dashboard
- Handle both bonding curve and PumpSwap AMM tokens
- Auto-cleanup: sell tokens and return SOL on completion

## What PUMPNUKE Cannot Do

- It cannot guarantee profit — volume generation costs SOL in fees and slippage
- It cannot bypass pump.fun's anti-bot measures if they exist
- It does not launch tokens — only generates volume on existing ones
- It does not use hop wallets, warmup, or advanced stealth techniques
- It does not dynamically adjust priority fees — uses a fixed fee from config
- It does not encrypt wallet keys — this is an open-source demo tool

## Tech Stack

- **Runtime**: Node.js 22+ (ESM)
- **Blockchain**: @solana/web3.js ^1.98, @solana/spl-token ^0.4.12
- **Trading**: @pump-fun/pump-sdk, @pump-fun/pump-swap-sdk
- **Encoding**: bs58 ^6.0
- **Config**: dotenv ^16.4
- **Dashboard**: Vanilla JS, no frameworks

## Safety Notes

- **Always start with `DRY_RUN=true`** and verify the simulation looks correct
- **Use a dedicated wallet** — never use your main wallet as the master
- **Start with small budgets** (0.1-0.5 SOL) until you're comfortable
- **Watch the logs** — the bot logs every transaction with colored output
- **SIGINT (Ctrl+C) is safe** — triggers full cleanup (sell + drain)
- Wallet private keys are stored in plaintext in job files — keep the `data/` directory secure

## Disclaimer

This software is provided for **educational and research purposes only**. Trading cryptocurrencies involves significant risk of financial loss. The authors are not responsible for any losses incurred through the use of this software. Use at your own risk and in compliance with all applicable laws and regulations. Volume generation may violate the terms of service of certain platforms.

## License

MIT
