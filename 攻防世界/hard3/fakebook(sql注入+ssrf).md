# 攻防世界 fakebook(难度3)sql注入+ssrf

## 思路

- 这个题我之前做过，现在做还是不会，感觉现在明白点了

## wp

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681957475689-5ad73f96-19d2-4ba5-ae12-8d5bd8db7e75.png#averageHue=%23fefdfd&clientId=uc5536b36-62d0-4&from=paste&height=400&id=u11d79f28&originHeight=799&originWidth=2075&originalType=binary&ratio=2&rotation=0&showTitle=false&size=75771&status=done&style=none&taskId=u9e6cf0c4-f9dd-4b66-ad63-45e9974d0e2&title=&width=1037.5)

- 注册之后，点击注册的用户，发现可能存在sql注入
- 用sqlmap扫，没扫出来，于是手动注册
- 内联注释绕过，发现是把数据序列化存入了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681957633874-fd8d1d36-2680-40c3-9dca-e36bf8c859b2.png#averageHue=%23fefefd&clientId=uc5536b36-62d0-4&from=paste&height=225&id=uedbf6dc5&originHeight=450&originWidth=2385&originalType=binary&ratio=2&rotation=0&showTitle=false&size=53495&status=done&style=none&taskId=u1abdd94b-b56a-426b-a516-b15f2cc38ae&title=&width=1192.5)

- 扫描后台，发现源码，这里有curl_exec，如果我把data的内容换了，是不是就可以读取flag.php了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681957666862-067fe424-11c7-4bfc-b774-c8ed1aca1302.png#averageHue=%23f2f1f0&clientId=uc5536b36-62d0-4&from=paste&height=354&id=u2330be84&originHeight=708&originWidth=950&originalType=binary&ratio=2&rotation=0&showTitle=false&size=34399&status=done&style=none&taskId=uc8fab84c-49b6-4f50-8af0-278536afbe0&title=&width=475)

- 通过union select 1,2,3,'序列化数据'，查看源码获得flag

## 总结

- 看到`curl_exec`想到file协议
- sql手动注入
- union select改数据