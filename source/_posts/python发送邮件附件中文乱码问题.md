---
title: python发送邮件附件中文乱码问题
date: 2021-01-08 08:56:48
tags: ['python']
---

<meta name="referrer" content="no-referrer" />

在用python发邮件时，使用的是如下代码：

参考：https://www.runoob.com/python/python-email.html

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.header import Header
 
sender = 'from@runoob.com'
receivers = ['429240967@qq.com']  # 接收邮件，可设置为你的QQ邮箱或者其他邮箱
 
#创建一个带附件的实例
message = MIMEMultipart()
message['From'] = Header("菜鸟教程", 'utf-8')
message['To'] =  Header("测试", 'utf-8')
subject = 'Python SMTP 邮件测试'
message['Subject'] = Header(subject, 'utf-8')
 
#邮件正文内容
message.attach(MIMEText('这是菜鸟教程Python 邮件发送测试……', 'plain', 'utf-8'))
 
# 构造附件1，传送当前目录下的 test.txt 文件
att1 = MIMEText(open('test.txt', 'rb').read(), 'base64', 'utf-8')
att1["Content-Type"] = 'application/octet-stream'
# 这里的filename可以任意写，写什么名字，邮件中显示什么名字
att1["Content-Disposition"] = 'attachment; filename="test.txt"'
message.attach(att1)
 
# 构造附件2，传送当前目录下的 runoob.txt 文件
att2 = MIMEText(open('runoob.txt', 'rb').read(), 'base64', 'utf-8')
att2["Content-Type"] = 'application/octet-stream'
att2["Content-Disposition"] = 'attachment; filename="runoob.txt"'
message.attach(att2)
 
try:
    smtpObj = smtplib.SMTP('localhost')
    smtpObj.sendmail(sender, receivers, message.as_string())
    print "邮件发送成功"
except smtplib.SMTPException:
    print "Error: 无法发送邮件"
```

但是，添加附件的时候，出现了附件无法附上的问题，甚至发往QQ邮箱，会出现`TCMIME322`类似的附件名称，后查阅资料，发现是中文乱码问题，如果附件设置成英文的话，则不存在这个问题

这是上面的代码，在`filename="test.txt"`这里，如果是中文的话，会出现附件附不上，出现乱码问题
```python
att1 = MIMEText(open('test.txt', 'rb').read(), 'base64', 'utf-8')
att1["Content-Type"] = 'application/octet-stream'
# 这里的filename可以任意写，写什么名字，邮件中显示什么名字
att1["Content-Disposition"] = 'attachment; filename="test.txt"'
message.attach(att1)

```

解决方法：

```python
import base64
fileName = "汇总月报.pdf"
# 用base64对文件名进行编码（b64的参数必须是字节类型，所以要先对文件名进行编码）
bs_fileName = base64.b64encode(fileName.encode('UTF-8'))

# bs_filename是字节类型数据，不能直接和字符串相连，所以要必须要decode。
att1.add_header('Content-Disposition', 'attachment', filename= '=?utf-8?b?' + bs_fileName.decode() + '?=')


```



# base64学习

Email在网络上传输时，采用MIME(Multipurpose Internet Mail Extensions)。邮件传输只能传送US-ASCII字符，邮件中包含的其他字符必须通过一定的编码转换之后才能传输。对于Subject或/和附件名称为中文字符的邮件，有些邮件系统因为缺少编码（字符编码和传输编码）信息，导致乱码情况的发生。下面分析Email系统的两种编码——Base64和Quoted-Printable。

首先需要分清楚字符编码和传输编码：
* 字符编码：UTF-8，GB2312
* 传输编码：Base64，Quoted-Printable


## Email Subject和附件名的表达格式

有了Base64和Quoted-Printable的编码方式，要有一定的格式指示采用的哪种传输编码，同时还要指定编码的字符所采用的字符编码方式。

Email的Subject和附件名的表达格式：
```
<prefix><charset>?<encodeMode>?<encodedContent><suffix>
```

其中，

```

<prefix>             固定为“=?”；
<charset>            为字符编码格式；
<encodeMode>         为传输编码格式：B代表Base64；Q代表Quote-Printable
<encodedContent>     为用encodeMode 编码过的字符编码为charset的字符串
<suffix>             固定为“?=”

```

所以，这就能解释上面的这一串了
```
<prefix><charset>?<encodeMode>?     <encodedContent>      <suffix>
  '=?     utf-8  ?     b      ?' + bs_fileName.decode() +   '?='
```





# 参考文献：

https://blog.csdn.net/thl789/article/details/7815866

https://www.douban.com/group/topic/40848076/

