# Linux 的文件权限
##  一、使用者与群组
- **1.文件拥有者**

     Linux是个多用户多任务的系统，会同时有多个人使用同一个主机进行工作，为考虑每个人的隐私权以及每个人喜好的工作环境，因此，这个“ **文件拥有**者 ”的角色就显的相当的重要了！

- **2. 群组概念**

     当你在团队开发资源的时候，假设小组一开发projectA，里面有成员 class1, class2, class3；小组一开发projectB，里面有成员 class4, class5, class6；这两个小组之间是有竞争性质的，但却要缴交同一份报告。每组的组员之间必须要能够互相修改对方的数据， 但是其他组的组员则不能看到本组自己的文件内容。这是就可以对文件的权限进行设置，就能达到上述的目的。这时的**小组**就相当于是一个**群组**。

- **3. 其他人的概念**

      就是不在两个小组当中的其他人。（root除外，root相当于万能的天神，拥有最高的权限）

## 二、Linux 文件权限概念
       
- **1.Linux文件属性**
> **ls**:查看文件的指令。  **su**:切换系统用户；**exit** :退出退出root用户。
     例如：切换身份为root后，使用 **la-al**命令

~~~
[dmtsai@study ~]$ su - # 先来切换一下身份看看
Password:
Last login: Tue Jun 2 19:32:31 CST 2015 on tty2
[root@study ~]# ls -al
total 48
dr-xr-x---. 5 root root 4096 May 29 16:08 .
dr-xr-xr-x. 17 root root 4096 May 4 17:56 ..
-rw-------. 1 root root 1816 May 4 17:57 anaconda-ks.cfg
-rw-------. 1 root root 927 Jun 2 11:27 .bash_history
-rw-r--r--. 1 root root 18 Dec 29 2013 .bash_logout
-rw-r--r--. 1 root root 176 Dec 29 2013 .bash_profile
-rw-r--r--. 1 root root 176 Dec 29 2013 .bashrc
drwxr-xr-x. 3 root root 17 May 6 00:14 .config &lt;=范例说明处
drwx------. 3 root root 24 May 4 17:59 .dbus
-rw-r--r--. 1 root root 1864 May 4 18:01 initial-setup-ks.cfg &lt;=范例说明处
[ 1 ][ 2 ][ 3 ][ 4 ][ 5 ][ 6 ] [ 7 ]
[ 权限 ][链接][拥有者][群组][文件大小][ 修改日期 ] [ 文件名 ]
~~~

例如： **drwxr-xr-x. 3 root root 17 May 6 00:14 .config**

- 第一栏代表文件的类型与权限：

1.第一个字符代表文件类型：
~~~
 - 当为[ d ]则是目录。
 - 当为[ - ]则是文件，例如上表文件名为“initial-setup-ks.cfg”那一行；
 - 若是[ l ]则表示为链接文件（link file）；
 - 若是[ b ]则表示为设备文件里面的可供储存的周边设备（可随机存取设备）；
 - 若是[ c ]则表示为设备文件里面的序列埠设备，例如键盘、鼠标（一次性读取设备）。
~~~

2.接下来的字符三个为一组， 均为“**rwx**” 的三个参数的组合 。

