##Visolink开发手册 ##

###  环境说明

* * *

| 中间件 |  版本   |                            备注                             |
| :----: | :-----: | :---------------------------------------------------------: |
|  JDK   |   1.8   |       强制要求,1.8以上版本请自行添加Java EE相关jar包        |
| MySQL  | 5.7.8 + |          强制要求,至少5.6!或者5.7当然8.0也没有问题          |
| Redis  |  3.2 +  | windows版只能使用Redis3.2,类Unix系统使用最新的5.0也没有关系 |

### JDK 说明

请使用mvn -v 命令查看关联的jdk版本，当开发环境存在多个版本jdk时候特别注意

```
mvn -v 
```

![image-20190827151011590](http://feicuibabatest.qiniudn.com/image-20190827151011590.png)



# 环境配置

### 1、配置修改

- Redis密码配置

```java
redis:
# Redis数据库索引（默认为0）
  database: 1
  lettuce:
    pool:
# 连接池最大连接数（使用负值表示没有限制）
      max-active: 500
# 连接池中的最大空闲连接
      max-idle: 10
# 连接池最大阻塞等待时间（使用负值表示没有限制）
      max-wait: -1
# 连接池中的最小空闲连接
      min-idle: 0
#	密码
  password: K(878ihdkjfiiijfieiyyiyi1
# 连接超时时间（毫秒）
  timeout: 5000
  sentinel:
#	多个节点逗号隔开
    nodes: 10.129.36.24:26379,10.129.36.25:26379,10.129.36.26:26379 
#	节点主分支
    master: mymaster
```

- 数据库密码配置,修改以下几个文件

```
//开发环境配置
visolink-sales-api/src/main/resources/config/application-dev.yml
//生产环境配置
visolink-sales-api/src/main/resources/config/application-prod.yml
```

- 数据源,修改账号和密码

```java
url: jdbc:mysql://10.129.37.52:3306/xuke?serverTimezone=Asia/Shanghai&characterEncoding=utf8&useSSL=false&useCursorFetch=true&defaultFetchSize=500
username: xuke
password: XUke#1234
```

### 2、启动

- 数据库密码配置,修改以下几个文件，不同的开发环境配置文件中配置对应的参数
```
//开发环境请启动dev
visolink-sales-api/src/main/resources/config/application-dev.yml
//部署服务器环境请启动prod
visolink-sales-api/src/main/resources/config/application-prod.yml
```


#开发契约

### 1、Lombok使用及其技巧说明

####  # 为什么使用lombok

还在编写无聊枯燥又难以维护的POJO吗？ 洁癖者的春天在哪里？请看Lombok！在过往的Java项目中，充斥着太多不友好的代码：POJO的getter/setter/toString；异常处理；I/O流的关闭操作等等，这些样板代码既没有技术含量，又影响着代码的美观，Lombok应运而生。首先说明一下：任何技术的出现都是为了解决某一类问题的，如果在此基础上再建立奇技淫巧，不如回归Java本身。应该保持合理使用而不滥用。

#### # 如何安装

当前你使用的ide未安装lombok. lombok能够达到的效果就是在源码中不需要写一些通用的方法，但是在编译生成的字节码文件中会帮我们生成这些方法,减少代码冗余.
[IDEA安装方法](https://blog.csdn.net/zhglance/article/details/54931430) |[eclipse安装方法](https://blog.csdn.net/arkblue/article/details/52608213)

#### # 开发中的使用例子，不讲常见的、技巧性的

- @AllArgsConstructor 替代@Autowired构造注入,多个bean 注入时更加清晰L

   ```java
   @Slf4j
   @Configuration
   @AllArgsConstructor
   public class RouterFunctionConfiguration {
      private final HystrixFallbackHandler hystrixFallbackHandler;
      private final ImageCodeHandler imageCodeHandler;
      
   }
   @Slf4j
   @Configuration
   public class RouterFunctionConfiguration {
      @Autowired
      private  HystrixFallbackHandler hystrixFallbackHandler;
      @Autowired
      private  ImageCodeHandler imageCodeHandler;
   }
   ```



- @SneakyThrows
```
@SneakyThrows
private void checkCode(ServerHttpRequest request) {
   String code = request.getQueryParams().getFirst("code");

   if (StrUtil.isBlank(code)) {
   	throw new ValidateCodeException("验证码不能为空");
   }

   redisTemplate.delete(key);
}

// 不使用就要加这个抛出
private void checkCode(ServerHttpRequest request) throws ValidateCodeException {
   String code = request.getQueryParams().getFirst("code");

   if (StrUtil.isBlank(code)) {
   	throw new ValidateCodeException("验证码不能为空");
   }
}
```

- @UtilityClass 工具类再也不用定义static的方法了，直接就可以Class.Method 使用
```
@UtilityClass
public class Utility {

    public String getName() {
        return "name";
    }
}

public static void main(String[] args) {
    System.out.println(Utility.getName());
}

```

- @CleanUp: 清理流对象,不用手动去关闭流，多么优雅

```
@Cleanup
OutputStream outStream = new FileOutputStream(new File("text.txt"));
@Cleanup
InputStream inStream = new FileInputStream(new File("text2.txt"));
byte[] b = new byte[65536];
while (true) {
   int r = inStream.read(b);
   if (r == -1) break;
   outStream.write(b, 0, r); 
}
```


#### # 总结

Lombok 常用的注解就那么几个，@Data 、@Getter/Setter ，使用例子中的几个可以让代码的更加优雅，建议在你的工程中使用



### 2、JAVA8的汇总资料

|  课时数  | 课时标题                                                     | 在线播放                                                     | 源码位置                                                     |
| :------: | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 第 1 课  | [课程介绍](https://github.com/biezhi/learn-java8)            | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051513399&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_1.html#page=1) ¦ [Youtube](https://youtu.be/A733pQxiEDk) | 无                                                           |
| 第 2 课  | [Java 8 的发展](https://github.com/biezhi/learn-java8/blob/master/java8-growing/README.md) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051508577&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_2.html#page=2) ¦ [Youtube](https://youtu.be/fHhgm1AZzhs) | [java8-growing](https://github.com/biezhi/learn-java8/tree/master/java8-growing/src/main/java/io/github/biezhi/java8/growing) |
| 第 3 课  | [理解 lambda](https://github.com/biezhi/learn-java8/blob/master/java8-lambda/README.md) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051516241&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_3.html#page=3) ¦ [Youtube](https://youtu.be/VkdMeFEGDH8) | [lambda1](https://github.com/biezhi/learn-java8/tree/master/java8-lambda/src/main/java/io/github/biezhi/java8/lambda/lesson1) |
| 第 4 课  | [初尝 lambda](https://github.com/biezhi/learn-java8/blob/master/java8-lambda/README.md) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051511463&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_4.html#page=4) ¦ [Youtube](https://youtu.be/X7Zv5vygjTc) | [lambda2](https://github.com/biezhi/learn-java8/tree/master/java8-lambda/src/main/java/io/github/biezhi/java8/lambda/lesson2) |
| 第 5 课  | [lambda 进阶](https://github.com/biezhi/learn-java8/blob/master/java8-lambda/README.md) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051518174&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_5.html#page=5) ¦ [Youtube](https://youtu.be/3G83it4IASc) | [lambda3](https://github.com/biezhi/learn-java8/tree/master/java8-lambda/src/main/java/io/github/biezhi/java8/lambda/lesson3) |
| 第 6 课  | [默认方法的妙用](https://github.com/biezhi/learn-java8/blob/master/java8-default-methods/README.md) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051518175&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_6.html#page=6) ¦ [Youtube](https://youtu.be/sAuEnkWezDM) | [default-method](https://github.com/biezhi/learn-java8/tree/master/java8-default-methods/src/main/java/io/github/biezhi/java8/defaultmethods) |
| 第 7 课  | [干掉空指针之 Optional](https://github.com/biezhi/learn-java8/blob/master/java8-optional/README.md) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051511464&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_7.html#page=7) ¦ [Youtube](https://youtu.be/br4kqCXPB9A) | [optional](https://github.com/biezhi/learn-java8/tree/master/java8-default-methods/src/main/java/io/github/biezhi/java8/optional) |
| 第 8 课  | [理解 Stream](https://github.com/biezhi/learn-java8/blob/master/java8-stream/README.md) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051555343&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_8.html#page=8) ¦ [Youtube](https://youtu.be/NB9mGlNMl-w) | [stream](https://github.com/biezhi/learn-java8/tree/master/java8-stream/src/main/java/io/github/biezhi/java8/stream/lesson1) |
| 第 9 课  | [Stream API（上）](https://github.com/biezhi/learn-java8/blob/master/java8-stream/README.md#使用流) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051566020&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_9.html#page=9) ¦ [Youtube](https://youtu.be/mGwFJERNzmY) | [stream](https://github.com/biezhi/learn-java8/blob/master/java8-stream/src/main/java/io/github/biezhi/java8/stream/lesson2) |
| 第 10 课 | [Stream API（下）](https://github.com/biezhi/learn-java8/blob/master/java8-stream/README.md#collector-收集) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051571684&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_10.html#page=10) ¦ [Youtube](https://youtu.be/iubE0ezu-xI) | [stream](https://github.com/biezhi/learn-java8/tree/master/java8-stream/src/main/java/io/github/biezhi/java8/stream/lesson3) |
| 第 11 课 | [新的日期时间 API](https://github.com/biezhi/learn-java8/blob/master/java8-datetime-api/README.md) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051571688&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_11.html#page=11) ¦ [Youtube](https://youtu.be/hKXJvh-id1E) | [datetime](https://github.com/biezhi/learn-java8/tree/master/java8-datetime-api/src/main/java/io/github/biezhi/datetime) |
| 第 12 课 | [并发增强](https://github.com/biezhi/learn-java8/blob/master/java8-concurrent/README.md) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051682806&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_12.html#page=12) ¦ [Youtube](https://youtu.be/OYkToWIDEEI) | [concurrent](https://github.com/biezhi/learn-java8/tree/master/java8-concurrent/src/main/java/io/github/biezhi/java8/concurrent) |
| 第 13 课 | [CompletableFuture](https://github.com/biezhi/learn-java8/blob/master/java8-completablefuture/README.md) | [网易云课堂](http://study.163.com/course/courseLearn.htm?courseId=1005047049&utm_campaign=commission&utm_source=cp-400000000397038&utm_medium=share#/learn/video?lessonId=1051908792&courseId=1005047049) ¦ [哔哩哔哩](https://www.bilibili.com/video/av19287893/index_13.html#page=13) ¦ [Youtube](https://youtu.be/4reRygD1dGo) | [completablefuture](https://github.com/biezhi/learn-java8/tree/master/java8-completablefuture/src/main/java/io/github/biezhi/java8/completablefuture) |

### 3、统一工具类使用说明

#### # 统一工具类的意义

Hutool帮助我们简化每一行代码，减少每一个方法，然代码可读性、容错性更高。完整文档方便使用 [hutool-doc](https://hutool.cn/docs),避免每个开发乱引入造成辣鸡代码。

强制使用hutool工具类


#### # hutool 提供类哪些功能
一个Java基础工具类，对文件、流、加密解密、转码、正则、线程、XML等JDK方法进行封装，组成各种Util工具类，同时提供以下组件：


- hutool-aop JDK动态代理封装，提供非IOC下的切面支持

- hutool-bloomFilter 布隆过滤，提供一些Hash算法的布隆过滤

- hutool-cache 缓存

- hutool-core 核心，包括Bean操作、日期、各种Util等

- hutool-cron 定时任务模块，提供类Crontab表达式的定时任务

- hutool-crypto 加密解密模块

- hutool-db JDBC封装后的数据操作，基于ActiveRecord思想

- hutool-dfa 基于DFA模型的多关键字查找

- hutool-extra 扩展模块，对第三方封装（模板引擎、邮件、Servlet、二维码等）

- hutool-http 基于HttpUrlConnection的Http客户端封装

- hutool-log 自动识别日志实现的日志门面

- hutool-script 脚本执行封装，例如Javascript

- hutool-setting 功能更强大的Setting配置文件和Properties封装

- hutool-system 系统参数调用封装（JVM信息等）

- hutool-json JSON实现

- hutool-captcha 图片验证码实现

- hutool-poi 针对POI中Excel的封装

可以根据需求对每个模块单独引入，也可以通过引入hutool-all方式引入所有模块。

#### # 项目中的使用

业务模块使用 visolink-sales-api的时候会自动引入 visolink-common模块，引入最新的hutool-all 工具类

```
<!--hutool-->
<dependency>
	<groupId>cn.hutool</groupId>
	<artifactId>hutool-all</artifactId>
	<version>${hutool.version}</version>
</dependency>
```

#### # 使用例子

比如记录日志时候，从request获取参数的字符串

```
HttpUtil.toParams(request.getParameterMap())
```

获取当前时间

```
//当前时间，格式 yyyy-MM-dd HH:mm:ss
DateUtil.now()
```

#### # 个人建议

在项目中尽量避免自己造轮子封装工具类，让代码可读性更高，所以我建议各位工程师在实际开发过程中一定要注意工具类这个问题。

### 4、lambda使用及其常用技巧汇总

#### # Mybatis Plus 3 的语法糖演示	
```
@Override
@CacheEvict(value = "menu_details", allEntries = true)
@Transactional(rollbackFor = Exception.class)
public Boolean removeRoleById(Integer id) {
   sysRoleMenuMapper.delete(Wrappers
      .<SysRoleMenu>update().lambda()
      .eq(SysRoleMenu::getRoleId, id));
   return this.removeById(id);
}
```
#### # 命令式和函数式	

**命令式编程**：命令“机器”如何去做事情(how)，这样不管你想要的是什么(what)，它都会按照你的命令实现。 

**声明式编程**：告诉“机器”你想要的是什么(what)，让机器想出如何去做(how)。


#### # 什么是函数式编程？

每个人对函数式编程的理解不尽相同。 我的理解是：**在完成一个编程任务时，通过使用不可变的值或函数，对他们进行处理，然后得到另一个值的过程。** 不同的语言社区往往对各自语言中的特性孤芳自赏。现在谈 Java 程序员如何定义函数式编程还为时尚早，但是，这根本不重要！ 我们关心的是如何写出好代码，而不是符合函数式编程风格的代码。


#### # 行为参数化

把算法的策略（行为）作为一个参数传递给函数。

#### # lambda 管中窥豹

- 匿名：它不像普通的方法那样有一个明确的名称：写得少而想得多！
- 函数：Lambda函数不像方法那样属于某个特定的类。但和方法一样，Lambda有参数列表、函数主体、返回类型，还可能有可以抛出的异常列表。
- 传递：Lambda表达式可以作为参数传递给方法或存储在变量中。
- 简洁：无需像匿名类那样写很多模板代码。

#### # 函数描述符

函数式接口的抽象方法的签名基本上就是Lambda表达式的签名，这种抽象方法叫作函数描述符。

#### # 函数式接口，类型推断

函数式接口定义且只定义了一个抽象方法，因为抽象方法的签名可以描述Lambda表达式的签名。 函数式接口的抽象方法的签名称为函数描述符。 所以为了应用不同的Lambda表达式，你需要一套能够描述常见函数描述符的函数式接口。

#### # Java 8中的常用函数式接口

|     函数式接口      | 函数描述符       | 原始类型特化                                                 |
| :-----------------: | :--------------- | :----------------------------------------------------------- |
|   `Predicate<T>`    | `T->boolean`     | `IntPredicate,LongPredicate, DoublePredicate`                |
|    `Consumer<T>`    | `T->void`        | `IntConsumer,LongConsumer, DoubleConsumer`                   |
|   `Function<T,R>`   | `T->R`           | `IntFunction<R>, IntToDoubleFunction,` `IntToLongFunction, LongFunction<R>,` `LongToDoubleFunction, LongToIntFunction,` `DoubleFunction<R>, ToIntFunction<T>,` `ToDoubleFunction<T>, ToLongFunction<T>` |
|    `Supplier<T>`    | `()->T`          | `BooleanSupplier,IntSupplier, LongSupplier, DoubleSupplier`  |
| `UnaryOperator<T>`  | `T->T`           | `IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator`   |
| `BinaryOperator<T>` | `(T,T)->T`       | `IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator` |
| `BiPredicate<L,R>`  | `(L,R)->boolean` |                                                              |
|  `BiConsumer<T,U>`  | `(T,U)->void`    | `ObjIntConsumer<T>, ObjLongConsumer<T>, ObjDoubleConsumer<T>` |
| `BiFunction<T,U,R>` | `(T,U)->R`       | `ToIntBiFunction<T,U>, ToLongBiFunction<T,U>, ToDoubleBiFunction<T,U>` |

#### # Lambdas及函数式接口的例子

|       使用案例        | Lambda 的例子                                                | 对应的函数式接口                                             |
| :-------------------: | :----------------------------------------------------------- | :----------------------------------------------------------- |
|      布尔表达式       | `(List<String> list) -> list.isEmpty()`                      | `Predicate<List<String>>`                                    |
|       创建对象        | `() -> new Project()`                                        | `Supplier<Project>`                                          |
|     消费一个对象      | `(Project p) -> System.out.println(p.getStars())`            | `Consumer<Project>`                                          |
| 从一个对象中选择/提取 | `(int a, int b) -> a * b`                                    | `IntBinaryOperator`                                          |
|     比较两个对象      | `(Project p1, Project p2) -> p1.getStars().compareTo(p2.getStars())` | `Comparator<Project> 或 BiFunction<Project,` `Project, Integer> 或 ToIntBiFunction<Project, Project>` |

#### # Lambdas及函数式接口的例子

方法引用让你可以重复使用现有的方法定义，并像Lambda一样传递它们。
#### # 总结

- lambda 表达式可以理解为一种匿名函数：它没有名称，但有参数列表、函数主体、返回 类型，可能还有一个可以抛出的异常的列表。
- lambda 表达式让你可以简洁地传递代码。
- 函数式接口就是仅仅声明了一个抽象方法的接口。
- 只有在接受函数式接口的地方才可以使用 lambda 表达式。
- lambda 表达式允许你直接内联，为函数式接口的抽象方法提供实现，并且将整个表达式作为函数式接口的一个实例。
- Java 8自带一些常用的函数式接口，放在 `java.util.function` 包里，包括 `Predicate<T>`、`Function<T,R>`、`Supplier<T>`、`Consumer<T>` 和 `BinaryOperator<T>`。
- Lambda表达式所需要代表的类型称为目标类型。
- 方法引用让你重复使用现有的方法实现并直接传递它们。
- `Comparator``、`Predicate`和`Function` 等函数式接口都有几个可以用来结合 lambda 表达式的默认方法。

### 4、streamapi使用及其常用技巧汇总

#### # 什么是流？

先来看看项目中的使用

```
@Override
@Transactional(rollbackFor = Exception.class)
public Boolean saveUser(UserDTO userDto) {
   SysUser sysUser = new SysUser();
   BeanUtils.copyProperties(userDto, sysUser);
   sysUser.setDelFlag(CommonConstants.STATUS_NORMAL);
   sysUser.setPassword(ENCODER.encode(userDto.getPassword()));
   baseMapper.insert(sysUser);
   List<SysUserRole> userRoleList = userDto.getRole()
      .stream().map(roleId -> {
         SysUserRole userRole = new SysUserRole();
         userRole.setUserId(sysUser.getUserId());
         userRole.setRoleId(roleId);
         return userRole;
      }).collect(Collectors.toList());
   return sysUserRoleService.saveBatch(userRoleList);
}

```

流是Java8引入的全新概念，它用来处理集合中的数据，暂且可以把它理解为一种高级集合。 众所周知，集合操作非常麻烦，若要对集合进行筛选、投影，需要写大量的代码，而流是以声明的形式操作集合，它就像SQL语句，我们只需告诉流需要对集合进行什么操作，它就会自动进行操作，并将执行结果交给你，无需我们自己手写代码。 因此，流的集合操作对我们来说是透明的，我们只需向流下达命令，它就会自动把我们想要的结果给我们。由于操作过程完全由Java处理，因此它可以根据当前硬件环境选择最优的方法处理，我们也无需编写复杂又容易出错的多线程代码了。



#### # 流的特点

1. 只能遍历一次 我们可以把流想象成一条流水线，流水线的源头是我们的数据源(一个集合)，数据源中的元素依次被输送到流水线上，我们可以在流水线上对元素进行各种操作。 一旦元素走到了流水线的另一头，那么这些元素就被“消费掉了”，我们无法再对这个流进行操作。当然，我们可以从数据源那里再获得一个新的流重新遍历一遍。
2. 采用内部迭代方式 若要对集合进行处理，则需我们手写处理代码，这就叫做外部迭代。 而要对流进行处理，我们只需告诉流我们需要什么结果，处理过程由流自行完成，这就称为内部迭代。

#### # 流的操作种类

流的操作分为两种，分别为中间操作和终端操作。

1. 中间操作 当数据源中的数据上了流水线后，这个过程对数据进行的所有操作都称为“中间操作”。 中间操作仍然会返回一个流对象，因此多个中间操作可以串连起来形成一个流水线。
2. 终端操作 当所有的中间操作完成后，若要将数据从流水线上拿下来，则需要执行终端操作。 终端操作将返回一个执行结果，这就是你想要的数据。

#### # 流的操作过程

使用流一共需要三步：

1. 准备一个数据源
2. 执行中间操作 中间操作可以有多个，它们可以串连起来形成流水线。
3. 执行终端操作 执行终端操作后本次流结束，你将获得一个执行结果。

####  # 使用流

##### # 创建流

在使用流之前，首先需要拥有一个数据源，并通过StreamAPI提供的一些方法获取该数据源的流对象。数据源可以有多种形式：

**1. 集合**

这种数据源较为常用，通过stream()方法即可获取流对象：

```
List<Person> list = new ArrayList<Person>(); 
Stream<Person> stream = list.stream();
```

**2. 数组**

通过Arrays类提供的静态函数stream()获取数组的流对象：

```
String[] names = {"chaimm","peter","john"};
Stream<String> stream = Arrays.stream(names);
```

**3. 值**

直接将几个值变成流对象：

```
Stream<String> stream = Stream.of("chaimm","peter","john");
```

**4. 文件**

```
try(Stream lines = Files.lines(Paths.get(“文件路径名”),Charset.defaultCharset())){
    //可对lines做一些操作
}catch(IOException e){
}
```

**5. iterator**

**创建无限流**

```
Stream.iterate(0, n -> n + 2)
      .limit(10)
      .forEach(System.out::println);
```

> PS：Java7简化了IO操作，把打开IO操作放在try后的括号中即可省略关闭IO的代码。

##### # 筛选 filter

filter 函数接收一个Lambda表达式作为参数，该表达式返回boolean，在执行过程中，流将元素逐一输送给filter，并筛选出执行结果为true的元素。 如，筛选出所有学生：

```
List<Person> result = list.stream()
                    .filter(Person::isStudent)
                    .collect(toList());

```

##### # 去重distinct

去掉重复的结果：

```
List<Person> result = list.stream()
                    .distinct()
                    .collect(toList());
```

##### # 截取

截取流的前N个元素：

```
List<Person> result = list.stream()
                    .limit(3)
                    .collect(toList());
```

##### # 跳过

跳过流的前n个元素：

```
List<Person> result = list.stream()
                    .skip(3)
                    .collect(toList());
```

##### # 映射

对流中的每个元素执行一个函数，使得元素转换成另一种类型输出。流会将每一个元素输送给map函数，并执行map中的Lambda表达式，最后将执行结果存入一个新的流中。 如，获取每个人的姓名(实则是将Perosn类型转换成String类型)：

```
List<Person> result = list.stream()
                    .map(Person::getName)
                    .collect(toList());
```

##### # 合并多个流

例：列出List中各不相同的单词，List集合如下：

```
List<String> list = new ArrayList<String>();
list.add("I am a boy");
list.add("I love the girl");
list.add("But the girl loves another girl");
```

思路如下：

首先将list变成流：

```
list.stream();
```

按空格分词：	

```
list.stream()
            .map(line->line.split(" "));
	
```

分完词之后，每个元素变成了一个String[]数组。

将每个 `String[]` 变成流：

```
list.stream()
            .map(line->line.split(" "))
            .map(Arrays::stream)
```

此时一个大流里面包含了一个个小流，我们需要将这些小流合并成一个流。

将小流合并成一个大流：用 `flatMap` 替换刚才的 map

```
list.stream()
    .map(line->line.split(" "))
    .flatMap(Arrays::stream)
```

去重

```
list.stream()
    .map(line->line.split(" "))
    .flatMap(Arrays::stream)
    .distinct()
    .collect(toList());
```

##### #  是否匹配任一元素：anyMatch

anyMatch用于判断流中是否存在至少一个元素满足指定的条件，这个判断条件通过Lambda表达式传递给anyMatch，执行结果为boolean类型。 如，判断list中是否有学生：

```
boolean result = list.stream()
            .anyMatch(Person::isStudent);
```

#####  # 是否匹配所有元素：allMatch

allMatch用于判断流中的所有元素是否都满足指定条件，这个判断条件通过Lambda表达式传递给anyMatch，执行结果为boolean类型。 如，判断是否所有人都是学生：

```
boolean result = list.stream()
            .allMatch(Person::isStudent);
```

##### # 是否未匹配所有元素：noneMatch

noneMatch与allMatch恰恰相反，它用于判断流中的所有元素是否都不满足指定条件：

```
boolean result = list.stream()
            .noneMatch(Person::isStudent);
```

##### # 获取任一元素findAny

findAny能够从流中随便选一个元素出来，它返回一个Optional类型的元素

```
Optional<Person> person = list.stream().findAny();
```

##### # 获取第一个元素findFirst

```
Optional<Person> person = list.stream().findFirst();
```

##### #  归约

归约是将集合中的所有元素经过指定运算，折叠成一个元素输出，如：求最值、平均数等，这些操作都是将一个集合的元素折叠成一个元素输出。

在流中，reduce函数能实现归约。 reduce函数接收两个参数：

1. 初始值
2. 进行归约操作的Lambda表达式

**元素求和：自定义Lambda表达式实现求和**

例：计算所有人的年龄总和

```
int age = list.stream().reduce(0, (person1,person2)->person1.getAge()+person2.getAge());
```

1. reduce的第一个参数表示初试值为0；
2. reduce的第二个参数为需要进行的归约操作，它接收一个拥有两个参数的Lambda表达式，reduce会把流中的元素两两输给Lambda表达式，最后将计算出累加之和。

**元素求和：使用Integer.sum函数求和**

上面的方法中我们自己定义了Lambda表达式实现求和运算，如果当前流的元素为数值类型，那么可以使用Integer提供了sum函数代替自定义的Lambda表达式，如：

```
int age = list.stream().reduce(0, Integer::sum);
```

Integer类还提供了 `min`、`max` 等一系列数值操作，当流中元素为数值类型时可以直接使用。

##### # 数值流的使用

采用reduce进行数值操作会涉及到基本数值类型和引用数值类型之间的装箱、拆箱操作，因此效率较低。 当流操作为纯数值操作时，使用数值流能获得较高的效率。

**将普通流转换成数值流**

StreamAPI提供了三种数值流：IntStream、DoubleStream、LongStream，也提供了将普通流转换成数值流的三种方法：mapToInt、mapToDouble、mapToLong。 如，将Person中的age转换成数值流：

```
IntStream stream = list.stream().mapToInt(Person::getAge);
```

**数值计算**

每种数值流都提供了数值计算函数，如max、min、sum等。如，找出最大的年龄：

```
OptionalInt maxAge = list.stream()
                                .mapToInt(Person::getAge)
                                .max();
```

由于数值流可能为空，并且给空的数值流计算最大值是没有意义的，因此max函数返回OptionalInt，它是Optional的一个子类，能够判断流是否为空，并对流为空的情况作相应的处理。 此外，mapToInt、mapToDouble、mapToLong进行数值操作后的返回结果分别为：OptionalInt、OptionalDouble、OptionalLong

##### # 中间操作和收集操作

|    操作     | 类型 | 返回类型      | 使用的类型/函数式接口    | 函数描述符       |
| :---------: | :--- | :------------ | :----------------------- | :--------------- |
|  `filter`   | 中间 | `Stream<T>`   | `Predicate<T>`           | `T -> boolean`   |
| `distinct`  | 中间 | `Stream<T>`   |                          |                  |
|   `skip`    | 中间 | `Stream<T>`   | long                     |                  |
|    `map`    | 中间 | `Stream<R>`   | `Function<T, R>`         | `T -> R`         |
|  `flatMap`  | 中间 | `Stream<R>`   | `Function<T, Stream<R>>` | `T -> Stream<R>` |
|   `limit`   | 中间 | `Stream<T>`   | long                     |                  |
|  `sorted`   | 中间 | `Stream<T>`   | `Comparator<T>`          | `(T, T) -> int`  |
| `anyMatch`  | 终端 | `boolean`     | `Predicate<T>`           | `T -> boolean`   |
| `noneMatch` | 终端 | `boolean`     | `Predicate<T>`           | `T -> boolean`   |
| `allMatch`  | 终端 | `boolean`     | `Predicate<T>`           | `T -> boolean`   |
|  `findAny`  | 终端 | `Optional<T>` |                          |                  |
| `findFirst` | 终端 | `Optional<T>` |                          |                  |
|  `forEach`  | 终端 | `void`        | `Consumer<T>`            | `T -> void`      |
|  `collect`  | 终端 | `R`           | `Collector<T, A, R>`     |                  |
|  `reduce`   | 终端 | `Optional<T>` | `BinaryOperator<T>`      | `(T, T) -> T`    |
|   `count`   | 终端 | `long`        |                          |                  |

##### # Collector 收集

收集器用来将经过筛选、映射的流进行最后的整理，可以使得最后的结果以不同的形式展现。 `collect` 方法即为收集器，它接收 `Collector` 接口的实现作为具体收集器的收集方法。 `Collector` 接口提供了很多默认实现的方法，我们可以直接使用它们格式化流的结果；也可以自定义 `Collector` 接口的实现，从而定制自己的收集器。

##### # 归约

流由一个个元素组成，归约就是将一个个元素“折叠”成一个值，如求和、求最值、求平均值都是归约操作。

##### #  一般性归约

若你需要自定义一个归约操作，那么需要使用 `Collectors.reducing` 函数，该函数接收三个参数：

- 第一个参数为归约的初始值
- 第二个参数为归约操作进行的字段
- 第三个参数为归约操作的过程

##### # 汇总

Collectors类专门为汇总提供了一个工厂方法：`Collectors.summingInt`。 它可接受一 个把对象映射为求和所需int的函数，并返回一个收集器；该收集器在传递给普通的 `collect` 方法后即执行我们需要的汇总操作。

##### # 分组

数据分组是一种更自然的分割数据操作，分组就是将流中的元素按照指定类别进行划分，类似于SQL语句中的 `GROUPBY`。	

##### # 多级分组

多级分组可以支持在完成一次分组后，分别对每个小组再进行分组。 使用具有两个参数的 `groupingBy` 重载方法即可实现多级分组。

- 第一个参数：一级分组的条件
- 第二个参数：一个新的 `groupingBy` 函数，该函数包含二级分组的条件

**Collectors 类的静态工厂方法**

|      工厂方法       | 返回类型               | 用途                                                         | 示例                                                         |
| :-----------------: | :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
|      `toList`       | `List<T>`              | 把流中所有项目收集到一个 List                                | `List<Project> projects = projectStream.collect(toList());`  |
|       `toSet`       | `Set<T>`               | 把流中所有项目收集到一个 Set，删除重复项                     | `Set<Project> projects = projectStream.collect(toSet());`    |
|   `toCollection`    | `Collection<T>`        | 把流中所有项目收集到给定的供应源创建的集合                   | `Collection<Project> projects = projectStream.collect(toCollection(), ArrayList::new);` |
|     `counting`      | `Long`                 | 计算流中元素的个数                                           | `long howManyProjects = projectStream.collect(counting());`  |
|    `summingInt`     | `Integer`              | 对流中项目的一个整数属性求和                                 | `int totalStars = projectStream.collect(summingInt(Project::getStars));` |
|   `averagingInt`    | `Double`               | 计算流中项目 Integer 属性的平均值                            | `double avgStars = projectStream.collect(averagingInt(Project::getStars));` |
|  `summarizingInt`   | `IntSummaryStatistics` | 收集关于流中项目 Integer 属性的统计值，例如最大、最小、 总和与平均值 | `IntSummaryStatistics projectStatistics = projectStream.collect(summarizingInt(Project::getStars));` |
|      `joining`      | `String`               | 连接对流中每个项目调用 toString 方法所生成的字符串           | `String shortProject = projectStream.map(Project::getName).collect(joining(", "));` |
|       `maxBy`       | `Optional<T>`          | 按照给定比较器选出的最大元素的 Optional， 或如果流为空则为 Optional.empty() | `Optional<Project> fattest = projectStream.collect(maxBy(comparingInt(Project::getStars)));` |
|       `minBy`       | `Optional<T>`          | 按照给定比较器选出的最小元素的 Optional， 或如果流为空则为 Optional.empty() | `Optional<Project> fattest = projectStream.collect(minBy(comparingInt(Project::getStars)));` |
|     `reducing`      | 归约操作产生的类型     | 从一个作为累加器的初始值开始，利用 BinaryOperator 与流中的元素逐个结合，从而将流归约为单个值 | `int totalStars = projectStream.collect(reducing(0, Project::getStars, Integer::sum));` |
| `collectingAndThen` | 转换函数返回的类型     | 包含另一个收集器，对其结果应用转换函数                       | `int howManyProjects = projectStream.collect(collectingAndThen(toList(), List::size));` |
|    `groupingBy`     | `Map<K, List<T>>`      | 根据项目的一个属性的值对流中的项目作问组，并将属性值作 为结果 Map 的键 | `Map<String,List<Project>> projectByLanguage = projectStream.collect(groupingBy(Project::getLanguage));` |
|  `partitioningBy`   | `Map<Boolean,List<T>>` | 根据对流中每个项目应用断言的结果来对项目进行分区             | `Map<Boolean,List<Project>> vegetarianDishes = projectStream.collect(partitioningBy(Project::isVegetarian));` |

##### # 转换类型

有一些收集器可以生成其他集合。比如前面已经见过的 `toList`，生成了 `java.util.List` 类的实例。 还有 `toSet` 和 `toCollection`，分别生成 `Set` 和 `Collection` 类的实例。 到目前为止， 我已经讲了很多流上的链式操作，但总有一些时候，需要最终生成一个集合——比如：

- 已有代码是为集合编写的，因此需要将流转换成集合传入；
- 在集合上进行一系列链式操作后，最终希望生成一个值；
- 写单元测试时，需要对某个具体的集合做断言。

使用 `toCollection`，用定制的集合收集元素

```
stream.collect(toCollection(TreeSet::new));
```

还可以利用收集器让流生成一个值。 `maxBy` 和 `minBy` 允许用户按某种特定的顺序生成一个值。

##### # 数据分区

分区是分组的特殊情况：由一个断言（返回一个布尔值的函数）作为分类函数，它称分区函数。 分区函数返回一个布尔值，这意味着得到的分组 `Map` 的键类型是 `Boolean`，于是它最多可以分为两组: true是一组，false是一组。

分区的好处在于保留了分区函数返回true或false的两套流元素列表。

##### # 并行流

并行流就是一个把内容分成多个数据块，并用不不同的线程分别处理每个数据块的流。最后合并每个数据块的计算结果。

将一个顺序执行的流转变成一个并发的流只要调用 `parallel()` 方法

```
public static long parallelSum(long n){
    return Stream.iterate(1L, i -> i +1).limit(n).parallel().reduce(0L,Long::sum);
}
```

将一个并发流转成顺序的流只要调用 `sequential()` 方法

```
stream.parallel().filter(...).sequential().map(...).parallel().reduce();
```

这两个方法可以多次调用，只有最后一个调用决定这个流是顺序的还是并发的。

并发流使用的默认线程数等于你机器的处理器核心数。

通过这个方法可以修改这个值，这是全局属性。

```
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "12");
```

并非使用多线程并行流处理数据的性能一定高于单线程顺序流的性能，因为性能受到多种因素的影响。 如何高效使用并发流的一些建议：

1. 如果不确定， 就自己测试。
2. 尽量使用基本类型的流 IntStream, LongStream, DoubleStream
3. 有些操作使用并发流的性能会比顺序流的性能更差，比如limit，findFirst，依赖元素顺序的操作在并发流中是极其消耗性能的。findAny的性能就会好很多，应为不依赖顺序。
4. 考虑流中计算的性能(Q)和操作的性能(N)的对比, Q表示单个处理所需的时间，N表示需要处理的数量，如果Q的值越大, 使用并发流的性能就会越高。
5. 数据量不大时使用并发流，性能得不到提升。
6. 考虑数据结构：并发流需要对数据进行分解，不同的数据结构被分解的性能时不一样的。

**流的数据源和可分解性**

|        源         | 可分解性 |
| :---------------: | :------- |
|    `ArrayList`    | 非常好   |
|   `LinkedList`    | 差       |
| `IntStream.range` | 非常好   |
| `Stream.iterate`  | 差       |
|     `HashSet`     | 好       |
|     `TreeSet`     | 好       |

**流的特性以及中间操作对流的修改都会对数据对分解性能造成影响。 比如固定大小的流在任务分解的时候就可以平均分配，但是如果有filter操作，那么流就不能预先知道在这个操作后还会剩余多少元素。**

**考虑终端操作的性能：如果终端操作在合并并发流的计算结果时的性能消耗太大，那么使用并发流提升的性能就会得不偿失。**



### 5、日志处理-SpringEvent处理机制

#### # @Log 注解

- 接口上使用@Log 注释当前接口的作用即可

```
/**
 * 保存单条
 *
 * @param param 保存参数
 * @return 是否添加成功
 */
@Log("保存数据到Account")
@CessBody
@ApiOperation(value = "保存", notes = "保存数据到Account")
@PostMapping(value = "/add.action")
public Integer addAccount(@RequestBody(required = false) AccountForm param) {
    Integer result = accountService.save(param);
    return result;
}
```

#### # 原理讲解

```
@Aspect
@Slf4j
public class LogAspect {

	@Around("@annotation(Log)")
	public Object around(ProceedingJoinPoint point, SysLog sysLog) throws Throwable {
		String strClassName = point.getTarget().getClass().getName();
		String strMethodName = point.getSignature().getName();
		log.debug("[类名]:{},[方法]:{}", strClassName, strMethodName);
		SpringContextHolder.publishEvent(new SysLogEvent(logVo));
		return obj;
	}

}
```

- 监听器在接收到日志事件后进行处理

```
@Slf4j
@AllArgsConstructor
public class LogListener {
	private final RemoteLogService remoteLogService;

	@Async
	@Order
	@EventListener(LogEvent.class)
	public void saveLog(SysLogEvent event) {
		Log log = (SysLog) event.getSource();
		remoteLogService.Log(Log, SecurityConstants.FROM_IN);
	}
}
```

#### # 异步操作说明 @EnableAsync

@EnableAsync 注解启用了 Spring 异步方法执行功能，在[ Spring Framework API](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/EnableAsync.html) 中有详细介绍。



##### # 日志查看

使用日志注解之后，每次请求接口都会在系统日志表中生成一条记录，分2种类型

1、info级别

```
SELECT * from log where log_type='INFO' ORDER BY create_time desc  LIMIT 0,10
```

2、error级别

```
SELECT * from log where log_type='ERROR' ORDER BY create_time desc  LIMIT 0,10
```



### 6、代码生成使用

#### # 功能支持

#####     # 前端（暂不支持）

#####  	# 后端

- entity
- mapper
- service
- Controller

#### # 开始使用

###### # 项目启动

1. 启动对应工具类

   路径：visolink-sales-api/src/main/java/cn/visolink/utilsMabatisPlusGenerator

2. 在控制台输入对应数据库表名、表前缀和模块名称

```
请输入表名：
s_account
```

```
请输入表前缀：
s
```

```
请输入模块名：
account
```

3、生成目录结构展示

生成代码结构,可以覆盖到指定业务模块

![image-20190827164313198](http://feicuibabatest.qiniudn.com/image-20190827164313198.png)



### 7、后端权限控制

本系统安全框架使用的是 `Spring Security + Jwt Token`， 访问后端接口需在请求头中携带`token`进行访问，请求头格式如下：

```
# Authorization: Bearer 登录时返回的token
Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhZG1pbiIsImV4cCI6MTU1ODk2NzY0OSwiaWF0IjoxNTU4OTQ2MDQ5fQ.jsJvqHa1tKbJazG0p9kq5J2tT7zAk5B6N_CspdOAQLWgEICStkMmvLE-qapFTtWnnDUPAjqmsmtPFSWYaH5LtA
```

也可以过滤一些接口如：`Druid`监控，`swagger`文档，支付宝回调通知等。

配置文件位于：`visolink-sales-api -> common -> security ->  config -> SecurityConfig`

```
// 关键代码，部分略
protected void configure(HttpSecurity httpSecurity) throws Exception {
    httpSecurity
            // 禁用 CSRF
            .csrf().disable()
            // 授权异常
            .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
            // 不创建会话
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
            .authorizeRequests()
            .antMatchers("/druid/**").permitAll()
            // 所有请求都需要认证
            .anyRequest().authenticated();
    httpSecurity
            .addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
}
```
<font color=red>permitAll()</font>方法指所有登录和未登录人员都可以访问，这个会经过 <font color=red>security filter</font>

<font color=red>anonymous() </font>所有人都能访问，但是这个不会经过  <font color=red>security filter</font>

#### # 系统数据交互

![](http://feicuibabatest.qiniudn.com/5c9c93b996ace.png)

#### # 接口权限控制

`Spring Security` 提供了`Spring EL`表达式，允许我们在定义接口访问的方法上面添加注解，来控制访问权限，相关 `EL`总结如下

| 表达式                         | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| hasRole([role])                | 当前用户是否拥有指定角色。                                   |
| hasAnyRole([role1,role2])      | 多个角色是一个以逗号进行分隔的字符串。如果当前用户拥有指定角色中的任意一个则返回true。 |
| hasAuthority([auth])           | 等同于hasRole                                                |
| hasAnyAuthority([auth1,auth2]) | 等同于hasAnyRole                                             |
| Principle                      | 代表当前用户的principle对象                                  |
| authentication                 | 直接从SecurityContext获取的当前Authentication对象            |
| permitAll                      | 总是返回true，表示允许所有的                                 |
| denyAll                        | 总是返回false，表示拒绝所有的                                |
| isAnonymous()                  | 当前用户是否是一个匿名用户                                   |
| isRememberMe()                 | 表示当前用户是否是通过Remember-Me自动登录的                  |
| isAuthenticated()              | 表示当前用户是否已经登录认证成功了。                         |
| isFullyAuthenticated()         | 如果当前用户既不是一个匿名用户，同时又不是通过Remember-Me自动登录的，则返回true。 |

### 8、异常处理

我们开发项目的时，数据在请求过程中发生错误是非常常见的事情。如：权限不足、数据唯一异常、数据不能为空异常、义务异常等。这些异常如果不经过处理会对前端开发人员和使用者造成不便，因此我们就需要统一处理他们。
源码位于：`visolink-common - > exception`

##### 定义异常实体

```
/**
 *
 * @date 2018-11-23
 */
@Data
class ApiError {

    private long code;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private String messages;

    private ApiError() {
    }

    public ApiError(long code,String messages) {
        this();
        this.code = code;
        this.messages = messages;
    }
    private ApiError(String message) {
        this();
        this.messages = messages;
    }

}
```

##### 封装异常处理

1、通用异常

封装了 `BadRequestException`，用于处理通用的异常

```
/**
 * @author WCL
 * @date 2018-11-23
 * 统一异常处理
 */
@Getter
@Slf4j
public class BadRequestException extends RuntimeException{

    private Integer code = BAD_REQUEST.value();

    private Exception exception;

    public BadRequestException(String msg){
        super(msg);
    }

    public BadRequestException(Integer code){
        super();
        this.code=code;
    }

    public BadRequestException(HttpStatus code,String msg){
        super(msg);
        this.code = code.value();
    }
    public BadRequestException(int code,String msg){
        super(msg);
        this.code = code;
    }

    public BadRequestException(int code,String msg,Exception e){
        super(msg);
       log.error(e.getMessage(),e);
        this.code = code;
        this.exception=e;
    }


}
```

##### 全局异常拦截

使用全局异常处理器 `@RestControllerAdvice` 处理请求发送的异常，部分代码如下：

```

/**
 * @author WCL
 * @date 2018-11-23
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 处理所有不可知的异常
     * @param e
     * @return
     */
    @ExceptionHandler(Throwable.class)
    public ResultBody handleException(Throwable e){
        // 打印堆栈信息
        log.error(ThrowableUtil.getStackTrace(e));
        ApiError apiError = new ApiError(BAD_REQUEST.value(),e.getMessage());
        return buildResponseEntity(apiError);
    }

    /**
     * 处理 接口无权访问异常AccessDeniedException
     * @param e
     * @return
     */
    @ExceptionHandler(AccessDeniedException.class)
    public ResultBody handleAccessDeniedException(AccessDeniedException e){
        // 打印堆栈信息
        log.error(ThrowableUtil.getStackTrace(e));
        ApiError apiError = new ApiError(FORBIDDEN.value(),e.getMessage());
        return buildResponseEntity(apiError);
    }

    /**
     * 处理自定义异常
     * @param e
     * @return
     */
	@ExceptionHandler(value = BadRequestException.class)
	public ResultBody badRequestException(BadRequestException e) {
        // 打印堆栈信息
        log.error(ThrowableUtil.getStackTrace(e));
        ApiError apiError = new ApiError(e.getCode(),e.getMessage());

        return buildResponseEntity(apiError);
	}


    /**
     * 处理所有接口数据验证异常
     * @param e
     * @returns
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResultBody handleMethodArgumentNotValidException(MethodArgumentNotValidException e){
        // 打印堆栈信息
        log.error(ThrowableUtil.getStackTrace(e));
        String[] str = e.getBindingResult().getAllErrors().get(0).getCodes()[1].split("\\.");
        StringBuffer msg = new StringBuffer(str[1]+":");
        msg.append(e.getBindingResult().getAllErrors().get(0).getDefaultMessage());
        ApiError apiError = new ApiError(BAD_REQUEST.value(),msg.toString());
        return buildResponseEntity(apiError);
    }

    /**
     * 统一返回
     * @param apiError
     * @return
     */
    private ResultBody buildResponseEntity(ApiError apiError) {
        return ResultUtil.error(apiError.getCode(),apiError.getMessages());
    }
}
```

##### 具体使用

```
throw new BadRequestException("发生了异常");
```

