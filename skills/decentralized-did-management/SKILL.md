---
name: "decentralized-did-management"
description: "Mints, resolves, and manages Ethereum and iden3 decentralized identities (DIDs) on the Billions Network."
license: MIT
---

# Decentralized DID Management

## Overview
This skill creates and maintains sovereign, on-chain verifiable decentralized identifiers (DIDs) for AI agents, establishing cryptographic keypairs and public DID documents.

## Key Capabilities
- **DID Minting**: Generates new Ethereum-compatible iden3 DIDs via `did_identity_creator`.
- **Vault Inspection**: Reads existing identity files stored in `$HOME/.openclaw/billions`.
- **DID Document Resolution**: Resolves public keys, verification methods, and service endpoints from the Billions registry.
- **Cryptographic Key Management**: Operates secp256k1 and babyjubjub keys for identity signing.

## Operational Workflow
1. **Identity Query**: Check local identity registry to inspect active agent DIDs.
2. **Key Generation**: If no identity exists, generate a new cryptographic keypair.
3. **DID Registration**: Mint new DID on the Billions Network and write configuration to the local identity vault.
4. **Document Export**: Output public DID identifier and document descriptor for peer discovery.
