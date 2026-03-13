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

```bash
# All skills
npx skills add bifrost-finance/bifrost-slpx-skills

# Individual skill
npx skills add bifrost-finance/bifrost-slpx-skills --skill bifrost-slpx-info
npx skills add bifrost-finance/bifrost-slpx-skills --skill bifrost-slpx-stake
```

## Links

- [Bifrost](https://bifrost.io)
- [Bifrost vETH](https://www.bifrost.io/vtoken/veth)
- [Bifrost dApp](https://app.bifrost.io/vstaking/vETH)