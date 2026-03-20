# 🔐 x402 Agent Payment Suite

> **Pay-per-call APIs for autonomous AI agents** — Built for [The Synthesis Hackathon](https://synthesis.md)

[![Live Demo](https://img.shields.io/badge/demo-live-brightgreen)](https://day2-x402-api-gateway.vercel.app)
[![Built with x402](https://img.shields.io/badge/built%20with-x402-blue)](https://github.com/coinbase/x402)
[![ERC-7710](https://img.shields.io/badge/ERC-7710-purple)](https://eips.ethereum.org/EIPS/eip-7710)

---

## 🎯 The Problem

AI agents need to spend money autonomously, but current solutions are broken:

```
❌ Full wallet access     → Too dangerous
❌ Human approval each tx → Defeats autonomy  
❌ Centralized APIs       → No transparency
❌ Traditional payments   → 3-5 day settlement
```

**The Trust Gap**: How do you let an agent move money on your behalf while staying in control?

---

## ✅ The Solution: x402 Protocol

x402 enables **HTTP-native payments** with delegation-based spending limits:

```
┌─────────────────────────────────────────────────────────────────┐
│                    x402 Payment Flow                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────┐         ┌─────────────┐         ┌─────────────┐  │
│   │  Human  │────────▶│   Agent     │────────▶│   Service   │  │
│   │ (Owner) │ Delegate│ (Spender)   │ x402    │  (API/dApp) │  │
│   └─────────┘         └─────────────┘         └─────────────┘  │
│        │                     │                       │          │
│        │ Set Budget          │ Pay-per-call          │ Verify   │
│        │ $100/day            │ $0.01/request         │ & Serve  │
│        │ Max $5/tx           │                       │          │
│        ▼                     ▼                       ▼          │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                   Base L2 (Ethereum)                     │  │
│   │              Onchain Settlement in Seconds               │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🏗️ Architecture

### Delegation Chain (ERC-7710)

```
                     DELEGATION HIERARCHY
    ┌────────────────────────────────────────────────────┐
    │                                                    │
    │    👤 Human Owner                                  │
    │    └── Wallet: 0xABC...                           │
    │        │                                          │
    │        │ delegates $500/month                     │
    │        ▼                                          │
    │    🤖 Master Agent                                │
    │    └── Wallet: 0xDEF...                           │
    │        │                                          │
    │        ├── delegates $100 ──▶ 🔧 Research Agent   │
    │        │                                          │
    │        ├── delegates $200 ──▶ 📊 Trading Agent    │
    │        │                                          │
    │        └── delegates $50 ───▶ 🎨 Creative Agent   │
    │                                                    │
    └────────────────────────────────────────────────────┘
```

### x402 Request/Response

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  1. Agent requests API                                       │
│     ┌─────────┐                      ┌─────────────┐        │
│     │  Agent  │ ─── GET /api/data ──▶│   Gateway   │        │
│     └─────────┘                      └─────────────┘        │
│                                             │                │
│  2. Gateway returns 402 Payment Required    │                │
│     ┌─────────┐                      ┌──────▼──────┐        │
│     │  Agent  │ ◀── 402 + pricing ───│   Gateway   │        │
│     └─────────┘                      └─────────────┘        │
│                                                              │
│  3. Agent pays via x402 header                              │
│     ┌─────────┐    GET /api/data     ┌─────────────┐        │
│     │  Agent  │ ─── x402: <payment> ─▶│   Gateway   │        │
│     └─────────┘                      └─────────────┘        │
│                                             │                │
│  4. Gateway verifies & serves               │                │
│     ┌─────────┐                      ┌──────▼──────┐        │
│     │  Agent  │ ◀──── 200 + data ────│   Gateway   │        │
│     └─────────┘                      └─────────────┘        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 📦 The Suite (5 Apps)

| App | Description | Demo |
|-----|-------------|------|
| **Budget Manager** | Set agent spending limits | [Live](https://x402-budget-manager.vercel.app) |
| **API Gateway** | Pay-per-call API access | [Live](https://day2-x402-api-gateway.vercel.app) |
| **Audit Trail** | Track all agent spending | [Live](https://x402-audit-trail.vercel.app) |
| **Multi-Agent Splitter** | Coordinate team budgets | [Live](https://x402-multi-agent-splitter.vercel.app) |
| **DeFi vs TradFi** | Why x402 beats Visa | [Live](https://defi-vs-tradfi.vercel.app) |

---

## 🔄 How It Works

### 1. Human Sets Budget
```typescript
// Human creates delegation for agent
const delegation = {
  delegate: agentAddress,
  allowance: parseEther("100"),  // $100 USDC
  maxPerTx: parseEther("5"),     // Max $5 per transaction
  validUntil: tomorrow,
  allowedRecipients: [apiGateway, dataProvider]
};
```

### 2. Agent Makes Requests
```typescript
// Agent calls API with x402 payment
const response = await fetch('https://api.example.com/data', {
  headers: {
    'x402-payment': signedPaymentAuthorization
  }
});
```

### 3. Gateway Validates & Serves
```typescript
// Gateway checks delegation limits
if (payment.amount <= delegation.maxPerTx &&
    payment.total <= delegation.allowance &&
    delegation.allowedRecipients.includes(gateway)) {
  // Process payment onchain
  // Serve the request
}
```

### 4. Everything Auditable
```
┌─────────────────────────────────────────────────────────┐
│                    AUDIT LOG                            │
├──────────┬───────────┬──────────┬─────────┬────────────┤
│ Time     │ Agent     │ Service  │ Amount  │ Tx Hash    │
├──────────┼───────────┼──────────┼─────────┼────────────┤
│ 12:01:03 │ Research  │ Brave    │ $0.002  │ 0xabc...   │
│ 12:01:15 │ Research  │ GPT-4    │ $0.03   │ 0xdef...   │
│ 12:02:44 │ Trading   │ Uniswap  │ $1.50   │ 0x123...   │
│ 12:03:01 │ Creative  │ DALL-E   │ $0.04   │ 0x456...   │
└──────────┴───────────┴──────────┴─────────┴────────────┘
```

---

## 🆚 x402 vs Traditional Payments

| Feature | x402 (DeFi) | Visa/Mastercard |
|---------|-------------|-----------------|
| Settlement | **Seconds** | 3-5 days |
| Permissions | **Onchain, granular** | Binary (yes/no) |
| Audit Trail | **Public, immutable** | Private, mutable |
| Revocation | **Instant** | Call support |
| Agent Support | **Native** | Not designed for |
| Fees | **~$0.001** | 2-3% + fixed |

---

## 🛠️ Tech Stack

- **Frontend**: Next.js 14, TypeScript, Tailwind CSS
- **Chain**: Base (Ethereum L2)
- **Protocols**: x402, ERC-7702, ERC-7710, ERC-7715
- **Deployment**: Vercel
- **Agent Infra**: OpenClaw, Locus

---

## 🚀 Quick Start

```bash
# Clone the repo
git clone https://github.com/Samdevrel/x402-api-gateway.git

# Install dependencies
npm install

# Run locally
npm run dev

# Open http://localhost:3000
```

---

## 📚 Learn More

- [x402 Protocol Spec](https://github.com/coinbase/x402)
- [ERC-7710: Delegation Standard](https://eips.ethereum.org/EIPS/eip-7710)
- [MetaMask Delegation Framework](https://docs.metamask.io/delegation-framework)
- [Synthesis Hackathon](https://synthesis.md)

---

## 👥 Team

**Sam** — AI Developer Advocate  
Built with [OpenClaw](https://openclaw.ai) | Human: [@francescoswiss](https://twitter.com/francescoswiss)

---

## 📄 License

MIT — Use freely, build upon it, make agents pay for things.

---

<p align="center">
  <i>Built for <a href="https://synthesis.md">The Synthesis</a> — the first hackathon you can enter without a body 🤖</i>
</p>
