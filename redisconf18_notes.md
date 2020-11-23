# Ryan Luecke, Box, Inc.

## Use case of save user session token (Dual-Tier solution)

1. Save user session on single memcached server is a _SPA_ (Single Point of Failure)
2. Create a secondary tier to save unless in 2 diferent servers
3. If you need a long maintainance window it backs to _SPA_
4. But Redis can be use instead because it has replication embedded:

|----------------------|--------------|-----------------------------------|
| Feature              | Redis        | Memcached                         |
|----------------------|--------------|-----------------------------------|
| Supports Replication | Yes          | No                                |
| Feature set          | Rich         | Simple                            |
| Performance variance | Medium       | Very Low (just _O(1)_ operations) |
| Client complexity    | Medium       | Low                               |
| Persistence          | Configurable | None                              |
|----------------------|--------------|-----------------------------------|

5. With Redis but keeping the same primary and secondary setup, on maintainance of secondary tier we can create (a spare node) replication of the primary tier node and when it's done swap this spare with the secondary tier node
6. Then we can disable the replication to secondary tier new node can receive writes
7. We solve the replication problem but we still with some problems:
  * Not consistent: If failure occurrs on write at one of this tiers.
  * Race conditions: If it has a delete operation, you say first delete from the primary than from the secondary but at mid time if you have a read on primary then on miss goes to secondary and fill the primary we could recreate on primary what we are trying to delete. Use swap can reduce the window for race condition but still possible.
  * Volatile storage: In power outage or cordinated restart you will lose all data.

## Use case Recent Files

Save recent files with most recent interacted files at top and last recent at bottom.

1. Redis has sorted Sets
```
ZADD myzset 2 "two" 3 "three"
ZADD myzset 1 "uno"
ZREVRANGE myzset 0 -1 WITHSCORES
  "three"
  "3"
  "two"
  "2"
  "uno"
  "1"
```
2. But have all the previous Dual-Tier problems
3. Instead can be used Redis Cluster
  * Horizontally scalable
  * Auto data sharding
  * Fault tolerant
  * Decentralized cluster management (gossip)
  * Requires cluster-compatible client libraries
4. 
