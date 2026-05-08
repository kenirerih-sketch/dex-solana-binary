# Configuration

All parameters can be configured via CLI flags (`--flag-name`) or environment variables.

## Authentication


| Env Var          | Default      | Description                                       |
| ---------------- | ------------ | ------------------------------------------------- |
| `OKX_API_KEY`    | **Required** | API Key (AK) for HMAC signature authentication **[Apply here for whitelist](https://form.typeform.com/to/k0Axsial)**   |
| `OKX_SECRET_KEY` | **Required** | Secret Key (SK) for HMAC signature authentication **[Apply here for whitelist](https://form.typeform.com/to/k0Axsial)**|
| `OKX_PASSPHRASE` | **Required** | Passphrase for OKX AK authentication              |


## Service & Connectivity


| Env Var                  | Default      | Description                                                                                                                                                                                                   |
| ------------------------ | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PORT`                   | `8080`       | Business API listen port                                                                                                                                                                                      |
| `METRICS_PORT`           | `9100`       | Ops Metrics listen port (/health, /ready, /status, /metrics)                                                                                                                                                  |
| `METRICS_BIND`           | `127.0.0.1`  | Ops Metrics bind address. Set to `0.0.0.0` for external access (e.g. Prometheus scraping)                                                                                                                     |
| `RPC_URL`                | **Required** | Solana RPC endpoint                                                                                                                                                                                           |
| `GEYSER_GRPC_ENDPOINT`   | Not set      | Geyser gRPC streaming endpoint                                                                                                                                                                                |
| `GEYSER_GRPC_X_TOKEN`    | Not set      | Geyser gRPC authentication token                                                                                                                                                                              |
| `GEYSER_MODE`            | `auto`       | `auto` / `yellowstone` / `richat` / `rpc-only`. In auto mode, uses Yellowstone if gRPC endpoint is set, otherwise falls back to RPC polling. Specifying yellowstone or richat requires `GEYSER_GRPC_ENDPOINT` |
| `GEYSER_GRPC_COMMITMENT` | `processed`  | Commitment level for gRPC subscriptions and RPC fallback. Valid values: `processed` / `confirmed` / `finalized` (case-insensitive)                                                                            |
| `SHUTDOWN_TIMEOUT_SECS`  | `1`          | Graceful shutdown timeout (seconds)                                                                                                                                                                           |
| `RPC_MAX_CONCURRENT_REQUESTS` | `100` | Max concurrent RPC `getMultipleAccounts` batch requests                                                                                                                                                       |
| `RPC_POLL_INTERVAL_MS`   | `200`        | Polling interval in milliseconds (RPC-only mode)                                                                                                                                                              |


## gRPC Streaming Tuning


| Env Var                                | Default | Description                                                               |
| -------------------------------------- | ------- | ------------------------------------------------------------------------- |
| `GEYSER_STREAMING_CHUNK_COUNT`         | `12`    | Number of parallel gRPC streams                                           |
| `GEYSER_GRPC_RECV_TIMEOUT`             | `60000` | Message receive timeout (ms); disconnects and reconnects on timeout       |
| `GEYSER_GRPC_CONNECT_TIMEOUT_MS`       | `10000` | Connection establishment timeout (ms)                                     |
| `GEYSER_GRPC_MAX_MESSAGE_DELAY`        | `10`    | Maximum allowed message delay (seconds); disconnects if exceeded          |
| `GEYSER_GRPC_RESET_ON_GAP`             | `300`   | Stream gap threshold (seconds) to trigger full RPC backfill; 0 to disable |
| `GEYSER_GRPC_CHANNEL_UPDATE_CAPACITY`  | `16384` | Internal channel capacity for account updates                             |
| `GRPC_CONSISTENCY_CHECK_INTERVAL_SECS` | `30`    | Consistency check interval (seconds)                                      |
| `RICHAT_MAX_PUBKEYS_PER_FILTER`        | `100`   | Richat mode: max pubkeys per filter (must be > 0)                         |
| `RICHAT_MAX_FILTERS_PER_SUBSCRIBE`     | `10`    | Richat mode: max filters per subscribe request (must be >= 2)             |
| `SOFT_RECONNECT_TIMEOUT_SECS`         | `300`   | Soft-reconnect timeout (seconds); escalates to HardReconnecting (503) when exceeded |
| `SLOT_FRESHNESS_THRESHOLD`             | `50`    | Max allowed slot lag during startup readiness gate                         |
| `SLOT_FRESHNESS_TIMEOUT_SECS`          | `30`    | Max wait time (seconds) for slot freshness gate at startup                |


## Threading & Performance


| Env Var            | Default    | Description                       |
| ------------------ | ---------- | --------------------------------- |
| `API_THREADS`      | `0` (auto) | API service thread count          |
| `QUOTE_THREADS`    | `0` (auto) | Quote computation thread count    |
| `PIPELINE_THREADS` | `0` (auto) | Data pipeline thread count        |
| `ROUTE_THREADS`    | `0` (auto) | Route computation Rayon pool size |


Each parameter is evaluated independently. When set to `0`, auto-allocation follows this table:


| CPU Cores | API | Quote | Pipeline | Route     |
| --------- | --- | ----- | -------- | --------- |
| <= 8      | 1   | 1     | 2        | Remaining |
| > 8       | 2   | 2     | 4        | Remaining |


## Market Data


| Env Var                | Default  | Description                                                                                                                                                                                     |
| ---------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ALLOWED_TOKEN_MINTS`  | Not set  | Token mint allowlist, comma-separated. Loads all if not set                                                                                                                                     |
| `BLOCKED_DEX_PROGRAMS` | Not set  | DEX program ID denylist, comma-separated                                                                                                                                                        |
| `ALLOWED_DEX_PROGRAMS` | Not set  | DEX program ID allowlist, comma-separated. Loads all if not set                                                                                                                                 |
| `BLOCKED_POOLS`        | Not set  | Startup pool denylist, comma-separated base58 pool addresses. Matched pools are excluded from all caches; invalid entries are skipped with a warning. Use `POST /evict-pools` to add at runtime |


## Data Directory


| Env Var                      | Default                                         | Description                                                                                                                                                           |
| ---------------------------- | ----------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PALLAS_DATA_DIR`            | `./data`                                        | Pallas persistent data directory. Currently stores the runtime pool denylist file. Auto-created on first write                                                        |
| `BLOCKED_POOLS_RUNTIME_FILE` | `${PALLAS_DATA_DIR}/blocked_pools_runtime.json` | Full path to the runtime pool denylist file. Written atomically (tmp + rename) after each successful `POST /evict-pools` call; merged with `BLOCKED_POOLS` on restart |


## Observability


| Env Var     | Default | Description                                      |
| ----------- | ------- | ------------------------------------------------ |
| `LOG_LEVEL` | `info`  | Log level (e.g. `info`, `debug`)                 |
| `LOG_JSON`  | `false` | JSON-formatted log output                        |
