# Blast Exit Checker

A read-only page for the Blast L2 shutdown. Paste a wallet address and it shows:

- what the address still holds on Blast (ETH, USDB, WETH, BLAST, other tokens), with USD value
- withdrawals the address already started, and where each one stands: waiting, ready to prove, proven (claim opens at …), ready to claim, claimed
- whether each withdrawal, or the remaining balance, is worth more than the Ethereum fees needed to get it out
- a countdown to 26 October 2026, the date Blast gave for its normal bridge interface

**No wallet connection, no signatures, no approvals, no transactions.** The page only reads public data. It has no field for a seed phrase or private key, and it cannot move funds.

Unofficial community tool, not affiliated with Blast. Withdraw only through the official bridge at https://blast.io/en/bridge or the contracts in Blast's guide: https://docs.blast.io/withdrawals/withdraw-to-ethereum

## How it works

Everything runs in the visitor's browser. There is no server, no analytics and no storage. The page is one file, `index.html`, plus two font files.

| Service | Used for | Address sent? |
|---|---|---|
| `rpc.blast.io`, `blast-rpc.publicnode.com` | Blast balances, transaction receipts | yes |
| `ethereum-rpc.publicnode.com` (fallbacks `1rpc.io`, `cloudflare-eth.com`) | withdrawal status on the Blast portal, gas price | withdrawal IDs only |
| `api.routescan.io` | token list and transaction history of the address | yes |
| `api.coingecko.com`, `api.dexscreener.com` | prices | no |

Contracts read on Ethereum, all through `eth_call`:
- OptimismPortal `0x0Ec68c5B10F21EFFb74f2A5C61DFe6b08C0Db6Cb`, functions `provenWithdrawals`, `finalizedWithdrawals` and `paused`
- L2OutputOracle `0x826D1B0D4111Ad9146Eb8941D7Ca2B6a44215c76`, functions `latestOutputIndex`, `getL2Output` and `FINALIZATION_PERIOD_SECONDS`
- ETH withdrawal queue `0x98078db053902644191f93988341E31289E1C8FE`, function `getLastFinalizedRequestId`

Fee estimates use the medians of recent Blast portal transactions as of 24 Oct 2026: prove ≈ 429,500 gas and claim ≈ 241,100 gas, multiplied by the live Ethereum base fee plus a 0.02 gwei tip.

## Fonts

Overpass and Overpass Mono, © The Overpass Project Authors, under the SIL Open Font License 1.1 (see `fonts/OFL-*.txt`).
