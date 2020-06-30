# Linux 的文件权限与目录配置
##  一、使用者与群组
- **1.文件拥有者**

     Linux是个多用户多任务的系统，会同时有多个人使用同一个主机进行工作，为考虑每个人的隐私权以及每个人喜好的工作环境，因此，这个“ **文件拥有**者 ”的角色就显的相当的重要了！

- **2. 群组概念**

     当你在团队开发资源的时候，假设小组一开发projectA，里面有成员 class1, class2, class3；小组一开发projectB，里面有成员 class4, class5, class6；这两个小组之间是有竞争性质的，但却要缴交同一份报告。每组的组员之间必须要能够互相修改对方的数据， 但是其他组的组员则不能看到本组自己的文件内容。这是就可以对文件的权限进行设置，就能达到上述的目的。这时的**小组**就相当于是一个**群组**。

- **3. 其他人的概念**

      就是不在两个小组当中的其他人。（root除外，root相当于万能的天神，拥有最高的权限）

## 二、Linux 文件权限概念
       
- **1.Linux文件属性**
> **ls**:查看文件的指令。  **su**:切换系统用户。
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


