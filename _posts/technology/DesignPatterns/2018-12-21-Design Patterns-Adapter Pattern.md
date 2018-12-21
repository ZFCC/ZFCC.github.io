---
layout: post
title: Design Patterns-Adapter Pattern 
date:   2018/12/21 19:50:34   
categories: document
tag: 设计模式

---

设计模式之适配器

#思想概述
适配器模式（Adapter Pattern）是作为两个不兼容的接口之间的桥梁。这种类型的设计模式属于结构型模式，它结合了两个独立接口的功能。
这种模式涉及到一个单一的类，该类负责加入独立的或不兼容的接口功能。

适配器模式是常用的模式之一，其主要意图就是做接口兼容：使得原本由于接口不兼容而不能一起工作的那些类可以一起工作

#适配器特征

**目的：**将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

**解决问题：**主要解决在软件系统中，常常要将一些"现存的对象"放到新的环境中，而新环境要求的接口是现对象不能满足的。

**应用场景：**
在软件开发中，也就是系统的数据和行为都正确，但接口不相符时，我们应该考虑用适配器，目的是使控制范围之外的一个原有对象与某个接口匹配。适配器模式主要应用于希望复用一些现存的类，但是接口又与复用环境要求不一致的情况。比如在需要对早期代码复用一些功能等应用上很有实际价值。适用场景大致包含三类：

- 1、已经存在的类的接口不符合我们的需求； 
- 2、创建一个可以复用的类，使得该类可以与其他不相关的类或不可预见的类（即那些接口可能不一定兼容的类）协同工作；
- 3、在不对每一个都进行子类化以匹配它们的接口的情况下，使用一些已经存在的子类。

**方法分类：**适配器模式分两种，类适配器（继承）和对象适配器（依赖）。广义来说，也有接口适配器。
**应用实例：**手机充电器，变压器，电脑主机电源适配器，tep-c接口转换器等等。

**优点：**  1、可以让任何两个没有关联的类一起运行。 2、提高了类的复用。 3、增加了类的透明度。 4、灵活性好。 

**缺点：**1、过多地使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是 A 接口，其实内部被适配成了 B 接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。 2.由于 JAVA 至多继承一个类，所以至多只能适配一个适配者类，而且目标类必须是抽象类。

#类适配器（继承）
**原理：通过继承来实现适配器功能。**

**应用场景：**
当我们要访问的接口A中没有我们想要的方法 ，却在另一个接口B中发现了合适的方法，我们又不能改变访问接口A，在这种情况下，我们可以定义一个适配器p来进行中转，这个适配器p要实现我们访问的接口A，这样我们就能继续访问当前接口A中的方法（虽然它目前不是我们的菜），然后再继承接口B的实现类BB，这样我们可以在适配器P中访问接口B的方法了，这时我们在适配器P中的接口A方法中直接引用BB中的合适方法，这样就完成了一个简单的类适配器。

**案例：**
以只能播放mp3文件的播放器转为能播放mp4或vlc文件为例（虽然实际生活中我们并不需要这样的转换）
类适配器结构图：

