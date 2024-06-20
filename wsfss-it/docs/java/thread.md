## 线程池

某些场景下为了提高接口性能，需要使用多线程来处理请求。为了防止线程被打满，我们需要使用线程池来管理线程。

线程池的大小一般根据CPU的核数来定，但最优配置还需结合任务类型、系统资源以及具体应用场景综合考虑。

```java title="CustomThreadPool.java" linenums="1"
/**
 * 创建一个固定大小的线程池
 */
private static ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
        // 核心线程数
        2,
        // 最大线程数
        2,
        // 空闲线程存活时间（单位：秒）
        60L, 
        TimeUnit.SECONDS,
        // 工作队列，这里使用有界队列
        new ArrayBlockingQueue<>(200),
        // 线程工厂，用于创建新线程
        Executors.defaultThreadFactory(),
        // 拒绝策略，在队列满时如何处理新的任务，默认AbortPolicy会抛出RejectedExecutionException异常，这里使用CallerRunsPolicy让提交的线程自己执行任务
        new ThreadPoolExecutor.CallerRunsPolicy()
);
```

为了获取异步任务的执行结果，可以使用Future接口的get方法，该方法会阻塞当前线程，直到任务执行完成。如果任务执行完成，则返回任务的执行结果，否则抛出异常。

```java title="CustomThreadPool.java" linenums="1"
/**
 * 执行异步任务无返回值
 */
public static CompletableFuture<Void> executeAsync(Runnable task) {
    return CompletableFuture.runAsync(task, threadPool);
}

/**
 * 执行异步任务带返回值
 */
public static<T> CompletableFuture<T> executeSupplyAsync(Supplier<T> task) {
    return CompletableFuture.supplyAsync(task, threadPool);
}
```

