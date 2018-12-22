---
layout: post
title: Design Patterns-Factory Pattern 
date:   2018/12/22 14:16:07  
categories: document
tag: 设计模式

---

* content
{:toc}


设计模式之工厂模式

# 思想概述：

工厂模式（Factory Pattern）是 Java 中最常用的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。

抽象工厂模式（Abstract Factory Pattern）是围绕一个超级工厂创建其他工厂。该超级工厂又称为其他工厂的工厂。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。

# 工厂模式特征：

**目的：**定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。

**解决问题：**主要解决接口选择的问题。

**应用场景：**明确地计划不同条件下创建不同实例时。抽象工厂则是，系统的产品有多于一个的产品族，而系统只消费其中某一族的产品。

**方法分类：**工厂模式分为三种，简单工厂模式，工厂方法模式，抽象工厂模式。

**应用实例：**1、您需要一辆汽车，可以直接从工厂里面提货，而不用去管这辆汽车是怎么做出来的，以及这个汽车里面的具体实现。 2、Hibernate 换数据库只需换方言和驱动就可以。3、日志记录器：记录可能记录到本地硬盘、系统事件、远程服务器等，用户可以选择记录日志到什么地方。 4、数据库访问，当用户不知道最后系统采用哪一类数据库，以及数据库可能有变化时。 5、设计一个连接服务器的框架，需要三个协议，"POP3"、"IMAP"、"HTTP"，可以把这三个作为产品类，共同实现一个接口。 6、抽象工厂，QQ 换皮肤，一整套一起换； 7、生成不同操作系统的程序。 

**优点：** 1、一个调用者想创建一个对象，只要知道其名称就可以了。 2、扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以。 3、屏蔽产品的具体实现，调用者只关心产品的接口。4、抽象工厂，当一个产品族中的多个对象被设计成一起工作时，它能保证客户端始终只使用同一个产品族中的对象。

**缺点：**每次增加一个产品时，都需要增加一个具体类和对象实现工厂，使得系统中类的个数成倍增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖。这并不是什么好事。 抽象工厂，产品族扩展非常困难，要增加一个系列的某一产品，既要在抽象的 Creator 里加代码，又要在具体的里面加代码。

**注意事项：**作为一种创建类模式，在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过 new 就可以完成创建的对象，无需使用工厂模式。如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度。抽象工厂需要产品族难扩展，产品等级易扩展。

# 简单工厂模式

1.我们以汽车厂建造小汽车为例，我们创建一个小汽车接口Car以及其实现类BmwCar，BenzCar，AudiCar分别代表宝马，奔驰，奥迪品牌的骑车实体，具体代码如下：
```
public interface Car {

    String getName();
}

public class BmwCar implements Car {
    @Override
    public String getName() {
        return "build BmwCar";
    }
}

public class BenzCar implements Car {
    @Override
    public String getName() {
        return "build BenzCar";
    }
}

public class AudiCar implements Car {
    @Override
    public String getName() {
        return "build AudiCar";
    }
}

```

2.创建简单汽车工厂类，SimpleCarFactory实现小汽车的统一建造，SimpleCarFactory工厂类服装统一建造不同品牌的汽车，代码如下：
```
public class SimpleCarFactory {
    private static String bmwCar = "BmwCar";
    private static String benzCar = "BenzCar";
    private static String audiCar = "AudiCar";

    public Car creatrCar(String carName){
        if (bmwCar.equalsIgnoreCase(carName)){
            return new BmwCar();
        }else if (benzCar.equalsIgnoreCase(carName)){
            return new BenzCar();
        }else if (audiCar.equalsIgnoreCase(carName)){
            return new AudiCar();
        }else {
            return null;
        }
    }
}
```

3.使用该工厂，通过传递类型信息通过工厂来获取实体类的对象。

```
public class SimpleFactoryDemo {

    public static void main(String[] args){
        SimpleCarFactory simpleCarFactory = new SimpleCarFactory();
        Car bmwcar = simpleCarFactory.creatrCar("bmwcar");
        System.out.println(bmwcar.getName());

        Car audiCar = simpleCarFactory.creatrCar("audicar");
        System.out.println(audiCar.getName());

        Car benzCar = simpleCarFactory.creatrCar("benzCar");
        System.out.println(benzCar.getName());


    }
}

//-------------------
//运行结果
//    build BmwCar
//    build AudiCar
//    build BenzCar
```

# 工厂方法模式

