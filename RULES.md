# RULES — Verified Agent Identity

## Operational Rules & Guardrails
1. **Mandatory Identity Verification Prerequisite**: Before executing linking, challenge signing, or x402 payment generation, the agent must verify that a valid identity exists; if missing, an identity must be minted and linked prior to proceeding.
2. **Immediate Halt on Script Failure**: If any cryptographic operation or script exits with a non-zero exit code, the agent must stop immediately; manual workarounds (generating keys via `openssl`, modifying files in `~/.openclaw/billions`) are strictly prohibited.
3. **No Raw Private Key Exfiltration**: Private keys and seed phrases must remain encrypted or held in memory during signing operations; they must never appear in logs, CLI output, or unencrypted persistent storage.
4. **Mandatory Human-in-the-Loop Linking**: Linking an agent DID to a human identity requires explicit human verification via a signed wallet message or verification link; automated synthetic linking is forbidden.
5. **Deterministic Challenge Validation**: Challenge responses must strictly adhere to the requested nonce, expiry window, and audience domain to prevent replay attacks across different servers.
6. **x402 Protocol Compliance**: The x402 payment header generation must conform to `@x402/core` specifications, including precise payment parameter hashing and valid network chain IDs.
7. **Strict KMS Key Safeguards**: When `BILLIONS_NETWORK_MASTER_KMS_KEY` is provided, key operations must route through hardware/KMS modules rather than disk-persisted secrets.
