---
layout:     post
title:      "使用IDEA+Maven+Embedded Jetty+Jersey构建Restful Web Service并打包成一个jar包发布"
date:  2017-08-05 12:00:00
author:     "孟琦Poet"
tags:
    - Jetty
    - Java
---

## 一、简要介绍
最近做的项目用到了嵌入式Jetty当服务器，并用Jersey来构建Restful api，看了老师的项目文件发现还有pom.xml文件，才知道Maven。
但因为不是一个组的老师，而且那个老师貌似前端精通的多一点，Maven什么的也不是很了解，从老师那里学的东西也不是很多。
因为项目相关，最后还是自己Google各种资料，一点一滴从零开始学习。国内关于嵌入式Jetty的资料真的少，大部分都是翻译官方文档，
很难找到完整的案例。因此，想在这里把自己学到的关于Embedded Jetty、Jersy的知识记录下来，希望提能给想入坑的提供点经验，少走点弯路。


## 二、环境搭建

文中介绍的东西都是基于**JDK1.8**、**Maven3**、**IDEA2017.2**，*Jetty*版本为**9.4.3.v20170317**，*Jersey*版本为**2.25.1**。



## 二、环境搭建
---

| 所需环境 | 推荐工具 |
| ------------- |:-------------:| -----:|
|Python编程工具|*PyCharm 2016.2*|
|Python版本|*Python 3.5*|
|PyQt版本|*PyQt5* |
|界面设计| *QtDesigner*|
|打包程序| *pyinstaller*|
* 1、官网下载安装Python3.5
* 2、官网下载安装PyCharm
* 3、可以在PyCharm打开setting>Project Interpreter>点击加号搜索PyQt5直接安装，其他安装方式请自行搜索
* 4、下载Qt5.7安装，自带QtDesigner和QtCreator
**注意事项**
网上搜索许多PyQt教程，他们的PyQt5中一般自带QtDesigner程序，不知道为什么我的没有，所以只好下载完整的Qt5.7安装包
* 5、PyInstaller支持Python3，cx_Freeze暂不支持

## 三、测试案例
---
使用1M、20M、100M的txt文档，1M、100M的doc文档，50M的docx文档，一个小型的数字文档测试对数字分词的准确率

