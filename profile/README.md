<div align="center">

<img src="../assets/banner.png" alt="Kovan — Agent autonomy. Human control." width="100%" />

<br /><br />

[![website][website-shield]][website-link]
[![agent guide][agent-shield]][agent-link]
[![sdk][sdk-shield]][sdk-link]
[![pitch deck][pitch-deck-shield]][pitch-deck-link]
[![github stars][github-star]][github-link]
[![built on Stellar][stellar-shield]][stellar-link]

</div>

### 👋 Welcome to Kovan Protocol

We're building **shared USDC budgets that AI agents can't overspend**.

Agents that spend need money to spend: data, APIs and compute bought per call, increasingly over [x402][x402-link]. Today you hand the agent a funded wallet and hope. A buggy, looping or prompt-injected agent can empty whatever it holds, and it has happened. **Kovan** is our answer: an owner locks USDC in one pool, and each agent gets its own Stellar smart account bound to an on-chain **covenant**: who it may pay, how much per payment, how much in total, and until when. The rules live in the agent's own account, so no bug, loop or prompt injection can talk its way past them.

Built on **Stellar** (Soroban smart contracts, OpenZeppelin smart accounts, x402) for **HackMeridian 2026** in Lisbon. Everything below is open source and runs on testnet today.

> *Humans configure. Agents only execute inside limits.*

### ⭐️ Our Projects

| Project | Description |
| :--- | :--- |
| [**🏦 Kovan**][kovan-github] | The monorepo: Soroban contracts (pool, covenant policy, agent account, verifier), the TypeScript SDK, the owner console, the agent tooling (`kovan.mjs`, Claude Code plugin, `agent.md`), the hosted signing and setup pages, and the website. [![][sdk-shield]][sdk-link] [![][contract-shield]][contract-link] |
| [**🌐 Website**][website-link] | How it works, security model, developer docs, an agent example and a playground. Also serves `/agent.md`, `/llms.txt`, `/sign` and `/setup` for agents. [![][website-shield]][website-link] |

### 📦 Ecosystem

| Repository / Package | Purpose | Language |
| :--- | :--- | :---: |
| [**kovan**][kovan-github] | Monorepo: contracts, SDK, console, agent tooling, signer, website | ![][lang-rust] ![][lang-typescript] |
| `contracts/pool` | `kovan-pool`: USDC custody, `sum_limits ≤ locked`, agent lifecycle, mandates, clawback, irreversible shutdown | ![][lang-rust] |
| `contracts/covenant-policy` | `kovan-covenant-policy`: OpenZeppelin policy that judges every outgoing call; payees, caps, expiry, allowed calls, mandates and receipts | ![][lang-rust] |
| `contracts/agent-account` | `kovan-agent-account`: OpenZeppelin smart account with one rule and no admin, one per agent | ![][lang-rust] |
| [`@kovan-protocol/sdk`][sdk-link] | Non-custodial TypeScript client: prepare → review → sign → submit, agent authorization signing, reads, decoded errors | ![][lang-typescript] |
| `dashboard/` | Owner console (Next.js): pools, agents, mandates, x402 payments, recovery, live event log | ![][lang-typescript] |
| `skills/kovan` + `.claude-plugin/` | Agent skill and Claude Code plugin: `kovan.mjs`, one self-contained Node file that prepares every transaction | ![][lang-typescript] |
| `signer/` | Hosted signing page (`/sign`) for cloud agents and one-link setup page (`/setup`) for chat assistants | ![][lang-typescript] |
| `web/` | Marketing site, docs, `agent.md` and `llms.txt` (Next.js on Vercel) | ![][lang-typescript] |

### 🏗️ How It Fits Together

```
        Owner (Freighter)                         LLM agent
              │                                       │
   fund · create_agent · create_mandate      kovan.mjs · SDK · console
   amend · revoke · close                    (prepares and signs agent actions)
              │                                       │
              ▼                                       ▼
      ┌───────────────┐   deploys    ┌──────────────────────────────┐
      │  kovan-pool   │ ───────────▶ │  kovan-agent-account         │
      │  USDC custody │              │  OpenZeppelin smart account   │
      │  sum_limits ≤ │ ◀─────────── │  1 rule · no admin           │
      │  locked       │   clawback   └──────────────┬───────────────┘
      └───────────────┘                             │ __check_auth
                                                    ▼
                                     ┌──────────────────────────────┐
                                     │  kovan-covenant-policy       │
                                     │  payees · caps · expiry      │
                                     │  calls · mandates · receipts │
                                     └──────────────┬───────────────┘
                                          allowed   │   refused
                                                    ▼
                                 USDC transfer (direct or x402)
                                 to an allowlisted provider
```

