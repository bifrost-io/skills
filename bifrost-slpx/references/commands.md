# Command reference

Always pass `--json` when invoking from an agent unless you intentionally need human-readable text.

Prefix: `npx -y @bifrostio/slpx-cli`

## Query commands (all vTokens)

### `rate [amount]`

Exchange rate. Default: 1 unit of base asset.

```bash
npx -y @bifrostio/slpx-cli rate --json
npx -y @bifrostio/slpx-cli rate 10 --json
npx -y @bifrostio/slpx-cli rate --token vDOT --json
npx -y @bifrostio/slpx-cli rate 100 --token vKSM --json
```

Output fields: `input`, `output`, `rate`, `reverseRate`, `token`, `source`

### `apy`

Staking APY (base + reward). Add `--lp` for LP pool yields from DeFiLlama.

```bash
npx -y @bifrostio/slpx-cli apy --json
npx -y @bifrostio/slpx-cli apy --token vDOT --json
npx -y @bifrostio/slpx-cli apy --token vMANTA --json
npx -y @bifrostio/slpx-cli apy --token vDOT --lp --json
```

Output fields: `token`, `totalApy`, `baseApy`, `rewardApy`, `rewardApyIncentiveAsset`

- `baseApy` — native staking yield on the underlying chain (Bifrost API: `apyBase`)
- `rewardApy` — Bifrost farming incentive component (Bifrost API: `apyReward`)
- `rewardApyIncentiveAsset` — **vDOT** for **vETH** only; **BNC** for every other vToken
- `totalApy` — `baseApy` + `rewardApy`

With `--lp`: adds `lpPools` — each entry: `symbol`, `project`, `chain`, `lpApy`, `tvl`

### `info`

Protocol overview: rate, APY, TVL, holders. vETH also: contract, chains, paused.

```bash
npx -y @bifrostio/slpx-cli info --json
npx -y @bifrostio/slpx-cli info --token vDOT --json
npx -y @bifrostio/slpx-cli info --token vASTR --json
```

Output fields: `protocol`, `token`, `rate`, `totalApy`, `tvl`, `totalStaked`, `totalSupply`, `holders`  
vETH extra: `contract`, `chains`, `paused`

## On-chain commands (vETH only)

### `balance <address>`

vETH balance and ETH equivalent. Comma-separated addresses for batch.

```bash
npx -y @bifrostio/slpx-cli balance 0x742d...bD18 --json
npx -y @bifrostio/slpx-cli balance 0x742d...bD18 --chain base --json
npx -y @bifrostio/slpx-cli balance 0xAddr1,0xAddr2,0xAddr3 --json
```

Single: `address`, `vethBalance`, `ethValue`, `chain`  
Batch: `results` (`{address, vethBalance, ethValue}`), `chain`

### `status <address>`

Redemption queue status.

```bash
npx -y @bifrostio/slpx-cli status 0x742d...bD18 --json
```

Output: `address`, `claimableEth`, `pendingAmount`, `chain`, `hint` (time estimate when pending)

### `mint <amount>`

Stake ETH → vETH. `--weth` uses WETH instead of native ETH.

```bash
npx -y @bifrostio/slpx-cli mint 0.1 --json --dry-run
npx -y @bifrostio/slpx-cli mint 0.5 --chain base --json
npx -y @bifrostio/slpx-cli mint 0.1 --weth --json --dry-run
npx -y @bifrostio/slpx-cli mint 0.5 --weth --chain arbitrum --json
```

Native ETH (unsigned): `action`, `input`, `expected`, `mode`, `unsigned.to`, `unsigned.value`, `unsigned.data`, `unsigned.chainId`  
WETH (unsigned): `action:mint-weth`, `input`, `expected`, `mode`, `wethAddress`, `steps` (Approve, Deposit)  
Signed: `action`, `input`, `expected`, `from`, `txHash`, `explorer`

WETH contract addresses: see `tokens-and-chains.md`.

### `redeem <amount>`

Redeem vETH — **not instant** (queue, often 1–3 days).

> Before a real `redeem`, confirm with the user: amount, chain, and that redemption is queued. Use `--dry-run` first.

```bash
npx -y @bifrostio/slpx-cli redeem 0.1 --json --dry-run --address 0x742d...
```

### `claim`

Claim completed ETH after redemption.

```bash
npx -y @bifrostio/slpx-cli claim --json --dry-run --address 0x742d...
```

## Global options

| Option | Description | Default |
|--------|-------------|---------|
| `--token <name>` | `vETH`, `vDOT`, `vKSM`, `vBNC`, `vGLMR`, `vMOVR`, `vFIL`, `vASTR`, `vMANTA`, `vPHA` | `vETH` |
| `--chain <name>` | `ethereum`, `base`, `optimism`, `arbitrum` (vETH on-chain only) | `ethereum` |
| `--rpc <url>` | Custom RPC | auto per chain |
| `--json` | JSON output | false |

## Transaction-related options

| Option | Description |
|--------|-------------|
| `--dry-run` | Build unsigned tx, do not broadcast |
| `--weth` | Mint from WETH (not native ETH) |
| `--lp` | LP yields on `apy` only |
| `--address <addr>` | Wallet address for operations when automatic signing is not configured (see CLI docs) |

## Workflow examples

### Compare APYs (with LP)

```bash
npx -y @bifrostio/slpx-cli apy --token vDOT --lp --json
npx -y @bifrostio/slpx-cli apy --token vETH --lp --json
npx -y @bifrostio/slpx-cli apy --token vKSM --json
```

### Batch balances

```bash
npx -y @bifrostio/slpx-cli balance 0xAddr1,0xAddr2,0xAddr3 --json
```

### Research → stake (vETH)

```bash
npx -y @bifrostio/slpx-cli info --json
npx -y @bifrostio/slpx-cli rate 1 --json
npx -y @bifrostio/slpx-cli apy --json
npx -y @bifrostio/slpx-cli mint 0.5 --json
```

### Redeem → claim (vETH)

```bash
npx -y @bifrostio/slpx-cli balance 0x... --json
npx -y @bifrostio/slpx-cli redeem 1.0 --json
npx -y @bifrostio/slpx-cli status 0x... --json
npx -y @bifrostio/slpx-cli claim --json
```

## Operational notes

1. Query commands use the Bifrost API for all 10 vTokens.
2. On-chain commands are vETH-only on the listed EVM chains.
3. Redemption goes through Bifrost’s cross-chain queue — not immediate settlement.
4. RPC: CLI may fall back to backup endpoints if primary RPC fails.
5. `--weth` unsigned flow emits Approve + Deposit steps.
6. `--lp` on `apy` pulls DeFiLlama pool data for the selected vToken.
