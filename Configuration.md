# Cobblemon Sync Configuration Guide

## Redis Configuration

The Redis configuration is stored in `config/cobblemon-sync/redis.json`:

```json
{
  "url": "redis://:SecurePassword!@localhost:29999",
  "enableDebugLogs": false,
  "threadPoolSize": 8,
  "maxPoolConnections": 128,
  "useCachedThreadPool": false,
  "timeouts": {
    "firstTimeConnection": 2000,
    "regularConnection": 10000,
    "serverSwitch": 15000,
    "existingDataCheck": 1000
  }
}
```

### Configuration Options:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `url` | String | `redis://:SecurePassword!@localhost:29999` | Redis connection URL |
| `enableDebugLogs` | Boolean | `false` | Whether to enable detailed debug logging |
| `threadPoolSize` | Integer | `8` | Number of threads in the worker pool for sync operations |
| `maxPoolConnections` | Integer | `128` | Maximum Redis connections in the connection pool |
| `useCachedThreadPool` | Boolean | `false` | Whether to use an auto-scaling cached thread pool instead of fixed size |
| `timeouts` | Object | See below | Contains various timeout settings in milliseconds |

### Timeout Options:

| Option | Type | Default (ms) | Description |
|--------|------|-------------|-------------|
| `firstTimeConnection` | Long | `2000` | Timeout for first-time player connections (no existing data) |
| `regularConnection` | Long | `10000` | Timeout for regular player connections (has existing data) |
| `serverSwitch` | Long | `15000` | Timeout for players switching between servers |
| `existingDataCheck` | Long | `1000` | Timeout when checking if player data exists on other servers |

### Redis URL Format:

The Redis URL follows this format:
```
redis://[username][:password@]host[:port][/database]
```

Examples:
- `redis://:password@localhost:6379` - Redis with password on localhost, standard port
- `redis://redis-server.example.com:6380` - Redis on custom host and port without password
- `redis://username:password@redis.example.com:6379/0` - Redis with username, password, and database selection

### Debug Logging

When `enableDebugLogs` is set to `true`, the mod will log detailed information about:
- Redis messages being sent and received
- Player state changes during server switching
- Detailed synchronization process information
- Cache operations

Set this to `false` in production to reduce log spam and improve performance.

### Thread Pool Configuration

The `threadPoolSize` setting controls how many worker threads are available for handling player sync operations concurrently. For servers with many players, you may want to increase this value.

When `useCachedThreadPool` is set to `true`, the mod will use a dynamically sized thread pool that automatically scales based on demand. This is recommended for servers with varying loads or occasional high traffic. The `threadPoolSize` setting is ignored when using a cached thread pool.

### Connection Pool Configuration

The `maxPoolConnections` setting controls how many Redis connections can be maintained in the pool simultaneously. The default of 128 should be sufficient for most servers. If you have a very high number of concurrent players, you might need to increase this value.

## Timeout Configuration

The timeout settings control how long the mod will wait during different phases of the synchronization process:

### `firstTimeConnection` (Default: 2000ms)
How long to wait when a player joins for the first time (no existing data). This can be set lower as we don't expect to find data.

### `regularConnection` (Default: 10000ms)
How long to wait when a returning player joins. This needs to be long enough to fetch data from other servers.

### `serverSwitch` (Default: 15000ms)
How long to wait when a player switches between servers in a network. This typically needs to be longer to ensure data is properly synchronized.

### `existingDataCheck` (Default: 1000ms)
How long to wait when checking if a player's data exists on any server. This is a quick operation that just verifies existence.

## Recommended Settings for Different Scenarios:

### Standard Setup (Default):
```json
{
  "threadPoolSize": 4,
  "useCachedThreadPool": false,
  "timeouts": {
    "firstTimeConnection": 2000,
    "regularConnection": 10000,
    "serverSwitch": 15000,
    "existingDataCheck": 1000
  }
}
```

### Fast Setup (Low Latency Network):
```json
{
  "threadPoolSize": 4,
  "useCachedThreadPool": true,
  "timeouts": {
    "firstTimeConnection": 1000,
    "regularConnection": 5000,
    "serverSwitch": 8000,
    "existingDataCheck": 500
  }
}
```

### High Traffic Setup:
```json
{
  "threadPoolSize": 8,
  "useCachedThreadPool": true,
  "maxPoolConnections": 256,
  "timeouts": {
    "firstTimeConnection": 1000,
    "regularConnection": 5000,
    "serverSwitch": 10000,
    "existingDataCheck": 500
  }
}
```

### Reliable Setup (High Latency Network):
```json
{
  "threadPoolSize": 6,
  "useCachedThreadPool": false,
  "timeouts": {
    "firstTimeConnection": 3000,
    "regularConnection": 15000,
    "serverSwitch": 20000,
    "existingDataCheck": 3000
  }
}
``` 