定义一套公开标准，然后不同的汽车由不同的厂家生产。宝马工厂生产宝马，奔驰工厂生产奔驰，有自己的个性化定制。

1.首先定义一个统一的小汽车接口Car，一级其实现类以及其实现类BmwCar，BenzCar，AudiCar分别代表宝马，奔驰，奥迪品牌的骑车实体，具体代码如下：
```
public interface Car {

    String getName();
}

public class BmwCar implements Car {
    @Override
    public String getName() {
        return "BmwCar";
    }
}

public class BenzCar implements Car {
    @Override
    public String getName() {
        return "BenzCar";
    }
}

public class AudiCar implements Car {
    @Override
    public String getName() {
        return "AudiCar";
    }
}

```

2.然后定义个统一，公开的工厂接口，CarFactory用来提供标准。宝马工厂按照公开的标准生产，然后自己做了一些定制化生产。奔驰工厂按照公开的标准生产，然后自己做了一些定制化生产。奥迪工厂按照公开的标准生产，然后自己做了一些定制化生产。具体代码如下
```
public interface CarFactory {

    Car createCar();
}

public class BmwCarFactory implements CarFactory {
    @Override
    public Car createCar() {
        return new BmwCar();
    }
}

public class BenzCarFactory implements CarFactory {
    @Override
    public Car createCar() {
        return new BenzCar();
    }
}

public class AudiCarFactory implements CarFactory {
    @Override
    public Car createCar() {
        return new AudiCar();
    }
}
```
3.不同的工厂生产不同的汽车，我们只需提供对应的要求即可，想要什么车就实例什么工厂，调用该工厂的汽车组装方法，获得对应的汽车：
```
public class MethodFactoryDemo {

    public static void main(String[] args){

        CarFactory benzCarFactory = new BenzCarFactory();
        String benzCar = benzCarFactory.createCar().getName();
        System.out.println(benzCar);

        CarFactory audiCarFactory = new AudiCarFactory();
        String audiCar = audiCarFactory.createCar().getName();
        System.out.println(audiCar);

        CarFactory bmwCarFactory = new BmwCarFactory();
        String bmwCar = bmwCarFactory.createCar().getName();
        System.out.println(bmwCar);

    }
}
```

# 抽象工厂模式

抽象工厂模式与工厂方法不同，为了便于区别，这里要多说一些。

咳咳。。敲黑板，划重点啦，注意注意后面才是精华！！！

**抽象工厂模式与工厂方法模式的区别:**抽象工厂模式是工厂方法模式的升级版本，他用来创建一组相关或者相互依赖的对象。他与工厂方法模式的区别就在于，工厂方法模式针对的是一个产品等级结构；而抽象工厂模式则是针对的多个产品等级结构。在编程中，通常一个产品结构，表现为一个接口或者抽象类，也就是说，工厂方法模式提供的所有产品都是衍生自同一个接口或抽象类，而抽象工厂模式所提供的产品则是衍生自不同的接口或抽象类。

在抽象工厂模式中，有一个产品族的概念：所谓的产品族，是指位于不同产品等级结构中功能相关联的产品组成的家族。抽象工厂模式所提供的一系列产品就组成一个产品族；而工厂方法提供的一系列产品称为一个等级结构。我们依然拿生产汽车的例子来说明他们之间的区别。

