---
layout: post
title:  String类的trim()方法之不能消除的空格
date: 2019/2/23 9:22:50 
categories: problem
tag: 踩过的坑

---

* content
{:toc}


在开发过程中，发现java后台接收一串数字字符串，后面带个空格，代码中利用String.trim()方法对传入的字符串数据进行空格清除，结果发现清除不掉，从debug的数据中还能发现字符串结尾带着一个空格。

去网上查了一下，发现String.trim()的方法不能清除全角空格，只能清除半角空格，什么是全角和半角：见下图，搜狗输入法中默认是半角输入，设置为全角输入的方式，再输入的字符就是全角，这时输入的空格 Unicode码是“\u00A0”, 不能被trim清除！

![](https://i.imgur.com/O0xyJr3.png)


下面是写个main方法实际操作测试trim函数对全角空格的无发清除。

```
public class TestMain {

    public static void main(String[] args) {
        String halfStr = "123456 ";
        String fullStr = "123456　";

        System.out.println(halfStr.trim()+"半角长度："+halfStr.trim().length());
        System.out.println(fullStr.trim()+"全角长度："+fullStr.trim().length());
    }
}
//测试结果：
//123456半角长度：6
//123456　全角长度：7
```

![](https://i.imgur.com/GF9ZbVC.png)

上图中可以看出 trim函数确实不能清除全角空格！

下面给出解决方式：
```
public class TestMain {

    public static void main(String[] args) {
        String halfStr = "123456 ";
        String fullStr = "123456　";

        System.out.println(halfStr.trim()+"半角长度："+halfStr.trim().length());
        System.out.println(fullStr.trim()+"全角长度："+fullStr.trim().length());

        //解决方式：把全角空格转为半角空格在利用trim函数进行清除。
        String str = fullStr.replaceAll("　"," ");
        //或者fullStr.replaceAll("\\u00A0","");
        System.out.println(str.trim()+"全角转为半角后的长度："+str.trim().length());
    }
}

//   结果：
//    123456半角长度：6
//    123456　全角长度：7
//    123456全角转为半角后的长度：6
```

![](https://i.imgur.com/WtRvRKX.png)