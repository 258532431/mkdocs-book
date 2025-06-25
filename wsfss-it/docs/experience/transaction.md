---
comments: true
---

## 场景描述

在复杂的业务流程中，往往涉及到异步数据操作。比如创建计划后，需将计划任务推送给大量下游用户，通常使用 `RocketMQ` 或者 `Kafka` 来实现异步数据操作。如果创建计划的数据写入数据库后，还未持久化完成，由于 `MySQL` 默认事务隔离级别为 `REPEATABLE_READ`，则可能导致异步操作查询不到该计划。

## 解决方案

### 修改数据库事务隔离级别

 发送MQ事务消息，并修改异步操作的数据库事务隔离级别为 `READ_COMMITTED`。如果业务场景未实现MQ事务消息，可以通过Spring事务管理器实现：

=== "Spring事务管理器"

    ```java
    TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronizationAdapter() {
        @Override
        public void afterCommit() {
            // 事务提交成功后进行异步操作
        }
    });
    ```

### Spring事务管理器注意事项

非事务方法A中没有CRUD操作，方法A中调用了事务方法B，本身没什么问题。但是当事务方法B中进行了事务监听时，需要保证整个方法调用链全部都是事务方法，这时需要给方法A也加上事务注解。