~~~
     [ r ]代表可读（read）、[ w ]代表可写（write）、[ x ]代表可执行（execute）。 
     注意：这三个权限的位置不会改变，如果没有权限，就会出现减号[ - ]而已。(这些权限都是根据账号来设计的）
   - 第一组为“文件拥有者可具备的权限”，以“initial-setup-ks.cfg”那个文件为例， 该文件的拥有者可以读写，但不可执行；
   - 第二组为“加入此群组之帐号的权限”；
   - 第三组为“非本人且没有加入本群组之其他帐号的权限”。
~~~

- 第二栏表示有多少文件名链接到此节点（i-node）：
~~~
目录树是以文件名来记录，因此每个文件名就会链接到一个i-node。
~~~

- 第三栏表示这个文件（或目录）的“拥有者帐号”

- 第四栏表示这个文件的所属群组

- 第五栏为这个文件的容量大小，默认单位为**Bytes**

- 第六栏为这个文件的创建日期或者是最近的修改日期：
~~~
[root@study ~]# ll /etc/services /root/initial-setup-ks.cfg
-rw-r--r--. 1 root root 670293 Jun 7 2013 /etc/services
-rw-r--r--. 1 root root 1864 May 4 18:01 /root/initial-setup-ks.cfg
# 如上所示，/etc/services 为 2013 年所修改过的文件，离现在太远之故，所以只显示年份；
# 至于 /root/initial-setup-ks.cfg 是今年 （2015） 所创建的，所以就显示完整的时间了。
~~~

- 第七栏为这个文件的文件名：
~~~
如果文件名之前多一个 "  . ", 则改文件为隐藏文件。
~~~

##  三、如何改变文件属性与权限
- 1. 改变所属群组, chgrp  （ change  group ）
~~~
[root@study ~]# chgrp [-R] dirname/filename ...
选项与参数：
-R : 进行递回（recursive）的持续变更，亦即连同次目录下的所有文件、目录
都更新成为这个群组之意。常常用在变更某一目录内所有的文件之情况。
范例：
[root@study ~]# chgrp users initial-setup-ks.cfg
[root@study ~]# ls -l
-rw-r--r--. 1 root users 1864 May 4 18:01 initial-setup-ks.cfg
[root@study ~]# chgrp testing initial-setup-ks.cfg
chgrp: invalid group: `testing' &lt;== 发生错误讯息啰～找不到这个群组名～
~~~
>  **注意**：要被改变的群组名称必须要在/etc/group文件内存在才行，否则就会显示错误！

- 2.改变文件拥有者, chown  ( change owner )
~~~
[root@study ~]# chown [-R] 帐号名称 文件或目录
[root@study ~]# chown [-R] 帐号名称:群组名称 文件或目录
选项与参数：
-R : 进行递回（recursive）的持续变更，亦即连同次目录下的所有文件都变更
范例：将 initial-setup-ks.cfg 的拥有者改为bin这个帐号：
[root@study ~]# chown bin initial-setup-ks.cfg
[root@study ~]# ls -l
-rw-r--r--. 1 bin users 1864 May 4 18:01 initial-setup-ks.cfg
范例：将 initial-setup-ks.cfg 的拥有者与群组改回为root：
[root@study ~]# chown root:root initial-setup-ks.cfg
[root@study ~]# ls -l
-rw-r--r--. 1 root root 1864 May 4 18:01 initial-setup-ks.cfg
~~~

> **注意：**  使用者必须是已经存在系统中的帐号，也就是在/etc/passwd 这个文件中有纪录的使用者名称才能改变。
复制文件也会复制执行者的属性与权限，假如文件用给其他使用者，那么修改这些权限就很必要了。
赋值文件命令：`[root@study ~]# cp 来源文件 目的文件`

- 3.改变权限, chmod 

* 数字类型改变文件权限
   **> r:4 > w:2 > x:1**
  语法：
~~~
           [root@study ~]# chmod [-R] xyz 文件或目录
           选项与参数：
           xyz : 就是刚刚提到的数字类型的权限属性，为 rwx 属性数值的相加。
           - R : 进行递回（recursive）的持续变更，亦即连同次目录下的所有文件都会变更
~~~

* 符号类型改变文件权限
       **u(user ), g( group ), o (other )**  来代表三种身份的权限！此外， **a ( all )** 亦即全部的身份！那么读写的权限就可以写成  **r,  w,   x** 。
语法：
~~~
| chmod | u g o a | +（加入） -（除去） =（设置） | r w x | 文件或目录 |
~~~

例如：
~~~
   [root@study ~]# chmod u=rwx,go=rx .bashrc
   # 注意喔！那个 u=rwx,go=rx 是连在一起的，中间并没有任何空白字符！
~~~

## 四、目录与文件之权限意义
- 权限对文件的重要性
文件是实际含有数据的地方，包括一般文本文件、数据库内容档、二进制可可执行文件（binary program）等等。 因此，权限对于文件来说，他的意义是这样的：
~~~
    r （read）：可读取此一文件的实际内容，如读取文本文件的文字内容等；
    w （write）：可以编辑、新增或者是修改该文件的内容（但不含删除该文件）；
    x （eXecute）：该文件具有可以被系统执行的权限。
~~~

- 权限对目录的重要性
~~~
    r :  表示具有读取目录结构清单的权限.
    w:   创建新的文件与目录；
          删除已经存在的文件与目录（不论该文件的权限为何！）；
          将已存在的文件或目录进行更名；
          搬移该目录内的文件、目录位置。 
    x: 使用者能否进入该目录成为工作目录的权限。
~~~


## 五、Linux文件种类与扩展名

- 1. 正规文件 
~~~
   *  纯文本文件（ASCII）：内容为我们人类可以直接读到的数据，例如数字、字母等等。
   *  二进制档（binary）：Linux当中的可可执行文件。
   *  数据格式文件（data）：特定格式的文件可以被称为数据文件。
~~~

- 2. 目录：  第一个属性为 [ d ].

- 3.链接文件（link）:第一个属性为 [ l ]。

- 4.  设备与设备文件（device）: 
~~~
     区块（block）设备文件: 就是一些储存数据， 以提供系统随机存取的周边设备，举例来说，硬盘与软盘等 。第一个属性为[ b ]
     字符（character）设备文件 ： 一些序列埠的周边设备， 例如键盘、鼠标等等 。
~~~

- 5. 数据接口文件（sockets）：这种类型的文件通常被用在网络上的数据承接。 第一个属性为 [ s ] 。

- 6.数据输送档（FIFO, pipe）：解决多个程序同时存取一个文件所造成的错误问题。第一个属性为[p]  。

- Linux文件扩展名：Linux的文件是没有所谓的“扩展名”的，一个Linux文件能不能被执行，与他的第一栏的十个属性有关， 与文件名根本一点关系也没有。
常用扩展名：
~~~
        *.sh ： 脚本或批处理文件 （scripts），因为批处理文件为使用shell写成的，所以扩展名
     就编成 .sh 啰；
        Z, .tar, .tar.gz, .zip, *.tgz： 经过打包的压缩文件。这是因为压缩软件分别为 gunzip, tar
      等等的，由于不同的压缩软件，而取其相关的扩展名啰！
         .html, .php：网页相关文件，分别代表 HTML 语法与 PHP 语法的网页文件啰！ .html 的
      文件可使用网页浏览器来直接打开，至于 .php 的文件， 则可以通过 client 端的浏览器来server 端浏览，以得到运算后的网页结果呢！
~~~

- Linux文件长度限制：单一文件或目录的最大容许文件名为 255Bytes。

- Linux文件名称的限制：避免使用以下特殊字符
> ? > < ; & ! [ ] | \ ' " ` （ ） { }
