# Troubleshooting

## macOS Security Prompt

When running the binary on macOS, the system may block it. To resolve:

1. Open **System Settings**
2. Go to **Privacy & Security**
3. Click **Open Anyway**

## Runtime Pool Eviction

When a DEX pool behaves abnormally (incorrect quotes, sudden liquidity changes, etc.) and needs to be immediately taken out of rotation, use `POST /evict-pools` to dynamically evict it without restarting Pallas. For API details, see [POST /evict-pools](api-evict-pools.md).

Eviction flow:

- Immediately removes the specified pool from all in-memory caches (pool metadata / token-to-markets index / account-to-pools / pool_tick_watchset / alt-to-pools / pool_slots / algorithm topology)
- Automatically detects orphaned ALTs and unsubscribes from gRPC; ALTs still referenced by other pools are retained
- Writes to the persistent runtime denylist file (`${BLOCKED_POOLS_RUNTIME_FILE}`), effective across restarts
- All subsequent reload paths (biz_tick incremental pushes, reconnection recovery, soft sync, etc.) skip denylisted pools

This endpoint is currently unauthenticated; access control is handled at the deployment layer via network policies / reverse proxy ACLs.

Monitoring metrics:


| Metric                                      | Type    | Description                                                                                                                       |
| ------------------------------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `pallas_evicted_pools_total{source,result}` | counter | `source` in {`startup`,`runtime`}, `result` in {`evicted`,`not_found`,`invalid`}. Aggregates startup filtering + runtime eviction |
| `pallas_blocked_pools_size`                 | gauge   | Current effective denylist size (startup union runtime)                                                                           |


**Restoring an evicted pool:** There is currently no unblock endpoint. To restore a pool, edit `${BLOCKED_POOLS_RUNTIME_FILE}` and/or the `BLOCKED_POOLS` environment variable, then restart Pallas.

# FAQ

**`/ready` returns 503 after startup**

This is expected. Pallas needs to load market data and account state after starting, which typically takes several minutes before entering the Normal state. Check loading progress via `/status` on the ops port (default 9100).

**Excessive RPC node load**

- Use gRPC streaming mode instead of RPC polling (configure `GEYSER_GRPC_ENDPOINT`)

**"Authentication failed" or "Invalid passphrase"**

The passphrase is the one you set when creating the API Key on the OKX Dev Portal, not your wallet password or OKX account password.

**"Forbidden" or API Key not working**

After creating an API Key, ensure the required permissions and IP whitelist are configured on the OKX Dev Portal.

**429 Too Many Requests from RPC node**

Free-tier RPC nodes have rate limits. Occasional 429 warnings during startup are normal — Pallas respects `RPC_MAX_CONCURRENT_REQUESTS` (default 100) to cap concurrent RPC calls, and will automatically retry. If 429 errors persist, consider using a paid RPC endpoint or reducing `RPC_MAX_CONCURRENT_REQUESTS` (e.g. to 10-20).

# Feedback & Support

For other issues or assistance, visit the GitHub repository to check existing issues or submit a new one:

[https://github.com/okx/dex-solana-binary](https://github.com/okx/dex-solana-binary)
