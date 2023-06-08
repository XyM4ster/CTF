# 攻防世界 comment(难度7) git源码补全

## 思路

- 上来有个登录密码的界面，就是让你爆破
- 但是我没看出来

## wp

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684052499655-0dea7cee-355f-45c0-a3ec-c87d3b4ae98e.png#averageHue=%231fc3c8&clientId=u9c604c00-420e-4&from=paste&height=288&id=u0866bee5&originHeight=576&originWidth=1181&originalType=binary&ratio=2&rotation=0&showTitle=false&size=15582&status=done&style=none&taskId=u6935c695-112a-435e-9bda-66dc65b6cf9&title=&width=590.5)

- 这种有提示的界面就是爆破，且一般就是数字
- 写个脚本，爆破出来是666
- 扫目录发现是git源码泄露，用Githack下载，但是下载后的源码不全

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684053798952-5d88cc10-27eb-496f-a88b-221e78764b03.png#averageHue=%23292b34&clientId=u9c604c00-420e-4&from=paste&height=369&id=ucc089665&originHeight=738&originWidth=887&originalType=binary&ratio=2&rotation=0&showTitle=false&size=77161&status=done&style=none&taskId=u4d939151-8cc0-431d-9747-9887cd124b8&title=&width=443.5)

- 这里明显write，comment后应该有东西，因为提交的页面抓包是`write`
- 在console处，也可以发现

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684054069979-63002c5b-8437-4c92-93c6-4866ae0fb11d.png#averageHue=%23d2d9d7&clientId=u9c604c00-420e-4&from=paste&height=237&id=u01c6f6f7&originHeight=473&originWidth=1911&originalType=binary&ratio=2&rotation=0&showTitle=false&size=75722&status=done&style=none&taskId=ueab78649-9275-4aa0-bf8b-7ef1470fdff&title=&width=955.5)

### git源码恢复

:::info
git log --reflog 
是 Git 中的一条命令，它可以显示 Git 引用日志，包括**所有分支的提交历史记录**和 HEAD 的更改历史记录。
引用日志只记录本地仓库的更改历史记录，而不记录远程仓库的更改历史记录。因此，如果你需要查看远程仓库的更改历史记录，需要使用其他 Git 命令。
git reset --hard 
**git reset --hard 是 Git 中的一条命令，它可以将当前分支重置为指定的提交**，并丢弃工作目录和暂存区中的任何更改。
当你运行 git reset --hard 命令时，Git 首先将 HEAD 指针移动到指定的提交。这意味着当前分支现在指向指定的提交，之后该提交之后对分支所做的任何更改都不再可用。然后，它将工作目录和暂存区重置为与指定提交时分支状态相匹配的状态。这意味着所有在指定提交后对工作目录或暂存区中的文件所做的更改都将被丢弃，并且文件将被恢复到该提交时的状态
:::

- 运行`git log --reflog`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684054220882-b01d43e7-5b19-4f85-be56-4f712e09cbeb.png#averageHue=%23292b35&clientId=u9c604c00-420e-4&from=paste&height=303&id=u422abd7b&originHeight=605&originWidth=1421&originalType=binary&ratio=2&rotation=0&showTitle=false&size=121731&status=done&style=none&taskId=u54c977aa-82bb-4617-aa90-bd69b214512&title=&width=710.5)

- 这里表示最近的提交是一个合并提交（Merge），它将两个父提交（bfbdf21 和 5556e3a）合并成一个新的提交（e5b2a24）。
- 恢复到这个状态`git reset --hard e5b2a2443c2b6d395d06960123142bc91123148c`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684054677194-216ce875-f790-43ca-906d-93d8a509b901.png#averageHue=%232e3039&clientId=u9c604c00-420e-4&from=paste&height=455&id=u53e33c99&originHeight=910&originWidth=1021&originalType=binary&ratio=2&rotation=0&showTitle=false&size=151477&status=done&style=none&taskId=ufe5f253c-bd24-4ac0-8a38-31450b18213&title=&width=510.5)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684054724128-b9f8cc20-0574-459c-912e-818eb3cc969f.png#averageHue=%232e3039&clientId=u9c604c00-420e-4&from=paste&height=447&id=u0c310889&originHeight=893&originWidth=1070&originalType=binary&ratio=2&rotation=0&showTitle=false&size=148916&status=done&style=none&taskId=u5e7b6e2e-6026-42f8-8321-970d8650955&title=&width=535)

- 这个我就不太懂了，wp说catgory没过滤
- 这里就可以用单引号闭合

```sql
category=1',content=(select database()),/*&content=*/#
```

- 注意这里要多行注释，中间执行不了用括号
- 但是整了一圈发现数据库里没什么东西，而且也不可写，我就没招了
- 查看`/etc/passwd`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684055477182-69395e46-d60b-4612-b9b9-7498374ba416.png#averageHue=%23f4f2f2&clientId=u9c604c00-420e-4&from=paste&height=316&id=ue494eebf&originHeight=632&originWidth=1676&originalType=binary&ratio=2&rotation=0&showTitle=false&size=219680&status=done&style=none&taskId=u5a5b02e9-af81-47f0-9c4c-09d725f93c1&title=&width=838)

- 可以看到www用户使用了bash操作，查看base的历史

```sql
select load_file('/home/www/.bash_history')
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684055599199-5bfdf7ff-1655-47bf-94da-6c0021d0e860.png#averageHue=%23fefdfd&clientId=u9c604c00-420e-4&from=paste&height=326&id=u1c7b4efd&originHeight=651&originWidth=1826&originalType=binary&ratio=2&rotation=0&showTitle=false&size=31208&status=done&style=none&taskId=u12438766-c26d-4100-bda2-1978e07369e&title=&width=913)

- 这里它先cd到`tmp`目录，解压了`html.zip`，又把所有的东西复制到了`/var/www`目录下，此时删除`/var/www/html`下的`.DS_Store`
- 说明之前的`html`目录下有`.DS_Store`
- load_file加载文件，并解密

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1684056432492-1871d694-759b-466e-b2d0-ac8795c28f75.png#averageHue=%23fbf9f9&clientId=uf113ce1f-6e32-4&from=paste&height=267&id=u2fe91a4a&originHeight=533&originWidth=2879&originalType=binary&ratio=2&rotation=0&showTitle=false&size=76516&status=done&style=none&taskId=ue40d64a6-a11c-4162-a92e-73aaed4e5fb&title=&width=1439.5)

- load_file加载文件即可