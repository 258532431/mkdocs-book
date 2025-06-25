---
comments: true
---

## 自定义线程池

直接使用 `new Thread()` 创建线程，线程数量过多会占用大量内存，导致JVM内存溢出。通过自定义线程池，可以解决这个问题。

=== "创建一个固定大小的线程池"

```java
private static final int corePoolSize = 2;
private static final int maxPoolSize = corePoolSize * 2;

/**
 * 创建一个固定大小的线程池
 */
private static ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
        // 核心线程数
        corePoolSize,
        // 最大线程数
        maxPoolSize,
        // 空闲线程存活时间（单位：秒）
        60L, 
        TimeUnit.SECONDS,
        // 工作队列，这里使用有界队列
        new ArrayBlockingQueue<>(200),
        // 线程工厂，用于创建新线程
        Executors.defaultThreadFactory(),
        // 拒绝策略，在队列满时如何处理新的任务，默认AbortPolicy会抛出RejectedExecutionException异常，这里使用CallerRunsPolicy让提交的线程自己执行任务
        new ThreadPoolExecutor.CallerRunsPolicy() {
            @Override
            public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
                log.keyword("CustomThreadPool", "线程池被打满").info("异步任务被拒绝，由请求线程自己执行， Runnable: " + r.toString() + ", 当前线程池状态: " + e.toString());
                super.rejectedExecution(r, e);
            }
        }
);
```

## 优雅停机

线程池的 `shutdown()` 方法会等待所有任务执行完毕，然后关闭线程池。但是，如果任务执行时间过长，或者任务数量过多，可能会导致停机时间过长。

=== "优雅停机"

```java
/**
 * 关闭线程池
 */
public static void shutdown() {
    if (!threadPool.isShutdown()) {
        log.keyword("CustomThreadPool").info("尝试关闭线程池");
        // 有序关闭线程池，继续执行已提交的任务，不再接受新任务
        threadPool.shutdown();
        try {
            // 等待所有任务完成或超时，再关闭线程池
            if (!threadPool.awaitTermination(180, TimeUnit.SECONDS)) {
                log.keyword("CustomThreadPool").info("线程池未在规定时间180秒内关闭");
            }
        } catch (InterruptedException ie) {
            log.keyword("CustomThreadPool").error("尝试关闭线程池异常 e:{}", ie);
            log.keyword("CustomThreadPool").info("尝试强制关闭线程池");
            // 尝试停止所有正在执行的任务，停止对等待任务的处理
            threadPool.shutdownNow();
            // 标记中断线程
            Thread.currentThread().interrupt();
        } finally {
            if (!threadPool.isTerminated()) {
                log.keyword("CustomThreadPool").info("线程池未在规定时间内关闭，尝试强制关闭线程池");
                threadPool.shutdownNow();
            }
        }
    }
}
```

## 异步任务

`CompletableFuture` 结合自定义线程池实现异步任务处理。

=== "异步任务"

```java
/**
 * 执行异步任务
 */
public static CompletableFuture<Void> executeAsync(Runnable task) {
    log.keyword("CustomThreadPool").info("提交异步任务: " + task.toString());
    String crid = Logtube.getProcessor().getCrid();
    return CompletableFuture.runAsync(() -> {
        Logtube.getProcessor().setCrid(crid);
        task.run();
    }, threadPool);
}

/**
 * 执行异步任务带返回值
 */
public static<T> CompletableFuture<T> executeSupplyAsync(Supplier<T> task) {
    log.keyword("CustomThreadPool").info("提交带返回值异步任务: " + task.toString());
    String crid = Logtube.getProcessor().getCrid();
    return CompletableFuture.supplyAsync(() -> {
        Logtube.getProcessor().setCrid(crid);
        return task.get();
    }, threadPool);
}
```