# ATI-001  
**Auditable Trust Infrastructure**  
**Core Specification**

**Status:** Draft  
**Category:** Governance Infrastructure Standard  
**Version:** 0.4  
**Revision Date:** 2026-03-06  

---

## 1. Abstract

This document defines the Auditable Trust Infrastructure (ATI), a reference architecture and deterministic governance protocol for high-impact AI and autonomous systems.

ATI provides deterministic risk evaluation, cryptographically verifiable decisions, immutable governance records, and externally verifiable audit trails. The goal is to transform governance from logging into admissible evidence.

---

## 1.1 Normative Language

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, and “MAY” in this document are to be interpreted as described in RFC 2119.

---

## 2. Scope

ATI defines requirements for systems that must support:  
- auditable automated decision making  
- deterministic governance enforcement  
- cryptographic traceability of decisions  
- external verification of system behavior  

Typical deployment contexts include high-stakes domains:  
- AI systems affecting civil rights or public safety  
- financial decision systems  
- security and defense platforms  
- critical infrastructure automation  

ATI governs the validation and certification of decisions. It does **not** define the internal structure or training of AI models.

---

## 3. Terminology

**Trust Vector**  
$$
T = (c, a, \theta, u, r, \hat{a}), \quad T_i \in (\varepsilon, 1], \quad \varepsilon = 10^{-6}
$$

**Trust Oracle**  
A deterministic function  
$$
T_i = g_i(\text{input_state})
$$  
where \(g_i\) is pure (same input → same output). Non-deterministic ML inference is prohibited inside oracles.

**Decision Envelope**  
The complete, hashable context required for deterministic replay.

**Proof Object**  
Cryptographically signed artifact (see Section 9).

**Constants**  
$$
\varepsilon = 10^{-6},\quad
\alpha \in [0.1, 10],\quad
\beta \in [0.1, 10],\quad
\mu \in \mathbb{R}
$$

---

## 4. Reference Architecture

ATI systems **MUST** implement the following layered pipeline:
External Input ↓ Secure Ingestion ↓ Cognitive Orchestrator (Umbrella) ↓ Trust Oracle Mesh (Guardian) ↓ Trust Vector Generation ↓ Deterministic Risk Engine (Spezzatura) ↓ Policy Engine ↓ Intervention Engine (Burn Engine) ↓ Cryptographic Ledger (Veritas)
---

## 5. Deterministic Risk Engine

The core of ATI is the Deterministic Multiplicative Trust Risk model (DMTR).

### 5.1 Multiplicative Trust  
$$
p = \prod_{i=1}^{6} T_i
$$

### 5.2 Numerical Stability  
Implementations **MUST** use:  
$$
\log_p = \sum_{i=1}^{6} \ln(T_i),\quad E = -\log_p
$$

### 5.3 Human-Interpretable Score  
$$
S = E / \ln(10)
$$

### 5.4 Reactive Shock Model  
$$
X(t) = \sum_j \text{shock}_j \cdot e^{-\lambda_j (t - t_j)},\quad \lambda_j = \frac{\ln 2}{\tau_j}
$$

### 5.5 Governance Surface  
$$
Z = \alpha S + \beta X - \mu
$$  
$$
\text{Risk} = \frac{1}{1 + e^{-Z}}
$$

### 5.6 Monotonicity Theorem  
Z is strictly increasing in each T_i because  
$$
\frac{\partial Z}{\partial T_i} = \alpha \cdot \frac{\partial S}{\partial T_i} = \alpha \cdot \frac{1}{T_i \cdot \ln(10)} > 0
$$  
and  
$$
\frac{d\,\text{Risk}}{dZ} = \text{Risk} \cdot (1 - \text{Risk}) > 0.
$$  
Therefore  
$$
\frac{\partial \text{Risk}}{\partial T_i} < 0 \quad \forall T_i,\ \forall \alpha > 0.
$$

### 5.7 Weakest-Link Guarantee  
$$
p \leq \min_i(T_i)
$$  
Equality holds iff all other components = 1. This guarantees no dimension can mask catastrophic failure in another.

---

## 6. Deterministic Envelope & Replay Equivalence

The governance decision is defined by the function  
$$
\text{decision} = f(\text{input_hash},\ \text{trust_vector},\ \text{policy_hash},\ \text{model_digest},\ \text{operator_surface},\ \text{timestamp})
$$

**Replay Equivalence**  
For any independent auditor,  
$$
\text{replay}(\text{DecisionEnvelope}) = \text{original_decision}
$$  
whenever all inputs are identical (including canonical serialization).

**Timestamp rule**: **MUST** be included in the envelope but **MUST NOT** influence risk computation (Z).

---

## 7. Cryptographic Requirements

ATI implementations **MUST** use:  
- SHA-256 (or stronger) for all hashing (input_hash, policy_hash, model_digest, decision_hash, previous_hash, Merkle trees).  
- Ed25519 or ECDSA P-256 for signatures on Proof Objects.  
- Cryptographic primitives compliant with NIST SP 800-131A (Transitioning the Use of Cryptographic Algorithms).

---

## 8. Time Semantics

Timestamps **MUST** be generated from a trusted time source synchronized via NTP (RFC 5905).  
Clock drift exceeding ±5 seconds between generation and verification **MUST** invalidate replay verification.  
Timestamps **MUST** be included in the DecisionEnvelope in RFC 3339 format but **MUST NOT** influence the risk computation Z.

---

## 9. Proof Object

**Canonical Serialization**  
DecisionProof **MUST** be serialized using RFC 8785 JSON Canonicalization Scheme before hashing or signing. The canonical form is the single source of truth for replay.

**Example structure (JSON Schema normative):**

```json
{
  "input_hash": "sha256:...",
  "trust_vector": [0.98, 0.95, ...],
  "risk_score": 0.87,
  "decision": "APPROVE",
  "policy_hash": "sha256:...",
  "model_digest": "sha256:...",
  "timestamp": "2026-03-06T16:44:00Z",
  "previous_hash": "sha256:...",
  "signature": "..."
}
10. Ledger Structure
Ledger Verification Protocol
An auditor verifies inclusion by:
1.  Recompute decision_hash
2.  Verify Merkle inclusion proof against merkle_root
3.  Verify signature on Proof Object
4.  Verify chain integrity (previous_hash links)
Verification cost: O(log n) with Merkle trees.

11. External Verification
An independent auditor MUST be able to verify a decision by performing:
1.  Recompute input_hash
2.  Recompute trust_vector (via deterministic oracles)
3.  Verify policy_hash
4.  Recompute decision (via Spezzatura)
5.  Verify ledger inclusion
6.  Verify signature
If all checks succeed, the decision is considered admissible.

12. Compliance Levels
Level 1 — Transparent
Systems provide traceable decision logs.
Level 2 — Deterministic
Systems support deterministic replay of governance decisions.
Level 3 — Cryptographically Admissible
Systems implement:
•  decision proofs
•  immutable ledgers
•  external verification

13. Security and Correctness Guarantees
ATI provides the following formal guarantees (G1–G5):
G1 – Weakest-Link Collapse
Any T_k → ε implies p → ε and Risk → 1.
G2 – Risk Monotonicity
Increasing any trust component never increases Risk.
G3 – Deterministic Replay
Identical DecisionEnvelope always produces identical decision (Replay Equivalence).
G4 – Cryptographic Traceability
Every decision produces a tamper-evident Proof Object in a Merkle-chained ledger.
G5 – External Verification
Any party without privileged access can verify admissibility in O(log n) time.
These guarantees hold under the adversarial threat model and with deterministic Trust Oracles.
