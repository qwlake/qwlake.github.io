---
layout: post
title: "[SpringBoot] Distributed Lock with Redis"
author: qwlake
categories: Spring
tags: springboot3 redis distributed lock
---
# Before we go

### What is Distributed Lock?

Syncronizing datas or api calls for multiple servers.

### Why Redis for Distributed Lock?

Eastest way in my case. Because of not heavy requirements.

### Why implementing all the lock codes?

You can use Redisson for lock easly instead of implementing all to locking code. But I need more customizing and using Lettuce already.

# Codes

```kotlin
@Service
class DistributeLockService(
    private val commonRedisTemplate: RedisTemplate<String, String>,
) {
    private val valueOps = commonRedisTemplate.opsForValue()
    companion object {
        private const val DefaultLockTimeoutMillis = 30_000L
        private val ReleaseScript: RedisScript<String> = RedisScript.of("some lock release script")
    }

    fun <T> doWait(space: String, lockTimeMillis: Long = DefaultLockTimeoutMillis, block: ()->T): T {
        val lockName = getLockName(space)
        val lockValue = acquireWait(lockName, lockTimeMillis)

        return internalDo(lockName, lockValue, lockTimeMillis, block)
    }

    private fun <T> internalDo(lockName: String, lockValue: String, lockTimeMillis: Long, block: ()->T): T {
        val beginAtNanos = System.nanoTime()
        return runCatching { block() }
            .onSuccess { release(lockName, lockValue, lockTimeMillis, beginAtNanos) }
            .getOrElse {
                release(lockName, lockValue, lockTimeMillis, beginAtNanos)
                throw it
            }
    }

    private fun acquireWait(lockName: String, lockTimeMillis: Long): String {
        val lockValue = "" // something unique string
        // Wait up to lockTimeMillis with trying to set lockValue to Redis.
        // If success to set lockValue, return it. If not throw some kind of Exception.
        return lockValue
    }

    private fun release(lockName: String, lockValue: String, lockTimeMillis: Long, beginAtNanos: Long) {
        val elapsedMillis = TimeUtil.calculateTimeGapMillis(beginAtNanos)
        if(elapsedMillis > lockTimeMillis) {
            return
        }

        runCatching { commonRedisTemplate.execute(ReleaseScript, mutableListOf(lockName), lockValue) }
    }
}
```

# Conclusion

This code call get command to Redis while trying to get lock. So it could be evoke stress to Redis even you did not intend. With Redis pub-sub, it can be less stress for Redis.
