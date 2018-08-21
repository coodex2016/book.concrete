# Counter

示例

Pojo
```java
public class Pojo implements Countable {
    private final int value;

    public Pojo(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

Pojo2, 继承自Pojo
```java
public class Pojo2 extends Pojo {
    public Pojo2(int value) {
        super(value);
    }
}
```

```java
@Sync // 同步统计
public class TotalCounter implements Counter<Pojo> {

    private final static Logger log = LoggerFactory.getLogger(TotalCounter.class);


    private AtomicInteger total = new AtomicInteger(0);

    @Override
    public void count(Pojo value) {
        log.debug("{} + {} = {}", total.get(), value.getValue(), total.addAndGet(value.getValue()));
    }
}
```

```java
public class MinMaxCounter implements Counter<Pojo> {
    private final static Logger log = LoggerFactory.getLogger(MinMaxCounter.class);

    private int min = Integer.MAX_VALUE, max = 0;

    @Override
    public void count(Pojo value) {
        int v = value.getValue();
        if (min > v)
            min = v;
        if (max < v)
            max = v;
        log.debug("min: {}, max: {}", min, max);
    }
}
```


```java
public class BoxedCounter implements SegmentedCounter<Pojo> {

    private final static Logger log = LoggerFactory.getLogger(BoxedCounter.class);


    private static final int boxCount = 15;
    private static final int boxSize = 100;
    private int[] boxes = new int[boxCount + 1];

    public BoxedCounter() {
        Arrays.fill(boxes, 0);
    }


    @Override
    public void count(Pojo value) {
        int boxNo = value.getValue() / boxSize;
        boxes[boxNo >= boxCount ? boxCount : boxNo]++;
        log.debug("boxes: {}", boxes);
    }

    public int[] getBoxes() {
        return boxes;
    }

    @Override
    public Segmentation getSegmentation() {
        return new SpecificMomentSegmentation("0/3 * * * * *");
    }

    @Override
    public void slice() {
        log.info("slice ...");
        Arrays.fill(boxes, 0);
    }
}
```

```java
    public static void main(String[] args) throws InterruptedException {

        new ClassPathXmlApplicationContext("counter.xml");

        for (int i = 0; i < 10000; i++) {
            CounterFacade.count(new Pojo2(Common.random(0, 2000)));
        }

        ExecutorsHelper.shutdownAll();

    }
```