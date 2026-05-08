# POST /evict-pools

Removes specified pools from routing and adds them to the runtime denylist. Evicted pools are excluded from all subsequent quotes until the service restarts or the denylist is cleared.

---

## Request

`POST /evict-pools` with JSON body.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `chainId` | uint64 | No | 501 | Chain identifier (501 = Solana) |
| `pools` | string[] | Yes | — | Base58-encoded pool addresses. 1-1000 items per request |

---

## Response

| Field | Type | Description |
|-------|------|-------------|
| `successful` | string[] | Pools successfully evicted or already absent (added to denylist either way) |
| `failed` | FailedItem[] | Pools that could not be processed |
| `items` | Item[] | Per-pool status in input order |

### Item

| Field | Type | Description |
|-------|------|-------------|
| `address` | string | Pool address |
| `status` | string | `evicted`, `not_found`, `invalid`, or `persist_error` |

### FailedItem

| Field | Type | Description |
|-------|------|-------------|
| `address` | string | Pool address |
| `status` | string | `invalid` or `persist_error` |
| `reason` | string? | Error detail |

### Status Reference

| Status | Meaning | Result |
|--------|---------|--------|
| `evicted` | Removed from cache and added to denylist | success |
| `not_found` | Not in cache, but added to denylist | success |
| `invalid` | Bad address format | failure |
| `persist_error` | Evicted in memory, but denylist file write failed | failure |

---

## Errors

| HTTP | Code | When |
|------|------|------|
| 400 | `INVALID_REQUEST` | Empty pools or > 1000 items |
| 400 | `CHAIN_NOT_CONFIGURED` | Unknown `chainId` |
| 400 | `CHAIN_NOT_SUPPORTED` | Chain does not support pool eviction |
| 503 | — | Engine offline or overloaded |

---

## Example

```bash
curl -s -X POST 'http://localhost:8080/evict-pools' \
  -H 'Content-Type: application/json' \
  -d '{
    "pools": [
      "8sLbNZoA1cfnvMJLPfp98ZLAnFSYCFApfJKMbiXNLwxj",
      "4GkRbcYg1VKsZropgai4dMf2418GNJRF1QwNe54CsBD5"
    ]
  }'
```

```json
{
  "successful": [
    "8sLbNZoA1cfnvMJLPfp98ZLAnFSYCFApfJKMbiXNLwxj",
    "4GkRbcYg1VKsZropgai4dMf2418GNJRF1QwNe54CsBD5"
  ],
  "failed": [],
  "items": [
    {
      "address": "8sLbNZoA1cfnvMJLPfp98ZLAnFSYCFApfJKMbiXNLwxj",
      "status": "evicted"
    },
    {
      "address": "4GkRbcYg1VKsZropgai4dMf2418GNJRF1QwNe54CsBD5",
      "status": "not_found"
    }
  ]
}
```

For operational guidance (eviction flow, persistence, monitoring, recovery), see [Support](support.md#runtime-pool-eviction).
