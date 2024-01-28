---
layout: post
title: "[SpringBoot] Redis Message Queue"
author: qwlake
categories: Spring
tags: springboot3 redis message-queue
---

# Why Redis for Message Queue?

In my case already using Redis. Useally it comes with Kafka but our team does not using that yet. And buisness requirements are not heavy. So We decided to use Redis for simple Message Queue.

# How works?

Basically it takes like bellow structur. Publish messages to queue with List format(LPUSH). Consume messages from queue with RPOP.

![redis-message-queue](/assets/redis-message-queue.png)

# Codes

### Publisher

Here is simple publishing code.

```kotlin
@Service
class PushService(
    private val redisTemplateService: RedisTemplateService,
    private val objectMapper: ObjectMapper,
) {
    fun publishData(data: DataDto) {
        val serializedData = objectMapper.writeValueAsString(data)
        redisTemplateService.lPush("some_key", serializedData)
    }
}
```

### Consumer

And here is subscribing code. With data processing step, it takes lock using redis for prevent race condition when same memberId consumed.

```kotlin
@Component
class EventSubscriber(
    private val redisSubscriber: RedisTemplateService,
    private val eventProcessService: EventProcessService,
    private val distributeLockService: DistributeLockService,
) {
    companion object {
        private const val LockTimeMillis = 1_000L * 5
        private const val LockWaitTimeMillis = 1_000L * 15
    }

    fun messageSub() {
        val popStr = redisSubscriber.bRPop("some_key", 1, TimeUnit.SECONDS)
        if (popStr == null) {
            Thread.sleep(10_000)
            return
        }

        /**
         * do with pop data
         */

        distributeLockService.doWait("some-data:${memberId}", LockTimeMillis, LockWaitTimeMillis) {
            eventProcessService.processData(memberDto.apply {
                this.memberId = memberId
            })
        }
    }
}
```

### Consumer executing

And we need to execute above consumer continuously with no delay. So here are executing codes.

```kotlin
@Configuration
class ExecutorServiceConfig {

    @Value("\\${schedule.thread-number}")
    private val threadCount = 0

    @Bean
    fun subscribeExecutorService(
        subscribeTaskRunnable: SubscribeTaskRunnable,
        subscribeExecutorManager: SubscribeExecutorManager,
    ): ScheduledExecutorService {
        return subscribeExecutorManager.getSubscribeExecutorService(subscribeTaskRunnable, threadCount)
    }
}
```

```kotlin
@Component
class SubscribeTaskRunnable(
    private val eventSubscriber: EventSubscriber,
): TaskRunnableInterface {
// TaskRunnableInterface: custom interface

    companion object {
        @Volatile
        private var taskAlive = true
        private val logger = LoggerFactory.getLogger(SubscribeTaskRunnable::class.java)
    }

    override fun run() {
        while (taskAlive) {
            kotlin.runCatching {
                eventSubscriber.messageSub()
            }.onFailure {
                logger.error(it.message, it)
                Thread.sleep(1_000L)
            }
        }
    }

    override fun destroy() {
        logger.info("Shutting down ${this::class.simpleName}, thread: ${Thread.currentThread()}, hashcode: ${this.hashCode()}")
        taskAlive = false
    }
}
```

```kotlin
@Component
class SubscribeExecutorManager {

    companion object {
        @Volatile
        private var executorService: ScheduledExecutorService? = null
        private val logger = LoggerFactory.getLogger(SubscribeExecutorManager::class.java)
        private val tasks = CopyOnWriteArrayList<TaskRunnableInterface>()
    }

    fun getSubscribeExecutorService(task: TaskRunnableInterface, threadCount: Int): ScheduledExecutorService {
        if (executorService == null) {
            synchronized(this) {
                if (executorService == null) {
                    executorService = Executors.newScheduledThreadPool(threadCount).apply {
                        for (i in 0 until threadCount) {
                            submit(task)
                            tasks.add(task)
                        }
                    }
                }
            }
        }
        return executorService!!
    }

    fun destroyExecutorService() {
        logger.info("Start Shutting down ${this::class.simpleName}")
        try {
            tasks.forEach { it.destroy() }

            logger.info("Shutting down ${this::class.simpleName} - ScheduledExecutorService")
            executorService!!.shutdown()

            if (!executorService!!.awaitTermination(60, TimeUnit.SECONDS)) {
                logger.warn("ExecutorService did not terminate in the specified time.")
                val droppedTasks = executorService!!.shutdownNow()
                logger.warn("ExecutorService was abruptly shut down. $droppedTasks tasks will not be executed.")
            }
        } catch (e: InterruptedException) {
            Thread.currentThread().interrupt()
            executorService!!.shutdownNow()
        }
    }
}
```

You have to call function `destroyExecutorService()` . Because it shutdown all runnable tasks gracefully.

# Conclusion

It was hard to setting up consumer executing codes. And consumer graceful shutdown was a little tricky. Be aware of before program shutdown, all tasks are done well.