**Per-payment flow:** the agent runtime simulates `usdc.transfer(agent, payee, amount)` → the account's `__check_auth` verifies the agent's ed25519 signature → the covenant policy checks payee, per-payment cap, total cap, expiry and mandate → allowed payments are signed and submitted; anything else is refused in simulation and nothing is ever sent. The same `transfer` shape is judged whether it comes from a direct payment or an x402 settlement.

### 🎯 What Makes Kovan Different

| | A single capped wallet | Kovan |
| :--- | :--- | :--- |
| Budget source | One wallet per agent, funded by hand | One pool backs many agents; limits can never add up to more than the pool holds |
| Who can change the rules | Usually the deployer, sometimes the agent | Nobody. No admin rule; the owner can only tighten or loosen numbers, or revoke |
| Lost or stolen agent key | Funds gone or stuck | Owner claws funds back through an allowance granted at creation |
| Sub-agents | Not supported | An agent delegates a narrower budget out of its own; revoking a parent freezes its children |
| Payment intent | Not supported | Owner-approved mandates with single-use payment IDs and on-chain receipts |
| x402 | Classic account or none | A smart-account (C-address) payer, settled through the x402.org facilitator |
| Setup | A dashboard | The agent sets itself up; the owner approves in Freighter |

**The wedge:** every earlier Stellar hackathon project put a spending limit on a single agent wallet. Kovan is the treasury layer around them: one pool, many agents, rules the agent's own account enforces.

### 🔐 Core Features (current release)

- **Shared pool with a hard invariant:** `sum_limits ≤ locked` on every `create_agent`, `delegate` and `amend`.
- **Frozen agent accounts:** one context rule (agent key + covenant policy), zero management rules. Nobody can add a key or remove the policy.
- **Covenant policy:** USDC `transfer` only to allowlisted providers and recipients, within per-transfer and total caps, before expiry. Everything else refused before any money moves.
- **Kill switch and clawback:** `revoke` freezes an agent and its children; `return_funds` and `close` recover USDC without the agent's key.
- **Delegation:** child covenants carved out of a parent's remaining allowance, bounded depth and count.
- **Payment mandates:** owner-approved `exact` or `budget` grants with intent hashes, deadlines, single-use payment IDs and on-chain receipts.
- **x402 from a smart account:** `AuthPayload` built for the agent's C-address, quote binding, a request journal so retries never pay twice, receipts confirmed against Stellar RPC.
- **Agent-native setup:** Claude Code plugin, `agent.md` for any agent, one link for chat assistants, a hosted signing page for cloud agents.

### 🔗 On-Chain (Stellar testnet)

| | |
| :--- | :--- |
| **Default pool** | [`CBOOAYAVDNAKBAH2OE57O4FDQW4OKCTDJDNJ6MGYM5AP2L46VTGBCON6`][contract-link] |
| **Covenant policy** | [`CA2NXNPDGZ674K467K3K2IXV342BTPJ7YJRLVZPT4LCSJS5GHLVSMEN7`][policy-link] |
| **ed25519 verifier** | [`CAEU4VYRTV4ZDHFGCRYQNPN6URDOYLP4TS4OMRAJX5RD7AMLWBTYW6G4`][verifier-link] |
| **USDC (Circle testnet, SAC)** | `CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA` |
| **Network** | `Test SDF Network ; September 2015` |
| **RPC** | `https://soroban-testnet.stellar.org` |
| **Explorer** | [stellar.expert (testnet)][explorer-link] |
| **Faucets** | [friendbot][friendbot-link] (XLM) · [faucet.circle.com][circle-faucet-link] (USDC) |

Pool and account WASM hashes, retained releases and the full transaction evidence live in [`deployments/testnet.json`][deployments-link] and the [README][kovan-github].

### 🚀 Try It With Your Agent

```text
Claude Code:      /plugin install kovan --marketplace kovan-protocol/kovan
Any agent:        "Read https://kovan-protocol.vercel.app/agent.md and follow it."
Chat assistant:   https://kovan-protocol.vercel.app/setup?fund=100&agent=name:research,draw:40,pay:G…,per:0.5,total:40,hours:48
Developers:       npm install @kovan-protocol/sdk @stellar/stellar-sdk@16.4.1
```

