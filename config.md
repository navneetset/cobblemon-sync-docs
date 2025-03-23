# CobblemonSync Configuration & Optimization Guide

This guide provides recommendations for optimizing CobblemonSync on high-traffic servers where many players may be joining or switching servers simultaneously.

## Configuration Options

The following configuration options in `config/cobblemon-sync/redis.json` can significantly impact performance:

```json
{
  "url": "redis://:password@localhost:6379",
  "enableDebugLogs": false,
  "threadPoolSize": 8,
  "maxPoolConnections": 128,
  "useCachedThreadPool": true,
  "timeouts": {
    "firstTimeConnection": 1000,
    "regularConnection": 5000,
    "serverSwitch": 10000,
    "existingDataCheck": 500
  }
}
```

### Key Options for High-Traffic Servers

1. **Thread Pool Configuration**
   - `threadPoolSize`: For high-traffic servers, increase this to 8-16 depending on CPU cores
   - `useCachedThreadPool`: Set to `true` for automatic scaling of threads based on demand

2. **Redis Connection Pool**
   - `maxPoolConnections`: Default 128 is sufficient for most servers, increase to 256 for very high traffic

3. **Optimized Timeouts**
   - `timeouts.firstTimeConnection`: 1000ms (1 second) is usually sufficient 
   - `timeouts.regularConnection`: For high-traffic environments, 5000ms (5 seconds) balances performance and reliability
   - `timeouts.serverSwitch`: 10000ms (10 seconds) for reliable server switching
   - `timeouts.existingDataCheck`: 500ms is usually fast enough for a response

## Server Hardware Recommendations

1. **Redis Server**
   - Ensure Redis is running on a dedicated server or container
   - Minimum 4GB RAM for Redis
   - Fast disk I/O (preferably SSD)
   - Low latency network connection between Minecraft and Redis servers

## Monitoring

Keep an eye on these metrics:

1. Watch the Minecraft server logs for messages about sync timeouts
2. Monitor Redis memory usage and connection count
3. Track CPU usage on both Minecraft and Redis servers
4. Monitor network latency between servers

## Troubleshooting High-Traffic Issues

1. **Sync Timeouts**
   - If players are experiencing timeout errors during login, increase the `timeouts.regularConnection` value
   - For server switching timeouts, increase `timeouts.serverSwitch`

2. **Minecraft Server Performance**
   - If the Minecraft server becomes laggy during mass player joins, increase `threadPoolSize`
   - Enable `useCachedThreadPool` to dynamically scale thread count

3. **CPU Bottlenecks**
   - If CPU usage is consistently high, consider splitting player load across more server instances 