![](https://i.imgur.com/iANCjnW.png)

在上面的类图中，Car和SUV是两个不同的汽车产品等级结构，而奔驰，宝马，奥迪则是是三个不同的产品族。具体来说，奔驰car是和奥迪宝马car同属于一个产品等级结构，奔驰SUV，宝马SUV，奥迪SUV是属于同一个产品结构；而奔驰car和奔驰SUV是同一个产品族，奥迪car和奥迪SUV属于同一个产品族，宝马car和宝马SUV属于同一个产品族。

明白了等级结构和产品族的概念，就理解工厂方法模式和抽象工厂模式的区别了，如果工厂的产品全部属于同一个等级结构，则属于工厂方法模式；如果工厂的产品来自多个等级结构，则属于抽象工厂模式。

上图中，一个产品结构构建一个工厂，由各自工厂统一创建实体。共有CarFactory和SUVFactory两个工厂，最后再抽象成一个抽象工厂AbstractFactory统一管理工厂。最后又封装一个工厂创造器。对于外界，想要什么结构的车，就获取什么结构的工厂，给工厂传入参数告诉工厂我需要什么样品牌的汽车，让其调创造汽车的方法来创造汽车。

这样是不是很明了啦！！！！具体来看一下实现过程吧。

**具体过层如下：**

1.创建小汽车接口以及实现类：
```
public interface Car {

    String getName();
}

public class BmwCar implements Car {
    @Override
    public String getName() {
        return "BmwCar";
    }
}

public class BenzCar implements Car {
    @Override
    public String getName() {
        return "BenzCar";
    }
}

public class AudiCar implements Car {
    @Override
    public String getName() {
        return "AudiCar";
    }
}
```

2.创建SUV骑车接口及实现类：
```
public interface SUV {
    String getName();
}

public class BmwSUV implements SUV {
    @Override
    public String getName() {
        return "BmwSUV";
    }
}

public class BenzSUV implements SUV {
    @Override
    public String getName() {
        return "BenzSUV";
    }
}

public class AudiSUV implements SUV {
    @Override
    public String getName() {
        return "AudiSUV";
    }
}

```

3.创建抽象汽车工厂以及小汽车工厂子类和SUV汽车工厂子类。

```
public abstract class AbstractFactory {
    public abstract Car createCar(String carName);
    public abstract SUV createSUV(String suvName);
}

public class CarFactory extends AbstractFactory {
    private static String bmwCar = "BmwCar";
    private static String benzCar = "BenzCar";
    private static String audiCar = "AudiCar";

    @Override
    public Car createCar(String carName) {
        if (bmwCar.equalsIgnoreCase(carName)){
            return new BmwCar();
        }else if (benzCar.equalsIgnoreCase(carName)){
            return new BenzCar();
        }else if (audiCar.equalsIgnoreCase(carName)){
            return new AudiCar();
        }else {
            return null;
        }
    }

    @Override
    public SUV createSUV(String suvName) {
        return null;
    }
}

public class SUVFactory extends AbstractFactory {
    private static String bmwSUV = "BmwSUV";
    private static String benzSUV = "BenzSUV";
    private static String audiSUV = "AudiSUV";
    @Override
    public Car createCar(String carName) {
        return null;
    }

    @Override
    public SUV createSUV(String suvName) {
        if (benzSUV.equalsIgnoreCase(suvName)){
            return new BenzSUV();
        }else if (bmwSUV.equalsIgnoreCase(suvName)){
            return new BmwSUV();
        }else if (audiSUV.equalsIgnoreCase(suvName)){
            return new AudiSUV();
        }else {
            return null;
        }
    }
}

```
4.创建一个汽车工厂创造器/生成器类，通过传递结构类型信息来获取car工厂或者SUV工厂。

```
public class AbstractFactoryProvider {

//AbstractFactoryProvider来获取 AbstractFactory，通过传递类型信息来获取实体类的对象。
    public AbstractFactory getFactory(String factory){
        if (factory.equalsIgnoreCase("Car")){
            return new CarFactory();
        }else if (factory.equalsIgnoreCase("SUV")){
            return new SUVFactory();
        }else {
            return null;
        }
    }
}
```

5.对于外界，只需要通过工厂创造器来获取工厂，给工厂传入参数告诉工厂我需要什么样品牌的汽车，让工厂调创造汽车的方法来创造汽车。

```
public class AbstractFactoryDemo {

//使用 FactoryProducer 来获取 AbstractFactory，通过传递类型信息来获取实体类的对象。
    public static void main(String[] args){
        AbstractFactoryProvider abstractFactoryProvider = new AbstractFactoryProvider();
        //获取小汽车工厂
        AbstractFactory carFactory = abstractFactoryProvider.getFactory("car");
        //获取奥迪品牌的小汽车对象
        Car audiCar = carFactory.createCar("audiCar");
        System.out.println(audiCar.getName());
        //获取奔驰品牌的小汽车对象
        Car benzCar = carFactory.createCar("BenzCar");
        System.out.println(benzCar.getName());


        ////获取SUV汽车工厂
        AbstractFactory suvFactory = abstractFactoryProvider.getFactory("suv");
        //获取奥迪品牌的小汽车对象
        SUV audiSUV = suvFactory.createSUV("audiSUV");
        System.out.println(audiSUV.getName());
        //获取奔驰品牌的小汽车对象
        SUV benzSUV = suvFactory.createSUV("BenzSUV");
        System.out.println(benzSUV.getName());
    }

}
```


至此 简单工厂、工厂方法和抽象工厂模式讲解完毕，有不懂之处可以联系我，一起探讨，想获取代码请移步：[想看代码戳我](https://github.com/ZFCC/springCloud)