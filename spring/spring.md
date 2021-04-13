框架：是能完成一定功能的半成品。
框架能够帮助我们完成的是：项目的整体框架、一些基础功能、规定了类和对象如何创建，如何协作等，当我们开发一个项目时，框架帮助我们完成了一部分功能，我们自己再完成一部分，那这个项目就完成了。

非侵入式设计：
从框架的角度可以理解为：无需继承框架提供的任何类
这样我们在更换框架时，之前写过的代码几乎可以继续使用。

低侵入，低耦合，并且方便整合其他框架。

#### 能帮我们做什么

- aop帮助我们进行日志记录，性能统计，安全控制等任务。

- 帮我们很方便地集成其他框架。

#### spring注入的不一定是原来对象的实例，也有可能是原来对象的代理

例如有一个service类UserService

如果在这个类上面不加上@Transactional，那么说明不开启事务，那么@Autowired自动注入的是原对象，但是如果加上了@Transactional，那么因为要开启事务功能，就要对原对象进行加强，那么就要用到aop与动态代理，那么注入的会是代理对象。



#### spring如何注入

- Spring首先会扫描解析指定位置的所有的类得到Resources（可以理解为.Class文件）
- 然后依照TypeFilter和@Conditional注解决定是否将这个类解析为BeanDefinition
- 稍后再把一个个BeanDefinition取出实例化成Bean



#### spring如何处理管理对象

不仅仅是一个大的map，要用啥取啥。

spring中真正存单例对象的map仅仅是很小的一部分。仅仅是beanfactory的一个字段，被称为单例池。





#### bean的作用域

singleton：单例

prototype：为每个bean请求创建一个实例

request：为每一个request请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。

session：与request类似，不过将范围扩大到session

global-session：全局作用域



#### 单例下bean的安全性



## 单例bean不是单例模式

单例模式是整个jvm只有一个对象，而单例bean是整个容器中只有一个实例。

单例bean是与名称相关联的。单例指的是通过名字获取到的永远是一个对象。

原型指的是通过名称获取的不是同一个对象。



#### bean生命周期

实例化，赋初值，初始化，销毁





#### spring启动流程

① 实例化BeanFactory【DefaultListableBeanFactory】工厂，用于生成Bean对象
② 实例化BeanDefinitionReader注解配置读取器，用于对特定注解（如@Service、@Repository）的类进行读取转化成  BeanDefinition 对象，（BeanDefinition 是 Spring 中极其重要的一个概念，它存储了 bean 对象的所有特征信息，如是否单例，是否懒加载，factoryBeanName 等）
③ 实例化ClassPathBeanDefinitionScanner路径扫描器，用于对指定的包目录进行扫描查找 bean 对象





将bean解析为definition。然后存在concurrentHashmap中

之后要取bean的话就从缓存中取，如果缓存中取不到就创建一个对象实例。