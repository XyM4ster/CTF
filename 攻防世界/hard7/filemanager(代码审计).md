# 攻防世界 filemanager(难度7)

## 思路

- 扫目录看到有源码泄露
- 代码审计，不知道咋做
- (这个时候我刚和dragon研究明白`addslashes`)

## addslashes

![ea8cf6943033e6df86b27eaae7bd3fc.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684230616292-77dc4809-854e-4ad4-9afa-2d1293c744aa.png#averageHue=%237f7f7e&clientId=u9da0a18a-01b1-4&from=paste&height=703&id=u3cb04883&originHeight=1405&originWidth=2632&originalType=binary&ratio=2&rotation=0&showTitle=false&size=261105&status=done&style=none&taskId=ue92939d3-bc38-4ccd-b4a6-2e393f57645&title=&width=1316)

- addslashes后，在存入数据库时，并不会加入反斜杠，也就是你写啥就存啥
- 但是当你从数据库中再取出来用的时候，这时候我之前构造的sql语句就会执行
- **所以重点关注addslashes后，什么从数据库中取出来了，并放在了SQL语句中，因为这样就可以执行我构造的命令了**

## wp

- 查看源码

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684232061472-65af63d3-87ed-4f18-b4ad-25eaa3348456.png#averageHue=%23251f1e&clientId=u9da0a18a-01b1-4&from=paste&height=472&id=u3ec2a865&originHeight=944&originWidth=2332&originalType=binary&ratio=2&rotation=0&showTitle=false&size=173405&status=done&style=none&taskId=uc84e62b4-6582-45d5-804f-565065ff59b&title=&width=1166)

- 发现这里会把存入的扩展名取出来，也就是这个`oldname、extension`是我可以构造的
- 这题显然是`getshell`，关键是我如何把一个后缀改成`.php`
- **后缀相同咋重命名？**
- **除非后缀为空。（卧槽，像是狄大人破案）**
- 显然我可以把上传的文件扩展名构造成空的。但是此时这个文件的文件名是1.txt.txt
- 如果我直接改1.txt.txt--->1.php
- 也就是此时`file_exists('1.txt')`，但是 并不存在`1.txt`的文件
- 所以我需要再上传一个1.txt，此时能过判断，再把`1.txt.txt`改成1.php，此时没有之前的后缀了，就改成功了

## 总结

- 找到`addslashes`后，又取出来放在sql语句中的地方
- 还要很耐心才可以取得最后的flag