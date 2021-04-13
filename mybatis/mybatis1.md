

![image-20210402220350065](C:\Users\86150\AppData\Roaming\Typora\typora-user-images\image-20210402220350065.png)

## 缓存

mybatis只有一个Cache顶层接口，和一个实现类PerpetualCache，剩下的类都是装饰器，通过构造函数传入一个Cache对象（接口不能直接new）。PerpetualCache缓存是由一个Map实现的。



### 一级缓存

#### 生命周期

一级缓存存在于Executor中，由于Executor与SqlSession同一生命周期，因此一级缓存的生命周期与SqlSession相同。

#### 实现

在执行第一次quey操作时，会调用queryFromDatabase方法，此时就创建缓存，将其添加到PerpetualCache的map中。用一个CacheKey对象中的List储存当前查询操作的各种信息，CacheKey是缓存map的key。CacheKey重写了eqauls和hashcode方法，所以属性相同可以从Map中取出。

一级缓存是sqlsession同生命周期的，因此一级缓存也应该在sqlsession中，事实上缓存在sqlsession的executor中。

#### 清空一级缓存

- 配置中是否开启缓存
- localCacheScope的范围，如果是Statement则每次查询清空缓存
- commit、rollback、update时清空缓存

#### 缺点

一级缓存在一个sqlsession中共享，在多线程或者分布式的环境下无法互相感知，会造成脏数据的问题



### 二级缓存

针对mapper（Dao）进行缓存

Spring和MyBatis整合时，每次查询之后都要进行关闭sqlsession，关闭之后数据被清空。所以MyBatis和Spring整合之后，一级缓存是没有意义的。如果开启二级缓存，关闭sqlsession后，会把该sqlsession一级缓存中的数据添加到mapper namespace的二级缓存中。这样，缓存在sqlsession关闭之后依然存在。

#### 生命周期

mapper的namespace共享，需要commit才可以生效

#### 实现

通过一个装饰类CachingExecutor来实现。CachingExecutor中有一个Executor用于执行sql语句，还有一个TransactionCache来管理二级缓存。TransactionCache就是Cache的拓展，本质上也是维护了一个map来实现缓存。TransactionCacheManager中有一个Cache到TransactionCache映射的map，对不同命名空间的缓存进行映射管理。

在查询操作时，首先会创建一个和一级缓存一样的CacheKey，然后到二级缓存中查询。如果二级缓存没有，那么之后的执行流程就和一级缓存一样，然后将结果存到二级缓存中。

在读取配置文件的时候，读到开启二级缓存的配置时，就会开始创建二级缓存，

自己的理解：（并将二级缓存添加到configuration对象中。而configuration其是用来配置创建sqlSessionFactory的配置类，因此相当与将二级缓存添加到sqlSessionFactory中，实现了全局缓存。）

别人的理解：在cachingExecutor中的query方法，获得了MapperStatement 中的Cache。MapperStatement的Cache是在创建的时候外部传入的。这个Cache的创建过程就是在上面我自己的理解中写的。流程如下。

![image-20210209233231016](D:\笔记\java笔记\mybatis\image-20210209233231016.png)

在调用cachingExecutor中的query方法时，获取到了MapperStatement的Cache，就是某个命名空间的Cache。利用该Cache往TransactionCacheManager管理的Cache- TransactionCache对中添加元素，这其实是二级缓存的一个步骤，进行事务管理，在cache上装饰层TransactionalCache。其实二级缓存还是在上面所说的Cache中，不过利用TransactionalCache对其进行事务管理。这样在事务提交（commit）、回滚（rollback）时，对二级缓存进行相应的操作。



**注意**：在事务提交之前，它只是暂存，不会存到真正的二级缓存中，事务提交了之后才会储存到真正的二级缓存中。因此需要commit二级缓存才有效。

#### 缺点

update操作会使缓存失效，因此适用于读多写少的环境。

并且不能用于多表操作，多表查询一定会产生脏数据



### 缓存穿透

用数据库中不存在的数据进行访问，每次访问都在缓存中查询不到，会直接访问数据库，导致压力过大；

解决方案：

- 将不存在的访问也缓存起来，返回空值
- 检测访问的数据是否合法
- 将所有可能访问的数据都hash起来存在hashmap中，那么不存在的数据则会被hashmap过滤。

