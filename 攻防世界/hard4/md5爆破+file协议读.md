# 攻防世界 md5爆破+file协议读

## 思路

- 上来发现这里需要验证码，我猜到是爆破，但是我没尝试
- 太可惜了，应该尝试一下的

## wp

- python写个多线程的脚本爆破，有三种方法

**方法1：**

- 这里最开始，我想按照之前那种多线程的方式写

```python
with concurrent.futures.ThreadPoolExecutor() as executor:
    for i in range(1, 100000):
        executor.submit(test_char, i)
```

- 但是代码在循环中生成大量的任务，并将它们提交给线程池，此时会导致内存占用过多，容易卡死

**方法2：**

```python
tasks = [str(i) for i in range(1, max_num)]
```

- 使用了一个简单的列表推导式（List Comprehension）来生成任务列表，这种方式会一次性生成所有的任务，可能会导致内存占用过高。

**方法3：**

- 下面这个使用生成器动态生成，一次只生成一个任务，可以节约内存。

```python
max_num = 100000
    with concurrent.futures.ThreadPoolExecutor() as executor:
        # 使用生成器动态生成任务
        tasks = (str(i) for i in range(1, max_num))
        # 使用 map 方法并发执行任务
        for _ in executor.map(find_char, tasks):
            pass
```

- 这将生成一个生成器对象，它可以通过迭代器协议（Iterator Protocol）来逐个生成任务。在 ThreadPoolExecutor.map() 方法中，当生成器生成新的任务时，线程池会自动将任务提交给线程池进行执行，直到生成器生成的所有任务都执行完毕。
- **并发是指多个任务在同一时间段内交替执行**

完整代码：

```python
import concurrent.futures
import hashlib

def find_char(i):
    md5 = hashlib.md5()
    md5.update(i.encode('utf-8'))
    encode_string = md5.hexdigest()
    if(encode_string[-6:] == "359475"):
        print(i)
        exit()

    #4be21a

if __name__ == "__main__":
    max_num = 100000
    with concurrent.futures.ThreadPoolExecutor() as executor:
        # 使用生成器动态生成任务
        tasks = (str(i) for i in range(1, max_num))
        # 使用 map 方法并发执行任务
        for _ in executor.map(find_char, tasks):
            pass
```

- 由于不需要使用迭代器中的元素，只需要并发地执行任务，因此使用一个下划线 _ 来占位，并在循环中使用 pass 语句来跳过迭代器中的元素。**也就是**`**find_char**`**函数没有返回值**
- 如果有返回值的话，按照这么写

```python
for result in executor.map(find_char, tasks):
    print(result)
```

- 此时用`file`访问/etc/passwd，发现有回显

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682344089131-c2cc3a7f-7c98-4169-93fc-2298b3bfe77d.png#averageHue=%2389cef1&clientId=u49f41384-2d63-4&from=paste&height=361&id=uce5bd693&originHeight=721&originWidth=2303&originalType=binary&ratio=2&rotation=0&showTitle=false&size=166642&status=done&style=none&taskId=u02259f00-4ad6-451d-91fc-7a4724c1f73&title=&width=1151.5)

- 访问`file:///flag`，但是提示`hack`
- 对`flag`进行url编码(all-characters)，是可以获得flag的

### apache配置文件

:::info
**/etc/apache2/sites-enabled/000-default.conf **是 Apache Web服务器的配置文件之一，它通常用于指定默认的虚拟主机和网站配置。在Debian和Ubuntu系统中，Apache的默认虚拟主机配置文件通常被存储在此目录下，以便系统管理员可以轻松地添加、编辑或删除虚拟主机。
虚拟主机是一种机制，允许在同一物理服务器上托管多个网站，每个网站都可以拥有自己的域名、IP地址、文档根目录和配置。
**000-default.conf **是默认的虚拟主机配置文件名称，它通常包含一些基本的网站配置，例如文档根目录、错误日志位置、访问日志格式等等。系统管理员可以通过编辑该文件来更改默认的网站配置，或添加新的虚拟主机配置。

:::

- 这里发现有个`47852`端口，访问它的网站默认页面`index.php`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682390556122-b3f5061d-f27e-4c22-a84b-018f7da8c686.png#averageHue=%23efefef&clientId=ub8cee2a5-d0f4-4&from=paste&height=116&id=u1320b133&originHeight=231&originWidth=843&originalType=binary&ratio=2&rotation=0&showTitle=false&size=13603&status=done&style=none&taskId=ucf6457b4-9ea5-441c-bbc3-491b57b3b81&title=&width=421.5)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682390627075-2472284b-7935-4ecb-9fde-035dd49cc5d8.png#averageHue=%23fefefe&clientId=ub8cee2a5-d0f4-4&from=paste&height=134&id=ua48034b6&originHeight=268&originWidth=812&originalType=binary&ratio=2&rotation=0&showTitle=false&size=9445&status=done&style=none&taskId=u94603c9a-0514-486d-9ddc-8c097233e73&title=&width=406)

- 这里有命令执行，用`gopher`发送Post包，将`cat /f* > /1`，读取文件

提示：`file:///f*`这样是不可以的，只有命令执行语句才可以用通配符

### url二次编码绕过对flag的过滤

:::info
我发现一个神奇的事情

- 用Hackbar、burpsuite就需要二次编码，符合我想的逻辑
- 但是直接在框里提交就一次编码就行了。
  :::

- 看源码发现用`pref_match`在当前页面过滤了flag，但是我没看到源码
- 如果对`flag`进行二次编码，相当于php解析了一次，但是没有找到flag，所以绕过了
- 但是file协议是可以读url编码后的`flag`的

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1682409262172-e45c6f33-d642-4ec5-9786-2158e52d37d3.png#averageHue=%23738661&clientId=u461406b0-7f78-4&from=paste&height=477&id=u1800dbea&originHeight=954&originWidth=1397&originalType=binary&ratio=2&rotation=0&showTitle=false&size=85781&status=done&style=none&taskId=ue004b027-e23c-44e4-83ea-d2699666171&title=&width=698.5)