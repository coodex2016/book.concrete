# 通用单例模式

## org.coodex.util.Singleton&lt;T>

很多场景下，我们需要用到单例模式，大部分情况会采用以下方式管理单例

```java
private static final A a = new A();
```

或者懒加载模式

```java
private static A a;

private static A get(){
    if(a == null){
        synchronized(A.class){
            if(a ==null){
                a = new A();
            }
        }
    }
    return a;
}
```

`coodex-utilities`简化了懒加载模式

```java
private static final Singleton<A> A_SINGLETON = new Singleton<A>(
    new Singleton.Builder<A>(){
        public A build(){
            return new A();
        }
    }
);
```

当你的工程使用到java8的lambda表达式时，会进一步简化代码

```java
private static final Singleton<A> A_SINGLETON = new Singleton<A>(A::new);
```

使用时

```java
A_SINGLETON.get();
```

当然，这个例子不能说明什么，关键是，通过Singleton.Builder，使用build模式，我们可以在代码中隔离掉构建的具体细节

## org.coodex.util.SingletonMap&lt;K, V>

SingletonMap按照键值对的方式管理单例的实例，每个键的值都维持一个单例。

同样的，也是通过build模式来构建实例，构建时，可以根据键信息进行构建。

SingletonMap也可以用作缓存，构造SingletonMap时，可以传入单例最大声明周期，当单例实例存活超过此周期时，SingletonMap会把它移除，再次需要获取时，会重新build一个。