### 缓存雪崩

大量的key在同一时间过期，导致访问数据库的请求突然增大。

解决方案：可以给缓存设置过期时间时加上一个随机值时间，使得每个key的过期时间分布开来，不会集中在同一时刻失效。

### 缓存击穿

一个存在的key，在缓存过期的一刻，同时有大量的请求，这些请求都会击穿到DB，造成瞬时DB请求量大、压力骤增。

- 解决方案：加锁，在找不到缓存时，给访问数据库的key进行加锁。这样就不会有大量相同的key同时访问数据库。mybatis用的就是这种方法。

- 检查更新：给缓存加一个时间字段。如果查询的时候发现缓存的剩余时间小于某个特定值，那么就直接在数据库中查询并更新缓存。

- 分级缓存：采用 L1 (一级缓存)和 L2(二级缓存) 缓存方式，L1 缓存失效时间短，L2 缓存失效时间长。 请求优先从 L1 缓存获取数据，如果 L1缓存未命中则加锁，只有 1 个线程获取到锁,这个线程再从数据库中读取数据并将数据再更新到到 L1 缓存和 L2 缓存中，而其他线程依旧从 L2 缓存获取数据并返回。





## sql语句执行流程

在项目启动的时候，注解和xml中的sql语句被加载到Configuration对象的一个Map中，在查询语句时，通过SqlSession进行查询。SqlSession中有Configuration对象和Excutor对象。通过Configuration对象获取对应的sql语句，通过Excutor执行sql语句。

但是在实际调用中我们是使用sqlSession的getMapper方法获取一个Mapper，通过这个Mapper调用接口。但是实际的执行方法却是在sqlSession的excutor中，因此需要代理模式。

getMapper方法最终调用到一个MapperRegistry类中。该类中有一个Map储存了接口（Dao与MapperProxyFactory的映射）。通过MapperProxyFactory的getInstance方法获取代理对象。

代理对象的执行类MapperProxy有一个sqlsession属性，有一个类名属性，有一个Map储存Method类到MapperMethod的映射。

调用方法的时候，会进入MapperProxy的invoke方法。invoke方法调用了MapperMethod的executor方法。

那么mapperMethod如何确定是哪个方法呢？通过接口（mapper）类型以及要执行的方法，从程序刚开始加载就存起来的方法缓存（应该是mappedstatement）中获取信息

MapperMethod通过被代理类Method对象，被代理类的类型，以及Configuration对象获取所需信息。

MapperMethod的excutor方法调用了sqlSession的对应的执行sql语句的方法。至此调用结束。

### MapperMethod和MappedStatement

在Mapper的代理对象调用的invoke方法，实际上是new了一个MapperMethod对象然后调用其execute方法。在new对象的过程中，是从configuration中获取MappedStatement对象来获取sql语句的信息。MappedStatement对象在程序刚加载的时候就已经创立，并且通过Map<String,MappedStatement>存在configuration中。



## sqlsession的生命周期

在mybatis原生的代码中，sqlsession不是线程安全的。因此此时sqlsession的生命周期是一次查询。

由于mybatis与spring省去了手动创建sqlsession的步骤，需要sqlsession一直存在容器中，因此要保证线程安全，因此用一个SqlSessionTemplate类替代DeafaultSqlSession。

SqlSessionTemplate对象中有一个sqlsession的代理。通过这个代理对象执行sql语句。



## 延迟加载

多级查询的时候，比如查询一个对象，该对象包含了另一个对象。如果被包含的对象不需要就不加载出来，称为延迟加载。



## 分层

mybatis大体上分为三层，最上层是暴露给用户的接口层sqlsession，第二层是核心处理层，包含了配置解析，参数映射，sql解析，sql执行，结果集映射等等。第三层是基础支撑层，是一些通用组件的封装，如日志、缓存、反射、数据源等等。



## 日志

mybatis没有实现日志功能，而是使用第三方日志。为了兼容这些日志，mybatis使用了适配器模式，提供了一个统一的日志接口。引入第三方组件只需要实现该类的接口，在接口中调用相应的api就可以。

在执行操作的时候要打印日志，相当于对方法进行增强，因此用到动态代理模式。



