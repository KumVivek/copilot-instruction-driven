Hybrid caching guardrails (L1 + L2):

Cache levels:
- Level 1 (L1): In-memory cache (per instance, fastest)
- Level 2 (L2): Redis (shared, source of truth)
- Database is the final source of truth

General rules:
- Use cache-aside pattern only
- Never cache data without successful DB read/write
- Cache logic must not exist in controllers
- Controllers may only call services

Read flow (GET):
- Check L1 first
- If miss, check L2
- If miss, read from database
- Populate L2 first, then L1
- L1 TTL must always be shorter than L2 TTL

Write flow (CREATE / UPDATE):
- Write to database first
- If DB write succeeds:
  - Update or overwrite L2
  - Invalidate L1 (do not repopulate immediately)
- Never update cache before DB commit

Delete flow (DELETE):
- Delete from database first
- If DB delete succeeds:
  - Remove entry from L2
  - Invalidate L1
- L2 must be cleared before L1 to avoid stale refills

Cache invalidation rules:
- Invalidation must be explicit
- Never rely on natural TTL expiry for correctness
- L1 invalidation must happen on every write or delete
- L2 invalidation is mandatory for deletes

Cache key rules:
- Keys must be deterministic and documented
- Format: <entity>:<identifier>[:<scope>]
- Do not include timestamps, random values, or environment names
- Same logical data must always resolve to the same key

Pattern-based cache flush:
- Pattern-based flush is allowed ONLY for Redis (L2)
- Redis pattern deletes must use SCAN, not KEYS
- Never perform pattern deletes inside request hot paths
- Pattern flushes must be executed:
  - In background jobs, OR
  - In admin / maintenance flows only

In-memory cache flush:
- In-memory cache does not support pattern deletion
- Pattern-based invalidation must be approximated by:
  - Key prefix tracking, OR
  - Full cache clear for the related entity type
- Do not attempt Redis-style patterns in memory cache

Concurrency rules:
- Cache writes must be idempotent
- Multiple concurrent cache misses must not corrupt state
- Prefer last-write-wins strategy
- Do not use distributed locks unless explicitly required

Failure handling:
- Cache failures must not break request flow
- Redis unavailability must fall back to DB
- In-memory cache failures must be ignored
- Never throw errors solely due to cache failure

What is forbidden:
- Caching inside controllers
- Writing to cache before DB success
- Caching nulls unless explicitly stated
- Caching exceptions or error responses
- Using Redis KEYS command
