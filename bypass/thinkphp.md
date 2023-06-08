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