![](https://i.imgur.com/hjzFIhP.png)

1.首先有MediaMP3Player这样一个接口，方法为play(String audioType, String fileName)，MediaMP3Player接口功能只能播放MP3文件，如下代码：
```
public interface MediaMP3Player {
    public void play(String audioType, String fileName);
}
```
2.AdvancedMediaPlayer这个接口，方法有playVlc(String fileName)，playMp4(String fileName)，可以播放MP4和vlc文件，但是我们现在只想用play方法播放MP4文件，
```
public interface AdvancedMediaPlayer {
    public void playVlc(String fileName);
    public void playMp4(String fileName);
}
```

3.AdvancedMediaPlayer方法有两个实现类Mp4Player和VlcPlayer分别实现AdvancedMediaPlayer接口中对应的方法。如下代码：
```
public class Mp4Player implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String fileName) {

    }

    @Override
    public void playMp4(String fileName) {
        System.out.println("Playing Mp4 file. Name:"+ fileName);
    }
}


public class VlcPlayer implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file. Name:"+ fileName);
    }

    @Override
    public void playMp4(String fileName) {

    }
}
```

4。这时我们就建造一个适配器MediaAdapter，用该适配器实现MediaMP3Player接口的play方法，并继承Mp4Player类为父类。在play方法中调用Mp4Player父类的playMp4方法。实现play方法能播放MP4文件的目的。代码如下：
```
public class MediaAdapter extends Mp4Player implements MediaMP3Player {

    //适配器MediaAdapter实现接口A方法，并且直接引用AdvancedMediaPlayer实现类Mp4Player中的合适方法
    //实现简单的适配器
    @Override
    public void play(String audioType, String fileName) {
        super.playMp4(fileName);
    }

}
```
5.最后写一个Demo测试我们的适配器是否能实现目的。
```
public class AdapterPatternMainDemo {

    public static void main(String[] args) {
        MediaMP3Player audioPlayer = new MediaAdapter();
        audioPlayer.play("mp4", "alone.mp4");

    }
}
//------------------------
//运行结果为：
//Playing Mp4 file. Name:alone.mp4
```

# 对象适配器（依赖）

**原理：** 通过组合来实现适配器功能。
**应用场景：** 当我们要访问的接口A中没有我们想要的方法 ，却在另一个接口B中发现了合适的方法，我们又不能改变接口A，在这种情况下，我们可以定义一个适配器p来进行中转，这个适配器p要实现我们访问的接口A，这样我们就能继续访问当前接口A中的方法（虽然它目前不是我们的菜），然后在适配器P中定义私有变量C（对象）（B接口指向变量名），再定义一个带参数的构造器用来为对象C赋值，再在A接口的方法实现中使用对象C调用其来源于B接口的方法。

**案例：** 我们任然以MP3文件播放器转其他MP4或vcl文件播放器为例：
对象适配器结构图：

![](https://i.imgur.com/9GEkznq.png)

1.首先我们有接口MediaMP3Player方法play(String audioType, String fileName)，只能实现播放MP3的功能，代码如下：。
```
public interface MediaMP3Player {
    public void play(String audioType, String fileName);
}
```

2.AdvancedMediaPlayer这个接口，方法有playVlc(String fileName)，playMp4(String fileName)，可以播放MP4和vlc文件，但我们想实现MP3，MP4，VCL文件都能播放。AdvancedMediaPlayer代码如下
```
public interface AdvancedMediaPlayer {
    public void playVlc(String fileName);
    public void playMp4(String fileName);
}

```
3.AdvancedMediaPlayer方法有两个实现类Mp4Player和VlcPlayer分别实现AdvancedMediaPlayer接口中对应的方法。如下代码：
```
public class Mp4Player implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String fileName) {

    }

    @Override
    public void playMp4(String fileName) {
        System.out.println("Playing Mp4 file. Name:"+ fileName);
    }
}


public class VlcPlayer implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file. Name:"+ fileName);
    }

    @Override
    public void playMp4(String fileName) {

    }
}
```
4.然后构建适配器MediaAdapter，使其实现接口MediaMP3Player的方法play，并且在MediaAdapter适配器中定义个私有变量指向AdvancedMediaPlayer接口（依赖），并且定义一个带参数的构造函数，以实现给advancedMusicPlayer赋值。代码如下：

```
public class MediaAdapter implements MediaMP3Player {

    //注入依赖，
    AdvancedMediaPlayer advancedMusicPlayer;

    //定义构造函数
    public MediaAdapter(String audioType){
        if(audioType.equalsIgnoreCase("vlc") ){
            advancedMusicPlayer = new VlcPlayer();
        } else if (audioType.equalsIgnoreCase("mp4")){
            advancedMusicPlayer = new Mp4Player();
        }
    }

    @Override
    public void play(String audioType, String fileName) {
        if(audioType.equalsIgnoreCase("vlc")){
            advancedMusicPlayer.playVlc(fileName);
        }else if(audioType.equalsIgnoreCase("mp4")){
            advancedMusicPlayer.playMp4(fileName);
        }
    }
}

```
5.以上便是我们的适配器，为了能实现我们的播放器，MP3，MP4，VCL文件都能播放，我们再写一个MediaMP3Player接口的实现类MediaMP3PlayerImp，并且注入我们刚写的适配器作为依赖，实现play方法
```
public class MediaMP3PlayerImp implements MediaMP3Player {

    //依赖适配器
    MediaAdapter mediaAdapter;

    @Override
    public void play(String audioType, String fileName) {
        //播放 mp3 音乐文件的内置支持
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing mp3 file. Name: " + fileName);
        }
        //mediaAdapter 提供了播放其他文件格式的支持
        else if (audioType.equalsIgnoreCase("vlc")
                || audioType.equalsIgnoreCase("mp4")) {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);
        } else {
            System.out.println("Invalid media. " +
                    audioType + " format not supported");
        }

    }
}

```
6.最后，写一个demo测试一下我们的适配器
```
public class AdapterPatternMainDemo {

    public static void main(String[] args) {
       MediaMP3PlayerImp audioPlayer = new MediaMP3PlayerImp();

        audioPlayer.play("mp3", "beyond the horizon.mp3");
        audioPlayer.play("mp4", "alone.mp4");
        audioPlayer.play("vlc", "far far away.vlc");
        audioPlayer.play("avi", "mind me.avi");
    }
}

//--------------------
//测试结果：
//    Playing mp3 file. Name: beyond the horizon.mp3
//    Playing Mp4 file. Name:alone.mp4
//    Playing vlc file. Name:far far away.vlc
//    Invalid media. avi format not supported
```

# 接口适配器

**原理：**通过抽象类来实现适配，这种适配稍别于上面所述的适配。

**应用场景：**
当存在这样一个接口，其中定义了N多的方法，而我们现在却只想使用其中的一个到几个方法，如果我们直接实现接口，那么我们要对所有的方法进行实现，哪怕我们仅仅是对不需要的方法进行置空（只写一对大括号，不做具体方法实现）也会导致这个类变得臃肿，调用也不方便，这时我们可以使用一个抽象类作为中间件，即适配器，用这个抽象类实现接口，而在抽象类中所有的方法都进行置空，那么我们在创建抽象类的继承类，而且重写我们需要使用的那几个方法即可。

**案例：** 一个媒体播放器支持很多种文件的播放，但是我们现在只想实现一个只支持RMVB的播放器。
接口适配器结构图：
![](https://i.imgur.com/kIdkkPD.png)

1.有这一个播放器接口AdvancedMediaPlayer，支持RMVB,MP4,VCL文件格式播放，如代码：
```
public interface AdvancedMediaPlayer {

   void playVlc(String fileName);
 
   void playMp4(String fileName);

   void playRmvb(String fileName);

}
```

2.目标是实现只能播放RMVB文件的播放器，构建适配器MediaAdapter（定义为抽象的类），实现接口AdvancedMediaPlayer，并实现所有方法（不做任何操作，保持方法body为空），具体如代码：
```
public abstract class MediaAdapter implements AdvancedMediaPlayer {

   public void playVlc(String fileName) {}
   

   public void playMp4(String fileName) {}
   

   public void playRmvb(String fileName) {}

}
```

3.创建适配器MediaAdapter的子类RmvbPlayerChild（及继承适配器），只实现其中的playRmvb方法，如下代码：

```
public class RmvbPlayerChild extends MediaAdapter {

   @Override
   public void playRmvb(String fileName) {
    System.out.println("Playing Rmvb file. Name:"+ fileName);
   }

}
```

4.完成任务，写一个demo测试，

```
public class MainDemo {

    public static void main(String[] args){
        RmvbPlayerChild rmvbPlayer = new RmvbPlayerChild();
        rmvbPlayer.playRmvb("text.rmvb");
    }
}
//--------------------
//测试结果：
//Playing Rmvb file. Name:text.rmvb
```

至此适配器模式，讲解完毕，代码请移步：[想看代码戳我](https://github.com/ZFCC/springCloud)