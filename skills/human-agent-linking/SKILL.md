---
name: "human-agent-linking"
description: "Binds agent DIDs to verified human owner identities via ERC-8004 and Billions Attestation Registries."
license: MIT
---

# Human-Agent Linking

## Overview
This skill creates immutable, cryptographically verifiable bindings between an autonomous agent DID and its human owner, establishing "Know Your Agent" (KYA) provenance.

## Key Capabilities
- **Pairing Handshake**: Generates challenge QR codes and web links for human wallet signature.
- **Attestation Registry Binding**: Submits signed human-to-agent pairing proofs to the Billions Network.
- **ERC-8004 Compliance**: Implements the ERC-8004 standard for agent identity ownership verification.
- **Ownership Auditing**: Allows third-party observers to verify the legal or organizational human owner of an agent.

## Operational Workflow
1. **Status Inspection**: Verify that an active agent DID exists in the local vault.
2. **Pairing Initiation**: Launch `human_identity_linker` to generate a human linking invitation.
3. **Wallet Confirmation**: Prompt the human owner to sign the linking payload with their Ethereum wallet.
4. **Attestation Record**: Commit signed attestation to the registry and update local identity metadata.
