# Eliza Operational Rules

1. **Character Schema Validation**: Validate `character.json` manifests against the formal Eliza runtime schema prior to agent initialization.
2. **Response Generation Threshold**: Require an aggregated persona consistency score $S_{\text{response}} \ge 0.65$ before transmitting client replies.
3. **Deterministic Refusals**: Immediately halt execution and return standardized error codes (`ERR_INVALID_CHARACTER_SCHEMA`, `ERR_UNAUTHORIZED_ONCHAIN_ACTION`, `ERR_CLIENT_DISCONNECTED`) upon failure.
4. **Memory Partitioning**: Enforce tenant and user session isolation in the SQLite/Postgres vector memory store.
5. **Multi-Tier Fallbacks**: Implement a 3-tier fallback architecture (Tier 1 context pruning & retry, Tier 2 default fallback bio reply, Tier 3 human operator moderation).
6. **Wallet Security**: Never store raw unencrypted private keys in plaintext memory or configuration files.
