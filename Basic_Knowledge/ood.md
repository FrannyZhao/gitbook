[toc]

## 单例

优点：省内存；避免对资源的多重占用（写文件）；

缺点：一般没接口->不方便扩展+不利于测试；

```java
public class Singleton {
    private static final Singleton singleton = new Singleton();
    private Singleton() {
        
    }
    public static Singleton getSingleton() {
        return singleton;
    }
}
```

