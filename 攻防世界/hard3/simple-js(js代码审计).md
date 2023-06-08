# 攻防世界 simple-js(难度3)js代码审计

## 思路

- 知道是看代码，但是我看不下去，所以没做出来
- 其实认真看，这个题挺简单的

## wp

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1681994090360-a9e4cd81-d171-43ef-b92c-9573827de375.png#averageHue=%23fefdfc&clientId=uc95170e7-2377-4&from=paste&height=314&id=u7b43ee46&originHeight=628&originWidth=1935&originalType=binary&ratio=2&rotation=0&showTitle=false&size=99320&status=done&style=none&taskId=u94959de6-e53d-48ab-970f-7c468ca1b4d&title=&width=967.5)

- 审计代码，发现这个输出和输入没什么关系
- 有一个可疑的字符串，把它用python输出一下,得到一串数据

55,56,54,79,115,69,114,116,107,49,50

- 猜测可能是ASCII码值，再用chr函数转换一下，就行了
- 只审计代码，也可以，把tab2改成tab，也行。