---
layout: post
title: Python imaplib自动下载来自某个发件人的邮件附件
tags:
  - python
  - email
  - imaplib
---

工作中遇到某个发件人账号不定时发送一些文件，需要把它们全部下载下来批量处理。用python的imaplib来处理，主要参考这篇[文章](https://zhuanlan.zhihu.com/p/32814371)。

折腾了一圈发现，

`resp, mails = conn.search(None,'FROM', 'someone')`
    
这种方式需要服务器支持才可以，不幸的是阿里云企业邮箱不支持这种方式。

另外，使用的过程中发现，一些来自其他发件人的邮件有很大的附件，

`resp, data = conn.fetch (mails[0].split()[len(mails[0].split())-1],'(RFC822)')`
   
这种方式把所有的附件内容都读取一遍，速度比较慢，而且容易造成socket error。

可以先读每封邮件RFC822.HEADER的内容，判断收件人，然后再读全部邮件内容。这样速度上也快很多。


```ruby
from email.parser import BytesParser, Parser
from email.policy import default
typ, data = conn.fetch(num,'(RFC822.HEADER)')
headers = Parser(policy=default).parsestr(data[0][1].decode('utf-8'))
content = headers['from'].split()[-1]
if content=='<xxx@xxx.com>':
    typ, data = conn.fetch(num, '(RFC822)')
```

是以为记。
