# 攻防世界 FlatScience(难度4)SQlite注入

## 思路

- 这个题也是之前做过有点印象
- 需要先扫一下目录，扫出来有`login.php`和`admin.php`
- 测试一下，发现`login.php`界面可以sql注入
- 但是数据库是`sqlite`不是`mysql`
- 我测试的时候一直不行，因为没有**仔细看包的回显信息**，一定要记住这个！！

## wp

- 注入测试之后，发现提示`admin.php`的密码在pdf文件中。
- 这里需要查看源码，发现`sha1`加密
- 用`wget`下载所有pdf
- 写脚本提取每一个单词，判断单词的`Sha1`是否和数据库中的相等从而找到密码

## 脚本

```python
import PyPDF2
import nltk
from nltk.tokenize import word_tokenize
import os
import glob
import hashlib

def get_words(pdf_file):

    # 创建PyPDF2的PdfFileReader对象
    pdf_reader = PyPDF2.PdfFileReader(pdf_file)

    # 获取PDF文件中的所有页面数量
    num_pages = pdf_reader.getNumPages()

    # 创建一个空字符串，用于存储PDF文件中的所有文本zl
    pdf_text = ''

    # 循环遍历PDF文件中的每一页，并将内容添加到pdf_text中
    for page in range(num_pages):
        page_obj = pdf_reader.getPage(page)
        pdf_text += page_obj.extractText()

        # 使用nltk库的word_tokenize函数将pdf_text中的文本拆分为单词
        words = word_tokenize(pdf_text)

        for word in words:
            sha1 = hashlib.sha1()
            salt_word = word + "Salz!"
            sha1.update(salt_word.encode('utf-8'))
            encoded_text = sha1.hexdigest()
            if(encoded_text == '3fab54a50e770d830c0416df817567662a9dc85c'):
                return word
    return 

if __name__ == "__main__":
    current_dir = os.getcwd()

    # 构造一个搜索所有PDF文件的通配符
    pdf_glob = os.path.join(current_dir, '*.pdf')

    # 使用glob.glob函数获取所有PDF文件的文件名列表
    pdf_files = glob.glob(pdf_glob)


    for pdf_file in pdf_files:
        # 打印PDF文件名
        print(get_words(pdf_file))

   
```

## 总结

- 看源码
- 看包