![第一次次测试文档](http://upload-images.jianshu.io/upload_images/1826540-4202a35306bbbf95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![第二次测试文档](http://upload-images.jianshu.io/upload_images/1826540-145e88487ee50456.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四、UI设计
***
* 1、在PyCharm中添加External Tools中添加QtDesigner和PyUIC工具
打开PyCharm>File>Settings>Tools>External Tools，单击＋号添加工具，出现下图界面
Name：自己定义
Group：External Tools
Program：QtDesigner.exe所在位置
Working directory：$ProjectFileDir$
![QtDesigner](http://upload-images.jianshu.io/upload_images/1826540-993b0cd29b182ba3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其他配置如上
Parameters：`-m PyQt5.uic.pyuic  $FileName$ -o $FileNameWithoutExtension$.py`
![PyUIC](http://upload-images.jianshu.io/upload_images/1826540-68a808015e1d5e09.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
配置完成后会在Tools>External Tools 中显示出来添加的工具
![External Tools](http://upload-images.jianshu.io/upload_images/1826540-78d40a1fca66d09a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



***
* 2、启动QtDesigner，新建Main Window窗体
**启动QtDesinger时，确保电源选项高级设置中可切换动态显卡全局设置为最佳性能，我的win10系统A卡显卡切换成最大化性能将会导致无法启动QtDesinger，具体原因不清楚**
![最佳性能](http://upload-images.jianshu.io/upload_images/1826540-8a4297e46ae4b9c1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Qt5.7中集成QtCreator、QtDesinger，二者都可以创建UI文件，设计界面，本文使用后者，前者功能更强大，有兴趣的同学可以尝试
![QtDesinger](http://upload-images.jianshu.io/upload_images/1826540-06ad2c0d9f6d9c3c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
创建Main Window后，可以在左侧看到具体的窗口插件，Qt支持多种小插件，我还不怎么会使用。将插件拖到中间主体窗口，调整布局即可完成UI设计，右侧显示的是窗体属性和信号槽设计区，使用Qt的信号槽机制可以完成多种逻辑操作。Ctrl+r组合可以预览界面。
![UI设计](http://upload-images.jianshu.io/upload_images/1826540-fbf99648b193334b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我自己设计的原始UI已经删除，下图是成品样子
![UI成品](http://upload-images.jianshu.io/upload_images/1826540-ffede10c42670a15.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


***
* 3、UI设计完成后，直接使用PyUIC工具生成.py文件，在PyCharm中选中ui文件，然后点击运行PyUIC即可，以下代码自动生成。
```python
# -*- coding: utf-8 -*-
# Form implementation generated from reading ui file 'untitled.ui'
#
# Created by: PyQt5 UI code generator 5.7
#
# WARNING! All changes made in this file will be lost!
from PyQt5 import QtCore, QtGui, QtWidgets
class Ui_mainWindow(object):
    def setupUi(self, mainWindow):
        mainWindow.setObjectName("mainWindow")
        mainWindow.resize(521, 555)
        icon = QtGui.QIcon()
        icon.addPixmap(QtGui.QPixmap("C:\\Python35\\Lib\\site-packages\\cx_Freeze\\samples\\unti\\1.png"), QtGui.QIcon.Normal, QtGui.QIcon.Off)
        mainWindow.setWindowIcon(icon)
        self.centralwidget = QtWidgets.QWidget(mainWindow)
        self.centralwidget.setObjectName("centralwidget")
        self.label = QtWidgets.QLabel(self.centralwidget)
        self.label.setGeometry(QtCore.QRect(30, 510, 451, 16))
        self.label.setText("")
        self.label.setObjectName("label")
        self.textBrowser_2 = QtWidgets.QTextBrowser(self.centralwidget)
        self.textBrowser_2.setGeometry(QtCore.QRect(230, 80, 251, 401))
        self.textBrowser_2.setObjectName("textBrowser_2")
        self.horizontalLayoutWidget = QtWidgets.QWidget(self.centralwidget)
        self.horizontalLayoutWidget.setGeometry(QtCore.QRect(20, 0, 482, 80))
        self.horizontalLayoutWidget.setObjectName("horizontalLayoutWidget")
        self.horizontalLayout = QtWidgets.QHBoxLayout(self.horizontalLayoutWidget)
        self.horizontalLayout.setContentsMargins(0, 0, 0, 0)
        self.horizontalLayout.setObjectName("horizontalLayout")
        self.pushButton = QtWidgets.QPushButton(self.horizontalLayoutWidget)
        self.pushButton.setObjectName("pushButton")
        self.horizontalLayout.addWidget(self.pushButton)
        self.pushButton_2 = QtWidgets.QPushButton(self.horizontalLayoutWidget)
        self.pushButton_2.setObjectName("pushButton_2")
        self.horizontalLayout.addWidget(self.pushButton_2)
        self.pushButton_3 = QtWidgets.QPushButton(self.horizontalLayoutWidget)
        self.pushButton_3.setObjectName("pushButton_3")
        self.horizontalLayout.addWidget(self.pushButton_3)
        self.pushButton_4 = QtWidgets.QPushButton(self.horizontalLayoutWidget)
        self.pushButton_4.setObjectName("pushButton_4")
        self.horizontalLayout.addWidget(self.pushButton_4)
        self.pushButton_5 = QtWidgets.QPushButton(self.horizontalLayoutWidget)
        self.pushButton_5.setObjectName("pushButton_5")
        self.horizontalLayout.addWidget(self.pushButton_5)
        self.pushButton_6 = QtWidgets.QPushButton(self.horizontalLayoutWidget)
        self.pushButton_6.setObjectName("pushButton_6")
        self.horizontalLayout.addWidget(self.pushButton_6)
        self.pushButton_7 = QtWidgets.QPushButton(self.centralwidget)
        self.pushButton_7.setGeometry(QtCore.QRect(41, 220, 75, 23))
        self.pushButton_7.setObjectName("pushButton_7")
        self.lineEdit_2 = QtWidgets.QLineEdit(self.centralwidget)
        self.lineEdit_2.setGeometry(QtCore.QRect(40, 260, 161, 20))
        self.lineEdit_2.setObjectName("lineEdit_2")
        self.layoutWidget = QtWidgets.QWidget(self.centralwidget)
        self.layoutWidget.setGeometry(QtCore.QRect(40, 139, 161, 61))
        self.layoutWidget.setObjectName("layoutWidget")
        self.verticalLayout_2 = QtWidgets.QVBoxLayout(self.layoutWidget)
        self.verticalLayout_2.setContentsMargins(0, 0, 0, 0)
        self.verticalLayout_2.setObjectName("verticalLayout_2")
        self.label_2 = QtWidgets.QLabel(self.layoutWidget)
        self.label_2.setObjectName("label_2")
        self.verticalLayout_2.addWidget(self.label_2)
        self.lineEdit = QtWidgets.QLineEdit(self.layoutWidget)
        self.lineEdit.setObjectName("lineEdit")
        self.verticalLayout_2.addWidget(self.lineEdit)
        self.progressBar = QtWidgets.QProgressBar(self.centralwidget)
        self.progressBar.setGeometry(QtCore.QRect(40, 490, 441, 23))
        self.progressBar.setProperty("value", 24)
        self.progressBar.setObjectName("progressBar")
        mainWindow.setCentralWidget(self.centralwidget)

        self.retranslateUi(mainWindow)
        self.pushButton_6.clicked.connect(mainWindow.close)
        QtCore.QMetaObject.connectSlotsByName(mainWindow)

    def retranslateUi(self, mainWindow):
        _translate = QtCore.QCoreApplication.translate
        mainWindow.setWindowTitle(_translate("mainWindow", "词频统计"))
        self.pushButton.setText(_translate("mainWindow", "打开文档"))
        self.pushButton_2.setText(_translate("mainWindow", "统计词频"))
        self.pushButton_3.setText(_translate("mainWindow", "批量查询"))
        self.pushButton_4.setText(_translate("mainWindow", "清除所有"))
        self.pushButton_5.setText(_translate("mainWindow", "保存结果"))
        self.pushButton_6.setText(_translate("mainWindow", "退出程序"))
        self.pushButton_7.setText(_translate("mainWindow", "查询"))
        self.label_2.setText(_translate("mainWindow", "输入查询单词"))
```
添加图标`icon.addPixmap(QtGui.QPixmap(path), QtGui.QIcon.Normal, QtGui.QIcon.Off)`
UI中加入六个`PushButton`组成第一排基本操作按钮
一个`Label`提示输入查询单词
另一个`Label`在最下方显示统计单词耗费的时间
UI右侧是一个`Text Browser`显示统计结果
最下方还有一个进度条`ProgressBar`


## 五、主程序编写
##一、打开文件
```python
#打开文件
filename_tup=QFileDialog.getOpenFileName(self,'选择文件')
if filename_tup==('', ''): #点击打开文件按钮但未选择文件，为防止闪退，设置pass
        pass           
    elif filename_tup :
        self.sword_dic={} #初始化字典
        self.progressBar.show() #显示进度条
        while self.completed<50: #设置进度条速度，先加载50%，然后读入文件，最后再加载%50
            self.completed+=0.0001
            self.progressBar.setValue(self.completed)
```
##二、读取文件，存入字典
```python
#读取文件，存入字典
doctype = re.findall(r'\..+', filename_tup[0]) #使用正则表达式找出文件类型名
start = time.clock() #设置开始时间点
if doctype==['.txt']:
    with open(filename_tup[0],'r',encoding='utf-8') as f:
        for line in f: #如果是txt文档打开后逐行读入
            words_list=[]
            words_list = re.findall(r'\d+\.\d+|[a-z0-9A-Z]+', str(line).lower()) #使用正则表达式找出每一行的单词和数字，存入列表
            for i in words_list: 
                 if (i not in self.sword_dic):  #如果列表中的单词不存在于字典中，将单词存入字典，初始value置1
                     self.sword_dic[i] = 1
                 else:  #如果列表中的单词存在于字典中，将单词存入字典，value加1
                     self.sword_dic[i] = self.sword_dic[i] + 1
elif doctype==['.docx'] or doctype==['.wps']: #如果是docx文件，使用docx模块读取
    doc = docx.Document(filename_tup[0])
    pc = doc.paragraphs
    for each in pc:
        words_list = []
        words_list = re.findall(r'\d+\.\d+|[a-z0-9A-Z]+', str(each.text).lower())
        for i in words_list:
            if (i not in self.sword_dic):
                self.sword_dic[i] = 1
            else:
                self.sword_dic[i] = self.sword_dic[i] + 1
     doc.save(filename_tup[0])
end=time.clock()  #结束时间
while self.completed<100:  #加载剩余%50进度条
    self.completed=self.completed+0.0001
    self.progressBar.setValue(self.completed)
time.sleep(0.3) 
self.progressBar.hide() #0.3秒后隐藏进度条
self.label.setText('导入成功  耗时:'+str(end-start))  #在下方的label中显示消耗时间
```
##三、统计词频
**将字典中的单词按照key值(即单词)降序排序，如不考虑词形变换还原情况，直接输出结果即可**
```
dict=sorted(self.sword_dic.items(), key=lambda d: d[0], reverse=False) 
```

**以下程序是进行简单的词形还原，仅考虑后缀为`houzhui = ['s', 'es', 'ed', 'd', 'ing']`的情况，不具有参考性，具体的还原过程可以使用NLTK模块中的`lemmatizer`语法，详情参考[ZMonster's Blog](http://www.zmonster.me/)关于词干提取词形还原的文章 **
> http://www.zmonster.me/2016/01/21/lemmatization-survey.html

```python
for k, v in dict: #将key和value分别存入两个列表
    k_list.append(k)
    v_list.append(v)
dict.clear()
i = 0
k_len = len(k_list) #len出列表长度
houzhui = ['s', 'es', 'ed', 'd', 'ing']
while i < (k_len - 1): #从第一个单词开始进行词形还原
    if len(k_list[i]) > 2: #还原的都是长度大于2的单词
        for each in houzhui: #对于后缀列表中的每个后缀，如果单词加上后缀后等于后一个单词，则将后一个单词变为前一个单词，
                             #如第三个单词'egg'+'s'==第四个单词'eggs'，将第四个'eggs'变为'egg'
            if k_list[i + 1] == k_list[i] + each:
                k_list[i + 1] = k_list[i]
    i = i + 1
for num in k_list: #变换完成后，再次统计词频，存在key，value两个列表中
    i = k_list.index(num)
    while i < k_len - 1 and num == k_list[i + 1]:
        v_list[i] = v_list[i] + v_list[i + 1]
        k_list.pop(i + 1)
        v_list.pop(i + 1)
        k_len = k_len - 1
i=0
k_len=len(k_list)
self.dic={}
while i<k_len:  #key、value列表转为字典
    self.dic[k_list[i]]=v_list[i]
    i=i+1
self.sword_dic={}
self.sword_dic = sorted(self.dic.items(), key=lambda d: d[1], reverse=True)
for k, v in self.sword_dic: #在'textBrowser'中插入统计结果
    self.textBrowser_2.insertPlainText((str(k) + ' 出现 ' + str(v) + ' 次\n'))
```
##四、查询单词频率
```python
temp=self.lineEdit.text().lower()  #获取linEdit中输入的单词
if temp in self.dic:
    self.lineEdit_2.setText(temp+' 出现 '+str(self.dic[temp])+' 次')
else:
    self.lineEdit_2.setText('未查找到')
```
##五、批量查询
```python
filename_tup = QFileDialog.getOpenFileName(self, '选择文件')
if filename_tup==('', ''):
    pass
elif filename_tup :
    with open(filename_tup[0], 'r', encoding='utf-8') as f:
        for line in f:  #将要查询的单词统一放在txt文档中，读取稳定将单词写入查询列表
            words_list = []
            words_list = re.findall(r'\d+\.\d+|[a-z0-9A-Z]+', str(line).lower())
            for each in words_list:
                query_word.append(each)
for k in query_word:  #对列表中的每个单词进行查询并输出结果
    if k in self.dic:
        self.textBrowser_2.insertPlainText((str(k) + ' 出现 ' + str(self.dic[k]) + ' 次\n'))
    else:
        self.textBrowser_2.insertPlainText((str(k) + '不存在\n'))
```
##六、保存查询结果
```python
 filename=QFileDialog.getSaveFileName(self,'文件保存','D:/','Text Files (*.txt)')  #PyQt5中保存文件的语句
if filename==('', ''):
    pass
elif filename:
    fn=open(filename[0],'w')
    for k, v in self.sword_dic:
        fn.writelines((str(k) + ' occur ' + str(v) + ' times\n'))
    fn.close()
```
##七、主函数和程序入口
```python
#主函数
def main():
    app=QApplication(sys.argv)
    win=ExampleApp()
    win.show()
    app.exec_()

#程序入口
if __name__ == '__main__':
    main()
```
##八、打包程序
cmd下使用`pyinstaller -F -w -i 图标名.ico 文件名.py`命令进行打包
-F 打包成一个exe文件
-i  图标
-w 使用窗口，无控制台
打包过程中遇到了一个找不到crt,runtime,dll之类的问题，具体记不清楚了，大概是因为缺少的dll放在了win10系统的`C:\Windows\WinSxS`中，把这个文件夹加入系统变量中后打包成功
##九、测试程序
![100M测试文档](http://upload-images.jianshu.io/upload_images/1826540-072f60797366df43.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![文件读取中](http://upload-images.jianshu.io/upload_images/1826540-ce06e48a7f557172.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![导入文档消耗时间](http://upload-images.jianshu.io/upload_images/1826540-f16f6d5d3cc6a497.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![统计结果](http://upload-images.jianshu.io/upload_images/1826540-52492b6d4a70982e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![结果对比](http://upload-images.jianshu.io/upload_images/1826540-73734d7b771098e8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![数字统计](http://upload-images.jianshu.io/upload_images/1826540-e786ea72564be647.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##总结
这个程序是我们软件工程课的作业，本文提供的代码不是最终版，最终版源码文件因为放在C盘不小心删除，只保存了这一版，所以如有错误请指教。
因为选择的数据结构不理想，所以在读取大文件的时候，速度会很慢，100M的txt文件大概需要20s，正常使用字典树可以控制在10s以内，有兴趣的同学可以尝试下