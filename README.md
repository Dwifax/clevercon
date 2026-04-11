<div align="center">

```
   ██████╗██╗     ███████╗██╗   ██╗███████╗██████╗  ██████╗ ██████╗ ███╗   ██╗
  ██╔════╝██║     ██╔════╝██║   ██║██╔════╝██╔══██╗██╔════╝██╔═══██╗████╗  ██║
  ██║     ██║     █████╗  ██║   ██║█████╗  ██████╔╝██║     ██║   ██║██╔██╗ ██║
  ██║     ██║     ██╔══╝  ╚██╗ ██╔╝██╔══╝  ██╔══██╗██║     ██║   ██║██║╚██╗██║
  ╚██████╗███████╗███████╗ ╚████╔╝ ███████╗██║  ██║╚██████╗╚██████╔╝██║ ╚████║
   ╚═════╝╚══════╝╚══════╝  ╚═══╝  ╚══════╝╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚═╝  ╚═══╝
```

**The AI agent marketplace where every payment is real, every budget is on-chain, and you — not the operator — hold the funds.**

[![Stellar Testnet](https://img.shields.io/badge/Stellar-Testnet-3DAAFF?logo=stellar&logoColor=white&style=flat-square)](https://stellar.org)
[![Soroban](https://img.shields.io/badge/Soroban-2%20Contracts%20Live-A855F7?style=flat-square)](https://soroban.stellar.org)
[![x402](https://img.shields.io/badge/x402-Live%20Payments-F97316?style=flat-square)](https://www.x402.org)
[![MPP](https://img.shields.io/badge/MPP-Streaming%20Live-10B981?style=flat-square)](https://stripe.com/machine-payments)
[![USDC](https://img.shields.io/badge/USDC-Not%20XLM-2775CA?style=flat-square)](https://stellar.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-gray?style=flat-square)](LICENSE)

</div>

---

## The honest version of this idea

Many projects tackle AI + Stellar payments. Most demonstrate the concept with simulated responses, simplified verification, and no smart contract custody. Soroban integration, real MPP, and stablecoin payments are usually listed as *"future work."*

CleverCon ships that future work as the present.

| What most projects plan | What CleverCon ships |
|---|---|
| x402 with simplified verification | x402 via the production facilitator, real USDC |
| Simulated agent responses | Live Claude Sonnet + Haiku for planning and rating |
| XLM micropayments | USDC stablecoin — agents earn and spend stable value |
| Soroban as future work | **Two Soroban contracts deployed to testnet** |
| MPP as future work | **MPP streaming payments live** |
| No fund custody | **CleverVault holds user USDC — not the operator** |
| Soft budget limits in code | **BudgetGuardian approves every spend on-chain** |
| Single agent | Per-user named orchestrators with their own Stellar wallets |
| No wallet integration | Freighter · xBull · Albedo · LOBSTR · Rabet |
| No persistent state | JSON stores + on-chain state, survives restarts |

---

## What is CleverCon?

CleverCon is a **multi-agent AI marketplace** built on Stellar. You connect your wallet, deposit USDC into a smart contract vault, name your personal orchestrator, and give it tasks in plain English.

The orchestrator — powered by Claude Sonnet — decomposes your request into steps, recruits specialist agents from an open registry, and pays them per call in real USDC via x402 and MPP. Every payment is on-chain. Your funds never touch an operator wallet until they're released by the contract for a specific step.

```
You type:  "Research XLM price, analyze the trend, write a briefing"

  Planner decomposes:
    Step 1 → StellarOracle   $0.020  x402   "fetch live DEX data"
    Step 2 → AnalysisBot     $0.050  MPP    "analyze price trend"
    Step 3 → ReporterBot     $0.030  x402   "format as briefing"
    ─────────────────────────────────────────────────────
    Total estimated cost: $0.100   [ Approve ] [ Reject ]

  You approve.

  BudgetGuardian (Soroban) → create_task(budget=$0.100)  ← on-chain
  CleverVault   (Soroban) → lock $0.100 from your balance ← on-chain

  Step 1 executes:
    BudgetGuardian.approve_spend(task_42, $0.020) → true   ← on-chain
    CleverVault.release_payment(task_42, $0.020)           ← on-chain
    POST /query + x402 → StellarOracle
    Claude Haiku rates response: 4/5 → reputation updated

  Step 2 → Step 3 → task complete → receipt saved
  Unspent budget refunded. On-chain. Automatically.
```

Three agents. Three Stellar transactions. One verifiable on-chain receipt.

---

## Architecture

```
╔══════════════════════════════════════════════════════════════════════╗
║                     Dashboard  (React 19 + Tailwind)                ║
║  Task queue  ·  Activity feed  ·  CleverVault  ·  History  ·  Agents ║
╚══════════════════════════╤═══════════════════════════════════════════╝
                           │  WebSocket + REST
╔══════════════════════════▼═══════════════════════════════════════════╗
║                        Orchestrator  (port 3000)                     ║
║  Planner (Claude Sonnet)  ·  Selector  ·  Executor  ·  Rater (Haiku) ║
╚═══════╤══════════════════════════════════════╤════════════════════════╝
        │                                      │
        │  HTTP / REST                         │  Soroban RPC
╔═══════▼═══════╗                ╔════════════▼══════════════════════╗
║  Registry     ║                ║       Stellar Testnet              ║
║  port 4000    ║                ║                                    ║
║               ║                ║  ┌──────────────────────────────┐ ║
║  manifests    ║                ║  │     CleverVault Contract     │ ║
║  reputation   ║                ║  │  Holds user USDC · releases  │ ║
║  ownership    ║                ║  │  per-step · enforces locks   │ ║
╚═══════╤═══════╝                ║  └──────────────────────────────┘ ║
        │                        ║  ┌──────────────────────────────┐ ║
        │ self-register           ║  │   BudgetGuardian Contract    │ ║
        │ on startup              ║  │  Approves every spend · hard │ ║
        │                        ║  │  on-chain ceiling per task   │ ║
╔═══════▼═══════════════════════╗║  └──────────────────────────────┘ ║
║          Agent Network         ║╚════════════════════════════════════╝
║                                ║
║  StellarOracle   port 4001  x402  $0.020/call                        ║
║  WebIntel v1     port 4002  x402  $0.010/call                        ║
║  WebIntel v2     port 4003  x402  $0.010/call                        ║
║  AnalysisBot     port 4004  MPP   $0.050/call  (streaming)           ║
║  ReporterBot     port 4005  x402  $0.030/call                        ║
║                                                                      ║
║  + any agent you register via the dashboard                          ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## The Two Soroban Contracts

### CleverVault — Trustless USDC Treasury

Users deposit USDC directly into this contract. The operator never holds user funds.

```
           User Wallet
               │  deposit()  — user signs in their wallet
               ▼
    ┌─────────────────────────┐
    │    CleverVault (Soroban) │  ← holds USDC, not the operator
    │                          │
    │  balance     10.000 USDC │
    │  locked       0.100 USDC │  ← reserved for active task
    │  available    9.900 USDC │
    └────────────┬─────────────┘
                 │  release_payment()  — orchestrator signs, per step
                 ▼
    Orchestrator wallet  (~$0 normally — just a relay)
                 │  x402 payment
                 ▼
    Agent wallet  (earns USDC per call, verified on stellar.expert)
```

**On-chain invariants enforced by the contract (not by the app):**
- `create_task` blocked if available balance < plan cost
- `release_payment` blocked if it would exceed the task budget
- `withdraw` blocked while a task is active
- `force_complete` available after 30 min — no funds ever permanently locked

### BudgetGuardian — Spend Enforcement

A second contract that must approve every payment before USDC moves.

```
Before every agent payment:
  orchestrator → BudgetGuardian.approve_spend(task_id, $0.020)
                                                │
                                           true ──▶ payment proceeds
                                           false ─▶ step denied, task stops
```

The budget ceiling is **a Soroban constraint, not a JavaScript if-statement**.

---

## Payment Protocols — Both Live

### x402 — Per-Call HTTP Micropayments

```
Orchestrator                          StellarOracle
     │                                     │
     │──── POST /query ───────────────────▶│
     │                                     │
     │◀─── 402 Payment Required ───────────│
     │      { amount: "0.02",              │
     │        currency: "USDC",            │
     │        address: "G...",             │
     │        network: "stellar:testnet" } │
     │                                     │
     │  [CleverVault releases $0.02]       │
     │  [BudgetGuardian approves spend]    │
     │                                     │
     │──── POST /query ───────────────────▶│
     │      X-Payment: <tx_hash>           │
     │                                     │
     │◀─── 200 OK + data ─────────────────│
```

Used by: StellarOracle, WebIntel v1/v2, ReporterBot

### MPP — Machine Payment Protocol (Streaming)

Pre-authorized session for longer tasks. AnalysisBot opens a payment channel, streams analysis output, final amount settled per usage.

Used by: AnalysisBot

---

## Features in Full

### 🏛️ Personal AI Orchestrators
Every user gets their own named orchestrator — a dedicated Stellar wallet + Claude Sonnet brain. Orchestrators are registered in CleverVault on-chain, binding the user's wallet to their orchestrator's address.

### 🏪 Open Agent Registry
Anyone can register a specialist agent and earn USDC per task. The registry is open — no gatekeeping.

- Agents self-register with the registry on startup — always current
- Ownership-controlled: only the registrant's wallet can rename or remove their agent
- **Sponsored wallet provisioning**: new agents receive a funded Stellar wallet automatically — no XLM needed to start

### 📈 Reputation Engine
Elo-style scoring updated after every completed job.

| Factor | Weight | How measured |
|---|---|---|
| Capability match | 40% | Advertised capability tags vs task requirements |
| Reputation score | 25% | 0–100 running score, updated per job |
| Price efficiency | 20% | Cost vs budget; cheaper wins when quality is equal |
| Latency | 10% | Rolling average response time |
| Discovery bonus | 5% | Small lift for agents with few jobs |

Claude Haiku rates every agent response 1–5. A 5/5 raises the score; a 1/5 lowers it. Better agents surface higher automatically.

### 🗓️ Task Queue with Scheduling
Submit multiple tasks in advance with configurable delays between them. First task runs immediately; subsequent tasks wait their scheduled interval.

### 💳 Multi-Wallet Support
```
Detected wallets on page load:

  ✓ Freighter      Ready to connect
  ✓ xBull          Ready to connect
  ✓ Albedo         Ready to connect
  ○ LOBSTR         Not installed  ↗
  ○ Rabet          Not installed  ↗
```
Wallet detection polls three times over 2 seconds to give extensions time to inject — no false "not available."

### 📊 Financial Dashboard
Real vault transaction history — every deposit, withdrawal, and agent payment — with Stellar Explorer links on every row. No simulated numbers.

### 📋 Task History
Every completed task is persisted with its full output and a line-item receipt. Expand any task to see which agents ran, what they were paid, and links to the on-chain transactions.

---

## Stellar Integrations

| Integration | Status | Details |
|---|---|---|
| USDC stablecoin payments | ✅ Live | Every agent earns USDC, not volatile XLM |
| x402 per-call payments | ✅ Live | 4 of 5 agents, real facilitator verification |
| MPP streaming payments | ✅ Live | AnalysisBot, session-based |
| CleverVault — Soroban contract | ✅ Deployed | Holds user USDC, step-by-step release |
| BudgetGuardian — Soroban contract | ✅ Deployed | Hard on-chain spend ceiling per task |
| Horizon API (live data) | ✅ Live | StellarOracle: DEX, orderbooks, prices, stats |
| Sponsored account creation | ✅ Live | New agents get funded wallets automatically |
| USDC trustline management | ✅ Live | Setup scripts + in-dashboard UI |
| On-chain orchestrator registration | ✅ Live | User signs `register_orchestrator` in wallet |
| Multi-wallet connector | ✅ Live | Freighter, xBull, Albedo, LOBSTR, Rabet |
| Real tx_hash on every payment | ✅ Live | Every payment links to stellar.expert |
| Per-user data isolation | ✅ Live | All data filtered by connected wallet address |

---

## Quick Start

### Prerequisites
- Node.js 20+
- Anthropic API key
- A Stellar wallet browser extension (Freighter recommended for testnet)

### 1 — Clone and install

```bash
git clone git@github.com:Bosun-Josh121/clevercon.git
cd clevercon
npm install
```

### 2 — Configure

```bash
cp .env.example .env
# Open .env and add your ANTHROPIC_API_KEY
```

### 3 — Set up wallets (first time only)

```bash
npx tsx scripts/setup-wallets.ts         # Generate keypairs → writes to .env
npx tsx scripts/add-usdc-trustlines.ts   # Add USDC trustlines to each wallet
npx tsx scripts/distribute-usdc.ts       # Fund agent wallets with testnet USDC
```

### 4 — Start everything

```bash
./scripts/start.sh
```

One command. It:
- Kills any stale processes on ports 3000–4005
- Builds the React dashboard (Vite)
- Compiles the orchestrator (esbuild ESM)
- Starts the registry → waits for health ✓
- Starts all 5 agents → waits for self-registration ✓
- Starts the orchestrator → waits for health ✓
- Prints the URL

Open **http://localhost:3000**, connect your Freighter wallet (testnet), and submit a task.

### 5 — Stop

```bash
./scripts/stop.sh
```

### Optional — Seed reputation data

```bash
npx tsx scripts/bootstrap.ts --auto-approve
# Runs 25 diverse tasks to build agent reputation history
```

---

## Deploy the Soroban Contracts

Requires Rust + `stellar-cli` 25+:

```bash
# Trustless USDC treasury
cd contracts/agent-vault && ./deploy.sh
# → writes AGENT_VAULT_CONTRACT_ID to .env

# On-chain spend enforcement
cd contracts/budget-guardian && ./deploy.sh
# → writes BUDGET_CONTRACT_ID to .env
```

Each script: build Rust → WASM → deploy → init → smoke-test → auto-update `.env`.

**Testnet USDC SAC:** `CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA`

---

## Environment Variables

```bash
# Required
ANTHROPIC_API_KEY=sk-ant-...

# Stellar wallets (generated by setup-wallets.ts)
ORCHESTRATOR_SECRET_KEY=S...
STELLAR_ORACLE_SECRET_KEY=S...
WEB_INTEL_SECRET_KEY=S...
WEB_INTEL_V2_SECRET_KEY=S...
ANALYSIS_AGENT_SECRET_KEY=S...
REPORT_AGENT_SECRET_KEY=S...

# Smart contracts (written by deploy.sh)
AGENT_VAULT_CONTRACT_ID=C...
BUDGET_CONTRACT_ID=C...

# Optional
ORCHESTRATOR_PORT=3000
REGISTRY_PORT=4000
PLAN_APPROVAL_TIMEOUT_MS=60000
DEFAULT_BUDGET=1.0
```

---

## Register Your Own Agent

Go to **Register Agent** in the dashboard:

| Field | Notes |
|---|---|
| Agent ID | Lowercase, hyphens (`my-agent`) |
| Endpoint | Your service's query URL |
| Stellar Address | Paste a `G...` key or tick **Provision wallet** for automatic setup |
| Capabilities | Comma-separated tags used by the orchestrator for matching |
| Payment model | `x402` (per-call) or `mpp` (session-based) |
| Price per call | USDC amount |

Your endpoint must implement:

```
GET  /health  →  { "status": "ok" }

POST /your-endpoint
  Without payment:   return 402 + payment details
  With X-Payment:    return 200 + your data
```

Once registered, the orchestrator hires your agent automatically when its capabilities match a task. Your `registered_by` address owns the listing — only you can rename or remove it.

---

## Project Structure

```
clevercon/
├── contracts/
│   ├── agent-vault/           CleverVault — trustless USDC treasury (Soroban/Rust)
│   └── budget-guardian/       BudgetGuardian — spend enforcement (Soroban/Rust)
├── packages/
│   ├── agents/
│   │   ├── stellar-oracle/    Live Stellar/Horizon data (x402)
│   │   ├── web-intel/         Blockchain news v1 (x402)
│   │   ├── web-intel-v2/      Blockchain news v2 (x402)
│   │   ├── analysis/          Claude analysis, streaming (MPP)
│   │   └── reporter/          Claude report formatting (x402)
│   ├── common/                Shared TypeScript types
│   ├── dashboard/             React 19 + Vite + Tailwind
│   ├── orchestrator/          Planner · Executor · Vault client · WS hub
│   └── registry/              Agent registry with reputation engine
├── scripts/
│   ├── start.sh               Start all services
│   ├── stop.sh                Stop all services
│   ├── bootstrap.ts           Seed 25 tasks for reputation history
│   ├── setup-wallets.ts       Generate Stellar keypairs
│   ├── add-usdc-trustlines.ts
│   └── distribute-usdc.ts
├── render.yaml                One-click Render deployment blueprint
└── docs/sections/             Phase-by-phase build documentation (13 phases)
```

---

## Deploy Free to Render

`render.yaml` is included. Push to GitHub, go to [render.com](https://render.com), click **New → Blueprint**, connect the repo. Seven services deploy from one config.

After the first deploy, update the `*_SELF_URL` and `REGISTRY_URL` env vars to the assigned `.onrender.com` URLs, then redeploy. Agents re-register themselves on startup — the registry self-heals.

---

## Hackathon Submission

**Stellar Hacks: Agents** · April 2026 · Open Innovation Track

**Every requirement — exceeded:**

| Requirement | Status |
|---|---|
| Open-source repo | ✅ This repo, MIT licensed |
| 2–3 min demo video | ✅ [Video link] |
| Stellar testnet interaction | ✅ Every agent call is a real Stellar transaction |

**What this demonstrates at the protocol level:**

- x402 HTTP micropayments with real USDC on Stellar testnet — not XLM, not simulated
- MPP machine-to-machine streaming payments
- Two Soroban smart contracts: one holding real user funds, one enforcing spend limits
- Live Horizon API queries for on-chain data
- Sponsored account creation making agent onboarding permissionless
- The full agent economy loop: deposit → plan → pay → earn — all on-chain

---

<div align="center">

**CleverCon** — Built for Stellar Hacks: Agents, April 2026

*The agents think. The contracts enforce. The blockchain settles.*
*Your funds stay yours until the work is done.*

</div>
