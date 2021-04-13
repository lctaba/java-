### final修饰符

被final修饰的变量，只能被赋值一次

```java
void funt(final int i){
    i = 5;
    //报错，不可以赋值，在调用的时候参数已经被赋值了，不可以再赋值
    //赋值可以参考后面的引用传参理解
}
```

### 数组

二维数组每维大小可能不同

### 增强for循环

```java
for(int each:array){}
```

### 方法重载多个参数

```java
void funt(Class... class){
    //参数class是一个Class类的数组
    //类似与python的收集参数
}
```

### this的使用

```java
this()//在构造函数中调用构造函数使用this();
```

### 引用传参

```java
    public Hero(String name,float hp){
        this.name = name;
        this.hp = hp;
    }
 
    //复活
    public void revive(Hero h){
        h = new Hero("提莫",383);
    }
 
    public static void main(String[] args) {
        Hero teemo =  new Hero("提莫",383);
         
        //受到400伤害，挂了
        teemo.hp = teemo.hp - 400;
         
        teemo.revive(teemo);
         
        //问题： System.out.println(teemo.hp); 输出多少？ 怎么理解？
         
    }
```

此时Teemo仍然是死亡的，仔细想清楚传参过程

在main方法中我们声明了一个名为teemo的引用，并给它new了个对象。然后在受到伤害之后此时teemo.hp的值为13，这时调用的teemo.revive(teemo)，问输出的血量是多少？ 第一步，刚才我们new了个对象，所以teemo在栈空间开辟了一个内存，假设地址值为0x666，teemo就指向了0x666这个地址值。然后调用revive方法的时候，把teemo的地址值（0x666）赋给了形参Hero h（记住赋过去的是地址值，而不是对象），这个时候teemo和h就同时指向了栈空间中的0x666，然后题目又说h=new Hero(“提莫”，383)，这一步的意思是，h又new了一个新的对象，然后在栈空间中开辟一个新的内存，假设地址值为0x999，此时h不再指向teemo的0x666了，而是转为指向0x999，这时候h里内存存的是“提莫，383”，但是它是指向0x999的啊！跟teemo指向的0x666有毛关系！

### 类中static关键字

- static变量

所有对象共享一个变量，比如用来记录对象数量

- static方法

可以不用对象访问

### 引用类型转换

- 子类转为父类一定可以。
- 父类如果是由子类转换而来，则可以再转为子类，否则不行。
- 接口类似（类也可以转接口）
- instanceof 用于判断是否为此类或此类的父类

### 重写与隐藏

当父类的引用指向一个子类对象时，执行的对象方法是子类的对象方法，而执行的类方法是父类的类方法（对象方法与类方法都重写了），不推荐对象调用类方法

### 子类构造函数

子类调用构造函数时，一定会调用父类的构造方法。如果父类没有无参构造方法，则在子类构造函数中调用有参父类构造函数或者添加父类无参构造函数

