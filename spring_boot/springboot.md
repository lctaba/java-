## controller

- 在controller层，如果使用的是@controller注解：
  - 没有使用@RequestBody注解，这时返回的字符串代表的是跳转地址，如return "a"代表的是跳转到“/static/a.html”的页面
  - 如果使用了@RequestBody注解，那么此时返回的数据会是json对象，return "a"就会在浏览器上显示字符串a
- 使用@RestController注解，就是@controller+@RequestBody





## 整合mybatis

在要使用数据库操作接口（Dao层）的层（如service层），要使用@Autowired注解自动注入Dao接口类型的一个对象，这样就可以使用其中的操作数据库的方法

```java
public interface UserDao{
    @Select(....)
    public User selectById(int id) {}
}
```

``` java
public class UserService{
    @Autowired
    private UserDao userDao;
    public User selectById(int id){
		return userDao.selectById(id);
    }
}
```



## 配置文件

1.在springboot整合配置文件，分成两大类：

application.properties

application.yml

或者是

Bootstrap.properties

Bootstrap.yml

相对于来说yml文件格式写法更加精简，减少配置文件的冗余性。

 

2.加载顺序：

bootstrap.yml 先加载 application.yml后加载

bootstrap.yml 用于应用程序上下文的引导阶段。

bootstrap.yml 由父Spring ApplicationContext加载。

3. 区别：

bootstrap.yml 和 application.yml 都可以用来配置参数。

bootstrap.yml 用来程序引导时执行，应用于更加早期配置信息读取。可以理解成系统级别的一些参数配置，这些参数一般是不会变动的。一旦bootStrap.yml 被加载，则内容不会被覆盖。

application.yml 可以用来定义应用级别的， 应用程序特有配置信息，可以用来配置后续各个模块中需使用的公共参数等。



### 读取配置文件的注解

#### @Value

读取单个配置，如

``` java
@Value("${classname.classproperty}")
private Property classproperty;

```

#### @ConfigerationProperties

通过配置文件的前缀读取含有这个前缀的所有配置，如

```java
@ConfigerationProperties(prefix = "ClassName")
public class ClassName {
    private Property property1;
    private Property property2;
    private Property property3;
    private Property property4;
}
```

#### 配置文件的占位符

应用场景：例如要注入的配置类想要生成随机参数

在SpringBoot的配置文件中，我们可以使用SpringBoot提供的的一些随机数

${random.value}、${random.int}、${random.long}

${random.int(10)}、${random.int[1024,65536]}

-${app.name:默认值} 来制定找不到属性时的默认值



#### 多环境配置

``` yaml
spring:
  profiles:
    active: dev
```



#### Tomcat端口号配置与访问路径配置

``` yaml
server:
  port: 8081
  servlet:
    context-path: /xxx
    #访问127.0.0.1：8080/xxx/...才能访问到
    #访问127.0.0.1：8080/...访问不到
```





application-dev.yml：开发环境application-test.yml：测试环境application-prd.yml：生产环境

不同配置文件对应不同环境下的配置



## 应用程序上下文是什么？





## 整合日志

### LogBack

- 导入lombok依赖

- 在property文件中指定logback.xml配置路径

  ``` yaml
  logging:
    config: classpath:log/logback.xml
  ```

- 在resources文件夹下新建logback.xml文件，并将配置信息复制

- 在要打印日志的类上加上注解@slf4j

- log.debug();debug级别日志

- log.info();info级别日志

- log.error();error级别日志

### Log4j

- 排除默认logback依赖
- 添加maven依赖
- 添加配置文件log4j.properties在resources文件夹下
- 在property文件中

``` yaml
logging:
  config: classpath:log4j.properties
```

### 定时任务（不支持分布式与集群）

 在启动类加上@EnableScheduling注解

在要启动定时任务的类上加上@Component注解

在要启动定时任务的方法上加上@Scheduled()注解指定时间间隔

- cron表达式表示时间间隔



## 异步

 在启动类加上@EnableAsync注解

在异步执行的方法上加入@Async注解

不要在大的项目上使用，因为很耗cpu，一般用Mq异步实现

@Async注解失效：将异步方法重新写在一个类中，不要与原类写在一起，但是没有结合线程池

开启线程池：编写线程池配置类，启用线程池只需要在@Async(ThreadPool)注解指名线程池名称,ThreadPool为注入容器线程池方法名称。





## 全局异常拦截

在出现异常后要执行的方法上加注解@ExceptionHandler(RuntimeException.class)注解