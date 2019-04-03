---
layout: post
title: Design Patterns-Singleton Pattern 
date:  2019/4/3 10:35:00    
categories: document
tag: 设计模式

---

* content
{:toc}


设计模式之单例模式

# 思想概述

单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。

**注意：**
    1、单例类只能有一个实例。
    2、单例类必须自己创建自己的唯一实例。
    3、单例类必须给所有其他对象提供这一实例。

单例模式特征

**目的：**保证一个类仅有一个实例，并提供一个访问它的全局访问点。

**解决问题：**一个全局使用的类频繁地创建与销毁。

**应用场景：**1.当您想控制实例数目，节省系统资源的时候。判断系统是否已经有这个单例，如果有则返回，如果没有则创建。2、要求生产唯一序列号。 3、WEB 中的计数器，不用每次刷新都在数据库里加一次，用单例先缓存起来。 4、创建的一个对象需要消耗的资源过多，比如 I/O 与数据库的连接等。 

 **应用实例：**1、一个党只能有一个书记。2、Windows 是多进程多线程的，在操作一个文件的时候，就不可避免地出现多个进程或线程同时操作一个文件的现象，所以所有文件的处理必须通过唯一的实例来进行。 3、一些设备管理器常常设计为单例模式，比如一个电脑有两台打印机，在输出的时候就要处理不能两台打印机打印同一个文件。 

**优点：**1、在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例（比如管理学院首页页面缓存）。2、避免对资源的多重占用（比如写文件操作）。 

**缺点**没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。


**基本的实现思路：**单例模式要求类能够有返回对象一个引用(永远是同一个)和一个获得该实例的方法（必须是静态方法，通常使用getInstance这个名称）。单例的实现主要是通过以下两个步骤：

1. 将该类的构造方法定义为私有方法，这样其他处的代码就无法通过调用该类的构造方法来实例化该类的对象，只有通过该类提供的静态方法来得到该类的唯一实例；
2. 在该类内提供一个静态方法，当我们调用这个方法时，如果类持有的引用不为空就返回这个引用，如果类保持的引用为空就创建该类的实例并将实例的引用赋予该类保持的引用。

**使用场景：**1、要求生产唯一序列号。2、WEB 中的计数器，不用每次刷新都在数据库里加一次，用单例先缓存起来。3、创建的一个对象需要消耗的资源过多，比如 I/O 与数据库的连接等。

# 实现过程

创建一个 SingleObject 类。SingleObject 类有它的私有构造函数和本身的一个静态实例。 
SingleObject 类提供了一个静态方法，供外界获取它的静态实例。SingletonPatternDemo，我们的演示类使用 SingleObject 类来获取 SingleObject 对象。

```
public class SingleObject {
 
   //创建 SingleObject 的一个对象
   private static SingleObject instance = new SingleObject();
 
   //让构造函数为 private，这样该类就不会被实例化
   private SingleObject(){}
 
   //获取唯一可用的对象
   public static SingleObject getInstance(){
      return instance;
   }
 
   public void showMessage(){
      System.out.println("Hello World!");
   }
}
```

写创建一个测试类，写main方法，从 singleton 类获取唯一的对象。
```
public class SingletonPatternDemo {
   public static void main(String[] args) {
 
      //不合法的构造函数
      //编译时错误：构造函数 SingleObject() 是不可见的
      //SingleObject object = new SingleObject();
 
      //获取唯一可用的对象
      SingleObject object = SingleObject.getInstance();
 
      //显示消息
      object.showMessage();
   }
}
//执行程序，输出结果：
//Hello World!
```

# 单例模式的八种写法

### 1.饿汉式（静态常量）[可用]，

```
public class Singleton {

//创建 SingleObject 的一个对象
    private final static Singleton INSTANCE = new Singleton();

//让构造函数为 private，这样该类就不会被实例化
    private Singleton(){}

//获取唯一可用的对象
    public static Singleton getInstance(){
        return INSTANCE;
    }
}
 ```
优点：这种写法比较简单，就是在类装载的时候就完成实例化。避免了线程同步问题
缺点：在类装载的时候就完成实例化，没有达到Lazy Loading的效果。如果从始至终从未使用过这个实例，则会造成内存的浪费。

 
### 2、饿汉式（静态代码块）[可用]

```
public class Singleton {

    private static Singleton instance;

    static {
        instance = new Singleton();
    }

    private Singleton() {}

    public Singleton getInstance() {
        return instance;
    }
}
```

这种方式和上面的方式其实类似，只不过将类实例化的过程放在了静态代码块中，也是在类装载的时候，就执行静态代码块中的代码，初始化类的实例。优缺点和上面是一样的。

### 3、懒汉式(线程不安全)[不可用]


```
public class Singleton {

    private static Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}

```

