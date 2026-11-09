<div align="center">

```text
                         |\
                         | \
              .  ~=~=~=~ |  \______________________________________________
           .  ~=~=~=~=~=~|==/:::::::::::::::::::::|                        \
         ~=~=~=~=~=~=~=~=|=|::::::::::::::::::::::|          \ | /          |
       <~=~=~=~=~=~=~=~=~|=|:::::  P U M P  ::::::|        --( O )--        |
         ~=~=~=~=~=~=~=~=|=|::::::::::::::::::::::|          / | \          |
           '  ~=~=~=~=~=~|==\:::::::::::::::::::::|                        /
              '  ~=~=~=~ |  /----------------------------------------------
                         | /
                         |/
```

```text
██████╗ ██╗   ██╗███╗   ███╗██████╗ ███╗   ██╗██╗   ██╗██╗  ██╗███████╗
██╔══██╗██║   ██║████╗ ████║██╔══██╗████╗  ██║██║   ██║██║ ██╔╝██╔════╝
██████╔╝██║   ██║██╔████╔██║██████╔╝██╔██╗ ██║██║   ██║█████╔╝ █████╗
██╔═══╝ ██║   ██║██║╚██╔╝██║██╔═══╝ ██║╚██╗██║██║   ██║██╔═██╗ ██╔══╝
██║     ╚██████╔╝██║ ╚═╝ ██║██║     ██║ ╚████║╚██████╔╝██║  ██╗███████╗
╚═╝      ╚═════╝ ╚═╝     ╚═╝╚═╝     ╚═╝  ╚═══╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝
```

```text
┌────────────────────────────────────────────────────────────────────────────────┐
│   >> VOLUME ENGINE FOR PUMP.FUN  //  WALLET POOL  //  CLI + LIVE DASHBOARD <<  │
└────────────────────────────────────────────────────────────────────────────────┘
```

