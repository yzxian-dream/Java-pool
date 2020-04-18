## 一、Linux的常用命令：

#### 1.**切换目录命令**

`cd app`：切换到app目录。
`cd ..`：切换到上一层目录。
`cd /`：切换到系统根目录。
`cd ~`：切换到用户主目录。
`cd -`：切换到上一个所在的目录。

#### 2.**列出文件列表**

`ls [参数] [路径或文件名]`：用来显示当前目录下的内容。
`ls`显示当前目录下的内容，不包括隐藏文件，显示的格式的为横向输出，观看不便。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311212548501.png)
`ls -l，缩写成ll`：竖向显示当前目录下的内容，不包括隐藏文件(在linux中以“.”开头)，这个命令我们很常用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311212735388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMTM4MjA2,size_16,color_FFFFFF,t_70)

 `ls -a`：用来显示当前目录所有内容，包括隐藏文件，在linux中以“ **.**” 开头的文件都是隐藏文件。

`ls -h`:列出指定路径下的所有文件/文件夹的名称，以列表的形式并且在显示文档大小的时候以**可读性较高的形式显示**

####    3.**创建目录和移除目录**

  ` mkdir app`在当前目录下创建app目录。
  `mkdir -p app1/app2`级联创建目录，即在当前目录下创建app1目录，同时在app1目录下创建app2目录。
  `rmdir app`删除当前目录下app的目录，如果app目录中存在其他目录或文件，是无法删除的。

####    4.浏览文件

  `cat [选项] 文件名`：用来一次性显示整个文件的内容，还可以将多个文件连接起来显示，它常与重定向符号配合使用，合适于文件内容少的情况。

 `more和less`：一般用于显示文件内容超过一屏的内容，并且提供翻页的功能。more比cat强大，提供分页显示的功能，less和more更强大，提供翻页，跳转，查找等等命令，而且more和less都支持用空格显示下一页，按键B显示上一页。

`more`：使用B键和空格进行上下翻页，回车显示下一行数据，按q键退出查看。

`less`：使用B键和空格进行上下翻页，回车显示下一行数据，并且也可以使用Pgup和PgDn进行上下翻页，按q键退出查看。
`tail`：该命令实在实际使用过程中使用最多的一个命令，它的功能是用来显示文件后几行的内容。

`tail -10 test.txt`：查看当前目录下test.txt文件后10行的数据。
`tail -f test.txt`：可以动态查看文件内容，使用Ctrl+c结束查看

#### 5.文件操作

`rm [选项] 文件`：删除文件。
`rm a.txt`：删除当前目录下a.txt文件，删除需要用户确认，y确认删除，n不删除。
`rm -f a.txt`：不询问直接删除内容。
`rm -r aaa`：删除需要用户确认，递归删除aaa目录，如果aaa目录下有内容，也一并删除。

`rm -rf aaa`：不询问用户，直接递归删除。不用
`rm -rf /*`：删除所有文件，删库，不用
`cp`：该命令可以将文件从一处复制到另一处，一般在使用该命令时将一个文件复制成另一个文件或复制到某目录时，需要制定源文件名与目标文件名或目录。

`cp a.txt b.txt`：将a.txt复制为b.txt文件。
`cp a.txt ../`：将a.txtt文件复制到上一层目录。
`mv命令`：移动或重命名。

`cp a.txt aaa/c.txt`：将a.txt复制到aaa文件夹并命名为c.txt。

`mv命令`：剪切移动或重命名。
`mv a.txt ../`：将a.txt文件移动（剪切）到上一层目录中。
`mv a.txt b.txt`：将a.txt文件重命名为b.txt。

`tar`：命令，它能够将用户所指定的文件或目录打包成一个文件，**但不做压缩**，一般Linux上常用的压缩方式就选用tar将许多文件打包成一个文件，再以gzip压缩命令压缩成xxx.tar.gz（或称为xxx.tgz）的文件，常用参数如下：
`-c`：创建一个新tar文件。

`-v`：显示运行过程的信息。
`-f`：指定文件名。
`-z`：调用gzip压缩命令进行压缩。
`-t`：查看压缩文件的内容。
`-x`：解开tar文件。

`tar -cvf test.tar ./*`：将指定目录下内容打包成test.tar。
`tar -zcvf xxx.tar.gz ./*`：将指定目录下的内容打包并压缩成test.tar.gz。
`tar -xvf xxx.tar`：解压。

`tar -zxvf xxx.tar.gz -C /usr/aaa`：将指定目录下的压缩文件解压解压到/usr/aaa中。

`find`：该命令用于查找符合条件的文件。
`find / -name "ins* -ls"`：查找文件名称是以ins开头的文件。
`find / -user root -ls`：查找用户root的目录。
`find / -perm -777 -ls`：查找权限是777的文件。

`grep`：查找文件里符合条件的字符串。
`grep lang test.cfg`：在文件中查找lang字符串。
`grep lang text.cfg -color`：在文件中查找lang字符串，并高亮显示查找内容。

`grep Adrress /root/logs/catalina.2018-10-30.log -color -A1 -B1`：在/root/logs/catalina.2018-10-30.log文件中查找Address字符串，并高亮显示查找内容,且显示上一行和下一行的内容。

