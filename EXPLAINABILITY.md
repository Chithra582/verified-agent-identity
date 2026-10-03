# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Verified Agent Identity** (`verified-agent-identity`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Verified Agent Identity (`verified-agent-identity`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Security / Decentralized Agent Identity (DID), Attestation & x402 Protocol  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

Verified Agent Identity governs decentralized machine identity (DIDs), human attestation linking, and cryptographic authentication handshakes on the Billions Network. Operations transition through five deterministic stages:

### 1. Decision Architecture

The runtime intake, state classification, evaluation, and execution tracking operate across a deterministic, five-stage pipeline:

```
[ Authentication / Verification Request ]
                     │
                     ▼
[ 1. Local Identity Vault & DID Verification ]
                     │
                     ▼
[ 2. Human Owner Attestation Verification ]
                     │
                     ▼
[ 3. Challenge Validation & Replay Defense ]
                     │
                     ▼
[ 4. Cryptographic Signing & Proof Authoring ]
                     │
                     ▼
[ 5. Signature Delivery / x402 Replay Execution ]
```

### 2. Decision Logic & Routing Formulations

When authenticating peer agents or evaluating inbound identity credentials, the verification engine evaluates a multi-factor attestation verification metric $V_{\text{auth}} \in [0, 1]$:

$$V_{\text{auth}} = w_{\text{sig}} \cdot S_{\text{sig}} + w_{\text{att}} \cdot A_{\text{attestation}} + w_{\text{fresh}} \cdot F_{\text{freshness}} + w_{\text{rep}} \cdot R_{\text{registry}}$$

Where:
- $S_{\text{sig}} \in \{0, 1\}$: Binary cryptographic curve verification (secp256k1/babyjubjub) validating signature over the challenge nonce.
- $A_{\text{attestation}} \in \{0, 1\}$: Verification of valid on-chain ERC-8004 human owner attestation in the Billions Network registry.
- $F_{\text{freshness}} \in [0, 1]$: Challenge freshness score $F_{\text{freshness}} = \max\left(0, 1 - \frac{|t_{\text{current}} - t_{\text{challenge}}|}{\Delta t_{\text{max}}}\right)$ where $\Delta t_{\text{max}} = 300\text{s}$.
- $R_{\text{registry}} \in \{0, 1\}$: Revocation status check ($1$ if DID active, $0$ if revoked or expired).
- Parameter weights: $w_{\text{sig}} = 0.40$, $w_{\text{att}} = 0.30$, $w_{\text{fresh}} = 0.15$, $w_{\text{rep}} = 0.15$ ($\sum w_i = 1.0$).

Authentication is accepted if and only if $V_{\text{auth}} = 1.0$, requiring flawless cryptographic validity across all parameters.

### 3. Thresholding & Refusal Decision Criteria

Verified Agent Identity enforces strict operational boundaries and deterministic refusal thresholds:
- **Refusal on ERR_NO_IDENTITY_FOUND**: Action requested but no DID identity exists in local vault halts execution with code `ERR_NO_IDENTITY_FOUND`.
- **Refusal on ERR_UNLINKED_HUMAN_OWNER**: Agent DID is not cryptographically linked to a verified human owner halts execution with code `ERR_UNLINKED_HUMAN_OWNER`.
- **Refusal on ERR_INVALID_CHALLENGE_NONCE**: Incoming challenge timestamp expired or nonce has been seen previously halts execution with code `ERR_INVALID_CHALLENGE_NONCE`.
- **Refusal on ERR_EXCEEDED_X402_BUDGET**: HTTP 402 payment demand exceeds agent `max_fee_allowance` halts execution with code `ERR_EXCEEDED_X402_BUDGET`.
- **Refusal on ERR_CRYPTO_EXECUTION_FAILURE**: Underling cryptographic helper script returns non-zero exit code halts execution with code `ERR_CRYPTO_EXECUTION_FAILURE`.

### 4. Fallback Decision Mechanism

