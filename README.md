# Bifrost agent skills

## Installation

Install **one** skill by name:

```bash
npx skills add bifrost-io/skills --skill bifrost-slpx
```

Install **all** skills from this repo:

```bash
npx skills add bifrost-io/skills --skill '*'
```

Install globally (so the skill is available across projects), add **`-g`** if your `skills` CLI supports it:

```bash
npx skills add bifrost-io/skills --skill '*' -g
```

## Skills

### Hand-maintained

| Skill                           | Description                                                                                                                                                          |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [bifrost-slpx](./bifrost-slpx/) | Bifrost SLPx liquid staking via [`@bifrostio/slpx-cli`](https://www.npmjs.com/package/@bifrostio/slpx-cli): rates, APY, info, vETH mint/redeem/claim/balance/status. |

## Related repos

- **CLI only (npm package, no skills bundled):** [slpx-cli](https://github.com/bifrost-io/slpx-cli) — `@bifrostio/slpx-cli`

## License

[MIT](./LICENSE) — same terms as the skill markdown and references in this repository.
