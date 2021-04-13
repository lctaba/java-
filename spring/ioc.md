## 依赖注入

- 接口注入，set注入和构造器注入。
- 要使用的时候在容器中查找。
- 某些对象与系统运行并无很强的关联，这些对象之间的相互依赖关系也是比较稳定的，一般不会随着应用的运行状态的改变而改变。

#### 容器的表示，容器也成为应用上下文

用两个接口表示容器，BeanFactory和ApplicationContext

- BeanFactory本质上是一个hashmap，key是bean的name，value是bean的实例。提供get和put功能，低级容器
  - 解析配置文件，储存实例
  - 调用get方法的时候，如果map中存在，就拿出来，如果由依赖，就递归调用逐一获取。

- ApplicationContext 高级容器，比BeanFactory支持更多功能。如资源获取，支持多种消息。提供了refresh方法，用于刷新容器，重新加载所有bean。

  

#### bean生命周期

实例化

赋初值：将autowired，resources，value标记的属性进行赋值。（依赖注入）

初始化：实现initializeBean接口，重写afterpropertiesset，这部分的逻辑是程序员自己实现的。

销毁