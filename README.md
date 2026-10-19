<div align="center">

# Meridian Vault

**Institutional platform for issuing, holding, transferring and servicing tokenized real-world assets**

Compliance, custody and settlement enforced by smart contracts

![Status](https://img.shields.io/badge/status-Parts_1_to_5_of_6_complete-blue)
![Live](https://img.shields.io/badge/live-Sepolia_%7C_Base_%7C_Robinhood_Chain-2ea44f)
![Invariants](https://img.shields.io/badge/invariants-10k_runs_passing-brightgreen)
![Coverage](https://img.shields.io/badge/coverage-98.76%25-brightgreen)
![ERC-3643](https://img.shields.io/badge/ERC--3643-T--REX-0B2545)
![Foundry](https://img.shields.io/badge/Foundry-tested-orange)

### [▶ Live demo: meridian-vault-slice1.vercel.app](https://meridian-vault-slice1.vercel.app)

Live reads from the deployed fund on Robinhood Chain, Base Sepolia and Ethereum Sepolia

</div>

> Built in public, one part at a time. Source code is private; this repository is the public overview. Qualified hiring managers and clients can [request a walkthrough](#request-access).

![Parts 1 to 5 results: contracts on 5 networks, custody with no key in the app, real-time compliance screening, an AI analyst that cannot act, chaos-tested reliability](assets/parts1to5_results.png)

## The problem

Banks and asset managers are moving funds, bonds and credit on-chain (BlackRock BUIDL, Franklin Templeton BENJI, JPMorgan Kinexys). Each needs the same hard parts done right: investor eligibility enforced at the token level, institutional custody with policy-based signing, regulatory-grade monitoring, and operations that run on both private and public networks.

## The scenario

A fund manager tokenizes a **$50M private credit fund**. KYC-verified investors subscribe with a stablecoin, hold shares, transfer only to other eligible investors, and cash out through a redemption queue priced at the fund's net asset value.

## Build status

| Part | Scope | Status |
|---|---|---|
| 1 | Contracts: ERC-3643 fund shares, NAV oracle, ERC-4626/7540 vault, compliance modules, multi-chain deploys | Complete |
| 2 | Custody: Rust signing service, MPC / HSM / cloud KMS, wallet tiers | Complete: AWS KMS, Turnkey and DFNS live on Base Sepolia; 500 concurrent signings from one wallet, 0 nonce gaps |
| 3 | Compliance pipeline: Go services, Kafka, gRPC, case management | Complete: every transfer screened in real time; freeze only through 2 named approvals |
| 4 | AI compliance analyst with human approval, evals and guardrails | Complete: held-out test set recall 1.00, precision 0.966; prompt-injection set 30 of 30 with a hard rule; the AI cannot act |
| 5 | Kubernetes, observability, load and chaos testing | Complete: every latency and availability target met on a clean AWS machine; 11 chaos tests pass, 7 real bugs found and fixed |
| **6** | **Investor and admin portals, Solana Token-2022, demo video** | **In progress** |

## Part 1 results

The same contracts and the same deployment script ran the full investor journey (KYC, subscribe, transfer, blocked transfer to an unverified wallet, redemption request, fulfilment, claim) on four settlement models.

| Settlement model | Network | Full flow | Cost of the flow |
|---|---|---|---|
| Private consortium | Hyperledger Besu, 4 QBFT validators | 57 / 57 transactions | No public fees |
| Institutional permissioned | Avalanche L1 (local), chain-level allowlists | 57 / 57 transactions | No public fees |
| Company public L2 (Arbitrum) | Robinhood Chain Testnet | 57 / 57 transactions (1 resent with more gas) | 0.000492 test ETH |
| Company public L2 (OP Stack) | Base Sepolia | 57 / 57 transactions | 0.000283 test ETH |
| Public reference (L1) | Ethereum Sepolia | 57 / 57 transactions | 0.049161 test ETH |

![Same fund on private, permissioned and public chains: 57 of 57 transactions on every network](assets/part1_results.png)

Costs come from transaction receipts on a single run; testnet gas prices vary.

On the permissioned Avalanche L1, the chain itself rejected a non-allowlisted wallet and blocked contract deployment by a non-approved account, before any contract-level KYC check ran.

| Quality evidence | Result |
|---|---|
| Automated tests | 140 passing |
| Invariants | 6 safety rules held across 10,000 randomized runs, 500,000 actions each |
| Line coverage | 98.76% (CI gate: 95%) |
| Static analysis | Slither and Aderyn, zero high-severity findings after review |
| Threat model | 18 STRIDE threats mapped to controls and test evidence |

### Live contracts (testnets)

| Contract | Ethereum Sepolia | Base Sepolia | Robinhood Chain Testnet |
|---|---|---|---|
| Fund share token (ERC-3643) | [0x1E8A…3d41](https://sepolia.etherscan.io/address/0x1E8A3D7A08F1C450BE22c84082c79CffE9bF3d41) | [0x277E…E9d9](https://sepolia.basescan.org/address/0x277E4e94036926E990216d6FE969A51A4E2dE9d9) | [0x277E…E9d9](https://explorer.testnet.chain.robinhood.com/address/0x277E4e94036926E990216d6FE969A51A4E2dE9d9) |
| Fund vault (ERC-4626 / 7540) | [0x4e4C…D6Fb](https://sepolia.etherscan.io/address/0x4e4C292f9387e3A5d6b961FF8D02EC78Ee45D6Fb) | [0xf339…9dC7](https://sepolia.basescan.org/address/0xf3396bc840b147Cb0e0e6fE98C94909a03B89dC7) | [0xf339…9dC7](https://explorer.testnet.chain.robinhood.com/address/0xf3396bc840b147Cb0e0e6fE98C94909a03B89dC7) |

Verified source on Etherscan and Basescan for the stablecoin, NAV oracle and vault. Base and Robinhood Chain addresses match because the same deployer created the contracts in the same order on both chains.

## Parts 2 to 5 results

| Part | What was built | Measured result |
|---|---|---|
| 2. Custody | Rust signing service; keys stay in AWS KMS, an HSM, Turnkey or DFNS; policy engine with M-of-N approval (the requester cannot approve their own request); cold, warm and hot wallet tiers with a 24-hour delay on admin changes | 500 concurrent signings from one wallet: 0 nonce gaps, 0 duplicates. Real AWS KMS signing p99 184 ms; Turnkey and DFNS signed live on Base Sepolia |
| 3. Compliance | Go indexer with a transactional outbox, Kafka, Go compliance service with versioned screening rules, case management, gRPC API; freezes go through two named approvals and the signer | Every transfer screened before release: p99 30 to 104 ms at 500 events/s over 3 runs (target 500 ms). One of three Kafka brokers killed under load: 46,340 of 46,340 events decided |
| 4. AI analyst | Triage agent and analyst copilot over read-only tools and a read-only MCP server; guardrails on input, output, citations and cost; the AI can recommend, never act | Held-out set of 100 labelled cases: recall 1.00, precision 0.966. Prompt-injection set: 30 of 30 held (one case fooled the model; a hard rule blocks it, verified live). Copilot: 20 of 20 answers with valid citations |
| 5. Reliability | OpenTelemetry traces and metrics, Grafana dashboards, SLO alerts, backpressure, chaos tests for every failure mode in the design | Every latency and availability target met on a clean AWS machine; 126,000 requests over 30 minutes, 0 failed. 11 of 11 chaos tests pass; they found 7 real bugs, each fixed with a test that fails on the old code. Database destroyed and restored: back in 1.4 s, 0 events lost |

Testnets and synthetic data only. The AI results come from Gemini 3.5 Flash Lite, a small model, on synthetic cases.

## Architecture

Parts 1 to 5 are built and tested; the portals and the Solana share class are Part 6, in progress. Use the diagram's full-screen and zoom controls to explore it.

```mermaid
%%{init: {"theme": "default", "themeVariables": {"fontSize": "18px"}, "flowchart": {"nodeSpacing": 40, "rankSpacing": 70}}}%%
flowchart TB
  subgraph Frontend["Next.js + Wagmi + Viem (Part 6, in progress)"]
    I[Investor Portal]
    A[Issuer / Compliance Admin]
  end
  subgraph Services["Go services, gRPC (Parts 2, 3, 6)"]
    API["Portal API, Part 6"]
    RM[(Read model)]
    POL[Policy Engine]
    CMP[Compliance + Case Mgmt]
    IDX[Indexer]
  end
  subgraph Custody["Rust Signer (Part 2, live)"]
    SG[Signing Service]
    KMS[AWS KMS]
    HSM["HSM via PKCS 11"]
    TK[Turnkey]
    DF[DFNS]
  end
  subgraph AI["AI Compliance Analyst (Part 4, live)"]
    AG[LLM Agents + Evals + Guardrails]
  end
  K[(Kafka)]
  subgraph Chains["Settlement networks (Part 1, live)"]
    BESU[Hyperledger Besu]
    AVA[Avalanche L1]
    RH[Robinhood Chain]
    BASE[Base]
    SEP[Ethereum Sepolia]
  end
  I --> API
  A --> API
  API --> POL --> SG
  SG --> KMS
  SG --> HSM
  SG --> TK
  SG --> DF
  SG --> Chains
  Chains --> IDX --> K
  K --> CMP
  K --> AG
  K --> RM
  RM --> API
  AG -->|recommends| CMP
  CMP --> A
```

<sub>Note: an abbreviated view. It omits supporting components (nonce manager, outbox, dead-letter topics, observability stack) that are specified in the system design RFC. Fireblocks was deferred in Part 2; Turnkey and DFNS cover MPC. Detailed component and sequence diagrams are available in a walkthrough.</sub>

### Design notes

The architecture is specified in a system design RFC with capacity targets, ten design decisions and a failure-mode table. Highlights:

| Concern | Design | Status |
|---|---|---|
| Transaction ordering under load | Per-wallet single-writer nonce manager and a pool of hot wallets, with replacement-by-fee for stuck transactions | Built: 0 nonce gaps in 500 concurrent signings; stuck-transaction replacement tested |
| Retries and duplicates | Idempotency keys on every write API; transactional outbox for every emitted event | Built: outbox in Part 3; idempotency keys on the portal API in Part 6 |
| Chain reorganisations | Finality-aware indexing with per-chain confirmation depth; idempotent, replay-safe consumers | Built and tested: reorg retracts and replays with 0 duplicates |
| Read scale | Read model fed by the indexer; portals never read the chain directly | In progress (Part 6) |
| Ordering and scale of events | Kafka partitioned by fund; dead-letter topics; lag alerts | Built: lag alert fired during a frozen-consumer test |
| Overload | Bounded queues and backpressure in front of policy and signer | Built: overload answered with 429 and Retry-After, nothing dropped |
| Safety | Policy decision required before any signature; fail closed; humans approve AI-recommended actions | Built and tested: fails closed when the policy engine is down; the AI cannot act |

Scale targets are stated per component (for example peak write and read rates, policy and signing latency, event lag, availability, recovery objectives) and will be published as measured results in Part 5. No capacity figure is claimed before it is measured.

## ERC standards

| Standard | Purpose in the platform | Status |
|---|---|---|
| **ERC-3643** (T-REX) | Permissioned fund share token: on-chain identity registry, modular compliance, freeze, forced transfer, wallet recovery | Live |
| **ERC-734 / ERC-735** (ONCHAINID) | Identity keys and KYC claims checked on every transfer | Live |
| **ERC-4626** | Fund share vault with NAV-per-share accounting | Live |
| **ERC-7540** | Asynchronous redemption queue (request, fulfil, claim) | Live |
| **ERC-20** + **EIP-2612** | Settlement stablecoin (test) for subscriptions and redemptions | Live |
| **ERC-1400** | Partitioned share classes, lockup tranche, document anchoring | Live (thin) |
| **ERC-721** | Asset certificate for the underlying loan pool | Live (thin) |
| **ERC-1155** | Fractional units and distribution vouchers | Live (thin) |
| **ERC-1967 / ERC-1822** (UUPS) | Upgradeable vault and oracle; admin-controlled today, timelock and multisig planned | Live (admin) |
| **ERC-165** | Interface detection | Live |

## Capabilities

| Capability | What it demonstrates | Status |
|---|---|---|
| On-chain compliance | Transfers to unverified or restricted-country wallets are rejected by the contract itself | Live |
| Compliance modules | Country restriction, investor cap, lockup | Live |
| NAV-priced subscriptions | Price feed with staleness and deviation guards | Live |
| Asynchronous redemptions | Request, fulfil at the next NAV, claim; shares escrowed by freezing | Live |
| Private, permissioned and public chains | Same contracts on Besu, an allowlisted Avalanche L1, Robinhood Chain, Base and Ethereum | Live |
| Chain-level permissioning | Avalanche L1 transaction and deployer allowlists | Live (local) |
| Rust signing service and MPC custody | Policy-based signing, M-of-N approval, wallet tiers | Live (testnets) |
| Event-driven compliance and AML monitoring | Kafka-fed screening, rules, case management, freeze | Live (testnets) |
| AI compliance analyst | Alert triage and analyst copilot with human approval of every action | Live (synthetic data) |
| Atomic DvP settlement and corporate actions | Single-transaction share and stablecoin swap; automated distributions | Planned |
| Upgrade governance | Timelock and multisig authorization | Planned |

## Engineering approach

| Principle | Application |
|---|---|
| Two deep areas | Custody and signing security; compliance and AI pipeline |
| Measured | Results published only from real runs (receipts, test runs, CI) |
| Reasoned | 43 architecture decision records and 6 design RFCs, each with rejected alternatives and failure modes |
| Failure-ready | Threat models for contracts and custody; 11 chaos tests, one per failure mode in the design; incident runbook still to write |

## Security and governance

| Practice | Detail |
|---|---|
| Testing | Foundry unit, fuzz and invariant tests (98.76% line coverage); Go and Rust unit and integration tests; load and chaos tests on AWS |
| Static analysis | Slither and Aderyn gates in CI |
| Threat modeling | STRIDE; contracts and custody complete |
| Control mapping | NYDFS Part 500, NIST CSF 2.0, SOC 2, CCSS, OWASP SCSVS: 24 controls, 22 implemented or partial with evidence |
| Architecture decisions | ADRs, for example ERC-3643 as primary standard, async redemptions, settlement network lineup, durable spent-approval store, AI may never recommend closing a sanctions alert |

Control mapping shows design alignment only. It is not a certification or regulatory approval. Testnets only; no real funds.

## Tech stack

| Layer | Technologies | Status |
|---|---|---|
| Smart contracts | Solidity, Foundry, OpenZeppelin, T-REX, ONCHAINID | Live |
| Chains | Hyperledger Besu, Avalanche L1 (Subnet-EVM), Robinhood Chain, Base, Ethereum Sepolia | Live |
| CI and security | GitHub Actions, Slither, Aderyn, coverage and gas gates | Live |
| Backend | Go, Rust, gRPC, Kafka, PostgreSQL | Live |
| AI | LLM agents, read-only MCP server, evaluation harness (Python), guardrails | Live |
| Custody | AWS KMS, Turnkey, DFNS, PKCS#11 HSM (Fireblocks deferred) | Live (testnets) |
| Frontend | Next.js, React, TypeScript, Wagmi, Viem | Partial: public demo site live; portals in Part 6 |
| Infrastructure and observability | Docker, Kubernetes (k3s), Terraform, AWS, OpenTelemetry, Prometheus, Grafana, Tempo | Live |

## Related work

| Project | Description |
|---|---|
| [nexus-protocol](https://github.com/colemanwhaylon/nexus-protocol) | DeFi, NFT and enterprise tokenization platform with contracts deployed on Sepolia |

## Request access

A guided architecture walkthrough, test results and the source code are available to hiring managers and prospective clients.

Contact: [Whaylon Coleman on GitHub](https://github.com/colemanwhaylon)

---

<sub>Copyright (c) 2026 Whaylon Coleman. All rights reserved. Meridian Vault is a portfolio reference implementation.</sub>
