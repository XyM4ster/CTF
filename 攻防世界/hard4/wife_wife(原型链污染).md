# 攻防世界 wife_wife(难度4) 原型链污染

## 思路

**思路1：**

- 这个题只有登录，注册的界面
- 发现注册的地方，有isAdmin，要填邀请码，还不需要爆破
- 显然需要是isAdmin,才能获取flag
- 所以思路就是得把isAdmin改成true
- 那怎么改值，不会

**思路2：**

- 我以为下载的图片会有什么问题，但其实并没有

## wp

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682045715577-b030353f-b4fd-401e-bb84-51a3e5c108bd.png#averageHue=%23c6c4b2&clientId=u3005aeb6-fb14-4&from=paste&height=337&id=u7e3d989d&originHeight=674&originWidth=1358&originalType=binary&ratio=2&rotation=0&showTitle=false&size=361679&status=done&style=none&taskId=u962f3bf2-0345-46f6-9fff-52862808eda&title=&width=679)

- Object.assign() 是一个 JavaScript 函数，用于将一个或多个源对象的属性复制到目标对象中。它的语法如下：

```
Object.assign(target, ...sources)
```

- 其中，target 表示目标对象，sources 表示一个或多个源对象

```javascript
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

Object.assign(target, source);

console.log(target); // { a: 1, b: 4, c: 5 }
```

- 在上面的示例中，target 对象的属性 b 被源对象 source 的属性 b 覆盖了，属性 c 则被复制到了 target 对象中。
- 所以在注册的时候，添加一个`"__proto__":{"isAdmin":"true"}`