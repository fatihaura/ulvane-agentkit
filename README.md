# ulvane-agentkit

A [Coinbase AgentKit](https://github.com/coinbase/agentkit) action provider for [Ulvane](https://www.ulvane.xyz): x402-priced onchain intelligence for Base. Your agent's own wallet pays per call in USDC on Base mainnet — no API keys, no accounts.

| Action | What it does | Price |
|---|---|---|
| `ulvane_token_risk` | Verified / impersonator / unvetted check for a token | $0.01 |
| `ulvane_wallet_pnl` | Realized/unrealized PnL from recent swaps | $0.02 |
| `ulvane_wallet_risk` | Has this wallet traded flagged impersonator tokens? | $0.02 |
| `ulvane_token_top_traders` | Biggest recent traders of a token, with PnL | $0.03 |
| `ulvane_pre_trade_gate` | Token risk + top traders + optional wallet risk → clear / caution / avoid | $0.05 |
| `ulvane_pricing` | Machine-readable price list | free |
| `ulvane_leaderboard` | Top Aerodrome wallets by realized PnL | free |

## Use

```bash
npm install ulvane-agentkit @coinbase/agentkit
```

```ts
import { AgentKit } from "@coinbase/agentkit";
import { ulvaneActionProvider } from "ulvane-agentkit";

const agentKit = await AgentKit.from({
  walletProvider, // an EVM wallet provider on base-mainnet holding a little USDC
  actionProviders: [ulvaneActionProvider()],
});
```

Options: `ulvaneActionProvider({ maxPriceUsd: 0.05, baseUrl, trustedSigner })`.

Use a dedicated wallet holding only a few dollars of USDC.

## Safety

- **Spend cap, checked before anything is signed.** Each paid action first asks Ulvane what it would charge (an unpaid request that returns the 402) and refuses unless the demand is USDC on Base mainnet within `maxPriceUsd` (default $0.05). It does not rely on a price table baked into this package.
- **Responses are signature-checked.** Ulvane signs every paid response (EIP-191). Results include `_verification` with `signatureValid` (body not altered) and `trustedSigner` (signed by Ulvane's published address `0xdCac…ECe8`). Treat `trustedSigner: false` as "don't rely on this".
- **On-chain text is untrusted.** Token names and symbols come from arbitrary contracts. Ulvane strips control/invisible characters and caps their length, and every result carries a `_notice` telling the model to treat them as data, never as instructions.
- Addresses are validated before any request is made. Payment is only settled when a request succeeds (status below 400). Wallets on other networks are refused.

Payments go through AgentKit's built-in `x402ActionProvider`.

## License

MIT
