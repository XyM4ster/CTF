# CTF
Here are some summarized CTF ideas.

![](https://cdn.nlark.com/yuque/0/2023/jpeg/29405061/1684055051552-21b4d5bd-ce76-42c2-9d00-79758dc55ef8.jpeg)


## 有提示的地方

`console`、查看源码、Git上找项目名称

## linux的关键目录

- 如果`/etc/passwd`中有`/bin/bash`
- 访问`/home/www/.bash_history`可以看到执行过的历史命令

## 语言或者框架

### .pl

- 找它的关键特征，如后缀，版本号
- 结合具体给出的信息，在网上找相关的漏洞
- 如给出后缀是`.pl`，存在文件上传界面，查`perl的文件上传`

### JSESSIONID

- java框架，加载`struct2`的相关目录

## fuzz测试

## 存在文件包含，可以查看源码

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684117085992-fa58d411-00a5-4f4d-91e1-5530a533619c.png#averageHue=%23fdfafa&clientId=u692b80b7-f57b-4&from=paste&height=324&id=u3fd0519f&originHeight=647&originWidth=1374&originalType=binary&ratio=2&rotation=0&showTitle=false&size=70955&status=done&style=none&taskId=ua1102a26-0071-4a11-84d8-7a048c5c484&title=&width=687)

## 找到可以交互的地方

- 有可能出现在包的cookie处
- 404页面处
- 一切地方，**总之仔细看包**

## sql注入题

### 解题步骤

- 一般是有一个`register.php`和一个`login.php`
- 正常注册账号密码，登录进去

#### 找注入点

:::info

1. 一切我输入的东西，之后显示出来了，都说明存在交互，也就是都可能存在注入
2. 如果没有回显，可能需要盲注
   :::

#### 注入点是登陆之后的信息

> **攻防世界 难度5 unfinish**

- 在注册界面是输入`email、username、password`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683366455492-950529a6-6c85-4443-969a-687077564044.png#averageHue=%23fdfcfc&clientId=uf3d3d679-8c5f-4&from=paste&height=381&id=u812acb41&originHeight=761&originWidth=1609&originalType=binary&ratio=2&rotation=0&showTitle=false&size=48979&status=done&style=none&taskId=u76307dd2-56b8-4754-af63-8153b90164e&title=&width=804.5)

- 这里发现输入只需要输入`email、password`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683366514601-85587276-7c46-41d4-b7d6-521d19fde33a.png#averageHue=%23e0c4a3&clientId=uf3d3d679-8c5f-4&from=paste&height=226&id=ud5546633&originHeight=452&originWidth=548&originalType=binary&ratio=2&rotation=0&showTitle=false&size=6568&status=done&style=none&taskId=ue5fbb362-0526-4dd6-b42d-3214096c62f&title=&width=274)

- 登录之后，发现显示了用户名，所有交互的界面，只有这里

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683366543989-a00f2dfb-39ca-4c30-9d9e-7b6e12fbd0ce.png#averageHue=%23363b4c&clientId=uf3d3d679-8c5f-4&from=paste&height=171&id=u6b9444ab&originHeight=341&originWidth=395&originalType=binary&ratio=2&rotation=0&showTitle=false&size=39030&status=done&style=none&taskId=uada80384-331f-40d2-a600-5a830bb0813&title=&width=197.5)

- 所有可能在username处存在注入

#### 测试可用的字符

:::info

1. 首先测试的时候要多试，到底过滤了没，比如遇到过如果注册成功包返回302，没成功返回200的。要用Burp试、hackbar试、登录框试
   :::

- fuzz测试可用的字符

#### 命令闭合

##### insert 语句

- 比如注册时候填的信息是被insert到数据库中的

```sql
insert into user('username','password') values('$username','$password')
```

- 就可以用Mysql的运算闭合
- 输入

```sql
0'+select hex(database())+'0
```

- 之后就可以用`ascii + substr`爆破具体的信息

### addslashes

- addslashes后，在存入数据库时，并不会加入反斜杠，也就是你写啥就存啥
- 但是当你从数据库中再取出来用的时候，这时候我之前构造的sql语句就会执行
- **所以重点关注addslashes后，什么从数据库中取出来
