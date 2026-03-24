# Error codes

All errors return structured JSON with `error`, `code`, and `message` fields.

| Code | Meaning |
|------|---------|
| `INVALID_TOKEN` | Unknown vToken name |
| `UNSUPPORTED_TOKEN` | Token not available for on-chain operations |
| `INVALID_ADDRESS` | Bad Ethereum address format |
| `INVALID_AMOUNT` | Amount must be a positive number |
| `INVALID_CHAIN` | Unknown EVM chain name |
| `INSUFFICIENT_BALANCE` | Not enough vETH |
| `CONTRACT_PAUSED` | vETH contract is paused |
| `NOTHING_TO_CLAIM` | No completed redemptions to claim |
| `NO_WALLET` | Automatic signing not configured per CLI docs, and no `--address` provided where required |
| `RPC_ERROR` | RPC connection failed |
| `API_ERROR` | Bifrost API unavailable or timed out |
| `TX_ERROR` | Transaction execution failed |
| `CLI_ERROR` | Missing argument or invalid option |
