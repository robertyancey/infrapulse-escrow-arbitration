# InfraPulse Escrow + Deterministic Arbitration (v1)

InfraPulse is a **non-custodial, deterministic escrow and verification layer** for autonomous AI systems.

It enables:
- Agent ↔ Agent commerce
- Agent ↔ Service commerce
- Ecosystem ↔ Ecosystem settlement
- Cross-chain and Web2/Web3 interoperability

InfraPulse does **not** hold funds.
It emits cryptographically signed PASS/FAIL verdicts that external settlement rails enforce.

---

# Core Principles

- Deterministic verification only
- Binary outcomes (PASS / FAIL)
- Canonical JSON hashing
- Cryptographically signed verdicts
- Non-custodial architecture
- Rail-agnostic settlement
- Replay-stable evaluation

---

# Architecture Position

InfraPulse sits between:

Data Layer (e.g., blockchain data APIs)
↓  
Wallet / Payment Layer (Stripe, x402, onchain)
↓  
Compute Layer
↓  
Agent Framework Layer
↓  
**InfraPulse = Enforcement + Verification Layer**

InfraPulse does not compete with wallets or chains.
It enforces trust between them.

---

# Escrow Protocol (infrapulse-escrow/1)

## Agreement Core

Agreement Core MUST be canonical JSON:

- UTF-8 encoded
- Keys sorted lexicographically
- Separators (",", ":")
- No float ambiguity for money (use strings)

Example:

```json
{
  "spec": "infrapulse-escrow/1",
  "parties": {
    "buyer": "agent_A",
    "seller": "agent_B"
  },
  "economics": {
    "amount": "100.00",
    "currency": "USD",
    "settlement_rail": "stripe"
  },
  "job": {
    "job_type": "code_generation",
    "schema": { "function": "add(a,b)" },
    "acceptance": {
      "required_execution_hash": "sha256:abc123..."
    }
  },
  "time": {
    "created_at": "ISO8601",
    "deadline_at": "ISO8601",
    "dispute_window_s": 86400
  },
  "policy": {
    "privacy_mode": "hash_only",
    "replay_stable": true
  }
}
