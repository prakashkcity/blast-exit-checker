# Blast Exit Checker

A page for the Blast L2 shutdown. Paste a wallet address and it shows:

- what the address still holds on Blast (ETH, USDB, WETH, BLAST, other tokens), with USD value
- money parked in apps: lending receipts, vault shares and liquidity tokens, valued by what they hold
- NFTs on Blast, which the official bridge cannot move
- how much ETH the address has on Ethereum to pay for prove and claim
- withdrawals the address already started, and where each one stands: waiting, ready to prove, proven (claim opens at …), ready to claim, claimed
- whether each withdrawal, or the remaining balance, is worth more than the Ethereum fees needed to get it out
- a countdown to 26 October 2026, the date Blast gave for its normal bridge interface

**Checking needs no wallet.** The page reads public data only. It has no field for a seed phrase or private key.

**Withdraw and Unwrap (optional).** ETH and USDB still on Blast get a Withdraw button, WETH an Unwrap button. Withdraw sends one transaction on Blast from the wallet that holds the funds, as Blast's docs describe for contracts: `bridgeETHTo(to, 200000, 0x)` on the L2BlastBridge `0x4300…0005` with the ETH as value, or `bridgeERC20To(USDB, DAI, to, amount, 800000, 0x)` (no approval; it arrives as DAI). Unwrap calls `withdraw(wad)` on WETH `0x4300…0004`. "Max" keeps the Blast gas aside, including the L1 data fee Blast's node adds to its balance check, and is recomputed right before sending.

**Prove and Claim (optional).** Each started withdrawal gets a button when it is ready: Prove, Claim, or, for USDB's last step, Claim DAI. Pressing one asks the visitor's own wallet (Rabby, MetaMask, any EIP-1193 / EIP-6963 wallet) to send one transaction on Ethereum to Blast's portal, or to Blast's DAI withdrawal queue. These call Blast's contracts directly, so they keep working after the bridge page closes. No token approvals, no message signatures. Any wallet can pay the gas, so a friend can finish someone else's withdrawal; the funds always go to the address set inside the withdrawal. Only the DAI claim must come from its recipient.

Before the wallet is asked, the page rebuilds the transaction from Blast and Ethereum data and simulates it with `eth_estimateGas`: a proof must match the output root Blast posted on Ethereum, and a claim's gas limit comes from a simulation as `Constants.ESTIMATION_ADDRESS` (`0x…0001`), which makes the portal revert instead of finalizing a withdrawal whose payout would fail. The calldata is byte-for-byte what the command-line claimer this page grew out of builds (checked on 26 Oct 2026 for a prove and a claim).

Unofficial community tool, not affiliated with Blast. Withdraw only through the official bridge at https://blast.io/en/bridge or the contracts in Blast's guide: https://docs.blast.io/withdrawals/withdraw-to-ethereum

## How it works

Everything runs in the visitor's browser. There is no server, no analytics and no storage. The page is one file, `index.html`, plus two font files and, loaded only when a Prove / Claim button is pressed, `vendor/ethers-6.13.4.umd.min.js`: an unmodified copy of ethers.js 6.13.4 (`dist/ethers.umd.min.js`, MIT licence in `vendor/`), loaded with a subresource-integrity hash.

| Service | Used for | Address sent? |
|---|---|---|
| `rpc.blast.io`, `blast-rpc.publicnode.com` | Blast balances, transaction receipts | yes |
| `ethereum-rpc.publicnode.com` (fallbacks `1rpc.io`, `cloudflare-eth.com`) | withdrawal status on the Blast portal, gas price | withdrawal IDs only |
| `api.routescan.io` | token list and transaction history of the address | yes |
| `api.coingecko.com`, `api.dexscreener.com` | prices | no |

Contracts read on Ethereum, all through `eth_call`:
- OptimismPortal `0x0Ec68c5B10F21EFFb74f2A5C61DFe6b08C0Db6Cb`, functions `provenWithdrawals`, `finalizedWithdrawals` and `paused`
- L2OutputOracle `0x826D1B0D4111Ad9146Eb8941D7Ca2B6a44215c76`, functions `latestOutputIndex`, `getL2Output` and `FINALIZATION_PERIOD_SECONDS`
- ETH withdrawal queue `0x98078db053902644191f93988341E31289E1C8FE`, function `getLastFinalizedRequestId` (and, for a claim, `getLastCheckpointId`, `findCheckpointHint`, `getClaimableEther`)
- USDB→DAI withdrawal queue `0xa230285d5683C74935aD14c446e137c8c8828438`, functions `getWithdrawalRequests` and `getWithdrawalStatus` (and, for a DAI claim, the same hint functions)

Transactions the buttons ask a wallet to send: on Blast, `bridgeETHTo` / `bridgeERC20To` on the L2BlastBridge and `withdraw` on WETH; on Ethereum, `proveWithdrawalTransaction` and `finalizeWithdrawalTransaction` on the portal, `claimWithdrawal` on the DAI queue. A proof also reads `eth_getProof` from `rpc.blast.io`, and "Max" reads `getL1Fee` from Blast's GasPriceOracle `0x4200…000F`.

Fee estimates use the medians of recent Blast portal transactions as of 24 Oct 2026: prove ≈ 429,500 gas and claim ≈ 241,100 gas, multiplied by the live Ethereum base fee plus a 0.02 gwei tip.

## Tip jar

If the page helped you, a small tip is welcome but never required: any token or NFT on any EVM chain to `0xfa2F25f62c16EEF1Feb6E474eCaF525eB335C4b2`. The checker works the same either way, and nothing is unlocked by tipping.

## Coming soon

A read-only checker like this one for Abstract.

## Fonts

Overpass and Overpass Mono, © The Overpass Project Authors, under the SIL Open Font License 1.1 (see `fonts/OFL-*.txt`).