这种写法起到了Lazy Loading的效果，但是只能在单线程下使用。如果在多线程下，一个线程进入了if (singleton == null)判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例。所以在多线程环境下不可使用这种方式。

### 4、懒汉式(线程安全，同步方法)[不推荐用]

```
public class Singleton {

    private static Singleton singleton;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```
解决上面第三种实现方式的线程不安全问题，做个线程同步就可以了，于是就对getInstance()方法进行了线程同步。
缺点：效率太低了，每个线程在想获得类的实例时候，执行getInstance()方法都要进行同步。而其实这个方法只执行一次实例化代码就够了，后面的想获得该类实例，直接return就行了。方法进行同步效率太低要改进。

### 5、懒汉式(线程安全，同步代码块)[不可用]

```
public class Singleton {

    private static Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                singleton = new Singleton();
            }
        }
        return singleton;
    }
}
```
由于第四种实现方式同步效率太低，所以摒弃同步方法，改为同步产生实例化的的代码块。但是这种同步并不能起到线程同步的作用。跟第3种实现方式遇到的情形一致，假如一个线程进入了if (singleton == null)判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例。

### 6、双重检查[推荐用]
```
public class Singleton {

    private static volatile Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```
Double-Check概念对于多线程开发者来说不会陌生，如代码中所示，我们进行了两次if (singleton == null)检查，这样就可以保证线程安全了。这样，实例化代码只用执行一次，后面再次访问时，判断if (singleton == null)，直接return实例化对象。
优点：线程安全；延迟加载；效率较高。


### 7、静态内部类[推荐用]
```
public class Singleton {

    private Singleton() {}

    private static class SingletonInstance {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonInstance.INSTANCE;
    }
}
```
这种方式跟饿汉式方式采用的机制类似，但又有不同。两者都是采用了类装载的机制来保证初始化实例时只有一个线程。不同的地方在饿汉式方式是只要Singleton类被装载就会实例化，没有Lazy-Loading的作用，而静态内部类方式在Singleton类被装载时并不会立即实例化，而是在需要实例化时，调用getInstance方法，才会装载SingletonInstance类，从而完成Singleton的实例化。
类的静态属性只会在第一次加载类的时候初始化，所以在这里，JVM帮助我们保证了线程的安全性，在类进行初始化时，别的线程是无法进入的。
优点：避免了线程不安全，延迟加载，效率高。

### 8、枚举[推荐用]

```
public enum Singleton {
    INSTANCE;
    public void whateverMethod() {
		return INSTANCE;
    }
}
```
借助枚举来实现单例模式。不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。可能是因为枚举在JDK1.5中才添加，所以在实际项目开发中，很少见人这么写过。
优点：系统内存中该类只存在一个对象，节省了系统资源，对于一些需要频繁创建销毁的对象，使用单例模式可以提高系统性能。
缺点：当想实例化一个单例类的时候，必须要记住使用相应的获取对象的方法，而不是使用new，可能会给其他开发人员造成困扰，特别是看不到源码的时候。

使用场景
- 需要频繁的进行创建和销毁的对象；
- 创建对象时耗时过多或耗费资源过多，但又经常用到的对象；
- 工具类对象；
- 频繁访问数据库或文件的对象。


其实枚举类的方法并没有简单，**要避免反射攻击**：枚举Enum是个抽象类，其实一旦一个类声明为枚举，实际上就是继承了Enum，所以会有（String.class,int.class）的构造器。既然是可以获取到父类Enum的构造器，那你也许会说刚才我的反射是因为自身的类没有无参构造方法才导致的异常，并不能说单例枚举避免了反射攻击。其实不是这样的。而是反射在通过newInstance创建对象时，会检查该类是否ENUM修饰，如果是则抛出异常，反射失败。
下面是正确的处理反射攻击的方式。
```
public enum  SerEnumSingleton implements Serializable {
    INSTANCE;
    private  String content;
    public String getContent() {
        return content;
    }
    public void setContent(String content) {
        this.content = content;
    }
    private SerEnumSingleton() {
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        SerEnumSingleton s = SerEnumSingleton.INSTANCE;
        s.setContent("枚举单例序列化");
        System.out.println("枚举序列化前读取其中的内容："+s.getContent());
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("SerEnumSingleton.obj"));
        oos.writeObject(s);
        oos.flush();
        oos.close();

        FileInputStream fis = new FileInputStream("SerEnumSingleton.obj");
        ObjectInputStream ois = new ObjectInputStream(fis);
        SerEnumSingleton s1 = (SerEnumSingleton)ois.readObject();
        ois.close();
        System.out.println(s+"\n"+s1);
        System.out.println("枚举序列化后读取其中的内容："+s1.getContent());
        System.out.println("枚举序列化前后两个是否同一个："+(s==s1));
    }
}
```

建议多使用枚举类《Effective Java》一书中说：单元素的枚举类型已经成为实现Singleton的最佳方法