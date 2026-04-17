# Verun Protocol — Algorand MVP

> **Slogan:** "Accredited Agents for Regulated Finance."

## Was ist Verun?

Verun ist ein **Agent Trust Score Layer** für regulierte europäische Kapitalmärkte. Verun aggregiert bestehende KYA-Signale (Mastercard, Visa x402, On-Chain History, Validator Consensus) zu einem einheitlichen **Verun Accredited Agent Score (0–1000)**.

Wichtig: Verun baut **kein eigenes KYA** — wir sind KYA-kompatibel und aggregieren bestehende Signale.

## Architektur

```
Algorand  = Agentic Trust Layer  (KYA Score, ASA Trust Token, x402, Bazaar Discovery)
tokenforge = Regulated Gate      (Chain API verbindet beide Layer)
Stellar   = Asset Settlement     (eWpG Mint, ISIN, EURC)
```

Beide Chains sind **komplementär, nicht kompetitiv.**

## 3 Produkt-Modi

| Modus | Beschreibung |
|---|---|
| **Discovery** | Agent scannt validierte Plattformen, findet tokenforge, sendet Empfehlung |
| **Supervised** | Human bekommt strukturierte Empfehlung, approvet mit einem Klick |
| **Autonomous** | Nach Human-Approval: Agent führt vollautomatisch aus, jede Action on-chain |

## Score-Gates (tokenforge Chain API)

| Operation | Mindest-Score |
|---|---|
| read | 300+ |
| transfer / mint | 500+ |
| order | 600+ |

## Fee Split

```
70%  → Verun Protocol Treasury
10%  → Validator (Fee Share)
10%  → Agent Kickback
10%  → Reserve
```

## Founding Validator: tokenforge

- **Was:** White-label Security Token Plattform (TokenSuite), eWpG-konform, BaFin-relevant, MiFID II-ready
- **Warum kritisch:** tokenforge Chain API ist das Gate das Verun absichert. Mint, Transfer, Order laufen über tokenforge.
- **5 APIs:** Core Backend (KYC/AML via IDnow, Custody via Tangany/NYALA), Chain API (ERC-1400, eWpG), Auth, MediaService, Auction Engine
- **Links:** tokenforge.io · docs.tokenforge.io

## Was bereits live ist

- **Vercel:** https://verun-algorand-mvp.vercel.app
- **GitHub:** https://github.com/rafaschul/verun-algorand-mvp
- **Algorand Testnet:** Note-Transactions als on-chain Anchoring aktiv

### API Endpoints (live + new)

```
GET  /api/health         → Service status + network
GET  /api/validators     → List available validators
GET  /api/funding-status → Wallet balance + faucet guidance
GET  /api/config-check   → Env + mnemonic + algod diagnostics
POST /api/score          → Score evaluation (kein Anchor)
POST /api/evaluate       → Score + 2-of-N Consensus + Algorand Anchor
POST /api/mint-sbt       → Mint Verun SBT (ASA, defaultFrozen, clawback)
```

### Beispiel-Call

```bash
curl -X POST https://verun-algorand-mvp.vercel.app/api/evaluate \
  -H "Content-Type: application/json" \
  -d '{"agentId": "agt_demo", "score": 820, "operation": "transfer"}'
```

## Codebase Überblick

```
api/
  health.js      → GET /api/health
  score.js       → POST /api/score  (nur evaluate, kein Anchor)
  evaluate.js    → POST /api/evaluate (evaluate + Algorand Anchor)

src/
  evaluate.js    → Scoring-Logik, Validator-Votes, Consensus
  anchor.js      → Algorand Note-TX (algosdk, schreibt auf Testnet)
  validators.json → 3 Validators (weight 1 each, 2-of-3 required)
  api.js         → Express Server (lokaler Dev)

scripts/
  check.js       → Algorand Wallet + Balance prüfen
  selftx.js      → Test-TX auf Testnet senden

index.html       → Landing Page (dark, Algorand-Design)
```

## Env Vars (Vercel Production)

```
ALGO_MNEMONIC         → 25-Wort Algorand Testnet Wallet (sensitiv!)
ALGO_TESTNET_ADDRESS  → Öffentliche Testnet-Adresse
ALGOD_URL             → https://testnet-api.algonode.cloud
ALGOD_TOKEN           → (optional/leer)
```

## Soulbound Token (SBT) — geplant, noch nicht implementiert

Jeder verifizierte Agent bekommt einen non-transferable ASA:

```python
# ASA Config für Verun SBT
total          = 1            # unique per agent
decimals       = 0
default_frozen = True         # KEY: frozen ab Geburt = non-transferable
manager        = ""           # unveränderlich nach Creation
freeze         = ""           # niemand kann unfreeze
clawback       = VERUN_ADDR   # nur Verun kann revoken (Accreditation widerrufen)
```

**Roadmap:**
- MVP: plain ASA + Flags (oben)
- v2: Smart ASA (ARC-20) mit Score + Tier in AVM Box Storage

Wallet-Binding via Ed25519 Challenge-Response (AVM `ed25519verify` Opcode) — **nicht** ARC-52.

## Was noch offen ist

- [ ] SBT Minting implementieren (ASA + defaultFrozen + clawback)
- [ ] Echter Agent-Testnet-Flow (Discovery → Supervised → Autonomous)
- [ ] tokenforge Chain API Integration als Validator-Onboarding Beispiel
- [ ] Algorand Grant Proposal: Testnet-Beweis (echte TXIDs) + Next Steps
- [ ] Landing Page Demo: echte `/api/evaluate` Calls statt simulierter Daten
- [ ] x402 Payment Gate via GoPlausible Facilitator

## Kontext & Links

- **BCP Partners GmbH** · Berlin · bcpp.io
- **Verun Stellar Demo:** verun-stellar-demo.vercel.app
- **Algorand Testnet Explorer:** testnet.algoexplorer.io
- **© 2026 BCP Partners GmbH · All rights reserved**