Continuous operational stability is maintained through layered fault recovery:
- **Local Vault Fallback**: If remote Billions Network RPC nodes experience transient latency, the agent verifies cached local identity claims and queued attestation receipts before failing over to secondary RPC endpoints.
- **x402 Retry Fallback**: If an x402 payment signature fails due to gas estimation or transient mempool congestion, the agent reestimates gas parameters and retries once before surfacing the payment wall to the user.
- **Manual HumanLink Fallback**: If automated browserbased pairing fails, the toolkit offers a commandline fallback (`manualLinkHumanToAgent.js`) allowing operators to sign raw attestation strings directly in their hardware or desktop wallets.
- **Model Fallback Cascade**: High-level reasoning and synthesis default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

Human operators retain sovereign authority over the multi-agent execution lifecycle:
- **Mandatory Human Linking**: Every autonomous agent must be formally bonded to an authenticated human principal; unlinked agents cannot issue valid KYA credentials.
- **Spending Ceiling Approvals**: x402 micro-payments exceeding configured spending allowances require explicit human confirmation before dispatch.
- **Sovereign Key Custody**: Private keys are managed within encrypted local vaults (`~/.openclaw/billions`) or secure hardware modules (KMS), with zero cloud key custody.

---

## The Data It Uses

Verified Agent Identity operates under strict principles of data minimization, environment isolation, and privacy protection.

### 1. Ingested Input Data

The framework processes only operational data necessary to perform its functions:
- **Authentication Challenges**: Server nonces, domain audience strings, and expiration timestamps.
- **Human Attestation Signatures**: Cryptographic wallet signatures from human principals asserting ownership over agent DIDs.
- **x402 Payment Specifications**: Resource pricing, recipient contract addresses, token symbols, and blockchain network chain IDs.

### 2. Configuration & Reference Data

- **Billions Network Registry**: On-chain smart contract registries maintaining public DID documents, verification keys, and attestation claims.
- **Local Identity Store**: Encrypted keypair files, identity indices, and attestation receipts stored in `$HOME/.openclaw/billions`.
- **iden3 Protocol Schemas**: Cryptographic claim definitions for identity attestations and zero-knowledge credentials.

### 3. Base Model & Inference Lineage

- **Cryptographic Core**: Pure audited JavaScript/TypeScript cryptography leveraging `@iden3/js-iden3-core`, `@noble/curves`, `viem`, and `ethers`.
- **Payment Engine**: Standardized `@x402/core` and `@x402/evm` integration for autonomous micro-payment authorization.
- **Agent Integration**: Compatible with OpenClaw, clawhub, and shell-capable agent platforms.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against indirect prompt injection, credential leakage, and unauthorized external API dispatch.
- **Local Environment Isolation**: Agent execution workspaces, intermediate scratchpads, and vector stores reside strictly within designated local project directories.
- **Automated Secret Scrubbing**: API keys, database credentials, and personal credentials are automatically redacted prior to embedding or logging.
- **Zero Commercial Monetization**: Prompts, intermediate reasoning trajectories, and task deliverables are never commercialized or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of Verified Agent Identity is essential for effective deployment.

### 1. Mathematical Scoring & Routing Formulation
When authenticating peer agents or evaluating inbound identity credentials, the verification engine evaluates a multi-factor attestation verification metric $V_{\text{auth}} \in [0, 1]$:

$$V_{\text{auth}} = w_{\text{sig}} \cdot S_{\text{sig}} + w_{\text{att}} \cdot A_{\text{attestation}} + w_{\text{fresh}} \cdot F_{\text{freshness}} + w_{\text{rep}} \cdot R_{\text{registry}}$$