## 配置文件

配置文件中的environment标签，用于配置不同的环境，比如开发环境，生产环境，测试环境等等，就是不同的数据库

environment标签中有一个transactionManager的标签，用于指定JDBC和MANAGED为事务管理器





## 嵌套查询和嵌套结果

嵌套结果：通过association配置resuiltMap，用一条sql就可以进行关联查询。

嵌套查询：在配置resultMap的时候使用association，并指明了嵌套查询的查询语句的id。需要使用多条sql语句。

嵌套查询会导致N+1问题。查老师和老师所有的指导学生。查老师查出来一个列表，用了一条sql，对每个老师又都要查询其学生，就要N条sql，这就是性能的瓶颈。用嵌套结果，就是一条sql就可以查出来。



## # $

在#传入的时候，是把数值转为字符串，而用$就是直接把值传进去。

#将那个位置换为？，然后用prepareStatement构建sql语句。

orderby需要用$

$存在sql注入的问题，比如用户输入中包含了sql语句，要查一个userName = chenyuhan or 1=1





## 查询主键自增中最大的一条数据

- 用max函数
- 用desc然后limit1。





## 分页

偏移量过大会导致数据查询的效率很低。

- 可以先查主键，再根据主键查询内容
- 可以设置查询条件，例如where id>10000 limit 10





## 执行器

- simple

它的特点是在一次会话中，每次处理sql的时候，在进过StatementHandler 都会构建一个新的Statement,会导致即使相同的sql也无法重用Statement。

- flushstatement：在rollback和commit、close之前对statement进行刷新。（batch和reuse才有）

- batch

在本来jdbc的batch方法中，是先给statement加入一批参数，然后addBatch，然后executeBatch。在BatchExecutor中在update的时候仅仅是将参数绑定，而真正提交是在flushStatement中。就是在rollback，commit，close之前提交，或者在下一次查询之前提交。

BatchExecutor 用做批处理，会将所有的sql请求集中起来，最后在调用Executor.flushStatements()方法时，一次性将所有请求发送至数据库。（只用于update的批处理）

- reuse（执行完毕之后statement放到map中，以后要用到就先从map中找）

ResueExecutor区别在于它会将在会话期间内的Statement 进行缓存，并使用sql语句作为key，在执行下一次请求的时候，如果sql完全相同，直接拿出statement 进行执行。与简单执行器的区别就是，对statement进行缓存，如果是相同的sql，则复用statement。

- base

维护公共的逻辑，比如doquery，一级缓存的判断

- caching

对原有的executor进行装饰，实现二级缓存的功能。

![http://www.coderead.cn/p/mybatis/html/img/image-20200603165448345.png](https://img-blog.csdnimg.cn/20200818105702358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hVRENIU0RJ,size_16,color_FFFFFF,t_70#pic_center)





## 延迟加载的实现（可解决n+1问题）

通过ConcurrentLinkedQueue实现

- 在查询中a有b的引用，在要使用b的时候，先判断b是不是null，如果是null，就把事先保存好的sql发送运行。

## deferredLoad

- deferredLoad（可以用于解决循环查询的问题：学生查老师，老师再查学生）：用于实现本地懒加载。如果关联查询中查询不到，现在本地的缓存中查找。如果本地的缓存中找不到，再去数据库查询。

- 解决循环查询：在从数据库查询之前先在缓存中加入一个占位符，查到结果之后再将占位符用结果覆盖。然后判断有没有子查询。没有子查询则查询然后写入缓存，有子查询先执行子查询。

  如果在子查询中还有子查询，且该子查询就是上层查询，那么会在从缓存中得到占位符。此时就将本次查询的结果替换占位符，然后停止子查询，并将本次子查询放入deferredload中。

  queryStack用于记录嵌套查询在第几层。

  queryStack=0表示查询完毕，此时就将deferredLoad中的查询装载到第二次查询中，本次装载是通过本地缓存查到的。

  ![image-20210402211014112](C:\Users\86150\AppData\Roaming\Typora\typora-user-images\image-20210402211014112.png)

![image-20210402211140627](C:\Users\86150\AppData\Roaming\Typora\typora-user-images\image-20210402211140627.png)

## example

