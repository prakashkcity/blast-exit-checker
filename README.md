# Blast Exit Checker

A page for the Blast L2 shutdown. Blast paused its Ethereum portal on 3 November 2026 and moved the bridge's funds to a recovery claim at https://blast.io. Paste a wallet address and this page shows:

- what Blast's recovery list holds for the address, in ETH and DAI, per list, and whether it has been claimed
- the other accounts the wallet is approved to claim for, such as a Safe or a contract it deployed, and what they hold
- the Ethereum fee for those claims, and how much ETH the address holds on Ethereum to pay it
- what the address still holds on Blast (ETH, USDB, WETH, BLAST, other tokens), with USD value
- money parked in apps: lending receipts, vault shares and liquidity tokens, valued by what they hold
- NFTs on Blast, which the recovery does not cover
- withdrawals the address started on the old bridge

**Checking needs no wallet.** The page reads public data only. It has no field for a seed phrase or private key. A link like `…/blast-exit-checker/#0xabc…` opens with that address checked, and each card has a Copy link button. The part after `#` stays in the browser.

**Claim (optional, no service fee).** A wallet card has a Claim button at the top, and one per row when there are several claims. It sends the same `claim()` call as Blast's own page, https://blast.io, built from the same public files, to Blast's recovery contract on Ethereum. Only the wallet that held the funds or a wallet approved for it can send it, and the ETH and DAI go to the wallet that sends it, so another wallet cannot pay the gas. Several claims go as one transaction when the wallet can batch them (EIP-5792); otherwise they go one at a time, so any wallet works. A Safe claims on blast.io.

Before the wallet is asked, and again right before sending, the proof must rebuild the root in Blast's manifest, the contract must accept that root and not be paused, the account must be unclaimed, the wallet must be approved (`isApprovedRedeemer`), and the call must simulate from that wallet. No token approvals, no message signatures. The contract takes no fee and neither does this page. Other tools can call the same contract too; some of them charge a percentage.

**The old route.** While the portal reads as paused, or its state cannot be read, the page offers none of its old Withdraw, Unwrap, Prove and Claim buttons, and started withdrawals show as "Bridge closed". The code stays in the file: if the portal reads as open, those buttons come back. They ask the visitor's own wallet (any EIP-1193 / EIP-6963 wallet) to send one transaction, after rebuilding it from Blast and Ethereum data and simulating it with `eth_estimateGas`.

Unofficial community tool, not affiliated with Blast.

## How it works

Everything runs in the visitor's browser. There is no server, no analytics and no storage. The page is one file, `index.html`, plus two font files and, loaded only when a Claim button or the tip window needs it, `vendor/ethers-6.13.4.umd.min.js`: an unmodified copy of ethers.js 6.13.4 (`dist/ethers.umd.min.js`, MIT licence in `vendor/`), loaded with a subresource-integrity hash.

The recovery amounts come from the same static files blast.io loads: `claims/contract-1/` and `claims/contract-2/`, each with a `manifest.json` and `shards/<first 3 hex characters of the address>.json`. An entry is `[ethWei, daiWei, tree, leaf, redeemers?, pendingWithdrawals?]`, and a 404 means an empty shard. `redeemers/<first 2 hex characters>.json` maps a wallet to the accounts it is approved to claim for; the page shows up to 30 of them and says when there are more. `trees/<tree>/<chunk>.bin` holds the Merkle nodes a proof is read from. Any failed read is shown as an error, never as "not listed".

| Service | Used for | Address sent? |
|---|---|---|
| `blast.io` | Blast's recovery list | first 2 or 3 characters only |
| `rpc.blast.io` (the only keyless Blast RPC left; the page stays under its limits of 40 calls a second and 600 a minute) | Blast balances, transaction receipts | yes |
| `ethereum-rpc.publicnode.com` (fallbacks `1rpc.io`, `cloudflare-eth.com`) | claim status, withdrawal status on the Blast portal, gas price | yes |
| `api.routescan.io` | token list and transaction history of the address | yes |
| `api.coingecko.com`, `api.dexscreener.com` | prices | no |

Contracts read on Ethereum, all through `eth_call`:
- BlastExit `0x9D31771458B421C7EA8A06CB8D12663E91617869` and `0x23eA8c6ACEb5673BBd712137cE2Aa33db36CB513`, functions `claimed`, `paused`, `roots` and `isApprovedRedeemer`
- OptimismPortal `0x0Ec68c5B10F21EFFb74f2A5C61DFe6b08C0Db6Cb`, functions `provenWithdrawals`, `finalizedWithdrawals` and `paused`
- L2OutputOracle `0x826D1B0D4111Ad9146Eb8941D7Ca2B6a44215c76`, functions `latestOutputIndex`, `getL2Output` and `FINALIZATION_PERIOD_SECONDS`
- ETH withdrawal queue `0x98078db053902644191f93988341E31289E1C8FE`, function `getLastFinalizedRequestId`
- USDB→DAI withdrawal queue `0xa230285d5683C74935aD14c446e137c8c8828438`, functions `getWithdrawalRequests` and `getWithdrawalStatus`

The claim fee uses 88,000 gas, from the 87,980 median of the last 100 real claims on 4 Nov 2026, multiplied by the live Ethereum base fee plus a 0.02 gwei tip.

## Tip jar

If the page helped you, a small tip is welcome but never required: any token or NFT on any EVM chain to `0xfa2F25f62c16EEF1Feb6E474eCaF525eB335C4b2`. The checker works the same either way, and nothing is unlocked by tipping.

## Coming soon

A read-only checker like this one for Abstract.

## Fonts

Overpass and Overpass Mono, © The Overpass Project Authors, under the SIL Open Font License 1.1 (see `fonts/OFL-*.txt`).
