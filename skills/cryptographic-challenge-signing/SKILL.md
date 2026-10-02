---
name: "cryptographic-challenge-signing"
description: "Executes challenge-response authentication, signing nonces and verifying peer agent signatures."
license: MIT
---

# Cryptographic Challenge Signing

## Overview
This skill enables agents to prove ownership of their decentralized identity through non-interactive and challenge-response digital signature protocols.

## Key Capabilities
- **Nonce Signing**: Signs server-provided authentication challenges using the agent's private key via `challenge_signer`.
- **Replay Protection**: Verifies timestamps and nonces to ensure challenges cannot be reused.
- **Peer Signature Verification**: Validates digital signatures submitted by external agents via `identity_verifier`.
- **Zero-Knowledge Integration**: Formats authentication claims compatible with iden3 zero-knowledge proof verifiers.

## Operational Workflow
1. **Challenge Ingestion**: Receive authentication payload containing server nonce and audience domain.
2. **Signature Generation**: Sign challenge using local agent DID private key without exposing raw key bytes.
3. **Proof Delivery**: Return serialized cryptographic proof to challenging server.
4. **Mutual Verification**: If required, verify challenging entity's signature against its published DID document.
