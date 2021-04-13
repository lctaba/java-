## 静态代理

静态代理很直观，只需要把被代理类的一个对象放入代理类中，那么就可以通过代理类来执行被代理类的方法。

但是静态代理有以下问题：

- 如果每个类都需要一个代理类，那么会造成类的冗余，为了解决这个问题，引入了动态代理模式。

## 动态代理（运行时动态创建）

如果要解决静态代理的问题，必须要解决以下两个子问题：

- 如何根据被加载到内存中的被代理类生成代理对象。
- 如何获取被代理对象中的相应方法。

解决第一个问题：

通过Proxy类提供的**newProxyInstance()静态方法**获取被代理类

以下是Proxy.newProxyInstance()方法的参数解析：

``` java
public static Object newProxyInstance(ClassLoader loader,
                                     Class<?>[] interfaces,
                                     InvocationHandler h)
    //第一个参数：通过类加载器获得被代理对象
    //第二个参数：获取被代理类的所有实现接口对象数组，在方法内部会克隆一份加载进Proxy字节码对象中。
    //第三个参数：调用处理器（InvocationHandler）接口的一个实现类，该类实现的invoke（）方法就是代理对象实际调用的handler。
    //每当代理类调用方法时，这个方法调用就自动转移为Handler类里invoke方法体中的对应扩展方法调用。
```

代理类工厂：

``` java
class ProxyFactory {
    //调用此方法返回一个代理类的对象，解决问题一
    public static Object getProxyInstance(Object obj) {//obj:被代理类的对象
        MyInvocationHandler handler = new MyInvocationHandler();

        handler.bind(obj);

        //返回一个代理类
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), handler);
    }
}
```



解决第二个问题：

每当代理类调用方法时，这个方法调用就自动转移为Handler类（下面的实例是MyInvocationHandler类）里invoke方法体中的对应扩展方法调用。

InvocationHandler接口的实现类：

```java
class MyInvocationHandler implements InvocationHandler {
    private Object obj;//赋值时，需要使用被代理类的对象进行赋值

    public void bind(Object obj) {
        this.obj = obj;
    }

    /**
     * 解决问题二
     * 当我们通过代理类的对象，调用方法a时，就会自动的调用如下的方法：invoke()
     * 将被代理类要执行的方法a的功能就声明在invoke()方法中
     * @param proxy  代理类的对象
     * @param method 代理类对象要调用的方法，这里就是什么方法,是通过反射机制获得的对被代理类方法的描述，即代理对象当前所要执行的方法的描述对象
     * @param args   要调用的方法的参数列表
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        HumanUtil util = new HumanUtil(); // 这个对象用于对方法的修饰

        util.method1();

        //method：即为代理类对象调用的方法，此方法作为被代理类对象要调用的方法。
        //obj就是被代理类的对象，因此需要bind方法将被代理对象绑定进来
        Object returnValue = method.invoke(this.obj, args);

        util.method2();

        //上述方法的返回值就作为当前类中的invoke()方法的返回值。
        return returnValue;
    }
}
```

invoke方法详解：

代理类的invoke()方法调用了参数method的invoke()方法，其具体细节为

``` java
method.invoke(this.obj, args);
//作用就是调用method类代表的方法，其中obj是对象名，args是传入method方法的参数
```

method必须为代理对象已经声明（实现）的接口

如下，注意最后一个代码块的第五行

``` java
interface Human {
    String getBelief();
    void eat(String food);
}
```

``` java
//用作被代理类
class SuperMan implements Human {

    @Override
    public String getBelief() {
        System.out.println("I believe I can fly");
        return "I believe I can fly";
    }

    @Override
    public void eat(String food) {
        System.out.println("我喜欢吃" + food);
    }
}
```

``` java
public class ProxyTest {
    public static void main(String[] args) {
        SuperMan superMan = new SuperMan();
        //proxyInstance：代理类的对象
        //这边必须是Human类型而不能是其他类型，因为Human类型是SuperMan的接口，与SuperMan有相同的接口，否则无法找到被代理对象想要执行的方法
        Human proxyInstance = (Human) ProxyFactory.getProxyInstance(superMan);
        //当通过代理类对象调用方法时，会自动的调用被代理类中的同名的方法
        String belief = proxyInstance.getBelief();
    }
}
```

## 以上是基于接口的动态代理，需要有与被代理类同名方法的接口才能执行，为了克服这个缺点，又有基于子类的动态代理

- CGLIB

为什么用cdlib？因为jdk动态代理只能代理接口，需要有与被代理类实现同样的接口。

为什么jdk只能代理接口？因为代理类继承了Proxy类，由于java是单继承的，所以只能代理接口。

CGLIB用子类进行代理。