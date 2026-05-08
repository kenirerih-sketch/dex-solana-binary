# POST /swap-instruction

Returns an optimal swap quote and decomposed raw Solana instructions. The caller assembles them into a custom transaction for greater flexibility.

---

## Request

`POST /swap-instruction` with JSON body.

Request parameters are identical to [`POST /swap`](api-swap).

### Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fromTokenAddress` | String | Yes | The contract address of a token you want to send (e.g., `So11111111111111111111111111111111111111112`) |
| `toTokenAddress` | String | Yes | The contract address of a token you want to receive (e.g., `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`) |
| `amount` | String | Yes | The input amount of a token to be sold (set in minimal divisible units, e.g., 1.00 SOL set as `1000000000`) |
| `slippagePercent` | String | No | Slippage limit. The slippage setting has a minimum value of `0` and a maximum value of less than `100` (e.g., `0.5` means that the maximum slippage for this transaction is `0.5%`). Default: `0.5` |
| `userWalletAddress` | String | Yes | User's wallet address (e.g., `J5CBzXpcYn6WR2JBah8zU4Yxct985CAFGwXRcFaX2pbS`) |
| `dexIds` | String | No | DexId of the liquidity pool for limited quotes, multiple combinations separated by `,`. Use `/program-id-to-label` to look up IDs |
| `excludedDexIds` | String | No | The dexId of the liquidity pool that will not be used, multiple combinations separated by `,` |
| `directRoute` | Boolean | No | Default is `false`. When enabled, restricts routing to a single liquidity pool only |
| `singleRouteOnly` | Boolean | No | Default is `false`. When enabled, routing is restricted to a single route. Multi-hop and multi-pool routes are allowed, but no parallel split routes will be constructed |
| `singlePoolPerHop` | Boolean | No | Default is `false`. When enabled, each hop in the route is restricted to a single pool |
| `stableIntermediateTokensOnly` | Boolean | No | Default is `false`. When enabled, routing will restrict intermediate tokens to stablecoins (e.g. USDC, USDT) to reduce high-slippage path risk |
| `enableCyclicArbitrage` | Boolean | No | Default is `false`. When enabled, enables cyclic arbitrage mode. `fromTokenAddress` and `toTokenAddress` must be the same, forming a circular route |
| `cyclicArbitrageIntermediateTokens` | String | No | Custom intermediate token mints, comma-separated. Only effective when `enableCyclicArbitrage` is `true` |
| `maxAccounts` | String | No | Provides an estimate of the maximum number of accounts that used for an instruction. It's useful when composing your own transaction, or if you want more precise resource accounting to optimize routing. Default: `64` |
| `swapReceiverAddress` | String | No | Recipient address of a purchased token. If not set, `userWalletAddress` will receive a purchased token |
| `computeUnitPrice` | String | No | Used for transactions on the Solana network and similar to gasPrice on Ethereum. This price determines the priority level of the transaction. The higher the price, the more likely that the transaction can be processed faster |
| `computeUnitLimit` | String | No | Used for transactions on the Solana network and analogous to gasLimit on Ethereum, which ensures that the transaction won't take too much computing resource. If the parameter `tips` is not `0`, then `computeUnitLimit` should be set to `0`. Otherwise, the fee is wasted |
| `tips` | String | No | Jito tips in lamports. This is used for MEV protection |
| `useTokenLedger` | Boolean | No | Default is `false`. When `true`, uses token ledger for dynamic input amount detection |


## Response

