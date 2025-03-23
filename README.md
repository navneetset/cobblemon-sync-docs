# Cobblemon Sync Flow

This diagram explains how the login synchronization process works in Cobblemon Sync.

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Login Synchronization Flow                      │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────┐     ┌───────────────┐     ┌──────────────────┐
│  Client      │     │ Login Handler │     │ Redis PubSub     │
└──────┬───────┘     └───────┬───────┘     └────────┬─────────┘
       │                     │                      │
       │    Connect          │                      │
       │─────────────────────>                      │
       │                     │                      │
       │                     │ Get Player UUID      │
       │                     │─────────┐            │
       │                     │         │            │
       │                     │<────────┘            │
       │                     │                      │
       │                     │ playerJoining:UUID   │
       │                     │─────────────────────>│
       │                     │                      │ Publish to all servers
       │                     │                      │────────┐
       │                     │                      │        │
       │                     │                      │<───────┘
       │                     │                      │
       │                     │  Wait for sync       │
       │                     │  completion          │
       │                     │  (LoginSynchronizer) │
       │                     │─────────┐            │
       │                     │         │            │
       │                     │<────────┘            │
       │                     │                      │
       │                     │                      │ syncComplete:UUID
       │                     │                      │<───────────────────────┐
       │                     │                      │                        │
       │                     │ Check if still in    │                        │
       │                     │ transit              │                        │
       │                     │─────────┐            │                        │
       │                     │         │            │                        │
       │                     │<────────┘            │                        │
       │                     │                      │                        │
       │     Login accepted  │                      │                        │
       │<────────────────────│                      │                        │
       │                     │                      │                        │
┌──────┴───────┐     ┌───────┴───────┐     ┌────────┴─────────┐     ┌────────┴───────┐
│  Client      │     │ Login Handler │     │ Redis PubSub     │     │ Other Servers  │
└──────────────┘     └───────────────┘     └──────────────────┘     └────────────────┘
```

## Detailed Explanation

1. **Client Connection**:
   - Player connects to the server
   - ServerLoginConnectionEvents.QUERY_START captures the connection

2. **Redis Synchronization**:
   - Player UUID is obtained from their profile
   - The UUID is added to the playerLockMap to mark it as syncing
   - A "playerJoining:UUID" message is published to Redis

3. **Login Blocking**:
   - The LoginSynchronizer blocks the login process
   - The server periodically checks if the player is still marked as in transit in Redis
   - If Redis confirms sync is complete or timeout is reached, login proceeds

4. **Cache Management**:
   - On successful login, the Cobblemon player cache is cleared
   - A "syncComplete:UUID" message is published to notify other servers

5. **Handling by Other Servers**:
   - Other servers process the "playerJoining" message
   - They can respond by sending relevant player data through Redis
   - When done, they can send a "syncComplete:UUID" message

This approach ensures that players only connect after their data is synchronized, preventing data inconsistencies across servers. 
