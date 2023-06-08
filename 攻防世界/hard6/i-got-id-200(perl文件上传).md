# 攻防世界 i-got-id-200(难度6) perl文件上传

## 思路

- 没有思路，烦死了

## wp

[攻防世界-web-i-got-id-200（perl文件上传+ARGV造成任意文件读取和任意命令执行） - zhengna - 博客园](https://www.cnblogs.com/zhengna/p/13344832.html)

- 都是.pl文件，.pl文件都是用perl编写的网页文件
- 尝试之后发现，文件上传界面，可以把传入的文件中的文件内容打印出来。
- 这就需要很了解这种语言的特性
- 猜测后台代码是

```perl
use strict;
use warnings; 
use CGI;
my $cgi= CGI->new;
if ( $cgi->upload( 'file' ) ) { 
    my $file= $cgi->param( 'file' );
     while ( <$file> ) { print "$_"; }
}
```

- 这段代码使用param()方法获取文件名，用了尖括号符号<>，这表示从文件句柄中读取一行文本并赋值给变量$_。然后，print语句将$_打印到标准输出中。因此，这个while循环会一行一行地读取文件内容，直到文件结尾
  :::info
  在给定的代码中，使用$cgi->param('file')来获取上传文件的参数值。

1. 如果上传了多个文件，则**$file只会包含列表中的第一个文件名或文件句柄，而其他文件名或文件句柄则会被忽略。 my $file= $cgi->param( 'file' );这个代码返回的是文件内容**

如果需要处理多个文件，使用列表遍历
my [@files ](/files ) = $cgi->param('file'); 
foreach my $file ([@files) ](/files) ) { 

# 处理每个文件

}
:::

- 因此如果我传入第一个文件的内容是`ARGV`，它读的就是参数中的值，因此构造一下参数，就可以了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29405061/1683686752937-e23d6603-a284-4502-b9c2-1f6048a67af9.png#averageHue=%23f4efef&clientId=u48653028-7dea-4&from=paste&height=567&id=u0b98be5f&originHeight=1134&originWidth=1081&originalType=binary&ratio=2&rotation=0&showTitle=false&size=87237&status=done&style=none&taskId=uc645e10e-51f2-483f-a579-ecd0a732f67&title=&width=540.5)
:::info
/bin/bash表示启用bash命令

-c 选项用于指定要执行的命令
%20是空格，这里表示url的空格
${IFS}是分隔符，默认是\t\n，也就是命令的空格
bash -c ls \|  表示执行ls \ 并用管道(|)把它读到输入流中
:::

## 总结

- `perl`项目的目录位于`/var/www/cgi-bin/`下
- 注意bash命令的使用