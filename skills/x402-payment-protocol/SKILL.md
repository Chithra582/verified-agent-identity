---
name: "x402-payment-protocol"
description: "Resolves HTTP 402 Payment Required challenges, constructing and signing PAYMENT-SIGNATURE headers."
license: MIT
---

# x402 Payment Protocol

## Overview
This skill implements the x402 payment protocol specification, allowing AI agents to autonomously handle HTTP 402 Payment Required responses and purchase micro-services or API access via signed payment proofs.

## Key Capabilities
- **402 Challenge Parsing**: Extracts payment requirements (recipient address, token amount, currency, chain ID) from HTTP 402 response headers.
- **Payment Signature Authoring**: Formulates and signs `PAYMENT-SIGNATURE` headers via `x402_payment_signer`.
- **EVM Integration**: Interacts with Ethereum and Layer-2 payment contracts via `@x402/evm` and `viem`.
- **Micro-Payment Proofs**: Unlocks paywalled API resources autonomously while enforcing spending limits.

## Operational Workflow
1. **Response Detection**: Intercept HTTP `402 Payment Required` response from external API.
2. **Requirement Parsing**: Extract recipient contract address, price, and token parameters.
3. **Budget Verification**: Check agent spending allowance before proceeding with payment.
4. **Signature Construction & Retry**: Build signed `PAYMENT-SIGNATURE` header and replay HTTP request to obtain resource.