Then ask your agent in plain words: *"Set me up a $100 Kovan pool with 2 agents."* You approve every owner action in Freighter; the agent never holds your key.

### 🤝 Contributing

We welcome PRs, bug reports and design feedback across the repository.

- **Contracts:** pool accounting, covenant rules, mandates and delegation in `contracts/` (Rust, `soroban-sdk`, OpenZeppelin `stellar-accounts`).
- **SDK and agent tooling:** builders, auth payloads and `kovan.mjs` in `sdk/` and `skills/kovan/`.
- **Console, signer, website:** Next.js and Vite apps in `dashboard/`, `signer/` and `web/`.
- **Docs:** [`docs/`][docs-link] (pool, mandates, SDK, yield, agentic safety) and the [pitch deck][pitch-deck-link].

### 🪪 License

[MIT][license-link].

---

> [!WARNING]
>
> **Testnet only.** Hackathon prototype on Stellar testnet. Testnet USDC has no value. Not audited. Not a financial product.

> [!TIP]
>
> **Start here:** install the plugin or hand your agent [`agent.md`][agent-link] → it creates a pool, you fund it in Freighter → it creates agents inside limits you set → watch a payment outside the covenant get refused before it is ever signed.

[website-link]: https://kovan-protocol.vercel.app
[website-shield]: https://img.shields.io/badge/website-kovan--protocol.vercel.app-3D74FF?labelColor=07080B&style=flat-square&logo=safari&logoColor=white

[agent-link]: https://kovan-protocol.vercel.app/agent.md
[agent-shield]: https://img.shields.io/badge/agent%20guide-agent.md-3D74FF?labelColor=07080B&style=flat-square&logo=openai&logoColor=white

[sdk-link]: https://www.npmjs.com/package/@kovan-protocol/sdk
[sdk-shield]: https://img.shields.io/npm/v/%40kovan-protocol%2Fsdk?label=sdk&color=3D74FF&labelColor=07080B&style=flat-square&logo=npm&logoColor=white

[pitch-deck-link]: https://github.com/kovan-protocol/kovan/blob/main/docs/kovan-pitch-deck.pdf
[pitch-deck-shield]: https://img.shields.io/badge/pitch%20deck-view-3D74FF?labelColor=07080B&style=flat-square&logo=adobeacrobatreader&logoColor=white

[github-link]: https://github.com/kovan-protocol
[github-star]: https://img.shields.io/github/stars/kovan-protocol?color=3D74FF&labelColor=07080B&style=flat-square&logo=github

[stellar-link]: https://stellar.org
[stellar-shield]: https://img.shields.io/badge/built%20on-Stellar-3D74FF?labelColor=07080B&style=flat-square&logo=stellar&logoColor=white

[x402-link]: https://x402.org

[kovan-github]: https://github.com/kovan-protocol/kovan
[docs-link]: https://github.com/kovan-protocol/kovan/tree/main/docs
[deployments-link]: https://github.com/kovan-protocol/kovan/blob/main/deployments/testnet.json
[license-link]: https://github.com/kovan-protocol/kovan/blob/main/LICENSE

[contract-link]: https://stellar.expert/explorer/testnet/contract/CBOOAYAVDNAKBAH2OE57O4FDQW4OKCTDJDNJ6MGYM5AP2L46VTGBCON6
[policy-link]: https://stellar.expert/explorer/testnet/contract/CA2NXNPDGZ674K467K3K2IXV342BTPJ7YJRLVZPT4LCSJS5GHLVSMEN7
[verifier-link]: https://stellar.expert/explorer/testnet/contract/CAEU4VYRTV4ZDHFGCRYQNPN6URDOYLP4TS4OMRAJX5RD7AMLWBTYW6G4
[contract-shield]: https://img.shields.io/badge/contracts-Stellar%20testnet-3D74FF?labelColor=07080B&style=flat-square&logo=stellar&logoColor=white

[explorer-link]: https://stellar.expert/explorer/testnet
[friendbot-link]: https://friendbot.stellar.org
[circle-faucet-link]: https://faucet.circle.com

[lang-rust]: https://img.shields.io/badge/rust-3D74FF?labelColor=07080B&style=flat-square&logo=rust&logoColor=white
[lang-typescript]: https://img.shields.io/badge/typescript-3D74FF?labelColor=07080B&style=flat-square&logo=typescript&logoColor=white
