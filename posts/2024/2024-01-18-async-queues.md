<!--
.. title: Arq and TaskIQ
.. slug: arq-taskiq
.. date: 2024-01-18 00:00:00
.. tags: python
.. category: 
.. link: 
.. description: 
.. type: text
-->

At work, we recently found ourselves in the market for an asynchronous task queue for python. A traditional task queue like Celery can be said to be "asynchronous" in the sense that your web server can kick a task into the queue and continue processing the web request without waiting for the task to complete. However it is "synchronous" in the sense that the task functions in your queue must be synchronous functions (declared with `def` rather than `async def`). If you want to queue an `async` function, you need an async worker to process it.

The two contenders we've been looking at in this space are [arq](https://arq-docs.helpmanual.io/) and [taskiq](https://taskiq-python.github.io/). These two solutions take slightly different approaches to solving the same problem.

Taskiq takes a conceptually simple push/pop approach to interacting with the queue. This is the same model used by popular synchronous packages like Celery and rq. When a worker is free to take a task, it pops a task off the queue and then executes it. The downside of this approach is that if a worker pops a task off the queue and then shuts down without processing the task to completion, that task is already gone from your queue without having been run to completion. Another worker can't try it again.

Arq takes a different approach called "pessimistic execution" which solves that specific problem. When an arq worker takes a task from the queue, it doesn't remove it from the queue yet. The task stays in the queue while it is being run. The task is finally deleted from the queue in a post hook after the task is complete. This means a task is only removed from the queue after it has run to completion.

In order to ensure every worker in your cluster is not trying to process the same task at once, arq also maintains some additional shared state. When a worker takes a task, the worker acquires a lock on a task. That lock is automatically set to release after a timeout. If a worker never deletes the task in the post hook, the task is eventually unlocked and becomes available for another worker to process at a later time once the timeout expires.

This gives arq some slightly different characteristics than taskiq.

Arq will ideally try to deliver your task exactly once, but guarantees "at least once execution". Executing your task multiple times is considered preferable to executing it zero times. This means no lost tasks, but it also means if you use arq, your tasks must be written to be idempotent.

The simple push/pop relationship with the queue employed by taskiq lends itself to being compatible with a wide range of backends. Taskiq already has plugins for using NATS, Redis, RabbitMQ and Kafka as brokers. Taskiq defines a [plugin interface](https://taskiq-python.github.io/extending-taskiq/broker.html) for brokers, so it would be possible to write plugins for aditional backends like SQS for example.

Conversely, arq has a more complicated relationship with the data store. The additional shared state required to implement the locking behaviour needs a richer set of operations. As such, arq is tightly coupled to redis as a backend. There is no mechanism to substitute another broker.

So here's a summary of those tradeoffs:

- Arq is tied to redis. It provides stronger guarantees about eventual task execution and requires you to write your tasks with the assumption they could be attempted multiple times.
- Taskiq follows a model similar to Celery or rq. It provides weaker assurances, but this conceptually simpler model means you can assume your tasks will only be executed once. This setup also allows for compatibility with a wider range of brokers.

Our project has a lot of long-running tasks, which are vulnerable to being killed off before running to completion by deploy or scale-in events. Because of this, we prefer the pessimistic execution model offered by arq. We ended up moving forward with arq for our project.
