# GET /program-id-to-label

Returns a mapping from on-chain program IDs to human-readable protocol labels. Use this to look up IDs for the `dexIds` / `excludedDexIds` parameters in `/swap` and `/swap-instruction`.

---

## Request

`GET /program-id-to-label`

No parameters.

---

## Response

`200 OK` — JSON object. Keys are program IDs, values are protocol labels.

```json
{
  "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8": "Raydium",
  "CAMMCzo5YL8w4VFF8KVHrK22GGUsp5VTaW7grrKgrWqK": "Raydium CLMM",
  "whirLbMiicVdio4qvUfM5KAg6Ct8VwpYzGff3uctyCc": "Orca Whirlpool",
  "LBUZKhRxPF3XUpBCjp4YzTKgLccjZhTSDM9YuVaPwxo": "Meteora DLMM",
  "PhoeNiXZ8ByJGLkxNfZRnkUfjvmuYqLR89jjFHGqdXY": "Phoenix",
  "6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P": "Pump.fun"
}
```

---

## Example

```bash
curl -s 'http://localhost:8080/program-id-to-label' | jq .
```