# Bifrost SLPx Skills

Agent skills for [Bifrost](https://bifrost.io) vETH liquid staking — query on-chain data and execute stake/redeem/claim operations via the SLPx ERC-4626 vault.

## Skills

| Skill | Description |
|-------|-------------|
| [bifrost-slpx-info](./bifrost-slpx-info/) | Query vETH exchange rates, APY, TVL, balances, redemption status, and protocol stats |
| [bifrost-slpx-stake](./bifrost-slpx-stake/) | Mint vETH (stake ETH), redeem vETH (unstake), and claim redeemed ETH |

## Supported Chains

Ethereum · Base · Optimism · Arbitrum

All chains share the same vETH contract: `0xc3997ff81f2831929499c4eE4Ee4e0F08F42D4D8`

## Install

### Option 1: /learn command (Recommended)

If you have the [/learn command](https://agentskill.sh/install) installed, just type in your AI chat:

```
/learn bifrost
```

Or install a specific skill directly:

```
/learn @bifrost-io/skills/bifrost-slpx-info
/learn @bifrost-io/skills/bifrost-slpx-stake
```

### Option 2: Git clone (per platform)

Clone this repo into your platform's skills directory:

| Platform | Command |
|----------|---------|
| Cursor | `git clone https://github.com/bifrost-io/skills.git ~/.cursor/skills/bifrost-skills` |
| Claude Code | `git clone https://github.com/bifrost-io/skills.git ~/.claude/skills/bifrost-skills` |
| Windsurf | `git clone https://github.com/bifrost-io/skills.git ~/.windsurf/skills/bifrost-skills` |
| Cline | `git clone https://github.com/bifrost-io/skills.git ~/.cline/skills/bifrost-skills` |
| GitHub Copilot | `git clone https://github.com/bifrost-io/skills.git .github/copilot/skills/bifrost-skills` |
| OpenClaw | `git clone https://github.com/bifrost-io/skills.git ./skills/bifrost-skills` |

For project-specific install, clone into the project directory instead of `~`.

### Option 3: Manual copy

Download the `SKILL.md` files from this repo and place them in your platform's skills directory.

## Prerequisites

- [Foundry](https://book.getfoundry.sh/getting-started/installation) (`cast` CLI) — for on-chain queries and transaction signing
- An Ethereum RPC endpoint (public defaults are included, or set `BIFROST_RPC_URL`)

## Links

- [Bifrost](https://bifrost.io)
- [Bifrost vETH](https://www.bifrost.io/vtoken/veth)
- [Bifrost dApp](https://app.bifrost.io/vstaking/vETH)