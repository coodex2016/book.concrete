# 计数器

> #### Note::
>
> 好长时间没动它了，现在看着有点WTF，为啥默认实现在concrete-core里？？为啥用动态字节码？？
>
> org.coodex.count目前在审视中，为啥默认实现在concrete-core里？？Countable还要不要？CountFacade机制还要不要？结果要不要notify机制？等等等等。。。
>
> 跟mock一样，肯定是需要重新设计v2了，手动捂脸

count算是一个极xN轻量级的流式计算模型。

```java
package org.coodex.concrete.demo.boot;

import org.coodex.concrete.spring.SpecificMomentSegmentation;
import org.coodex.concurrent.ExecutorsHelper;
import org.coodex.count.*;
import org.coodex.util.Common;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Arrays;
import java.util.concurrent.atomic.AtomicInteger;

public class CounterDemo {
    private final static Logger log = LoggerFactory.getLogger(CounterDemo.class);

    public static class CountableData implements Countable{
        public CountableData(int value) {
            this.value = value;
        }

        public int value;
    }

    @Sync // 同步计算
    // 累计的计数器
    public static class TotalCounter implements Counter<CountableData>{
        private AtomicInteger total = new AtomicInteger(0);
        @Override
        public void count(CountableData value) {
            log.debug("{} + {} = {}", total.get(), value.value, total.addAndGet(value.value));
        }
    }

    // 最大最小值的计数器
    public static class MinMaxCounter implements Counter<CountableData> {
        private int min = Integer.MAX_VALUE, max = 0;

        @Override
        public void count(CountableData value) {
            int v = value.value;
            if (min > v)
                min = v;
            if (max < v)
                max = v;
            log.debug("min: {}, max: {}", min, max);
        }
    }

    // 数据分区的计数器，支持时序分段统计
    public static class BoxedCounter implements SegmentedCounter<CountableData> {

        private final static Logger log = LoggerFactory.getLogger(BoxedCounter.class);


        private static final int boxCount = 15;
        private static final int boxSize = 100;
        private int[] boxes = new int[boxCount + 1];

        public BoxedCounter() {
            Arrays.fill(boxes, 0);
        }


        @Override
        public void count(CountableData value) {
            int boxNo = value.value / boxSize;
            boxes[Math.min(boxNo, boxCount)]++;
            log.debug("boxes: {}", boxes);
        }

        public int[] getBoxes() {
            return boxes;
        }

        @Override
        public Segmentation getSegmentation() {
            // 每三秒一个分段
            return new SpecificMomentSegmentation("*/3 * * * * *");
        }

        @Override
        public void slice() {
            log.info("slice ...");
            Arrays.fill(boxes, 0);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            CounterFacade.count(new CountableData(Common.random(0, 2000)));
            Thread.sleep(50);
        }

        ExecutorsHelper.shutdownAll();
    }

}
```

运行以后日志太长，我们把三个Counter的部分关键信息摘出来看看:

```txt
11:51:27.014 [main] DEBUG org.coodex.concrete.demo.boot.CounterDemo - 979455 + 334 = 979789
11:51:27.065 [main] DEBUG org.coodex.concrete.demo.boot.CounterDemo - 979789 + 1902 = 981691
```

这个到没什么，主要是看一下线程名，因为是同步计算，所以是`main`

```txt
11:54:44.097 [counter-5] DEBUG org.coodex.concrete.demo.boot.CounterDemo - boxes: [1, 1, 4, 1, 3, 2, 1, 1, 2, 2, 2, 3, 0, 3, 2, 14]
11:54:44.097 [main] DEBUG org.coodex.concrete.demo.boot.CounterDemo - 158976 + 1745 = 160721
11:54:44.097 [counter-6] DEBUG org.coodex.concrete.demo.boot.CounterDemo - min: 11, max: 1985
11:54:44.148 [counter-3] DEBUG org.coodex.concrete.demo.boot.CounterDemo - boxes: [2, 1, 4, 1, 3, 2, 1, 1, 2, 2, 2, 3, 0, 3, 2, 14]
11:54:44.148 [main] DEBUG org.coodex.concrete.demo.boot.CounterDemo - 160721 + 0 = 160721
11:54:44.148 [counter-4] DEBUG org.coodex.concrete.demo.boot.CounterDemo - min: 0, max: 1985
```

这三个一起看吧，第一，BoxedCounter和MinMaxCounter是异步的，使用名为counter的线程池；第二，我们看第三行和最后一样，最小值从11变成0，中间BoxedCounter的0号盒增加了1，total那个加了个0

```txt
11:54:44.962 [counter-8] DEBUG org.coodex.concrete.demo.boot.CounterDemo - boxes: [2, 1, 4, 3, 4, 2, 2, 2, 3, 3, 3, 4, 0, 5, 3, 18]
11:54:44.962 [counter-7] DEBUG org.coodex.concrete.demo.boot.CounterDemo - min: 0, max: 1998
11:54:45.001 [count.scheduler-1] INFO org.coodex.concrete.demo.boot.CounterDemo - slice ...
11:54:45.013 [counter-1] DEBUG org.coodex.concrete.demo.boot.CounterDemo - boxes: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1]
```

每三秒会做一次slice，这种方式下，我们可以保存窗口数据，然后内存清0重开一个窗口进行计算
