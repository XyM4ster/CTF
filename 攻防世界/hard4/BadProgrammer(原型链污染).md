# 攻防世界 BadProgrammer(难度5)原型链污染

## 思路

- 这个题也是没什么发现，直接看wp

## wp

- dirsearch扫目录，发现`/static../`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683185712520-da91fec7-4b4d-4fbb-82c3-707f353c7ef1.png#averageHue=%23fafaf9&clientId=u3ca0fbff-d659-4&from=paste&height=336&id=ue14a846a&originHeight=672&originWidth=1481&originalType=binary&ratio=2&rotation=0&showTitle=false&size=87168&status=done&style=none&taskId=u3b2b1ed6-3728-4018-8c6d-c3ec438bf4a&title=&width=740.5)

- 看`package.json`有安装了什么包的信息

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683185807278-95b6e985-3273-4f2e-860d-ba909fdee559.png#averageHue=%23fdf8f8&clientId=u3ca0fbff-d659-4&from=paste&height=288&id=uf3d359e9&originHeight=576&originWidth=904&originalType=binary&ratio=2&rotation=0&showTitle=false&size=37803&status=done&style=none&taskId=u7e48c451-bc81-4889-9e49-d579b0b68f4&title=&width=452)

- 也就是说这个`express-fileupload`可能有问题，直接百度查`exp`
- 查看`app.js`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683185869818-a9b2edb8-4663-4586-a440-108749d10eee.png#averageHue=%23fdfdfc&clientId=u3ca0fbff-d659-4&from=paste&height=398&id=u120a58fc&originHeight=796&originWidth=1507&originalType=binary&ratio=2&rotation=0&showTitle=false&size=96687&status=done&style=none&taskId=ub9bd962c-65d8-4c62-abfd-79efb7c1d51&title=&width=753.5)

- 这里发现一个可以`post`提交的地方
- 用脚本post提交，将flag复制到可以查看的地方，`package.json`中发现项目的路径是`/app`