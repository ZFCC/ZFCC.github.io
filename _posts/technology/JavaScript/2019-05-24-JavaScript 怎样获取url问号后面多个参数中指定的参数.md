---
layout: post
title: JavaScript 怎样获取url问号后面多个参数中指定的参数
date: 2019/5/24 11:12:49 
categories: document
tag: JavaScript

---

* content
{:toc}



前一段时间写前端js获取页面url（即地址栏的链接）参数的时候遇到了一些小问题，在网上查了一下，大部分都是获取url "？" 后面的单个参数，几乎没有查到有多个参数时怎样获取指定参数值的例子。今天在这里分享一下在多个参数中获取指定参数的方式。该方法也适用于单个参数获取值。
直接上代码：

```
//获取url中"?"后的指定参数
  var myfunc = function(key) {
			
            var url = window.location.href;//首先获取url
            if (url.indexOf("?") != -1) {    //判断是否有参数
				var strSub =null;                
				var str = url.split("?");//根据？截取url
                var strs = str[1].split("&");//str[1]为获取？号后的字符串，并且根据&符号进行截取，得到新的字符串数组，这个字符串数组中有n个key=value字符串
                for (var i = 0; i < strs.length; i++) {//遍历字符串数组
                    strSub = strs[i].split("=");//取第i个key=value字符串，并用“=”截取获得新的字符串数组 这个数组里面第0个字符是key，第1个字符value
                    if (strSub[0] == key) {//判断第0个字符与该方法的参数key是否相等，如果相等 则返回对应的值value。
                        return strSub[1];
                    }
                }
            }
            return "";
        }
var id =myfunc("id");//调用此方法，获取？后所有参数当中为id参数的值

```

重点解析都在注释里我不多说了，看代码。