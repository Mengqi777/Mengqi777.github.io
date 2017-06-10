---
layout:     post
title:      "Python Socket编程：利用SMTP发送MIME协议邮件"
date:  2016-12-06 12:00:00
author:     "孟琦Poet"
tags:
    - Python
    - socket
    - MIME
---
# Socket编程简介
写完程序也还是不理解什么事Socket编程，但在知乎里看到的一个问题里面的回答很不错，这里分享一下：[Socket编程简介](https://www.zhihu.com/question/29637351?sort=created)
##SMTP简介
>SMTP（Simple Mail Transfer Protocol）即[简单邮件传输协议](http://baike.baidu.com/view/576460.htm),它是一组用于由源地址到目的地址传送[邮件](http://baike.baidu.com/view/172138.htm)的规则，由它来控制信件的中转方式。[SMTP协议](http://baike.baidu.com/view/21931.htm)属于[TCP/IP协议簇](http://baike.baidu.com/view/2173338.htm)，它帮助每台[计算机](http://baike.baidu.com/view/3314.htm)在发送或中转信件时找到下一个目的地。通过SMTP协议所指定的[服务器](http://baike.baidu.com/view/899.htm),就可以把E-mail寄到收信人的服务器上了，整个过程只要几分钟。SMTP[服务器](http://baike.baidu.com/view/899.htm)则是遵循SMTP协议的发送[邮件服务器](http://baike.baidu.com/view/54241.htm)，用来发送或中转发出的[电子邮件](http://baike.baidu.com/view/1524.htm)。
它使用由TCP提供的可靠的数据传输服务把邮件消息从发信人的邮件服务器传送到收信人的邮件服务器。跟大多数应用层协议一样，SMTP也存在两个 端：在发信人的邮件服务器上执行的客户端和在收信人的邮件服务器上执行的服务器端。SMTP的客户端和服务器端同时运行在每个邮件服务器上。当一个邮件服 务器在向其他邮件服务器发送邮件消息时，它是作为SMTP客户在运行。
SMTP协议与人们用于面对面交互的礼仪之间有许多相似之处。首先，运行在发送端邮件服务器主机上的SMTP客户，发起建立一个到运行在接收端邮件服务 器主机上的SMTP服务器端口号25之间的TCP连接。如果接收邮件服务器当前不在工作，SMTP客户就等待一段时间后再尝试建立该连接。SMTP客户和服务器先执行一些应用层握手操作。就像人们在转手东西之前往往先自我介绍那样，SMTP客户和服务器也在传送信息之前先自我介绍一下。 在这个SMTP握手阶段，SMTP客户向服务器分别指出发信人和收信人的电子邮件地址。彼此自我介绍完毕之后，客户发出邮件消息。

## MIME简介
阮一峰的文章给了我很多灵感，具体请看[MIME笔记](http://www.ruanyifeng.com/blog/2008/06/mime.html)
##代码
这里以QQ邮箱为例，之前用过126邮箱，用普通的25端口就能发送邮件，QQ邮箱则不可以，所以专门研究了下QQ邮箱
```python
import socket, ssl
import base64

msg = b'\r\n'#发送附件时需要在附件内容开始部分加这个
endmsg = b'\r\n.\r\n'整个邮件发送结束发送这个结束命令
msgsubject = b'Subject: Test E-mail\r\n'#邮件主题
msgtype=b"Content-Type:multipart/mixed;boundary='BOUNDARY'\r\n\r\n"#设置邮件不同部分的分隔标志即'boundary='\r\n\r\n''，不清楚需要几个\r\n，保险起见，我加了俩
msgboundary=b'--BOUNDARY\r\n'#邮件不同部分要设置分隔标志，需要在前面设置的boundary之前加--
msgmailer=b'X-Mailer:mengqi\'s mailer\r\n'#设置的邮件发送端，大概这个意思
msgMIMI=b'MIME-Version:1.0\r\n'#MIMEvision设置，无论是什么版本，这里都写1.0
name='lol.png' #自己定义的文件名
msgfileType=b"Content-type:image/gif;\r\n"传送的文件类型是图片
msgfilename=b"Content-Disposition: attachment; filename='%s'\r\n"%name.encode('utf-8')#字符串需要encode，因为socket只能传送byte（字节码）
msgfileID=b'Content-ID:<lol123>\r\n'#给传送的文件定义的id，在邮件里插入图片时会用到
mailserver = ('smtp.qq.com', 465)#465/587  QQ邮箱有ssl安全认证，25端口发的一般会当成垃圾邮件，服务器直接拦截了，使用传统的465端口，
#可以使用ssl方法发送邮件，现在一般是587端口，需要使用startssl，在python自带的smtplib模块中有startssl方法，还没用过，应该比较方便

clientSocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)#建立套接字
sslclientSocket = ssl.wrap_socket(clientSocket, cert_reqs=ssl.CERT_NONE,ssl_version=ssl.PROTOCOL_SSLv23)#ssl认证的套接字
sslclientSocket.connect(mailserver)#发送套接字连接服务器
recv = sslclientSocket.recv(1024).decode('utf-8')
print('000+ ',recv)
if recv[:3] != '220':
    print('220 reply not received from server.')

heloCommand = b'HELO qq.com\r\n'#类似发出的一个问候
sslclientSocket.send(heloCommand)
recv1 = sslclientSocket.recv(1024).decode('utf-8')
print('111+ ',recv1)
if recv1[:3] != '250':
    print('250 replay not received from server')

loginCommand = b'AUTH login\r\n'#问候完需要认证，这个是发送登陆认证，返回的是base64编码的'username'这个字符串，意思是需要我们提供用户名了
sslclientSocket.send(loginCommand)
recv2 = sslclientSocket.recv(1024).decode('utf-8')
print('222+ ',recv2)

userCommand = b'***********\r\n'#在这里发送用户名，必须是qq邮箱，然后base64,b64encode(b'username')，转换成base64编码，*****指的是转成base64格式后的用户名
sslclientSocket.send(userCommand)
recv3 = sslclientSocket.recv(1024).decode('utf-8')
print('333+ ', recv3)

passwordCommand = b'******\r\n'#在这里发送密码，必须是qq邮箱授权码，需要开启邮箱smtp服务后才能发短信获取，具体方法参见qq邮箱账户设置，然后base64,b64encode(b'password')，转换成base64编码，*****指的是转成base64格式后的授权码
sslclientSocket.send(passwordCommand)
recv4 = sslclientSocket.recv(1024).decode('utf-8')
print('444+ ', recv4)

qq='*******@qq.com'#这里要发送的是邮件发送者的邮箱地址，需要跟上面的一致
mailfromCommand = b'MAIL FROM:<%s>\r\n'%qq.encode()
sslclientSocket.send(mailfromCommand)
recv4 = sslclientSocket.recv(1024).decode('utf-8')
print('555+ ', recv4)


mailtoCommand = b'RCPT TO:<27400000086@qq.com>\r\n'#收件人地址
sslclientSocket.send(mailtoCommand)
recv5 = sslclientSocket.recv(1024).decode('utf-8')
print('666+ ', recv5)

dataCommand = b'DATA\r\n'#发送这个命令，意思是接下来我要发送邮件正文了
sslclientSocket.send(dataCommand)
recv6 = sslclientSocket.recv(1024).decode('utf-8')
print('777+ ', recv6)



sslclientSocket.send(msgsubject)#发送邮件主题
sslclientSocket.send(msgmailer)#发送邮件发送端
sslclientSocket.send(msgtype)#发送邮件类型，如果是文本图片附件html等混合的，
#整体的邮件类型需要是'Content-Type:multipart/mixed'
sslclientSocket.send(b'Content-Transfer-Encoding:7bit\r\n\r\n')#邮件传送的编码格式，不太懂
# sslclientSocket.send(b'This is a multi-part message in MIME format\r\n\r\n')  在网上看到说这里可以发送邮件头部信息，不懂什么意思
'''
发送邮件内容
'''
sslclientSocket.send(b'\r\n\r\n'+msgboundary)#发送分割标志
sslclientSocket.send(b'Content-Type:text/html;\r\n')#text/html类型的，可以插入图片
sslclientSocket.send(b'Content-Transfer-Encoding:7bit\r\n\r\n')#需要7bit格式传输
sslclientSocket.send(b'<img src="cid:lol123">')#插入图片，html代码一样，只不过这个cid必须跟下面那个msgfileID一样
sslclientSocket.send(b'123')

sslclientSocket.send(b'\r\n\r\n'+msgboundary)#发送分割标志
sslclientSocket.send(msgfileType)#发送要插入的图片
sslclientSocket.send(msgfileID)
sslclientSocket.send(msgfilename)
sslclientSocket.send(b'Content-Transfer-Encoding:base64\r\n\r\n')#这里必须base64编码
sslclientSocket.send(msg)
fb=open('D:\\users\\Python\\Socket\\SMTP\\社会主义核心价值观.jpg','rb')
while True:
    filedata=fb.read(1024)
    print(filedata)
    if not filedata:
        break
    sslclientSocket.send(base64.b64encode(filedata))
fb.close()

sslclientSocket.send(b'\r\n\r\n'+msgboundary)#发送分割标志
sslclientSocket.send(b"Content-type:application/octet-stream;\r\n")#发送附件可以有多种格式，具体参见MIME协议内容
sslclientSocket.send(b"Content-Disposition: attachment; filename='123.rar'\r\n")
sslclientSocket.send(b'Content-Transfer-Encoding:base64\r\n\r\n')#必须base64
sslclientSocket.send(msg)
fb=open('D:\\users\\Python\\Socket\\SMTP\\社会主义核心价值观.rar','rb')
while True:
    filedata=fb.read(1024)
    print(filedata)
    if not filedata:
        break
    sslclientSocket.send(base64.b64encode(filedata))
fb.close()
sslclientSocket.send(endmsg)#发送邮件内容完毕命令

quitCommand = b'QUIT\r\n'#发送结束请求
sslclientSocket.send(quitCommand)
recv7 = sslclientSocket.recv(1024).decode('utf-8')
print('888+ ', recv7)

sslclientSocket.close()#断开连接
```

## 收获和结论
网上几乎没有pythonSocket编程发送邮件的内容，也可能我没找到，好多东西是借鉴C语言Socket编程发送邮件和基础的MIME协议写出来的，其实这些功能使用smtplib模块完全可以解决，而且非常完美
我这个只是实现了基本的outlook的基本功能，但也收获不少，贴两张图吧

![程序运行图](http://upload-images.jianshu.io/upload_images/1826540-58dc4fe4c44b0150.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![收到的邮件样式](http://upload-images.jianshu.io/upload_images/1826540-ca107076dfd28b96.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
本次收获如下
* tcp/ip协议三次握手很重要，
* 代码执行速度比网速快的多
* Outlook发送邮件速度慢是有原因的
* 自己造的轮子并没有大神造的好用(纯粹为了完成作业)
* 能不能把实验室网速提快点