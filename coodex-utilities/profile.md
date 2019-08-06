# Profile

profile是一个key-value的模型。

## 主要接口

- getInt(String)/ getInt(String, int)

    从profile中获取指定key的整型值

- getBool(String)/ getBool(String, boolean)

    从profile中获取指定key的布尔值

- getLong(String)/ getLong(String, long)

    从profile中获取指定key的长整型值

- getString(String)/ getString(String, String)

    从profile中获取指定key的字符串值

- getStrList(String)/ getStrList(String, String)/ getStrList(String, String, String[])

    从profile中获取指定key的字符串数组

## usage

```java
    Profile profile = Profile.get("profileName");
```

profile默认支持.properties，如果你的工程中引入了snakeyaml，则profile支持.yml|yaml文件

## 扩展

`Profile`支持扩展你自己的键值对格式资源，实现一个`org.coodex.util.ProfileProvider`放到SPI即可。