| Field | Type | Description |
|-------|------|-------------|
| `tokenLedgerInstruction` | Instruction | Token ledger snapshot instruction (present when `useTokenLedger=true`) |
| `computeBudgetInstructions` | Instruction[] | Compute budget instructions (CU limit + priority fee) |
| `setupInstructions` | Instruction[] | Token account creation/initialization instructions |
| `swapInstruction` | Instruction | Core swap instruction |
| `cleanupInstruction` | Instruction | SOL wrap/unwrap cleanup instruction (null if not needed) |
| `otherInstructions` | Instruction[] | Additional instructions (e.g. cyclic arbitrage second leg) |
| `tipInstruction` | Instruction | Jito tip transfer instruction (present when `tips` is set) |
| `addressLookupTableAddresses` | String[] | Address Lookup Table Account. Used to optimize the management and referencing of addresses in transactions by storing related addresses in a table and referencing them via index values |
| `prioritizationFeeLamports` | String | Total prioritization fee in lamports |
| `blockhashWithMetadata` | BlockhashMetadata | Recent blockhash and its last valid block height |
| `routerResult` | QuoteResponse | Quote path data (see [`POST /swap`](api-swap)) |

### Instruction

| Field | Type | Description |
|-------|------|-------------|
| `programId` | String | Program ID for instruction execution |
| `accounts` | AccountMeta[] | Instruction account information |
| `data` | String | Instruction data (base64 encoded) |

### AccountMeta

| Field | Type | Description |
|-------|------|-------------|
| `pubkey` | String | Public key address of the account |
| `isSigner` | Boolean | Whether the account is a signer |
| `isWritable` | Boolean | Whether the account is writable |

### BlockhashMetadata

| Field | Type | Description |
|-------|------|-------------|
| `blockhash` | String | Recent blockhash (base58) |
| `lastValidBlockHeight` | String | Last valid block height for this blockhash |

---

## Transaction Assembly Order

Assemble instructions into a Solana transaction in this order:

1. `tokenLedgerInstruction` (if present)
2. `computeBudgetInstructions`
3. `setupInstructions`
4. `swapInstruction`
5. `cleanupInstruction` (if present)
6. `otherInstructions`
7. `tipInstruction` (if present)

Use `addressLookupTableAddresses` to fetch ALTs for building a Versioned Transaction, then sign and submit.

---

## Example

```bash
curl -s -X POST 'http://localhost:8080/swap-instruction' \
  -H 'Content-Type: application/json' \
  -d '{
    "fromTokenAddress": "So11111111111111111111111111111111111111112",
    "toTokenAddress": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
    "amount": "1000000000",
    "slippagePercent": "0.5",
    "userWalletAddress": "YOUR_WALLET_PUBLIC_KEY"
  }'
```

```json
{
  "computeBudgetInstructions": [
    {
      "programId": "ComputeBudget111111111111111111111111111111",
      "accounts": [],
      "data": "AgY6BAA="
    },
    {
      "programId": "ComputeBudget111111111111111111111111111111",
      "accounts": [],
      "data": "Axm5AgAAAAAA"
    }
  ],
  "setupInstructions": [
    {
      "programId": "ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL",
      "accounts": [
        {"pubkey": "...", "isSigner": true, "isWritable": true},
        {"pubkey": "...", "isSigner": false, "isWritable": true}
      ],
      "data": ""
    }
  ],
  "swapInstruction": {
    "programId": "proVF4pMXVaYqmy4NjniPh4pqKNfMmsihgd4wdkCX3u",
    "accounts": [
      {"pubkey": "...", "isSigner": true, "isWritable": true}
    ],
    "data": "..."
  },
  "cleanupInstruction": null,
  "otherInstructions": [],
  "tipInstruction": null,
  "addressLookupTableAddresses": [
    "...",
    "..."
  ],
  "prioritizationFeeLamports": "200000",
  "blockhashWithMetadata": {
    "blockhash": "GHtXQBpY7s...",
    "lastValidBlockHeight": "123456789"
  },
  "routerResult": {
    "fromTokenAddress": "So11111111111111111111111111111111111111112",
    "toTokenAddress": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
    "fromTokenAmount": "1000000000",
    "toTokenAmount": "87521745",
    "contextSlot": "310482917",
    "slippagePercent": "0.5",
    "dexRouterList": [...]
  }
}
```
