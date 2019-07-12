---
title: Linux命令学习
date: 2019-05-21 20:53:30
tags: Linux命令
categories: Linux
---
> 每天学习一个命令
> author: @Tiffanysbear 

1、mkdir：新建文件夹

2、mv：移动或者重命名文件，用来为文件或目录改名、或将文件或目录移入其它位置。

    - mv source dest

<!-- more -->

3、tar: [tar命令](https://www.runoob.com/linux/linux-comm-tar.html)

```
-A或--catenate：新增文件到以存在的备份文件；
-B：设置区块大小；
-c或--create：建立新的备份文件；
-C <目录>：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。
-d：记录文件的差别；
-x或--extract或--get：从备份文件中还原文件；
-t或--list：列出备份文件的内容；
-z或--gzip或--ungzip：通过gzip指令处理备份文件；
-Z或--compress或--uncompress：通过compress指令处理备份文件；
-f<备份文件>或--file=<备份文件>：指定备份文件；
-v或--verbose：显示指令执行过程；
-r：添加文件到已经压缩的文件；
-u：添加改变了和现有的文件到已经存在的压缩文件；
-j：支持bzip2解压文件；
-v：显示操作过程；
-l：文件系统边界设置；
-k：保留原有文件不覆盖；
-m：保留文件不被覆盖；
-w：确认压缩文件的正确性；
-p或--same-permissions：用原来的文件权限还原文件；
-P或--absolute-names：文件名使用绝对名称，不移除文件名称前的“/”号；
-N <日期格式> 或 --newer=<日期时间>：只将较指定日期更新的文件保存到备份文件里；
--exclude=<范本样式>：排除符合范本样式的文件。
```
常用的tar命令有：

```
01-.tar格式
解包：[＊＊＊＊＊＊＊]$ tar xvf FileName.tar
打包：[＊＊＊＊＊＊＊]$ tar cvf FileName.tar DirName（注：tar是打包，不是压缩！）

02-.gz格式
解压1：[＊＊＊＊＊＊＊]$ gunzip FileName.gz
解压2：[＊＊＊＊＊＊＊]$ gzip -d FileName.gz
压 缩：[＊＊＊＊＊＊＊]$ gzip FileName

03-.tar.gz格式
解压：[＊＊＊＊＊＊＊]$ tar zxvf FileName.tar.gz
压缩：[＊＊＊＊＊＊＊]$ tar zcvf FileName.tar.gz DirName

04-.bz2格式
解压1：[＊＊＊＊＊＊＊]$ bzip2 -d FileName.bz2
解压2：[＊＊＊＊＊＊＊]$ bunzip2 FileName.bz2
压 缩： [＊＊＊＊＊＊＊]$ bzip2 -z FileName

05-.tar.bz2格式
解压：[＊＊＊＊＊＊＊]$ tar jxvf FileName.tar.bz2
压缩：[＊＊＊＊＊＊＊]$ tar jcvf FileName.tar.bz2 DirName

06-.bz格式
解压1：[＊＊＊＊＊＊＊]$ bzip2 -d FileName.bz
解压2：[＊＊＊＊＊＊＊]$ bunzip2 FileName.bz

07-.tar.bz格式
解压：[＊＊＊＊＊＊＊]$ tar jxvf FileName.tar.bz

08-.Z格式
解压：[＊＊＊＊＊＊＊]$ uncompress FileName.Z
压缩：[＊＊＊＊＊＊＊]$ compress FileName

09-.tar.Z格式
解压：[＊＊＊＊＊＊＊]$ tar Zxvf FileName.tar.Z
压缩：[＊＊＊＊＊＊＊]$ tar Zcvf FileName.tar.Z DirName

10-.tgz格式
解压：[＊＊＊＊＊＊＊]$ tar zxvf FileName.tgz

11-.tar.tgz格式
解压：[＊＊＊＊＊＊＊]$ tar zxvf FileName.tar.tgz
压缩：[＊＊＊＊＊＊＊]$ tar zcvf FileName.tar.tgz FileName

12-.zip格式
解压：[＊＊＊＊＊＊＊]$ unzip FileName.zip
压缩：[＊＊＊＊＊＊＊]$ zip FileName.zip DirName

13-.lha格式
解压：[＊＊＊＊＊＊＊]$ lha -e FileName.lha
压缩：[＊＊＊＊＊＊＊]$ lha -a FileName.lha FileName

14-.rar格式
解压：[＊＊＊＊＊＊＊]$ rar a FileName.rar
压缩：[＊＊＊＊＊＊＊]$ rar e FileName.rar     

```

