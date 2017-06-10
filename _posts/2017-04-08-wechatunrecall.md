---
layout:     post
title:      "Python3.5+PyQt5多线程+itchat实现微信防撤回桌面版"
date:        2016-12-06 12:00:00
author:     "孟琦Poet"
tags:
    - Python
    - PyQt5
    - itchat
    - 微信防撤回
    - 多线程
---


前几日在某乎看到有大神用itchat实现了[微信防撤回](https://zhuanlan.zhihu.com/p/25689314?utm_source=zhihu&utm_medium=social)功能，，觉得很有趣，看到下面评论很多人求桌面版，于是乎，手痒便利用清明节几天时间做了一个简陋的桌面程序。废话不多说，先上图位敬。
### 运行环境
win10专业版64位系统1703创造者更新
### 开发环境
*  win10专业版64位系统1703创造者更新
* Python3.5.2
* PyCharm 2016
* PyQt5.7

### 程序演示图

![简陋的程序界面](http://upload-images.jianshu.io/upload_images/1826540-1d642ec5e4bb711a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当有新消息撤回时，会把撤回的消息发送到个人的文件传输助手中，并在桌面程序运行日志中显示，日志可以保存在电脑上。

![文件传输助手回送撤回消息](http://upload-images.jianshu.io/upload_images/1826540-3622236b9b378d63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![电脑运行日志显示撤回消息](http://upload-images.jianshu.io/upload_images/1826540-7ec6dace74cb0304.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

目前只能能显示将消息以文字方式显示，图片和文件会保存在电脑上，可以在程序目录BackUp文件夹中查看。但是无法保存表情包，有时候会出现文件保存不及时的问题，不知道为什么。

#  下载地址
链接：[网页链接](http://pan.baidu.com/s/1hsn5ZRe) 密码：e8bt
  
# weChatThread线程类
之前一直不会python多线程，写这个程序的时候，发现不用多线程会陷入无限未响应状态。于是学了半天python多线程，但是在主函数里写的时候，发现一个问题，Ui主线程和工作线程没有分离，使用itchat等库的时候会堵塞主线程，换句话说**PyQt中子线程不能操作GUI界面**。之前写的多线程仍然属于Ui主线程，是其子线程，所以才造成未响应。
既然知道问题了，那就查资料解决问题，后来，在几篇博客上找到了解决办法
* [PyQt 分离UI主线程与工作线程](http://m.blog.csdn.net/article/details?id=46945011)
* [python pyqt4 PyQT实现了使用QThread后台处理数据](http://blog.chinaunix.net/uid-21961132-id-396744.html)
* [多线程中的信号/槽](http://www.cnblogs.com/txw1958/archive/2011/12/22/Threading-Signals-Slots.html)

然后仿照第一篇博客，重写了QThread类，并借鉴第三篇博客，学会了PyQt多线程中的信号/槽机制，用来传递参数。

### 下面贴代码
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
__author__ = 'memgq'

from PyQt5.QtCore import QThread,pyqtSignal
import itchat
import time,os
import shutil
import re

from itchat.content import *

class weChatWord(QThread):
    getMsgSignal = pyqtSignal(str) #pyqtSignal()必须写在__init__前面，里面可接收的参数类型挺多的，str,list,dict都支持
    def __init__(self,parent=None):
        super(weChatWord,self).__init__(parent)
        self.msg_list=[]
        self.type_list=['Picture','Recording', 'Attachment','Video']



    def clearList(self):
        '''
        清空缓存消息和文件
        :return: 
        '''
        tm_now=time.time()
        len_list=len(self.msg_list)
        if len_list>0:
            for i in range(len_list):
                if tm_now-self.msg_list[i]['msg_time']>121:
                    if self.msg_list[i]['msg_type'] in self.type_list:
                        try:
                            os.remove(".\\BackUp\\"+self.msg_list[i]['msg_content'])
                        except Exception as e:
                            print(e)
                        finally:
                            pass
                else:break
            self.msg_list=self.msg_list[i:]



    def run(self):
        '''
        重写run()函数，
        :return:
        '''
        @itchat.msg_register([TEXT, PICTURE, MAP, CARD, SHARING, RECORDING, ATTACHMENT, VIDEO, FRIENDS], isFriendChat=True,
                      isGroupChat=True)
        def getMsg(msg):
            '''
            注册消息类型，并对不同类型的消息执行不用的操作
            :param msg: 
            :return: 
            '''
            msg_dict={}
            # pprint.pprint(msg)
            msg_id = msg['MsgId']  # 消息ID
            msg_time = msg['CreateTime']
            msg_url=None
            msg_group=""
            if (itchat.search_friends(userName=msg['FromUserName'])):
                if itchat.search_friends(userName=msg['FromUserName'])['RemarkName']:
                    msg_from = itchat.search_friends(userName=msg['FromUserName'])['RemarkName']  # 消息发送人备注
                elif itchat.search_friends(userName=msg['FromUserName'])['NickName']:  # 消息发送人昵称
                    msg_from = itchat.search_friends(userName=msg['FromUserName'])['NickName']  # 消息发送人昵称
                else:
                    msg_from = r"读取发送消息好友失败"
            else:
                msg_group = msg['User']['NickName']
                msg_from = msg['ActualNickName']
            msg_type = msg['Type']
            if msg_type in ['Text', 'Friends','Sharing']:
                msg_content = msg['Text']
                msg_url = msg['Url']
            elif msg_type in self.type_list:
                msg_content=msg['FileName']
                msg['Text'](msg['FileName'])
                shutil.move(msg_content,r'.\\BackUp\\')
            elif msg['Type'] == 'Card':
                msg_content = msg['RecommendInfo']['NickName'] + r" 的名片"
            elif msg['Type'] == 'Map':
                x, y, location = re.search("<location x=\"(.*?)\" y=\"(.*?)\".*label=\"(.*?)\".*",
                                           msg['OriContent']).group(1,
                                                                    2,
                                                                    3)
                if location is None:
                    msg_content = r"纬度->" + x.__str__() + " 经度->" + y.__str__()
                else:
                    msg_content = r"" + location

            msg_dict={'msg_id':msg_id,'msg_time':msg_time,'msg_from':msg_from,'msg_group':msg_group,
                      'msg_content':msg_content,'msg_type':msg_type,'msg_url':msg_url}
            self.msg_list.append(msg_dict)
            self.clearList()


        @itchat.msg_register([NOTE],isFriendChat=True, isGroupChat=True)
        def recall(msg):
            '''
            当消息类型为通知类的时候，查找消息内容是否为撤回消息，如果是，则执行撤回后的防撤回操作
            :param msg: 
            :return: 
            '''
            # pprint.pprint(msg)
            msg_content=msg['Content']
            if re.search(r'\<replacemsg\>\<!\[CDATA\[(.*)撤回了一条消息\]\]\>\<\/replacemsg\>',msg_content):
                msg_note=re.search(r'\<replacemsg\>\<!\[CDATA\[(.*)\]\]\>\<\/replacemsg\>',msg_content).group(1)
                old_msg_id=re.search(r'\<msgid\>([0-9]+)\</msgid\>',msg_content).group(1)
                for each in self.msg_list:
                    if each['msg_id']==old_msg_id:
                        timeArray = time.localtime()
                        otherStyleTime = time.strftime("%Y-%m-%d %H:%M:%S,", timeArray)
                        msg_note = msg_note + '，撤回内容为：' + each['msg_content']
                        if each['msg_group']!='':
                            msg_note = "群组（"+each['msg_group']+")中"+msg_note
                        msg_note=otherStyleTime+msg_note
                        itchat.send(msg_note,toUserName='filehelper')
                        self.msg_list.pop(self.msg_list.index(each))
                        self.getMsgSignal.emit(msg_note)
                        break

        #创建BuckUp文件夹
        if not os.path.exists(".\\BackUp\\"):
            os.mkdir('.\\BackUp\\')
        #启动itchat()    
        itchat.auto_login(hotReload=True)
        itchat.run()
```

# 主程序类
```python
class mainwindowapp(QMainWindow,wechatunrecall.Ui_MainWindow):
    def __init__(self):
        super().__init__()
        self.setupUi(self)
        self.createActions()
        self.createTrayIcon()
        self.pushButton.clicked.connect(self.saveLog)
        self.pushButton_2.clicked.connect(self.clearlog)
        self.pushButton_3.clicked.connect(self.houtai)
        self.trayIcon.activated.connect(self.iconActivated)
        timeArray = time.localtime()
        otherStyleTime = time.strftime("%Y-%m-%d %H:%M:%S", timeArray)
        self.setLog(otherStyleTime+"，程序运行时，请用手机扫描弹出的二维码进行登录，并确保电脑上自带的Window照片查"
                                   "看器可用，撤回的图片文件等可下载附件连同运行日志保存在程序目录下BackUp文件夹中。\n")
        self.weChatBigWord()


    def saveLog(self):
        '''
        保存日志
        :return: 
        '''
        if not os.path.exists(".\\BackUp\\"):
            os.mkdir(".\\BackUp\\")
        timeArray = time.localtime()
        otherStyleTime = time.strftime("%Y-%m-%d%H%M%S", timeArray)
        text=self.textBrowser.toPlainText()
        logPath=".\\BackUp\\"+otherStyleTime+'.txt'
        with open(logPath,'w') as f:
            f.write(text)

    def setLog(self,msg):
        '''
        往运行日志窗口写撤回消息的内容
        :param msg: 
        :return: 
        '''
        self.textBrowser.append(msg)

    def createTrayIcon(self):
        '''
        创建托盘图标，可以让程序最小化到windows托盘中运行
        :return: 
        '''
        self.trayIconMenu=QMenu(self)
        self.trayIconMenu.addAction(self.restoreAction)
        self.trayIconMenu.addSeparator()
        self.trayIconMenu.addAction(self.quitAction)
        self.trayIcon=QSystemTrayIcon(self)
        self.trayIcon.setContextMenu(self.trayIconMenu)
        self.trayIcon.setIcon(QIcon('./media/images/maincion.png'))
        self.setWindowIcon(QIcon('./media/images/maincion.png'))
        self.trayIcon.show()

    def createActions(self):
        '''
        为托盘图标添加功能
        :return: 
        '''
        self.restoreAction=QAction("恢复",self,triggered=self.showNormal)
        self.quitAction=QAction("退出",self,triggered=QApplication.instance().quit)


    def iconActivated(self,reason):
        '''
        激活托盘功能
        :param reason: 
        :return: 
        '''
        if reason in (QSystemTrayIcon.Trigger, QSystemTrayIcon.DoubleClick):
            self.showNormal()


    def houtai(self):
        self.hide()

    def clearlog(self):
        self.textBrowser.clear()


    def weChatBigWord(self):
        '''
        weChatThread类实例化，并启动线程
        :return: 
        '''
        from weChatThread import weChatWord
        self.wcBWThread=weChatWord()
        self.wcBWThread.getMsgSignal.connect(self.setLog)
        self.wcBWThread.start()
```

#程序界面
程序界面仍然由Qtdesigner设计

#后记
第一次尝试多线程编程，并且具体应用到实际项目中去，收获良多。
最近打算学点Django，有没有推荐教程？