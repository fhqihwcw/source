---
title: linux学习-恢复误删除的文件
date: 2016-01-03 20:36:01
categories: linux
tags: [linux,文件误删除]
---

## 基本命令

touch<br>
作用：用来创建文件<br>
语法：touch 文件名<br>


>rm -rf 文件名或目录名

ext4文件系统上删除文件，可以恢复：
>软件是:extundelete


扩展：
>linux文件系统由三部分组成：文件名，inode，block   

查看文件名的inode
>通过stat命令查看inode中包含的内容   


如何避免误删除的文件内容被覆盖？  
>1.卸载需要恢复文件的分区：或以只读的方式挂载。  
>2.注：删除时，只删除了文件名。另外，我们可以从inode中读出文件名的名字，可以通过indode

上传extundelete到linux中：
从windows上传extundelete文件到linux，安装xmanager  v4   或者ＣＲＴ
[root@xuegod63 ~]# rpm -ivh /mnt/Packages/lrzsz-0.12.20-27.1.el6.x86_64.rpm  
安装后，就有了ｒｚ命令和ｓｚ命令
ｒｚ　：　上传windows中的文件到linux
sz ：将linux中的文件传到windows  

解压并安装extundelet
[root@xuegod63 extundelete-0.2.4]# tar jxvf extundelete-0.2.4.tar.bz2 

[root@xuegod63 ~]# cd extundelete-0.2.4
[root@xuegod63 extundelete-0.2.4]# ls
acinclude.m4  autogen.sh   configure     depcomp     LICENSE      Makefile.in  README
aclocal.m4    config.h.in  configure.ac  install-sh  Makefile.am  missing      src
[root@xuegod63 extundelete-0.2.4]# ./configure   #检查系统安装环境
Configuring extundelete 0.2.4
configure: error: Can't find ext2fs library

解决方法：
方法1：[root@xuegod63 Packages]# rpm -ivh ext2fs^C   #按两下tab键。 一般情况，ext2fs就是要安装的软件包的名字开头。如果存在 自动补全

方法2：[root@xuegod63 Packages]# ls *ext2fs*   #查找关键字
方法3：[root@xuegod63 Packages]# ls *2fs*    #查找部分关键字


开始恢复：
方法1：通过inode结点恢复
方法二：通过文件名恢复
方法三：恢复某个目录，如目录a下的所有文件：
方法四：恢复所有的文件  

方法1：
通过inode结点查看被删除的文件名字：
[root@xuegod63 test]# extundelete /dev/sda4 --inode 2
.                                                2
lost+found                                        11
passwd                                            12             Deleted
hosts                                             13             Deleted
a                                                 7313           Deleted

[root@xuegod63 ~]# mkdir test  #创建一个目录使用于存放恢复的数据
 [root@xuegod63 test]# extundelete /dev/sda4 --restore-inode 12
NOTICE: Extended attributes are not restored.
Loading filesystem metadata ... 9 groups loaded.
Loading journal descriptors ... 63 descriptors loaded.
[root@xuegod63 test]# ls 
RECOVERED_FILES
[root@xuegod63 test]# diff /etc/passwd RECOVERED_FILES/file.12  # 没有任何输出，说明一样

方法二，通过文件名恢复
[root@xuegod63 test]# extundelete /dev/sda4 --restore-file passwd

方法三：恢复某个目录，如目录a下的所有文件：
[root@xuegod63 test]# extundelete /dev/sda4 --restore-directory a
[root@xuegod63 test]# tree RECOVERED_FILES/a/
RECOVERED_FILES/a/
├── a.txt
└── b
└── a.txt

下面是原来的目录结构：
[root@xuegod63 ~]# tree /root/sda4-back/a/
/root/sda4-back/a/
├── a.txt
└── b
    ├── a.txt
    ├── c
└── kong.txt

方法四：恢复所有的文件
 [root@xuegod63 test]# extundelete /dev/sda4 --restore-all
[root@xuegod63 test]# tree RECOVERED_FILES/
RECOVERED_FILES/
├── a
│   ├── a.txt
│   └── b
│       └── a.txt
├── hosts
└── passwd

这是删除前的数据：
[root@xuegod63 ~]# tree /tmp/sda4/
/tmp/sda4/
├── a
│   ├── a.txt
│   └── b
│       ├── a.txt
│       ├── c  #空目录
│       └── kong.txt  #空文件
├── hosts
├── lost+found
└── passwd




总结：
方法1：通过inode结点恢复
方法二：通过文件名恢复
方法三：恢复某个目录，如目录a下的所有文件：
方法四：恢复所有的文件
