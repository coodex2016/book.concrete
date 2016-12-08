# AbstractErrorCodes

用于定义的错误号，所有错误编号应继承自该类型。错误将如何表现请参见[@ErrorMsg](ErrorMsg.md)。

## 实践方案

### 按模块划分

```java
public class ProjectErrorCodes extends AbstractErrorCodes{
    //非具体错误号不要用public
    protected static final int MODULE1 = CUSTOM_LOWER_BOUND;
    
    protected static final int MODULE2 = MODULE1 + 5000; 
    
    // ....
}


public class Module1ErrorCodes extends ProjectErrorCodes{
    
    public static final int ERROR1 = MODULE1 + 1;
    
    //....
}

// ....

```

### 团队协作时按人划分

```java
public class ProjectErrorCodes extends AbstractErrorCodes{
    
    //非具体错误号不要用public
    protected static final int 西门吹雪 = CUSTOM_LOWER_BOUND;
    
    protected static final int 陆小凤 = 西门吹雪 + 5000; 
    
    // ....
}


public class 西门吹雪ErrorCodes extends ProjectErrorCodes{
    
    public static final int 挂点 = 西门吹雪 + 1;
    
    //....
}

// ....

```

好吧，实际上怎么划分无所谓，只要**不重复**，用起来方便，其他看心情就好。