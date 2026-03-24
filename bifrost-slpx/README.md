# bifrost-slpx

Agent skill for **Bifrost SLPx** (vTokens, vETH on-chain flows) using the [`@bifrostio/slpx-cli`](https://www.npmjs.com/package/@bifrostio/slpx-cli) CLI. See **`SKILL.md`** for behavior, pipelines, and progressive disclosure.

## Install

From the **skills collection** root, see **[../README.md](../README.md)** for overview and FAQ.

```bash
npx skills add bifrost-io/skills --skill bifrost-slpx
```

## Prerequisites

Set the environment variable **`BIFROST_SKILL_PRIVATEKEY`** to your private key for `mint`, `redeem`, and `claim`. **Never** paste private keys into chat.

## Contents

| Path | Purpose |
| ---- | ------- |
| `SKILL.md` | Entry point: when to use, pre-flight, mint/redeem/claim pipeline |
| `references/commands.md` | CLI commands and flags |
| `references/tokens-and-chains.md` | vToken matrix, vETH contract, WETH addresses |
| `references/errors.md` | JSON error `code` values |
| `references/pre-tx-checklist.md` | Checks before broadcasting |

The CLI package itself lives in a separate repo and does **not** ship these markdown files.
