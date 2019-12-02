# 轻量级重试机制

在异步任务的场景中，为了保证正确及最终一致性，我们通常需要多次尝试直到任务成功或者尝试次数大于阈值。`org.coodex.util.Retry`基于这种场景提供了一个轻量级的解决方案。

示例：

```java
        Retry.newBuilder()
//                .maxTimes(50) // 最大尝试次数，默认5次
//                .initDelay(20, TimeUnit.SECONDS) // 第一次执行延迟时间，默认为0
//                .next(new Retry.TimeUnitNextDelay(TimeUnit.SECONDS) { // 延迟策略，默认每5秒执行一次
//                    @Override
//                    protected long delay(int times) {
//                        return 5;
//                    }
//                })
//                .scheduler(ExecutorsHelper.newSingleThreadScheduledExecutor("test")) // 指定线程池，为空则使用默认线程池，大小为核心数x2
                .build()
//                .handle(new Retry.AllFailedHandle() { // 当任务尝试数超出最大阈值依然失败时的handle
//                    @Override
//                    public void allFailed(Calendar start, int times) {
//                        log.info("all failed");
//                    }
//                })
                .execute(new Retry.Task() { // 要多次尝试执行的任务
                    @Override
                    public boolean run(int times) throws Exception { // 返回值为真表示执行完成，为假或者抛出异常则视为未能完成
                        log.debug("times: {}", times);
                        if (times % 2 == 0) {
                            throw new RuntimeException("mock exception:" + times);
                        }
                        return times == 5;
                    }
                });
```