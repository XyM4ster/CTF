# 攻防世界 Confusion1(难度4) SSTI

## 思路

- 这个题我之前做过，所以有点印象，看这个图和python有关，所以想到SSTI

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682240783923-2a3d2bf4-ac5e-46f6-bf94-ab934244c6ce.png#averageHue=%23d4e0e6&clientId=u4864fe20-f26f-4&from=paste&height=489&id=uf8edd3e6&originHeight=977&originWidth=1380&originalType=binary&ratio=2&rotation=0&showTitle=false&size=311533&status=done&style=none&taskId=uba377df1-51ea-44a6-a881-8b3d49f84dc&title=&width=690)

- 但是我始终没有找到入口，在哪输入，其实我也测试了`/{{7*7}}`,但是我没有仔细看回显，其实在404的报错页面是有回显的

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682240908378-4cbb8ff2-8f61-4926-8509-ca13d5fb5bd9.png#averageHue=%23f7f6f6&clientId=u4864fe20-f26f-4&from=paste&height=367&id=Tcc57&originHeight=734&originWidth=2296&originalType=binary&ratio=2&rotation=0&showTitle=false&size=93550&status=done&style=none&taskId=ub8b9e1ab-dee2-4ef3-9540-a7c591fe27c&title=&width=1148)

- 就差一点！！！！我好气
- 然后就直接上payload就行了

## 总结

- 一定要多看回显的信息，多看包！！！