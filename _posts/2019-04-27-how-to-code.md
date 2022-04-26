---
layout: post
title:  "如何编码有效代码"
date:   2019-12-07
categories: 规范
---
## 什么是有效的代码
特征按重要度排序
- 无bug
- 结构清晰
- 简洁
- 高性能

## 编码宗旨
- 复用, 测试
- 接口, 组合
- 代码布局
- 空间, 时间, 无状态

## 禁忌
- 禁用util
- 慎用public
- 适当final

## 面向接口编程
- 描述主业务行为, 定义接口类, 异常类
- 内部模块分包，定义资源访问, 请求处理, 结果包装 等接口
- 接口方法不要public
- 关注方法定义, 参数/返回值/异常, 同步/异步
- 做减法!

## 如何设计接口
- 更通用的输入类型
- 异常处理错误

```java
public interface XXX {

    ReturnType handle(File file);

    ReturnType handle(InputStream is) throw XXException;

    Future<ReturnType> handle(InputStream is); // 何时关闭Stream, 谁开谁负责

    Future<ReturnType> handle(Reader reader);
}
```

## 如何使用 / 设计异常
- UncheckedException 输入方引起的异常, 如: IllegalArgument
- CheckedException 系统内部状态导致, 如: FileNotFound, IO

```java
public enum ErrorCode {

    ERR_XXX(n, "message={}");

    private int code;
    private String pattern;

    public String message(Object... options) {
        return options == null || options.length == 0
            ? pattern : MessageFormat.format(pattern, options);
    }

    public CodeException exception(Object... options) {
        return new CodeException(code, message(options));
    }

}
```

```java

public class CodeException extends RuntimeException implements CodeMessage {

    private final int code; // 错误代号
    private Object data;    // 可选上下文

    public CodeException(String message) {
        this(-1, message);
    }
    
    public CodeException(int code, Object... options) {
        super(message);
        this.code = code;
    }
    
    public CodeException(int code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }
    ...
}
```

## 几个问题
- 代码写的越多, 水平越高 ?
- 语言特性越多, 水平越高 ?

## 如何实现接口
- 共有资源, 逻辑提取到父类

```java
abstract class AbstractXXX {  // -public -interface
    protected final Logger logger = LoggerFactory.getLogger(getClass()); // +final

    public ReturnType sharedMethodOfInterface() {
        ...
        methodOfInterface(param);
    }

    public abstract ReturnType methodOfInterface(ParamType param);

    protected ... sharedUtil(...) { // -util
    }
}
```

## 如何实现接口
```java
public class DefaultXXX extends AbstractXXX implements XXX {

    @Override
    public ReturnType methodOfInterface(ParamType param) {
        logger.debug("params={}", param);
        ...
    }
}

// 不要final死，给后人一条活路
public class MyXXX extends DefaultXXX {

    @Override
    protected ... sharedUtil(...) {
    }
}
```

## 如何布局代码
```java
public class XXX {

    private static final ... = ...;
    private final ;

    protected ... = ...;

    public xxx abc(yyy) {
        ...
    }

    private ... xxx(..) {
    }
}
```


## 如何布局代码
```java

    public xxx abc(yyy) {

        Preconditions.checkNotNull(yyy, "");

        try {
            ...
        } catch(XException e) {
            throw new RuntimeException(e);
        } catch(YException e) {
            logger.error("xx", e);
            Monitor.record("XXXFail");
            return error;
        }

        // last line;
    }
```

## ELSE 有罪
```java

    if (name != null {            // if (name == null) {
        jdbc.query("xxxx", name); //     return default_value;
    } else {                      // }
        return default_value;     // return jdbc.query("xxx", name);
    }

    //
    if (count == 1) {             // return count == 1
        return true;
    } else {
        return false;
    }
```

## 空间 / 时间
```java
    Map fixedSize = new HashMap(size);
    Map autoExtend = new LinkedHashMap();
    Map empty = Collections.emptyMap();
    Map constant = Collections.unmodifiableMap(map);
    ConcurrentMap concurrent = new ConcurrentHashMap();

    // List list =
    // Set set =
    concurrent.putIfAbsent()

    // Koloboke @ github
```

## 力荐 guava
```java
final Joiner _ = Joiner.on('_');
final Splitter comma = Splitter.on(',').omitEmptyStrings().trimResults();
String name = _.join("John", "Constantine");
Iterator<String> = comma.split("xxx,yyy");
List<String> = comma.splitToList("xxx,yyy");

Preconditions.checkNotNull();
LoadingCache<Key, Value) cache = CacheBuilder.newBuilder().maxCapacity().build(new CacheLoader() {
    public Value load(Key key) throws {
        // create value;
    }
});
Supplier<T> supplier = Suppliers.memoize(() -> ...)
final HashFunction md5 = Hashing.md5();
String hash = md5.hashString("xx", Charsets.UTF_8).toString()/.asInt();
String path = Optional.fromNullable(map.get("path").or("/tmp");
```

## 日志 (接口)
```java
    // log4j
    log.debug("x=" + x + ", y=" + y);

    if (log.isDebugEnalbe()) { // 避免字符串运算
        StringBuilder info = new StringBuilder("x=").append(x).append(", y=").append(y);
        log.info(into.toString());
    }

    // slf4j
    log.debug("x={}, y={}", x, y);
```

## 日志 (实现)
- jcl-over-slf4j   替换Java Commons Logging
- log4j-over-slf4j 替换Log4j
- slf4j-log4jxx    使用log4j作为slf4j-api输出
- logback-classic  使用logback作为slf4j-api输出

## 线程安全
- SimpleDateFormat / FastDateFormat
- ObjectMapper / JsonMapper

## 技巧 - Builder代替大量构造方法
```java
public class XXXBuilder {

private String name = "defaultName";   // 默认值

    public static XXXBuilder create() {   // 将来修改，比new更容易兼容
        return new XXXBuilder();
    }

    public XXXBuilder setName(String name) { // setXXX 符合Spring等框架约定
        this.name = name;
        return this;
    }

    public XX build() {
        return new XX(name, .., .., .., .., ...);
    }
}
```


## 技巧 - server内部类无get/set
```java
public Config {

    @JsonIgnore
    public Integer id;  // 存储主键, 用于更新性能优化 where id=?

    public String name;
    public String content;
}
```

## 技巧 - Big Service
```java
@Service
public CompositeService extends AbstractService implements A, B, C {

    public Object methodOfA(Object input) {
        jdbc.query("...");
    }

    public Object methodOfB(Object input) {
        cache.get("..");
    }

    ...
}

    @Resource
    A serviceA;

    @Resource
    B serviceB;
```

## 技巧 - 重复检查
```java
String name = (String) map.get("name");
String address = (String) map.get("address");
if (name == null)
    throw xxx;

if (address == null)
    throw yyy;

protected T required(T value, String message) {
    Preconditions.checkNotNull(value);
    return value;
}

String name = required(map.get("name"));
// service.get("name")
// service.mustGet("name")
```

## 技巧 - return this
```java
Map param = new HashMap();
param.put("name", name);
param.put("age", age);

execute(param);

class Param<K, V> extends Map {
    Param fill(K key, V value) {
        put(key, value);
        return this;
    }
}

execute(new Param().fill("name" name).fill("age", age));
```

## 无状态
- 不可变数据
- 会话线程绑定
- CopyOnWriteXXX
- Disruptor