Where:
- $S_{\text{sig}} \in \{0, 1\}$: Binary cryptographic curve verification (secp256k1/babyjubjub) validating signature over the challenge nonce.
- $A_{\text{attestation}} \in \{0, 1\}$: Verification of valid on-chain ERC-8004 human owner attestation in the Billions Network registry.
- $F_{\text{freshness}} \in [0, 1]$: Challenge freshness score $F_{\text{freshness}} = \max\left(0, 1 - \frac{|t_{\text{current}} - t_{\text{challenge}}|}{\Delta t_{\text{max}}}\right)$ where $\Delta t_{\text{max}} = 300\text{s}$.
- $R_{\text{registry}} \in \{0, 1\}$: Revocation status check ($1$ if DID active, $0$ if revoked or expired).
- Parameter weights: $w_{\text{sig}} = 0.40$, $w_{\text{att}} = 0.30$, $w_{\text{fresh}} = 0.15$, $w_{\text{rep}} = 0.15$ ($\sum w_i = 1.0$).

Authentication is accepted if and only if $V_{\text{auth}} = 1.0$, requiring flawless cryptographic validity across all parameters.

### 2. Refusal Criteria & Decision Thresholds
The toolkit enforces rigid zero-trust refusal rules:

| Trigger Scenario | Operational Action | Error Code |
| :--- | :--- | :--- |
| Action requested but no DID identity exists in local vault | Halt execution; mandate running `createNewEthereumIdentity.js` | `ERR_NO_IDENTITY_FOUND` |
| Agent DID is not cryptographically linked to a verified human owner | Refuse challenge signing; require human attestation linking | `ERR_UNLINKED_HUMAN_OWNER` |
| Incoming challenge timestamp expired or nonce has been seen previously | Reject challenge; prevent potential replay attack | `ERR_INVALID_CHALLENGE_NONCE` |
| HTTP 402 payment demand exceeds agent `max_fee_allowance` | Abort payment signature; alert operator of paywall cost | `ERR_EXCEEDED_X402_BUDGET` |
| Underling cryptographic helper script returns non-zero exit code | Halt immediately; forbid manual key workarounds | `ERR_CRYPTO_EXECUTION_FAILURE` |

### 3. Multi-Tier Fallback Mechanisms
1. **Local Vault Fallback**: If remote Billions Network RPC nodes experience transient latency, the agent verifies cached local identity claims and queued attestation receipts before failing over to secondary RPC endpoints.
2. **x402 Retry Fallback**: If an x402 payment signature fails due to gas estimation or transient mempool congestion, the agent re-estimates gas parameters and retries once before surfacing the payment wall to the user.
3. **Manual Human-Link Fallback**: If automated browser-based pairing fails, the toolkit offers a command-line fallback (`manualLinkHumanToAgent.js`) allowing operators to sign raw attestation strings directly in their hardware or desktop wallets.

### 4. Human-in-the-Loop Governance
- **Mandatory Human Linking**: Every autonomous agent must be formally bonded to an authenticated human principal; unlinked agents cannot issue valid KYA credentials.
- **Spending Ceiling Approvals**: x402 micro-payments exceeding configured spending allowances require explicit human confirmation before dispatch.
- **Sovereign Key Custody**: Private keys are managed within encrypted local vaults (`~/.openclaw/billions`) or secure hardware modules (KMS), with zero cloud key custody.

---

## The Data It Uses

### 1. Input Data Types
- **Authentication Challenges**: Server nonces, domain audience strings, and expiration timestamps.
- **Human Attestation Signatures**: Cryptographic wallet signatures from human principals asserting ownership over agent DIDs.
- **x402 Payment Specifications**: Resource pricing, recipient contract addresses, token symbols, and blockchain network chain IDs.

### 2. Reference & Configuration Data
- **Billions Network Registry**: On-chain smart contract registries maintaining public DID documents, verification keys, and attestation claims.
- **Local Identity Store**: Encrypted keypair files, identity indices, and attestation receipts stored in `$HOME/.openclaw/billions`.
- **iden3 Protocol Schemas**: Cryptographic claim definitions for identity attestations and zero-knowledge credentials.

### 3. Model Lineage & System Architecture
- **Cryptographic Core**: Pure audited JavaScript/TypeScript cryptography leveraging `@iden3/js-iden3-core`, `@noble/curves`, `viem`, and `ethers`.
- **Payment Engine**: Standardized `@x402/core` and `@x402/evm` integration for autonomous micro-payment authorization.
- **Agent Integration**: Compatible with OpenClaw, clawhub, and shell-capable agent platforms.