#### 6.**其他常用命令。**

`pwd`：显示当前所在的文件夹。
`touch a.txt`：创建一个名为a.txt的空文件。
`clear/ctrl+L`：清屏命令。

## 二、Vi和Vim编辑器：

#### 1.**Vim编辑器**

在Linux下一般使用Vi编辑器来编辑文件，vi既可以查看文件也可以编辑文件，有三种模式：**命令行、插入模式以及底行模式**。
**切换到命令行模式：按Esc键；**
**切换到底行模式：按冒号`：`**
**切换到插入模式：按i、o、a键；**

- i：在当前位置插入。
- I：在当前行首插入。
- a：在当前位置后插入。
- A：在当前行尾插入。
- o：在当前行之后插入一行。
- O：在当前行之前插入一行。

`打开文件`：vim file
`修改文件`：输入i键进入插入模式。
`保存并退出`：**esc键**切换到命令行模式，再按“:”进入到底行模式，在按**wq**。
`不保存退出`：**esc键**切换到命令行模式，在按“:”进入到底行模式，再按**q!**

#### **2.重定向输出>和>>**

`>重定向输出，覆盖原有内容；>>重定向输出，有追加功能；`
`cat /etc/passwd > a.txt`：将输出定向到a.txt中。

`cat b.txt > a.txt`：将b.txt输出内容定向到a.txt中（覆盖）。

`cat /etc/passwd >> a.txt`：输出并且追加。
`ifconfig > inconfig.txt`：将ifconfig输出内容覆盖到ifconfig文件中。

#### **3.系统管理命令**

`ps -ef`：查看所有进程
`ps -ef | grep [查找进程名]`：查找某一进程
`kill 2868` ：杀掉2868编号的进程。
`kill -9 2868`：强制杀死2868的进程。

#### **4.管道 |**

**管道是Linux命令中重要的一个概念，起作用是将一个命令输出用作另一个命令的输入。**

`ls --help | more`：分页查询帮助信息。
`ps -ef | grep java`：查询名称中包含java的进程(组合使用)，`ps  -ef`查看所有进程，在这个输出中，再用`grep java`在前面的输出中查找Java相关程序

## 三、Linux的权限命令

#### 1.**文件权限**

![](E:\大JAVA家族\黑马javaweb\课程资料\day29_Linux\一、 Linux的概述：.assets\wps40.jpg)

- 一共十位，第一位表示文件类型，一般“-”表示文件类型，“d”表示文件夹(目录)，后面三个一组分别表示当前用户，组内其他用户，和其他组用户对该文件的权限。

`r`：可读，对应4，可以对文件进行读取内容的操作，对目录是可以`ls`查看。
`w`：可写，对应2，可以对文件进行修改内容的操作，对目录是指可以创建或者删除子节点（目录或文件）。
`x`：可执行，对应1，对文件是指是否可以运行这个文件，对目录是指是否可以`cd`进入这个目录。

#### 2.**Linux三种文件类型**

`普通文件`：包括文本文件、数据文件、可执行的二进制程序文件等等。
`目录文件`：Linux系统把目录看成是一种特殊的文件，利用它构成文件系统的树形结构。
`设备文件`：Linux系统把每一个设备都看成一个文件

#### 3.文件权限管理

`chmod`：变更文件或目录的权限。
`chmod u=rwx,g=rw,o=rw a.txt`：更改a.txt文件的权限，也可以使用这个命令：`chmod 755 a.txt`用的多

## 四、Linux上常用网络操作

#### 1.**主机名配置**

`hostname`：查看主机名。
`hostname xxx`：修改主机名，重启后无效，如果想要永久生效，可以修改/etc/sysconfig/network文件。

#### **2.IP地址配置**

`ifconfig`：查看（修改）ip地址（重启后无效）。
`inconfig 网卡名称 ip地址`：修改ip地址，如果想要永久生效，请Vim修改`/etc/sysconfig/network-scripts/ifcfg-eth0`文件。

```
DEVICE=eth0 #网卡名称
BOOTPROTO=static #获取ip的方式(static/dhcp/bootp/none)
HWADDR=00:0C:29:B5:B2:69 #MAC地址
IPADDR=12.168.177.129 #IP地址
NETMASK=255.255.255.0 #子网掩码
NETWORK=192.168.177.0 #网络地址
BROADCAST=192.168.0.255 #广播地址
NBOOT=yes #  系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备。
```

#### 3.网络服务管理

`service network status`：查看指定服务服务的状态。
`service network stop`：停止指定服务。
`service network start`：启动指定服务。
`service network restart`： 重启指定服务。
`service --status-all`：查看系统中所有后台服务。
`service -nltp`：查看系统中网络进程的端口监听情况


#### 4.防火墙设置

防火墙根据配置文件/tec/sysconfig/iptables来控制本机的“出”、“入”网络访问行为。
`service iptables status`：查看防火墙的状态。
`service iptables stop`：关闭防火墙。
`service iptables start`：启动防火墙。
`service iptables off`：启动防火墙自启。