![node](https://img.shields.io/badge/node-22%2B-3ddc84?style=flat-square&labelColor=0a0e12)
![solana](https://img.shields.io/badge/solana-web3.js-9945FF?style=flat-square&labelColor=0a0e12)
![pump.fun](https://img.shields.io/badge/pump.fun-sdk-3ddc84?style=flat-square&labelColor=0a0e12)
![esm](https://img.shields.io/badge/ESM-only-ffb020?style=flat-square&labelColor=0a0e12)

**[ quick start ](#01--quick-start)** · **[ flow ](#02--how-it-works)** · **[ strategies ](#03--strategies)** · **[ cli ](#04--cli)** · **[ dashboard ](#05--dashboard)** · **[ config ](#06--configuration)** · **[ internals ](#07--under-the-hood)** · **[ faq ](#08--faq)**

</div>

```text
┌────────────────────────────────────┬────────────────────────────────────┐
│ [x2] STRATEGIES                    │ [@@] WALLET POOL                   │
│      basic · staggered timing      │      generate · fund · drain       │
├────────────────────────────────────┼────────────────────────────────────┤
│ [<>] LRU ROTATION                  │ [%%] PARTIAL SELLS                 │
│      never same wallet twice       │      40-70% of holdings            │
├────────────────────────────────────┼────────────────────────────────────┤
│ [!!] PANIC SELL                    │ [##] LIVE DASHBOARD                │
│      dump all + drain, 1 cmd       │      dark UI, refresh 3s           │
├────────────────────────────────────┼────────────────────────────────────┤
│ [~~] CURVE + PUMPSWAP              │ [XX] AUTO STOP                     │
│      venue auto-detect             │      5 errors in a row = halt      │
└────────────────────────────────────┴────────────────────────────────────┘
```

PUMPNUKE spins up a pool of fresh Solana wallets, funds them from your master wallet and runs continuous buy/sell cycles on a pump.fun token — rotating wallets, randomized sizes, strategy-driven timing. When the run ends or you hit `Ctrl+C`, it sells every remaining token and drains all SOL back to master.

```text
$ node cli/nuke.mjs --mint DYdf...pump --budget 0.5 --wallets 8 --strategy staggered

  P U M P N U K E  ·  LIVE  ·  8 wallets  ·  0.5 SOL  ·  15 min
  Proceed? (y/n) y

10:47:40.489 [JOB]   job ecc2434de1a8 started — Ctrl+C to stop gracefully
10:47:40.592 [FUND]  C2K4..DkrT 0.0719 SOL  sig=3kPq9wZx..
10:47:41.104 [FUND]  AorA..W8Ar 0.0719 SOL  sig=5mTr2vLb..
10:47:44.407 [BUY]   Ae8m..exZV 0.0270 SOL (initial)
10:47:48.118 [SELL]  C2K4..DkrT 41% tokens
10:47:51.723 [BUY]   Cx9M..AVbP 0.0134 SOL
10:47:55.025 [SELL]  AorA..W8Ar 63% tokens
10:47:58.327 [BUY]   C2K4..DkrT 0.0118 SOL
   ...
11:02:41.796 [DRAIN] Cx9M..AVbP 0.0581 SOL  sig=2zHn7cQe..
11:02:42.901 [JOB]   done — 214 buys, 141 sells, 0.9861 SOL spent
```

---

<a id="01--quick-start"></a>

```text
╔════════════════════════════════════════════════════════════════════════════╗
║  [01]  QUICK START                                                         ║
╚════════════════════════════════════════════════════════════════════════════╝
```

```bash
git clone https://github.com/zostaff/PUMPNUKE.git && cd PUMPNUKE
npm install
cp .env.example .env
```

```env
RPC_URL=https://mainnet.helius-rpc.com/?api-key=YOUR_KEY
WALLET_PRIVATE_KEY=your_base58_private_key
DRY_RUN=false
```

```bash
node cli/nuke.mjs --mint <TOKEN_CA> --budget 0.5 --wallets 8 --strategy staggered
node dashboard/server.mjs        # -> http://localhost:3002
```

---

<a id="02--how-it-works"></a>

```text
╔════════════════════════════════════════════════════════════════════════════╗
║  [02]  HOW IT WORKS                                                        ║
╚════════════════════════════════════════════════════════════════════════════╝
```

```text
 ┌──────────┐    ┌────────────┐    ┌─────────────┐    ┌─────────────────┐    ┌──────────────┐
 │  MASTER  │───>│ FUND POOL  │───>│ INITIAL BUY │───>│  BUY  <->  SELL │───>│  SELL ALL    │
 │  wallet  │    │ N wallets  │    │ 30-50% bal  │    │ rotate · delay  │    │  DRAIN  ->   │
 └──────────┘    └────────────┘    └─────────────┘    └───┬─────────▲───┘    │  MASTER      │
                                                          │         │        └──────────────┘
      PHASE 0         PHASE 1           PHASE 2           └─────────┘           CLEANUP
                                                        until duration ends
```

| phase | action | details |
|:---:|---|---|
| `0` | master holds the budget | balance checked before start |
| `1` | **fund** pool wallets | equal split, 500 ms between transfers |
| `2` | **initial buy** | 30–50% of each wallet's SOL, timing per strategy |
| `3` | **cycling** | rotating buy/sell until duration ends |
| `·` | **cleanup** | `sellAll` → `drainWallets` → master |

```text
  loop until now >= endTime  or  job.stopping
  │
  ├─ pool  = wallets − previous
  ├─ pool  = sort by lastUsedAt  (oldest first)
  ├─ pick  = random(pool[0..2])
  ├─ isBuy = !pick.hasTokens  ||  random() < buyBias
  │
  ├─ BUY  ─> 30% of (balance − fee reserve)
  ├─ SELL ─> 40–70% of token balance
  │
  ├─ error ─> consecutiveErrors++  ─>  5 ? STOP
  └─ sleep(strategy.cycleDelay())
```

---

<a id="03--strategies"></a>

```text
╔════════════════════════════════════════════════════════════════════════════╗
║  [03]  STRATEGIES                                                          ║
╚════════════════════════════════════════════════════════════════════════════╝
```

```text
  BASIC ............ initial: all at once ............ cycle delay: 1-3s
  t=0 ─┬──────────────────────────────────────────────────────────────>
       o   B S B B S B  S B B S B B S  B S B B S B B S B  S B B S B
       o
       o
       o
       o

  STAGGERED ........ initial: 2-6s apart ............ cycle delay: 2-8s
  t=0 ─┬───┬───┬───┬───┬───┬──────────────────────────────────────────>
       o   o   o   o   o   o     B     S      B    B       S     B   S

  legend:  o initial buy    B buy    S sell
```

| | `basic` | `staggered` |
|---|:---:|:---:|
| distribution | equal | equal |
| initial buy | all at once | 2–6 s apart |
| trade delay | 1–3 s | 2–8 s |
| density | high | medium |
| default | | ✔ |

---

<a id="04--cli"></a>

```text
╔════════════════════════════════════════════════════════════════════════════╗
║  [04]  CLI                                                                 ║
╚════════════════════════════════════════════════════════════════════════════╝
```

```text
┌─────────────────────────────────────┬────────────────────────────────────────┐
│ node cli/nuke.mjs --mint <CA>       │ preview -> confirm -> start            │
│ node cli/status.mjs [jobId]         │ all jobs / details + last 10 trades    │
│ node cli/panic.mjs <jobId>          │ sell every token, then drain           │
│ node cli/drain.mjs <jobId>          │ SOL back to master, no selling         │
└─────────────────────────────────────┴────────────────────────────────────────┘
```

| flag | description | default |
|---|---|:---:|
| `-m, --mint` | token contract address | **required** |
| `-b, --budget` | total SOL budget | `0.5` |
| `-w, --wallets` | wallets in the pool | `8` |
| `-s, --strategy` | `basic` \| `staggered` | `staggered` |
| `-d, --duration` | run length, minutes | `15` |
| `--bias` | buy probability `0–1` | `0.6` |
| `--slippage` | slippage, bps | `1000` |
| `-y, --yes` | skip confirmation | `false` |

```bash
npm run nuke -- --mint <CA> --budget 0.3
npm run status
npm run panic -- <jobId>
npm run drain -- <jobId>
npm run dashboard
```

---

<a id="05--dashboard"></a>

```text
╔════════════════════════════════════════════════════════════════════════════╗
║  [05]  DASHBOARD                                                           ║
╚════════════════════════════════════════════════════════════════════════════╝
```

```text
┌─ PUMPNUKE ────────────────────────────────────────────────────────── LIVE ─┐
│   BUYS 114     SELLS 85     CYCLES 195     VOLUME 3.32 SOL                 │
├─────────────────────────────────────────────┬──────────────────────────────┤
│ BUY / SELL FLOW                             │ WALLET POOL                  │
│                                             │ Cx9M..AVbP ████████░░ 0.066  │
│ buy   ▃▅▂▇▄▆█▃▅▄█▆▃▇▄█▃▆▅▇▂▆█▄▇▅█▆▄▇        │ C2K4..DkrT ██████░░░░ 0.051  │
│ sell  ▁▂▃▁▂▂▃▂▁▂▄▂▃▂▂▄▁▃▂▃▁▂▄▂▃▁▄▂▁▃        │ AorA..W8Ar █████████░ 0.073  │
│                                             │ Ae8m..exZV █████░░░░░ 0.038  │
│                                             │ 3UdP..DKUH ███████░░░ 0.058  │
│                                             │ HtUc..vUMK ██████░░░░ 0.046  │
├─────────────────────────────────────────────┴──────────────────────────────┤
│ TRADE LOG                                                                  │
│   10:48:15  SELL  HtUc..vUMK  58%          sig=4xQ9..mT2a                  │
│   10:48:15  BUY   3UdP..DKUH  0.0470 SOL   sig=2bLk..9vPe                  │
│   10:48:14  SELL  HtUc..vUMK  63%          sig=5nRw..Zc1d                  │
│   10:48:14  BUY   Ccif..ZkgK  0.0260 SOL   sig=3hTy..Qa8k                  │
├────────────────────────────────────────────────────────────────────────────┤
│   [ START ]   [ STOP ]   [ PANIC ]                 auto-refresh 3s         │
└────────────────────────────────────────────────────────────────────────────┘
```

| method | endpoint | description |
|:---:|---|---|
| `GET` | `/api/jobs` | all jobs |
| `GET` | `/api/jobs/:id` | details, public keys, last 50 trades |
| `POST` | `/api/jobs` | create + start a job |
| `POST` | `/api/jobs/:id/stop` | graceful stop |
| `POST` | `/api/jobs/:id/panic` | panic sell + drain |

```bash
curl -X POST http://localhost:3002/api/jobs \
  -H 'Content-Type: application/json' \
  -d '{"mint":"<CA>","budget":0.3,"numWallets":6,"strategy":"basic","durationMin":10}'
```

---

<a id="06--configuration"></a>

```text
╔════════════════════════════════════════════════════════════════════════════╗
║  [06]  CONFIGURATION                                                       ║
╚════════════════════════════════════════════════════════════════════════════╝
```

| variable | description | default |
|---|---|:---:|
| `RPC_URL` | Solana RPC endpoint | public mainnet |
| `WALLET_PRIVATE_KEY` | master wallet, base58 | — |
| `DRY_RUN` | set to `false` | `false` |
| `DEFAULT_STRATEGY` | `basic` / `staggered` | `staggered` |
| `DEFAULT_WALLETS` | pool size | `8` |
| `DEFAULT_BUDGET` | budget, SOL | `0.5` |
| `DEFAULT_DURATION_MIN` | run length, minutes | `15` |
| `DEFAULT_BUY_BIAS` | buy probability | `0.6` |
| `DEFAULT_SLIPPAGE_BPS` | slippage, bps | `1000` |
| `PRIORITY_FEE` | compute unit price, µLamports | `300000` |
| `JITO_TIP` | Jito tip, SOL | `0.0001` |
| `DASHBOARD_PORT` | dashboard port | `3002` |
| `SOL_RESERVE` | SOL kept per wallet for fees | `0.01` |

---

<a id="07--under-the-hood"></a>

```text
╔════════════════════════════════════════════════════════════════════════════╗
║  [07]  UNDER THE HOOD                                                      ║
╚════════════════════════════════════════════════════════════════════════════╝
```

```text
pumpnuke/
├── lib/
│   ├── config.mjs ............ env, RPC connection, master keypair
│   ├── constants.mjs ......... compute limits, Jito endpoint, tip accounts
│   ├── logger.mjs ............ ANSI: buy green · sell red · fund yellow · drain blue
│   ├── wallet-pool.mjs ....... generateWallets · fundWallets · drainWallets · sellAll
│   ├── trade-builder.mjs ..... buildBuy / buildSell — pump-sdk + pump-swap-sdk
│   ├── volume-engine.mjs ..... runVolume — fund → initial buy → cycling → cleanup
│   ├── job-manager.mjs ....... prepare → confirm → run → stop · persistence · SIGINT
│   └── jito.mjs .............. bundle submit + status polling
├── cli/
│   ├── nuke.mjs .............. start a job
│   ├── status.mjs ............ list / inspect jobs
│   ├── panic.mjs ............. sell everything + drain
│   └── drain.mjs ............. SOL back to master
└── dashboard/
    ├── server.mjs ............ HTTP API on :3002
    └── public/index.html ..... live UI
```

```text
  prepared ──confirm──> funding ──> buying ──> running ──> cleanup ──> completed
                           │          │          │
                           └──────────┴───stop───┴─────────────────> stopped
                                              panic ───────────────> panic-stopped
                                    fatal error ── auto drain ─────> failed
```

```text
  TX      VersionedTransaction v0 · CU limit 250 000 · CU price = PRIORITY_FEE
  VENUE   bonding curve -> buyV2 / sellV2   ·   graduated -> PumpSwap
  TOKEN   SPL / Token-2022 auto-detected from mint owner
  MATH    BigInt everywhere · SOL in lamports
  STATE   one active job · saved every 10 s to data/jobs/<id>.json
  SAFETY  5 consecutive errors -> stop · Ctrl+C -> sell all -> drain -> stats
```

---

<a id="08--faq"></a>

```text
╔════════════════════════════════════════════════════════════════════════════╗
║  [08]  FAQ                                                                 ║
╚════════════════════════════════════════════════════════════════════════════╝
```

<details>
<summary><b>> Ctrl+C mid-run?</b></summary>

All active jobs stop, remaining tokens are sold, SOL is drained to master and session stats are printed. Press twice to force-quit.
</details>

<details>
<summary><b>> Job stopped by itself?</b></summary>

Five failed trades in a row trigger auto-stop — dead token, drained curve or RPC issues. Check `node cli/status.mjs <jobId>`.
</details>

<details>
<summary><b>> Where are the wallet keys?</b></summary>

`data/jobs/<id>.json`, plaintext, git-ignored. `node cli/drain.mjs <id>` recovers SOL from any job.
</details>

<details>
<summary><b>> Token graduated mid-run?</b></summary>

The trade builder checks the curve on every trade and switches to PumpSwap automatically.
</details>

---

```text
┌──────────────────────────────────────┬──────────────────────────────────────┐
│ [+] CAN                              │ [-] CANNOT                           │
├──────────────────────────────────────┼──────────────────────────────────────┤
│ buy/sell cycles across 2-20 wallets  │ guarantee profit                     │
│ trade on curve and PumpSwap          │ launch new tokens                    │
│ panic sell + drain in one command    │ tune priority fee dynamically        │
│ recover SOL from any saved job       │ encrypt stored wallet keys           │
│ monitor everything from a dashboard  │ recover keys if data/ is deleted     │
└──────────────────────────────────────┴──────────────────────────────────────┘
```

```text
[!] DISCLAIMER
    Crypto trading carries a high risk of loss. Artificial volume misleads other
    market participants and may be illegal in your jurisdiction or against
    platform terms. You are solely responsible for how you use this software.
```

<div align="center">

```text
·  ·  ·  ~=~=~=<[ PUMP | NUKE ]>  ·  ·  ·
```

<sub>built by <a href="https://github.com/zostaff">zostaff</a></sub>

</div>