### 4. Data Privacy, Retention & Sanitization
- **Strict Private Key Isolation**: Raw private keys never leave local runtime memory; only public DIDs, signed challenge proofs, and payment headers are transmitted.
- **Zero Centralized Tracking**: No centralized server logs authentication handshakes; all verifications execute peer-to-peer or against public blockchain ledgers.
- **Ephemeral Challenge Storage**: Signed challenge nonces are purged immediately following server acknowledgment to prevent replay caching.

---

## Limitations

### 1. Blockchain Network Latency for Initial Minting
- **Limitation**: Minting new DIDs and publishing human attestation claims on public testnets/mainnets incurs transaction confirmation latency.
- **Mitigation**: Cache resolved DID documents locally to enable instantaneous offline authentication for subsequent interactions.

### 2. Hardware / Browser Wallet Dependency
- **Limitation**: Human-to-agent linking requires the human operator to interact with an EVM-compatible wallet (MetaMask, Rabby).
- **Mitigation**: Provide direct CLI signing scripts (`manualLinkHumanToAgent.js`) alongside browser-based QR code pairing.

### 3. Gas Token Requirements for On-Chain Attestations
- **Limitation**: Submitting on-chain attestation claims on Ethereum-compatible networks requires gas tokens (ETH or native network gas).
- **Mitigation**: Support subsidized gas relays and Layer-2 networks to make agent identity creation virtually costless.

### 4. Cross-Chain Identity Resolution
- **Limitation**: DIDs minted on the Billions Network require cross-chain verification bridges when authenticating with non-EVM blockchains.
- **Mitigation**: Standardize on W3C DID document formatting to allow universal DID resolution across heterogeneous chains.

### 5. Nonce Expiration Drift
- **Limitation**: Clock drift between the challenging server and agent host machine can cause valid challenge signatures to fail freshness checks.
- **Mitigation**: Enforce NTP time synchronization and include a 60-second grace tolerance window on challenge timestamps.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & routing formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested input data & query streams | Section 1 | Verified |
| - Configuration & reference schemas | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Mathematical Scoring & Routing Formulation
When authenticating peer agents or evaluating inbound identity credentials, the verification engine evaluates a multi-factor attestation verification metric $V_{\text{auth}} \in [0, 1]$:

$$V_{\text{auth}} = w_{\text{sig}} \cdot S_{\text{sig}} + w_{\text{att}} \cdot A_{\text{attestation}} + w_{\text{fresh}} \cdot F_{\text{freshness}} + w_{\text{rep}} \cdot R_{\text{registry}}$$

Where:
- $S_{\text{sig}} \in \{0, 1\}$: Binary cryptographic curve verification (secp256k1/babyjubjub) validating signature over the challenge nonce.
- $A_{\text{attestation}} \in \{0, 1\}$: Verification of valid on-chain ERC-8004 human owner attestation in the Billions Network registry.
- $F_{\text{freshness}} \in [0, 1]$: Challenge freshness score $F_{\text{freshness}} = \max\left(0, 1 - \frac{|t_{\text{current}} - t_{\text{challenge}}|}{\Delta t_{\text{max}}}\right)$ where $\Delta t_{\text{max}} = 300\text{s}$.
- $R_{\text{registry}} \in \{0, 1\}$: Revocation status check ($1$ if DID active, $0$ if revoked or expired).
- Parameter weights: $w_{\text{sig}} = 0.40$, $w_{\text{att}} = 0.30$, $w_{\text{fresh}} = 0.15$, $w_{\text{rep}} = 0.15$ ($\sum w_i = 1.0$).

Authentication is accepted if and only if $V_{\text{auth}} = 1.0$, requiring flawless cryptographic validity across all parameters.

### 2. Refusal Criteria & Decision Thresholds
The toolkit enforces rigid zero-trust refusal rules:

| Trigger Scenario | Operational Action | Error Code |
| :--- | :--- | :--- |
| Action requested but no DID identity exists in local vault | Halt execution; mandate running `createNewEthereumIdentity.js` | `ERR_NO_IDENTITY_FOUND` |
| Agent DID is not cryptographically linked to a verified human owner | Refuse challenge signing; require human attestation linking | `ERR_UNLINKED_HUMAN_OWNER` |
| Incoming challenge timestamp expired or nonce has been seen previously | Reject challenge; prevent potential replay attack | `ERR_INVALID_CHALLENGE_NONCE` |
| HTTP 402 payment demand exceeds agent `max_fee_allowance` | Abort payment signature; alert operator of paywall cost | `ERR_EXCEEDED_X402_BUDGET` |
| Underling cryptographic helper script returns non-zero exit code | Halt immediately; forbid manual key workarounds | `ERR_CRYPTO_EXECUTION_FAILURE` |

### 3. Multi-Tier Fallback Mechanisms
1. **Local Vault Fallback**: If remote Billions Network RPC nodes experience transient latency, the agent verifies cached local identity claims and queued attestation receipts before failing over to secondary RPC endpoints.
2. **x402 Retry Fallback**: If an x402 payment signature fails due to gas estimation or transient mempool congestion, the agent re-estimates gas parameters and retries once before surfacing the payment wall to the user.
3. **Manual Human-Link Fallback**: If automated browser-based pairing fails, the toolkit offers a command-line fallback (`manualLinkHumanToAgent.js`) allowing operators to sign raw attestation strings directly in their hardware or desktop wallets.

### 4. Human-in-the-Loop Governance
- **Mandatory Human Linking**: Every autonomous agent must be formally bonded to an authenticated human principal; unlinked agents cannot issue valid KYA credentials.
- **Spending Ceiling Approvals**: x402 micro-payments exceeding configured spending allowances require explicit human confirmation before dispatch.
- **Sovereign Key Custody**: Private keys are managed within encrypted local vaults (`~/.openclaw/billions`) or secure hardware modules (KMS), with zero cloud key custody.

---

## The Data It Uses

### 1. Input Data Types
- **Authentication Challenges**: Server nonces, domain audience strings, and expiration timestamps.
- **Human Attestation Signatures**: Cryptographic wallet signatures from human principals asserting ownership over agent DIDs.
- **x402 Payment Specifications**: Resource pricing, recipient contract addresses, token symbols, and blockchain network chain IDs.

### 2. Reference & Configuration Data
- **Billions Network Registry**: On-chain smart contract registries maintaining public DID documents, verification keys, and attestation claims.
- **Local Identity Store**: Encrypted keypair files, identity indices, and attestation receipts stored in `$HOME/.openclaw/billions`.
- **iden3 Protocol Schemas**: Cryptographic claim definitions for identity attestations and zero-knowledge credentials.

### 3. Model Lineage & System Architecture
- **Cryptographic Core**: Pure audited JavaScript/TypeScript cryptography leveraging `@iden3/js-iden3-core`, `@noble/curves`, `viem`, and `ethers`.
- **Payment Engine**: Standardized `@x402/core` and `@x402/evm` integration for autonomous micro-payment authorization.
- **Agent Integration**: Compatible with OpenClaw, clawhub, and shell-capable agent platforms.

### 4. Data Privacy, Retention & Sanitization
- **Strict Private Key Isolation**: Raw private keys never leave local runtime memory; only public DIDs, signed challenge proofs, and payment headers are transmitted.
- **Zero Centralized Tracking**: No centralized server logs authentication handshakes; all verifications execute peer-to-peer or against public blockchain ledgers.
- **Ephemeral Challenge Storage**: Signed challenge nonces are purged immediately following server acknowledgment to prevent replay caching.

---

## Limitations

### 1. Blockchain Network Latency for Initial Minting | Section 1 | Verified |
| - Hardware / Browser Wallet Dependency | Section 2 | Verified |
| - Gas Token Requirements for On-Chain Attestations | Section 3 | Verified |
| - Cross-Chain Identity Resolution | Section 4 | Verified |
| - Nonce Expiration Drift | Section 5 | Verified |
