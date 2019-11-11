---
title: 使用Python发送邮件
date: 2019-11-11 11:12:27
tags:
---
# 使用Python发送邮件

之前写了一个小工具从远程服务器下载文件，然后通过附件方式发送给自己的邮箱，中间用到了Python里跟邮件相关的两个标准库`email`和`smtp`，使用也并不复杂。

```python
import smtplib
import ssl
from email import encoders
from email.mime.base import MIMEBase
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from typing import List, Tuple

from app.settings import config

context = ssl.create_default_context() # 初始化ssl上下文


def make_message(sender_email: str, receiver_email: str, subject: str,
                 content: str,
                 attachments: List[Tuple[str, bytes]]) -> MIMEBase:
    message = MIMEMultipart() # 创建一个Message对象并设置邮件的基本信息
    message['From'] = sender_email
    message['To'] = receiver_email
    message['Subject'] = subject
    message['Bcc'] = receiver_email

    message.attach(MIMEText(content, 'plain'))

    for name, binary_content in attachments: # 添加附件
        attachment = MIMEBase("application", "octet-stream")
        attachment.set_payload(binary_content)
        encoders.encode_base64(attachment)
        attachment.add_header(
            "Content-Disposition",
            f"attachment; filename= {name}",
        )
        message.attach(attachment)

    return message


def send_email(receiver_email: str, subject: str, content: str,
               attachments: List[Tuple[str, bytes]]):
    with smtplib.SMTP_SSL(config.host, config.port, context=context) as server: # 配置smtp发送服务器
        sender_email, password = config.email, config.password
        server.login(sender_email, password) # 登录邮箱账户
        message = make_message(sender_email, receiver_email, subject, content,
                               attachments)
        server.send_message(message, sender_email, receiver_email)
```